---
layout: single
title:  "[CEDEC 2020] Mini-summary: How is the aibo's character AI implemented?"
date:   2020-09-19 10:00:00 +0900
tags: [game-ai, cedec-2020, session-summary-mini, robots, behavior-trees]
categories: [conference-reports]
---

[Sony's robot pet *aibo*](https://aibo.sony.jp/) has come a long way from [its original 1999 incarnation](https://www.sony.co.jp/SonyInfo/design/gallery/ERS-110/). Sony engineer and aibo evangelist Yoshihide Fujimoto describes the design and tech in the 2018 aibo.

Original title: 『aiboのキャラクタAIを如何にして実現したのか？』 by 藤本 吉秀（ソニー株式会社）

---

* The design goal of the aibo is to have it "feel like a living thing", a companion to humans with believable behavior. Unlike other Sony products where humans initiate interactions, the aibo was designed with the goal of making a product that autonomously wants to get closer to humans.
* The 2018 aibo boasts an impressive suite of sensors, including a SLAM sensor on the back and touch sensors on the head, chin, and back to detect your pats.
* The information obtained is processed by a human-designed behavior system enhanced by cloud-based machine learning -- for instance, the robot dog can be updated with new behaviors after [owners "train" it to recognize a new gesture](https://us.aibo.com/fan/challenge/snoot/).
* These behaviors are then acted out by the 22 actuators in the robot's body and expressive LED eyes, allowing for a diverse array of movements.
* !["Doggie language" designed for the aibo](https://i.imgur.com/NtBIum6.png) The body language of the aibo was designed using a simulator built in Unity. Lifelikeness and cuteness were realized by adding behaviors that would be unnecessary in an industrial robot, such as "sniffing" or "frolicking".
	* Fujimoto brought an aibo to the session, and I didn't notice any repeated "idle" animations during the 3 minutes' Q&A.
* !["Approach human" action plan](https://i.imgur.com/Pcfv9cX.png) Behaviors were implemented using a behavior tree. The picture shows the BT for the "approach human" action plan. Blue nodes represent actions required to achieve the "approach human" task. Pink nodes represent actions that are solely to express the aibo's personality and life-likeness. A substantial amount of effort was put in these strictly-unnecessary actions, in contrast to Sony's other consumer/industrial products.
	* I wasn't aware of this before, but apparently the use of BTs has been gaining traction in the robotics world -- some recent examples: a [workshop specifically on BTs in robotics](https://behavior-trees-iros-workshop.github.io/) at last year's International Conference on Intelligent Robots and Systems, and a textbook, [Behavior Trees in Robotics and AI](https://btirai.github.io/) published in 2018. Perhaps game AI developers working with BTs can get some ideas from the robotics world now that more people are researching BTs?
* The aibo can be programmed with a web API and also comes with a [visual programming tool](https://us.aibo.com/fan/visual_programming/), which I think is a great way to get people interested in programming.
* The engineering team found a creative way to "detect" belly pats even though they didn't add a touch sensor there -- apparently, data from the six-axis gyro was successfully used to train a model to idenfity belly pats, and similarly data from the six-axis gyro was also used to distinguish between pats on the head and harder smacks.

