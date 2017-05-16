# Rust::from(lang)

Welcome! This project is a series of guides that help people with experience in a non-Rust programming langauge get used to programming in Rust.

If you're new to Rust, some of its concepts might seem quite foreign in comparison. Sometimes it's lifetimes and the borrow checker, sometimes it's attempting to apply OOP directly when Rust's `Trait` and `impl` system calls for a slightly differnt design. Rust's error handling without using exceptions might seem tricky to work with at first. Whatever it is, that's where this project comes into play.

We have one directory for each other language. In each directory you'll find our guides about how to approach Rust, explained in terms of the language that you're already familiar with. If our guides don't cover the topic that you're looking for, then you can also check the [External](./External.md) file in the project root, which links to several other places with Rust information oriented to users of existing languages.

Of course, if you want a top to bottom explanation of Rust you should check out "[The Book](https://doc.rust-lang.org/beta/book/)", which is the community maintained complete beginner's guide to Rust. It attempt to approach Rust from a neutral background, but it's very comprehensive. At the time of this writing it's in the process of having a second edition written, but even the second edition is fairly complete.

People who wish to become more advanced Rust users should also read [The Rustonomicon](https://doc.rust-lang.org/nomicon/), which delves more into the systems that Rust uses to achieve memory and concurrency safety without use of garbage collection or a runtime.

## Contributing

Project contributions big or small are all welcomed.

* If you're experienced with a non-Rust language, write some guides to ease people into Rust and submit a [Pull Request](https://github.com/mgattozzi/rust-from-lang/pulls).
* If you spot an error, bring it up in that language's issue in the [Issue Tracker](https://github.com/mgattozzi/rust-from-lang/issues).

This is a project to help spread love and knowledge of Rust in accordance with the Rust Community's 2017 goals of [having a lower learning curve](https://github.com/aturon/rfcs/blob/roadmap-2017/text/0000-roadmap-2017.md#rust-should-have-a-lower-learning-curve).

## Writing an Article

You can take a look at the existing articles for an example of what the general format is. For example, the [Haskell - Error Handling](./Haskell/error-handling.md) article, where we cover Rust's error handling process from the perspective of a Haskell user.

1. The concept or problem is introduced. Error Handling, File Reading and Parsing, User Interaction, etc. Things that are important for being able to use a language, but that can be tricky at first because they touch on several parts of Rust at once.
2. Mention how you solve the problem in the non-Rust language, so that readers are on the same page as you and people aren't assuming different things.
3. Explain how to tackle the same problem using what's available in Rust. Feel free to use either the standard library or any crates on cargo that you feel are stable enough to present to beginners.
4. Summarize the strengths of Rust, but admit where Rust falls short when it does.

When possible, speak to the reader as if they **do** actually know what they're doing in the other language, and give them a "real" demonstration of their language's use to compare your Rust version to. Honesty is the key with this. If you don't treat their language fairly, they won't be inclined to treat Rust fairly either, and they won't be inclined to believe what you're saying.

## License

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/">
  <img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" />
</a>
<br />

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>. Any contributions that you submit must be added under the same license.
