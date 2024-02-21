# Arduino_Button_to_UDP

This repo features the source for the microcontroller button and the desktop based interface software.


## Microcontroller Software

Name: single_button_to_serial

This is a platformio project written using the arduino framework. Messages are sent to and from the device by serial.
The devices presents itself over the USB protocol as a virtual serial device. Within windows this can be found at a typical location of “COM1”, within Linux it can be commonly found named “/dev/ttyUSB0”.

## Desktop software

Name: single_button_desktop_interface

This is a c program that connects to the device over serial. It sets up the device and handles all quality of service parts of the device. If a button event is triggered, the application will transmit a 1 or 0 over UDP in client mode. 
The software requires a UDP server to currectly function.

## API Documentation
The payload sent and received from the device is an 8-bit message. This comprises of a 4 bit command and 4 bit parameter. The command is bit shifted 4 bits and the parameter is OR’ed to the command message to create a 8-bit payload.

##### Example:
Message = (control_message << 4) | parameter

##### Quality of Service (QoS)
The device features 2 modes for quality of service, mode 1 and mode 2. In mode 1, when sampling the button, the device will send a message to the client at most once. In mode 2 the device will send the message at least once. Mode 2 requires the client to acknowledge that the button message has been received.
Due to the read-only nature of the processor, the quality of service must be set prior to starting sampling. The device will reset after each power cycle.


### Command List
All returned messages will include the initial 4-bit command plus a 4-bit return value.
Note 1: When button command initiated, the device will send a message without needing a request. The client software must be ready to receive messages the point the sampling is activated. When sampling activated, only the sampling status will be returned. No other commands will be acknowledged until the sampling has been stopped.

#### Info Command

|Command|Parameter|Return|Description|
|-------|---------|------|-----------|
|0x01   |0x01     |0x01  |Ready Message. This message should be sent initially to check that the device is connected. The returned value has no relevance and is an indicator that the device is online.|
|       |0x02     |0x01 = True  0x02 = False|Sampling Status. This will return true or false on status of sampling running.|
|       |0x03     |0x01 = 1  0x02 = 2|QoS. Get the quality of service mode. Return with active mode.|


#### Control Commands
|Command|Parameter|Return        |Description|
|-------|---------|--------------|-----------|
|0x02   |0x01     |0x01 = True <br>0x02 = False|Start Sampling.  <br>This starts the detection of the button press.  See Note 1 above for guidelines on usage. Will return success  of starting sampling.|
|       |0x02     |0x01 = True <br>0x02 = False|Stop Sampling.  <br>Will return success of stopping sampling.|
|       |0x03     |0x01 = 1 <br>0x02 = 2|Set quality of service mode 1.  <br>Return with the QoS mode stored|
|       |0x04     |0x01 = 1 <br>0x02 = 2|Set quality of service mode 2.  <br>Return with the QoS mode stored|

#### Button Commands
|Command|Parameter|Return        |Description|
|-------|---------|--------------|-----------|
|0x03   | 0x01    |0x01          |Confirmation of message received.  <br>This is only required when QoS mode 2 is active.
| |  |0x00| Message received when a button release is detected.|
| |  |0x02| Message received when a button press is detected.|
