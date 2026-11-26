# Low-Level System Design: Beginner to Advanced

A learning path of low-level design (LLD) concepts, in the order to learn them.
Each item lists what it is and why it matters. Practice problems at the end of
each level reference the exercises in
[system-design-primer](https://github.com/davialos/system-design-primer/tree/master/solutions/object_oriented_design).

---

## Level 1: Beginner (Foundations)

1. **Classes and objects**: Bundle state and behavior; the unit of all LLD.
2. **Encapsulation**: Hide internal state behind methods; protects invariants.
3. **Abstraction**: Expose what something does, not how.
4. **Inheritance**: Reuse and specialize behavior (is-a); use sparingly.
5. **Polymorphism**: Treat different types through one interface.
6. **Interfaces and abstract classes**: Define contracts without implementations.
7. **Composition vs inheritance**: Prefer has-a over is-a for flexibility.
8. **Relationships in UML**: Association, aggregation, composition, dependency.
9. **Basic UML diagrams**: Class and sequence diagrams to communicate design.
10. **Naming and readability**: Clear names, small functions, single purpose.
11. **Error handling**: Exceptions vs return codes, fail fast, custom errors.
12. **Basic data structures**: Arrays, lists, maps, sets, stacks, queues, and their trade-offs.

**Practice:** Parking lot, deck of cards.

---

## Level 2: Intermediate (Principles and Core Patterns)

13. **SOLID principles**
    - Single Responsibility: one reason to change.
    - Open/Closed: extend without modifying.
    - Liskov Substitution: subtypes must be usable as their base types.
    - Interface Segregation: many small interfaces beat one large one.
    - Dependency Inversion: depend on abstractions, not concretions.
14. **DRY, KISS, YAGNI**: Avoid duplication, keep it simple, don't build ahead.
15. **Law of Demeter**: Talk only to your direct collaborators.
16. **Dependency injection**: Pass dependencies in; enables testing and swapping.
17. **Creational patterns**: Singleton, Factory Method, Abstract Factory, Builder, Prototype.
18. **Structural patterns**: Adapter, Decorator, Facade, Composite, Proxy.
19. **Behavioral patterns**: Strategy, Observer, Command, State, Template Method, Iterator.
20. **Immutability and value objects**: Safer, easier to reason about.
21. **Enums, state machines, and modeling state**: Make illegal states unrepresentable.
22. **Unit testing and test doubles**: Mocks, stubs, fakes; design for testability.

**Practice:** Hash map, LRU cache, call center, online chat.

---

## Level 3: Advanced (Real-World Design)

23. **Domain-driven design basics**: Entities, value objects, aggregates, repositories, bounded contexts.
24. **Advanced patterns**: Visitor, Mediator, Chain of Responsibility, Memento, Flyweight, Bridge.
25. **Concurrency fundamentals**: Threads, race conditions, deadlocks, atomicity, visibility.
26. **Synchronization primitives**: Locks, semaphores, condition variables, read-write locks.
27. **Thread-safe design**: Immutability, thread confinement, concurrent collections.
28. **Producer-consumer and thread pools**: Bounded queues, backpressure, worker design.
29. **Async and event-driven design**: Futures, callbacks, event loops, message passing.
30. **Rate limiter and resilience patterns**: Token bucket, circuit breaker, retry with backoff, bulkhead.
31. **Cache design**: Eviction policies (LRU, LFU), write-through vs write-back, invalidation.
32. **API and interface design**: Idempotency, versioning, backward compatibility, pagination.
33. **Data modeling and schema design**: Normalization, indexing, access-pattern-driven design.
34. **Transactions and consistency at code level**: ACID, optimistic vs pessimistic locking.
35. **Extensibility and plugin architecture**: Design for change without rewrites.
36. **Performance and memory**: Big-O analysis, object pooling, avoiding allocations, profiling.
37. **Observability in code**: Structured logging, metrics, tracing hooks.
38. **Refactoring and managing technical debt**: Code smells, safe incremental change.

**Practice:** Rate limiter, in-memory cache with TTL, thread pool, notification system, elevator system, ride-sharing matcher, library or booking system.

**Worked case study:** [Autonomous agent harness](case-studies/autonomous-agent-harness.md): Strategy, Command, Observer and Composite patterns, concurrency and resilience patterns applied to a real Rust codebase ([rustAgent](https://github.com/davialos/rustAgent)).

---

## How to Approach an LLD Problem

1. Clarify requirements and use cases.
2. Identify core entities (nouns) and actions (verbs).
3. Define classes, interfaces, and relationships.
4. Apply principles and patterns where they earn their place.
5. Walk through key flows (sequence diagrams).
6. Consider edge cases, concurrency, and extensibility.
7. Write and test the code.
