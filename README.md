# 5-DOF Robotic Arm — STM32F411RE

Firmware for a 5 degree-of-freedom robotic arm, built around a Nucleo-F411RE. Five hobby servos are driven off two hardware timers, and joint angles are streamed in over UART from a host (PC, app, joystick controller — anything that can push text over serial).

No RTOS, no HAL abstraction layer on top of HAL itself — just timers, interrupts, and a small parser. This was built to get comfortable with PWM generation and interrupt-driven UART on STM32 before layering anything more complex on top.

## How it works

Each servo needs a standard 50 Hz PWM signal with a 1–2 ms pulse width (500–2500 µs across the constrained range used here). Two timers cover the five channels:

| Servo | Joint | Timer / Channel | MCU Pin |
|-------|-------|------------------|---------|
| 0 | Base | TIM2_CH1 | PA0 |
| 1 | Shoulder | TIM2_CH2 | PA1 |
| 2 | Elbow | TIM2_CH3 | PB10 |
| 3 | Wrist | TIM3_CH1 | PA6 |
| 4 | Gripper | TIM3_CH2 | PA7 |

Both timers run off a 1 MHz counter clock (16 MHz HSI, prescaler 15) with a 20000-tick period — a clean 20 ms PWM frame — and PWM Mode 1 output compare on every channel.

Angle commands come in over USART1 (PA9/PA10, 115200 baud, interrupt-driven — `HAL_UART_Receive_IT`, one byte at a time, no DMA). The framing is deliberately simple:
