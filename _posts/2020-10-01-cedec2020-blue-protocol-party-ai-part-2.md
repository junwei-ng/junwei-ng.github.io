---
layout: single
title:  "[CEDEC 2020] Group Coordination AI Supporting the Party Battles of Blue Protocol - Dynamically Determining Battle Participants"
date:   2020-10-01 11:00:00 +0900
tags: [game-ai, cedec-2020, session-summary, character-ai, coordination]
categories: [conference-reports]
---

Part 2 of this series goes into detail on how Lead AI Programmer Yohei Hase of Bandai Namco Studios and his team implemented dynamic party formation in *Blue Protocol*.

Original title: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』 by 長谷 洋平（株式会社バンダイナムコスタジオ）
[Presentation slides on CEDEC Digital Library (signup required, free)](https://cedil.cesa.or.jp/cedil_sessions/view/2271)

---

### Table of Contents

This is a series going in-depth into the contents of the CEDEC 2020 session "Group Coordination AI Supporting the Party Battles of Blue Protocol" by Yohei Hase of Bandai Namco Studios.

1. [Introduction]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai %})
2. **Dynamically Determining Battle Participants**
3. [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})
4. [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})
5. [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

---

## Dynamically Determining Battle Participants

*Blue Protocol* has a hierarchical AI architecture, with an AI Director at the top handling the gameplay experience all the way to the Character AI at the bottom controlling the behavior of a single agent.

The layers directly involved in a single battle instance are the Character layer at the bottom and the layer directly above, the Combat Coordinator. The Combat Coordinator is responsible for the flow of a single battle instance, doing things like managing attack tokens[^1] and assigning each AI character their battle roles.

![*Blue Protocol's* AI hierarchy](https://i.imgur.com/ew8zlVd.png)

How, then, can we determine what a single "battle instance" is in order to assign a Combat Coordinator, and how can we also determine what characters are in that battle instance?

In a game with pre-determined battle participants like most Final Fantasy games or tactical games like X-COM, this is a straightforward task. In single-player or multiplayer co-op games, an easy solution may be to treat all active combat instances as a single "battle".

*Blue Protocol* does not have pre-determined battle participants as an enemy group can be "pulled" to other groups and other players can join ongoing fights to form one big battle. Players and player groups can also be very far apart from each other, so battle instances have to be managed separately.

The picture below shows a possible scenario in *Blue Protocol*.
![An example of a battle in *Blue Protocol*](https://i.imgur.com/tqiMnVu.png)

We want to treat the two monsters and player character near the foreground as one battle instance and the monster and other player in the background as another, as they're far apart and effectively two distinct fights.

At the same time, in the battle instance nearer to the camera, both monsters are separately-spawned and not in a pre-determined group, but because they're both targeting the same player (or player group) we want them to team up.

However, if the two players were to get closer to group up, we also want the option of merging the two instances into one.

### Determining Player Clusters

*Blue Protocol's* solution to determining "player groups" around the map is to perform hierarchical agglomerative clustering.

![Hierarchical clustering to determine player groups](https://i.imgur.com/QpK48pz.png)
The picture above shows an example hierarchy of players, with "player clusters" as determined by their algorithm surrounded by a yellow dotted box.

According to Hase, hierarchical clustering was chosen because unlike k-means clustering, the number of clusters does not have to be specified.

While hierarchical clustering is relatively computation-heavy compared to other methods, by capping the amount of work the algorithm can do each frame the algorithm can be made to have minimal performance impact. As the algorithm completes in a few frames, there is minimal impact to accuracy too.

The main metric used in the hierarchical clustering algorithm is each player's coordinates. Biases are also applied so players in the same party, for example, are more likely to be grouped together. 

![Metrics used in hierarchical clustering](https://i.imgur.com/BLgNUlA.png)
The picture above shows three player characters in the same area, with two players in the same party linked with the yellow line. Without bias (the picture on the left), the two players on the right are nearer to each other and so are grouped together, while applying a bias groups the two players in the same formal party in the same player group.

Of course, hierarchical clustering mainly based on spatial distance works for *Blue Protocol* because its combat only takes place at melee to mid-range. This method of analyzing player groups wouldn't work as well for shooters with the option of long-range combat in large levels like the *Battlefield* games or *PUBG*.

### Forming AI Groups

Each individual enemy AI agent has their own perception system, and sensed players are added to the individual agent's Target List. The cluster each player belongs to is also recorded.

![Assigning enemy to a Combat Coordinator](https://i.imgur.com/IGCOtsZ.png)

Generally, each player cluster will face off against one enemy party, managed by one Combat Coordinator.

Each enemy agent has their own Context Data, which among other things keeps track of Combat Coordinators it can potentially belong to, and scores them based on information like their own state (whether they are in battle, patrolling, or something else), their Target List, and (if they belong to one) their pre-determined enemy group.

The scores are calculated to highly-rank Combat Coordinators that contain nearer hostile targets, more hostile targets, and (if any) other pre-determined group members.

![Assigning a newly-spawned enemy agent to a group](https://i.imgur.com/p378z0v.png)

When an AI enemy character is spawned, the AI Director assigns it to the appropriate Faction Coordinator based on their faction data. The Faction Coordinator then either creates a new Group Coordinator or places the AI character into an existing Group Coordinator based on the AI character's Context Data.

The Combat Coordinator is one type of Group Coordinator specialized for combat situations, but there are other types such as the Patrol Group Coordinator for non-combat situations.

![On player cluster changes](https://i.imgur.com/mmqb3Yp.png)

The AI Director will modify player clusters when appropriate, and when that happens the AI Director can request the Faction Coordinator to update their Combat Coordinators. The Faction Coordinator will then create, modify, or delete the Combat Coordinators under management and reassign the characters under those Combat Coordinators if necessary.

![On enemy character Context Data change](https://i.imgur.com/Nfmzyx2.png)

Finally, if an enemy character's Context Data changes, for example if it enters a battle state from a patrol state, the individual's Character AI can request the Faction Coordinator to reassign them to another Group Coordinator.

With player clusters and the AI Groups, the AI of *Blue Protocol* is able to flexibly create reasonable battle instances and coordinate the AI characters in them, while dynamically merging and forming instances as necessary.

Next part: [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})

[Back to Table of Contents](#table-of-contents)

[^1]: Tokens are a conceptually-simple and commonly-used technique in game AI to limit the number of AI agents simultaneously performing some action. An AI agent needs to obtain a "token" before they can perform that action, and these tokens are limited in number. By changing the number of available tokens and how the tokens are distributed or reclaimed, a designer can tweak the difficulty and flow of situations involving those actions in an intuitive way. Often, this is used to control the number of AI enemies attacking the player at any given moment so the player is neither overwhelmed nor bored.
