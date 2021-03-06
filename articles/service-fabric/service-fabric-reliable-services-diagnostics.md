---
title: Azure Service Fabric Stateful Reliable Services diagnostics | Microsoft Docs
description: Diagnostic functionality for Stateful Reliable Services in Azure Service Fabric
services: service-fabric
documentationcenter: .net
author: dkkapur
manager: timlt
editor: ''

ms.assetid: ae0e8f99-69ab-4d45-896d-1fa80ed45659
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 10/15/2017
ms.author: dekapur

---
# Diagnostic functionality for Stateful Reliable Services
The Azure Service Fabric Stateful Reliable Services StatefulServiceBase class emits [EventSource](https://msdn.microsoft.com/library/system.diagnostics.tracing.eventsource.aspx) events that can be used to debug the service, provide insights into how the runtime is operating, and help with troubleshooting.

## EventSource events
The EventSource name for the Stateful Reliable Services StatefulServiceBase class is "Microsoft-ServiceFabric-Services." Events from this event source appear in the
[Diagnostics Events](service-fabric-diagnostics-how-to-monitor-and-diagnose-services-locally.md#view-service-fabric-system-events-in-visual-studio) window when the service is being [debugged in Visual Studio](service-fabric-debugging-your-application.md).

Examples of tools and technologies that help in collecting and/or viewing EventSource events are [PerfView](http://www.microsoft.com/download/details.aspx?id=28567),
[Azure Diagnostics](../cloud-services/cloud-services-dotnet-diagnostics.md), and the
[Microsoft TraceEvent Library](http://www.nuget.org/packages/Microsoft.Diagnostics.Tracing.TraceEvent).

## Events
| Event name | Event ID | Level | Event description |
| --- | --- | --- | --- |
| StatefulRunAsyncInvocation |1 |Informational |Emitted when the service RunAsync task is started |
| StatefulRunAsyncCancellation |2 |Informational |Emitted when the service RunAsync task is canceled |
| StatefulRunAsyncCompletion |3 |Informational |Emitted when the service RunAsync task is finished |
| StatefulRunAsyncSlowCancellation |4 |Warning |Emitted when the service RunAsync task takes too long to complete cancellation |
| StatefulRunAsyncFailure |5 |Error |Emitted when the service RunAsync task throws an exception |

## Interpret events
StatefulRunAsyncInvocation, StatefulRunAsyncCompletion, and StatefulRunAsyncCancellation events are useful to the service writer to understand the lifecycle of a service, as well as the timing for when a service starts, cancels, or finishes. This information can be useful when debugging service issues or understanding the service lifecycle.

Service writers should pay close attention to StatefulRunAsyncSlowCancellation and StatefulRunAsyncFailure events because they indicate issues with the service.

StatefulRunAsyncFailure is emitted whenever the service RunAsync() task throws an exception. Typically, an exception thrown indicates an error or bug in the service. Additionally, the exception causes the service to fail, so it is moved to a different node. This operation can be expensive and can delay incoming requests while the service is moved. Service writers should determine the cause of the exception and, if possible, mitigate it.

StatefulRunAsyncSlowCancellation is emitted whenever a cancellation request for the RunAsync task takes longer than four seconds. When a service takes too long to complete cancellation, it affects
the ability of the service to be quickly restarted on another node. This scenario might affect the overall availability of the service.

## Performance counters
The Reliable Services runtime defines the following performance counter categories:

| Category | Description |
| --- | --- |
| Service Fabric Transactional Replicator |Counters specific to the Azure Service Fabric Transactional Replicator |

The Service Fabric Transactional Replicator is used by the [Reliable State Manager](service-fabric-reliable-services-reliable-collections-internals.md) to replicate transactions within a given set of [replicas](service-fabric-concepts-replica-lifecycle.md). 

The [Windows Performance Monitor](https://technet.microsoft.com/library/cc749249.aspx) application that is available by default in the Windows operating system can be used to collect and view performance counter data. [Azure Diagnostics](../cloud-services/cloud-services-dotnet-diagnostics.md) is another option for collecting performance counter data and uploading it to Azure tables.

### Performance counter instance names
A cluster that has a large number of reliable services or reliable service partitions will have a large number of transactional replicator performance counter instances. The performance counter instance names can help in identifying the specific [partition](service-fabric-concepts-partitioning.md) and service replica that the performance counter instance is associated with.

#### Service Fabric Transactional Replicator category
For the category `Service Fabric Transactional Replicator`, the counter instance names are in the following format:

`ServiceFabricPartitionId:ServiceFabricReplicaId`

*ServiceFabricPartitionId* is the string representation of the Service Fabric partition ID that the performance counter instance is associated with. The partition ID is a GUID, and its string representation is generated through [`Guid.ToString`](https://msdn.microsoft.com/library/97af8hh4.aspx) with format specifier "D".

*ServiceFabricReplicaId* is the ID associated with a given replica of a reliable service. Replica ID is included in the performance counter instance name to ensure its uniqueness and avoid conflict with other performance counter instances generated by the same partition. Further details about replicas and their role in reliable services can be found [here](service-fabric-concepts-replica-lifecycle.md).

The following counter instance name is typical for a counter under the `Service Fabric Transactional Replicator` category:

`00d0126d-3e36-4d68-98da-cc4f7195d85e:131652217797162571`

In the preceding example, `00d0126d-3e36-4d68-98da-cc4f7195d85e` is the string representation of the Service Fabric partition ID, and `131652217797162571` is the replica ID.

### Transactional Replicator performance counters

The Reliable Services runtime emits the following events under the `Service Fabric Transactional Replicator` category

 Counter name | Description |
| --- | --- |
| Begin Txn Operations/sec | The number of new write transactions created per second.|
| Txn Operations/sec | The number of add/update/delete operations performed on reliable collections per second.|
| Avg. Flush Latency (ms) | The number of bytes being flushed to the disk by the Transactional Replicator per second |
| Throttled Operations/sec | The number of operations rejected every second by the Transactional Replicator due to throttling. |
| Avg. Transaction ms/Commit | Average commit latency per transaction in milliseconds |
| Avg. Flush Latency (ms) | Average duration of disk flush operations initiated by the Transactional Replicator in milliseconds |

## Next steps
[EventSource providers in PerfView](https://blogs.msdn.microsoft.com/vancem/2012/07/09/introduction-tutorial-logging-etw-events-in-c-system-diagnostics-tracing-eventsource/)
