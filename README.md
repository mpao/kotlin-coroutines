# kotlin-coroutines
codebase from https://codelabs.developers.google.com/codelabs/kotlin-coroutines/

## 1. Coroutines in Kotlin

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

#### Controlling the UI with coroutines

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

## 2. Converting existing callback APIs with coroutines

starting from commit [b42393a](b42393a7f418d0b90fc746076af883db89526b31).

The goal of this exercise is to expose our network API as a suspend function so that `refreshTitle` can be rewritten as a coroutine.
To do that Kotlin provides a function `suspendCoroutine` that's used to convert callback-based APIs to suspend functions.
Calling `suspendCoroutine` will immediately suspend the current coroutine. `suspendCoroutine` will give you a `continuation`
object that you can use to resume the coroutine. A continuation does what it sounds like:
it holds all the context needed to continue, or resume, a suspended coroutine.
The `continuation` that `suspendCoroutine` provides has two functions: `resume` and `resumeWithException`.
Calling either function will cause `suspendCoroutine` to resume immediately.
You can use `suspendCoroutine` to suspend before waiting for a callback. Then, after the callback is called call `resume`
or `resumeWithException` to resume with the result of the callback.
An example of `suspendCoroutine` looks like this:
```java
// Example of suspendCoroutine

/**
 * A class that passes strings to callbacks
 */
class Call {
  fun addCallback(callback: (String) -> Unit)
}

/**
 * Exposes callback based API as a suspend function so it can be used in coroutines.
 */
suspend fun convertToSuspend(call: Call): String {
   // 1: suspendCoroutine and will immediately *suspend*
   // the coroutine. It can be only *resumed* by the
   // continuation object passed to the block.
   return suspendCoroutine { continuation ->
       // 2: pass a block to suspendCoroutine to register callbacks

       // 3: add a callback to wait for the result
       call.addCallback { value ->
           // 4: use continuation.resume to *resume* the coroutine
           // with the value. The value passed to resume will be
           // the result of suspendCoroutine.
           continuation.resume(value)
       }
   }
}
```

This example shows how to use suspendCoroutine to convert a callback-based API on Call into a suspend function.
You can now use Call directly in coroutine based code, for example
```java
suspend fun exampleUsage() {
    val call = makeLongRunningCall()
    convertToSuspend(call) // suspends until the long running call completes
}
```

