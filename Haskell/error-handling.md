# Rust for Haskell Users: Error Handling

This guide is part of a series of guides that's aimed at teaching Rust to people that already know Haskell. If you don't already know Haskell then this guide is probably not going to help you, and you should try [one of our other guides](https://github.com/mgattozzi/rust-from-lang) aimed at people of different backgrounds.

This article goes over the "error handling" related issues.

## Maybe

When we want to cover the possibility of a value not existing, we wrap the
type in question in the [Maybe](https://hackage.haskell.org/package/base/docs/Data-Maybe.html) type.

```haskell
lookup :: Ord k => k -> Map k v -> Maybe v
safeDiv :: Integral a => a -> a -> Maybe a
```

`Maybe` is actually a very simple data type, and it gets most of its great
support in Haskell through its various typeclass instances.

```
ghci> :info Maybe
data Maybe a = Nothing | Just a         -- Defined in `GHC.Base'
instance Eq a => Eq (Maybe a) -- Defined in `GHC.Base'
instance Monad Maybe -- Defined in `GHC.Base'
instance Functor Maybe -- Defined in `GHC.Base'
instance Ord a => Ord (Maybe a) -- Defined in `GHC.Base'
instance Read a => Read (Maybe a) -- Defined in `GHC.Read'
instance Show a => Show (Maybe a) -- Defined in `GHC.Show'
instance Applicative Maybe -- Defined in `GHC.Base'
instance Foldable Maybe -- Defined in `Data.Foldable'
instance Traversable Maybe -- Defined in `Data.Traversable'
instance Monoid a => Monoid (Maybe a) -- Defined in `GHC.Base'
```

Rust has a similar type they call [Option<T>](https://doc.rust-lang.org/std/option/enum.Option.html), and it's defined essentially the same way as with `Maybe`:

```rust
pub enum Option<T> {
    None,
    Some(T),
}
```

The biggest difference between `Maybe` and `Option` is in the fact that Rust doesn't support the `Functor`, `Applicative`, and `Monad` typeclasses. So while the same operations _exist_, in rust they are particular to the `Option` type or whatever other type. In other words, you can't write any nice generic code that works with "anything you can map over". You have to specialize on `Option`, `Result`, `Iterator`, or something else like that. This is deeply annoying, but that's just how it is for now.

Let's go over some of the operations defined in `Data.Maybe` and see how we'd do them with `Option` instead:

### Examine The Case

Rust supports a `match` expression that's very similar to Haskell's `case` expression, but sometimes you want to just check the case you've got with a function call, and we can do that.

```haskell
isJust :: Maybe a -> Bool
isNothing :: Maybe a -> Bool
```

```rust
fn is_some(&self) -> bool
fn is_none(&self) -> bool
```

### Unwrap The Contents Safely

Usually you want what might be inside more than you want the wrapped up value. Since we sometimes don't have a value, that can be a problem. There's a few ways to approach the situation. If we want to be safe about it, we can provide a default value.

```haskell
fromMaybe :: a -> Maybe a -> a
```

```rust
fn unwrap_or(self, def: T) -> T
```

### Unwrap The Contents Unsafely

Sometimes we're so sure that we have an actual value that we want to gamble the life of the program (or at least the life of the current thread) on our assumption.

```haskell
fromJust :: Maybe a -> a
```

```rust
fn unwrap(self) -> T
```

In Haskell you get an exception if you unwrap a `Nothing`. In Rust you get a panic if you unwrap a `None`. Panics in Rust are arguably even harsher than exceptions are in Haskell (see below), so you really gotta be sure about it. Hopefully you'll only use or see `unwrap` in demos and blog posts.

### Transform as we Unwrap

```haskell
maybe :: b -> (a -> b) -> Maybe a -> b
```

This rather simple function actually does two things at once: It provides for a default value and also performs a map operation. We've seen `unwrap_or` already. There's also the ability to `map` over an `Option` as we unwrap

```rust
fn map<U, F>(self, f: F) -> Option<U> 
    where F: FnOnce(T) -> U
```

And this covers any use of `fmap` or `(<$>)`, at least the ones that you'd do with `Maybe`. Rust includes a method for combining the two if we want to map as we unwrap

```rust
fn map_or<U, F>(self, default: U, f: F) -> U 
    where F: FnOnce(T) -> U
```

### Treat it like a List

Sometimes we want to act like a `Maybe` is either a 0 or 1 element list.

```haskell
listToMaybe :: [a] -> Maybe a
maybeToList :: Maybe a -> [a]
```

In Rust you'd convert the `Option` into an [Iterator](https://doc.rust-lang.org/std/iter/trait.Iterator.html), and from there you can do any `Iterator` operation, including the [collect](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.collect) method, which converts the `Iterator` it into some sort of collection. Could be a `Vector`, `LinkedList`, or whatever else you like. You turn an `Option` (and any other type that supports iteration) into an `Iterator` with one of three functions:

```rust
fn into_iter(self) -> IntoIter<T>
fn iter(&self) -> Iter<T>
fn iter_mut(&mut self) -> IterMut<T>
```

Remember that, because of how rust manages memory, you have to pick if you're going to "consume" the `Option` in the process of iteration or not. If not, you still have to pick if you plan to mutate the contents of the `Option` or not.

### "Lists" and Maybe

```haskell
catMaybes :: [Maybe a] -> [a]
mapMaybe :: (a -> Maybe b) -> [a] -> [b]
```

Here we're doing another transform as we go through a list. Rust has the normal sort of [filter](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter) action with their `Iterator` trait of course.

```rust
fn filter<P>(self, predicate: P) -> Filter<Self, P> 
    where P: FnMut(&Self::Item) -> bool
```

But to transform as we go we can use [filter_map](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.filter_map) as well.

```rust
fn filter_map<B, F>(self, f: F) -> FilterMap<Self, F> 
    where F: FnMut(Self::Item) -> Option<B>
```

Note that `filter` and `filter_map` are `Iterator` operations, and those `Filter` and `FilterMap` output types also implement `Iterator`, so you canchain all the various `Iterator` calls together however much you need to and collect your results just once at the end of a chain of iteration operations. In the special case of `Iterator`, Rust even manages to make it all into a lazy computation (normally Rust uses strict computation of course).

### Putting It Together

Instead of combining `Maybe` steps with do-notation or `liftA2` or something similar:

```haskell
weirdLookupDiv :: Char -> Map Char Int -> Int -> Maybe Int
weirdLookupDiv key mapping divisor = do
    lookOutput <- lookup key mapping
    firstDiv <- safeDiv lookOutput divisor
    safeDiv firstDiv divisor
```

we combine them with a function call chain:

```rust
fn weird_lookup_div(key: char, mapping: HashMap<char,isize>, divisor: isize) -> Option<isize> {
    mapping.get(key)
        .and_then(|lookOutput| safeDiv(lookOutput,divisor))
        .and_then(|firstDiv| safeDiv(firstDiv,divisor))
}
```

## Either

Rust also has a version of Haskell's `Either e a`, which is `Result<T, E>`.

```rust
enum Result<T, E> {
   Ok(T),
   Err(E),
}
```

Note that the "error" case's type is second in Rust rather than first. That's because you can't have partly applied types in Rust, so they don't have to worry about how you make your Functor instance and such.

The difference here between Rust and Haskell that's worth noting on top of what's alreay been explained is that it's very common practice in Rust to use a type alias so that they can elide the error type in funciton signatures.

```rust
// in std::io::Error, https://doc.rust-lang.org/std/io/struct.Error.html
pub struct Error { /* fields omitted */ }

// in std::io::Result, https://doc.rust-lang.org/std/io/type.Result.html
type Result<T> = Result<T, Error>;
```

As the docs for `std::io::Result` say, this aliasing can be confusing, so the various result types will normally be prefixed with the module they come from, such as `io::Result`, to keep it straight.

Most of how you'd use `Result` is similar to how you'd use `Option`, so we don't need to go over the rest really. Except to note that `Result` has two mapping options: `map` and `map_err`. The former is just like you'd exect, producing a new `Ok` value only if you call it on an `Ok` value, or returning the same `Err` if it was an `Err`. The latter is the opposite situation: it does something if you call it on an `Err` case and nothing otherwise. You'll often see code where a `Result` is obtained from a call, then `map_err` is used to cover the error case before using `map`, `and_then`, and others are used to proceed forward with the successful case.

## Exceptions

Rust doesn't have exceptions like Haskell does. Instead, they are really, _really_ encouraged to use Result at all times, because if things still somehow get into an impossbily bad state then a function is forced to [panic](https://doc.rust-lang.org/std/macro.panic.html). What happens at this point is actually _not_ defined by reading any of the code anywhere in the project. Instead, you have to check the _compilation configuration_ of the project. A `panic` can either `unwind` or `abort`.

* The `unwind` route does mostly what you'd expect for a C-style language. Each function on the stack "immediately returns", which makes them have no return value but does allow them to call destructors as if everything just went out of scope normally. Depending on how your `Drop` implementations are written, this might or might not restore you to a sane state. Note that this might affect both your internal program state and your external world state, since a file might now be sitting on disk only partly written or something bad like that.

* The `abort` route just kills the process right then and there. No cleanup fuss. Why would you give up the option for cleanup? Well, the unwind mechanisms have mild overhead, and if you compile with panic=abort then you don't have to have that overhead. Usually the OS cleans up things like memory and file handles automatically for you, so in many cases just aborting the process is actually a reasonable thing to do.

If you're planning on definitely compiling with `unwind` mode, there's a function called [catch_unwind](https://doc.rust-lang.org/std/panic/fn.catch_unwind.html) which lets you run a closure and then get the result of that as a `Result` value. This is _kinda_ like [try](https://hackage.haskell.org/package/base-4.9.1.0/docs/Control-Exception.html#g:7) from `Control.Exception`, but it's actually totally unreliable as a control flow system in library code, because users of the library might compile the final binary with panic=abort, and then you're hosed.

So, just use `Result` and never, ever `panic` in library code.
