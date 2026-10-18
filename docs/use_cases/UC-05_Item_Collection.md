# UC-05 Collect Item Drops

| Author | Estimate | User stories                                                                                                                                                                                                                                                               |
| ------ | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Kilian | 13 SP    | [US-IC-01](https://github.com/xniklasx-dev/MIGHTYCorgi/issues/32), [US-IC-02](https://github.com/xniklasx-dev/MIGHTYCorgi/issues/33), [US-IC-03](https://github.com/xniklasx-dev/MIGHTYCorgi/issues/34), [US-IC-04](https://github.com/xniklasx-dev/MIGHTYCorgi/issues/35) |

## 1 Brief description

In this use case, the player collects items dropped by defeated enemies or awarded by room events during a room-based roguelike run. The game evaluates the relevant drop chances, creates an item with a defined type and rarity, and presents it as a pickup. When the Corgi collects the item, the system determines whether it grants an immediate temporary treat effect or belongs in the collected-item inventory. Players can inspect the obtained item's name, rarity and description; outside a run, unwanted collected items can be converted into experience. Equipping, swapping and unequipping items belong to a separate use case and are explicitly out of scope.

## 2 Actors

- **Primary:** Player
- **Secondary:** None. Enemy defeat events, drop rules and inventory state are internal game subsystems, not external UML actors.

## 3 Mockup

![UC-05 Item Drop and Collection Wireframe](../mockups/UC-05_Item_Collection.png)

Wireframe shows an item on the ground, collection feedback and the obtained-item details.

## 4 Flow of events

### 4.1 Basic flow

1. During a run, the player defeats an enemy or completes an event that can award an item.
2. The system evaluates the associated item-drop rules and, if the roll succeeds, generates a collectible item with its type and rarity.
3. The system displays the dropped item in the game world with a visual indication of its type or rarity.
4. The player moves the Corgi into pickup range and collects the item.
5. The system validates that the pickup is available and determines the item's defined behavior.
6. For a collectible inventory item, the system adds the item to the player's collected items and marks the world pickup as collected.
7. The system shows collection feedback, including the item's name and rarity, and updates the inventory state.

### 4.2 Alternative flows

**2a · No item drops.** At step 2, the drop roll does not succeed. The system creates no pickup. The use case ends without an inventory change.

**2b · Event-specific drop.** At step 2, a room or special event has a defined reward. The system uses that event's applicable item rules and continues at step 3.

**4a · Pickup not collected.** At step 4, the player leaves the item on the ground. The item remains subject to the run's pickup-lifetime rules. The use case ends without adding the item to the inventory. _Despawn timing has not yet been specified._

**5a · Temporary treat.** At step 5, the pickup is a temporary treat instead of a persistent inventory item. The system immediately applies its specified effect (for example, regeneration or a temporary attack-speed boost), removes the world pickup, displays feedback and ends the use case. It does not add permanent equipment to the inventory.

**5b · Already collected.** At step 5, the pickup has already been collected or is no longer available. The system does not grant the item a second time and ends the use case.

**6a · Inventory cannot accept the item.** At step 6, the inventory cannot receive the drop under the game's chosen capacity rules. The system informs the player and does not remove the world item until the rule has been resolved. _Whether the inventory has a capacity limit is not yet specified._

**7a · Inspect item.** After step 7, the player opens the inventory to view the obtained item's name, rarity, type and description. The system displays these properties without changing equipment.

**7b · Convert unwanted item to XP.** Outside an active run, the player selects an eligible unequipped item in the inventory and requests conversion to experience points. The system displays the proposed XP gain and asks for confirmation; upon confirmation it removes exactly one item unit and grants XP once. On cancellation, neither the item nor the XP balance changes. _Conversion amounts and restrictions are not yet defined._

### 4.3 Sequence diagram

![UC-05 Sequence Diagram](../diagrams/UC-05_sequence.png)

## 5 Special requirements

- **NFR-IS-01 — Drop consistency:** For 100% of enemy defeat events in an automated test set, no more than one drop decision shall be processed for the same defined drop opportunity.
- **NFR-IS-02 — Pickup consistency:** In 100 consecutive pickup attempts on the same item instance, the inventory item count shall increase by at most one.
- **NFR-IS-03 — Collection feedback:** At least 95% of successful inventory-item pickups shall display the collected item's name and rarity within 250 ms under a defined local performance test. _(Proposed target, not a measured result.)_
- **NFR-IS-04 — Conversion safety:** 100% of item-to-XP conversions shall require confirmation; cancellation shall leave both item count and XP unchanged.

These are use-case-specific requirements and do not replace the SRS quality requirements in sections 3.2–3.5.

## 6 Preconditions

- A valid run is active for the item-drop-and-pickup main flow.
- The Corgi is alive and can interact with a dropped item.
- Drop rules and item definitions are available to the game system.
- For the conversion alternative, the run has ended and the player has an eligible collected item.

## 7 Postconditions

- **Successful inventory pickup:** Exactly one collected item is added and its world pickup is no longer available; its type and rarity are retained.
- **Temporary treat:** The specified temporary effect is applied and the world pickup disappears without creating permanent equipment.
- **No drop / missed pickup / invalid pickup:** No new item is added to the inventory.
- **Confirmed conversion (optional alternative):** One eligible item is removed and the defined XP gain is applied once.
- Equipping and equipment slot changes do not occur in this use case.
