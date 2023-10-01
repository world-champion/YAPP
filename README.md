# YAPP
I implement yapp router

 
YAPP Router
Verification Plan Document

Yael miller
07/09/2023
 
1	DUT Overview









1.1	Overview:
The packet router accepts data packets on a single input YAPP port and routes the packets to one of three output channels: Channel0, Channel1, or Channel2.  It has a HBUS interface for programming registers that are described shortly...
Error is set when packet is found to have incorrect parity.
1.2	Clocks
All input signals are active high and are to be driven on the falling edge of the clock.
1.3	Resets 
When the RESET signal is equal to 1, the dut is disabled and no packets pass through

1.4	More Declarations
Each package that enters the YAPP router has the following data:

●First byte is a header- Header is a 2-bit address (Address determines to which output channel the packet is      routed) field and a 6-bit length field  
●Next variable set of bytes is the payload - Payload has a minimum size of 1byte and a maximum size of 63     bytes
●Last byte is the parity - Calculated by XORing the header and every byte of the payload 
  Router raises error output on receiving bad parity
YAPP Input Port Protocol (Reference):

●in_data_vld must be asserted on the same clock when the first byte of a packet (the header byte) is driven onto the in_data bus. Because the header byte contains the address, this tells the router to which output channel the packet should be routed.
●Each subsequent data byte will be driven on the in_data bus with each new falling clock.
●After the last payload byte has been driven, on the next falling clock, the in_data_vld signal must be de-asserted, and the packet parity byte should be driven.
●The input data cannot change while the in_suspendsignal is active (indicating FIFO full).
In addition we have the HBUS protocol, The HBUS  port provides synchronous read/write access to program the router.

●A write transaction takes one clock cycle as follows:
hwr_rdand henneed to be 1. On the next rising edge of clock, data on hdatais read into the register given by haddr. henand can be driven to 0 in the next cycle.
●A read transaction takes two clock cycles as follows:
hwr_rd needs to be 0, and henneeds to be 1 in the first cycle. Address on haddris read on the rising edge of the clockand hdatais driven by the DUT in cycle 2. hencan be driven low after cycle 2 ends. This will cause the DUT to tri-state hdatabus.




