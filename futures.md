Futures and Tokio
---

Servers as Futures
--
The top level server loop should be a Future, perhaps returning a status code to be used as an OS exit code. By
default you can then run a server like this:

```
use std::process::exit;

fn main() {
  let mut lp = Loop::new().unwrap();
  exit(lp.run(Server::new()).unwrap());
}
```

If you want to run multiple servers in a single loop, you can use lp.handle().spawn() to start them up.

Polling Streams
---
When using Streams, be sure to call poll() repeatedly if you get a Poll::Ok() result. Otherwise, it may not
get to the point where it will re-register the socket for a read.

Futures Not Running
---
If you have a future that's not running when expected, check for two things:

 - Was the future scheduled in a running event loop? If you create a future but it is dropped, it will not execute.
 - Did the future attempt IO (or poll_read / poll_write) on the sources it's waiting on? If it has not, the task will not be woken when a new event arrives from the source.

Error Mapping
---

Error messages can be extremely confusing because often they appear "backwards" - the compiler complains that the containing function doesn't match the inner function. It's also often the case that the Error type of the Future is what's not matching. It can be very difficult to figure out what needs to be changed to make it work.

One suggestion is to do map_err early (right after the construction of the future) so that the rest of the chain sees the final error type rather than the original error type. If you leave the map_err to the end of the chain, it's easy for it to get removed during editing or attached to something that is producing some other type of error.

Top Level Futures
---

It can be handy to start a top-level Empty Future (one which permanently returns Poll::NotReady) and then join other futures to it. The top level future will never complete, and the joined futures will be polled each time the task is awakened.

```
futures::empty::<(),()>()
  .join(<future here>)
  .join(<future here>)
  .boxed()
```

You can use other base futures - for instance, futures::finished() always returns finished and successful, meaning any and_then() attached to it will execute.


