**Q: "How would you handle a sudden 10x traffic spike?"**
```
A: "I'd implement predictive auto-scaling based on historical patterns, use CloudFront for static content,
implement circuit breakers to protect downstream services, and have pre-warmed standby capacity. For extreme
spikes, I'd use SQS to queue requests and process them asynchronously."
```

**Q: "What if the database becomes the bottleneck?"**
```
A: "I'd implement read replicas for read scaling, connection pooling to reduce connection overhead, query
optimization and indexing, caching frequently accessed data in Redis, and consider database sharding for write
scaling if needed."
```

**Q: "How do you ensure zero downtime deployments?"**
```
A: "I'd use blue-green deployments with health checks, implement circuit breakers and graceful degradation, use
feature flags for gradual rollouts, maintain backward compatibility in APIs, and have automated rollback
procedures."
```









