---
title: How I Do Factories In Unity
date: 2024-02-26T00:00:00
draft: false
description: First post documenting how I do project architecture in Unity.
post_type: blog
tags:
  - GameDev
---

I spent this week backfilling unit tests and have no progress to show off. But, I've been meaning to document the patterns I use to build my game architecture for about a year now so here goes with the first one.

This is not an opinionated guide on how to do factories, probably not even the best pattern for it, but *it is the way I like to do it*.

## What is a factory?

[Wikipedia](https://en.wikipedia.org/wiki/Factory_%28object-oriented_programming%29) says:

> In object-oriented programming, a factory is an object for creating other objects

Which pretty much covers it, you have a thing who's responsibility is purely to create the objects that other things in your code need to do their jobs.

IMO, a factory has another function. It's also the place where you put your [smelly code](https://en.wikipedia.org/wiki/Code_smell). The majority of the ugly things you can't avoid but don't want to include in your lovely clean modules. For example, you need to create a different subclass of a things based off an enum value, and you don't put the ugly switch statement into your good code, you put it in your factory.

When you can't avoid doing something dirty, try push it as close to the entry point as possible.

## How I structure modules & use factories

I like to separate different things in my games into modules. For example I might have the following 3 modules:

- **Game module:** Handles loading scenes, player preferences and save data.
- **Level Module:** Handles the higher level state that happens in a level, such as the start screen, playing, pause, respawn etc.
- **Player Module:** Handles the in game player character.

Each of these modules should be more or less self contained. They would have their own factory, own controllers etc:

```txt
|- Scripts
	|-Game
		|-GameBehaviour.cs (MonoBehaviour)
		|-GameController.cs
		|-GameFactory.cs
		|-GameState.cs
		|-SaveController.cs
		|-SceneController.cs
	|-Level
		|-LevelBehaviour.cs (MonoBehaviour)
		|-LevelController.cs
		|-LevelFactory.cs
	|-Player
		|-InputController.cs
		|-PlayerBehaviour.cs (MonoBehaviour)
		|-PlayerController.cs
		|-PlayerFactory.cs
```

Ideally all these classes should implement an interface, but for clarity I've left those out.

The `*Behaviour` classes are Unity MonoBehaviours that act as the entry point (or a facade) to the modules. Modules interact with each other through these.

Each module has a main controller class, (eg `GameController`) and various sub-controllers. I like to make the main controller implement the same interface as the Behaviour/Facade and pass everything through to it.

This is an example of what that code would look like:

`GameController.cs`
{.code-caption}
```cs
public class GameController : MonoBehaviour, IGameController
{
	[SerializeField] private float autosaveInterval;
	[SerializeField] private LoadingScreen loadingScreen;
	
	private GameFactory _factory;
	private GameController _controller;

	private void Start()
	{
		_factory = new GameFactory(autosaveInterval, loadingScreen);
		_controller = _factory.Controller;
	}

	private void Update() => _controller.Update(Time.time);
	public void OnLoadScene(string sceneName) => StartCoroutine(_controller.OnSomeEvent(sceneName));
}
```

`GameFactory.cs`
{.code-caption}
```cs
public class GameFactory
{
	private float _autosaveInterval;
	private ILoadingScreen _loadingScreen;
	
	public GameFactory(
		float autosaveInterval,
		ILoadingScreen loadingScreen
	) {
		_autosaveInterval = autosaveInterval;
		_loadingScreen = loadingScreen;
	}

	private GameController _controller;
	public GameController => _controller ??= new GameController(
		SceneController,
		SaveController,
		_autosaveInterval
	);

	private SceneController _sceneController;
	private SceneController => _sceneController ??= new SceneController(
		GameState,
		_loadingScreen
	);

	private SaveController _saveController;
	private SaveController SaveController ??= new SaveController(
		GameState
	);

	private GameState _gameState;
	private GameState GameState => _gameState ??= new GameState();
}
```

`GameController.cs`
{.code-caption}
```cs
public class GameController : IGameController
{
	private ISceneController _sceneController;
	private ISaveController _saveController;
	private float _autosaveInterval;

	private float _nextSaveTime;

	public GameController(
		ISceneController sceneController,
		ISaveController saveController,
		float autosaveInterval
	) {
		_sceneController = sceneController;
		_saveController = saveController;
		_autosaveInterval = autosaveInterval;
	}

	public void Update(float time)
	{
		// Don't save on first update
		if(_nextSaveTime == 0) _nextSaveTime = time + _autosaveInterval;
		
		if(time > _nextSaveTime)
		{
			_saveController.Save();
			_nextSaveTime = time + _autosaveInterval;
		}
	}

	public IEnumerator LoadScene(string sceneName)
	{
		yield return _sceneController.LoadScene(sceneName);
		_saveController.SetLastScene(sceneName);
		_saveController.Save();
	}
}
```

This is basically an extension of the [Humble Object Pattern](https://blog.unity.com/technology/unit-testing-part-2-unit-testing-monobehaviours), but with the object creation handed off to the factory. As you can see the MonoBehaviour has basically become a container that passes lifecycle and events to the main controller, which then handles everything.

## Things I like about this pattern
### 1. Testability

MonoBehaviours are hard to test. This pattern makes it way easier, especially if you use NSubstitute:

`GameControllerTest.cs`
{.code-caption}
```cs
[Test]
public void OnUpdate_AutosavesAfterInterval()
{
	var saveController = Substitute.For<ISaveController>();
	var gameController = new GameController(
		Substitute.For<ISceneController>(),
		saveController,
		autosaveInterval: 2
	);

	gameController.Update(0);
	gameController.Update(1);
	gameController.Update(2);
	gameController.Update(2.1f);

	Assert.AreEqual(1, saveController.ReceivedCalls().Count());
}

[UnityTest]
public IEnumerator OnLoadScene_LoadsScene()
{
	var sceneController = Substitute.For<ISceneController>();
	var gameController = new GameController(
		sceneController,
		Substitute.For<ISceneController>(),
		autosaveInterval: 1
	);
	
	yield return gameController.LoadScene("New Scene");

	sceneController.Recieved().LoadScene("New Scene");
}
```
### 2. Lazy Instantiation

If you look in the `GameFactory` you can see that the `GameState` is injected into both the `SceneController` and `SaveController`. Usually you'd need to ensure that the `GameState` object is instantiated before trying to instantiate the controllers, but we can get around that by using backed properties and the null coalescing assignment operator.

What that means is the first thing to require the `GameState` object will cause it to be instantiated, then all others will use that same object.

### 3. Finite State Machine State Creation

If your controller is using an FSM implementation you can use a factory to instantiate your states rather than doing it inside your object.

What I like to do with this is define an interface for the state factory and inject that into the controller:

`IGameStateFactory.cs`
{.code-caption}
```cs
public interface IGameStateFactory
{
	public IState NewMainMenuState();
	public IState NewLevelState(string sceneName);
}
```

`GameController.cs`
{.code-caption}
```cs
public class GameController
{
	private IStateMachine _stateMachine;
	private IStateFactory _stateFactory;

	public GameController(
		IStateMachine stateMachine,
		IStateFactory stateFactory
	) {
		_stateMachine = stateMachine;
		_stateFactory = stateFactory;
	}

	public IEnumerator Start()
	{
		yield return _stateMachine.StartWithState(
			_stateFactory.NewMainMenuState()
		);
	}

	public void Update(float deltaTime)
	{
		_stateMachine.CurrentState.Update(deltaTime);
	}

	public IEnumerator OnLoadLevel(string sceneName)
	{
		yield return _stateMachine.MoveToState(
			_stateFactory.NewLevelState(sceneName)
		);
	}
}
```

This also allows easy testing, because you can create a substitute for your `GameStateFactory` and check which states were asked for.
### 4. Instantiating Game Objects

You can also push Game Object instantiation into your factories and let them do all the required hookups and initialisations.

`LevelFactory.cs`
{.code-caption}
```cs
private IPlayer _player;  
private IPlayer Player => _player ??= InstantiatePlayer();

private IPlayer InstantiatePlayer()  
{  
    var player = Object.Instantiate(_components.PlayerPrefab);  
    player.transform.position = _components.SpawnPoint.position;  
    // Hook up cameras  
    player.Initialise(new Transform(_components.MainCamera.transform));  
    _components.MainCamera.SetLookTarget(player.transform);  
    return player;  
}
```

Now my code doesn't need to worry about if the player exists in the scene, it'll be created automatically.

## Wrap Up

That's about it, if you want to ask me any questions you can reach me on [Mastodon](https://mastodon.ie/@abban).

I haven't covered how I pass in and use core Unity components inside my code, I might write that up next (Hint: Interface everything).