# Project 4 README

Student 1: Srujana Kuppala (921005375)
Student 2: Lucia Zeng (920689462)

## Project Status


## Known Issues


## References


## Generative AI Use
### Prompt 1
What does a frame receiving module in a Variable Pulse Width Bus Controller do?

### Response
At its core, the frame receiving module in a Variable Pulse Width (VPW) Bus Controller listens to the bus and turns the variable-width pulses into usable digital data.

In simpler terms:

Monitors the bus line for pulses.

Measures how long each pulse lasts (pulse width).

Decides if each pulse represents a 0 or 1 based on its width.

Collects all the bits in the correct order to form a complete frame/message.

Checks for errors (wrong pulse widths, bad checksum, etc.).

Sends the finished frame to the microcontroller or CPU so the data can be used by the system.

So basically, it translates the “language” of pulses on the wire into actual digital messages the system can understand.

### Changes 1
I tried figuring out the BRX based on this. 

### Prompt 2
How can I test each module individually before putting them into the complete system?

### Response 2
Test one module at a time – give it inputs and see if the outputs are correct.

For the frame receiver, feed it fake VPW pulses and check if it gives the right bits.

For the FIFO, put in some bytes and make sure they come out in the same order.

For the transmitter, give it bits and check if it outputs the correct VPW pulses.

Use small, controlled tests – try easy examples first, then tricky edge cases (like very long frames or pulses right on the threshold).

Check for errors – make sure modules handle “empty” or “full” states correctly.

Combine gradually – once a module works, connect it to the next one and test the combined behavior before adding more.

Basically: test each part alone first, then connect them step by step.

### Changes 2
Attempted to test before using them in next modules

### Prompt 3
What is bus arbitration and why is it important in multi-device bus systems?

### Answers 3
1. What is Bus Arbitration?

Bus arbitration is the process of deciding which device gets to use the shared communication bus when two or more devices want to send data at the same time.

In multi-device systems (like VPW, CAN, or SPI buses), only one device can drive the bus at a time.

Arbitration ensures that there are no collisions or data corruption when multiple devices try to communicate simultaneously.

2. Why It’s Important

Prevents Data Collisions:
Without arbitration, two devices could send data at the same time, causing errors on the bus.

Ensures Fair Access:
Each device gets a chance to use the bus according to a priority scheme.

Maintains System Reliability:
Proper arbitration avoids lost or corrupted messages, which is crucial in systems like cars or industrial controllers.

3. How It Works (Simple Example)

Priority-Based: Each device has a priority. The device with the highest priority wins the bus.

First-Come-First-Served: Devices wait until the bus is free, then start transmitting.

Collision Detection (like in CAN): Devices monitor the bus while transmitting. If a device loses arbitration, it stops and retries later.

### Changes 3
Was able to complete Arbiter with a better understanding of what it does. 

### Prompt 4
How do I connect all submodules with proper enable signals and data flow?

### Response 4
Decide the data flow:

Bus → Frame Receiver → FIFO → CPU or Frame Transmitter → Bus

Use enable signals to control modules:

Frame Receiver: enable when a new frame starts on the bus

FIFO input: enable when receiver has data and FIFO isn’t full

FIFO output: enable when CPU or transmitter is ready

Frame Transmitter: enable when FIFO has data and bus is free

Connect modules in order:

Receiver output → FIFO input → CPU/transmitter → bus

Check status flags:

FIFO full/empty, bus idle/busy, frame complete → these signals control the enables so nothing conflicts

### Changes 4
Understood how the bus controller flowed better

## Gen Prompt 1


### Prompt 
Create a controller that 

### Response 

### Changes 
