---
layout: single
title:  "[CEDEC 2020] Group Coordination AI Supporting the Party Battles of Blue Protocol - Tactical Decision-Making"
date:   2020-10-3 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary, character-ai, coordination]
categories: [conference-reports]
---

Part 4 of this series describes how the Combat Coordinator comes up with tactical plans, and how the plans are executed by individual Character AIs.

Original title: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』 by 長谷 洋平（株式会社バンダイナムコスタジオ）
[Presentation slides on CEDEC Digital Library (signup required, free)](https://cedil.cesa.or.jp/cedil_sessions/view/2271)

---

### Table of Contents

This is a series going in-depth into the contents of the CEDEC 2020 session "Group Coordination AI Supporting the Party Battles of Blue Protocol" by Yohei Hase of Bandai Namco Studios.

0. [Introduction]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai %})
1. [Dynamically Determining Battle Participants]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai-part-2 %})
2. [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})
3. **Tactical Decision-Making**
4. [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

---

## Tactical Decision-Making

So far, we learned about how AI Groups are formed and how each individual character's behaviors are determined. Roles like Defenders and Healers allow for specialized, party-like behaviors and AI Groups allow individuals to cooperate.

However, this still leaves three problems.

1. Multiple characters will perform the same Role actions, leading to redundancy. Because characters are controlled by their Roles, if there are multiple Defenders in the same group but only 1 Healer, all the Defenders will swarm around the lone Healer to guard it. Similarly, if there are multiple Healers and one character on low health, all the Healers will simultaneously try to heal that character.
2. Every character is still acting on their own, so the group's overall actions lack coherence and goals.
3. Each character is strongly defined by their Role, and that results in silly or boring behaviors. For example, Healers prefer to perform healing actions over other things, so if a group only has Healers they'll keep healing each other and not attack.

This is where the Combat Coordinator comes in.

![Combat Coordinator](https://i.imgur.com/zuiNyZR.png)

The Combat Coordinator uses information from each character and the environment, and gives out orders to the AI characters it manages. Coordination is routed through the Combat Coordinator and not carried out between agents.

![Combat Coordinator AI Architecture](https://i.imgur.com/JU8TtqU.png)

The Combat Coordinator has the same 3-layer AI architecture as Character AI. However, unlike a character, the Combat Coordinator does not physically exist and so doesn't act on the game world. Instead, it modifies the behavior of characters by sending them orders.

![*Blue Protocol* Influence Map](https://i.imgur.com/VcLdIMn.png)

* The Perception layer collects data from the environment, mainly by environmental queries on an Influence Map. *Blue Protocol's* influence maps are generated from the navigation mesh.

* The Brain layer decides what each character managed by the Combat Coordinator should do. Details in a bit.

* The Action layer sends out orders to each character to modify their behavior.

### The Combat Coordinator's Brain

![*Blue Protocol* Combat Coordinator Brain layer architecture](https://i.imgur.com/Ebr0hwn.png)

There are 3 sub-layers in the Combat Coordinator's Brain, similar to the Character AI.

* The Utility System takes data from the Perception layer and evaluates possible high-level strategies. 
* The HTN Planning layer creates a tactical plan to achieve the chosen strategy, assigning a Tactic to each character.
* The Behavior Tree layer executes the tactical plan by giving orders to the relevant characters.

#### Utility System

![Hold Position or Surround?](https://i.imgur.com/G1iYPxN.png)

In this example, the Utility System is used to decide between the "Hold Position" and "Surround" strategies. The evaluation criteria are the distance between the player cluster and AI party, and how much of the player cluster the enemy party overlaps.

![Hold Position and Surround scores](https://i.imgur.com/61gBX8e.png)

If the player cluster is far away and there's not much overlap, the Hold Position strategy will be scored higher and AI characters will try to only lob ranged attacks.

When the player cluster gets closer and the AI party overlaps with the player cluster, the Surround strategy will then be scored higher and the characters in the AI party will actively try to flank and surround the player cluster.

After a strategy is selected, some processing is done before it is passed to the HTN planner.

* Parameters that are required for planning are set. This includes things like the number of attack tokens, which can vary from strategy to strategy.
* Roles are assigned to each member.

![Role assignment](https://i.imgur.com/K4kvsNo.png)

The Combat Coordinator has a priority and minimum count for each role, while each character has preferences for the roles they want to be in ranging from 0 (impossible to perform that role) to 1 (ideal for that role.)

The algorithm for assigning roles simply greedily iterates through the roles the Combat Coordinator wants filled from highest to lowest priority, and assigns the character with the highest preference rating for that role until all slots are filled. If no characters can fill a role, that role is skipped.

In the example given, the Combat Coordinator wants to fill 2 Attacker slots with the highest priority of 1.0, 1 Defender slot at a priority of 0.8, and the rest are unrestricted. The available characters in the Combat Coordinator and their role preferences are as follows:

| Character | Attacker Pref. | Defender Pref. | Healer Pref. |
|:----------|:---------------|:---------|:-------|
| Alice     | 0.1            | 0.0      | 1.0    |
| Bob       | 0.5            | 1.0      | 0.0    |
| Charlie   | 0.1            | 0.0      | 1.0    |

1. The greedy algorithm first tries to fill in the 2 Attacker slots. Bob has the highest Attacker preference at 0.5 and is chosen, followed by Alice who is first in the character list.
2. Next, it tries to fill the 1 Defender slot, but the only remaining member, Charlie, is unable to be a Defender. Thus, the Defender slot is skipped.
3. Finally, the unrestricted slots are filled, and Charlie is assigned his preferred Healer role.

#### HTN Planner

When preprocessing is complete, the HTN Planning layer assigns Tactics to the various characters managed by the Combat Coordinator. Planning is complete when every character has an assigned Tactic.

![HTN for tactical planning](https://i.imgur.com/ot1IyIi.png)

Possible Tactics are broken down as tasks in the planner. These Tactics can either be performed by an individual or by multiple characters. In this case, there are three Tactics available: Attack, Defend, and Area Buff. Attack and Defend are actions performed by an individual, while Area Buff is an action performed by multiple characters.

![Breakdown of the planning of the Attack Tactic](https://i.imgur.com/YJQPBHi.png)

In this example, the Attack Tactic comprises of two simple tasks: Finding an Attack Order, and Assigning the Attack Order. An Attack Order task takes in two parameters: Attacker and Target, and has the following properties:

* Preconditions: Number of available attack tokens > 0, and Attacker's `isAssigned` flag is `false`
* Effects: Decrement number of available attack tokens by 1, and Set Attacker's `isAssigned` flag to `true`
* Preferences: Target evaluation score from Attacker's Character AI's Target List

![Tactics in the Combat Coordinator's HTN Planner are Tactical Skills](https://i.imgur.com/01iSlzY.png)

Tactics in the Combat Coordinator's HTN Planner are implemented as Tactical Skills, and so can dynamically be added to or removed from the HTN domain.

Some basic Tactics all characters can perform, like Attack or Idle, are available in the Combat Coordinator by default. Special Tactics only some characters can perform, like Defend or Area Buff, are added to the Combat Coordinator's planner when those characters join the Combat Coordinator.

This means that the planner does not need to plan over Tactics that nobody in group can perform.

#### Tactical Orders

After Tactics are assigned to all characters, by the HTN Planner layer, the Behavior Tree layer executes the assigned Tactic. This is done by giving *Tactical Orders* to characters.

Tactical Orders are a *list of Tactical Skills* and their necessary parameters. When a character receives a Tactical Order, they *add* the Tactical Skills to their HTN Planner and the parameters to their Blackboard. The Character AI then does planning as usual -- allowing Tactical Orders to seamlessly modify the behavior of the character.

![Before receiving a Tactical Order](https://i.imgur.com/AIop8qO.png)

The picture above shows an example of a HTN subdomain prior to an Attack Tactical Order.

The subdomain on the left is the Combat subdomain of a Character AI, comprising two empty methods: OrderedAction and Wait.

The subdomain on the right is the Attack Tactical Order subdomain, comprising of an Attack complex task with three methods: Melee Attack A, Melee Attack B, and Ranged Attack A. 

Since the Character AI's Combat subdomain does not contain any simple tasks, plans do not contain any combat actions.

![After receiving a Tactical Order](https://i.imgur.com/ItEjgY6.png)

After receiving the Attack Tactical Order, the OrderedAction task is linked to the Attack Tactical Order subdomain. This makes the attack methods of Melee Attack A, Melee Attack B, and Ranged Attack A available to the HTN Planner.

Implementing Tactical Orders with Tactical Skills and using the HTN Planner to actually effect Tactical Orders gives three benefits:

1. Tactical Orders can simply be ignored when they are completely inappropriate, without having to take into account all sorts of minor details. The Utility System layer determines the high-level goal before HTN Planning takes place. As such, if the character is incapacitated, the Utility System will cause the HTN Planner to work on a "Idle" plan. Any Tactical Orders linked to the "Combat" subdomain will simply not be acted on, eliminating the need to do special `if (!isIncapacitated)` checks.
2. Tactical Skills contain a HTN subdomain as well as executable actions for the Behavior Tree layer. This means that Tactical Orders to perform special actions can be created as easily as other Tactical Skills.
3. Tactical Skills can contain preferences, which means Tactical Orders can influence the final plan too. A Tactical Order can contain a preference for melee attacks, and turn the AI character into a melee attacker without having to modify the rest of its HTN domain.

![Example of a Melee Attack Tactical Order](https://i.imgur.com/6MJimLl.png)

The picture above shows a monster with an active Melee Attack Tactical Order implemented with preferences. The planning log shows that plans containing ranged attacks (`P_Shoot_ThrowStone`) are rejected during planning.

In the next part, we'll see how *Blue Protocol's* AI team supports a lesser-known method for communication between agents.

Next part: [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

[Back to Table of Contents](#table-of-contents)
