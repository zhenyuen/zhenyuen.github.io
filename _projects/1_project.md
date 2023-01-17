---
layout: page
title: Space Invaders 2.0
description: A CLI-based game written in C++.
img: assets/img/1_project/2.jpg
importance: 1
category: university
---

**May 2020**

My first C++ project. It is an CLI-based game inspired by the popular arcade game, Space Invaders, with added twists and features. Installation is done by running a bash script which invokes a Makefile. Recorded gameplay clips can be found [here](https://www.youtube.com/watch?v=dEybnJ5xhr0&t=116s).

Space Invaders (Japanese: スペースインベーダー, Hepburn: Supēsu Inbēdā) is a 1978 arcade game created by Tomohiro Nishikado (Wikipedia, 2020).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/1_project/1.jpg" title="gameplay" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Gameplay screenshots.
</div>

This project introduced me to object-oriented programming, multi-threading and basic algorithms/data structures.

One of the harder challenges when developing the game was implementation of the bullet-collision mechanism. Bullets fired by enemy units and your weapons are stored individually by their coordinates in a linked-list. Due to the use of concurrency/multi-threading for receiving user-inputs, computing the game state and updating the display, deletion of intermediate nodes (corresponding collided bullets) in the linked list must be performed carefully.