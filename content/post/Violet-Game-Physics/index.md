---
title: Game Physics
description: Behind the Scenes of Game Physics, The Power of Rigidbodies and Colliders.
slug: Phsyics
date: 2025-08-19 00:00:00+0000
image: cover.jpg
categories:
    - dot8
tags:
    - development
weight: 1     
---

## Inside the dot8 Physics Engine ##
Building a physics engine isn’t just about moving objects—it’s about creating a consistent and believable world. Rigidbodies define how objects react to forces, gravity, and collisions, while colliders determine the shape and boundaries for those interactions. Together, they form the backbone of any physics simulation, enabling everything from subtle character movement to complex dynamic environments. Understanding how these components work together is key for developers who want precision, performance, and realism in their games. Mastering these fundamentals isn’t just a technical skill—it’s what transforms code into immersive experiences.

### Rigidbody: Motion in Action ###

A Rigidbody defines how an entity moves in the game world:

- Velocity - 2D vector for movement per frame.
- Acceleration - Enables force-based motion.
- Gravity - Global or per object (set to 0 for flying entities).
- Damping/Friction - Gradually reduces speed for smooth stops.
- Integration Step - Updates position each frame with a fixed timestep.

```
struct	Rigidbody
ID			            db	0
ObjectID		        db	0
Flags			        db	0
VelocityX		        dw	0
VelocityY		        dw	0
AccelerationX		    dw	0
AccelerationY		    dw	0
FrictionX		        dw	0
FrictionY		        dw	0
ConstantAcceleration	dw	0
ForceDuration	    	db	0
State			        db	0
FSM			            dw	0
	ends
```

Key Insight: A Rigidbody doesn’t detect collisions — it only updates position based on forces.

---

### Collider: The Silent Watcher ###

-A Collider focuses on where interactions happen. In retro engines, this is usually an AABB (Axis-Aligned Bounding Box):

- Offset - Position offset from the entity’s origin to align with the sprite.
- Size - Defines collision area.
- Trigger Flag - Marks collider as “non-physical” (no bounce/push, only events).
- Layer/Mask - Filters what this collider interacts with (e.g., Player layer vs. Enemy layer).
- Response - None (for triggers) or simple reaction (bounce, stop).

```
 struct Collider
ID			db	0
ObjectID		db	0
Flags			db	0
OffsetX			db	0
OffsetY			db	0
Width			db	0
Height			db	0
Layer			db	0
EventCollision	        dw	0
EventTrigger	        dw	0
	ends
```
Key Insight: A Collider never moves objects — it only reports contacts.

---

<video src="colliders.mp4" width="480" height="360" controls></video>

See Colliders in action

---

### Why Keep Them Separate? ###
Splitting Rigidbody and Collider gives dot8:

- Modularity - An object can have physics without collision, or vice versa.
- Performance - Collider checks can be optimized separately.
- Flexibility - Triggers and sensors can exist without affecting motion.

---

Rigidbodies and Colliders work together to bring physics to life. One controls movement, the other defines interaction. Simple, but powerful. How will you use them in your game?