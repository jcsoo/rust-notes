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
