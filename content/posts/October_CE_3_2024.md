---
title: "CryEngine - ECStasy - Part 1"
date: 2024-10-19T17:47:26-04:00
draft: false
---

A little later but here is the continue of the CRYENGINE research. Next we are going to check *IScryptSystem*, this interface has a few methods, and also *CScriptSystem* module is implementing this interface. So at runtime, we load DLL.

This module works with Lua, that's a really interesting implementation I will try to explain some details, but the explanation for now would not be complete, because the actual implementation works with at least 3 modules, but we have to start somewhere.

So *CScriptSystem* has an Init call, let's take a closer look:

- It initializes some Lua context, just some direct calls to Lua, 
- Some Meta tables initialization (for now that's not really clear what they do besides caching some Lua buffers)
- Additional initialization + subscription to external Lua commands 
- Initialization of the debugger (yes they have their own one)
- It executes file "*script/common.lua*", again that's probably some Lua side initialization, executes call read file data through CryPak (some asset reader), and calls ExecuteBuffer which calls all Lua-related functions. And doing all Lua-related calls you expect for buffer. This file is not in the source code folder, so they are probably generated on the first run or just missed.

I put this last, but somewhere in the middle, we initialize *CScriptBindings*, that's where we are going to see interesting code. *CScriptBindings* wraps a few basic *CSCriptBind_...* implementations, like: *CScriptBind_System, CScriptBind_Particle,  CScriptBind_Sound, CScriptBind_Movie, CScriptBind_Script, CScriptBind_Physics*.

![17](/img/october-ce/17.jpg)

All these classes are *CScriptableBase* children, which hold some basic functions to work with Lua. For example, it has *RegisterFunction* call, which registers calls on the Lua side, and they never intend to be called from the C side. For example *CScriptBind_System* - has a few pretty System calls, like *Log, LogToConsole, Warning, IsEditor, GetEntity, ShowConsole, ShowDebugger, BrowseURL, Quit, GetFrameID.* As you can see they are pretty high level, and implement something Lua scripts expect you from System level calls.

![18](/img/october-ce/18.jpg)

That's really interesting architecture, and I'm being honest never thought about a solution like this. I'm a fan of the plugins centric solutions, and I really like how CRYENGINE has done this.

*RegisterFunction* is doing a few things - adding your func to m_pMethodsTable. Which is CScriptTable class, and wrapper on Lua calls. It has *AddFunction* call, which is a wrapper on Lua calls, and at the end of the method it has *lua_pushcclosure*. Which will register your call on Lua's side.

![19](/img/october-ce/19.jpg)

Let's take a look at who else has *CScriptableBase* as a parent class. A really interesting one is *CScriptBind_Entity*, from the header comment we should expect all entity-related stuff. As you could already think, the constructor will register everything to Lua with *RegisterFunction*. You can see calls like *LoadObject, LoadCharacter, LoadGeomCache, LoadLight, GetPos, GetAngles, GetLocalPos, SetLocalPos, GetWorldPos, DestroyPhysics, GetWorldBBox, IsHidden, Damage, SetMaterial etc...* Once again, that's pretty high level and there are a lot of different calls, for everything you want to do with entity in the scene.  This class has ~275 calls to GetEntity, in every call from Lua, seems like there is no guarantee on the pointer we getting from IEntitySystem.

After that, we have some additional init calls, not really important for our research right now, and we have subscriptions on different Update calls (UpdateNever, Visible, ..., Always). 

Let's take a look at a few examples of callbacks:

- *LoadObject* allows to loading of different files - character or geometry, these files has own extensions, after that call redirect call to CEntity instance.
- *LoadLight* just doing redirect to *CEntity::LoadLight()*
- *AddImpulse* some inline parsing, probably for performance, and after that code extracts *IPhysicalEntity* from *CEntity* and redirects the call it if the component is available.
- *GoToState* calls *GetProxy(ENTITY_PROXY_SCRIPT)*, it will return object which implements *IEntityScriptComponent* and redirect the call it if component is available.

AI has a few children of *CScriptableBase*, like *CScriptBind_AIAction, CScriptBind_GameAI*. But right now that's our of our research.

Other examples: For environment, we have *CScriptBind_LightingArc, CScriptBind_InteractiveObject.* I will skip them for now.

And last example we will check: *CScriptBind_Game, CScriptBind_GameRules, CScriptBind_Item*

- *CScriptBind_Game* has some usual registration calls like *ShowMainMenu, Pause, etc...* Also it has some high level calls like *LoadPrefabLibrary*. This call will extract prefabManager from WorldBuilder and will Load the prefab library for you.
- *CScriptBind_GameRules* creates some collections for players and other instances, this class knows about a network, and that's interesting, to be honest. And this class has some calls like *IsServer, IsClient, ServerExplosion*
- *CScriptBind_Item* some high level logic like calls: *Reset, CanPickup, CanUse, Use, IsInUse, OnUsed, AllowDrop, DissalowDrop*. All these methods extract *CItem* from *CItemSystem*.

So that's all for today, In general, I really like the architecture behind this, I like how CRYTEK (Timur) implemented this system, like there is a facade between Lua code and actual logic, and you can change one side w/o changing another. Probably even at runtime (not sure about their code, but feels like it should be). The similar implementation can work for you in almost any project, I have used it a lot, Unity uses it, and you can find some details in implementation in ARFoundation's Subsystem Management plugin.

Next time we will take a look at CEntitySystem and CItemSystem.

![20](/img/october-ce/20.jpg)

