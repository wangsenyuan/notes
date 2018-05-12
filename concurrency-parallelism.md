If there's one thing most people know about Go, is that it is designed for concurrency. No introduction to Go is complete without a demonstration of its goroutines and channels.

But when people hear the word concurrency they often think of parallelism, a related but quite distinct concept. In programming, concurrency is the composition of independently executing processes, while parallelism is the simultaneous execution of (possibly related) computations. Concurrency is about dealing with lots of things at once. Parallelism is about doing lots of things at once.

Concurrency means that an application is making progress on more than one task at the same time (concurrently).

Parallelism means that an application splits its tasks up into smaller subtasks which can be processed in parallel, for instance on multiple CPUs at the exact same time.

As you can see, concurrency is related to how an application handles multiple tasks it works on. An application may process one task at at time (sequentially) or work on multiple tasks at the same time (concurrently).

Parallelism on the other hand, is related to how an application handles each individual task. An application may process the task serially from start to end, or split the task up into subtasks which can be completed in parallel

Concurrency 更多和系统的能力有关，多少CPU，进程，线程，IO处理能力，同步/异步处理等有关。一个系统应该总是在同时(concurrently)处理很多任务；

Parallelism 是指如何并行的完成一个任务，这个任务可能需要调用更多的任务，这些任务可以一个接一个的完成（串行), 但如果（其中某些）任务没有相互依赖，可以让它们同时(paralle)执行，最后组合结果。编程更多的是处理这种情况。