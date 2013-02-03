# FastTEvent
This library is designed to be a fast and easy way to handle your events.

# Author
Miguel Martin - [miguel.martin7.5@hotmail.com](mailto:miguel.martin7.5@hotmail.com)

# Usage

## Event Senders

An event sender is used to send events to multiple call-backs immediately. 

### 1. Create an Event Sender Object

To create an event sender, you create an object of the class ``EventSender<T>``, where T is the signature of your call-backs. The syntax is the same as ``std::function<T>``.

#### Example

```
// Create a sender object that sends an int
EventSender<void (int)> sender;
```

#### Note

---
It is reccomended to have a return value that is void, as returning a value with multiple call-backs does not make much sense.

---

### 2. Subcsribing/Un-subcsribing to Events

To subcsribe/un-subscribe to events that will be emitted/sent, you simply call ``add()`` to subscribe and ``remove()`` to un-subscribe. The ``add()`` and ``remove()`` functions work for both member-functions and global functions, since it is overloaded.

To use add/remove with a global function, you simply pass the address of the function to add/remove.

#### Example

```
void doSomething(int x)
{
	std::cout << x << '\n';
}

// ...

// Create a sender
EventSender<void (int)> sender;

// subcsribe
sender.add(&doSomething);

// un-subscribe
sender.remove(&doSomething);
```

To use add/remove with a member-function, you must first pass a pointer to the object you wish to bind the event to and then the member-function.

#### Example

```
struct ExampleClass
{
	void doSomething(int x)
	{
		std::cout << x << '\n';
	}
};

// ... 

// create our object that will subscribe to events
ExampleClass obj;

// create a sender
EventSender<void (int)> sender;

// subscribe
sender.add(&obj, &ExampleClass::doSomething);

// un-subscribe
sender.remove(&obj, &ExampleClass::doSomething);
```

### 3. Emitting events

To emit events to your subcribers (call-backs), you simply call the ``emit(T...)`` function. Where ``T...`` is the paramters of your events.

#### Example:

```
// create a sender
EventSender<void (int)> sender;

// subcsribe to events...
// do other things 

// emit the event
sender.emit(32);
```  

## Event Queues

An event queue is used for when you don't want to send events out immediately. 

### 1. Create an Event Data Structure

In order to create an event queue, you must have a data structure that resembles the data you will be sending for your events. Typically, all data in the data structure will be const and assigned through a constructor, and there will be no other methods.

#### Example:

```
struct CollisionEvent
{
	const Entity& obj1, obj2;
	CollisionEvent(const Entity& Obj1, const Entity& Obj2)
			: obj1(Obj1), obj2(Obj2) 
	{
	}
};
```

#### Note:

----
Feel free to make the data non-const, especially if you need to modify sent data.

----

### 2. Create an EventQueue Object
To create an EventQueue, you must first ``#include "EventQueue.h"``, and then define an object. The template-paramter for EventQueue is the type of data you are going to send.

#### Example:

```
EventQueue<CollisionEvent> collisionEventQueue;
```

### 3. Subscribing/Un-subscribing to events

Subscribing and un-subcribing to/from an event queue is the same as an event sender, you call ``add()`` to subscribe, and ``remove()`` to unsubscribe. As with an event sender, ``add()`` and ``remove()`` does have overloads so you can use it with member functions.

Example:

```
// our call-back
void handleCollision(const CollisionEvent&)
{
	std::cout << "Collision occured!\n";
}

EventQueue<CollisionEvent> collisionEventQueue;

// subscribe to events sent
collisionEventQueue.add(&handleCollision);

// un-subscribe the event
collisionEventQueue.remove(&handleCollision);

```

#### Note:
---

EventQueue actually uses an EventSender object to send out events. The only difference is, an ``EventQueue<T>`` uses the following function prototype:

```
void foo(const T&);
```
The name of your method may be anything, as with an event sender, but all functions MUST return void. Unlike an event sender, where it is optional (but doesn't really make sense).

---


### 4. Pushing Data to the Event Queue

To push data to the event queue, simply call the method ``push(const T&)``, where T is the type of your event.

#### Example:

```
EventQueue<CollisionEvent> collisionEventQueue;

// ...

// push some data to the event queue
collisionEventQueue.push(CollisionEvent(entity1, entity2));

```

### 5. Emitting your events

Finally, to emit your events you have pushed to the event queue, call ```emit()``` on your event queue object. Please note, that this will clear the data that you have pushed to the event queue.

#### Example:

```
collisionEventQueue.emit(); // emit my events :)
```