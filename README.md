# MuProkaron

![Build](https://travis-ci.org/EDI-Systems/M5P1_MuProkaron.svg?branch=master) 
[![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1684/badge)](https://bestpractices.coreinfrastructure.org/projects/1684) 
[![Join the chat at https://gitter.im/M5P1_MuProkaron/Lobby](https://badges.gitter.im/M5P1_MuProkaron/Lobby.svg)](https://gitter.im/M5P1_MuProkaron/Lobby?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

点击 **[这里](README_CN.md)** 查看中文版。

**RMP** is a small real-time operating system which focuses on formal reliability and simplicity. It achieves reliability by deployment of formal techniques(not completed yet). All the basic functionalities that are necessary for RTOSes are povided, but nothing more. This guarantees that the system is the minimum possible kernel and is also suitable to be used as a guest operating system when hosted on virtual machine monitors.
This operating system is much **leaner** than any other RTOSes, especilly when compared to _FreeRTOS_ or _RT-Thread_, and understanding it should be simple enough.

Read **[Contributing](CONTRIBUTING.md)** and **[Code of Conduct](CODE_OF_CONDUCT.md)** if you want to contribute, and **[Pull Request Template](PULL_REQUEST_TEMPLATE.md)** when you make pull requests.
This software is **triple-licensed**: it is either **[LGPL v3](LICENSE.md)** or **[modified MIT license](MODMIT.md)**. **Commercial** licenses are also available upon request.

For vendor-supplied packages and hardware abstraction libraries, please refer to the **[M0P0_Library](https://github.com/EDI-Systems/M0P0_Library)** repo to download and use them properly.

## Quick Demo
### Linux minimal runnable binary
Download the precompiled 32-bit linux binary **[here](Project/ECLIPSE-GCC-LINUX/RMP/Debug/RMP)** and watch benchmark results!

### Basic thread operations
**Create a thread**
```C
    RMP_Thd_Crt(&Thd_1            /* Thread control block */, 
                Func_1            /* Thread entry */,
                &Stack_1[238]     /* Stack address */,
                (void*)0x12345678 /* Parameter */,
                1                 /* Priority */, 
                5                 /* Timeslices */);
```
**Delete a thread**
```C
    RMP_Thd_Del(&Thd_1            /* Thread control block */);
```
**Suspend a thread**
```C
    RMP_Thd_Suspend(&Thd_1        /* Thread control block */);
```
**Resume a thread**
```C
    RMP_Thd_Resume(&Thd_1         /* Thread control block */);
```

### Delaying a thread
![Delay](https://raw.githubusercontent.com/EDI-Systems/M5P1_MuProkaron/master/Documents/Demo/Delay.gif)
```C
    void Func_1(void* Param)
    {
        RMP_PRINTK_S("Parameter passed is ");
        RMP_PRINTK_U((ptr_t)Param);
        RMP_PRINTK_S("\r\n");
        while(1)
        {
            RMP_Thd_Delay(30000);
            RMP_PRINTK_S("Delayed 30000 cycles\r\n\r\n");
        };
    }

    void RMP_Init_Hook(void)
    {
        RMP_Thd_Crt(&Thd_1, Func_1, &Stack_1[238], (void*)0x12345678, 1, 5);
    }
```
### Send from one thread to another
![Send](https://raw.githubusercontent.com/EDI-Systems/M5P1_MuProkaron/master/Documents/Demo/Send.gif)
```C
    void Func_1(void* Param)
    {
        ptr_t Time=0;
        while(1)
        {
            RMP_Thd_Delay(30000);
            RMP_Thd_Snd(&Thd_2, Time, RMP_MAX_SLICES);
            Time++;
        };
    }

    void Func_2(void* Param)
    {
        ptr_t Data;
        while(1)
        {
            RMP_Thd_Rcv(&Data, RMP_MAX_SLICES);
            RMP_PRINTK_S("Received ");
            RMP_PRINTK_I(Data);
            RMP_PRINTK_S("\n");
        };
    }

    void RMP_Init_Hook(void)
    {
        RMP_Thd_Crt(&Thd_1, Func_1, &Stack_1[238], (void*)0x12345678, 1, 5);
        RMP_Thd_Crt(&Thd_2, Func_2, &Stack_2[238], (void*)0x87654321, 1, 5);
    }
```

### Counting semaphores
![Semaphore](https://raw.githubusercontent.com/EDI-Systems/M5P1_MuProkaron/master/Documents/Demo/Semaphore.gif)
```C
    void Func_1(void* Param)
    {
        while(1)
        {
            RMP_Thd_Delay(30000);
            RMP_Sem_Post(&Sem_1, 1);
        };
    }

    void Func_2(void* Param)
    {
        ptr_t Data;
        while(1)
        {
            RMP_Sem_Pend(&Sem_1, RMP_MAX_SLICES);
            RMP_PRINTK_S("Semaphore successfully acquired!\r\n\r\n");
        };
    }

    void RMP_Init_Hook(void)
    {
        RMP_Sem_Crt(&Sem_1,0);
        RMP_Thd_Crt(&Thd_1, Func_1, &Stack_1[238], (void*)0x12345678, 1, 5);
        RMP_Thd_Crt(&Thd_2, Func_2, &Stack_2[238], (void*)0x87654321, 1, 5);
    }
```

### Typical performance figures for all supported architectures
|Machine      |Toolchain     |Flash|SRAM|Yield|Mailbox|Semaphore|Mailbox/Int|Semaphore/Int|
|:-----------:|:------------:|:---:|:--:|:---:|:-----:|:-------:|:---------:|:-----------:|
|MSP430       |TI CCS6       |2.90 |0.64|1254 |2386   |2281     |2378       |2245         |
|MSP430       |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-M0    |Keil uVision 5|4.94 |1.65|374  |663    |616      |659        |617          |
|Cortex-M0+   |Keil uVision 5|6.25 |1.65|334  |607    |544      |588        |552          |
|Cortex-M3    |Keil uVision 5|2.60 |1.65|246  |456    |422      |443        |409          |
|Cortex-M3    |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-M4    |Keil uVision 5|2.70 |1.66|184  |339    |325      |374        |361          |
|Cortex-M4    |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-M7    |Keil uVision 5|6.66 |1.65|170  |256    |230      |274        |268          |
|Cortex-M7    |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-R4    |Keil uVision 5|TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-R4    |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-R5    |Keil uVision 5|TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|Cortex-R5    |GCC           |TBT  |TBT |TBT  |TBT    |TBT      |TBT        |TBT          |
|MIPS M14k    |XC32-GCC      |17.2 |2.46|264  |358    |340      |421        |415          |
|X86-LINUX    |GCC           |N/A  |N/A |33000|35000  |33000    |35000      |33000        |

**Flash and SRAM consumption is calculated in kB, while the other figures are calculated in CPU clock cycles. All values listed here are typical (useful system) values, not minimum values, because minimum values on system size seldom make any real sense.**

- MSP430 is evaluated with MSP430FR5994.
- Cortex-M0 is evaluated with STM32F030F4P6.
- Cortex-M0+ is evaluated with STM32L053C8T6.
- Cortex-M3 is evaluated with STM32F103RET6.
- Cortex-M4 is evaluated with STM32F405RGT6.
- Cortex-M7 is evaluated with STM32F767IGT6.
- MIPS M14k is evaluated with PIC32MZ2048EFM100.
- X86 Linux is evaluated with Ubuntu 16.04 on i7-4820k @ 3.7GHz.

All compiler options are the highest optimization (usually -O3) and optimized for time. 
- Yield: The time to yield between different threads.  
- Mailbox: The mailbox communication time between two threads.  
- Semaphore: The semaphore communication time between two threads.  
- Mailbox/Int: The time to send to a mailbox from interrupt.  
- Semaphore/Int: The time to post to a semaphore from interrupt.  

## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. See deployment for notes on how to deploy the project on a live system.

### Prerequisites

You need **_Cortex-M or Cortex-R or MIPS or MSP430_** microcontroller development kits to run the tests. This RTOS focuses on value-line MCUs and do not concentrate on high-end MCUs or MPUs. Do not use QEMU simulator to test the projects because they do not behave correctly in many scenarios.  
If you don't have a development board, a **_x86-based Linux port_** of RMP is also available. However, running RMP on top of linux uses the [ptrace](https://en.wikipedia.org/wiki/Ptrace) system call and [signal](https://en.wikipedia.org/wiki/Signal_(IPC)) system, thus it is not particularly fast. Just run the example and observe benchmark output.
Other platform supports should be simple to implement, however they are not scheduled yet. For Cortex-A and other CPUs with a memory management unit ([MMU](https://en.wikipedia.org/wiki/Memory_management_unit)), go [M7M1_MuEukaron](https://github.com/EDI-Systems/M7M1_MuEukaron) _Real-Time Multi-Core Microkernel_ instead; M7M1 supports some Cortex-Ms and Cortex-Rs as well.

### Compilation

The **Vendor Toolchain** or **Eclipse** projects for various microcontrollers are available in the **_Project_** folder. Refer to the readme files in each folder for specific instructions about how to run them. However, keep in mind that some examples may need vendor-specific libraries such as the STMicroelectronics HAL. Some additional drivers may be required too. These can be found in **[M0P0_Library](https://github.com/EDI-Systems/M0P0_Library)** repo.

## Running the tests

To run the sample programs, simply download them into the development board and start step-by-step debugging. Some examples will use one or two LEDs to indicate the system status. In that case, it is necessary to fill the LED blinking wrapper functions.

## Deployment

When deploying this into a production system, it is recommended that you read the manual in the **_Documents_** folder carefully to configure all macros correctly.

## Built With

- Keil uVision 5 (armcc)
- Code composer studio
- MPLAB X XC32
- gcc/clang-llvm

Other toolchains are not recommended nor supported at this point, though it might be possible to support them later on.

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## EDI Project Information
Mutate - Protero - Prokaron (M5P1 R4T1)
