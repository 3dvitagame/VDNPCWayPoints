# VD NPC WayPoints — Advanced Waypoint Navigation, Bottleneck Traffic & Interaction System for UE 5.5+

![Unreal Engine](https://img.shields.io/badge/Unreal%20Engine-5.5%20%7C%205.7%20%7C%205.8-blue?logo=unrealengine)
![Platform](https://img.shields.io/badge/Platform-Win64%20%7C%20Mac%20%7C%20Linux%20%7C%20Android%20%7C%20iOS-brightgreen)
![Language](https://img.shields.io/badge/Language-C%2B%2B%20%7C%20Blueprints-orange)

**VD NPC WayPoints** is an enterprise-grade, modular, high-performance C++ plugin for Unreal Engine 5. It provides a complete simulation framework for NPC behaviors in open and indoor environments: cyclic and branching patrols, crowd bottleneck queue arbitration, interactive door handling, prop pickup/transport with Motion Warping alignment, dynamic squad tactical formations, player escort mechanics, contextual dialogues, and physical collision bump reactions.

---

## 📑 Table of Contents
1. [Core Features](#-core-features)
2. [Architecture Overview](#-architecture-overview)
3. [Quick Start Guide](#-quick-start-guide)
4. [Interactive Objects & Doors (INPCInteractableInterface)](#-interactive-objects--doors-inpcinteractableinterface)
5. [Traffic Queues & Bottlenecks](#-traffic-queues--bottlenecks)
6. [Prop Carrying & Motion Warping](#-prop-carrying--motion-warping)
7. [Smart Actions (Sit, Lean, Workstation)](#-smart-actions-sit-lean-workstation)
8. [Squad Tactical Formations & Escort Quests](#-squad-tactical-formations--escort-quests)
9. [Dynamic Head Look-At](#-dynamic-head-look-at)
10. [Collision Bump & Flinch Reactions](#-collision-bump--flinch-reactions)
11. [Waypoint Actor Property Reference](#-waypoint-actor-property-reference)
12. [Delegates & Event Dispatchers](#-delegates--event-dispatchers)

---

## 🚀 Core Features

* 🔹 **Universal World Interactions:** Open doors, blast gates, drawbridges, and elevator platforms via an abstract C++ interface.
* 🔹 **Bottleneck Traffic Arbitration:** Resolves narrow passage deadlocks (doors, single-file bridges) using an exclusive zone reservation token system.
* 🔹 **Dynamic Obstacle Avoidance:** Proactively navigates around standing players and idling NPCs without breaking waypoint sequence.
* 🔹 **Item Pickup & Carrying:** Supports Motion Warping alignment steps, socket attachment, carry animations, and linked locomotion layers.
* 🔹 **Squad Formations:** 5 formation archetypes (Wedge, Column, Line, Box, Companion) with dynamic corridor compression and automatic leader succession upon death.
* 🔹 **Escort Quest Framework:** Speed matching, tether range validation, waiting behaviors, and catch-up callouts.
* 🔹 **Procedural Bump Reactions:** Physics-based tilt impulses, directional flinch montages, and complaint voice lines upon high-speed player collisions.
* 🔹 **Designer-Friendly In-Editor Workflow:** Fast 1-click chaining (`Spawn Next Waypoint`), auto-snapping to NavMesh/floor, color-coded velocity vectors.

---

## 🛠 Architecture Overview

The system is built upon three foundational components:
1. `ANPCWaypointActor` — In-world waypoint anchor holding sequence links, wait delays, montages, dialogue lines, and interaction descriptors.
2. `UNPCPathFollowerComponent` — Actor component attached to `ACharacter` executing pathfinding requests, state machines, queues, and audio.
3. `UNPCInteractableInterface` — Abstract interface for any interactive obstacle (doors, laser barriers, trapdoors, levers).

---

## ⚡ Quick Start Guide

### Step 1. NPC Character Setup
1. Open your NPC Character Blueprint.
2. Add the **`NPCPathFollowerComponent`**.
3. Ensure the character has a valid **`AIController`** assigned in its Pawn properties.

### Step 2. Build the Waypoint Route
1. Drag **`NPCWaypointActor`** from the Content Browser into your level (it will automatically project to the floor).
2. In the *Details* panel, click **`Spawn Next Waypoint (3m)`** to spawn and link the next node forward.
3. Form your patrol loop. On the final waypoint, assign the first node to **`Next Waypoint`**.

### Step 3. Launch Patrol
1. Select your NPC in the level and set **`Initial Waypoint`** on its `NPCPathFollowerComponent`.
2. Ensure **`Auto Start On Begin Play`** is set to `True`.
3. Press **Play (PIE)** — your NPC will immediately begin following the path.

---

## 🚪 Interactive Objects & Doors (INPCInteractableInterface)

Door navigation is fully decoupled from door geometry and kinematic implementation.

### Setting up an Interactive Door Actor:
1. Open your Door Blueprint (e.g., `BP_SecurityDoor`).
2. Go to **Class Settings** and add **`NPCInteractableInterface`**.
3. Implement the interface events:
   * **`IsDoorClosed`** $\rightarrow$ Returns `true` if the barrier blocks passage.
   * **`OpenDoor`** $\rightarrow$ Executes the opening sequence (Timeline rotation, collision toggle, audio).
   * **`CloseDoor`** $\rightarrow$ Executes the closing sequence.
4. Ensure your door static mesh component affects runtime dynamic navigation:
   * **Project Settings $\rightarrow$ Navigation Mesh $\rightarrow$ Dynamic $\rightarrow$ Force Rebuild on Runtime = True**
   * Or set `Dynamic Obstacle = True` on the door component.

### Waypoint Door Settings:
* **`Door Actor`** = Reference to your in-level door actor.
* **`Door Open Montage`** = Character interaction montage (handle twist, keycard swipe).
* **`Door Open Wait Delay`** = Delay in seconds to allow the door to finish opening before walking through.
* **`Auto Close Door After Pass`** = `True` if the NPC should close the door behind itself once reaching the next node.

---

## 🚦 Traffic Queues & Bottlenecks

Prevents crowding and physical deadlocks at narrow gateways, ladders, and walkways.

* Enable **`bIsBottleneck = True`** on the gateway waypoint.
* Set **`Bottleneck Radius`** (e.g., `150.0`).
* **How it works:** The first NPC to approach claims reservation of the bottleneck node. Subsequent NPCs will stop outside the reservation radius and wait in queue until the occupant departs.

---

## 📦 Prop Carrying & Motion Warping

Enables NPCs to approach items, align via Motion Warping, execute pickup montages, attach props to skeleton sockets, and patrol with dedicated carry poses.

1. **Pickup Waypoint:**
   * **`Prop Actor To Pickup`** = Target actor in the level.
   * **`Prop Attach Socket`** = Target skeleton socket name (e.g., `hand_r`).
   * **`Arrival Montage`** = Pickup animation montage.
   * **`Pickup Delay`** = Time offset from montage start to socket attachment.
   * **`Carry Loop Montage`** or **`Linked Carry Anim Layer`** = Locomotion override while holding the item.
2. **Drop Waypoint:**
   * **`bDropHeldPropOnArrival = True`**.
   * **`Drop Placement Mode`**:
     * `At Waypoint` — Exact transform relative to waypoint.
     * `Snap To Floor` — Projects vertically onto underlying geometry.
     * `Drop With Physics` — Releases item from hands with active physics simulation.

---

## 🪑 Smart Actions (Sit, Lean, Workstation)

Enables ambient world interactions such as sitting on chairs, leaning on railings, or crafting at workbenches.

* **`Action Type`** = `Sit` / `Lean` / `Workstation` / `CustomAction`.
* **`Action Target Actor`** = Target chair, bench, or environment prop.
* **`Target Socket Or Component Name`** = Socket or component name to align to.
* **`Enter Montage`** $\rightarrow$ **`Loop Montage`** $\rightarrow$ **`Exit Montage`**.
* **`bDisableCollisionDuringAction`** = Temporarily disables pawn capsule collision to prevent physics clipping.

---

## 👥 Squad Tactical Formations & Escort Quests

### Squad Patrol Mode:
1. On the designated squad leader, set **`bIsSquadLeader = True`**.
2. Populate the **`Squad Followers`** array with follower NPC characters.
3. Select **`Squad Formation Type`** (`Wedge`, `Column`, `Line`, `Box`).
4. **Dynamic Formation Adaptation:** When traversing narrow corridors, the squad automatically compresses into a single-file column.
5. If the leader dies, leadership is automatically passed to the next surviving squad member.

### Escort Quest Mode:
1. Set **`bIsEscortMode = True`**.
2. **`Escort Target`** = Player character reference.
3. **`Max Escort Distance`** = Distance limit before the NPC pauses, faces the player, and plays a dialogue cue from `Escort Wait Dialogues`.

---

## 👀 Dynamic Head Look-At

Allows NPCs to glance toward nearby players, conversation partners, or points of interest without rotating their entire body.

1. In `NPCPathFollowerComponent`, set **`bEnableHeadLookAt = True`**.
2. In your Animation Blueprint's **AnimGraph**, insert a **Look At** or **Transform (Modify) Bone** node targeting the `head` bone.
3. Feed in the component output values:
   * `GetHeadLookAtRotation()` — Clamped local rotational offset.
   * `GetHeadLookAtAlpha()` — Interpolated blending alpha (0.0 to 1.0).

---

## 💥 Collision Bump & Flinch Reactions

Triggers realistic procedural and animated responses when an NPC is impacted by a sprinting player:
1. Checks impact velocity threshold (`Min Bump Speed Threshold`).
2. Applies a directional procedural spring tilt to the skeletal mesh (`bEnableProceduralShake`).
3. Plays a directional flinch montage (`Bump Montage Front` / `Bump Montage Back`).
4. Spawns contextual voice line audio and subtitles from `Bump Dialogues`.

---

## 📋 Waypoint Actor Property Reference

| Category | Property | Type | Description |
| :--- | :--- | :--- | :--- |
| **Waypoint** | `Next Waypoint` | `ANPCWaypointActor*` | Next sequential patrol node |
| **Branches** | `Branches` | `TArray<FWaypointBranch>` | Weighted branch candidates for random routing |
| **Traffic** | `bIsBottleneck` | `bool` | Enables exclusive queue reservation |
| **Traffic** | `Bottleneck Radius` | `float` | Reservation zone radius in cm |
| **Doors** | `Door Actor` | `AActor*` | Target door/gate implementing `INPCInteractableInterface` |
| **Doors** | `Door Open Montage`| `UAnimMontage*` | Door interaction montage |
| **Movement** | `Movement Speed` | `float` | Locomotion speed toward this waypoint (cm/s) |
| **Movement** | `Wait Duration` | `float` | Idle duration upon arrival (seconds) |
| **Movement** | `bWaitUntilTriggered` | `bool` | Wait indefinitely until `TriggerContinue()` is called |
| **LookAt** | `bLookAroundOnArrival` | `bool` | Guard surveillance glance sweep (left/right) |
| **Props** | `Prop Actor To Pickup` | `AActor*` | Target world item to pick up |
| **Props** | `bDropHeldPropOnArrival`| `bool` | Detach and place currently held prop |
| **Dialogue** | `Dialogue Trigger` | `Enum` | Audio/subtitle phase trigger (WalkStart / Arrival) |

---

## 📢 Delegates & Event Dispatchers

Bindable via Blueprints or C++:

* `OnWaypointReached(ANPCWaypointActor* Waypoint, AActor* NPC)` — Fires on arrival.
* `OnWaypointDeparted(ANPCWaypointActor* Waypoint, AActor* NPC)` — Fires on departure.
* `OnSubtitleTriggered(FText SubtitleText, float Duration)` — Dispatches UI subtitles.
* `OnPropPickedUp(AActor* PropActor, AActor* NPC)` — Broadcasts item pickup.
* `OnPropPutDown(AActor* PropActor, AActor* NPC)` — Broadcasts item drop.
* `OnDoorInteracted(AActor* DoorActor, AActor* NPC)` — Broadcasts door trigger.
* `OnNPCBumped(AActor* NPC, AActor* BumperActor, FVector HitNormal)` — Broadcasts collision flinch.
* `OnSquadLeaderChanged(AActor* NewLeader, AActor* OldLeader)` — Fires on leader succession.
