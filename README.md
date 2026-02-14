## Name: Fayz Reshid,

## Student no. 041066116,

## Course code: CST8917

## Assignment title: Serverless Computing - Critical Analysis

## Date: February 11, 2026

# Part 1: Paper Summary

In their 2019 paper “Serverless Computing: One Step Forward, Two Steps Back,” Hellerstein and his co-authors take a critical look at the rise of serverless computing. They argue that while serverless represents real progress in cloud development, it also reintroduces limitations that earlier distributed systems had already learned to overcome. The phrase “one step forward, two steps back” captures this tension. Serverless simplifies infrastructure management and deployment, but it does so by imposing technical constraints that make certain applications harder to design efficiently.

The authors acknowledge that platforms such as AWS Lambda make scaling nearly effortless. Developers no longer need to provision or manage virtual machines, and billing is based only on the time code actually runs. Compared to traditional cloud models, this lowers operational complexity and makes cloud development more accessible. However, these benefits are not without trade-offs.

One major concern is strict execution time limits. Early serverless platforms cap how long a function can run, forcing developers to divide long-running processes into smaller components. Although this aligns with event-driven design, it often increases system complexity because developers must manually coordinate those components and manage state across them. Rather than eliminating complexity, serverless can shift it into workflow coordination.

Communication limitations are another key issue. Serverless functions are not directly addressable and cannot easily maintain persistent connections. Instead, they rely on external storage systems, such as databases or object stores, to exchange information. This design introduces latency and can create I/O bottlenecks, especially for applications requiring fast, frequent communication. In contrast, traditional distributed systems often allow direct process-to-process communication, which can be more efficient.

The paper also introduces the “data shipping” anti-pattern. In many serverless architectures, data must be transferred to wherever the function happens to execute. Instead of bringing computation closer to the data, the system moves data to short-lived compute instances. This increases network overhead and cost, contradicting long-standing distributed systems principles that emphasize data locality.

Additionally, serverless platforms typically restrict access to specialized hardware such as GPUs or high-performance networking. This makes them less suitable for compute-intensive workloads like machine learning or large-scale analytics.

Finally, the authors argue that serverless architectures struggle with stateful and distributed applications. Because functions are stateless and ephemeral, managing shared or long-term state requires constant interaction with external storage, increasing overhead and reducing performance.

To move forward, the authors propose a more evolved cloud programming model one that better supports long-running and stateful services, enables low-latency communication between components, improves data locality, and provides access to specialized hardware. Overall, they argue that serverless must evolve beyond simplicity alone and incorporate the strengths of traditional distributed systems.

# Part 2: Azure Durable Functions Deep Dive

## Orchestration Model

Azure Durable Functions extends the Azure Functions platform by introducing a structured orchestration model composed of client, orchestrator, and activity functions. A client function initiates a workflow. The orchestrator defines the sequence of tasks and workflow logic, while activity functions perform the actual operations (Microsoft, 2023a).

Unlike basic FaaS, where each function executes independently and coordination must be handled externally, Durable Functions allows developers to define workflows directly in code. The orchestration logic lives in a central function that controls execution order, retries, and parallel tasks. This reduces the need to manually connect independent functions through triggers or external messaging systems. In practice, this makes distributed workflows more organized and easier to reason about than in first-generation serverless platforms.

## State Management

A central criticism in the paper is that serverless functions are inherently stateless. Durable Functions addresses this using event sourcing and checkpointing. As an orchestrator executes, its history and state are stored in Azure Storage. When the workflow pauses for example, while waiting for an activity or timer the runtime can replay the orchestrator’s execution using the stored history to rebuild its state (Microsoft, 2023b).

This approach allows developers to write workflows that appear stateful, even though the underlying infrastructure remains scalable and fault-tolerant. Developers do not need to manually persist state between steps, as the framework manages it automatically. While the system still depends on storage, the burden of explicit state management is significantly reduced.

## Execution Timeouts

Standard Azure Functions are subject to execution time limits depending on the hosting plan. Durable Function orchestrators, however, can run for extended periods because they are event-driven and do not consume compute resources while waiting (Microsoft, 2023c). The orchestrator effectively pauses between events and resumes when triggered.

