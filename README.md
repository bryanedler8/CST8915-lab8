# CST8915-lab8






**Student Name**: Bryan Edler  
**Student ID**: 041016930  
**Course**: CST8915 Full-stack Cloud-native Development  
**Semester**: Winter 2026  



## Demo Video

🎥 [Watch Demo Video](https://youtu.be/hFEAlE_E9uQ)

---


## RabbitMQ Configuration Analysis

### Is RabbitMQ Stateless or Stateful?

**RabbitMQ is a stateful application.** It maintains persistent state including queues, messages, exchanges, bindings, and user configurations. Unlike stateless applications that can be terminated and recreated without consequence, RabbitMQ's core value is reliably storing and delivering messages across application restarts and failures. The demonstration proved this stateful nature when all queues and messages disappeared after pod deletion, confirming that RabbitMQ requires persistent state to function properly.

### Implications of Running RabbitMQ Without Persistent Storage

Running RabbitMQ without persistent storage creates severe production risks. Complete data loss occurs on every pod restart, including all queued messages, queue definitions, exchanges, and configuration. This means customer orders can be lost permanently, payment processing fails, notifications never send, and applications experience data inconsistency. Additionally, there is no disaster recovery capability, no high availability, and manual queue recreation is required after every restart. These implications make it unsuitable for any production workload requiring message reliability.

### What Happens When the RabbitMQ Pod is Deleted or Restarted

When the RabbitMQ pod is deleted, Kubernetes terminates the container, wiping the ephemeral storage where RabbitMQ stores data in `/var/lib/rabbitmq`. After deletion, Kubernetes automatically creates a new pod from the image, but it starts with a completely fresh RabbitMQ instance. As demonstrated, all previously created queues (order-queue, payment-queue, notification-queue) and their messages are permanently lost. Applications reconnect to the new instance but find no queues or data, leading to broken business processes and lost transactions.

### Potential Solutions to This Problem (Research-Based)

**Solution 1: Persistent Volume Claims (PVC)** - Add a PVC to the deployment storing RabbitMQ data at `/var/lib/rabbitmq`. This retains data across pod restarts but still has single-node failure risk.

**Solution 2: StatefulSet with Persistent Storage** - Replace the Deployment with a StatefulSet that provides stable network identities and each replica gets dedicated persistent storage, improving reliability.

**Solution 3: RabbitMQ Cluster with High Availability** - Deploy a multi-node RabbitMQ cluster using 3+ replicas with queue mirroring, enabling automatic failover and no single point of failure.

**Solution 4: Use Azure Service Bus** - Leverage a fully managed cloud message queue service that eliminates infrastructure management entirely.

### Does Using Azure Service Bus Solve the Issues?

**Yes, Azure Service Bus completely solves all identified issues.** It provides built-in data persistence across multiple availability zones with a 99.95% SLA guarantee, eliminating data loss concerns. As a fully managed service, there are no pods to manage, no persistent storage to configure, and no clustering to maintain. It offers automatic failover, geo-redundancy, built-in monitoring, and enterprise features like dead-letter queues and message sessions out-of-the-box. For cloud-native applications, Azure Service Bus represents the ideal solution, allowing development teams to focus on business logic rather than infrastructure management while achieving higher reliability and lower operational overhead.

### Setup Challenges and Lessons Learned

**Resource Allocation:** The initial challenge was finding an available Azure region due to subscription policies. East US 2 and Canada Central ultimately worked after several attempts.

**RabbitMQ API Usage:** Learning to use the RabbitMQ management API with curl required proper JSON formatting including `properties` and `payload_encoding` fields, which was initially tricky.

**Port Forwarding:** Managing multiple terminals for port-forwarding, queue monitoring, and pod operations required careful organization but provided excellent real-time visibility.

**Cleanup:** Ensuring complete resource deletion required running both cluster deletion and resource group deletion commands, with verification taking 10-15 minutes.

**Key Takeaway:** Always use persistent storage for stateful applications in production, or better yet, leverage managed services like Azure Service Bus to eliminate infrastructure complexity.

---



