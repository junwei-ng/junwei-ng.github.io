---
layout: single
title:  "[CEDEC 2020] Group Coordination AI Supporting the Party Battles of Blue Protocol - Introduction"
date:   2020-10-01 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary, character-ai, coordination]
categories: [conference-reports]
---

In this session, Lead AI Programmer Yohei Hase of Bandai Namco Studios describes the various techniques used to coordinate and implement group behaviors in their upcoming online action RPG *Blue Protocol*. If you're working on squad/party-based character AI, this session contains some great ideas for implementing dynamic party formation, group decision-making, and inter-agent communication.

Original title: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』 by 長谷 洋平（株式会社バンダイナムコスタジオ）
[Presentation slides on CEDEC Digital Library (signup required, free)](https://cedil.cesa.or.jp/cedil_sessions/view/2271)

---

### Table of Contents

This is a series going in-depth into the contents of the CEDEC 2020 session "Group Coordination AI Supporting the Party Battles of Blue Protocol" by Yohei Hase of Bandai Namco Studios.

0. **Introduction**
1. [Dynamically Determining Battle Participants]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai-part-2 %})
2. [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})
3. [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})
4. [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

---

Hase has spoken at every CEDEC since 2015, and this is a follow-up to his excellent presentation at CEDEC 2019 on the [decision making architecture in *Blue Protocol*](https://cedil.cesa.or.jp/cedil_sessions/view/2102). This year's session focuses on the tech coordinating AI-controlled enemy monsters, bringing to life one core pillar of *Blue Protocol* -- dynamic party battles.

### The Context Behind the Tech

*Blue Protocol's* PvE[^1] party battles has three main design goals:

1. AI parties need to work together and act like a party.
2. Battles need to be tactical and the AI enemies must feel like their actions have reasonable intentions.
3. Battles must feel dynamic. Human players must be able to casually team up in the field and fight together against the AI without formally creating a party, and each battle must feel fresh and mustn't have the AI act the same way again and again.

As a 3D multiplayer online action RPG, *Blue Protocol* features multiple player classes and numerous enemy types. Players and AI enemies are able to move freely around the environment and use a wide variety of skills in real-time. At a basic level, the AI thus has to support reasonable-looking movement, attacking from various ranges, appropriately using skills, and a variety of character archetypes and sizes.

Being an online game, multiple human players can share the same game space. Additionally, there's the design requirement of allowing players to casually team up in the field.

On the flip side, if you were fleeing from a group of AI-controlled enemies and ran into another player fighting another group of AI enemies, it'd be more interesting if those two fights merged into one bigger battle. For that to happen, the previously-distinct groups of AI enemies have to be able to team up to take both players down.

This means the game has to coordinate a larger number of AI enemies than a single-player game and across distinct fights in the same scene, and that the AI can't just look for a magic `Player` object or fixed "Player Party" to focus their behaviors around. 

Hase summarizes this context into two broad problems the AI team needs to tackle:
* The party battle AI system has to be robust and be able to handle the coordination of a large variety of AI character types and on-the-fly changes to the participants of a battle.
* The AI system in general needs to be performant and efficiently process and coordinate the behaviors of large number of AI agents.

To this end, he describes how *Blue Protocol's* AI solves these issues in four areas:

1. [Dynamically Determining Battle Participants]({% post_url 2020-10-01-cedec2020-blue-protocol-party-ai-part-2 %})
2. [Character AI Behavior]({% post_url 2020-10-02-cedec2020-blue-protocol-party-ai-part-3 %})
3. [Tactical Decision-Making]({% post_url 2020-10-03-cedec2020-blue-protocol-party-ai-part-4 %})
4. [Inter-agent Communication]({% post_url 2020-10-04-cedec2020-blue-protocol-party-ai-part-5 %})

The links above will bring you to the relevant posts in this series!

[^1]: Player vs Environment, a term referring to human players battling against AI-controlled characters. c.f. PvP -- Player vs Player -- where human players battle against other human players.
