---
title: "CryEngine - ECStasy - Part 2"
date: 2024-11-26T09:47:34-05:00
draft: false
---

Let's take a look at the Entity class. When CE creates a new one, it receives Entity class and Archetype in the PreInit call, this call also will create Proxy if needed (the call will take it from the Class). The next call is the Init, inside it casts Class to *CEntityClass*, and tries to serialize if XML exists. Serialize will call LoadComponent. *CEntity* has CreateProxy method which can create different components like *CEntityComponentAudio, CEntityComponentArea, CEntityComponentCamera* etc


![](/img/november-ce/01.jpg)

This class also has a few handy methods like:

*AddComponent*—The call will check *Class* desc from passed params. If desc is null, the method will try to get the factory for the component. Finally, we have the *AddComponentInternal* call with different parameters.

![](/img/november-ce/02.jpg)

*CEntity* holds the pointer to the *CEntityRender* (we will take a look on this class later), and has some public proxy calls like LoadGeometry, LoadCharacter, and LoadLight. They will call the same methods on the *CEntityRender* pointer.

Let's step out and check who controls Entities—*CEntityLoadManager*. This class uses *EntityPool*, which is the vector of units. *LoadEntites* is the first interesting call. 

It's getting XML as an argument, and will reserve the entities' ids - simple iteration on the XML and parsing the id - if the id and tag are valid - we ask IEntitySystem to reserve the id. Next CE starts batch creation - the code is really self-explanatory

![](/img/november-ce/03.jpg)

![](/img/november-ce/04.jpg)

Let's take a look at *ParseEntities*. It reserves *StaticEntity* ids with the g_pIEntitySystem->ReserveStaticEntityIds call. We have counted from XML. Next, CE iterates on the XML children. If the node exists and the tag is valid, CE tries to parse the XML. If it gets success, the call will try to extract *LoadParams* with the ExtractEntityLoadParams call. We will put this data into *loadParameterStorage*.

Out of the iteration, CE reserves memory in the pool and will iterate on loadParameterStorage instances. Finally, we have a *CreateEntity* call.

Step inside to the *CreateEntity* call. 

The first call we have is g_pIEntitySystem->SpawnPreallocatedEntity. Next, we are trying to restore components by the XML. That's pretty generic code the interesting thing here is the pSpawnedEntity->LoadCharacter or LoadGeometry calls.

Let's check the Archetype.

First, we want to check the *CEntityArchetype*. The constructor gets the *IEntityClass*, and will try to get *CEntityScript*, if it exists we will try to load it (Lua interactions).

![](/img/november-ce/05.jpg)

That's a pretty basic class  - it has its own methods to load itself.

Now we want to check *CEntityArchetypeManager* - this class has a *CreateArchetype* call, it gets *Class* and archetype name, and the manager has a cache for types. The call will try to create a new CEntityArchetype instance *if* we don't have n in the cache. For this, we have to cast the class to *CEntityClass* and pass it to the constructor. Another interesting call is the *LoadArchetype* with type name as argument. This call checks if we have loaded the library for the passed archetype name. If no - CE is trying to load the library for the given archetype name.

*LoadLibrary* is pretty straightforward - load XML by the library name (CE constructs it from the archetype name). Get the *СlassRegistry* from g_pIEntitySystem. Next CE will be iterating on the XML, will try to extract the Name and Class, check if it's in the registry, and will create a new archetype withthe  *CreateArchetype* call. And will add the library to the cache at the end of the call.

![](/img/november-ce/06.jpg)

