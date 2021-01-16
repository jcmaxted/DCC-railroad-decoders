# DCC-railroad-decoders
This repository contains Arduino sketches that implement DCC decoder functions for model railroading

AccDec_Multifunction

Version 8.0 - This Arduino sketch is derived from Geoff Bunza's blog SMA20 Low Cost 17 Channel DCC Decoders 
Ver 6.01 with Sound,Triggered Sound,Stepper,Dual Motor,LED and Servo Control. I have enhanced the functionality 
of his original sketch, primarily with the intent of using it as the basis for servo controlled model railroad 
switch machines.  This sketch assigns a mode to up to 17 pins on the Arduino Pro Mini or UNO micro-controller
boards.  Each of the pins is controlled by 5 CVs which are documented as follows.  

CV usage for the various functions
   
 There are now eight different operating modes for each pin.  Each function is controlled by five
 CVs.  These are in order CVmode, CVincr, CVstrt, CVstop, and CVcurr.  The CVs are used in different 
 ways depending on the function mode
  
 CVmode = modeServo (0) - Servo output.  This mode drives a servo between the start and stop angles.  
   CVincr - Servo travel rate. Low value is slow, high value is fast. Value range 1 to 50.
            Set the CVincr = 0 to enter limit set mode.  First FX select starts servo 
            slowly moving in one direction.  Second FX select saves the stop angle and 
            starts servo moving in the opposite direction.  Third FX select saves the start 
            angle and sets CVincr = 1.
            If the servo moves in the wrong direction for function ON, set the MSB of 
            the CVincr value by adding 128 to the current value.
   CVstrt - Servo angle for function off
   CVstop - Servo angle for function on
   CVcurr - Last saved servo angle.  The servo starts at this angle when reset.
    
 CVmode = modeOnoff (1) - Output pin on/off.  This mode will set the associated output pin high or low.
   CVincr - N/A
   CVstrt - N/A
   CVstop - N/A
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
    
 CVmode = modeBlink (2) - Blink output pin.  This mode will blink the associated output pin.
   CVincr - Blink rate.  Low value is fast, High value is slow. Value range 0 to 255
   CVstrt - N/A
   CVstop - N/A
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
    
 CVmode = modeBlnk2 (3) - Output pins double blink.  This mode blinks the output pin and pin +1.
   CVincr - Blink rate.  Low value is fast, High value is slow. Value range 0 to 255
   CVstrt - N/A
   CVstop - N/A
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
   
 CVmode = modePulse (4) - Pulse output pin.  This mode pulses the associated output pin.
   CVincr - Pulse time; CVincr value * 10 milliseconds.  Value range is 1 to 255.
   CVstrt - N/A
   CVstop - N/A
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
    
 CVmode = modeFade (5) - Output pin fade.  This mode fades in the associated output pin.
   CVincr - Fade in time.  Value range is 1 to 255.
   CVstrt - N/A
   CVstop - N/A
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
    
 CVmode = modeFlikr (6) - Output pin flicker.  This mode flickers the associated output pin.
   CVincr - Flicker rate.  Value range is 1 to 255.
   CVstrt - Flicker minimum brightness.  Value range is 0 to 50.
   CVstop - Flicker maximum brightness.  Value range is 51 to 100.
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
   
 CVmode = modeSvmon (13) - Servo monitor.  This mode monitors a servo function position and turns on output
   CVincr - The function number to monitor
   CVstrt - The servo state to be detected; 1 = at stop angle, 0 = at start angle, 2, 3 = midpoint
   CVstop - Output on time.  0 = always on, 1 - 255 = pulse time
   CVcurr - Selects the output polarity; 1 = positive, 0 = negative
    
 CVmode = modeActiv (14) - Function activation.  The associated input pin is used to activate another function.
            The associated pin is configured as INPUT_PULLUP.  Connect switch from pin to ground
   CVincr - The function number to activate
   CVstrt - The activation mode: 0 = funciton off, 1 = function on, 2 = momentary, 3 = toggle
   CVstop - Switch debounce time.
   CVcurr - Current switch state 1 = off, 0 = on
   
 CVmode = modeUndef (15) - Function undefined.
 
 The modeSvmon (13) is intended to monitor a servo output (modeServo) and to generate a signal on the associated 
function pin.  The servo state to be detected can be one of four conditions; servo at the start or stop position or 
servo at the midpoint moving to either the start or stop position.  Action can be either activate the output when
the detected condition is true or generate at pulse output when the detected condition is true.  I have used this
mode to light an LED when the switch is at the desired position; red for diverging route or green for through route.
I an currently using this mode to pulse a dual coil latching relay; the relay contacts then control power to the 
switch frog and the indicating LEDs

The modeActiv (14) is intended to monitor the associated pin as an input.  The pin is configured as input with pullup
and connected to a pushbutton switch to ground.  When the input is debounced and when the desired input state change is 
detected, the desired function is activated.  One of four activation modes can be selected; turn the function on, turn 
the function off, turn the function on when pressed then turn the function off when released (momentary) or first press
turns the function on, second press turns the function off (toggle).  I currently use this mode to activate the switch
servo with a pushbutton at the switch.

Assigning accessory addresses:
 The accessory addresses to which this decoder will respond start with the address defined in "accessory_address"
 (initially 40) and increment by 1 for every defined active function.  This has the effect that even though the
 decoder may have a maximum of 17 functions, it will not respond to accessory addresses above that needed to control
 the decoder's active functions.  What determines the number of active functions is the function mode value.  A 
 function mode value less than modeSvmon (13) is classified as an active function since the function is assumed to
 be controlled by an accessory address.  Function codes at and aboe modeSvmon (13) are supervisory modes or not 
 defined.  For example, assume that function F0 = modeServo, F1 = modeSvmon, F2 = modeSvmon, F3 = modeActiv, 
 F8 = modeFlikr, F10 = modeBlink and the rest of the functions are set to modeUndef.  When the decoder is initialized,
 the decoder will only respond to accessory addresses 40 for F0, 41 for F8, and 42 for F10.  The rest of the functions
 are not counted since they are undefined or supervisory functions.  Note that if decoder CVs are modified to enable 
 disable functions, the changes will not take effect until the decoder os reset.

