# Kafka Cheatsheet — Quick Learning & Revision

Goal: concise Kafka concepts, .NET patterns, producer/consumer examples and best practices.

## Basics
- Concepts: broker, topic, partition, offset, consumer group
- Local: Confluent Platform or Kafka Docker images

## Producer (Confluent.Kafka)
```csharp
var config = new ProducerConfig { BootstrapServers = "localhost:9092" };
using var p = new ProducerBuilder<Null, string>(config).Build();
await p.ProduceAsync("topic", new Message<Null,string>{ Value = "hello" });
p.Flush(TimeSpan.FromSeconds(10));
```

## Consumer
```csharp
var conf = new ConsumerConfig { GroupId = "g1", BootstrapServers = "localhost:9092", AutoOffsetReset = AutoOffsetReset.Earliest };
using var c = new ConsumerBuilder<Ignore, string>(conf).Build();
c.Subscribe("topic");
while(true) {
  var cr = c.Consume(ct);
  // process cr.Message.Value
  c.Commit(cr);
}
```

## Patterns & semantics
- At-least-once: commit after processing.
- Exactly-once: requires transactions (careful setup).
- Compacted topics for changelog/state.

## Schemas & serialization
- Use Schema Registry + Avro for schema evolution; or versioned JSON.

## Scaling & partitioning
- Partition key controls ordering; increase partitions to scale consumers.
- Monitor consumer lag.

## Error handling
- Dead-letter topic for poison messages; idempotent consumers or dedup keys.

## Real-world use
- Event sourcing: append events to topic; rebuild by replaying events.

## Deeper Kafka topics

### Topic & partition design
- Choose partition key to maintain ordering where needed (per-customer, per-entity).
- Number of partitions limits parallelism for consumer groups — plan growth.

### Consumer rebalance handling
- Handle Rebalance events to close resources and commit offsets safely:
```csharp
consumer.Subscribe("topic");
consumer.OnPartitionsAssigned += (c, pts) => { /* init */ };
consumer.OnPartitionsRevoked += (c, pts) => { c.Commit(); /* cleanup */ };
```

### Exactly-once semantics (EOS)
- EOS requires idempotent producers and transactional producers/consumers (enable.idempotence, transactional.id).
- Use transactions carefully — increases latency and broker resource usage.

### Schema Registry & compatibility
- Use Avro + Schema Registry to enforce forward/backward compatibility and reduce payload size.
- Maintain schema evolution policies (BACKWARD, FORWARD).

### Tuning & monitoring
- Monitor consumer lag, broker metrics, and partition distribution.
- Adjust linger.ms, batch.size, compression to optimize throughput/latency.

### Error handling
- Implement DLQ (dead-letter topic) and poison-message handling.
- Prefer retry with backoff outside of consumer loop, avoid tight retry loops.
