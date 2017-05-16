# Using threads for map/reduce

A common application pattern (especially in Go) is to spin up threads to
do some work in parallel, and then aggregate the results from those
workers into a single result. In this post, we will examine a very
simple example of this: computing the sum of the squares of a series of
numbers. We will do the squaring of each number in a different thread of
execution, and accumulate the results as the squares are computed.

## The Go Way

In Go, the most common way to solve this particular problem is using CSP
style. We create a channel for the worker results (i.e., the squared
numbers), spin up a goroutine for each number that sends the squared
number on that channel, and then we iterate over the channel at the end
to produce the sum. Note that we also need to take care to *close* the
channel when all the workers have finished so that the accumulator knows
when it is done. This *could* be done by counting how many numbers we
have received, but using a
[`WaitGroup`](https://golang.org/pkg/sync/#WaitGroup) is more idiomatic.

The Go code ends up looking like this:

```go
package main

import (
    "fmt"
    "sync"
)

func main() {
    var wg sync.WaitGroup
    nums := make(chan int)

    // start workers
    for i := 0; i < 100; i++ {
        wg.Add(1)
        go func(i int) {
            defer wg.Done()
            nums <- i * i
        }(i)
    }

    go func() {
        // close channel when they're done
        wg.Wait()
        close(nums)
    }()

    // accumulate worker results
    sum := 0
    for num := range nums {
        sum += num
    }

    // print result
    fmt.Println("sum is", sum)
}
```

When run, it prints out:

```console
$ go run main.go
sum is 328350
```

## The Rust Way
Much like Go, the Rust way to implement this pattern is to use a
multi-producer, single-consumer channel, called
[`mpsc`](https://doc.rust-lang.org/std/sync/mpsc/) in the standard
library. Rust doesn't have lightweight user-space threads similar to
Go's goroutines, so we instead use full operating system threads
provided by [`std::thread`](https://doc.rust-lang.org/std/thread/).

Note that Rust's use of
[RAII](https://en.wikipedia.org/wiki/Resource_acquisition_is_initialization)
simplifies the logic around closing the worker channel. When the last
worker exits, and its channel transmit handle is
[dropped](https://doc.rust-lang.org/book/drop.html), the channel will be
*automatically* closed, and the accumulator will finish.

Let's take a look at a Rust version of the code above:

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    // set up accumulator channel
    let (tx, rx) = mpsc::channel();

    // start all workers
    for i in 0..100 {
        let tx = tx.clone();
        thread::spawn(move || {
            // send our work result, and warn if master thread had a panic
            tx.send(i * i).expect("master thread exited prematurely");
        });
    }

    // close our transmit handle so channel is closed after workers exit
    drop(tx);

    // accumulate worker results
    let sum: u64 = rx.into_iter().sum();

    // print result
    println!("sum is {}", sum);
}
```

If we run this code we (unsurprisingly) get the same result as the Go
code above:

```console
$ rustc main.rs; ./main
sum is 328350
```

### Similarities between the two

 - Both languages encourage the use of channels to communicate between
   threads.
 - Both languages are written in a primarily imperative style, so the
   code ends up looking fairly similar. Moving between Rust and Go code
   (for reading) has relatively little cognitive overhead.

### Strengths Compared to Go

 - Rust is a more expressive language than Go (this is a design choice
   for both languages), and thus allows some of the logic to be
   written in a more concise way. `rx.into_iter().sum()` is a prime
   example of this.
 - Since Rust explicitly drops variables when they run out of scope,
   resource clean-up (such as closing a channel) is more straightforward
   than in Go. For example,
   [`Drop`](https://doc.rust-lang.org/std/ops/trait.Drop.html) can be
   used to easily add logic such as "close channel when the last writer
   goes away".

### Weaknesses Compared to Go

 - Rust does not have something akin to Go's goroutines for lightweight
   threading. Instead, users must use full-blown operating system
   threads. The cost of this can be ameliorated by using a [thread pool
   library](https://crates.io/search?q=thread pool) which re-uses
   threads across many jobs.
