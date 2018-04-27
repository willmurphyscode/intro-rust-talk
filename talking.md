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

// OPTIONAL ADDITIONAL DETAIL

To understand Rust, we really want to understand this problem, as it's
one of the core problems that Rust is trying to solve. Typical programming
languages have two regions in memory: The Stack and the Heap.

(note that this section will be a little rough since this is not really
a Ruby talk. If you want better details: 
[Garbage Collection and the Ruby Heap](https://www.scribd.com/doc/27174770/Garbage-Collection-and-the-Ruby-Heap))
Let's say you ask the Ruby interpreter to assign a string to the local
variable `foo`. (You're not using `rubocop` right now so nothing will
yell at you.) The Ruby interpreter will do a few things:

1. Ruby will allocate a memory for a C struct called RString part of the heap it reserves for this purpose
2. That C struct has a pointer to the heap, where all the characters in the string will live

But what if there are no slots available for step 1?

Ruby will:

1. Stop your program
2. Mark all objects that can still be reached.
3. Free the memory of all other objects.

Then it will attempt step 1 again. If there are still no free slots, Ruby will ask the operating system
for more memory and try the whole thing again.

This has two basic effects:

1. You don't have to worry about memory.
2. Sometimes your program just pauses.

// END OPTIONAL ADDITIONAL DETAIL

Now let's look at the other half of this problem: Why is C++ dangerous. Basically,
C++ is dangerous because it has no garbage collector. Because it has no garbage collector,
that means that you manage your own memory. If you need an array of 10 integers, you
call a function that finds a place for 10 integers and gives you back a
memory address. When you're done with that array, you have to remember to say
that you're done with the memory, and then you have to remember
not to use that address again, and you have to remember how big the array was,
because if you ask for the 11th item in a 10 item array, you'll get something,
but it won't be what you want. If you break these rules, you get "Undefined Behavior,"
which basically means your program may crash, or may not crash and start doing bad
things. Heartbleed was a small memory management bug in some important C code.

Here's an example. What will this C code do?

``` C
#include <stdio.h>

int main(void) 
{
    char name[5] = "Will";
    int ix = 27 * 24; 
    printf("And then we have a '%c' \n", name[ix]); // what will this line print? 
    return 0;
}
```

Will it:

A. Fail to compile
B. Create an error an runtime
C. Always print "And then we have a ''"
D. Sometimes do random different things.

The answer is D. 

Now this is an example of obviously incorrect memory management, but what if `name`
and `ix` were both defined in other files, or calculated from user input? That
could be bad.

One of Rust's goals is that unsafe memory things should fail to compile, or, if they
can't fail to compile, always produce the same error at runtime, so that you never
have a program that does random, strange things. The innovation of Rust
is that it has to be very safe without spending much time while your programming
is running double-checking things for you or cleaning up after you.

How does Rust achieve both performance and safety? That's what we'll learn about
today. But before I completely lose your interest, let's look at some Rust source
code.

// TODO: this code sample is now a total non sequitur. Fix.

Let's see what happens if we try to do the same nonsense in Rust that worked
in C.

``` rust
fn main() {
    let name = String::from("Will");
    let ix : usize = 24 * 27;
    println!("And then we have a '{}'", name[ix]);
}
```

We get:

```
$ rustc temp.rs -o safe
error[E0277]: the trait bound `std::string::String: std::ops::Index<usize>` is not satisfied
 --> temp.rs:4:41
  |
4 |     println!("And then we have a '{}'", name[ix]);
  |                                         ^^^^^^^^ the type `std::string::String` cannot be indexed by `usize`
  |
  = help: the trait `std::ops::Index<usize>` is not implemented for `std::string::String`

```

Let's try again:

``` rust
fn main() {
    let name = String::from("Will");
    let ix : usize = 24 * 27;
    println!("And then we have a '{:?}'", name.chars().nth(ix));
}
```

Gives us:

```
$ rustc temp.rs -o safe
$ ./safe
And then we have a 'None'
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

Rust is telling us, "Hey wait! `value` is going out of scope before you're done with it.
The won't work because at runtime, `&value` will be a pointer into freed memory." In contrast,
Ruby would have fixed your program automatically, but slowed it down while it was running to do
so, and C++ would have just said, "hey, it's your life pal. Good luck."

// TAKE QUESTIONS HERE FOR A MINUTE

Here I want to pivot and talk about what problems Rust is good at solving. Here are
some prominent use cases that I think are great examples:

First, there's an anti-spam reverse proxy company. Basically, they make a very thin
web server that sits in front of your app, and rejects requests that look like spam.
That means it has to decide, very quickly, whether each request should be let in.
It is both security and performance critical. They originally built the service
in Java, but users whose requests arrived during a GC pause would have a bad
experience. Rust has low, and fairly steady execution times for a given load.

Second, there's `RipGrep`, which is like grep, but easier to work on and
way, way faster.

Third, there's Firefox. Parts of Firefox are being re-written in Rust, especially
parts that require parallelization or processing untrusted files. For example,
`stylo` is a parallel CSS parsing library written in Rust, and Servo is a browser
engine written in Rust.

// TODO: Maybe skip this part - no one at this talk will need help running a shell script.


We'll spend the rest of this talk playing with Rust code, and teaching folks to get
started. First, go to https://rustup.rs. This Runs a shell script that basically
detects which version of Rustup you need, and then downloads and installs the
correct installer. If you want, you can go download the installer yourself:
https://github.com/rust-lang-nursery/rustup.rs/#other-installation-methods

Once that runs, you can run `cargo --version` to check that things installed
correctly. Cargo is a bit like bundler and a bit like rake. 






Sources:

[Writing Parsers Like It's 2017](http://spw17.langsec.org/papers/chifflier-parsing-in-2017.pdf)
[Mozilla Wiki: Oxidation](https://wiki.mozilla.org/Oxidation#Rust_Strengths)
[StackOverflow answer Stack and Heap](https://stackoverflow.com/a/80113/3896861)
[StackOverlow answer on Ruby Local Variables](https://stackoverflow.com/a/13639298/3896861)
[How Ruby Manages Memory and Garbage Collection](https://www.scribd.com/doc/27174770/Garbage-Collection-and-the-Ruby-Heap)













