<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

## How it works

# TIMER WITH 7-SEGMENT DISPLAY

The circuit is a flip-flop-based timer that shows the passage of time on a 7-segment display. 
It runs from an oscillator with F_osc = 1 [Hz] and has 3 operating modes, selected with the switches according to which inputs are enabled: 
1. switch 1 activates the 16-second mode
2. switch 2 the 32-second mode
3. switch 3 the counting mode

## How to test

# MODES 1 AND 2: 16-SECOND AND 32-SECOND TIMERS

The circuit contains two independent blocks, one of 96 seconds (1:36 [min]) and one of 192 seconds (3:12 [min]). 
Each block has two parts. The lower row of flip-flops acts as a seconds counter (trigger): 
it divides the 1 [Hz] clock and generates one pulse every 16 or 32 seconds. 

Each pulse advances the main row of 7 flip-flops, whose outputs go to the display: 
six light up the segments of the circle and the seventh controls the blinking. 

Adding one flip-flop to the seconds counter doubles the cycle: 2^4 -> 16 and 2^5 -> 32, where the exponent is the number of flip-flops. 
Since the circle has 6 segments, the total time is 6 x 16 s = 96 s with 4 flip-flops and 6 x 32 s = 192 s with 5 flip-flops.

The flip-flops are DR type so that the outputs can be reset. 
OR gates were added in the upper part to merge the outputs of both blocks onto the same lines going to the display. 
This way, the outputs of the 32-second timer do not feed back into the 16-second timer, and the block that is switched off does not run by mistake.

When the cycle is complete, the display keeps blinking: 
an AND gate receives the clock and the output of the last flip-flop, and its output goes to the 7-segment display.

- Mode 1: the timer advances every 16 seconds and is activated with switch 1.
- Mode 2: the timer advances every 32 seconds and is activated with switch 2.

To use it, first apply the reset and the 1 [Hz] clock. Then flip the switch (1 or 2), and the seconds indicator (dot) lights up together with the top segment. 
After 16 or 32 seconds the right-hand segment lights up, and so on clockwise until the circle is complete (steps 1 to 6).


# MODE 3: CYCLE NUMBER ON THE DISPLAY

It is activated with the last switch and uses a separate circuit made of three parts. 
1. A clock divider (the 4 lower flip-flops) lowers the frequency so that the change is visible.
2. A 4-bit counter (flip-flops with XOR and AND gates) outputs a binary number, for example 0101 = 5. 
3. A combinational decoder (ANDs and ORs, one block per segment a-g) converts that number into the segments to turn on. 
Each AND detects a combination of the 4 bits, direct or negated, and each OR gathers the cases in which the segment must be lit. 
For example, segment a is on for 0, 2, 3, 5, 6, 7, 8 and 9. The display counts in sequence (0, 1, 2...) and the reset returns it to 0.

Mode 3 flow:
slow clock -> 4-bit counter -> decoder (ANDs + ORs) -> 7 segments


## External hardware

It works with the board 7 segment led, so it doesnt need anny external hardware
