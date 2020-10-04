---
layout: single
title:  "[CEDEC 2020] Group Coordination AI Supporting the Party Battles of Blue Protocol - Character AI"
date:   2020-10-2 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary, character-ai, coordination]
categories: [conference-reports]
---

Part 3 of this series describes the bits of the Character AI system relevant to the party battles of *Blue Protocol*.

Original title: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』 by 長谷 洋平（株式会社バンダイナムコスタジオ）
[Presentation slides on CEDEC Digital Library (signup required, free)](https://cedil.cesa.or.jp/cedil_sessions/view/2271)

---

### Table of Contents

This is a series going in-depth into the contents of the CEDEC 2020 session "Group Coordination AI Supporting the Party Battles of Blue Protocol" by Yohei Hase of Bandai Namco Studios.

1. [Introduction]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai %})
2. [Dynamically Determining Battle Participants]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai-part-2 %})
3. **Character AI Behavior**
4. [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})
5. [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

---

## Character AI Behavior

The next section of the presentation describes how each individual AI determines and controls their actions. This is a summarized version of Hase's presentation at CEDEC 2019, [The Decision-Making Systems Behind The Distinctly Individual Characters Of *Blue Protocol*](https://cedil.cesa.or.jp/cedil_sessions/view/2102), focused on the parts that are relevant to party battles.

### Agent Architecture

![Diagram of the AI architecture of *Blue Protocol's* Character AI](https://i.imgur.com/WHc8yW9.png)
*Blue Protocol's* Character AI uses a 3-layer architecture similar to the Sense-Think-Act loop in robotics.

* The Perception layer collects data from the environment using sensors and queries.
* The Brain layer uses the processed data from the Perception layer to decide what to do next.
* The Action layer manages animations and performs the necessary calculations to perform what the Brain layer has decided.

### The Perception Layer

The Perception layer uses sensors and queries to collect data from the environment. One interesting thing is how queries in *Blue Protocol* are defined using a unique Perception Tree structure. 

![Perception Tree slide from CEDEC 2017](https://i.imgur.com/ln7n0M6.png)

Perception Trees reuse the Behavior Tree control architecture to perform environmental queries. They are described in Hase's [CEDEC 2017 presentation](https://cedil.cesa.or.jp/cedil_sessions/view/1657) (one slide shown above; the big PDF icon contains the slides, while the text link to a .zip file contains the demo videos.) 

### The Brain Layer, In-Depth

![Diagram showing the architecture of the Brain layer in *Blue Protocol*](https://i.imgur.com/7stlomO.png)

The Brain layer itself is made up of three layers of different decision-making architectures, utilizing the strengths of each.

* Processing the data from the Perception layer is the Utility System, which decides what to do. These are general high-level goals like "Fight", "Patrol", or "Flee".
* The chosen goal is passed to the HTN[^1] Planning layer, which comes up with a plan to achieve it.
* Finally, the individual tasks in the plan created by the HTN Planning layer are implemented with a Behavior Tree that concretely defines how the task is performed.

Hase goes into more detail on the HTN Planning layer.

### Preference-based HTN Planning in *Blue Protocol*

HTNs are a planning architecture, which means it takes it ultimately comes up with a series of concrete steps (a "plan") to attain a specified objective. All the tasks an AI agent can do are defined in the agent's *HTN domain*, and the variables relevant to planner are stored in a *world state*.

The core idea of HTN planning is repeatedly decomposing complex tasks into simple tasks that have a concrete implementation. This decomposition process takes into account the *preconditions* necessary to execute each simple task and also how the world state change due to the *effects* of each simple task.

By explicitly reasoning on the preconditions and effects of each simple task, the HTN planner can think several steps ahead and plan through a series of *intermediate states* prior to attaining the *goal state*.

For more details on HTNs, Troy Humphreys' article [*Exploring HTN Planners through Example* in Game AI Pro](http://www.gameaipro.com/GameAIPro/GameAIPro_Chapter12_Exploring_HTN_Planners_through_Example.pdf) is a great article for learning about the basics of HTN planning.

*Blue Protocol* implements a *preference-based* HTN planner. In a typical HTN planner, given a particular world state, objective, and HTN domain, the planner will always come up with the same plan. Preference-based planning lets a designer to additionally define *preferences* separately from the HTN domain, so the final plan can differ solely due to those preferences even with all other factors being equal.

Supporting preference-based planning thus makes HTN domains (which are relatively complex to design and test compared to preferences) reusable while allowing designers to easily create variations in behavior.

From what I can find out -- and please correct me if I'm wrong -- preference-based HTN planning was first introduced by Sohrabi, Baier, and McIlraith from the University of Toronto in their 2009 paper [HTN Planning with Preferences](https://web.archive.org/web/20201004044013/http://www.cs.toronto.edu/~sheila/publications/soh-bai-mci-ijcai09.pdf). This paper wasn't explicitly mentioned by Hase, so the preference-based HTN planner used in *Blue Protocol* could be original or inspired by other preference-based planner literature.

A *metric* is computed based on the number of preferences accommodated by the plan (in the UToronto paper the metric is based on the number of *violated* preferences.) This metric is used to evaluate how good a plausible plan is, and the plan with the highest score is chosen.

In *Blue Protocol*, preferences can be hard or soft. Hard preferences (or *constraints*) must be accommodated by the final plan, while soft preferences (or simply *preferences*) are optional but good to have.

Three types of preferences are supported in *Blue Protocol's* AI.

![Precondition preference](https://i.imgur.com/4CShpkA.png)

*Precondition preferences* are defined on each task and describe the preferred conditions of the world state when the task is chosen.

For example, take a "Prepare Breakfast" complex task that has two methods "Cook Rice" and "Toast Bread" and a world state containing the cups of rice and slices of bread available.

Precondition preferences can be applied on "Cook Rice" and "Toast Bread" such that "Cook Rice" is preferred when more cups of rice are available than slices of bread, and "Toast Bread" is preferred when there are more slices of bread available than cups of rice.

![Goal preference](https://i.imgur.com/0kkWSRc.png)

*Goal preferences* (or *simple preferences* in the UToronto paper) are defined on the objective and describe the preferred conditions of the world state when the chosen plan is fully-executed.

Continuing from the breakfast prep example, cooking rice and toasting bread results in dirty dishes that have to be washed. A goal preference for "Fewer Dishes To Wash" can be defined when planning to prepare breakfast.

Since the "Cook Rice" method creates more dishes than "Toast Bread", with the "Fewer Dishes To Wash" goal preference, "Toast Bread" is more likely to be chosen.

![Trajectory preference](https://i.imgur.com/EU7hC52.png)

*Trajectory preferences* (or *temporally extended preferences* in the UToronto paper) define preferences on the *order* in which tasks are performed.

For example, if the tasks "Change Clothes" and "Eat Breakfast" need to be performed before "Leaving the House", a trajectory preference *operator* `SometimeAfter` can be used to bias the order of the two tasks such that "Change Clothes" preferably takes place after "Eat Breakfast" and vice versa.

*Blue Protocol's* HTN planner supports operators like `Always`, `AlwaysBefore`, `AlwaysAfter`, `Sometime`, `SometimeBefore`, and `SometimeAfter`.

### Tactical Skills

*Blue Protocol* does not have HTN domains for each character type. Instead, the domain is broken down into subdomain templates generally defining how a major action can be performed.

A HTN domain can then be composed of various subdomains. Data-wise, a subdomain (with the necessary parameters) and preferences together form a *Tactical Skill* which defines an action a character can perform. The entire set of Tactical Skills attached to a character define all actions it can do. 

![Example of a Tactical Skill](https://i.imgur.com/HNbAEqs.png)

The picture above shows a `StabAttack` Tactical Skill which is a possible action under the `HTN_Attack_Melee` subdomain template. Various parameters like `Range To Ready For Attack` can be set per Tactical Skill and preferences can be defined via Blueprint. 

![Different behaviors with different Tactical Skills](https://i.imgur.com/sFYo9lz.png)

One major benefit of HTN planning is the relative ease of creating behavior variants, especially dynamically. Planning is done on the HTN domain, which is simply the set of Tactical Skills a character has. As such, creating a `GoblinWarlord` that can `ShieldGuard` be done simply by adding the `ShieldGuard` Tactical Skill to that `GoblinWarlord`'s list.

These behavior modifications can also be done on-the-fly, because the HTN planner considers the entire domain each time a new plan is created.

### Roles

*Roles* are the main way party-like behaviors are defined in *Blue Protocol*. Some Roles include Supporters, who prefer to stay back and apply buffs, and Defenders, who protect characters with the Healer role. Every AI character in a battle has a role assigned to it.

Each role contains a set of Tactical Skills defining how the role should prefer to behave, and these role-dependent Tactical Skills are added to the HTN domain for planning.

Because roles interface with decision-making through Tactical Skills, it's easy to dynamically change the role of a character. For example, if a character's shield breaks in the middle of a fight and it can no longer block incoming attacks, its role can dynamically be changed from Defender to Attacker and this seamlessly leads to completely different behavior.

Planning on Tactical Skills and Roles are key to the coordinated behaviors in *Blue Protocol*, and in the next part we'll see how the Combat Coordinator works with individual Character AIs.

Next part: [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})

[Back to Table of Contents](#table-of-contents)

[^1]: [Hierarchical Task Network](http://en.wikipedia.org/wiki/Hierarchical_task_network)