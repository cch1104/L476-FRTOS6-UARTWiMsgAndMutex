# L476-FRTOS6-UARTWiMsgAndMutex
# STM32 CMSIS-RTOS2 Message Queue and Mutex Example

## Overview

This project demonstrates the use of:

* CMSIS-RTOS2
* FreeRTOS
* Message Queue
* Mutex
* UART Debug Output

The application creates three tasks:

| Task   | Function                                                      |
| ------ | ------------------------------------------------------------- |
| Task01 | Produces messages every 3 seconds                             |
| Task02 | Produces messages every 5 seconds                             |
| Task03 | Consumes messages from the queue and prints them through UART |

A Message Queue is used for communication between producer tasks and the consumer task.

A Mutex is used to protect access to the shared queue operation.

---

## Hardware

* STM32 Nucleo-L476RG
* USART2 (Virtual COM Port)
* STM32CubeIDE
* FreeRTOS (CMSIS-RTOS2)

UART Configuration:

| Parameter | Value |
| --------- | ----- |
| Baud Rate | 9600  |
| Data Bits | 8     |
| Stop Bits | 1     |
| Parity    | None  |

---

## RTOS Objects

### Message Queue

```c
osMessageQueueId_t myQueue01Handle;

myQueue01Handle =
    osMessageQueueNew(
        16,
        sizeof(MSGQUEUE_OBJ_t),
        &myQueue01_attributes);
```

Queue capacity:

* 16 messages

Message structure:

```c
typedef struct{
    char buf[32];
    uint32_t tick;
} MSGQUEUE_OBJ_t;
```

---

### Mutex

```c
osMutexId_t myMutex01Handle;

myMutex01Handle =
    osMutexNew(&myMutex01_attributes);
```

The mutex protects queue write operations when multiple producer tasks access the queue.

---

## Task Architecture

```text
Task01 ----\
             \
              ---> Message Queue ---> Task03 ---> UART
             /
Task02 ----/
```

### Task01

Produces a message every 3 seconds.

```c
strcpy(msg.buf, "Task 1");
msg.tick = osKernelGetTickCount();

osMessageQueuePut(...);
```

Delay:

```c
osDelay(3000);
```

---

### Task02

Produces a message every 5 seconds.

```c
strcpy(msg.buf, "Task 2");
msg.tick = osKernelGetTickCount();

osMessageQueuePut(...);
```

Delay:

```c
osDelay(5000);
```

---

### Task03

Receives messages from the queue and prints them through UART.

```c
status = osMessageQueueGet(
            myQueue01Handle,
            &msg,
            NULL,
            0U);
```

Output:

```c
printf("%s @ %5ld tick\r\n",
       msg.buf,
       msg.tick);
```

---

## Expected Output

Example UART terminal output:

```text
Task 1 @ 3000 tick
Task 2 @ 5000 tick
Task 1 @ 6000 tick
Task 1 @ 9000 tick
Task 2 @10000 tick
Task 1 @12000 tick
Task 2 @15000 tick
```

Actual timing may vary depending on scheduler timing and system clock configuration.

---

## Why Use a Message Queue?

Without a Message Queue:

* Tasks must share global variables.
* Data can be overwritten.
* Synchronization becomes difficult.

With a Message Queue:

* Tasks are decoupled.
* Messages are buffered.
* Multiple producers can safely communicate with one consumer.

---

## Why Use a Mutex?

A mutex protects shared resources.

Example:

```c
osMutexAcquire(myMutex01Handle,
               osWaitForever);

osMessageQueuePut(...);

osMutexRelease(myMutex01Handle);
```

Benefits:

* Prevents race conditions.
* Ensures only one task accesses the protected section at a time.
* Improves data consistency.

---

## Notes

In this example, FreeRTOS message queues are already thread-safe internally.

Therefore, the mutex is not strictly required around:

```c
osMessageQueuePut()
```

The mutex is included for educational purposes to demonstrate:

* Mutex creation
* Mutex acquisition
* Mutex release
* Protection of critical sections

In real-world applications, mutexes are commonly used for:

* UART transmission
* Shared buffers
* File systems
* Shared peripherals
* Global data structures

---
