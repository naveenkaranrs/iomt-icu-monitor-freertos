# IoMT ICU Real-Time Telemetry Monitor (FreeRTOS) 🏥

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Kernel: FreeRTOS](https://img.shields.io/badge/Kernel-FreeRTOS-008B45?logo=freertos)](https://www.freertos.org)
[![Target: ESP32](https://img.shields.io/badge/Target-ESP32-E7352C?logo=espressif)](https://www.espressif.com)

A deterministic medical telemetry firmware prototype engineered for Intensive Care Unit (ICU) multiparameter patient monitoring.

Built with a preemptive **FreeRTOS** real-time kernel architecture, this system ensures hard deadline compliance for vital sign acquisitions, priority-driven clinical alarm dispatching, and secure clinical telemetry relay.

---

## 🔬 Architecture & FreeRTOS Task Matrix

```
                     ┌───────────────────────────────┐
                     │   FreeRTOS Preemptive Kernel  │
                     └───────────────┬───────────────┘
                                     │
         ┌───────────────────────────┼───────────────────────────┐
         ▼                           ▼                           ▼
┌──────────────────┐       ┌──────────────────┐       ┌──────────────────┐
│  vTaskAcquisition│       │   vTaskAnalysis  │       │  vTaskTelemetry  │
│ Priority: HIGH (3│       │ Priority: MED (2)│       │ Priority: LOW (1)│
│ Period: 50ms     │       │ Period: 100ms    │       │ Period: 200ms    │
└────────┬─────────┘       └────────┬─────────┘       └────────┬─────────┘
         │                          │                          │
         └────────► Queue: Vitals ──┴────────► Queue: Alarms ──┘
```

| Task Name | Priority | Period | Stack | Functionality |
| :--- | :---: | :---: | :---: | :--- |
| `vTaskAcquisition` | **3 (High)** | 50 ms | 2048 B | Deterministic I2C/SPI sensor polling (HR, SpO2, Temp, Arterial pressure). |
| `vTaskAnalysis` | **2 (Med)** | 100 ms | 4096 B | Moving-window anomaly detection, arrhythmia classification & clinical thresholds. |
| `vTaskTelemetry` | **1 (Low)** | 200 ms | 4096 B | WebSocket/MQTT packet serialization, dispatch to hospital clinical dashboard. |
| `vTaskSafetyGuard`| **4 (Highest)**| Watchdog | 1024 B | Hardware task liveness monitoring & fail-safe emergency buzzer trigger. |

---

## 🛡️ Inter-Task Communication & Synchronization

- **Vital Data Queue (`xQueueVitals`):** Thread-safe ring buffer preventing race conditions between sensor sampling and analytical calculation.
- **Alarm Binary Semaphore (`xSemAlarm`):** Instant preemption mechanism triggering critical audible/visual alarms upon physiological limit violations.
- **Mutex Bus Lock (`xI2CMutex`):** Priority inheritance-enabled mutex preventing priority inversion during multi-sensor I2C transactions.

---

## 💻 Clinical Dashboard Integration
The system includes an industrial ICU web monitor interface supporting real-time ECG waveform rendering, patient vitals tracking, and clinical alert logging located in `web_dashboard/index.html`.

---

## 📄 License
Released under the [MIT License](LICENSE).