Let's check the new discovery - *CEntityClassRegistry*. It has a few interesting calls, let's check the first one *RegisterEntityClass*. It gets the Class as an argument and will be storing the class in the map by the guid. *FindClass*/*FindClassByGUID* - simple search. *RegisterStdClass* will create a new *CEntityClass* instance, if passed as an argument descriptor have the script file - CE will load the *CEntityScript* and will set it to the *CEntityClass* instance. At the end, we have a *RegisterEntityClass* call.

Last interesting call here - *InitializeDefaultClasses*. 

![](/img/november-ce/07.jpg)

We observed 2 more classes - *CEntityClass* and *CEntityScript*. Let's start with the first one.

The first method that takes our attention in *CEntityClass* is the *LoadScript*. If the *CEntityScript* member is not null CE will try to load the script. Seems like *CEntityScript* is optional, so there exists a *SetEntityScript* call. The class also has *GetEventInfo* call, which will extract the vent from the *CEntityScript* member.

And seems like all ways coming to the *CEntityScript*. The main initialization in the Init call. It has 2 versions - one with only names for table and scriptfile, second allow you to pass the table object. Also this version will delegate properties to this table and will create another table for properties. 

Another interesting method is the *LoadScript*. It will try to execute file by the filename you passed before. will delegate properties and will call the EnumStates.

*EnumStates* wil lconnect al lstates Lua has, so seems like connections to some state machine from Lua. At the end every stated get callback from Lua in the InitializeStateTable. You will see these calls later when we will be researching the *CEntityComponentLuaScript*

![](/img/november-ce/08.jpg)

With all this knowledge let's review the *CreateProxy* method in the *CEntity* class.

*CEntityComponentAudio* is the simplest one, Initialize method will extract and will change the name. After that we have *CreateObject(SCreateObjectData)* on pAudioSystem pointer. Another interesting method is the *ProcessEvent*. It will parse the event type and will notify about it.

![](/img/november-ce/09.jpg)

*CEntityComponentCamera* is another interesting but marked as a legacy component, *UpdateMaterialCamera* is probably the most interesting call here.

![](/img/november-ce/10.jpg)

*CEntityRender* is the class *CEntity* holds, we still have not reviewed it, so let's fix this. By the comment, this class contains multiple subobjects within the slots. A few interesting methods here - *AllocSlot(index)* - resize + new slot if no space, *IsRendered* flag - will ask slots to determine the value, *SetSlotLocalTM*, *UpdateRenderNodes* - proxy to slot, *SetSlotGeometry*(index) - pass to slot.

Another good example - *LoadGeometry(index, name).* It will try to create the slot if it doesn't exist, Will load the *Object* for the filename with the *I3DEngine* global variable. And will call *SetSlotGeometry*.

Next is *LoadCharacter* - pretty similar to the method above but data from *ICharacterManager*. At the end, we have SetSlotCharacter call.

This class has a few similar calls like *SetSlotGeometry*, *SetSlotCharacter*, *SetParticleEmitter*, *LoadParticleEmitter*, *LoadLight*, *LoadCloudBlocker*, *LoadFogVolume*, *LoadGeomCache*... 

They are all pretty similar - *GetOrMakeSlot*, *slot->SetRenderNode()* *ComputeBounds*, and *SendEvent(ENTITY_EVENT_SLOT_CHANGED).*

The final class in this hierarchy is the *CEntitySlot*, let's check it, as the comment suggests - defines a particular geometry object or character inside the entity. 

The constructor takes the *CEntity* pointer, it has a *ReleaseObjects* call - cleanup of object vars and also call to *p3DEngine* to delete *RenderNode*. *UpdateRenderNode* - it will try to update some flags and will try to insert node to octree with p3DEngine->RegisterEntity call. 

Another interesting call - SetAsLight. CE creates the *RenderNode* with gEnv->p3DEngine->CreateLightSource(). will cast it to *ILightSource*. And will set light properties.

The last Script we will check is the *CEntityComponentLuaScript*, it stores a pointer to *CEntityScript*. This class also can create *ScriptTable*, this class is interesting because it shows some internal Lua work, for example, it works directly with "*__this*" and to *ScriptHandle* as well as "id" and "class". This class has an Update call, but it checks the time difference, if we have enough span - we call *CallStateFunction* on the *CEntityScript* object.

![](/img/november-ce/11.jpg)

So the *ProcessEvent* is the most interesting method here, with respect to the type of the event we redirect calls to the *CEntityScript*, so the call takes *CurrentState*, *ScriptTable*, and *ScriptStateType*

![](/img/november-ce/11.jpg)

*GoToState* is an interesting example of Lua interactions in CE. It calls *CallStateFunction* with *ScriptState_OnEndState*. Sends the event with ENTITY_EVENT_LEAVE_SCRIPT_STATE. Calls the *CallStateFunction* with *ScriptState_OnBeginState*. Checks if the current state has an Update method with CurrentState()->IsStateFunctionImplemented(ScriptState_OnUpdate) call. And finally sends the event with ENTITY_EVENT_ENTER_SCRIPT_STATE.

Some additional helpers exist like: 

*IsInState(name)* - it will calls the *CEntityScript->GetStateId()*

*SetEventTargets(XML)* - will connect source targets with event targets

Family of public *CallEvent* calls - simple proxy to the *CEntityScript*.

*OnCollision* - it's interesting because it works with physics.

And the SetPhysParams. Proxy to the g_pIEntitySystem->GetScriptBindEntity()->SetEntityPhysicParams.



That's it for this side of the hierarchy, I hope you found this useful.

Good luck.