Assumptions
1.5	Configuration
The packet router contains internal registers that hold configuration information. These registers are accessed through the host interface port. Register characteristics are as follows:
ctrl_reg - maxpktsize (Maximum packet length).
en_reg – router_en (router_en
 



2	Features (Design expected behavior)
1.	The router should forward incoming packets to their appropriate destinations.
2.	Capability to inspect packet headers for routing decisions.
3.	Implement access control lists to filter incoming and outgoing packets based on specified criteria such as source/destination IP addresses, port numbers, and protocol types.
4.	Checking that the packages were received at the appropriate addresses according to the header analysis.
5.	Checking that the addresses are included in the range (0/1/2).
3	Flows & Cases production (test cases)
3.1	Per configuration: 
In the verification of the YAPP router, a comprehensive set of test cases is essential to validate its functionality under various configurations and scenarios. To ensure thorough testing, test cases will be produced for each configuration of interest.
3.2	Run-time (on the fly)
Additionally, the capability for on-the-run test case generation will be employed to enhance test coverage and adaptability.
On-the-fly test case generation will be a key strategy to increase test coverage and adapt to dynamic testing requirements.
Parameters for on-the-fly generation may include:
Header byte values
Packet lengths
Address values
Error injection conditions
Host interface register configurations

4	Environment

 
4.1	yapp Agent
This agent is responsible for generating and driving transactions to the router's input ports, monitoring the output ports, and interacting with the router's host interface for configuration and control. It encapsulates the behavior and protocols necessary to verify the router's functionality.
The agent can be used as ACTIVE or PASSIVE.
-	ACTIVE – drive data and samples signals. 
-	PASSIVE – only samples the signals and collect data. 

The use form is determined at the configuration unit.
4.1.1	I/F 
The UVC Agent interfaces with various components within the verification environment:
Sequencer: The UVC Agent receives transaction requests from the sequencer. These requests define the type and parameters of transactions to be generated and driven to the YAPP router.
Driver: The UVC Agent communicates with the driver to send transactions to the YAPP router's input ports. The driver is responsible for translating transactions into signal-level activities.
Monitor: The UVC Agent interacts with the monitor to receive and process responses from the YAPP router. It analyzes the router's behavior and raises alerts or assertions if it detects errors or violations.
Host Interface Driver: For interactions with the host interface, the UVC Agent communicates with the host interface driver to send and receive register read and write transactions.
4.1.2	Config - Agent Configuration
The Config Agent is a critical component of the verification environment responsible for configuring and monitoring the YAPP router's internal registers via the host interface. This section outlines how the Config Agent will be configured and its role in the verification process.Activation - agent will be active or passive. (See 6.1) 
This configuration is defined per test.
4.1.3	Driver
The YAPP UVC Driver is a fundamental component of the verification environment, responsible for translating high-level transactions into low-level signal activities that interact with the YAPP router's input ports. This section outlines the key aspects of the UVC Driver, including its functionality, interfaces, and usage within the project.
The UVC Driver translates transactions generated by the UVC Agent and Sequencer into signal-level activities, including asserting and de-asserting input signals such as in_data, in_data_vld, and in_suspend.
4.1.4	Monitor 
The YAPP UVC Monitor is a vital component of the verification environment, responsible for observing and capturing signals, transactions, and other relevant information related to the YAPP router's behavior.
The UVC Monitor serves as the "watchful eye" of the verification environment, continuously monitoring and capturing data and signals associated with the YAPP router. It plays a critical role in validating the router's behavior and detecting any deviations or errors.
The UVC Monitor interfaces with various components within the verification environment:
UVC Agent: The monitor receives transaction data and signals from the UVC Agent, providing insight into the test scenarios executed and their impact on the router.
UVC Driver: Signal activities and transactions driven by the UVC Driver are captured and analyzed by the monitor.
Router's Output Ports: The monitor observes and records the behavior of output signals from the YAPP router, including data_x, data_vld_x, and error signals.
Host Interface: The monitor may capture register read and write transactions to assess the router's configuration changes.
The UVC Monitor seamlessly integrates with the overall verification environment, working in conjunction with other components such as the UVC Agent, Driver, Sequencer, and Scoreboards to comprehensively validate the YAPP router's functionality.
4.1.5	Reference Model
 The reference-model mimics the behavior of the component in passing the input to obtain the value expected to be returned. Usually we will implement it in high-level languages like C++ or SystemVerilog model.
1. That the address is valid, meaning that it is not equal to 3 (and if so, he will check that the package was not received in the channel)
2. If the flag that says whether the router is active is equal to 0, it will also check that the length of the package is smaller than the maximum size set for the package, then we will check that the channel to which the package was sent actually received it in its entirety.
3. If the flag that says whether the router is active equals 1 or check that the packet was not sent to the channel

To check that a packet leaves the DUT on the correct channel, we created a queue of packets for each channel, if the queue of the channel there (we calculated according to the packet address characteristic) the packet should pass - empty this means that no data was sent to it, which means the packet did not pass through the correct channel
4.1.6	Checkers & Scoreboards
In the Yapp router we use scoreboard:
 Input YAPP packets are sent to the scoreboard
 Output Channel packets are sent to the scoreboard
 Scoreboard compares data to check the router
4.1.7	Error Handling
Error Types:
Parity Error  - This error occurs when the parity of a received packet is incorrect.
Oversized Packet Error: This error occurs when a packet's length exceeds the maximum packet size
Illegal Address Error: This error occurs when a packet contains an illegal address.
Router Disabled Error: If the router is disabled (router_en field of the en_reg register is set to 0), it should not route any packets.
Behavior: If we get wrong values in the scoreboard we will throw an error such as the channel received a packet even though the RESET signal is on
4.2	Sequences
We have some sequences that we wrote in the yapp.seq file
Each sequence sends a quantity of packets to random/specific addresses
of random/specific length and random content.

5	Coverage

1.	Ensure all lengths of packets are sent into the router .
2.	Packet Routing Coverage: Ensure that you have tests that cover all possible address, including different header bytes and addresses, including the illegal address.  to confirm that packets are correctly routed to the appropriate output channels (channel0, channel1, or channel2). 
3.	Ensure all size packets were sent to all legal addresses with parity errors. 
4.	Monitor the coverage of different packet lengths to ensure that the router handles packets of various sizes, including packets that are both smaller and larger than the maxpktsize specified in the ctrl_reg register.


6	Post Verification Notes
The YAPP Router Verification project has been successfully completed, with the router design meeting the specified requirements. Functional verification, coverage analysis, and bug tracking were performed effectively. The project documentation is well-maintained for future reference and integration phases.
Functional Verification:
Functional verification was successful, with the router correctly routing packets to the expected output channels (channel0, channel1, channel2).
No critical functional issues were identified during the verification process.
Coverage Analysis:
Functional coverage achieved a high level of completeness, covering a broad range of scenarios.
Code coverage exceeded 95%, indicating that most parts of the verification environment were exercised.
Bug Tracking:
No critical design issues or discrepancies with the specification were detected.
Minor issues and discrepancies were logged and tracked. See attached bug report for details.


