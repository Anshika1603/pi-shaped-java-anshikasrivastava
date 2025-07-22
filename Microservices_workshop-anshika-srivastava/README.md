# Anshika Srivastava

## Day 3 - Core Concepts of Service Discovery

## What's the difference between client-side and server-side service discovery?
Client-Side Discovery	Server-Side Discovery
The client is responsible for discovering and choosing a service instance from the registry. A load balancer (often part of the infrastructure) routes the request to an appropriate service instance.
**Example** : Netflix Eureka with Spring Cloud LoadBalancer.	
**Example** : Kubernetes Services or AWS ALB/ELB.
More flexible and lightweight. Easier to centralize routing and security.

---

## How does Eureka detect dead service instances?
Eureka uses heartbeat (health checks) to monitor service status:

Each registered instance sends a heartbeat (renewal) every 30 seconds (by default).

If Eureka doesn’t receive a heartbeat from an instance within the configured threshold (default: 90 seconds), it marks the instance as DOWN.

The instance is eventually removed from the registry.

---

## Why is the /actuator/health endpoint important for service discovery?
Eureka uses /actuator/health to determine the readiness and health of a service.

If /actuator/health returns "status": "UP", Eureka considers the service healthy and routes traffic to it.

If "status": "DOWN" or it’s unreachable, Eureka stops routing traffic to it.

Helps implement graceful shutdown, readiness checks, and failover mechanisms.

---

## How does lb:// routing in Gateway work with DiscoveryClient?
When lb://user-service is configured in spring.cloud.gateway.routes.uri, Gateway uses DiscoveryClient to resolve user-service to a list of available instances.

Then, Spring Cloud LoadBalancer chooses one instance from the list.

Gateway proxies the request to the resolved instance based on the selected strategy (like round-robin).

**Example** : lb://user-service → http://localhost:8081

---

## What happens when a registered service goes down?
Eureka detects the service as unhealthy if:

- Heartbeats stop.

- /actuator/health is DOWN.

- The service is marked OUT_OF_SERVICE or DOWN.

- Clients or Gateway stop routing requests to it.

- Eureka eventually evicts the service from the registry after a configured delay.

---

## Compare Eureka, Consul, and Kubernetes DNS-based discovery
| Feature                 | Eureka                       | Consul                         | Kubernetes DNS         |
| ----------------------- | ---------------------------- | ------------------------------ | ---------------------- |
| Type                    | Client-Side Discovery        | Both Client & Server           | Server-Side (DNS)      |
| Registry                | Yes                          | Yes                            | Uses DNS               |
| Health Check            | Application-level heartbeats | External & HTTP checks         | Pod readiness/liveness |
| Built-in UI             | Yes                          | Yes                            | No                     |
| Integration with Spring | Native (Netflix OSS)         | Good (via Spring Cloud Consul) | Basic (via DNS lookup) |
| Load Balancing          | Spring LoadBalancer          | External or custom             | kube-proxy + iptables  |


---

## How does Spring LoadBalancer choose which instance to call?
- Use a load balancing algorithm, typically round-robin by default.

- You can customize the strategy using:

  - @LoadBalancerClient configuration

  - Profiles like RandomLoadBalancer, ZoneAwareLoadBalancer

- It picks one healthy instance from the list provided by DiscoveryClient.

---
