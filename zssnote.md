_Zephyrr's notes_

There can be 1 to 10 Segments.

Each segment controls a contiguous portion of the overall neopixel string
*By default there is one segment controlling the whole string*

There are two structures per segment:
- _segment describes the segment configuration (not normally evolved within the library, must be called from sketch)
- _segment_runtime has runtime info tracking state of an executing effect, changed dynamically as library runs

_segment
-  has a start and end index into neopixel string
-  is controlled by one effect (idx 0 .. 55 currently)
-  has a palette of 1..3 colors
-  has a speed (mSec); for simple effects, this is the time between updates

_segment_runtime
-  has a number of iterations (calls) counter, since effect begun on this segment

main loop:
-  for each active segment, checks whether it's ready to be called again
-  if so, calls and uses returned time to set next call time
-  can also call if triggered (rather than by elapsed time)

For each mode there are RAM arrays:
-  _mode[] has pointer to (member) function implementing effect
-  _name[] has pointer to flash resident text string name of effect


