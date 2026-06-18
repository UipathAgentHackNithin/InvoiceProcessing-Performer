# InvoiceProcessing-Performer

## Overview

**InvoiceProcessing-Performer** is a UiPath REFramework bot that acts as the **Performer** role in the Invoice Processing automation pipeline. It dequeues invoice transactions from a UiPath Orchestrator Queue (loaded by the [InvoiceProcessing-Dispatcher](../InvoiceProcessing-Dispatcher)) and processes each one end-to-end.

> **Project code:** `3201-invoice-processing`

---

## REFramework: Dispatcher / Performer Pattern

This bot follows the standard UiPath **Dispatcher-Performer** split within the Robotic Enterprise Framework (REFramework):

```
+----------------------------------+        +-----------------------------------+
|  InvoiceProcessing-Dispatcher    |        |  InvoiceProcessing-Performer      |
|                                  |        |                                   |
|  Invoice source (inbox/folder)   |        |  Orchestrator Queue               |
|       |                          |        |       |                           |
|       v                          |        |       v                           |
|  Read & enqueue invoice items    |------> |  Dequeue & process each invoice   |
|                           Queue  |        |       |                           |
|                                  |        |       v                           |
|                                  |        |  Invoice processing end-to-end    |
+----------------------------------+        +-----------------------------------+
```

### Performer Responsibilities (this repo)

- Dequeue invoice transactions one at a time from the Orchestrator Queue
- Open the target invoicing / ERP / AP application
- Execute the invoice processing steps (validate, enter, post, approve)
- Update the invoice status in the source system
- Handle business exceptions (BizRuleException) and application exceptions (SystemException) per REFramework retry logic
- Report queue item outcomes (Successful / BusinessException / ApplicationException)

---

## Architecture

The bot follows the REFramework state machine:

| State | Description |
|---|---|
| **Initialization** | Read config from Orchestrator Assets; open invoice application |
| **Get Transaction Data** | Dequeue next invoice item from Orchestrator Queue |
| **Process Transaction** | Execute invoice processing logic for the dequeued item |
| **End Process** | Close applications; finalize reporting |

### Exception Handling

| Exception Type | Behaviour |
|---|---|
| **SystemException** | Retry up to `MaxRetryNumber` times; set item to Failed after exhausting retries |
| **BusinessException** | Mark item as Failed immediately; log details for manual review |

---

## Tech Stack

| Component | Technology |
|---|---|
| RPA Framework | UiPath REFramework |
| Orchestration | UiPath Orchestrator (Queues, Assets, Logs) |
| Target Application | Invoice / ERP / AP system (configured per deployment) |
| Language | UiPath XAML Workflows |

---

## Configuration

The following Orchestrator Assets must be created before deployment:

| Asset Name | Type | Description |
|---|---|---|
| `Invoice_Queue_Name` | String | Source Orchestrator queue name (must match Dispatcher) |
| `MaxRetryNumber` | Integer | Max retry attempts for SystemExceptions |
| `Invoice_App_URL` | String | URL or path of the invoice application |

---

## Related Repositories

- [InvoiceProcessing-Dispatcher](../InvoiceProcessing-Dispatcher) - populates the queue this bot consumes
