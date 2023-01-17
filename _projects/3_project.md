---
layout: page
title: Integrated Design Project
description: Software drivers for sensor integration with Arduino microcontroller.
img: assets/img/3_project/1.jpg
importance: 3
category: university
---

**Nov-Dec 2021**

As the software-lead for my team, I was responsible for writing software drivers for writing external libraries for individual sensors and sub-systems.

<div class="row">
    <div class="col-sm mt-4 mt-md-0">
        {% include figure.html path="assets/img/3_project/2.jpg" title="CAD design" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-4 mt-md-0">
        {% include figure.html path="assets/img/3_project/1.jpg" title="final prototype" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
        On the left, initial CAD design. Right, the realized prototype.
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3_project/3.jpg" title="system layout" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Block diagram / system layout.
</div>


The overall system is composed of multiple sub-systems that interact together to execute a sequence of tasks to fulfil the given mission. Each sub-system is responsible for handling a unique set of components.

The drive and navigation sub-system consist of line sensors, the accelerometer, and the DC motors. Aside from controlling the motion of the robot (moving forwards/backwards, steering and rotating), this sub-system uses an accelerometer to identify the position of the robot, which are classified into 4 states – normal ground, ascending or descending the ramp and on the ramp (elevated ground). This allows the robot to not only be aware of its location – the starting half or the collection half of the map, but also allows for a line following algorithm with dynamic steering capabilities (smaller turning radius on level ground, but a larger turning radius when climbing the ramp in exchange for increased torque).

The dummy detection and identification sub-system consist of the IR receiver, the ultrasound sensor, and two phototransistors, which are hooded and unhooded respectively. The unhooded phototransistor picks up IR signals emitted by the dummies in a larger radius, providing an approximate direction of the dummy relative to the robot. The hooded phototransistor has a much smaller detection radius and is used in conjunction with the IR receiver to precisely align the robot to the dummy and identify the dummy. With the ultrasound sensor, the robot can stop within 5 cm of the dummy as required.

The dummy collection sub-system consists of two servos, one for opening or closing the claw, and another for raising or lowering the claw. The LED indicator sub-system consists of the red, green, and amber LEDs.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3_project/4.jpeg" title="code structure" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Code structure & algorithms.
</div>

For modularity, a custom library is defined for each component (except for line sensors due to latency considerations and the LEDs for simplicity), with their implementation being masked from the main program. The main program is written in a non-blocking manner, such that the tasks are executed continuously in the loop function, allowing for the accelerometer to operate in the background. The tasks are executed in sequence from the top to bottom as shown in the figure above. However, blocking functions and code segments are used for more complex manoeuvres which require the use of delays.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/3_project/5.jpg" title="execution flow" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Execution flow.
</div>