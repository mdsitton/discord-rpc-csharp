# Discord Rich Presence

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/a3fc8999eb734774bff83179fee2409e)](https://app.codacy.com/app/Lachee/discord-rpc-csharp?utm_source=github.com&utm_medium=referral&utm_content=Lachee/discord-rpc-csharp&utm_campaign=badger)

This is a C# _implementation_ of the [Discord RPC](https://github.com/discordapp/discord-rpc) library which was originally written in C++. This avoids having to use the official C++ and instead provides a managed way of using the Rich Presence within the .NET environment*.

This library supports all features of the Rich Presence that the official C++ library supports, plus a few extra:
 - **Message Queuing**
 - **Threaded Reads**
 - **Managed Pipes***
 - **Error Handling** & **Error Checking** with automatic reconnects
 - **Events from Discord** (such as presence update and join requests)
 - **Full Rich Presence Implementation** (including Join / Spectate)
 - **Inline Documented** (for all your intelli-sense needs)
 - **Helper Functionality** (eg: AvatarURL generator from Join Requests)
 - **Ghost Prevention** (Tells discord to clear the RP on disposal)

_*Managed Pipes is unavailable for the Unity3D game engine due to technical limitations and a substitute native wrapper library is required._

_*Proper Documentation outside of inline is planned and will be implemented hopefully soon_

_*HTTP Support has been removed as Discord removed their implementation. Talks of bringing it back and this library will update if it happens._

# Installation
Within the Visual Studio solution, there are 3 projects. The main library is located in `Discord RPC`, the example project is `DiscordRPC.Example` and a native pipe wrapper is `DiscordRPC.Native`. 

If you are using Unity3D, you can download the Unity Package for a quick setup [DiscordRPC_Unity.unitypackage](https://github.com/Lachee/discord-rpc-csharp/raw/master/DiscordRPC_Unity.unitypackage). This maybe a older version of the project.

**Dependencies:**
 - Newtonsoft.Json 
 - .NET 3.5* or above
 
_*Unity3D can run in .NET 2.0, however it **cannot** run in .NET 2.0 __subset___
_**Unity3D requires the `DiscordRPC.Native` library to be built_

**Standard .NET (build)**
For standard use, the only project that needs to be build is `Discord RPC`. This will create a managed .dll for use in your .NET projects. Simply right-click the project and click build. The .dll will be generated at `<project>/Discord RPC/bin`. Copy this .dll to your target project and add it as a reference.

**Standard .NET (nuget)**
There is currently no nuget package available. One will be released in the future when this project is in a stable state.

**Unity3D Game Engine**
Download the Unity Package for a quick setup [DiscordRPC_Unity.unitypackage](https://github.com/Lachee/discord-rpc-csharp/raw/master/DiscordRPC_Unity.unitypackage). This maybe a older version of the project.

Unity3D has a limitation with its managed version of pipes. It does not support asynchronous mode (even in .NET 4.6) and will block waiting for a read to finish. Aborting a read will cause a hard freeze of the editor.

This library provides a solution that should only be used when this problem occurs. It is a native library and as such is platform specific and needs to be built for every planned release platform (win32, win64, mac, linux). 

To build the library with the native pipes, you want to build `Discord RPC` and `DiscordRPC.Native`. Copy both the .dll generated by the libraries and paste them within your project folder. If you are using the `.NET 2.0` within unity, you are required to copy the `DiscordRPC.Native` dll to the root of the project folder (outside `assets`).


## Usage

The Discord.Example project within the solution contains example code, showing how to use all available features. For Unity Specific examples, check out the example project included. 

**Connecting**
```csharp
public DiscordRpcClient client;

//Called when your application first starts.
//For example, just before your main loop, on OnEnable for unity.
void Initialize() 
{
	/*
	Create a discord client
	NOTE: 	If you are using Unity3D, you must use the full constructor and define
			 the pipe connection as DiscordRPC.IO.NativeNamedPipeClient
	*/
	client = new DiscordRpcClient(ClientID, true, DiscordPipe))					
	
	//Set the logger
	client.Logger = new ConsoleLogger() { Level = LogLevel.Warning };

	//Subscribe to events
	client.OnPresenceUpdate += (sender, e) =>
	{
		Console.WriteLine("Received Update! {0}", e.Presence);
	};
	
	//Connect to the RPC
	client.Initialize();

	//Set the rich presence
	client.SetPresence(new RichPresence()
	{
		Details = "Example Project",
		State = "csharp example",
		Assets = new Assets()
		{
			LargeImageKey = "image_large",
			LargeImageText = "Lachee's Discord IPC Library",
			SmallImageKey = "image_small"
		}
	});	
}
```

**Invoking Events**
Make sure you call invoke regularly to dequeue stored messages received by discord. You can either call `Invoke()` to call the events or manually dequeue and process the messages with `DequeueMessages()`.
```csharp
//The main loop of your application, or some sort of timer.
void Update() 
{
	//Invoke all the events, such as OnPresenceUpdate
	client.Invoke();
}
```

**Disposing**
Its important that you dispose your client before you application terminates. This will stop the threads, abort the pipe reads and tell discord to clear the presence. Failure to do so may result in a memory leak!
```csharp
//Called when your application terminates.
//For example, just after your main loop, on OnDisable for unity.
void Deinitialize() 
{
	client.Dispose();
}
```

## Build script parameters
The following parameters can be passed to the build script:
1. **`target`**  
Valid parameters are:
- `Default` has the same effect as executing the script without the parameter
- `NugetBuild` will build a Nuget pacakge and attempt to push it to the Nuget repository (This is geared for running from a build server like AppVeyor)
2. **`ScriptArgs`**  
Can be used to pass one or both of the following build arguments:
- `buildCounter` is used to know how many builds the server has generated and is used to generate a CI version number for [continuous Nuget releases](https://www.xavierdecoster.com/post/2013/04/29/semantic-versioning-auto-incremented-nuget-package-versions.html) - Defaults to 0
- `buildType` can be used to pass the MSBuild configuration that should be execute - Defaults to `Release`

To run the script on a build server, pass the following line:
```
.\build.ps1 -target NugetBuild -ScriptArgs '-buildCounter=1','-buildType="Release"'
```
