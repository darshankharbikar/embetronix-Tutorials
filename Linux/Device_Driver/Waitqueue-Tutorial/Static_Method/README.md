# Linux Device Driver - Workqueue in Linux Kernel

## Overview

A Workqueue is a Linux kernel mechanism used to defer work from interrupt context to process context.

It is one of the Linux **Bottom Half** mechanisms.

When an interrupt occurs, the Interrupt Service Routine (ISR) should execute quickly. Any time-consuming work should be postponed to a Workqueue. Workqueues execute the deferred work in kernel worker threads (`kworker`) running in process context.

---

# Why Workqueue?

Interrupt handlers have strict limitations:

* Must execute quickly
* Cannot sleep
* Cannot perform lengthy processing

Workqueues solve this by moving heavy work out of the ISR. The deferred work executes later in process context where sleeping is allowed.

---

# Top Half vs Bottom Half

```text
Interrupt Occurs
       |
       v
+------------------+
| Top Half (ISR)   |
+------------------+
       |
       v
Schedule Work
       |
       v
+------------------+
| Bottom Half      |
| Workqueue        |
+------------------+
       |
       v
Deferred Processing
```

Workqueue is a Bottom Half mechanism.

---

# Linux Bottom Half Mechanisms

Linux provides:

1. Workqueue
2. Threaded IRQ
3. SoftIRQ
4. Tasklet

| Mechanism    | Context         | Can Sleep |
| ------------ | --------------- | --------- |
| Workqueue    | Process Context | Yes       |
| Threaded IRQ | Process Context | Yes       |
| SoftIRQ      | Atomic Context  | No        |
| Tasklet      | Atomic Context  | No        |

Workqueues are preferred when deferred work must sleep or block.

---

# Workqueue Architecture

```text
Hardware Interrupt
        |
        v
Interrupt Handler
        |
        v
schedule_work()
        |
        v
Global Workqueue
        |
        v
kworker Thread
        |
        v
Workqueue Function
```

---

# Key Structures

Header File:

```c
#include <linux/workqueue.h>
```

---

## work_struct

Represents a work item.

```c
struct work_struct
```

Each work item contains:

* Work information
* Callback function
* Scheduling information

---

## workqueue_struct

Represents the actual workqueue.

```c
struct workqueue_struct
```

Used when creating dedicated workqueues.

---

# Methods of Using Workqueue

Two approaches:

## 1. Global Workqueue

Uses kernel-provided worker threads.

```text
Driver
   |
   v
Kernel Global Workqueue
   |
   v
kworker Thread
```

No custom workqueue creation required.

---

## 2. Custom Workqueue

Driver creates its own workqueue.

```text
Driver
   |
   v
Own Workqueue
   |
   v
Dedicated Worker Thread
```

Provides better isolation and control.

---

# Static Work Initialization

## DECLARE_WORK()

Creates and initializes work statically.

```c
DECLARE_WORK(name, function);
```

Example:

```c
void workqueue_fn(struct work_struct *work);

DECLARE_WORK(my_work, workqueue_fn);
```

---

# Dynamic Work Initialization

## INIT_WORK()

Initialize work dynamically.

```c
INIT_WORK(
        &my_work,
        workqueue_fn);
```

Example:

```c
struct work_struct my_work;

INIT_WORK(
        &my_work,
        workqueue_fn);
```

---

# Workqueue Callback Function

Executed by the worker thread.

```c
void workqueue_fn(
        struct work_struct *work)
{
    printk(KERN_INFO
           "Workqueue Executed\n");
}
```

This function performs deferred processing.

---

# Scheduling Work

## schedule_work()

Queue work to the kernel global workqueue.

```c
schedule_work(&my_work);
```

Returns:

```text
0 -> Already queued
1 -> Successfully queued
```

---

## schedule_delayed_work()

Schedule work after a delay.

```c
schedule_delayed_work(
        &my_dwork,
        delay);
```

Example:

```c
schedule_delayed_work(
        &my_dwork,
        msecs_to_jiffies(5000));
```

---

## schedule_work_on()

Queue work on a specific CPU.

```c
schedule_work_on(
        cpu,
        &my_work);
```

---

# Delayed Work

Used when execution should occur later.

Structure:

```c
struct delayed_work
```

Example:

```c
struct delayed_work my_dwork;
```

Initialization:

```c
INIT_DELAYED_WORK(
        &my_dwork,
        workqueue_fn);
```

---

# Workqueue Example with Interrupt

ISR:

```c
static irqreturn_t irq_handler(
                int irq,
                void *dev_id)
{
    schedule_work(&my_work);

    return IRQ_HANDLED;
}
```

