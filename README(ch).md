# Nous_dm-mc02_FEETECH_6-axis_robotic_arm

This repository contains the firmware for a FEETECH six-axis robotic arm based on the STM32H723VGT6. It is mainly used to drive FEETECH STS3215 serial servos over a serial bus, supporting joint motion control, servo scanning, and basic debugging.

## Project Overview

The project provides the following features:

- Communication with the FEETECH servo bus through USART10
- Support for the FEETECH STS3215 serial servo protocol
- Debug output through UART7 to a serial terminal or PC
- Servo scanning, communication testing, and status debugging after power-up
- A basic control foundation for a 6-DOF robotic arm

## Hardware Platform

- Main MCU: STM32H723VGT6
- Servo model: FEETECH STS3215
- Development tools: Keil MDK / VS Code
- Programmer / debugger: ST-Link
- Firmware library: STM32Cube HAL

## Wiring

### 1. Servo bus wiring (USART10)

The robotic arm servos communicate with the MCU through USART10:

- PE3 (USART10_TX) -> servo RX
- PE2 (USART10_RX) <- servo TX

### 2. Debug serial wiring (UART7)

Debug information is output through UART7:

- PE8 (UART7_TX) -> USB-to-serial module RX
- PE7 (UART7_RX) <- USB-to-serial module TX

### 3. Wiring notes

- The MCU board and the servos must share a common ground
- Use a separate and stable power supply for the servos
- Do not power the servos directly from MCU pins
- If you perform a PE3 and PE2 loopback test, disconnect the servos first to avoid bus conflicts
- If communication fails, check the following first:
  - TX and RX wiring
  - Common ground
  - Servo power supply
  - Serial settings

## Features

### 1. Servo communication

This project supports FEETECH STS3215 serial servo communication for:

- Writing target position
- Setting speed
- Setting acceleration
- Reading servo position, speed, load, voltage, temperature, current, and other status data

### 2. Servo scanning

After startup, the firmware can scan the servo bus to detect which servos are online and print their IDs.

By default, it scans a range of IDs to help confirm that each joint of the arm is connected correctly.

### 3. Debug output

The firmware prints logs through UART7, which is useful for:

- Checking startup status
- Verifying communication success
- Debugging servo connections
- Inspecting scan results and error messages

### 4. Basic robotic arm control

This project can serve as a base for 6-axis robotic arm control and can be extended later for:

- Joint angle control
- Multi-servo synchronized motion
- Arm motion sequencing
- Homing and pose control

## STS3215 Code Explanation

The STS3215-related code in this project is organized into four layers: communication, control, status, and protocol definitions. They correspond to different files under Device/STS3215/.

### 1. Communication Layer: STS3215_comm.c / STS3215_comm.h

This layer handles byte-level communication with the servo bus and wraps the HAL UART interfaces into functions that the STS3215 protocol can use.

- ftUart_Send(): sends servo command data
- ftUart_Read(): receives servo response data
- ftBus_Delay(): bus switching delay to keep the transmit/receive timing correct
- readSCS() / writeSCS() / writeByteSCS(): low-level protocol read/write interfaces
- rFlushSCS() / wFlushSCS(): clears the receive and transmit buffers

This layer does not care about the actual motion being controlled. Its only job is to make sure the data is sent and received correctly.

### 2. Protocol and Control Layer: STS3215_control.c / STS3215_control.h

This layer converts control requests such as target position, speed, acceleration, and mode switching into commands that the STS3215 servo can understand.

- STS3215_Init(): initializes the servo protocol environment and performs a servo scan
- Find_STS3215(): scans servo IDs on the bus and prints the devices that respond
- STS3215_SetPosEx(): standard position control for moving a single servo directly
- STS3215_SetPosEx_Reg(): deferred position write for batching commands before execution
- STS3215_SetPosEx_Sync(): synchronized position write for multiple servos, useful for robotic arm motion
- STS3215_WheelMode(): switches to wheel mode
- STS3215_SetSpeed_WheelMode(): controls speed in wheel mode
- STS3215_CalibrationOfs(): center offset calibration
- STS3215_unLockEPROMEx() / STS3215_LockEPROMEx(): unlocks or locks the servo EEPROM area

In practice, this layer first packs position, speed, and acceleration into the byte arrays required by the servo protocol, then sends the command through the lower-level write interface.

### 3. Status Layer: STS3215_status.c / STS3215_status.h

This layer reads servo status for debugging, tuning, and fault diagnosis.

- STS3215_Get_AllPos_Status(): reads multiple status values
- STS3215_ReadStatus(): reads servo status by ID
- FeedBack(): reads and processes feedback data
- ReadPos() / ReadSpeed() / ReadLoad() / ReadVoltage() / ReadTemper() / ReadMove() / ReadCurrent(): read position, speed, load, voltage, temperature, motion state, and current

This layer is useful when checking whether the servo really executed a command, or when diagnosing power, load, or temperature problems.

### 4. Memory Table Definitions: STS3215_control.h / STS3215_status.h

These header files define the STS3215 register addresses, such as:

- STS3215_ID: servo ID
- STS3215_BAUD_RATE: baud rate
- STS3215_GOAL_POSITION_L/H: target position
- STS3215_PRESENT_POSITION_L/H: current position
- STS3215_PRESENT_SPEED_L/H: current speed
- STS3215_PRESENT_LOAD_L/H: current load
- STS3215_PRESENT_VOLTAGE: current voltage
- STS3215_PRESENT_TEMPERATURE: current temperature

You can think of these definitions as the servo's internal register address map. The upper-level code only needs to use these names instead of remembering the raw numeric values.

## Communication Settings

- Servo bus: USART10
- Debug serial: UART7
- Servo communication parameters: 115200, 8N1
- Output: serial logs plus servo status information

## Usage

1. Make sure the hardware wiring is correct
2. Power on the MCU board and the servos
3. Download the firmware to the development board
4. Open the debug serial port and view the logs
5. Watch the servo scan and communication output to confirm normal operation

## Common Issues

### 1. Servos do not respond

- Check whether the servos are powered
- Check whether PE3 and PE2 are wired correctly
- Check whether the servos and MCU share a common ground
- Check whether the servo ID and protocol are correct

### 2. Garbled serial output

- Check whether the UART7 baud rate is correct
- Check whether the serial cable is wired correctly
- Check whether the USB-to-serial module is working properly

### 3. Servo communication fails

- Check whether the USART10 configuration is correct
- Check whether the bus speed matches the firmware settings
- Check whether the servo model is FEETECH STS3215
- Check whether multiple devices are conflicting on the bus

## Project Structure

- Core/ - main program, startup files, and peripheral initialization
- Device/STS3215/ - FEETECH STS3215 protocol and control code
- Drivers/ - STM32 HAL low-level drivers
- FEETECH_FTServo_Routine/ - FEETECH FTServo STM32HAL reference routines
- MDK-ARM/ - Keil project files

## Notes

This repository is mainly used for low-level communication and control of the FEETECH six-axis robotic arm, and it is suitable as a starting point for robotic arm development.

---

Author: Nous3508
