# kotlin-coroutines
codebase from https://codelabs.developers.google.com/codelabs/kotlin-coroutines/

### Coroutines in Kotlin

The keyword `suspend` is Kotlin's way of marking a function,
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

### Controlling the UI with coroutines

In Kotlin, all coroutines run inside a [CoroutineScope](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/index.html).
A scope controls the lifetime of coroutines through its job.
When you cancel the job of a scope, it cancels all coroutines started
in that scope. On Android, you can use a scope to cancel all running
coroutines when, for example, the user navigates away
from an Activity or Fragment.

Scopes also allow you to specify a default dispatcher.
A dispatcher controls which thread runs a coroutine.
Our `uiScope` will start coroutines in `Dispatchers.Main` which is the main
thread on Android.
A coroutine started on the main won't block the main thread while suspended.
Since a `ViewModel` coroutine almost always updates the UI on the main thread,
starting coroutines on the main thread is a reasonable default.
As we'll see later in this codelab, a coroutine can switch dispatchers
any time after it's started.
For example, a coroutine can start on the main dispatcher
then use another dispatcher to parse a large JSON result off the main thread.

```java
private val viewModelJob = Job()

private val uiScope = CoroutineScope(Dispatchers.Main + viewModelJob)
```

**Important**: You must pass `CoroutineScope` a `Job` in order to cancel all coroutines started in the scope. If you don't, the scope will run until your app is terminated.
Scopes created with the `CoroutineScope` constructor add an implicit job, which you can cancel using
```java
uiScope.coroutineContext.cancel()
```