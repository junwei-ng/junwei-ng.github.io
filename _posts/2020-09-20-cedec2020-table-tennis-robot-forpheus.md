---
layout: single
title:  "[CEDEC 2020] Mini-summary: Applying Game AI Technology to the Table Tennis Robot Forpheus"
date:   2020-09-20 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary-mini, meta-ai, robots]
categories: [conference-reports]
---

This session goes into the tech behind [Omron's table tennis-playing robot Forpheus](https://www.youtube.com/watch?v=UbVVcyZ8alc) and the techniques used by the AI developers from collaborators Square Enix to create a more engaging experience for the player.

Four speakers -- Youichiro Miyake and Yuta Mizuno from Square Enix, Masamune Nakayama from Omron, and Kensuke Fujita from Loftwork -- jointly present this session. This article mainly summarises the hardware and software tech in Forpheus, as covered by Omron's Nakayama and Square Enix's Mizuno.

Original title: 『卓球ロボット「フォルフェウス」におけるゲームAI技術の応用』 by 三宅　陽一郎（株式会社スクウェア・エニックス）、中山　雅宗（オムロン株式会社）、藤田　健介（株式会社ロフトワーク）、水野　勇太（株式会社 スクウェア・エニックス）

---

* The sensing portion of the robot's AI architecture uses:
	* 2 cameras in a stereo layout to determine the location of the ball in 3D space
	* 1 camera to record the orientation of the player's racket (helped by green stickers on the hitting surface of the racket), and
	* 2 depth sensing cameras for player pose estimation.
* In addition, not only is the player's face is tracked with another camera for emotion detection, the player's heart rate and other biological signs are also recorded.
* ![Algorithm for ball trajectory prediction](https://i.imgur.com/QpaBrkD.png) The ball trajectory is predicted using a physical model (aerodynamics + table bounce) combined with an original rotation prediction algorithm and estimated with an unscented Kalman filter.
* ![Algorithm for return stroke computation](https://i.imgur.com/FE3UvTP.png) A physically-viable return stroke is computed with a physical model (aerodynamics + racket bounce) and an original two-point boundary value problem solver. The software model and precise hardware controls allows Forpheus to send the ball back with an error of at most just 100mm.
* The key improvement on the previous iteration of Forpheus (from a gameplay perspective) is the inclusion of a "Meta AI".
	* ![General architecture of a Meta AI, from Mizuno and Satoi's GDC 2019 session](https://i.imgur.com/2dKx6gZ.png) "Meta AI" is defined by Mizuno and other Square Enix AI researchers as an AI system that senses the whole game world and dynamically controls the game, in contrast to "Character AI" (behavior of individual characters) and "Navigation AI" (pathfinding and locomotion.) For more details, refer to Mizuno and fellow Square Enix designer Daiki Satoi's presentation at GDC 2019[^1].
* The Meta AI aims to keep players engaged by controlling the difficulty of the game and guiding the players' emotions. The player's skills are estimated by a neural network from their returns and their engagement with the game is estimated using data from the camera and biological sensors. Players generally get returns that they are able to handle, but Forpheus will return the ball to harder positions to keep things interesting.
	* In theory, players' engagement increases when they're able to overcome a return that the player thinks is "difficult", but a return that is overly-difficult will backfire and reduce players' engagement.
	* The developers found that this bit of design led to more interesting and varied rallies.
* ![Circumplex model for Forpheus (right)](https://i.imgur.com/xsJ0bzk.png) Mizuno developed an emotion circumplex model for Forpheus with axes of "difficulty of challenge" and "success rate". This model was based on [James Russell's emotion circumplex model](https://www.wikiwand.com/en/Emotion_classification#/Circumplex_model) with axes modified and parameterized for Meta AI control. The hypothetical "ideal state" of the player is in the top right quadrant of the graph, where the player is overcoming what they perceive to be relatively difficult challenges with a relatively high success rate.
	* "Difficulty of challenge" is rated on rally speed (lobbing or fast balls) and expected return stroke (backhand or forehand, cross or straight ball.)
	* "Success rate" is the likelihood of the player successfully returning the ball. This is calculated based on factors like the ball's trajectory, spin, and speed.
	* While Square Enix had previously developed a two-axis model for modelling and guiding player emotion (presented in the same GDC session describing Meta AI[^1]), the Forpheus team found that the feelings on the axes of that model ("fear of loss" and "hope of win") were not directly applicable to the moment-to-moment play in a round and set-based game like table tennis.
* This emotion model provides a quantitative foundation for the "Game Maker" component of the Forpheus Meta AI. Using factors like the current difficulty, player's perceived emotional state, and player's estimated skill, the Game Maker determines a few suitable challenges (continue the current difficulty? return balls that are easier for the player?) for the player to move their emotional state towards the "ideal" top-right quadrant and the actions Forpheus should output (forehand stroke? increase ball speed?)
* The "Operation Generator" component then computes viable physical movements to achieve the actions generated by the Game Maker, taking into account factors like the state of the ball and the player's position. These movements are then passed to and executed in the "Interaction Space" -- in this case, the hardware controlling the racket.
* The team concludes that the concept of a Meta AI is viable for games in a physical space, not just in video games. However, as emotions are currently estimated on a per-rally basis, one problem is underestimating changes in player emotion between rallies, like after they fail to return a ball. In theory, the Meta AI design allows for inputs over long time spans, so they would like to improve their emotional model and have it estimate player emotions over the course of a game.

[^1]: Changing the Game: Measuring and Influencing Player Emotions Through Meta AI by Yuta Mizuno and Daiki Satoi (Square Enix), presented at GDC2019 -- [GDC Vault](https://gdcvault.com/play/1025902/Changing-the-Game-Measuring-and), [slides](http://www.jp.square-enix.com/tech/library/pdf/MetaAI_GDC2019.pdf)

