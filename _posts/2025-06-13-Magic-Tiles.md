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

## **Implementation**
### **How we realize interaction between FPGA and Python**

#### **1. Build UART module**
First, I try to learn the principle of UART, especially the types of the signal and how the
states should be. It had been a long time before the module was completed, though, just as
you can see in our work log. We searched for much info online.

Figure 2-1. UART transmits one bit at once,

and we must wrap the desired signal between start bit (0) and stop bit (1).

In Figure 2-3, they show the transmitter of UART (module UART_TX_CTRL).

**(1) Baud rate setting**

We make our FPGA transfer 9600 bits in a second. In Row 19 and 23, we design a
counter to return 0 every 10416 clocks.

**(2) FSM (txState)**

⚫ RDY (Idle state)

Waits for SEND to go high.

⚫ LOAD_BIT

Loads txData as {1'b1, DATA, 1'b0. } That is,
stop bit (1), 8 data bits, start bit (0).

⚫ SEND_BIT

Uses bitTmr to maintain the baud rate.
Increments bitIndex until all 10 bits are sent
(start + 8 data + stop). 

Figure 2-2. UART_TX_CTRL

In terms of the receiver (module UART_RX_CTRL),

**FSM of receiver**

⚫ IDLE

Wait for the start bit. If start bit detected,
reset bit_timer and bit_index, and go to START.

⚫ START

Confirm the start bit is valid, if not, go back IDLE.

BIT_MID = ~half of BIT_TMR_MAX (Row 37), ensuring sampling at the center of the bit.

⚫ RECEIVE

Store a bit in shift_reg . After receiving 8 bits, move to STOP.

⚫ STOP

Wait for stop bit, transfer the 8-bit data to DATA, and return to IDLE.

#### **2. Write Python Code**

**(1) Connect with FPGA**

uart.py is to interact with FPGA.
We first check device manager to know the
port name “COM3” and set the baud rate as
9600 (the same as we design in Verilog). And
as Row 11 ~18 in Figure 2-5, we open serial
port via giving the info of FPGA UART.

**(2) Connect with AD2**

Thankfully, WaveForms SDK has provided
sample code for downloader to manage AD2
with Python/C++ (Ref[4]).
Yet, we must modify it. The file note_rx.py
is to used interact with AD2.

The main function of AD2 UART is to
receive notes when the game starts. In
Figure 2-6, Python send ‘R’ to FPGA to
request for left-hand notes and ‘r’ to request
right-hand notes. As Row 60-61, we must
decode the 8-bit received data with ASCII
to get the alphabet. The notes_raw (Row 68) is like ‘A2E2C3...’ with ABC being the
number corresponding to keypad and 123
being the duration.

Another function of AD2 is to detect
the rail location. There are 4 rails for a
hand, and we randomly determine it with
module lfsr. The random rail code is
obtained by detecting the high/low level of
the DIO[14] and DIO[15] when the new tile
is spawned.

**(3) Rendering**

We use pygame to do the rendering. In Figure
2-7, the left 4 rails are for left-hand player, and the
right 4 rails are for right-hand player.

#### **3. Signal Flow in the Game Loop**

This table describes how signals flow between the PC (Personal Computer) and FPGA during different phases of the rhythm game.

| Time          | Direction   | Function                                      |
|---------------|-------------|-----------------------------------------------|
| Game start    | PC → FPGA   | Request for notation                          |
| After request | FPGA → PC   | Transfer info about notes                     |
| During Game   | FPGA → PC   | Send user input (FPGA buttons or keyboard)    |
| During Game   | PC → FPGA   | Feedback on the correctness of the notes      |
| Game end      | FPGA → PC   | Close the game                                |