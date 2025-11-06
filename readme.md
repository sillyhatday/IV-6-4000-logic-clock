# IV-6 VFD Clock w/ 4000 Series Logic

This is my take of a classic project dating back to the 1970's. The amount of more modern projects I've done has worn me down, having to fiddle around with tiny SMD components and waiting on PCBs to be delivered before I can do anything. I've jumped back in time, back to the kinds of things I was doing at college, using through hole components. Being able to pull out a breadboard and start pushing components into it to make a circuit, is refreshing. Testing ideas there and then makes things much more interesting. The only trouble is, ideas pile up!

For the longest time I've had the parts for building a clock from discrete logic. I just never knew how to design it to be usable. By this I mean, do I have a bare PCB, an enclosure, multiple PCBs, or something to hang from the wall. So the project got abandoned.

It wasn't until I saw a kit for such a clock on AliExpress, that my spark for this came back. I grabbed the kit and as soon as it arrived I began reversing the circuit board. I've made an exact copy of that kit in Kicad. Maybe I'll put that up here also.

So this project is a mixture of different ideas and existing projects. The main difference being the  display technology used here is vacuum fluorescent displays. These displays being right angled, made it a simpler design as a single PCB laid flat would work. Stick it on a base, line up the displays and they point sideways. (My next project isn't so simple).

<img width="1730" height="1210" alt="4000-logic-clock-VFD-front" src="https://github.com/user-attachments/assets/84c5f874-55e6-47f3-89d4-c31001160805" />

## Project Breakdown

### Features

There are not many features in this project. It is a bare bones style clock. It's mostly thing that would be taken for granted, such as, voltage regulation that compensates for input supply voltage changes, hardware button debouncing, compatibility with modern USB C chargers, a pause switch and the main thing, a display on off control. This can be used for turning off the displays while the clock continues to keep time, or just to save the life of the tubes while not being used.

### Displays

The reason you are probably reading this, the display technology. I'll not write the history of vacuum tubes here, there are plenty of places to read about that. The main thing to know about them is that they are beautiful.

I chose the IV-6 as I've previously used IV-3's for another project, and I had 6x IV-6 in stock. Mostly as I already had them. They are also quite common in the scope of these old tubes.

One thing to note when buying them is that, due to age or poor build quality, a "batch" purchased can vary in brightness. This batch has a really dim tube and a really bright one. The others are mostly similar in brightness. Good job I have more of these already, so mixing and matching should sort that out.

I personally think I have spaced the displays too far apart from each other. I've no intention of redesigning the PCB, I'll just live with it. It still looks fine. Plus, it's my first clock project, so I'll get better from here.

### CD4060

This is the heart of the clock, quite literally. It is what generates the pulses which the whole clock is based upon. It's a 14 stage ripple counter. Which isn't as complex as it sounds.

The ripple counter divides the frequency of an oscillator by up to 14 times. This is not to say that the frequency is divided by 14 but to say divided by 2, 14 times. With a 32.768KHz crystal, that divides as follows.

32.768KHz
16.384KHz
8.192KHz
4.096KHz
2.048KHz
1.024KHz
512Hz
256Hz
128Hz
64Hz
32Hz
16Hz
8Hz
4Hz
2Hz

On the 14th divide, magically there is a steady 2Hz frequency. Not what you would expect for a clock. 1Hz would make sense, 1 pulse per second. This is sorted later.

### CD4511

The 4511 is responsible for taking the output of the counters, then transforming it into a set of signals to light the correct parts of the displays. This is to display human readable digits.

The outputs are driven high, to 5v, when active. This lines up with common cathode displays. The same type as all vacuum tube displays. The displays require a positive voltage on the segments to attract electrons, these electrons hitting the segments cause the phosphor on the segments to glow.

### CD4013

This is responsible for getting the steady 1Hz signal for counting seconds. By connecting the inverted output to the data input, the chip provides its own data to latch. The latch is triggered by the 2Hz signal.

Each rising edge of the 2Hz signal, triggers the latching of data in the flip flop. As the data input is always the opposite of the main output, two pulses are second are needed to change the output state at a 1Hz frequency. One rising edge to change the output level high and one rising edge to lower the output level.

The inverted output is also used for the colon separator. The regular output could also be used but the colons normally illuminate when the seconds digit changes. Otherwise the colons switch off when the digit changes.

### CD4518

These chips are backbone of the clock. They take the pulses generated by the flip flop to count them. Each count is pushed to the output pins in binary format. They automatically know to reset to 0 after counting up from 9. The output is 4-bit wide, which can count up to 16 but in this case, not all combinations are used.

0000 = 0
0001 = 1 
0010 = 2
…
1000 = 8
1001 = 9

The 1Hz clock input to these is connected to the enable pin. Looking at the datasheet, there is a dedicated clock pin. The name is a little misleading but, it really is an enable pin. The clock input is gated with enable in AND logic. The clock input is inverted before this. The enable must be held high to allow clock pulses through. Depending on when the counter needs to count, these pins can be swapped. When used as described, the counter changes to the next number when the clock input goes high. In this setup, to carry to the next counter when the first counter loops back to 0, it would need extra logic. To get around this, the enable pin is used as a clock input. This way the counter increments when the clock signal goes low. This means when the binary 8 output goes low, if connected to the enable of the next counter, it will count.

The problem comes when resetting the tens digit. Time works on 60 seconds to a minute and 60 minutes to an hour. The tens digit needs to be reset when changing to 60. This does require external logic and the use of the reset pin of the counter. To increment the hour digit, the same principle can be used, only this time using the extra logic to trigger the count for the next digit. Simply, when binary 60 is reached, the extra logic outputs a high level signal, resetting the counter to 0 from 6. When this happens, the logic output goes low and triggers the count for the next counter.

### CD4081

This is simply the quad two channel AND gate chip. Three of the four gates are used to reset the tens digit of each hour, minute and second when they reach 60 or 24. They also are used as the counter for carrying to the next digit. Eg, 60 seconds triggers a count of 1 min, 60 mins triggers a count of 1 hour.

### 2N2981A

These are for level shifting. Used in this way called high side switching. This is needed due to the display tubes needing 28v for operation. The logic circuits used here all run at 5v, not 28v. You can't simply control 28v with 5v. It is quite simple to control, but not in the sense that you can use a single transistor to switch on and off 28v. This chip shrinks down all the components that would be needed otherwise. A simple discrete logic alternative would require 2 transistors per digit segment, 14 transistors per tube and a total of 84 transistors. Without counting the resistors required that would be in the hundreds.

These are a useful little guy. It's a shame they have been out of production for years now, as there isn't really a good replacement. There are some SMD alternatives, but in the DIP world there just isn't. The 2N2904 is something I considered, but it requires an extra transistor for each of the outputs, making the project even larger.

### MC34063

In the effort to keep the project looking mostly like it came from 1985, the 34063 is an easy to use buck/boost supply controller that was around back then.

This project pushes the chip to its limit. It has an output switching current of 1.5A while this project requires 1.37A for the 28v rail. The actual current needed is more like 100mA. The switching current is just the peak current through the inductor, which if the input voltage is being boosted a lot, will be a high current.

The display tubes require a low voltage for the directly heated cathode. This is also created from the use of the 34063 in a buck converter topology. The heater voltage can easily be created with a series resistor from the 5v supply. This has the problem of the voltage changing directly in proportion with the supply. The USB voltage can vary a large amount, this would cause the display brightness to vary a lot. It could even damage the cathode. Should the voltage drift too high. Using the buck converter for powering the cathodes, allows the 34063 to adjust the output voltage regardless of the input voltage.
