# IoMT ICU Real-Time Telemetry Monitor (FreeRTOS) 🏥

![License](https://img.shields.io/badge/License-MIT-yellow.svg)
![Kernel](https://img.shields.io/badge/Kernel-FreeRTOS-008B45?logo=freertos)
![Target](https://img.shields.io/badge/Target-ESP32-E7352C?logo=espressif)

## Real-Time Embedded Medical Monitoring System

A real-time IoMT (Internet of Medical Things) telemetry firmware prototype designed for intensive care unit (ICU) patient monitoring applications.

The project demonstrates how modern patient monitoring systems use a Real-Time Operating System (RTOS) to acquire physiological data, process clinical parameters, generate alarms, and transmit telemetry information while maintaining deterministic timing behavior.

Built on the **ESP32** platform and powered by the **FreeRTOS** kernel, the system simulates key software and hardware concepts found in commercial medical monitoring equipment.

---

## Table of Contents

- [Project Overview](#project-overview)
- [System Architecture](#system-architecture)
- [Hardware Architecture](#hardware-architecture)
- [Software Architecture](#software-architecture)
- [FreeRTOS Task Architecture](#freertos-task-architecture)
- [Inter-Task Communication](#inter-task-communication)
- [Telemetry Data Flow](#telemetry-data-flow)
- [Clinical Alarm Management](#clinical-alarm-management)
- [Code Implementation Examples](#code-implementation-examples)
- [Engineering Concepts Demonstrated](#engineering-concepts-demonstrated)
- [Repository Structure](#repository-structure)
- [Future Enhancements](#future-enhancements)
- [License](#license)

---

# Project Overview

The objective of this project is to design and implement a deterministic embedded medical monitoring platform using FreeRTOS.

The firmware is responsible for:

- Acquiring patient vital signs
- Processing incoming sensor data
- Detecting abnormal physiological conditions
- Generating clinical alerts
- Transmitting telemetry to a monitoring dashboard
- Maintaining real-time responsiveness

This project demonstrates practical implementation of:

- RTOS-based firmware architecture
- Embedded multitasking
- Medical telemetry systems
- IoMT device design
- Sensor interfacing
- Embedded networking
- Real-time scheduling
- Fault detection and recovery

---

# System Architecture

```text
                     ICU TELEMETRY MONITOR

 ┌────────────────────────────────────────────┐
 │                Patient Sensors             │
 ├────────────────────────────────────────────┤
 │ Heart Rate Sensor                          │
 │ SpO₂ Sensor                                │
 │ Temperature Sensor                         │
 │ Blood Pressure Sensor                      │
 └───────────────────┬────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────┐
         │ ESP32 + FreeRTOS Kernel   │
         └─────────────┬─────────────┘
                       │
 ┌─────────────────────┼─────────────────────┐
 ▼                     ▼                     ▼

Acquisition      Analysis Task      Telemetry Task
Task

 └──────────────► Alarm Manager ◄────────────┘
                         │
                         ▼
           ICU Clinical Dashboard
```

---

# Hardware Architecture

## Microcontroller

### ESP32

| Parameter | Value |
|------------|---------|
| CPU | Xtensa Dual Core |
| Clock Speed | Up to 240 MHz |
| SRAM | 520 KB |
| Flash | External SPI Flash |
| Wireless | Wi-Fi + Bluetooth |
| RTOS Support | FreeRTOS |

The ESP32 serves as the system controller responsible for sensor acquisition, data processing, alarm generation, and network communication.

---

## Patient Monitoring Sensors

### Heart Rate and SpO₂ Sensor

```text
MAX30102
```

Measures:

- Heart Rate (BPM)
- Blood Oxygen Saturation (SpO₂)

Interface:

```text
I2C
```

---

### Body Temperature Sensor

```text
DS18B20
```

Measures:

- Patient body temperature

Interface:

```text
1-Wire
```

---

### Blood Pressure Sensor

```text
Pressure Transducer + ADC
```

Measures:

- Systolic Pressure
- Diastolic Pressure
- Mean Arterial Pressure

Interface:

```text
Analog Input (ADC)
```

---

## Communication Hardware

Telemetry communication uses:

```text
ESP32 Wi-Fi Interface
```

Supported protocols:

- MQTT
- WebSocket
- HTTP

Used for:

- Dashboard updates
- Patient data streaming
- Alarm notifications
- Remote monitoring

---

# Software Architecture

The firmware follows a layered architecture to separate hardware-dependent operations from application logic.

```text
+-----------------------------------+
|       Clinical Application        |
+-----------------------------------+
|      Monitoring Logic Layer       |
+-----------------------------------+
|       FreeRTOS Task Layer         |
+-----------------------------------+
| Driver Layer (GPIO/I2C/SPI/UART)  |
+-----------------------------------+
|         ESP32 Hardware            |
+-----------------------------------+
```

### Application Layer

Responsible for:

- Patient monitoring workflows
- Clinical threshold configuration
- Alarm decision logic

### RTOS Layer

Responsible for:

- Task scheduling
- Context switching
- Synchronization
- CPU resource allocation

### Driver Layer

Responsible for:

- Sensor communication
- Peripheral control
- Device initialization

---

# FreeRTOS Task Architecture

## Task Matrix

| Task | Priority | Period | Stack |
|--------|----------|----------|---------|
| Acquisition | 3 | 50 ms | 2048 B |
| Analysis | 2 | 100 ms | 4096 B |
| Telemetry | 1 | 200 ms | 4096 B |
| Safety Guard | 4 | Watchdog | 1024 B |

---

## Task Flow

```text
Sensor Acquisition
        │
        ▼
 Vital Data Queue
        │
        ▼
 Clinical Analysis
        │
        ▼
   Alarm Queue
        │
        ▼
 Telemetry Service
        │
        ▼
 ICU Dashboard
```

---

## Task Responsibilities

### vTaskAcquisition

Responsibilities:

- Sensor polling
- ADC conversions
- Data formatting
- Queue transmission

Priority:

```text
HIGH (3)
```

Period:

```text
50 ms
```

---

### vTaskAnalysis

Responsibilities:

- Threshold monitoring
- Signal processing
- Trend analysis
- Alarm generation

Priority:

```text
MEDIUM (2)
```

Period:

```text
100 ms
```

---

### vTaskTelemetry

Responsibilities:

- MQTT publishing
- Dashboard updates
- Network communication

Priority:

```text
LOW (1)
```

Period:

```text
200 ms
```

---

### vTaskSafetyGuard

Responsibilities:

- Watchdog monitoring
- Task health validation
- Emergency alarm activation

Priority:

```text
HIGHEST (4)
```

---

# Inter-Task Communication

## Vital Data Queue

```c
QueueHandle_t xQueueVitals;
```

Used for communication between:

```text
Acquisition Task
        ↓
Analysis Task
```

Benefits:

- Thread-safe
- FIFO operation
- Deterministic execution

---

## Alarm Semaphore

```c
SemaphoreHandle_t xAlarmSemaphore;
```

Used when:

```text
Critical Event Detected
        ↓
Immediate Alarm Trigger
```

Benefits:

- Fast synchronization
- Low latency response

---

## I2C Bus Mutex

```c
SemaphoreHandle_t xI2CMutex;
```

Protects:

- Shared I2C bus
- Sensor communication channels

Prevents:

- Race conditions
- Priority inversion

---

# Telemetry Data Flow

```text
Patient
   │
   ▼
Sensors
   │
   ▼
ESP32 Acquisition Task
   │
   ▼
Analysis Engine
   │
   ▼
Alarm Verification
   │
   ▼
Wi-Fi Telemetry
   │
   ▼
Hospital Dashboard
```

Example telemetry packet:

```json
{
  "patient_id": "ICU102",
  "heart_rate": 78,
  "spo2": 98,
  "temperature": 36.9,
  "blood_pressure": "120/80",
  "timestamp": "2026-01-01T12:00:00Z"
}
```

---

# Clinical Alarm Management

## Threshold Table

| Parameter | Condition |
|------------|------------|
| Heart Rate | > 120 BPM |
| Heart Rate | < 50 BPM |
| SpO₂ | < 90% |
| Temperature | > 38°C |
| Temperature | < 35°C |
| MAP | < 65 mmHg |

Alarm sequence:

```text
Abnormal Reading
        │
        ▼
Threshold Violation
        │
        ▼
Alarm Semaphore Triggered
        │
        ▼
Dashboard Notification
        │
        ▼
Buzzer Activation
```

---

# Code Implementation Examples

## Sensor Acquisition Task

```c
void vTaskAcquisition(void *pvParameters)
{
    VitalPacket vitals;

    while (1)
    {
        vitals.hr   = readHeartRate();
        vitals.spo2 = readSpO2();
        vitals.temp = readTemperature();

        xQueueSend(
            xQueueVitals,
            &vitals,
            portMAX_DELAY
        );

        vTaskDelay(pdMS_TO_TICKS(50));
    }
}
```

---

## Analysis Task

```c
void vTaskAnalysis(void *pvParameters)
{
    VitalPacket vitals;

    while (1)
    {
        if (xQueueReceive(
                xQueueVitals,
                &vitals,
                portMAX_DELAY))
        {
            if (vitals.hr > 120)
            {
                xSemaphoreGive(
                    xAlarmSemaphore
                );
            }
        }
    }
}
```

---

## Telemetry Task

```c
void vTaskTelemetry(void *pvParameters)
{
    while (1)
    {
        transmitVitalsMQTT();

        vTaskDelay(
            pdMS_TO_TICKS(200)
        );
    }
}
```

---

## Safety Guard Task

```c
void vTaskSafetyGuard(void *pvParameters)
{
    while (1)
    {
        if (taskHealthCheckFailed())
        {
            activateEmergencyBuzzer();
        }

        vTaskDelay(
            pdMS_TO_TICKS(500)
        );
    }
}
```

---

## FreeRTOS Object Creation

```c
void system_init(void)
{
    xQueueVitals =
        xQueueCreate(
            10,
            sizeof(VitalPacket)
        );

    xAlarmSemaphore =
        xSemaphoreCreateBinary();

    xI2CMutex =
        xSemaphoreCreateMutex();
}
```

---

# Engineering Concepts Demonstrated

This project demonstrates:

- Real-Time Operating Systems (RTOS)
- FreeRTOS Scheduling
- Context Switching
- Multitasking
- Queue-Based IPC
- Binary Semaphores
- Mutex Protection
- Priority Inheritance
- Embedded Networking
- MQTT Communication
- WebSocket Communication
- Medical Telemetry Systems
- Watchdog Systems
- Fault-Tolerant Firmware
- Sensor Interfacing
- Embedded Software Architecture
- IoMT Systems Design

---

# Repository Structure

```text
iomt-icu-monitor-freertos/

├── src/
│   ├── main.c
│   ├── tasks/
│   │   ├── acquisition_task.c
│   │   ├── analysis_task.c
│   │   ├── telemetry_task.c
│   │   └─```

---

# Future Enhancements

- ECG waveform acquisition
- BLE medical device integration
- TLS-secured MQTT communication
- Cloud database storage
- AI-assisted anomaly detection
- HL7/FHIR healthcare interoperability
- Multi-patient monitoring
- Remote physician dashboard
- OTA firmware updates

---

# License

Distributed under the MIT License.

See the `LICENSE` file for details.

---

# Author

**Naveen Karan R. S.**

Electronics and Communication Engineering  
Embedded Systems | FreeRTOS | IoMT | Real-Time Firmware

GitHub: **[@naveenkaranrs](https://github.com/naveenkaranrs)**

> Building scalable medical telemetry systems through real-time operating systems, embedded networking, deterministic scheduling, and hardware-aware firmware engineering.
