# Netmotive
A vehicle communications platform designed to connect to the vehicle CAN bus, and create a network of clients, sensors, and control surfaces to add additional functionality.

The platform centers around the use of various SBCs and MCUs which either act as a server or various clients, either reading vehicle messages, controling vehicle surfaces, measuring sensor data, or any other automotive application. The interconnect between all the components is done via socket programming over the TCP/IP stack.

## Protocol
The protocol used is for a standardized format of network data transmission. Sent and received as a bytestream in network byte order. The port at which the service typically operates is TCP/585. The total stream size is fixed, at 32 bytes, which allows for the transmission of, up to, a full CAN2.0B message.

    {uint8:recipient_id}{uint8:sender_id}{uint8:msg_type}{char[16]:msg}{char[12]:extd}{uint8:flags}

The protocol is bi-directional as the command and response streams are identical in structure. The breakdown of the message is as follows:

### recipient_id: 1 byte / 8 bits
The ID of the node receiving the message. This ID determines whether or not the rest of the message will be processed or thrown out, depending on if the message was directed to the node/broadcast. The only predetermined ID is the server, which always will have an ID of 1. A recipient_id of 0 will be treated as a broadcast message from the node. An unrecognized ID will be treated as a garbage message and won't be processed.

### sender_id: 1 byte / 8 bits
The ID of the node sending the message. Again, the only predetermined ID is the server, which always will have an ID of 1. A sender_id of 0 indicates that the node doesn't have an ID assigned yet, the server will assign an ID to the node after identifying it. An unrecognized ID will be treated as a garbage message and it will halt futher processing of the message.

### msg_type: 1 byte / 8 bits
The type identifier allows the node to determine the type of message it will receive from a specified, common list.  
**Generic (0-9):**   
0 - Generic Command  
1 - Generic Response  
2 - Shutdown Command from Server  
3 - Node Shutdown Broadcast  
4 - Node Identification to Server (ID Request)  
5 - Server ID Assignment  
6 - Node Online Broadcast (After ID Assigned)  
7 - Generic Error Response  
8 - Server Downtime Broadcast  
9 - Server Online Broadcast  

**General Traffic (10-99):**  
10 - HS CAN Traffic  
15 - MS CAN Traffic  
20 - Vehicle Sensor Traffic  
21 - Node Sensor Traffic  

**Specific Commands/Responses (100-199):**  
100 - Linux Command to Execute on Recipient  
101 - Linux Command Response to Sender  
120 - Are you alive? Message  
121 - Alive Response  
122 - Sleep Command  
123 - Sleep Acknowledge Response  
124 - Wake-up Command  
125 - Awake Response/Broadcast  
130 - Request IP Address of Recipient  
131 - IP Address Response to Sender  
132 - Request MAC Address of Recipient  
133 - MAC Address Response to Sender  
150 - Sending File to Recipient Alert  
151 - Acknowledge File Transfer from Sender  
152 - Verify File Receipt Command  
153 - Successful File Receipt Response  
154 - File Receipt Error, Resend  
155 - File Receipt Error, Don't Resend  

**Currently Unused (200-255)**  

### msg: 16 bytes / 128 bits
The data/message being sent from sender to recipient, identified by the msg_type parameter.

### extd: 12 bytes / 96 bits
An extended message block allowing for a versitile use, either for arguments to a command in the message block, an extension to the message to allow for a longer message, or any other purpose defined in the node utilizing it.

### flags: 1 byte / 8 bits
The flags identifier allows for the message processor of the node to make specific adjustments depending on the flag. They are node specific and can be 0 if unused.

## Server - Raspberry Pi 5 
The server-side is based on a Linux SBC (Raspberry Pi 5) which is directed to connect to all the client modules, act as a network router, receive/send CAN messages through its client CAN gateway, command and control control surface MCUs based on sensor/CAN data, and provide processor-heavy functionality.

The clients are broken up into various modules (with the current implementation, more to come):

## CAN Gateway - Raspberry Pi Zero 2W:
This client is specifically responsible for sniffing the CAN bus and injecting messages whenever needed. The device uses the Linux SocketCAN driver. In the testing implementation, the Waveshare 2-CH CAN HAT+ was used to communicate on the test vehicle's two CAN busses (HS @ 500kbit and MS @ 125kbit). The I/O of this client is done over the TCP/IP stack with the server and the candump can be stored locally and be overwritten per a set interval (per trip, per week, etc.)

## Adaptive Strut Control - Arduino Nano ESP32:
This client is responsible for controlling the adaptive struts on the vehicle, which rely on a 400Hz PWM signal to power a solenoid in each strut, adjusting its damping between firm and soft. When unpowered, the strut is in a firm setting. To adjust the strut to soft/comfort, a 1.3A push current is applied to the solenoid for 75ms and then a 0.5A holding current is needed to retain that state, otherwise the strut will return to the firm state. The MCU controls this functionality per strut based on either a network command from the server or based on sensor input.

More clients to come in the future (maybe security, cameras, etc.?)