Workqueue Function:

```c
void workqueue_fn(
        struct work_struct *work)
{
    printk("Deferred Work\n");
}
```

Execution Flow:

```text
Interrupt
    |
    v
ISR
    |
    v
schedule_work()
    |
    v
kworker Thread
    |
    v
workqueue_fn()
```

---

# Flushing Workqueue

Wait until work completes.

## flush_work()

```c
flush_work(&my_work);
```

Blocks until the work finishes.

---

## flush_scheduled_work()

Flush all global workqueue items.

```c
flush_scheduled_work();
```

---

# Cancel Work

## cancel_work_sync()

Cancel pending work.

```c
cancel_work_sync(
        &my_work);
```

If already running, waits for completion.

---

## cancel_delayed_work_sync()

Cancel delayed work.

```c
cancel_delayed_work_sync(
        &my_dwork);
```

---

# Check Pending Work

## work_pending()

```c
work_pending(
        &my_work);
```

Returns whether work is pending.

---

## delayed_work_pending()

```c
delayed_work_pending(
        &my_dwork);
```

---

# Creating Own Workqueue

## create_workqueue()

Create dedicated workqueue.

```c
struct workqueue_struct *wq;

wq = create_workqueue(
        "my_wq");
```

Internally implemented using `alloc_workqueue()`.

---

## create_singlethread_workqueue()

Single worker thread.

```c
wq =
create_singlethread_workqueue(
        "my_wq");
```

---

# Queue Work to Custom Workqueue

## queue_work()

```c
queue_work(
        wq,
        &my_work);
```

Queues work into a custom workqueue.

---

## queue_delayed_work()

```c
queue_delayed_work(
        wq,
        &my_dwork,
        delay);
```

---

# Destroy Workqueue

```c
destroy_workqueue(wq);
```

Always destroy custom workqueues during driver removal.

---

# schedule_work() vs queue_work()

| schedule_work()       | queue_work()                |
| --------------------- | --------------------------- |
| Uses global workqueue | Uses custom workqueue       |
| No creation required  | Requires workqueue creation |
| Shared worker thread  | Dedicated worker thread     |
| Simple                | More control                |

---

# Process Context Advantages

Since workqueue runs in process context:

Allowed:

```c
msleep();
mutex_lock();
schedule();
wait_event();
```

Not possible inside ISR.

Workqueues are suitable for:

* File operations
* Memory allocation
* Waiting for events
* Blocking I/O
* Long computations

---

# Common Driver Use Cases

## Network Driver

```text
ISR
  |
  +--> Schedule packet processing
```

---

## GPIO Driver

```text
Button Interrupt
       |
       +--> Debounce in Workqueue
```

---

## SPI Driver

```text
Transfer Complete
        |
        +--> Post-processing
```

---

## USB Driver

```text
USB Event
      |
      +--> Heavy Processing
```

---

# Debugging

View worker threads:

```bash
ps -ef | grep kworker
```

Kernel logs:

```bash
dmesg
```

Trace workqueue activity:

```bash
cat /sys/kernel/debug/tracing/trace_pipe
```

Workqueue processing is executed by kernel worker threads (`kworker`).

---

# Best Practices

* Keep ISR extremely short.
* Use workqueue for lengthy operations.
* Use dedicated workqueue for heavy workloads.
* Flush or cancel work before unloading driver.
* Avoid blocking the shared global workqueue for long periods.
* Use delayed work when timing control is needed.

---

# Important APIs Summary

## Work Initialization

```c
DECLARE_WORK()
INIT_WORK()
INIT_DELAYED_WORK()
```

---

## Global Workqueue

```c
schedule_work()
schedule_delayed_work()
schedule_work_on()
```

---

## Custom Workqueue

```c
create_workqueue()
create_singlethread_workqueue()
queue_work()
queue_delayed_work()
destroy_workqueue()
```

---

## Management

```c
flush_work()
flush_scheduled_work()
cancel_work_sync()
cancel_delayed_work_sync()
work_pending()
delayed_work_pending()
```

---

# Key Takeaways

* Workqueue is a Linux Bottom Half mechanism.
* Workqueue executes deferred work in process context.
* Workqueue can sleep, unlike ISR, SoftIRQ, and Tasklets.
* `schedule_work()` uses the global kernel workqueue.
* `queue_work()` uses a custom workqueue.
* Workqueues are executed by `kworker` kernel threads.
* Workqueue is the preferred mechanism when deferred work requires sleeping or blocking operations.
* Frequently used with interrupts to move heavy processing out of the ISR.
