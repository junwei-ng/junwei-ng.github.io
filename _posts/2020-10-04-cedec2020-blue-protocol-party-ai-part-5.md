---
layout: single
title:  "[CEDEC 2020] Group Coordination AI Supporting the Party Battles of Blue Protocol - Inter-agent Communication"
date:   2020-10-4 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary, character-ai, coordination]
categories: [conference-reports]
---

Part 5 and the final part of this series describes one lesser-known technique *Blue Protocol's* team used for inter-agent communication: the tuple space.

Original title: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』 by 長谷 洋平（株式会社バンダイナムコスタジオ）
[Presentation slides on CEDEC Digital Library (signup required, free)](https://cedil.cesa.or.jp/cedil_sessions/view/2271)

---

### Table of Contents

This is a series going in-depth into the contents of the CEDEC 2020 session "Group Coordination AI Supporting the Party Battles of Blue Protocol" by Yohei Hase of Bandai Namco Studios.

0. [Introduction]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai %})
1. [Dynamically Determining Battle Participants]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai-part-2 %})
2. [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})
3. [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})
4. **Inter-agent Communication**

---

## Inter-agent Communication

A key part of any coordination/cooperation AI is inter-agent communication. This can involve specific requests (such as a character requesting for reinforcements) or just information transfers (such as the sharing of a character's Target List.)

Hase brings up two common issues that come up when handling communication:

* Multiple parameters are usually involved, and so specialized structs or procedures usually need to be written to handle communication.
* Communication code can easily become complex when many message types and many character types are involved. For example, it's common to have requests that can be only acted upon by certain characters (like requests for healing that can only be acted upon by characters that can heal), and code can start growing out of control as the number of combinations grow.

![Communication methods in *Blue Protocol*](https://i.imgur.com/9ac0zVD.png)

*Blue Protocol* uses standard communication methods like Blackboards and bidirectional messaging, but recently began to use **tuple spaces** as well.

### Tuple spaces

Tuple spaces were originally developed in the distributed computing world for inter-process communication. A tuple space is a data structure containing *tuples* of data that can be accessed concurrently. The [Wikipedia article on tuple spaces](http://en.wikipedia.org/wiki/Tuple_space) is a good place to start learning more about this data structure.

*Blue Protocol's* tuple space implementation supports these four basic operations, with pseudocode examples:

![Tuple space operations](https://i.imgur.com/M0BePx9.png)

* `write`: writes a tuple of data to the tuple space
* `read`: returns a tuple matching the given pattern
* `take`: returns a tuple matching the given pattern and deletes that tuple from the tuple space
* `notify`: registers an agent's interest in tuples matching a given pattern, so a notification is received when a matching tuple is added to the tuple space

Hase lists the following as benefits of tuple spaces:

* Tuple spaces are both a data store and a method of communication.
* Multiple parameters can easily be accommodated by tuples.
* Tuples matching certain patterns in the tuple space can easily be filtered and retrieved.
* Tuple spaces only have four operations and so are relatively simple to implement (apparently, it took *Blue Protocol's* team only a couple of days to get their implementation working.)

#### Healing Requests With Tuple Spaces

Using an example of a healing request, Hase walks us through how communication using tuple spaces works in *Blue Protocol*.

![Registering interest in HealReq notifications](https://i.imgur.com/DbOZHy3.png)

First, healing request tuples are defined by the tuple `HealReq(target, targetHpRatio)`. Healers register their interest in `HealReq` notifications by using the `*` wildcard to receive notifications for all `HealReq` postings.

![Healing request](https://i.imgur.com/G15h8lw.png)

When some allied character wants healing, they write a `HealReq` tuple with their identifier and their current HP ratio to the tuple space. Characters subscribed to matching patterns are then notified.

![Healing action](https://i.imgur.com/6Xs92KG.png)

Healing *action* requests are defined by the tuple `HealAct(performer, target, evaluation)`. The `HealAct` tuple contains the performer of the healing action, the target of the healing action, and an evaluation of how effective the healing action is.

![Healing Tactical Order](https://i.imgur.com/cgIjmAJ.png)

Healing actions are implemented as Tactical Orders. `HealAct` tuples are collected and planning is done in the Combat Coordinator's Brain layer, taking into account the evaluation of each healing action request and the overall strategic situation.

#### A More Complex Example - Area Buffs

A more complex example demonstrating communication using tuple spaces is how Area Buffs are coordinated. The specifications of Area Buffs are as follows:

* When an Area Buff is performed, all units in the area around the buff-giver is strengthened.
* Designers want to limit the number of units entering the area to receive the buff for better combat flow.
* We don't want characters to enter the buff area only to find out their attacks can't reach the player, so only characters with long-ranged attacks can intentionally enter the buff area.

![Setting up the conditions for area buffs](https://i.imgur.com/DGAWAJw.png)

Three types of tuples are involved in area buffs:

* `AreaBuffReq(buffGiver, buffReceiverConditions)` defining a buff request, specifying the buff-giver and conditions any buff-receivers need to meet.
* `AreaBuffAct(buffGiver, evaluation)` defining a buff action request, specifying the buff-giver and the effectiveness evaluation of that buff-giver performing the buff.
* `AreaBuffRcv(buffGiver, buffReceiver, evaluation)` defining a buff reception request, specifying the buff-giver, buff-receiver, and the effectiveness evaluation of that buff-receiver receiving the buff from that buff-giver.

The Combat Coordinator needs to be prepped for Area Buffs.

Area Buffs are implemented as Tactics, so the buff-giver provides the Area Buff Tactic to the Combat Coordinator. The buff-giver also registers for notifications for `AreaBuffReq(self, *)` so it receives a notification whenever an `AreaBuffReq` with itself as the specified buff-giver is posted to the tuple space.

The buff-receiver registers for notifications for `AreaBuffReq(*, selfCombatAbility)`, so it receives a notification whenever an `AreaBuffReq` with receiver conditions it qualifies for is posted to the tuple space.

![Executing the Area Buff Tactic](https://i.imgur.com/Wvq2RHc.png)

When the Area Buff Tactic is chosen, the Combat Coordinators actually writes an `AreaBuffReq()` to its own tuple space. In this case, `AreaBuffReq(specificBuffGiver, canPerformRangedAttack)`.

The buff-giver and buff-receivers both receive notifications when that `AreaBuffReq` is posted. In response, the buff-giver posts an `AreaBuffAct()` and the buff-receiver posts an `AreaBuffRcv()`. The Combat Coordinator's HTN Planner then takes those requests into account in the next tactical plan in a similar to healing action requests.

Summarizing how communication flows in *Blue Protocol's* tuple space implementation,

1. Characters subscribe to the notifications they are interested in.
2. Some tuple is posted to the tuple space.
3. Characters interested in that tuple are notified.
4. Characters process that tuple, and the resulting tuple is posted to the tuple space.
5. The Combat Coordinator creates a plan, taking into account the tuples in the tuple space.

---

This wraps up the CEDEC 2020 presentation Group Coordination AI Supporting the Party Battles of Blue Protocol by Yohei Hase of Bandai Namco Studios.

[Back to Table of Contents](#table-of-contents)
