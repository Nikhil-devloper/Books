# Service Discovery

## 1. Simple Explanation

**Service discovery** is how services find each other's network locations when IPs change due to scaling/deploys.

## 2. Practical Backend Example

In ECS/Kubernetes, tasks get new IPs. Consumers don't hardcode IPs; they use DNS names / AWS Cloud Map / kube DNS / a service mesh.

## 3. How It Works (step by step)

1. Service registers on start (or orchestration tracks it).
2. Client looks up name via DNS/registry.
3. Gets current healthy endpoints.
4. Connects via LB or directly.

## 5. Why Do We Need It?

Dynamic infrastructure makes static IPs impractical.

## 6. Alternatives & Trade-offs

| Method | Example |
|--------|---------|
| DNS | ECS service discovery, k8s DNS |
| Client-side | Consul + client LB |
| Mesh | App Mesh / Istio |

## 7. Interview Revision Notes

- In AWS, ALB/Cloud Map/ECS DNS often enough.
- Don't hardcode private IPs.

## 8. Common Interview Q&A

**Q1. What problem does service discovery solve?**
Finding healthy instances as they change.

**Q2. Discovery vs load balancer?**
Often combined: discover LB DNS, LB discovers targets.

## 9. Practical Production Scenario

Hardcoded IP to inventory broke after redeploy. Switched to internal DNS name.

## 10. Self-Check Questions

1. Why IPs change?
2. Who deregisters unhealthy nodes?
3. Client-side vs server-side discovery?
