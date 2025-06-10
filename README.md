# Netmotive
A vehicle communications platform designed to connect to the vehicle CAN bus, and create a network of clients, sensors, and control surfaces to add additional functionality.

The platform centers around the use of various SBCs and MCUs which either act as a server or various clients, either reading vehicle messages, controling vehicle surfaces, measuring sensor data, or any other automotive application. The interconnect between all the components is done via socket programming over the TCP/IP stack.

The server-side is based on a Linux SBC (Raspberry Pi 5) which is directed to connect to all the client modules, act as a network router, receive/send CAN messages through its client CAN gateway, command and control control surface MCUs based on sensor/CAN data, and provide processor-heavy functionality.

The clients are broken up into various modules (with the current implementation, more to come):

CAN Gateway - Raspberry Pi Zero 2W:
This client is specifically responsible for sniffing the CAN bus and injecting messages whenever needed. The device uses the Linux SocketCAN driver. In the testing implementation, the Waveshare 2-CH CAN HAT+ was used to communicate on the test vehicle's two CAN busses (HS @ 500kbit and MS @ 125kbit). The I/O of this client is done over the TCP/IP stack with the server and the candump can be stored locally and be overwritten per a set interval (per trip, per week, etc.)

Adaptive Strut Control - Arduino Nano ESP32:
This client is responsible for controlling the adaptive struts on the vehicle, which rely on a 400Hz PWM signal to power a solenoid in each strut, adjusting its damping between firm and soft. When unpowered, the strut is in a firm setting. To adjust the strut to soft/comfort, a 1.3A push current is applied to the solenoid for 75ms and then a 0.5A holding current is needed to retain that state, otherwise the strut will return to the firm state. The MCU controls this functionality per strut based on either a network command from the server or based on sensor input.

More clients to come in the future (maybe security, cameras, etc.?)
