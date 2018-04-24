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

How does Rust achieve both performance and safety? Primarily by innovations in the
type system.

// honestly here I'm off the rails. This is too much detail that I don't want
// in the talk. I need to figure out how to convey this without losing people's interest.
// Maybe we need some code here?







