# Using anonymous functions and closures

JavaScript, more so than most other modern languages, is dominated by
the use of *callbacks*. Whenever an operation takes some time to
complete, or you need to wait for an external event, like the user
pressing a key or a network connection to be made, you pass in an
unnamed (anonymous) function to be called when execution can continue.
These kinds of functions also appear frequently in modular JavaScript
code, where they can be used to give an object "methods" by assigning an
anonymous function to one of the object's fields.

Anonymous functions in JavaScript are inherently
[*closures*](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29),
which is to say that they capture the values of all variables that were
in scope when the function was constructed. In other words, JavaScript
closures can access variables from the parent scope of where that
function was defined, even if is being called from a different scope.

This post will explore how JavaScript closures translate into Rust
closures, and will also highlight some of the differences between the
two. We will be using two working examples. First, we'll square and sum
a list of numbers to illustrate how closures are defined, and how they
can be passed to other functions. Second, we'll write our own `map`
function to show how to write a function that accepts another function
as an argument (also known as "[Higher-order
functions](https://en.wikipedia.org/wiki/Higher-order_function)").

## The JavaScript Way

Closures in JavaScript are declared and defined using `function(args) {}`,
i.e., by simply omitting the function name. Our first example can be
expressed in JavaScript as:

```javascript
var nums = [1, 2, 3, 4, 5];
var sos = nums
	.map(function(i) {
		return i * i;
	})
	.reduce(function(acc, i) {
		return acc + i;
	}, 0);
console.log(sos);
```

When run, this code prints `55`, which is the value you would get if you
manually computed the sum of the squares of the numbers between one and
five.

Writing a `map` function in JavaScript is also fairly straightforward:

```javascript
function map(list, fn) {
	// we need to make a new list so we don't modify the old one
	nlist = [];
	for (var i = 0; i < list.length; i++) {
		// apply fn to each element of the list
		nlist.push(fn(list[i]));
	}
	// the mapped list is in nlist
	return nlist;
}
```

Note that since JavaScript passes arrays by reference (i.e., it does
not copy them), we must create a *new* list in which to put the values
we have applied `fn` to. If we didn't, we would be modifying the array
that was passed to us, even for the caller! In fact, our current
solution isn't quite right either; if `list` is an array of objects, and
`fn` modifies its argument before returning it, then we will still be
modifying the original list. Furthermore, our `map` doesn't check the
type of its arguments at all — if the caller provided a `list` that
wasn't an array, or a `fn` wasn't a function, our function would break
in unpredictable ways! There are ways to mitigate this in JavaScript
using the [`typeof`
operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof),
or methods like
[`Array.isArray`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/isArray),
but these are (unfortunately) rarely used in practice.

## The Rust Way

Closures in Rust can be somewhat more involved than in JavaScript. This
is partially due to the borrow-checker, and partially due to Rust's
stricter type system. However, for our first example, we will see that
they actually work out to be quite similar:

```rust
fn main() {
    let nums = vec![1, 2, 3, 4, 5];
    let sos = nums.iter()
	    .map(|i| i * i)
	    .fold(0, |acc, i| acc + i);
    println!("{}", sos);
}
```

When we run this code, we get the same result as the JavaScript version
as expected:

```console
$ rustc main.rs; ./main
55
```

Notice that we did not need to annotate our code with types anywhere!
Rust automatically figured out that the `map` closure takes a number and
produces a number (`i32` is the default number type), and similarly for
`fold`. If you try changing the numbers in the list to be strings
instead, the code won't compiled since `*` isn't defined for strings,
nor is `+` with respect to a number as used with `fold`.

Now, for our `map` function:

```rust
fn map<T1, T2, F: Fn(&T1) -> T2>(list: &[T1], f: F) -> Vec<T2> {
    let mut nlist = Vec::new();
    for v in list {
        nlist.push(f(v));
    }
    nlist
}
```

Okay, there are a few tricky points we should cover here. First, since
we want our `map` function to be able to map a list of any type into a
list of any other type, our function needs to be *generic*. It is
generic over two types: the type we're converting from, and the type
we're converting to. This is `T1` and `T2` respectively.

Second, we can't just directly place `Fn(&T1) -> T2` as the type of the
`f` argument. Why not? Well, it's a little bit complicated, but **The
Rust Book** gives a [pretty good
explanation](https://doc.rust-lang.org/book/closures.html#taking-closures-as-arguments).
Essentially, the compiler needs to know which function to *actually*
call in order to generate the code for `map`. It's not satisfied by just
anything that can act as a function. We could change our code to instead
take a [function
*pointer*](https://doc.rust-lang.org/book/closures.html#function-pointers-and-closures),
but that's a topic for another time. The code as written above will
actually cause the compiler to generate a dedicated, and *specialized*
version of `map` **for every function it is ever called with**. This can
vastly improve performance, though it also increases the size of your
binary.

We take a *reference* to the list (`&[]`). Thus, we are guaranteeing
**at compile time** that our `map` function won't modify `list` in any
way (including anything contained within it). Since our `f` is only
given a reference to the value it is mapping (`&T1`), it also cannot
modify that value in place.

It is also worth noting that the Rust code does not suffer from the
same issue as the JavaScript code did of failing in unexpected ways if a
non-list or non-function is passed in. The code would simply not
compile.

Closures in Rust are a fairly involved topic, and there is *a lot* more
to be said about them. The examples above should be enough to get you
off the ground though. For more in-depth discussion, you should you read
the chapter on Closures in [**The Rust
Book**](https://doc.rust-lang.org/book/closures.html), and in
particular, the chapters on [`move`
closures](https://doc.rust-lang.org/book/closures.html#move-closures)
and on [returning
closures](https://doc.rust-lang.org/book/closures.html#returning-closures).

### Similarities between the two

 - Both languages have anonymous functions (yay!)
 - Functions are first-class primitives in both languages, meaning we
   can assign functions to variables, pass them as arguments to other
   functions, or even return functions from functions. This is what
   enables us to write a *higher-order function* like `map`.
 - Both languages allow anonymous functions to capture and access their
   environment (i.e., they both provide *closures*).

### Strengths Compared to JavaScript

 - `||` is shorter to type than `function()`
 - Rust functions (like the rest of Rust) are type checked, so type
   errors will be caught at compile time, rather than at runtime.
 - Through the borrow-checker, Rust code can give guarantees about *how*
   a closure may access its environment (see the [`Fn`, `FnMut`, and
   `FnOnce` traits](https://doc.rust-lang.org/std/ops/trait.FnMut.html)).
 - The compiler can, in many cases, generate more efficient code, since
   the code will be specialized for the exact types involved. In
   JavaScript, there is no way to know in advance what kind of functions
   or arguments we may be given. Some JS runtimes try to dynamically
   determine this information while your application is running, and
   optimize the functions on-the-fly, but this is a much more
   error-prone approach.

### Weaknesses Compared to JavaScript

 - Because of Rust's stronger type system, the Rust code for working
   with closures (and sometimes for constructing them) can grow more
   complex than their JavaScript counterparts. Type inference will often
   help with this, but when doing more advanced things, you may need to
   manually annotate the types.
 - Because Rust has different *kinds* of closures (again, see
   `Fn`/`FnMut`/`FnOnce`), it can sometimes be tricky to reason about
   what the right type is in each different context. This gets easier
   with practice, but can be a point of confusion when starting out.
