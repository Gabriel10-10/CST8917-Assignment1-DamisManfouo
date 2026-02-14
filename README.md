## DAMIS GABRIEL MANFOUO
## 041204270
## CST8917 Serverless Applications
## ASSIGNMENT 1 : Serverless Computing - Critical Analysis
## February 13 2026

**Part 1: Paper Summary**

1. `Main Argument`
The paper “Serverless Computing: One Step Forward, Two Steps Back” claims that first generation serverless platforms (FaaS) effectively provide autoscaling, pay-as-you-go execution, and eliminate server management, but they also introduce strong architectural limitations that make them unsuitable for modern data-centric and distributed workloads. The “one step forward” represents the move to a fully managed execution model where developers deploy functions and the platform scales them automatically while charging only for actual use. The “two steps back” highlights how current FaaS platforms restrict data access, communication patterns, and hardware options, which conflicts with the needs of data-intensive computing, open-source systems, and specialized hardware. As a consequence, today’s serverless platforms are mainly appropriate for simple, embarrassingly parallel tasks or as glue code for proprietary services, while limiting innovation in large-scale, stateful, cloud-native systems.

2. `Key Limitations Identified`
One major limitation is strict execution time limits: functions can run only for short periods, forcing long-running tasks to be broken into many executions with extra coordination and repeated startup costs, which increases latency and expenses. This is poorly suited for iterative workloads such as machine learning training or complex analytics that benefit from long-lived processes and in-memory state. A second limitation concerns communication and networking: functions are not directly addressable and cannot use fast communication channels, so they rely on external storage or messaging services, creating performance and cost bottlenecks. This weakens traditional distributed algorithms that depend on low-latency peer-to-peer communication. Third, the authors describe FaaS as a “data shipping” model, where data is repeatedly moved to short-lived, stateless functions instead of executing code close to the data, leading to wasted bandwidth and higher latency. Fourth, access to specialized hardware is limited, as most FaaS platforms do not expose GPUs or accelerators, even though such hardware is essential for modern workloads. Finally, supporting distributed and stateful applications is difficult because functions are stateless and temporary, requiring frequent interaction with external storage systems that offer weak consistency. Altogether, these issues prevent first generation serverless platforms from effectively supporting complex distributed and stateful applications.

3. `Proposed Future Directions`
To address these issues, the authors propose several directions for future cloud programming models. First, they recommend moving computation closer to data by allowing code to run alongside high-throughput data stores, enabling data-centric execution. Second, they argue for better support of heterogeneous hardware, where high-level abstractions can target GPUs and other accelerators without exposing infrastructure complexity to developers. Third, they suggest shifting from purely short-lived functions to longer-lived, addressable entities, such as actors, that can maintain state and directly participate in distributed coordination. Other suggestions include models better suited for massive parallelism, stronger ties to open-source ecosystems, and clearer ways to define and meet application-level performance and cost objectives. Overall, the authors envision a cloud platform that keeps the simplicity and elasticity of serverless computing while rethinking data placement, communication, and hardware access to better support large-scale, data-intensive innovation.

**Part 2: Azure Durable Functions Deep Dive**

1. `Orchestration Model`
Azure Durable Functions relies on a three-layer architecture composed of client functions that start workflows, orchestrator functions that control execution logic, and activity functions that carry out computation (Microsoft, 2024). This model clearly contrasts with basic FaaS, where functions execute independently without built-in coordination. Hellerstein et al. (2019) criticized this isolation, noting that developers must rely on external services to orchestrate workflows. Durable Functions addresses this issue by offering native orchestration primitives, enabling developers to define complex, multi-step workflows directly within the serverless environment. As a result, scenarios that previously required multiple cloud services and extensive custom logic can now be implemented through a single, coherent programming model, while still preserving autoscaling and pay-per-use characteristics.

2. `State Management`
Durable Functions handles state using an event-sourcing approach with automatic checkpointing, where the Durable Task Framework saves execution state to Azure Storage whenever an orchestrator pauses and later replays the history to restore state without repeating completed steps (Microsoft, 2024). This directly challenges Hellerstein et al.’s (2019) claim that serverless functions are inherently stateless and unable to share state across invocations. Through transparent state persistence, the framework maintains execution context across function calls. In addition, this replay mechanism provides fault tolerance, allowing orchestrations to resume from the last checkpoint after failures. Consequently, developers can write code that appears sequential, while the platform manages persistence and recovery behind the scenes.

