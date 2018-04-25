(this is a file where I'm just going to try to write out what I'll acutally say.)

Hello and welcome! If you've been within 50 feet of me in the past year, you've probably
heard me talking about a cool new programming language called Rust. Today I'm going to
teach you what makes Rust different, and why these differences are important. Then I'm
going to teach you to write a little bit of Rust, and talk about how to learn more
or get involved in the community.

Before we can talk about what makes Rust different, we have to talk about the
problems Rust was trying to solve: If we oversimplify a bit, we can think of this problem
as "Ruby is slow and C++ is dangerous."

Let's unpack that a bit. _Why_ is Ruby slow? Basically, Ruby is slow because it's
interpreted, and has a garbage collector. Being interpreted means that Ruby source
code is translated from Ruby into instructions the CPU can execute while your program
is running. This is slower than translating the instructions ahead of time, which
is what happens in compiled languages. Having a runtime garbage collector means
that sometimes your program pauses while executing to clean up garbage. Basically,
when you say `foo = "Some awesome string #{my_input}"`, The ruby interpreter allocates
some RAM for you to store the string in. What happens to that RAM when you're done
with it? Basically, the garbage collector will sometimes pause your program, find
all the objects that you don't need any more, and free the RAM for other uses. So
Ruby is slow for two reasons: It spends some of the time your program is running
being translated to instructions your computer can run directly, and some of the time
your program is running cleaning up objects you don't need any more.

Now let's look at the other half of this problem: Why is C++ dangerous. Basically,
C++ is dangerous because it has no garbage collector. Because it has no garbage collector,
that means that you manage your own memory. If you need an array of 10 integers, you
ask the operating system for a place to put such an array, and it gives you back a
memory address. When you're done with that array, you have to remember to tell
the operating system you're done with that memory, and then you have to remember
not to use that address again, and you have to remember how big the array was and
not use too much memory. If you break these rules, you get "Undefined Behavior,"
which basically means your program may crash, or may not crash and start doing bad
things. Heartbleed was a small memory management bug in some important C code.

Enter Rust: Rust has, among others, the goal of being fast and safe. That is, Rust
will keep you from doing bad things with memory, but will also not spend any
time while your programming is running either cleaning up unused memory or translating
your program into instructions for your computer. Both of these things were taken
care of at compile time.

How does Rust achieve both performance and safety? That's what we'll learn about
today. But before I completely lose your interest, let's look at some Rust source
code.

``` rust
fn main() {
    println!("Hello, world!");
}
```

A few things are different from Ruby, right? There are braces and semi-colons, and
`println!()` looks a little weird. That's not a very interesting program,
so let's look at some more programs.

// POLL the audience - how many of you have worked in a language with static typing?

// IF FEW

Where Rust really starts to innovate is the type system. To understand this type system,
let's talk first talk about what a type system is. 

// ENDIF FEW

Now we can talk about what is special about Rust's type system. Rust's type system will
check not just type, but also lifetime and ownership.

To look at lifetimes, let's look at a simple Ruby program:

``` ruby
class SomeCollection
    def insert(value)
        @values ||= []
        @values << value
    end
end

my_collection = SomeCollection.new
(1..10).each do |item|
    value = "value number #{item + 1}"
    my_collection.insert(value)
end
```

A few things to notice. First, `value` is created on line 3, but it ends up being owned by
`my_collection` on line 1. If we deleted `value` while we still had `my_collection` around,
*bad things* would happen. To prevent this, Ruby doesn't let us delete `value` ourselves.
Ruby will sometimes pause our program, figure out whether anything can still get to `value`,
and if not, free the memory used to represent value. How often this is happening is
something that you can see in monitoring software, like this:

// IMAGE INSERT gc monitoring screenshot

As you can see, there were are periods when Ruby has made a lot of extra objects, and needs
to do a lot of clean up. One of the big ideas of Rust is, "what if we could figure out
at compile time how long everything lives, and then free things as soon as we're done with
them, without pausing the program?"

Let's look at a similar program in Rust:

``` rust
struct SomeCollection<'a> {
    strings: Vec<&'a str>
}

impl<'a> SomeCollection<'a> {
    pub fn new() -> Self {
        SomeCollection { strings: Vec::new() }
    }
    
    pub fn insert(&mut self, s: &'a str) {
        self.strings.push(s);
    }
}

fn main() {
    let mut my_collection = SomeCollection::new();
    (0..10).for_each( |item| {
        let value = format!("value number {}", item + 1);
        my_collection.insert(&value);
    });
}

```

If we walk through this program, we'll see it's similar to the Ruby program. `SomeCollection`
just wraps a vector of strings and lets you insert new strings. But when we try to build this,
we get a cool compiler error:

```
Compiling playground v0.0.1 (file:///playground)
error[E0597]: `value` does not live long enough
  --> src/main.rs:19:31
   |
19 |         my_collection.insert(&value);
   |                               ^^^^^ borrowed value does not live long enough
20 |     });
   |     - `value` dropped here while still borrowed
21 | }
   | - borrowed value needs to live until here

error: aborting due to previous error

error: Could not compile `playground`.

To learn more, run the command again with --verbose.
```

Rust is telling us, "Hey wait! `value` is going out of scope before you're done with it."










