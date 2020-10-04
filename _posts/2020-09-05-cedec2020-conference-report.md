---
layout: single
title:  "[CEDEC 2020] Conference Report"
date:   2020-09-05 10:00:00 +0900
tags: [game-ai, cedec-2020]
categories: [conference-reports]
---

[CEDEC (Computer Entertainment Developers Conference)][cedec-url] is the largest and most-attended annual conference in Japan focused on game development. It is to the Japanese industry what GDC is to the English-speaking industry, and *the* place to learn about the latest in Japanese game development. 

As far as I know CEDEC does not have an equivalent of the GDC Vault where full session recordings are archived. With a CEDEC conference pass, attendees can view recordings for about a week after the event ends -- then they're gone.

However, their [official Youtube channel][cedec-youtube] has uploaded a small number of sessions from prior conferences and most sessions make their materials available on the [CEDEC Digital Library][cedil-url] (completely free, sign up required).

Most sessions are fully in Japanese, but every year there are several sessions in English mostly by invited non-Japanese speakers. However, the [CEDEC Digital Library][cedil-url] is simple to navigate and presentation slides contain a lot of detail, so even if you don't read Japanese you can follow most presentations with help from a translation tool.

I did not attend this year's CEDEC "live" due to work so for me this was purely lecture-watching, but there were unofficial post-conference mixers held online which seemed pretty fun!

## Sessions

The 2020 iteration featured about 200 sessions over 7 tracks. Due to the COVID-19 pandemic the conference was fully online, but some speakers spoke "live" in rooms provided by the organizers. Three "conference rooms" were also continually streamed on Youtube for free, which I thought was a great idea for casual viewers.

CEDEC generally has a fair number of sessions in both "traditional" game AI and more experimental machine learning approaches. In general, "traditional" game AI sessions are dominated by console game projects mostly from the larger companies, while machine learning sessions are a split between mobile game projects and startups. There were also several sessions on test/QA automation, continuing

The sessions I found most useful for a gameplay AI programmer were:

* [**The Group Coordination AI Supporting Blue Protocol's Party Battles**](https://cedil.cesa.or.jp/cedil_sessions/view/2271) by Yohei Hase (Bandai Namco Studios)[^2]
 
* **Knowledge Overload : Keeping AI Knowledge Organized In Large Scale Projects** by Guillaume Bouilly (Square Enix)

* **"Meta AI Design Patterns" For Your Game Design: 15 Basic Patterns** by Yuta Mizuno (Square Enix)[^3]

There's a lot of good info in these sessions I intend to write about in future posts.

Two sessions discussed the use of game AI-inspired technology in entertainment robots.

* **How is the aibo's character AI implemented?** by Yoshihide Fujimoto (Sony)[^4]

* **Applying Game AI Technology to the Table Tennis Robot Forpheus** by Youichiro Miyake (Square Enix), Masamume Nakayama (Omron), Kensuke Fujita (Loftwork Inc.), Yuta Mizuno (Square Enix)[^5]

This crossover is probably unsurprising since game AI's basic sense-think-act loop is derived from robotics and game AI developers are basically making virtual autonomous robots.

What's interesting to me was how *directly* game AI techniques were applied -- behavior trees being the core control system of the aibo, and the Meta AI architecture and game design techniques used in the Forpheus.

It's likely that game AI developers can learn from robotics too. Perhaps not so much in the hardware aspect, but certainly concepts and ideas in sensing and control systems and maybe data management or debugging.

There's also one session on pathfinding, and I found it interesting. I'm not an expert on pathfinding/navigation, but this session reflects certain trends in efficient 3D spatial data representation, like the use of hierarchical grids.

* [**A Pathfinding System For Next-Generation AI: Collision Aware Multi-Phase Multi-Layer 3D Path Search**](https://cedil.cesa.or.jp/cedil_sessions/view/2251) by Tsai Ting-Nan (Luminous Productions)[^6]

Some other thoughts (disclaimer: I barely know anything about ML/DL):

* Regarding learning bots, actually have to *play* the game seems like a limiting factor in attempts at getting decent results because of the time it takes. I wonder if a turbo mode would help, and if the next generation of engines from companies serious about using learning algorithms will have more functionality supporting learning.

* In a similar vein, the ability to record/rewind/replay/replicate game state is incredibly useful not just for machine learning but also for development purposes. Very specialized state histories are often built to help with debugging specific pieces of code, but a generalized implementation at the engine level would supersede that. Is "recording everything" feasible? For smaller tech demos developed by a few people, deciding what state to store isn't a huge problem, but how would this scale to a full-sized development team with people of varying skill levels?

* Speaking of rewind/reply, I wonder how the ["game debug history"](https://gdcvault.com/play/1023382/AI-Behavior-Editing-and-Debugging) of *The Division's* Snowdrop engine works and how many engines have such "first-class debug" functionality implemented.

This year's CEDEC maintained the technical depth of previous years', which was a great boon for learning during these COVID times. I really wish CEDEC granted access to session recordings like the GDC Vault though. There's lots of good stuff there.

[cedec-url]: https://cedec.cesa.or.jp/
[cedec-youtube]: https://www.youtube.com/channel/UCmHaPXvwn9_4pMNAV6ewgoA/videos
[cedil-url]: https://cedil.cesa.or.jp/

[^2]: 『BLUE PROTOCOLのパーティバトルを支える集団制御AI』　長谷 洋平（株式会社バンダイナムコスタジオ）
[^3]: 『ゲームデザインにおけるAI活用のための「メタAIデザインパターン」　―基本15パターン―』　水野　勇太（株式会社 スクウェア・エニックス）
[^4]: 『aiboのキャラクタAIを如何にして実現したのか？』　藤本 吉秀（ソニー株式会社）
[^5]: 『卓球ロボット「フォルフェウス」におけるゲームAI技術の応用』　三宅　陽一郎（株式会社スクウェア・エニックス）、中山　雅宗（オムロン株式会社）、藤田　健介（株式会社ロフトワーク）、水野　勇太（株式会社 スクウェア・エニックス）
[^6]: 『~次世代ゲームAIを支えるパス探索システム~衝突回避コストを軽減する多段階多階層3D空間パス探索』　蔡　廷南（株式会社Luminous Productions）