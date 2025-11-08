## Understanding Go Internals

Go has its own runtime, which is a crucial part of how Go programs execute. The runtime handles things like memory management, garbage collection, and scheduling of lightweight threads called goroutines.

Unlike Rust, which relies on LLVM as its compiler backend and generates intermediate representation (IR) code, Go’s entire toolchain including its compiler, linker, and runtime is developed and maintained by the Go team. It’s a self-contained ecosystem.

### Goroutines and Scheduling

A goroutine is a lightweight abstraction over OS threads. Instead of mapping one goroutine directly to one kernel thread (as with traditional threading), Go uses an M:N scheduling model, that is, M goroutines are multiplexed onto N operating system threads. This allows Go to manage both CPU-bound and IO-bound workloads efficiently.

- Go’s scheduler uses a work-stealing mechanism and maintains both global and per-processor (local) run queues:
    - The local run queue helps reduce contention by keeping most scheduling decisions local to a "processor."
    - The global queue serves as a fallback when local queues are empty or imbalanced.

This hybrid design improves scalability and helps Go handle high concurrency workloads efficiently.

Go’s scheduler also uses cooperative preemption. If a goroutine runs for too long (typically around 10ms), the runtime can interrupt it at a safe point to let others run, preventing starvation.

### The GMP Model

- To manage concurrency internally, Go uses the GMP model, where:
    - G stands for Goroutine
    - M stands for Machine (an OS thread)
    - P stands for Processor, a logical resource that runs goroutines on machines

While the M (thread) executes the code, the P (processor) is a logical construct that holds the local run queue and other scheduling data. Each P can run only one M at a time, and there are a fixed number of Ps at any moment this controls the degree of parallelism.

The P abstraction helps Go limit and balance parallel execution. Without it, you’d have either too few active threads (leading to underutilized CPUs) or too many (causing context-switch overhead). Essentially, Ps act as the bridge between goroutines (G) and threads (M), ensuring efficient scheduling and load distribution.

### The `g0` Stack and System Stack

Each M (OS thread) has a special system goroutine called g0, which runs on the system stack the regular kernel-provided stack. In contrast, ordinary goroutines use user-level stacks that grow and shrink dynamically as needed.

Certain low-level runtime operations, such as stack growth, garbage collection, and scheduler management, need to run on a stable, non-growing stack. These operations switch to the g0 stack when necessary.

### GOMAXPROCS

The number of Ps in the GMP model is controlled by the `GOMAXPROCS` setting. It defines how many OS threads can execute Go code simultaneously effectively the level of CPU parallelism.
By default, GOMAXPROCS is set to the number of logical CPUs on your machine. This can be changed at runtime using:

```go
runtime.GOMAXPROCS(n)
```

or by setting the environment variable: 

```
export GOMAXPROCS=n
```


Adjusting this value can be useful for fine-tuning performance in CPU-bound workloads or when running in containerized environments where available cores are limited.

### Goroutine Scheduling: Poll Order

When a P (processor) needs to find work to do, the Go scheduler follows a specific poll order to locate runnable goroutines. This systematic approach ensures efficient load distribution and prevents starvation:

1. Local Run Queue: The scheduler first checks the P's local run queue. This is the most common case and minimizes lock contention since the queue is private to the P.

2. Global Run Queue (GRQ): If the local queue is empty, the scheduler checks the global run queue. To prevent starvation of goroutines in the GRQ, the scheduler periodically checks it even when the local queue has work (roughly every 61st scheduling tick).

3. Network Poller: The scheduler then checks if there are any goroutines waiting on network I/O or timers that have become ready. This integrates I/O-bound work seamlessly with CPU-bound work.

4. Work Stealing: If all previous sources are empty, the P attempts to steal work from other Ps' local run queues. It randomly selects another P and tries to steal half of its runnable goroutines. This load balancing mechanism helps distribute work across all available processors.

5. Check GRQ Again: Before giving up, the scheduler checks the global run queue one more time.

6. Poll Network Again: As a final attempt, it polls the network poller again to see if any I/O has completed.

If all these attempts fail, the P goes idle and the M (OS thread) may be put to sleep or parked until new work arrives.

This poll order is designed to maximize throughput while maintaining fairness and preventing both starvation and excessive contention on shared resources.

### References

- https://youtu.be/-K11rY57K7k
    - Dmitry Vyukov — Go scheduler: Implementing language with lightweight concurrency

- https://go.dev/src/runtime/proc.go
    - Go runtime scheduler source (G, M, P logic)

- https://go.dev/src/runtime/proc.go#L3344
    - findRunnable() function showing the complete poll order logic

- https://go.dev/blog/go1.14
    - Go 1.14 release blog (introducing asynchronous goroutine preemption)

- https://github.com/golang/go/issues/24543
    - Issue on non-cooperative (preemptive) scheduling in Go's runtime

- https://go.dev/pkg/runtime
    - Official runtime package documentation (GOMAXPROCS, etc.)

- https://go.dev/src/runtime/stack.go
    - Stack growth, preemption checks, system vs goroutine stack logic

- https://rakyll.org/scheduler/
    - The Go scheduler by Jaana Dogan (visual explanation of work stealing)

- https://morsmachine.dk/go-scheduler
    - Daniel Morsing's deep dive into the Go scheduler mechanics