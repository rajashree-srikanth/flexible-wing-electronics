# Overview
This repository contains the Altium blueprints for the circuit and PCB that allows for transmitting signal from RC receiver to a USB serial port. The signal received from most RC receivers is SBus, which is an inverted UART signal. Serial ports, on the other hand, cannot read/recognize such signals, and require a dedicated circuit to un-invert this signal. 

This PCB circuit performs this specific function. 

For more information, refer https://docs.px4.io/main/en/tutorials/linux_sbus.html. 