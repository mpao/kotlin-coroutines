# kotlin-coroutines
codebase from https://codelabs.developers.google.com/codelabs/kotlin-coroutines/

### Notes

>The keyword `suspend` is Kotlin's way of marking a function,
or function type, available to coroutines.
When a coroutine calls a function marked `uspend`,
instead of blocking until that function returns like a
normal function call, it suspends execution until the result
is ready then it resumes where it left off with the result.
While it's suspended waiting for a result, it unblocks the thread that it's running on so other functions or coroutines can run.
**The suspend keyword doesn't specify the thread code runs on. Suspend functions may run on a background thread or the main thread.**

```java
// Slow request with coroutines
@UiThread
suspend fun makeNetworkRequest() {
    // slowFetch is another suspend function so instead of
    // blocking the main thread  makeNetworkRequest will `suspend` until the result is
    // ready
    val result = slowFetch()
    // continue to execute after the result is ready
    show(result)
}

suspend fun slowFetch(): SlowResult { ... }
```


```java
// Request data from network and save it to database with coroutines

// Because of the @WorkerThread, this function cannot be called on the
// main thread without causing an error.
@WorkerThread
suspend fun makeNetworkRequest() {
    // slowFetch and anotherFetch are suspend functions
    val slow = slowFetch()
    val another = anotherFetch()
    // save is a regular function and will block this thread
    database.save(slow, another)
}

suspend fun slowFetch(): SlowResult { ... }
suspend fun anotherFetch(): AnotherResult { ... }
```

