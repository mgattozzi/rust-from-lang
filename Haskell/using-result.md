# Using Result
Sometimes things go wrong, like a file is missing, or maybe a database
connection failed. Whatever the reason languages provide different ways
to handle errors when they occur.

## The Haskell Way
Haskell uses what's known as Monadic Error handling. It leverages its
powerful type system and pattern matching in order to handle errors as
they occur. Haskell has a type called `Either`. It can either be a value
with the `Right` result or the error is `Left` to another type.
Conventionally the `Right` value continues a wanted value and `Left` an
error type. By using `Either` it indicates to the user that the
function might fail and should have a way to be handled if it does.

You can read up on the methods available to `Either` [here](https://hackage.haskell.org/package/base-4.9.0.0/docs/Data-Either.html).

Let's take a look at a trivial example:

```haskell
main :: IO ()
main = do
 putStrLn x
 putStrLn y
 where
   x = case (divBy 30 10) of
         (Right a) -> show a
         (Left a) -> show a
   y = case (divBy 30 0) of
         (Right a) -> show a
         (Left a) -> show a

divBy :: (Fractional a, Eq a) => a -> a -> Either String a
divBy num denom = case denom of
                    0 -> (Left "Can't divide by 0")
                    _ -> (Right (num / denom))
```

When compiled and run it prints out:

```none
3.0
"Can't divide by 0"
```

Awesome! We now have a function that safely tells us whether our
division worked or not and it shows the correct output each time.
While the code is simple and not as robust as it could be in Haskell,
it shows off how `Either` works. Generally we would want to use a
custom error type instead of `String` that we could pattern match on.

## The Rust Way
Much like Haskell, Rust uses Monadic Error handling with a type similar
to `Either` called `Result`. `Result` can store either a value or an
error inside of it and provides methods that can be used to
handle it in ways much like Haskell's `>>=` operator or `do` notation
that work on monads like `Either`.

You can read up on the methods available to `Result` [here](https://doc.rust-lang.org/std/result/enum.Result.html).

Let's take a look at a Rust version of the code above:

```rust
// Note we're using the Num type from the num crate to
// have the same generic functionality of the Haskell example
extern crate num;
use num::Num;
use std::io::{ErrorKind, Error};

pub fn main() {
    match div_by(30, 10) {
        Ok(x) => println!("{}", x),
        Err(e) => println!("{}", e),
    };
    match div_by(30, 0) {
        Ok(y) => println!("{}", y),
        Err(e) => println!("{}", e),
    };

}

fn div_by<T: Num>(num: T, denom: T) -> Result<T, Error> {
    match denom.is_zero() {
        true  => Err(Error::new(ErrorKind::Other, "Can't divide by 0")),
        false => Ok(num / denom),
    }
}
```

If we run this code we get:

```none
3
Can't divide by 0
```

This is the same response we got from the Haskell code!

### Similarities between the two
- The way both languages handle errors is the same. They both use Monadic
  Error Handling in order to enforce types as a way to reduce errors and
  to let the user know that a function might fail and that they need to
  handle it somehow. This forces the user to not leave things up to
  chance by forgetting to deal with the possibility of an error.

### Weaknesses Compared to Haskell
- `Either` is a Monad and Rust doesn't have a monad's properties explicitly
  built into the language or `Result`. This means some transformations or
  patterns you might do easily with `Either` might not work the exact same
  with Rust's `Result`. However, `Result` provides a lot of functions like
  `map` or `unwrap_or` that cover many of the transformations you might
  want to do. Check the documentation for `Result` for a full list of the
  things you can do with it.
