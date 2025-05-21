# Tutorial 10 - Asynchronous Programming - Timer

![img](/screenshot/sentenceresult.png)

After calling spawner.spawn(...), we added a println!() right after it. When running the program, we see that the message "hey hey" is printed before "howdy!" and "done!".

This happens because the spawn(...) call only schedules the task to be executed asynchronously — it doesn’t block the current thread. So the main thread continues and prints "hey hey" immediately.

The actual spawned task runs after the spawner is dropped, which signals the executor that no more tasks will come. Only then it starts processing the task and prints "howdy!", waits 2 seconds, and finally prints "done!".