Activity functions still operate within normal execution limits. However, by separating workflow coordination from task execution, Durable Functions enables long-running processes without requiring a single function invocation to remain active the entire time. This design mitigates the strict time constraints identified in the paper, even if it does not remove all limits entirely.

## Communication Between Functions

The paper criticizes serverless platforms for relying on storage based communication, which can introduce latency. Durable Functions handles coordination through the Durable Task framework, which internally uses Azure Storage queues and tables to manage state and message passing (Microsoft, 2023a).

From a developer’s perspective, communication between orchestrator and activity functions feels seamless. The framework abstracts the underlying storage and messaging mechanics, reducing manual coordination. However, the architecture still relies on storage backed messaging rather than direct low-latency communication. As a result, while Durable Functions significantly improves usability and workflow structure, it does not fundamentally eliminate the I/O bottleneck described in the paper.

## Parallel Execution (Fan-Out/Fan-In)

Durable Functions supports the fan-out/fan-in pattern, allowing an orchestrator to initiate multiple activity functions in parallel and then wait for all of them to complete (Microsoft, 2023d). This is particularly useful for distributed tasks such as batch processing or parallel API calls. Developers can use familiar programming constructs, such as Task.WhenAll() in C#, to manage concurrency.

This built-in support for parallel workflows makes distributed execution more natural compared to early serverless platforms, where developers had to manually track task completion. Although storage is still involved under the hood, Durable Functions offers a much more structured approach to parallel coordination.

# Part 3: Critical Evaluation

While Azure Durable Functions addresses several weaknesses of first-generation serverless platforms, it does not fully resolve all of the concerns raised by Hellerstein et al. (2019). Two limitations that remain only partially addressed are storage based communication and data locality.

First, communication between functions still depends on storage intermediaries. Although Durable Functions abstracts coordination through the Durable Task framework, it relies on Azure Storage queues and tables internally (Microsoft, 2023a). This significantly improves developer experience by removing the need for manual coordination. However, functions still do not communicate through direct, low-latency channels. The architectural reliance on storage-backed messaging means that the I/O bottleneck identified in the paper has not been fundamentally eliminated.

Second, the data shipping problem persists. Durable Functions organizes workflows effectively, but it does not inherently move computation closer to where data resides. Large datasets stored in Azure services must still be transferred to activity functions for processing. The framework does not automatically optimize for data locality, meaning data-intensive workloads may continue to experience performance overhead due to data movement.

That said, Durable Functions makes substantial progress in other areas. Its event-sourcing and replay model enables logically stateful workflows (Microsoft, 2023b), and its long-running orchestration capabilities mitigate execution time limits (Microsoft, 2023c). These improvements expand the practical scope of serverless applications.

Overall, Azure Durable Functions represents meaningful but incremental progress toward the future envisioned by Hellerstein et al. It significantly improves workflow coordination and developer productivity. However, it operates largely within the architectural constraints of the existing serverless model. Rather than fundamentally redefining serverless computing, it refines and extends it.

In that sense, Durable Functions is an important evolution. It addresses many practical challenges developers face today. Yet the deeper structural issues related to communication latency and data movement remain. It moves serverless forward but not quite as far as the authors ultimately advocate.

# References

Hellerstein, J. M., et al. (2019). Serverless Computing: One Step Forward, Two Steps Back. CIDR. https://cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf

Microsoft. (2023a). Durable Functions overview.
🔗 https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-overview

Microsoft. (2023b). Orchestrator function code constraints and replay behavior.
🔗 https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-code-constraints

Microsoft. (2023c). Performance and scale in Durable Functions.
🔗 https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-perf-and-scale

Microsoft. (2023d). Fan-out/fan-in pattern in Durable Functions.
🔗 https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-cloud-backup

---

# AI Use Disclosure

In preparing this assignment, I used an AI writing assistant (ChatGPT) to help brainstorm ideas, organize my analysis, and improve clarity and structure. The AI tool was used to refine wording and strengthen the flow of my arguments, but all research, source selection, interpretation of the readings, and final critical evaluation reflect my own understanding of the material. I reviewed, edited, and verified all cited sources to ensure accuracy and alignment with the assignment requirements.

---
