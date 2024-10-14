# Overview
The readme provides an overview of the following:
1. The PCB that interfaces between the BNO055 IMU to a microcontroller and microcomputer via CAN interface
2. The function of the PCB
3. How to program the board
4. Some points to keep in mind

The Printed Circuit Board allows for IMU sensor data to be acquired and sent to the central microcomputer via I2C and CAN bus interfaces.

# Working
The data acquisition from the IMU occurs through I2C communication established between a local microcontroller (the master) and the IMU sensor (the slave). We use the Nucleo L432KC board for the microcontroller and the Adafruit BNO055 6d0f orientation sensor for the IMU. The MC board collects 4 types of data from the sensor - gyroscope, accelerometer, magnetometer and calibration. 

This is then sent to a central microcomputer via a CAN interface through two lines - CAN high and CAN low. We use the Nvidia Jetson Nano as the microcomputer. 

It is to be noted that the CAN bus must be properly terminated using a 120 Ohms CAN bus termination resistor at each end of the bus. Connector J2 ensures this (will be explained in the following sections).

CAN bus protocol works based on nodes assigned to the different communicating components, based on the order of priority, with lower index corresponding to higher priority. Jetson Nano is assigned as node 0, while the Microcontrollers are arbitrarily assigned higher node indices in increasing order. Thus, the PCB circuits correspond to nodes 1 and higher, and is meant to be installed at different locations along the wing of the ironbird. The board dimensioning, to some extent, accounts for the constraints that could be possibly posed by the ironbird.

The data obtained from the IMU sensor is transmitted by the MC into the CAN High and Low lines, and identified by the node id. Due to the large influx of data into the CAN line, they are arranged in a queue of a pre-defined size (we use a queue size of 10). The Jetson Nano progressively reads the data from the queue through an acquisition process. 

The MC, individually, is incapable of transmitting the data into the CAN lines. Hence, we use a transceiver connected to each MC to receive the data and successfully perform the transmission. We use the TCAN332 transcevier for our circuit. 

The altium schematic file provides a representation of the PCB circuit detailed above.

# Programming the board
We require two individual sets of code for using the PCB - one set for the MC and the other for the Jetson Nano. A working version of these codes are available in the flexible wing mockup repository under the folders mbed_imu_node and nvidia_code respetively. These codes are made specifically for the flexible wing prototype containing 3 PCB nodes. However, this code is generic enough to be extended to larger PCB node counts. 

The code for the MC is written in MC, and we need to perform cross-compiling within a python environment to flash the binary code to the device. This procedure is explained in the mbed_imu_mode/readme.md. The same instructions can be followed, with one exception. Keep in mind to change the board name from LPC1768 to L432KC. 

For Jetson Nano, the code contains a combination of C and C++ (depending on functionality constraints). The instructions and explaination can be found in nvidia_code/readme.mb. The master code launches either the 
calibration thread (required to be performed before any experiment) or parallelly the threads - acquisition and logging. 

# Additional points about the circuit
Datasheets:

MC - https://os.mbed.com/platforms/ST-Nucleo-L432KC/ 

Transceiver - https://www.ti.com/lit/ds/symlink/tcan332.pdf?ts=1716190918743&ref_url=https%253A%252F%252Fwww.ti.com%252Fproduct%252FTCAN332

IMU - https://cdn-shop.adafruit.com/datasheets/BST_BNO055_DS000_12.pdf  and https://learn.adafruit.com/adafruit-bno055-absolute-orientation-sensor/overview 

The MC has flexible power supply capability - through USB VBUS (5 V) or external source (3.3 V, 5 V, 7 - 12 V). On the other hand, the transceiver and IMU are restricted to 3.3 V supply. Thus, they receiver power from the 3.3 V Vin pin from the MC.

The connectors assigned J3 and J2 allow for modifying the CAN circuit based on node location or power supply scenario, using jumpers.

A console can be launch in order to see an output of the device in case of errors or misbehaviour of the sensor during its execution as well as for debugging purposes. In this case, since we are using the micro-usb port of the MC, we are providing power through the USB VBUS, and not through the external 5V power supply. The jumper J3, by default, is connected in the circuit, and allows for external power supply. In this specific scenario, when power is being supplied through USB, keep in mind to remove the jumper at J3 to avoid connection discontinuity on the overall CAN circuit. 

Since we require pull up resistors only at the ends of the CAN lines, the intermediate nodes must not be have these resistors. The connectors at J2 ensure this. Make sure that a jumper is used at this connector only for the extreme PCB nodes. For all intermediate PCBs, J2 must remain unconnected. 

We also incorporate a push button that manually resets the microcontroller (hard reset), and is equivalent to restarting the mc imu extraction code. This is especially useful for the current version of the code (in flexible wing mockup), since presently, the only way to reset the timer for the mbed code is by manually cutting off the power supply and restarting it. This is undesirable since it would reset the calibration parameters of the imu as well, and would require us to redo the tedious process of calibration for the wing mockup every time. 

We use right-angled connector pins for components that are to be soldered/placed right below the Nucleo microcontroller in the PCB. This was done in an attempt to made the PCB as small and compact as possible. Such connector pins allow to avoid neater wire connections. 
