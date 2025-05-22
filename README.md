# Tutorial 10 - Asynchronous Programming - Timer

## Understanding how it works

![img](/screenshot/sentenceresult.png)

After calling spawner.spawn(...), we added a println!() right after it. When running the program, we see that the message "hey hey" is printed before "howdy!" and "done!".

This happens because the spawn(...) call only schedules the task to be executed asynchronously — it doesn’t block the current thread. So the main thread continues and prints "hey hey" immediately.

The actual spawned task runs after the spawner is dropped, which signals the executor that no more tasks will come. Only then it starts processing the task and prints "howdy!", waits 2 seconds, and finally prints "done!".

## Multiple Spawn and removing drop


![img](/screenshot/dropspawneroff.png)

In this run, drop(spawner) was removed, and we can observe that while all tasks still execute, the order in which they complete is not strictly sequential. After the main message “hey hey”, we see the three “howdy” messages in the order they were spawned — howdy1, howdy2, howdy3 — because the spawning happens synchronously, before any actual waiting. However, the “done” messages appear in a different order: done3, done2, then done1. This is because each async task begins its two-second timer concurrently, and the completion timing depends on how the executor decides to poll and wake each future. Without drop(spawner), the executor lacks the definitive signal that no more tasks are coming, so the internal polling logic may vary. The result is a race condition between the completion of futures, leading to potentially non-deterministic ordering. This highlights that although the output may still appear “complete,” the behavior is unreliable and should not be trusted in larger applications. Proper use of drop(spawner) ensures more consistent and predictable execution behavior.

![img](/screenshot/dropspawneron.png)

In this run, the drop(spawner) statement was included, which correctly signals the executor to begin processing the task queue once all tasks have been spawned. As expected, the “howdy” messages appear in the order the tasks were spawned — howdy1, howdy2, howdy3 — since spawning is synchronous. However, the “done” messages appear out of order: done1 is printed first, then done3, and finally done2. This happens because once all tasks hit the await point on the timer, they wait concurrently for the same duration (two seconds). The executor then polls and wakes them based on its own internal scheduling logic, which is not guaranteed to follow the original spawning order. Therefore, even though all tasks had the same delay, slight differences in polling and waking caused done3 to finish before done2. This reflects the nature of asynchronous execution — tasks may start in order, but they can finish in any order depending on runtime factors. What’s important is that all tasks completed, which is ensured by properly dropping the spawner.