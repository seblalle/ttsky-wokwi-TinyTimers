<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->

# How it works:

## TIMER WITH 7-SEGMENT DISPLAY:

The circuit is a flip-flop-based timer that shows the passage of time on a 7-segment display. 
It runs from an oscillator with F_osc = 1 [Hz] and has 3 operating modes, selected with the switches according to which inputs are enabled: 
1. switch 1 activates the 16-second mode
2. switch 2 the 32-second mode
3. switch 3 the counting mode

# How to test:

## MODES 1 AND 2: 16-SECOND AND 32-SECOND TIMERS:

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


## MODE 3: CYCLE NUMBER ON THE DISPLAY:

It is activated with the last switch and uses a separate circuit made of three parts. 
1. A clock divider (the 4 lower flip-flops) lowers the frequency so that the change is visible.
2. A 4-bit counter (flip-flops with XOR and AND gates) outputs a binary number, for example 0101 = 5. 
3. A combinational decoder (ANDs and ORs, one block per segment a-g) converts that number into the segments to turn on. 
Each AND detects a combination of the 4 bits, direct or negated, and each OR gathers the cases in which the segment must be lit. 
For example, segment a is on for 0, 2, 3, 5, 6, 7, 8 and 9. The display counts in sequence (0, 1, 2...) and the reset returns it to 0.

Mode 3 flow:
slow clock -> 4-bit counter -> decoder (ANDs + ORs) -> 7 segments

# External hardware

It works with the integrated 7 segmented led, so it doesnt need any external hardware

## CABLE TYPES (COLORS)
The cables in the diagram use colors to distinguish the function of each signal:

1. Blue: clock signal, including the clock passed from one flip-flop to the next within the divider.
2. Green: feedback from NOTQ to D in the dividers, causing each flip-flop to change state on every edge; also used in the mode 3 decoder logic.
3. White: data and outputs. Includes the shift register, IN inputs, and lines leading to the segment OR gates.
4. Orange: reset. Originates at RST_N, passes through NOT gates, and clears the register when the time expires.
5. Red: clock enable (NOTQ from the "ready" flip-flop -> AND gate allowing the clock to pass) and VCC power supply.
6. Black: ground (GND).

Mode 3 adds three colors:

1. Light blue: detects when the counter is non-zero and enables the decoder.
2. Purple: detects the number 10 (in conjunction with the clock) to turn off the segments (final blink).
3. Brown: final outputs to OUT0 through OUT6.


# Disadvantages of the three entry modes
### MODES 1 AND 2: 96- AND 192-SECOND TIMERS
1. There are only two fixed durations. Modifying them requires adding or removing flip-flops, as they cannot be adjusted via the switches.
2. Resolution is low: the progression is displayed in 6 steps (every 16 or 32 seconds), and there is no indication of the remaining time within each step.
3. Cycles correspond to seconds only with a 1 Hz clock. On the actual chip, a slow external clock or a frequency divider would be required.
4. A RESET must be applied before each use; upon completion, the alarm remains active until the system is reset.
5. If the switch is deactivated mid-count, the bar empties in stages, but the count does not reset.
6. If switches 1 and 2 are activated simultaneously, the bars blend on the display because the OR gates do not prioritize either input.
7. Cascaded dividers and the clock-enable AND gate generate derived clock signals, which are more problematic in silicon (prone to transient faults or glitches).
8. The alarm consists solely of the central segment flashing, which may go unnoticed.

### MODE 3: CYCLE NUMBER DISPLAY
1. It is slow: each digit lasts for 16 clock cycles (i.e., 16 seconds at 1 Hz). With a fast clock, the changing digits would be unreadable.
2. It is exclusive: it only operates when switch 3 is set to 1 and switches 1 and 2 are set to 0. In any other configuration, the counter remains in a reset state. 3. The count stops upon reaching 10, and the display flashes. To resume counting, a RESET must be performed. 4. With the counter at 0000, all segments remain off, so the digit 0 is not displayed.
5. It is the largest block in the design (the decoder uses many AND and OR gates), which increases the occupied area.

## WHY FLIP-FLOPS?
1. A flip-flop stores 1 bit and changes state only on the clock edge, acting as the memory that tracks elapsed time; logic gates alone cannot perform this function.
2. By connecting NOT-Q to the D input, each flip-flop divides the frequency by 2. Using *n* flip-flops results in division by 2^n (e.g., 2^4 → 16 and 2^5 → 32), making this the most efficient method for measuring long time intervals with a slow clock signal.
3. In the shift register, each flip-flop tracks the current stage of the timer and activates a segment, thereby eliminating the need for a decoder.
4. D-type flip-flops with a reset input were selected; this allows the system to be reset to zero and the bar to be turned off once the process is complete.
5. The last flip-flop in each block stores the state indicating that the time has elapsed.

## WHY LOGIC GATES?
Flip-flops count and store state, but logic gates decide when each block counts and what appears on the display. In the design, they perform four functions:

1. **Clock control:** An AND gate for each timer (and2 for the 96s timer and and4 for the 192s timer) receives the CLK signal and the NOTQ output from the last flip-flop. While the timer is running, NOTQ is 1 and the clock signal passes through; when time runs out, NOTQ becomes 0 and the clock is blocked. This allows the counter to stop automatically, without adding extra logic to each individual flip-flop. In mode 3, one AND gate (and7) allows the clock signal to pass, while another (and6)—via a NOT gate (not4)—blocks it when the counter reaches 10.
2. **Output combination:** When the two blocks were connected directly to the same display lines, the outputs fed back into one another: in mode 2, the state from mode 1 would also be present, causing mode 1 to operate instead of mode 2. To prevent this, each line passes through an OR gate (or3 through or9) that receives an output from each block; this ensures that the outputs from the inactive block do not interfere with those of the active one. The alarm follows the same principle: an AND gate for each block (and1 and and3) combines the CLK signal with the output of the last flip-flop so that the center segment flashes only when the time is up, and an OR gate (or8) combines both alarm signals.
3. **Reset:** The RST_N signal is active-low, but the DR flip-flops are cleared by a high level; therefore, a NOT gate for each block (not1, not2, and not3) inverts the signal. Additionally, an OR gate (or1 and or2) combines the reset signal with the "ready" signal to clear the shift register, ensuring the bar turns off exactly when the alarm begins. 4. **Mode 3 decoding:** for each segment (a through g), a group of AND gates recognizes the bit combinations that trigger illumination—whether direct or inverted—and an OR gate combines these cases. Subsequently, other gates apply a mask: they disable the Mode 3 output when the mode is inactive to prevent interference with Modes 1 and 2, and they cause the display to flash upon reaching 10. A chain of NOT and AND gates (not5, not6, and and25 through and27) keeps the counter in a reset state unless switch 3 is set to 1 and switches 1 and 2 are set to 0, thereby prioritizing Modes 1 and 2.
