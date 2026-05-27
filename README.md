# STM32_ADC_DMA_PWM_Controller_LL
# High Efficiency Autonomous ADC DMA System with Hardware PWM & IWDG

A high performance, bare-metal embedded software application built for the **STM32F4** platform using **STM32 LL (Low-Level) Drivers**. This project demonstrates how to offload signal processing from the CPU by leveraging hardware **DMA in Circular Mode** to capture analog signals, implement a moving average filter, and drive an hardware breathing LED using **PWM**, all while protected by an independent hardware watchdog.

## Hardware Peripherals Used

| Peripheral | Configuration Mode | Purpose |
| :--- | :--- | :--- |
| **ADC1** | 12-Bit, 480 Cycles, Continuous | Continuous automated analog voltage measurements on `PA0` |
| **DMA2 (Stream 0)** | Circular Mode, Memory Increment | Hardware-driven transfer from ADC register directly to RAM buffer |
| **TIM2** | Prescaler 159, ARR 999, PWM CH2 | Hardware PWM generation for a variable brightness LED on `PA1` |
| **TIM3** | Prescaler 1599, ARR 2000 | Secondary timer configuration for application timing |
| **USART2** | 115200, 8N1, RXNE Interrupts | Asynchronous host communication interface (ASCII terminal gateway) |
| **IWDG** | Prescaler 32, Reload 4095 | Independent hardware watchdog to prevent system lockup |
| **EXTI13 (GPIOC)** | Falling Edge Trigger, Pull-Up | Captures tactile user button presses on `PC13` |

---

##  Software System Architecture

1. **Autonomous Capture:** The ADC continuously samples the sensor input and hands off the data to the DMA controller.
2. **Double Buffered Filter:** Once the 20-element circular buffer is full, the DMA engine trips the Transfer Complete (`TC0`) flag. 
3. **Processing & Transmission:** The CPU wakes up briefly to average the 20 samples, format the integer value into a text buffer via `sprintf`, and stream the payload out through the `USART2` data register.
4. **Hardware Safety:** If the hardware loop performs correctly, the IWDG timer is reset. If an infinite loop or bus jam occurs, the system triggers a hard hardware reset.
