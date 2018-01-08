_Zephyrr's notes_

There can be 1 to 10 Segments.

Each segment controls a contiguous portion of the overall neopixel string
*By default there is one segment controlling the whole string*

There are two structures per segment:
- segment describes the segment configuration (not normally evolved within the library, must be called from sketch)
- segment_runtime has runtime info tracking state of an executing effect, changed dynamically as library runs

~~~~
    typedef struct segment {
      uint8_t  mode;
      uint32_t colors[NUM_COLORS];
      uint16_t speed;
      uint16_t start;
      uint16_t stop;
      bool     reverse;
    } segment;
~~~~
This specifies one segment, its effect, and some common paramaters; this would normally be changed by external code as
part of configuring and setting up effects.
- start and stop are indexes into the neopixel array
- mode is the index of the selected effect (mode) for this segment; the rest are available resources for this segment:
- speed gives the effect an idea of speed in mSec (for simple effects it's the length of the cycle, but can be more complicated)
- colors is 1..3 colors used as a palette for the effect
- reverse is a state, effect fwd or bwd

~~~~
  // segment runtime parameters
  typedef struct segment_runtime {
    uint32_t counter_mode_step;
    uint32_t counter_mode_call;
    unsigned long next_time;
    uint16_t aux_param;
  } segment_runtime;
~~~~
This contains dynamic data about the current execution, changed during execution by the effect and the overall effect controller.
- next_time is the millisecond time when the effect should be called again (maintained and used by control)
- counter_mode_call is a counter for how many time the effect function has been called (maintained by control, used by effect)
    one usage would be doing phases of the effect by modulo of this number
- counter_mode_step is used by the effect to step its effect; typically it might be an index of an LED (maintained & used by effect)
- aux-praam is an auxiliarry data storage for whatever the effect wants to save (maintained and used by effect)

It's not clear to me why these are separate; they seem to be used together in lockstep.  Maybe I missed something, or
perhaps it's for some future extensio.

Anyway the basic overview it that after setting up some segments, one periodically calls service(), which then checks next_time
of each segment against the current millis() and calls each if needed; it receives back the delay before the next call.  If any
segment gets calls, then adafruit's neopixel library pushes the data out to the pixels.

~~~~
void WS2812FX::service() {
  if(_running || _triggered) {
    unsigned long now = millis(); // Be aware, millis() rolls over every 49 days
    bool doShow = false;
    for(uint8_t i=0; i < _num_segments; i++) {
      _segment_index = i;
      if(now > SEGMENT_RUNTIME.next_time || _triggered) {
        doShow = true;
        uint16_t delay = (this->*_mode[SEGMENT.mode])();
        SEGMENT_RUNTIME.next_time = now + max((int)delay, SPEED_MIN);
        SEGMENT_RUNTIME.counter_mode_call++;
      }
    }
    if(doShow) {
      delay(1); // for ESP32 (see https://forums.adafruit.com/viewtopic.php?f=47&t=117327)
      Adafruit_NeoPixel::show();
    }
    _triggered = false;
  }
}
~~~~
For each mode there are RAM arrays:
-  _mode[] has pointer to (member) function implementing effect
-  _name[] has pointer to flash resident text string name of effect

The effects are sometimes standalone.  Other times a useful and more general function is created and configured/customized
to give the specific effect desireed.

An effect (mode) member function (called through a pointer above).
