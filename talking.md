(this is a file where I'm just going to try to write out what I'll acutally say.)

Hello and welcome! If you've been within 50 feet of me in the past year, you've probably
heard me talking about a cool new programming language called Rust. Today I'm going to
teach you what makes Rust different, and why these differences are important. Then I'm
going to teach you to write a little bit of Rust, and talk about how to learn more
or get involved in the community.

Before we can talk about what makes Rust different, we have to talk about memory.
Most of us here write Ruby (or JavaScript) all day, and these languages both do
a good job of hiding memory from us. For example, if we have a short Ruby program
like

``` ruby
puts "What is your name?"
name = gets.chomp
puts "Hello #{name}"
```

We didn't have to think about memory _at all_. We asked Ruby to make some strings
and store some strings, and it just sort of happened. Somewhere, somebody allocated
enough bytes of RAM to store these strings, but we didn't really have to think about that.

For example, let's look at this C code:

``` c
// thanks, https://stackoverflow.com/a/22065708
#include <stdio.h>

int main(void) 
{
    char name[20]; // what happens if we enter a longer name?
    printf("Hello. What's your name?\n");
    fgets(name,20,stdin);
    printf("Hi there, %s", name);
    return 0;
}
```

Here you can see, we at least had to tell C where to store the string.

``` rust
use std::io;
use std::io::prelude::*;

fn main() {
    let stdin = io::stdin();
    println!("Hi there, what is your name?");
    for line in stdin.lock().lines() {
        println!("{}", line.unwrap());
    }
}
```

