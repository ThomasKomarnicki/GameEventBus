# GameEventBus
A simple EventBus library in C# for Unity

# Basic Usage

```c#

// Create a unique class for each type of event
class PlayerJumpEvent : EventBase {
  public Vector3 jumpPosition;

  public PlayerJumpEvent(Vector3 pos) {
    jumpPosition = pos;
  }
}

private void TestMethod() {
  IEventBus eventBus = new EventBus();
  eventBus.Subscribe<PlayerJumpEvent>(OnPlayerJump); 
   
  eventBus.Publish(new PlayerJumpEvent(new Vector3(1,1,0)));// OnPlayerJump will be invoked

  eventBus.Unsubscribe(OnPlayerJump);
}

private void OnPlayerJump(PlayerJumpEvent event) {
  Console.WriteLine(playerJumpEvent.newPosition);
}

```
Each event subscription must be removed by calling EventBus.Unsubscribe(actionDelegate). This means in order to
unsubscribe from an event, you must keep a reference to your action delegate, or use a function reference.
Make sure to remove a subscription whenever it's script is going to be destroyed to avoid memory leaks.

# Example in Unity

First, we can create a global instance of our EventBus in a global GameController or similarly scoped object.

```csharp
using GameEventBus;

public class GameController : MonoBehaviour {
  public static EventBus Bus = new EventBus();
}
```

Lets say we have a Player game object and script that responds when a collider with the tag "bullet" collides with it.
When your Player collides with a bullet, you can publish a BulletCollision event, with information about the collision.

```csharp

public class Player : Monobehaviour {
  void OnCollisionEnter(Collision collision) {
    if(collision.gameObject.tag == "bullet") {
      GameController.Bus.Publish<BulletCollision>(new BulletCollision(collision));
    }
  }
}


using GameEventBus.Events;

public class BulletCollisionEvent : EventBase {
  public Collision collision;

  public BulletCollisionEvent(Collision collision) {
    this.collision = collision;
  }
}

```
All of our scripts that subscribe to the BulletCollision event will be notified whenever our Player is hit with
a Bullet

```csharp

public class Hud : Monobehaviour {
  void Start() {
    GameController.Bus.Subscribe<BulletCollisionEvent>(OnBulletCollision);
  }

  void OnBulletCollision(BulletCollisionEvent event) {
    Debug.Log("Our HUD was notified of a bullet collision!")
  }

  void OnDestroy() {
    GameController.Bus.Unsubscribe(OnBulletCollision);
  }
}

```
