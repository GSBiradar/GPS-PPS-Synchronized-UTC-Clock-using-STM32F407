# GPS-PPS-Synchronized-UTC-Clock-using-STM32F407
Accurate timekeeping is critical in embedded systems, but microcontrollers suffer from clock drift. This project uses an STM32F407 with a GPS module, combining NMEA UTC data (UART) and a precise PPS signal (1 Hz) to create a synchronized clock with millisecond-level accuracy.

# GPS PPS Synchronized UTC Clock using STM32F407

*(Detailed Project Description)*

---

## 1. Introduction

Accurate timekeeping is a fundamental requirement in many embedded systems such as telecommunications, satellite systems, financial systems, and scientific instrumentation. While microcontrollers can maintain time using internal clocks, they suffer from **clock drift** due to oscillator inaccuracies.

To overcome this limitation, this project implements a **GPS-disciplined clock** using the **STM32F407 microcontroller**. The system leverages two key outputs from a GPS module:

1. **NMEA Data (UART)** → Provides UTC time (hours, minutes, seconds)
2. **PPS Signal (Pulse Per Second)** → Provides an extremely accurate 1 Hz timing reference

By combining these two, the system generates a **precisely synchronized UTC clock with millisecond resolution**.

---

## 2. Objective

The main objective of this project is to design a system that:

* Reads UTC time from GPS using NMEA sentences
* Uses the PPS signal to define the **exact start of each second**
* Generates a millisecond counter (0–999 ms) using an STM32 timer
* Outputs time in the format:

```
HH:MM:SS.mmm
```

* Maintains **tight synchronization between seconds and milliseconds**

---

## 3. System Overview

The system consists of the following components:

### Hardware

* STM32F407 microcontroller (Discovery board)
* GPS module (e.g., u-blox NEO-M8N)
* UART interface for GPS communication
* PPS signal connected to external interrupt pin
* Timer peripheral for millisecond generation

### Software Modules

* GPS UART driver
* NMEA parser
* PPS interrupt handler
* Timer-based millisecond generator
* UART debug output system

---

## 4. Working Principle

### 4.1 GPS NMEA Data (Time Source)

The GPS module continuously sends NMEA sentences such as:

```
$GPGGA,123456.00,...
$GPRMC,123456.00,...
```

From these, the system extracts:

```
UTC Time = 12:34:56
```

However, NMEA data is not perfectly time-aligned due to transmission delay.

---

### 4.2 PPS Signal (Synchronization Source)

The GPS module also outputs a **PPS signal**:

* Frequency: 1 Hz
* Accuracy: within nanoseconds
* Represents: **exact start of a new second**

This signal is connected to an STM32 GPIO interrupt pin.

---

### 4.3 Timer-Based Millisecond Generation

A hardware timer (TIM2) is configured to run at a known frequency.

Configuration:

* Timer clock: 84 MHz
* Prescaler: 8399
* Resulting frequency: 10 kHz
* Tick period: 0.1 ms

Timer counts:

```
0 → 9999 = 1 second
```

Milliseconds are calculated as:

```
milliseconds = timer_count / 10
```

---

### 4.4 PPS Synchronization Mechanism

Each time a PPS pulse occurs:

1. Timer counter is reset to 0
2. Millisecond counter restarts from 0
3. This ensures alignment with GPS second boundary

Thus:

```
PPS → 00 ms
Timer → 0..999 ms
Next PPS → reset
```

---

### 4.5 Combined Time Output

Final time is constructed as:

```
GPS UTC (HH:MM:SS) + Timer milliseconds
```

Example:

```
10:32:56.345
```

---

## 5. Functional Behavior

### 5.1 Before GPS Fix

When GPS has not acquired satellites:

* Time is invalid
* System displays a waiting message
* Milliseconds still run using timer

Example output:

```
GPS TIME: Waiting for fix... 00:00:00.000
GPS TIME: Waiting for fix... 00:00:00.101
GPS TIME: Waiting for fix... 00:00:00.202
```

---

### 5.2 After GPS Fix

Once valid GPS data is received:

* UTC time is extracted
* PPS synchronization begins
* System outputs precise time

Example:

```
GPS GGA Time: 12:34:56

GPS TIME: 12:34:56.000
GPS TIME: 12:34:56.101
GPS TIME: 12:34:56.202
...
GPS TIME: 12:34:56.999
GPS TIME: 12:34:57.000
```

---

## 6. Software Architecture

### 6.1 gps_pps.c

Handles:

* UART interrupt-based reception of GPS data
* NMEA sentence buffering and parsing
* Extraction of UTC time
* PPS interrupt handling
* Timer-based millisecond calculation

---

### 6.2 main.c

Handles:

* Peripheral initialization (UART, GPIO, Timer)
* Calling GPS processing logic
* Formatting and printing time output

---

## 7. Key Features

* Real-time UTC clock synchronized with GPS
* Millisecond-level resolution
* Hardware-based timing (no software delay loops)
* Interrupt-driven architecture
* Robust handling of GPS no-fix condition

---

## 8. Advantages of PPS Synchronization

Without PPS:

* MCU clock drift accumulates over time
* Time becomes inaccurate

With PPS:

* Clock is corrected every second
* No long-term drift
* High precision maintained

---

## 9. Accuracy Analysis

| Method                | Accuracy |
| --------------------- | -------- |
| Internal MCU clock    | ±50 ppm  |
| With PPS sync         | ±1 ms    |
| With advanced capture | ±100 ns  |

---

## 10. Applications

This system can be used in:

* GPS wall clocks
* Industrial time synchronization systems
* Data logging systems
* IoT timestamping devices
* Telecom timing units
* Satellite ground systems

---

## 11. Limitations

* Requires GPS signal visibility
* Initial lock time may take 30–60 seconds
* Accuracy limited by timer resolution (0.1 ms)

---

## 12. Future Enhancements

* Convert UTC to local time zone
* Add date display
* Display on LCD / TouchGFX UI
* Use Timer Input Capture for higher precision
* Store timestamps on SD card
* Add RTC backup system

---

## 13. Conclusion

This project demonstrates how to build a **GPS-disciplined embedded clock** using STM32. By combining GPS NMEA data with the PPS signal and a hardware timer, the system achieves **precise, drift-free timekeeping**.

It is a strong example of:

* Embedded systems design
* Real-time synchronization
* Hardware-software integration
* Interrupt-driven architecture

and forms the foundation for **professional timing systems used in industry**.

---
