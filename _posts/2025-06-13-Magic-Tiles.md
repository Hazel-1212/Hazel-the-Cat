---
layout: post
title:  "Magic Tiles"
date:   2025-06-13 18:42:19 +0800
categories: jekyll update
---

-  [Watch it in GitHub](https://github.com/Hazel-1212/Magic_Tiles.git)
- Keywords: *UART, Python, FPGA, Music Game*.

## **Introduction**
1. **About the game**

This project is a two-player rhythm game. Tiles with
a number fall from the top of the screen on laptop along a
rail, and players use keypad to input the number of the
falling tiles, while the switch allows song selection.

Two sets of 7-segment displays show individual
player scores. When either player reaches 60 points, the
Python interface is automatically closed, and LED color
indicates the winner.

The system uses bidirectional UART
communication, either via the FPGA board’s built-in
serial port or through AD2 pins.