3. `Execution Timeouts`
In the Consumption plan, standard Azure Functions impose a default five-minute timeout, extendable to ten minutes, reflecting Hellerstein et al.’s (2019) concern that time limits hinder long-running workloads. Durable Functions overcome this restriction through their checkpoint-and-replay design: when an orchestrator waits for an event or activity, it is unloaded and consumes no resources until it resumes (Microsoft, 2024). This allows orchestrations to run for extended periods, including days or indefinitely through the “eternal orchestration” pattern. However, activity functions still follow standard timeout limits since they execute computation. Despite this, the model provides a practical solution for supporting long workflows within a serverless framework.

4. `Communication Between Functions`
Communication between orchestrator and activity functions is managed by the Durable Task Framework, which internally uses Azure Storage but fully abstracts messaging, coordination, and result handling from developers (Microsoft, 2024). Developers interact with activities through simple method calls such as CallActivityAsync(). This is important given Hellerstein et al.’s (2019) criticism that serverless functions depend on slow storage-based communication and lack direct addressability. While Durable Functions still rely on storage internally, the abstraction offers a programming experience similar to direct calls, removing much of the coordination complexity. Support for sub-orchestrations and external events further enhances expressiveness beyond basic FaaS capabilities.

5. `Parallel Execution (Fan-Out/Fan-In)`
Durable Functions support parallel execution through the fan-out/fan-in pattern, where an orchestrator launches multiple activities concurrently and later waits for all results using *Task.WhenAll()* (Microsoft, 2024). This directly responds to Hellerstein et al.’s (2019) argument that serverless architectures complicate distributed computing due to limited coordination. Instead of manually managing parallel workers, aggregating results, or handling failures, developers rely on built-in primitives provided by the framework. As a result, distributed patterns such as parallel data processing or batch tasks become easier to implement, while maintaining the simplicity and scalability associated with serverless computing.

**Part 3: Critical Evaluation**

Despite the notable improvements introduced by Azure Durable Functions to the serverless model, several core criticisms identified by Hellerstein et al. (2019) remain unresolved or only partially addressed.

First, the issue of data locality continues to be a major architectural limitation. The authors describe serverless computing as suffering from a “data shipping” problem, where functions must constantly retrieve data from remote storage instead of executing near the data itself (Hellerstein et al., 2019). Azure Durable Functions does not solve this problem, as activity functions still run as short-lived compute units that depend on external data sources. Although orchestration improves coordination, it cannot remove the inefficiency of repeatedly moving large datasets to stateless functions. As a result, developers handling data-intensive workloads such as analytics or machine learning must tolerate performance overheads or avoid the serverless model altogether.

Second, the lack of access to specialized hardware remains a significant limitation. Hellerstein et al. (2019) noted that serverless platforms limit users to a narrow range of configurations without support for GPUs, TPUs, or FPGAs required by modern applications. Azure Durable Functions inherits this restriction from Azure Functions, which does not offer GPU-enabled environments under its consumption model (Microsoft, 2024). Although Azure provides GPU resources through other services, these options sacrifice serverless benefits such as automatic scaling and pay-per-use pricing, leaving many advanced workloads outside the serverless ecosystem.

Finally, after analyzing how Azure Durable Functions responds to the paper’s critiques, it can be argued that it represents advanced engineering workarounds rather than the deep architectural changes envisioned by Hellerstein et al. While the framework clearly enhances developer experience, it does so through abstraction rather than by resolving fundamental issues. For example, communication between functions appears seamless, but still relies on Azure Storage queues—the same “slow storage intermediaries” criticized in the paper. Likewise, indefinite orchestration through checkpointing avoids timeout limits without removing them. However, labeling these improvements as mere workarounds would be overly dismissive. The authors themselves suggested that FaaS platforms could evolve to mitigate their limitations, and Azure Durable Functions illustrates such practical evolution. For many real-world scenarios, especially event-driven workflows and service orchestration, the framework provides adequate functionality without requiring radical architectural change.

In conclusion, Azure Durable Functions significantly broadens the capabilities of serverless computing but does not fully resolve the tension between ephemeral, stateless execution and the demands of data-intensive distributed systems. Whether this constitutes meaningful progress ultimately depends on the workload, reflecting serverless computing as an evolving model rather than a finished solution.


**References**

Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). Serverless Computing: One Step Forward, Two Steps Back. Conference on Innovative Data Systems Research (CIDR). https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf

Microsoft. (2024). Azure Functions hosting options. Azure Documentation. https://learn.microsoft.com/en-us/azure/azure-functions/functions-scale

Microsoft. (2024). Durable Functions orchestrations. Azure Documentation. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-orchestrations

Microsoft. (2024). Durable Functions timers. Azure Documentation. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-timers

Microsoft. (2024). Durable Functions types and features. Azure Documentation. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-types-features-overview

Microsoft. (2024). Fan-out/fan-in pattern in Durable Functions. Azure Documentation. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-cloud-backup


**AI Disclosure Statement**
- Use AI for references
- Use AI to Summarize th part1 but rephrase it
