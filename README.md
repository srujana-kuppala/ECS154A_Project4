# Project 4 README

Student 1: Srujana Kuppala (921005375)
Student 2: Lucia Zeng (920689462)

## Project Status
Arbiter, Observer, and SRO (I thinK?) are completed, with varying progress on the other modules. 

## Known Issues
Currently, the SRI is almost complete but not quite complete (SRI doesn't start over, counter is also not quite right), most of the logic should be in the frame queue but not all of it, and the rest only has it partially complete.

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

### Prompt 
Create a bus controller that sends and recieves 12- bit frames of data.

### Response 
For the VPWBObserver- there is an error if 3 1's have been recieved in a row (data of which can be held in D flip flops (RX1-3, and resets when the module is not enabled)) or when it's already in an error state and that it's currently not idle. If all RX bits are 0, then the bus is currently idle. There's only a new bit if only one of first bit is true or if the incoming bit (RX) is true (bc if both are true, it's just a long bit, not a new one).

For the VPWBArbiter, use the oberserver to check for idleness. Short VS Long bit: as long as it's not an even bit and the incoming data bit being written to isn't one, it is a short bit that is being recieved. Otherwise it is a long bit being transmitted. The bit is complete when both the module is idle and the bus is idle, or bit2 is true and no loss of arbitrition, or bit1 is 1 and it's a short bit and arbitrition has not been lost.

VPWBSRO has a counter that counts down every time either load or reload is indicated, to restart the counter and the shift register. There's a mux in between the the para_in and the register holding the previous frame (which is selected when reload is true, otherwise the para_in is shifted in). If all of the bits left are 0, then the shift register is empty(use a NOR gate). The constant 1 should be put at the 12th bit to indicate the EOF.

For VPWBSRI, a counter was added, which increments every time a new bit is added. When the frame is full(the newbit counter should be full), that should be one of 3 criteria(read and enable must be on as well) for the frame to start reading to para out. New Bit indicates when the shift register should start moving bits in. 

For the VPWBFrameQueue, track the number of times something is enqueued with a counter, and have it decriment if dequeue is on so that the counter maintains the number of frames in the queue. This can also be used to check if it's full or empty, can use an AND gate for both;one without negation to check for fullness and one that has both inputs negated to check for emptiness. 4 registers are used for each frame queue(4 max), and there is a mux in between to select between frame in, or recieve the previous frame which depends on location (0,1,2,3 right to left on the circuit. if it's not at the position, then just recieve incoming frame).

VPWBRX- didn't figure out exactly what to do with the care or addr bits, mostly connected some of the pieces together via label.

VPWBTX is idle when the transmitter is idle or when the FIFO is empty-- I simply used an AND gate on all idle, and empty outputs to indicate idleness. I put a counter to check for the even bit is being written to, which reset whenever a new frame was being written.

For the VPWBController, since the data is being written to when CS and W is low, and the address is 1 and that can be indicated on the frame transmitter, and the data being read when CS and O is low and can be indicated with an AND gate. Then the data that gets transmitted out depends on what address is being recieved, (couldn't figure out how exactly to deal with the 0 address, but as of now it outputs whatever is in the datain register if it's 0, and W and CS are low) but whenever the address is 1, it just output whatever the reciever outputs. 

### Changes 
Needed to add a counter whenever enequeue is on for frame queue module.
Needed to add muxes in between each register for the frame queue.
