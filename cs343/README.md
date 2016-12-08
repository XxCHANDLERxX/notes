# CS 343 Peter Buhr Fall 2016
## Lecture 1 Thur Sept 8

6 assns
usually due 10pm
2 lates - 2 day late submission, except last one
oct 9 late => oct 13 submission
./submit cmd
./submit -tardy flag to use a late
make sure it runs on undergrad server env
last assn due last day of classes 11:55pm Dec 5
expected to debug on our own using prior knowledge from cs246

Buttons
compilation request - 5-10 min sanity check; unlimited usage
submission info
request remaining lates

Marks
correct
doc
test - all prior tools and knowledge used so far; must submit tests, data, and
documentation
must have specific, purpose-built tests along with justification (no fuzzing)
4 or 5 "I thought about the following, generated this test data, and this test
case" style, efficiency
each assn is 6%, varying difficulty; last assignment is 10% (bigger, partner
proj, marks split evenly)
assn is 40% total

Talk to IAs first
then prof
then Olga

Midterm - 20%
Nov 2 2016
Final - 40%
Must pass 30% worth of exam marks, as usual

Intro - Sequential Programming
You don't control your machine, the OS does
We have yet to write code that takes advantage of multi-core systems
EEs say MOAR TRANSISTORS
SEs need to write concurrent programs now
Some minor success in automatic concurrent programming conversion, but
generally bad
CP is an entirely new mindset "this is going to hurt"

Concurrency in C++
uC++ integrates advanced control flow
light weight threads (M:N not 1:1)

Outline
control flow
exception handling: synchronous
coroutines
    multiple stacks for one thread!
exception handling: suspending and resuming (concurrency precursor)
concurrency: multiple threads in shared memory
locking
monitors
errors
    most complicated thing about concurrency is that your usual debugging
    strategies fail
    e.g. print statements may slow your threads down enough to fix the error
tasks
optimization: sequential and concurrent models
other concurrency approaches: different concurrency paradigms
distributed: multiple threads in non-shared memory (~ half a class) (short
version of a distributed systems class!)

1 Advanced Control Flow
"This is the course where we stop accepting rubbish code. We'll hurt you if we
find some."
Bad example: cin >> d; // "priming code" repeated inside loop
Better: Exiting from the middle of a loop is great. Exits inside a loop are
crucial.
    Use a for ( ;; ) { if ( ! C1 ) break; S2; } // don't do unnecessary else
    statements that trigger S2
Nothing wrong with multiple exits from a loop.
Avoids "flag-itis".
Research paper: allowing exits from the middle of a loop means you can remove
ALL flag variables.
Static multi-level exits L1: { ... L2: { ... } break L1; } etc.
Good practice: label all exits (breaks).
    This increases reusablity in case we modify the control structures in the
    future.
Don't label control structures ("eye candy") if you aren't using them with
breaks.
"Break statements are just gotos."
"Death. Taxes. Go-tos."
You've been told go-to statements are bad, but for loops and while loops each
need 2 of them!  Labelled exits also avoids repeated code among nested else
blocks - just break the top-level control structure.
Rule of thumb: don't branch into control structures, only branch out of control
structures.

Dynamic Memory Allocation
Use the stack for temporary memory usage!
"Use the stack, Luke Skywalker!"



## Lecture 2

Advanced control flow (review)
Within a routine, virtually any control flow is possible.
Multi-exit loop (mid-test loop) has one or more exit locations within the body
of a for loop (not just the top, as in while) or bottom (do-while).
"You must have reasons for placement of your exit locations!"
This eliminates priming, duplicated code, etc.
Multiple exit locations also eliminates flag variables ("flagitis").
Labelled breaks allow us to jump out of multiple control structures. This lets
us do multi-level exits.
The above two techniques allow us to eliminate ALL flag variables (provable).
In languages without break statements, goto statements accomplish the same
thing.

Another common way to lose style marks: uncontrolled dynamic memory allocation.
Use the stack! Down the line, not using the stack will cause big-time
performance issues. Every time you do a 'new' or a 'malloc', ask yourself
whether this memory MUST exist outside of this block. If not, use the stack.
If so, you must use the heap.
e.g.
    Type *tp = new Type;
    return tp; // must be on the heap
Or, when the amount of data being read in is unknown, dynamic allocation is
implicit (e.g. when using a vector) and heap is also used.

Q: Is 'for ( ;; )' preferred over `while (true)'?
A: The two are equally valid, but the empty for loop makes cleanly inserting
debug statements (indices, etc.) easier.

Another situation where we use a heap is when large local variables are
allocated on a small stack. We would prefer to store a small pointer instead of
overflowing the stack.

Exceptions
Routine activation (calling/invoking) introduces complex control flow.
Among routines, control flow is controlled by call/return mechanisms.
Problem: We can't return directly to grandparent callers.
Solution: Dynamic multi-level exits.
Problem: Modularization fails when factoring exits, e.g. multi-level exits.
Labels only have routine scopes! Even within different block scopes within a
routine, labels must be unique.
So we must choose between labelling and modularization!

A routine can have multiple kinds of returns.
e.g. Exceptional return locations, say labels passed into a routine via
pointers.
e.g. Fortran code snippet on slide 21.
Fun fact: gcc has a flag that lets it compile Fortran.

Dynamic multi-level exit extends call/return semantics to transfer in the
reverse direction to normal routine calls, called non-local transfer. How? We
can set label variables! [Slide 23]
This lets us return to (different!) grandparent labelled locations.
e.g. h(g(f())), f returns to label set in h.
But now the crazy part: h or g could be called recursively! Now we must decide
which parent has the right label. This means label variables are a touple of
the stack frame and the location within the stack frame.
Non-local transfer thus needs:
1. Pointer to a block activation on the stack.
2. Transfer point within the block.
Non-local transfer is possible in C using:
- jmp_buf to declare a local variable
- setjmp
- longjmp
Note: There is a pattern where we save previous values of label L and reset it
later so that our code plays nicely with others.
Note: We don't "throw away" the goto statement, we tame it! With labelled
exits, we avoid goto's caveats.

Traditional approaches (alternatives to mle):
- Return codes (return a funky value that gets passed up the stack). Different
  error codes means a routine with multiple exit conditions - the problem we
  are trying to solve.
- Status flags (e.g. errno magic variable in "the cloud" everywhere in C)
- Fixup routines (function pointers passed that call powerful user-defined
  exception-handling code.)

Intermediate approach: return union (return a touple of (result, error code)
that must always be checked by the caller.)

Drawbacks: checking of traditional return codes is optional, so it can be
delayed or ignored. Return codes also mix exceptional and normal values.
Testing and handling of return codes is also done locally, which is bad in
situations like library code where we don't want to handle the error ourselves.
Also a few more problems on [slide 29].

Exception Handling
A routine can have a normal outcome, or several exceptional outcomes.
An exceptional event is typically know but ancilliary to the program. It also
occurs with low frequency. Exceptions aren't necessarily errors!

Robustness is when you can't skip return codes.

Execution Environments
An exception handling mechanism depends on the execution environment.
e.g. Object-oriented environments have destructors that must be called when a
stack is unwound.
Note: uC++ has labelled breaks and finally clauses, like Java! Finally
clauses typically aren't replicatable in C++.

[Terminology slide]
** source vs. faulting execution, propogation, propogation mechanism


## Lecture 3
Every time you open a curly brace, a new stack frame is created. Even with code
blocks, it's like a function call, with its own local variables. When a block
is closed, the stack is unwound.
Or, you can think of curly braces as lambda functions.
Q: How do we know how many stackframes are unwound in a multi-level jump/exit?
A: We stored the target stackframe address of the label in our 'L' variable.

Static/Dynamic Call/Return

All routine/dynamic control-flow can be characterized by two properties:
1. static/dynamic call: routine/exception name at the call/raise is looked up
   statically (compile-time) or dynamically (runtime)
2. static/dynamic return: after a routine/handler completes, it returns to its
   static (definition) or dynamic (call) context

Memorize this table:
                                call/raise   
return/handled       static                    dynamic
        static   1) sequel            3) termination exception
       dynamic   2) routine      4) routine pointer, virtual routine, resumption

So we have that calls are static but returns are dynamic. We can statically
know where the call is going, but only dynamically know where it will return.

A routine, say "foo", is essentially a constant when defined. We can also have
a routine pointer that can point to any routine with a matching signature.

Note: Virtual routines must be dynamic calls. So is resumption, which we will
get to later.

What is a static return? We must know statically where that routine will be
returning to. It sounds strange, but we've done it many times!

Static Propogation (Sequel) (Bob Tenant)
- special type of routine
- cannot return a value
- replaces a break statement in our example
- when you get to the closing curly brace, it doesn't return to the call, it
  returns to the end of the block in which the sequel is defined

e.g.
for ( ;; ) {
    sequel S1(...) { ... }
    ...
}
// sequel S1 jumps here, terminating the block it's inside of

Note: S1 is not visible outside of that block, as expected.

Note: A sequel defined in a simple, one-block program will kill the program.

Note: Some languages don't allow nested function definition, but it is very
useful! In fact, most (850/900) languages support nested routines. C++11
supports nested routines via lambdas.

The advantage of sequels is that they are statically known.

Advantages of dynamic call: I can find things on the stack, I can restructure a
program as it's running.

Now to block 4 of the chart.
I'll make you think more deeply about throw, try, catch [slide 42]
Example code: multiple try/catch blocks, each calling f(), with a struct E {}
defined above that we throw and catch. This is multilevel returning!

Note: Throw is essentially a function call. We call the handler (after a bit of
a search). The handler resides in a block of code, and has it's own local
variables.

A catch is a sequel! At the end of the catch block, control goes to the next
line. Imagine the catch is inside the try block. Then it behaves like a sequel.
The stack unwinds as the exception "searches" for a handler. We can't go back
up the stack if we tried, because it unwound. This is a multi-level return!

A programming language, Eiffel, says that each block needs to end successfully
(fulfilling a contract). If it doesn't, we go to a handler that tries to fulfil
the contract until it's right, restarting the block until it works.

This is like a retry statement.
e.g.
    try () {}
    catch () {
        goto finished;
        }

equivalent to:
    while (true) {
        try () {}
        catch() {
            if () break;
        }
    }

C++ I/O can be toggled to raise exceptions versus return codes.
i.e. We use try/catch blocks instead of function calls and working with the
returned value. EoF is then an exception that we can catch.

That's exactly what we want to do in this course, because we want to avoid flag
variables and return codes.

In uC++, there are no return codes! We are forcing you to use exceptions.
Note: We have uFile::Failure, not EoF.
Note: We can avoid doing this at first but must transition eventually. So just
do it.

Examples directory has a main routine opening and closing an output file that
nets us 15% of our marks out of the box!

"Now, a couple of Mythbusters. There are some common mistakes out there."
1. There can only be one exception going at a time.
WRONG! Any number of exceptions can be in effect simultaneously. Handlers can
throw exceptions. Even though the exception is caught, we can rethrow it.

2. A destructor can't raise an exception in C++.
WRONG! See code on slide 47 (catch statement after a block is closed and a
destructor is called).
You cannot do it if and only if another exception is being propogated.
e.g. An exception is unwinding the stack where a destructor wants to throw
another exception before the last one is caught.
Q: What if there's a local catch block in the destructor?
A: Yes, that will still work.
Note: This is a runtime check.

Resumption
Not many languages have resumption (100/900).
A corrective action.
See example on slide 50.
Instead of doing a throw and a catch, we do a resume and a catchresume.
This is a dynamic call, it looks down the stack for a matching catchresume.
It has to search rather than do a direct transfer.
After catchresume, it continues after the resume instruction.
The stack is not unwound!
This allows us to do a "fixup" routine, but without awkwardly passing such
routines around. It's fixup on steroids.
Key: dynamic call and dynamic return, but this one has a search whereas other
"box 4" situations have direct transfers.

Note: A1Q2 (corrected) dares us to rewrite code with catchresumes.
(But it cannot be done.)

Note: We can _underscore events and resumes.

Resume calls can pass pointers, and catchResume can edit exactly that data back
up the stack.

Tangent: While a resumption handler remains on the stack, once it catches an
exception, it is not reused until it has completed. Because the stack is not
unwinding, the try block has been unwound.

Note: slide 53 is important for our assignment ;)
Every time we enter a try block, we save a label variable, then we can update
it, and eventually restore it.
"Entering a try block has cost."
Remember that exceptions should be rare. But Bjorne insisted that try blocks
should be cost-free, so people moved heaven and earth to do this in C++. So
exceptions in C++ are mind-bogglingly slow.
"Zero-cost guarded block entry."

[slide 60] Guarded and unguarded blocks
"Guarded blocks have handlers."
You have everything you need now to do q1 and q2 on the assignment.


## Lecture 4
2.11 Additional Features
Derived exception types - provides a hierarchy of exception types, and allows
polymorphism.
Recall: Coupling and cohesion. This exception hierarchy provides decoupling.
e.g.
 - Exception
   - Arithmetic
     - Divide-by-zero
     - ...
   - IO
     - File error
     - ...

So we can throw a child exception structure, and catch the parent. BUT if the
child struct is larger, it will be truncated to the parent object. Catching
exceptions by reference is a common idiom in C++ because of this inheritance.

Catch-Any
A mechanism to match any exception propogating through a guarded block.
For termination, catch-any is used for general cleanup of nonspecific errors.
uC++ does have a finally clause for try/catch block that essentially does this.

Exception Parameters
Information is essential to recovering from exceptions. Exception parameters
allow us to pass information to catch/resume statements.
Without this feature (along with control flow), exceptions are pointless. It's
essential.

2 Camps, no middle ground:
1. Static check that makes sure that if g is said to throw E, then calling g is
   statically checked for an E handler.
   i.e. Exception list: part of a routine's prototype specifying which
   exception types may propagate from the routine to its caller.
2. We'll let you compile anyway, assume you won't throw an E, and that's okay.
   This is the dynamic check.

Checked/unchecked exception-type (Java, inheritance-based, static checked).
 - Java has static checks. It drives people crazy by insisting that you handle
   obscure exceptions.
 - But then, the Java programmer has two options: silently catch the obscure
   exception and essentially ignore it, or catch the obscure exception and
   throw it again, letting someone else deal with it.

Although checked exception types are good practice for software engineering and
robust software, reuse is precluded. If we must handle all exceptions, then we
limit polymorphism. Children of expected objects in our function's argument may
throw novel exceptions, and break our contract!

So we either allow contracts and break code reuse, or we favour code reuse.
There is no middle ground!

Note: In C/C++, proprietary libraries do not give you any insight into
underlying exception causes! This is another tertiary issue that arises with
exceptions outside of Javaland.

Determining an exception list for concurrent programs is essentially impossible
in Java, despite their efforts.


Coroutines
 - A is gonna call B, B is gonna call C, and C goes back to B, but C doesn't go
   away. B can then go back to C but not from the beginning. It starts it in
   the middle!
 - You've written (or wanted to write) code like this in the past!

Example:
1. Cocaller cocalls a coroutine. Cocaller becomes inactive, and coroutine
   becomes active. Cocaller's state (line 10) is stored.
2. Coroutine suspends at line 15, returning control to the cocaller (at line
   10). State of coroutine = line 15.
3. Cocaller reaches line 20 and resumes (not calls) the coroutine, at line 15.
   We store the cocaller's state (line 20).
4. Do this again a couple times.
5. Coroutine finally does a normal return, and terminates. It's now gone, and
   can't be resumed.

The state of a coroutine consists of:
 - Execution location
 - Execution state
 - execution status (active or inactive)

Note: Coroutines have nothing to do with concurrency. Coroutines are all
sequential. We haven't crossed the magical line into concourrency yet. People
claiming otherwise are wrong.
We have, however, moved into a place where we have multiple stacks.

2 types of coroutine: semi and full. It depends on how a coroutine is used;
they are both technically coroutines. Kinda like how some routines are
recursive, based on how they are used.

Semi-Coroutines are easier, since Full-coroutines resemble recursion and are
harder.

Semi-Coroutine Example:
Fibonacci sequence [slide 78].
We can all solve this problem trivially in first year.
Down the road, we learn the software engineering way of bundling up the code
into a routine in a library that we can sell.

But if we abstract away the fibonacci() function, it starts from the beginning
each time! It can't remember information between calls. So we add global
variables above the function declaration - bad! This limits us to one fibonacci
function, as there is only one instance of the global variables.

We also need to remember which case of the fibonacci algorithm we are in. This
means we use a flag variable - bad!

Next iteration: We create a class, so that multiple instances can be created.
But flag variables are still used.

Best iteration: A coroutine! It suspends after each case, so state is
implicitly stored in the coroutine state. The next() routine simply resumes the
coroutine, eventually looping within the `for ( ;; )` loop.

Multiple coroutine declarations have their own stacks, so multiple fibonacci
generators can exist at the same time!

Notice: In a coroutine, we have a "main" routine, where execution starts, and
calls the coroutine (on its own stack).

Note: A coroutine has every property of a class. It's exacly the same as a
class. But it has extra features!

Note: Hidden base class (uBaseCoroutine) provides basic suspend() and resume()
functions.

Now that main isn't available to start off our programs, uMain does that. We
are in charge of implementing uMain::main(). We will also return a uRetCode.
C++ includes should be used as needed. uC++ should work out of the box with
these constraints.

Note: A coroutine's signature always has no arguments and returns void.

Note: A coroutine has a blank signature because we interact with interface
routines - public members of the class - instead. Those public members interact
with our coroutine.

Note: You do not have to run off the end of a coroutine to destroy it. If a
main function finishes while some of its coroutines are suspended, they are
destroyed! Like if we suspend our uMain::main() program in our assignment ;)

"Talking" to coroutines requires a "communication variable" inside the class
that can be edited by both main and the coroutine. 

Main is typically private, because we don't want to cocall our main routine. We
interact with main via our interface. All of our resumes then belong in our
interface routines.

If you interact with an object and it never resumes, the object is not a
coroutine. You get one chance to start your main, and once it's done it's done.
We made this a design decision for educational reasons, because most of the
time when main is restarted its by accident.

Resume works because of context switching! :D


## Lecture 5

Recall: Coroutines - very powerful C++ class for us to use. Everything you can
do with a class you can take advantage of with coroutines.

uC++ has control of main. We get uMain::main() and its coroutine uMain.

Today's class is all about coroutines for a1q3 this weekend!

_Coroutine Fib {
    int fn;
    void main() {
        ...
    }
}

_Coroutine is a wrapper class - the main() inside is technically the coroutine.
The `int fn` above is a "communication variable" to track state that is
referenced by the coroutine between suspends/resumes.

We typically keep _Coroutine::main private (or protected if we want people to
replace it down the road.

Public helper members are in charge of resuming the coroutine. Not that we
don't even have to resume the coroutine - we could return a precomputed value,
etc. When we run off the end of main, main goes away, but the object persists
(but will error out if we try to resume).

The top of each deactivated stack will tell you where execution will resume.
The final stackframe is the suspend call.

Objects become a coroutine on the first resume, and become objects again when
main finishes.

Example: Formatted output.
Code: 4 nested for loops with a multi-level exit.

Notice that the exit statement is heavily outdented to give you amazing eye
candy. "The magic is right in your face!"

You would have written it with ugly flag variables, and linearized the looped
program by adding lots of if statements.

"Indentation is a social experience. It's not to make the code compile.  I
taught blind programmers once. They just keep writiiiiiing! When they finish
the program, that's their end of line!"

"You can write any (non-trivial) program with just one loop and a bunch of flag
variables. Outer loops can be condensed into if statements inside one loop."

Notice that we didn't actually add flag variables, but we have some basic
problems:
 - global variables
 - only one instance at a time

Next progression: Make the formatter a class. But it still has linearized code
inside!
We also put the special final EOL case in our destructor. "Cute!" ^^

With our coroutine, we'll put the suspend() statement on the innermost block of
the for loop, right before we accept a character. Then we put the first resume
call in the constructor! This is a great way to prime our coroutine before the
first user call.

If the condition is at the bottom of the loop, it's a do-while. Top is while.
Middle is multi-exit. So our `for ( ;; )` loop with an if statement at the
bottom is equivalent to a do-while. But we have the advantage of being able to
rearrange our conditions without adding or removing any characters! So we can
debug purely our code rearranging, without worrying about introducing errors of
changing while to do-while, etc.

"The key here [slide 100] is that this is forming some sort of template for
what we need on our assignment!"

Note: g and b are class variables, not local variables to main, because we
decided to use them from the destructor.

Slide 101:
Notice how when uMain calls format.print( ch ), it is calling the member
routine of a coroutine. The print routine stack frame is on uMain's stack!
So how do variables in the callframe get referenced by the Format object?
Recall that `this` is passed into every member object call.

Correct Coroutine Usage
"We'll provide a style guide, but you'll probably get hit hard on the first
assignment because you don't have any experience writing coroutines. You must
find the zen of the coroutine. Zen can't be taught. The thing about experience
is that you have to experience it. It's like giving you a style guide to life -
it just doesn't work."

Zen: Eliminate computation or flag variables retaining information about the
execution state.
e.g. Keeping track of parity of an input counter just by being after or before
the first suspend.
"Once I get restarted, what knowledge do I have?"
"The fact that I have simply woken up at this point tells me everything about
 the universe."
"You will desperately want to put in if statements, but no! You need almost no
if statements. If you suspend in the right places, you will know exactly what
you've seen and exactly where you are."

i.e. Implicit execution state > explicit execution state.

Look at this switch case code. You say, "sure, coroutines need to have a
suspend call" and write something like this. It works, but no zen! Negative
zen! Bad fen shoi! It's clearly facing the wrong direction.

"I think I have two or three suspends in my solution for the floating point
question."

Q: "So essentially if there is a special/corner case that only comes up once,
    you want us to put in the statement then call suspend?"
A: "Yes."

How should you attack question 3?
[Coroutine construction slide 104]
Read over a1q3 and pretend you're in first year. How would you solve it back
then? Write nested loops, etc. at first before making it into a coroutine. Then
in less than five minutes you can get that working program into a coroutine.
Write helper routines, etc. as you would in first year.

1. Put processing code into coroutine main.
2. Convert reads (program is consuming) or writes (program is producing) to
   suspend. Your assignment is a little of both though ;)
3. Use interface members and communication variables to transfer in/out of the
   coroutine.

One extra notch of difficulty in your assignment: some exceptions inside of the
question. On the one hand we're dealing with coroutines, then we bolt on some
more interesting and complicated exception handling.

We have one coroutine throw an exception to another coroutine!

4.5 Nonlocal Exceptions
We _Throw _Event structures and the caller of a member routine _CatchResumes a
uBaseCoroutine::UnhandledException.
Notice how we have `_Throw E();` outside of a try block - an unguarded block!
At the end of our Coroutine program, we would expect it to terminate, but no!
The base of the coroutine stack unwinds and throws the exception back to the
resumer.
Since resumers don't know what to expect (don't know E), we put E inside of the
general UnhandledException.

Note: The UnhandledException is changed to a resumption when it hits the bottom
of the coroutine's stack. So the resumer catches it with a CatchResume.
We can also catch it with a `catch` statement, but we'll get to that later!

Example that is the opposite of what we need to do on the assignment (thrower
becomes resumer) [slide 155]
We use _Enable to allow nonlocal exceptions, then do `Resume E() _At c;`.
The exception can't be delivered until we resume the coroutine with a member
routine, even when we throw an exception at it with _Resume.

When a coroutine starts up, exceptions can't be thrown at it, because we aren't
necessarily at a try block. So when coroutines start, their main functions have
nonlocal exceptions turned off by default. So we _Enable them inside of a try
block, where we have a handler! Then we are outside of the try { _Enable { ...
} } block, and won't be interrupted while we handle the nonlocal exception in
our catch block.

Common mistake: At least 10 bozos on the newsgroup will do this: Don't forget
to add _Enable in your coroutine!


## Lecture 6

Recall: Coroutine Construction - writing the code straight up then converting
it to a coroutine is easier, isn't it? We won't be able to do that when things
get more complicated, though.

Memory Management
Normal stack
STACK-> FREE <-HEAP

Multiple coroutine stacks
STACK1-> FREE <-STACK2<-STACK3<-HEAP

Recall: Declaring _Coroutine gives you magical inheritance of a bunch of useful
routines.
Hint: Coroutine has a "resumer" that you can throw exceptions at in a1q3.

uC++ stack was 64K (K, not M!) for the longest time. I upped it to 256K this
term.
Coroutines have fixed-sized stacks, so we don't want to allocate very large
data structures on the stack.
Note: There's a "guard page" on top of our stack that is watched, to throw
segfaults if we overflow. It's possible to jump over it, but hard to do.


Same Fringe
Two binary trees have the same fringe if all leaves are equal from left to
right.
Recall: Tree traversal (prefix, infix, and postfix).
"We love recursion for tree traversal. We get a stack for free--stackframes
pile up on the stack, giving us a free 'stack' data structure."
No direct solution to same-fringe without additional data structures; it
requires an iterator to keep track.
Notice how beautifully we can solve this with a coroutine: walk the tree to a
leaf, then suspend. If we need to resume, we're already close to the next leaf!
How do we use this?
Iterator for each of two trees. Each iterator checks each leaf from left to
right, which the caller can check stepwise.

Where else do we use coroutines?
Device Drivers
Crucially important; ~40% of OS code; 30-40% of OS errors originate from device
drivers; "amazingly difficult to write correctly".
 - Incorrect memory access means corrupting the operating system!
 - Also, a "rat's nest of code". OS calls driver to push out data and expects
   data to be returned. But drivers that aren't coroutines won't remember where
   they left off, and get convoluted."
e.g. Transmission protocol
STX ... message ... ESCETX ... ETX 2-byte CRC (checksum) "cyclic redundancy
check"

"For those of you who have taken Networks, all those protocols are essentially
finite-state machines, which can be implemented trivially as coroutines!"
"Some device drivers can have a few hundred states! Then it becomes a rat's
nest."

Producer-Consumer Pattern
Very common scenario in CS; many variations.
Can sometimes be handled very nicely by a coroutine.
[code example slide 114/115/116]
Note: Cons::p1 == this.p1 in member routine; it's the same scope.
Q: When the consumer is done, where was the last resume()?
A: In stop()!

Q: When the producer stops, where was the last resume?
A: In start(), called by uMain!

On slide 116: There is no suspend in coroutine::main! That means it probably
doesn't need to be a coroutine, unlike the code on slide 115. But this is just
a lead-up to the code that DOES need to both be coroutines later.

I lied! When you run off the end of a coroutine, it goes back to the *starter*
not its *last resumer*. Starter and resumer are usually the same, in 80-90% of
semi-coroutines.
The starter can pass on a pointer to you. You need to pass the coroutine to a
different couroutine in order to have a different resumer than starter.
If starter left and is out of scope, and you try to go back to the starter by
running off the end, you'll be fine. The other resumer can delete you safely.


Full Coroutines.
Sequential routines are "sticks" - linear calls, returns.
Semi-coroutine - branched out to different stacks, still doing calls, returns,
also suspends, resumes. A tree!
One more step: Turn tree into a network. Put cycles into the tree.
A full coroutine can do calls and returns, resumes and suspends, and
resume-resumes. "I can resume you, and you can resume back to me."
"It's the presence of the cycle that moves you to a full coroutine."
"Its still a coroutine, just with a cycle."

Before seeing a full coroutine example, let's define suspend and resume
formally. Both trigger a context switch (between stacks). Whichever coroutine
starts the context switch will go to sleep. The other side's behaviour
differentiates the two.
Special `uThisCoroutine` says, "you're in this coroutine".
Suspend goes back to the last resumer.
Resume activates the 'this'--whatever coroutine that you're currently inside of
is the one that's going to be made active.
Prod.start(): 'this' is the Prod object. But this.start callframe is on uMain's
stack. So uThisCoroutine==uMain becomes inactive, we step off of uMain's stack
and onto prod's stack. Last resumer becomes uThisCoroutine==uMain.
If 'this' variable isn't a coroutine, (say, a class), it doesn't have a stack
to switch to.

Our first full coroutine: Fc [slide 120].
1. uMain calls fc.Mem(); uThisCoroutine==uMain
2. Fc::main is called (first resume starts coroutine.
   - calls mem()
   - this==Fc and uThisCoroutine==Fc
   - makes yourself inactive, and active again
   - a one-node cycle!
3. control returns to end of mem(), resume() gets called again, same cycle
4. suspend()
   - last resumer is myself (Fc) -- this is unintuitive! You probably though
     this should be uMain, yes?
   - So we made it so! If you resume to yourself, we don't overwrite
     lastResumer with yourself
   - "resuming yourself is considered a noop"

How do we build our first non-trivial full coroutine?
"We can build an N-dimensional hypergraph of coroutines, and hop around in
there."
Three phases to any full coroutine:
1. starting the cycle
2. executing the cycle
3. stopping the cycle
"x needs to know about y, and y needs to know about x"
"But how do we prevent definition before use errors?"
"Use a constructor without an argument, then a partner routine that closes the
cycle and assigns the first coroutine's partner."
"It's a two-step with a partner."

Ending the cycle also gets complicated, if ping and pong are resuming each
other. If one stops, it goes to its resumer, then we get stuck if that resumer
tries to resume back to pong that's dead. We want our coroutines to run off the
end so that we get back to our starter (uMain). This idea of going back to your
starter works well! 80-90%, other 10% is hard.


## Lecture 7
Recall: The coroutine remembers its starter and its last resumer. There is
nothing special about creation.
Example: Ping and Pong coroutines. They can actually be the same coroutine!
uMain creates ping instance and pong instance, and connected them together by
passing in variables. One constructor gets a "partner", and the other doesn't.
Then the partner routine is called to close the cycle (and do the first
resume).
The creator ends up destroying the two coroutines.
If ping returns first, it returns to starter (uMain) and then uMain runs off
the end, so it destructs pong.
If pong returns first, it goes back to ping since ping started pong. Then ping
is done its N cycles, so it returns to uMain, and both coroutines are already
terminated. That isn't a requirement, but it's nice and clean.

Full coroutine example with Producer/Consumer (partners are different types).
Consumer constructor gets a partner by a reference variable.
"Ping is Prod and Pong is Cons; the picture on slide [125?] is identical."
When Prod does a c->delivery(rand, rand), we drop off two random items in cons'
place. Then we do a resume. Since we are in cons' place, we wake up cons.
Then cons goes over to prod's place to make a payment, and calls resume, we
wake up prod. But prod is in cons' delivey routine.
Then when prod wakes up and returns to prod, cons is asleep in prod's payment
routine.

"When we do full coroutines, we go to sleep in somebody else's place."
"We wake up in someone else's object and then come back to your object."
"Full routines are hard to understand because you can go to sleep in other
people's places. You're adults, you can do that now!"

"On your assignment, you make a circle, and then a potato gets thrown into the
cycle, and eventually the potato explodes, whoever has it has to leave the
circle."

Note: With the ping pong cycle, calling suspend instead of resume gets the
cycle going the other direction!

Coroutine Languages
Two forms:
1. Stackless (AKA iterators, generators) use the caller's stack and a
   fixed-sized stack
2. Stackful (separate stack and a fixed-sized local-state)

Python 3.4.1: Stackless, semi-coroutines, routine versus class, no calls,
single interface.
 - yield suspends (and sends a value)
 - (yield) receives a value
 - next resumes

C++ Boost Library (V1.61)
 - stackfull, semi/full coroutines, routine versus class, single interface
 - return activates creator
 - full coroutine uses resume(partner) ...

uC++ EHM
 - Exceptions are limited to _Error exception base class type
 - Only exception types can go into the exception hierarchy
 - Supports two forms of raising: throwing and resuming
 - _Event is just a class with some special properties.
 - Each exception type inherits members from uBaseEvent
 - You can write your own defaultResume and defaultTerminate routines.
 - In C++, you can get truncation in throws AND catches.
   - Recall: catch by pointer or reference
   - But in uC++, rethrowing a caught base class will throw the child class as
     expected. [145]
 - You can use as many CatchResumes as you want, but they must precede any
   catch handlers
 - Note: if we can't do the fixup in a CatchResume, don't just step out with
   goto (a static return), do a throw! (Dynamic return, as expected)


## Lecture 8

Recall: uC++ exception handling
defaultResume called if no valid catchResumes exist, and that throws the error
instead. So you can catch Resumes with just a catch statement.

Nonlocal Exceptions are disabled by default.
Certain base exceptions are enabled by default (they probably can't be
disabled).
Individual exception types can be enabled/disabled dynamically.
Multiple non-local exceptions are queued and delivered in FIFO order depending
on the current enabled exceptions.
Exception handling is complicated, especially between stacks. Resumption is
useful because it's safe.
"You can only resume non-locally; we took the throw away from you."
[157] _Resume Stop() _At \*c; // "We throw a resume at the consumer"

"Now it's time for the C word!"
Concurrency
"Coroutines are absolutely different than concurrency."
"Coroutines taught us multiple stacks, under one thread of execution."

Concurrency has multiple threads.
Think of a thread as a ball moving through your program (sequentially).
Note: A thread is ethereal. It comes into existence, moves on its own, and
disappears after a while.

You don't create a thread, you create a "shell" that a thread grows from.
Like how you don't create a person--you procreate, and create something that a
person grows out of, however they want to!

Each one of us in this classroom could be seen as processes, with (hopefully)
at least one thread inside you--tasks we are thinking about, heart beating,
etc. One thread listening to me, one thread thinking about the weekend, etc.
I can't see inside your memory (unless I do a vulcan mind meld).

A task is a "light weight program" (LWP); it's similar to a process except that
it is reduced along some particular dimension (like a ship vs. a boat).

Parallel execution is when 2 or more operations occur simultaneously; can only
occur with multiple CPUs.

Concurrent execution is multiple threads. They execute sequentially in a
single-CPU environment, but can take advantage of parallel environments. We
don't know anything about the parallelism when we write the program--we just
make as many threads as we want.

Why write concurrent programs?
 - Dividing problem into executing threads is an important technique
   "another layer of modularization"
 - Expressing a problem with multiple executing threads may be the natural way
 - Increased performance!

"It is going to hurt. I guaruntee it."
"I once bought a concurrency textbook that said 'we will make it as easy as
 sequential programs' so I closed the book and dropped it in the trash."
"I brought some drop forms with me!"
Why concurrency is difficult
 - difficult to understand
   - esp. when interacting
 - difficult to specify: 
   - how can/should a problem be broken up so that parts can be solved at the
     same time as others
   - if interaction is necessary, what information must be communicated during
     the interaction?
 - difficult to debug:
   - concurrent operations proceed at varying speeds and in non-deterministic
     order, hence execution is not repeatable (Heisenburg)
     "resorting to sequential debugging, print statements, etc. will alter its
     behaviour"
   - reasoning about multiple streams or threads of execution and their
     interactions is much more complex than for a single thread
"I get nightmares, and wake up at 3 in the morning yelling RACE CONDITION! RACE
 CONDITION!"

Real-life example: Moving.
There is always a big object you can't carry yourself (the beer fridge!). You
need helpers. But helpers need food, etc.
 - How many helpers?
   - Say you have N friends and N items.
   - If you invite N friends, they each grab an item, and run to the door... a
     bottleneck!
   - If items require more than one person, you may use more than N friends.
 - Bottlenecks
   - The door out of the room, large items, etc.
   - Foutons that unfold when moving down stairs are the worst.

"When you are communicating, you are not working!"
We must communicate crisply and get back to work ASAP.

Concurrent Hardware
Huge step: switching may occur non-deterministically between any two (machine)
instructions.
Switching can happen explicitly (blocking) or implicit (OS timer goes off,
forces you to switch).
If interrupts affect scheduling it's called preemptive; else non-preemptive.
Pointers among tasks on one multiprocessor environment works because memory is
shared.
Pointers among tasks between systems doesn't work (distributed machines).
 - That takes two 400-level courses: networking (the wire) and distributed
   systems.

Review: Execution States
new -> (ready, running, blocked/waiting) -> halted
Timer alarm: running -> ready queue
Completion of I/O operation: blocked -> ready
Exceeding some limit: running -> halted (usually an error)
Error: running -> halted

"Programmers are all taking the program drug. We live for the 2% of the time
 when our programs finally work."

A non-deterministic ready<->running transition makes basic operations unsafe.
int i = 0;
i += 1; // task 1
-----------------
i += 1; // task 2

Depending on architecture, we may or may not have increment/decrement
instructions. It could be a load/store combo, which can be interrupted.
Recall: Context switches save registers to that task's stack. Changes can
overwrite each other.
Single-instruction increment/decrement commands are safe, but most things are
unsafe by default.
Question that's "on the street", worth two marks: What's the worst-case,
smallest number result of the operations on slide [168]?

Threading Model
Defines relationship between threads and CPUs.
OS manages CPUs, provides logical access via kernel threads (virtual
processors) scheduled across the CPUs.
The scheduler lives in the OS.
You can ask for more kernel threads (not CPUs). You THINK you are asking for
multiple CPUs, but remember: the OS owns the computer, not you.
CPU time is broken up among tasks.
Scheduler creates some concurrency inside the process that previously didn't
have any.
Notice: CPU-Scheduler-KernelThread is the same picture as
        KernelThread-Scheduler-Tasks/UserThreads
It's a recursion!
User tasks could even divide up time into nano-tasks.
Stackless threads are cheap to create and tear down, so nano-tasks are
good in certain situations.
We can go DOWN a level--multiple operating systems: virtualization!
We're working in the side red box--scheduling user tasks.

## Lecture 9
Concurrency continued
"You can ask for more than one kernel thread. That's what Java/C# do."
"Box 2 and 4 [slide 169] are the models you will encounter."
"Box 4, user threads (tasks) model is what uC++, Go, Erlang do."
Note: Numbers are userThreads:KernelThreads:CPUs
1:1 (kernel threading) is box 2. M:M is box 4.

Concurrent Systems (3 major types):
1. attempting to discover concurrency (parallelizing loops, etc.)
2. concurrency through implicit constructs "light lifting; not precise; helps
   the compiler"
3. programmer provides concurrency through explicit constructs

"Concurrency is complicated by nature. If someone says there is magic, slap
them on the side of the face with a fish. The only people who offer simple
solutions to complex problems are politicians!"

Speedup
Program speedup is Sc = T1 / Tc, where C is the number of CPUs and T1 is
sequential execution.
e.g. 1 CPU takes 10 seconds, T1=10; 4 takes 2.5s T4=2.5 (linear speedup).
Linear speedup is ideal. Super-linear is unlikely. Sub-linear is common, as-is
non-linear (where adding more stops helping).
Three factors affect speedup:
1. Amount of concurrency
2. Critical path among concurrency
3. scheduler efficiency

Each program has sequential and concurrent parts.
e.g. Sequentially read matrix, concurrently sum rows, sequentially total
subtotals

Amdahl's Law (Gene Amdahl): Concurrent section of program is P, making
sequential section 1-P, then maximum speedup using C CPUs is:
Sc = 1/((1-P)+P/C) where T1=1, Tc = sequential+concurrent

Example: 10% of program is sequential; 90% concurrent. Then max speedup Sc =
1/(1-0.9) = max speedup of 10.
"Big speedups of concurrent code barely make a dent. The sequential part still
 slows us down way more."
"The sequential part of your program has a huge effect, so we want to get that
 down."

Concurrent part of program is bounded in length by its critical path.
Concurrency can be affected by scheduling mechanism used, and other things
[177?].

Concurrency requires 3 mechanisms in a programming language.
1. Creation - cause another thread of control to come into existence.
2. Synchronization
 - "Let's meet at the Bomber at 5."
 - Putting up your hand in class
 - Where communication can happen--wait while exaclty 1 person talks, etc.
3. Communication - transmit data among threads

Thread creation must be a primitive operation; cannot be built from other
operations in a language. New construct: COBEGIN/COEND.
Between those two, instructions execute asynchronously. COEND is a
synchronization point.
We can nest COBEGIN and COENDs--it will work as expected.
We are restricted to creating a thread/process graph that is a lattice (tree
with a root on top and on bottom).
We can only do "last will and testament communication". We start you up and
wait for you to die, then you leave behind information for me to read.

Alternative: Start/Wait, Fork/Join. Allows arbitrary thread graphs (not
trees!). This is strictly more powerful than the tree-only approach above.
Start/Wait can simulate COBEGIN/COEND by waiting after starting a few threads,
but the reverse is not true.

Notice: We call Wait on different "handles" that we start funcitons on. This
lets us have multiples of the same functions running.

Of course, we are using C++, an OO language. So wouldn't it be nice if we used
objects to implement concurrency?

Thread Objects
[183]
Use object allocation/deallocation to define thread lifetime.
Exiting a block where a \_Task was allocated means we need to wait for it to
finish - emulating COBEGIN/COEND.
"Simple OO examples look overdone (COBEGIN/COEND are much simpler for simple
tasks) but OO will be much better later on."
Allocation (new) and deallocation (delete) emulates start/wait.
Gotcha: Use a temp variable to store main result. Set a communication variable
in the destructor, where we know the concurrency has finished.

Termination Synchronization "Last will and testament"
A thread terminates when:
 - it finishes normally
 - it finishes with an error
 - it is killed by its parent (or sibling) (not supported in uC++)
 - because the parent terminates (not supported in uC++)
 * because it encourages bad concurrent programming style "create threads then
   kill all of them with a large weapon"

Matrix Example
int rows = 10;
COFOR(row, 0, rows, {...}) "a way of unrolling N COBEGIN/COEND, with each
thread running the passed in block of code."

Synchronization and Communication during Execution (not just after death)
For a3q1 "matrix multiply between two matrices"
next class: Thursday, not Tuesday.
"Insert and remove flags between threads as they execute"

## Lecture 10
[slide 192]
Producer consumer problem with tasks.
Main difference between coroutines and tasks: tasks start on their own; we
don't have control over order and speed of execution.
How do we handle this? Put up flags when data has been inserted into the 'data'
variable. The remove flag is down, and the producer waits on the remove flag
(busy wait). This `while (!Remove) {}` loop is an infinite loop, but through
the magic of concurrency it breaks!

Cons:
    while (!insert) {} // busy wait
    insert = false;
    data = Data;
    Remove = true;

These are "busy loops" because we are "busily" checking for a change.

Exceptions
Can be local or non-local.
Note: uC++ has many subtle "detection points" where a routine is checked for
exceptions tacked onto it by someone else.
Coroutines need to be resumed, since they are inactive while you throw an
exception at them.
Tasks are already running, and can choose when to listen to an exception, "like
getting a phone call". We listen for exceptions inside of \_Enable blocks.

Three things needed for concurrency:
 -
 -
 -
One more thing: Critical Sections.
 - Shared (non-concurrent) objects must be operated on with atomic (can't be
   broken down) instructions.
 - A group of instructions that must be atomic is a critical section. We create
   critical sections with mutal exclusion code, surrounding the block.
As programmers, we need to learn how to detect critical sections.
 - One way to handle sharing is to serialize all access (one person looking at
   the blackboard at a time).
 - Better: multiple people reading, one person writing, don't write when people
   are reading.

Static Variables
 - 1000 instances of a class has one instance of the static variable.
 - There are 1000 separate instances of the other non-static variables.
 - 1000 tasks with a static variable shares it.
 - Instead of looping and passing unique IDs into a constructor, we can use a
   static variable to count how many objects have been instantiated, so we
   don't need to pass anything in to the constructor.
Takeaway: Just do the thing at the bottom! [slide 200] Do the loop thing!


The Mutual Exclusion Game
 - Win condition: Guaruntee that only one thread can be in the critical section
   at a time (Safety).
 - "Board games have rulebooks because young kids will just jump to the winning
   square!"
 - We can't control order or speed of execution.
 - A thread cannot hold the lock to the critical section and block out others.
 - The "you go first" rule. We can't postpone selection indefinitely
   (livelock). "We don't find Canadians passed out on the floor from saying,
   'no you go first' at a doorway. But computers are happy to get stuck in this
   loop."
 - We also want to prevent unfairness/starvation.
 - FIFO ordering is "unfairness". Bargers who go ahead are unfair.

Bathroom Example
    uBaseTask *CurrTid;
    ::CurrTid = &uThisTask();
    for (int i=0; i<100; i++) {
        work();
        if ( ::CurrTid != &uThisTask() ) uAbort("interference");
    }

Locks
If we aren't in a cheap bathroom without locks, we need to add some.
Bathroom = critical section. Mutual exclusion = lock.
"We are anthropomorphizing this example (giving human traits to inanimate
objects). But be careful doing this, because humans and computers solve
problems in different ways. Like how washing machines don't walk down to a
river and beat clothes on a rock--they spin inhumanly fast!"

If we just loop and check if the lock is unlocked, Mary and I could put our
hands on the lock at the same time, enter at the same time, and lock the door!
This breaks rule 1.

Alternation
"Blackboard on the inside of the door `while (::Last == me) {};
CriticalSection(); ::Last = me;`"
But if only you are home, you can't go two times in a row--alternation means
someone else needs to go in after you.
This satisfies rule 1 and 2 but breaks rule 3.

Declare Intent
    me = WantIn;
    while (you == WantIn) {}
    CriticalSection();
    me = DontWantIn;

But this leads to a livelock situation (breaks rule 4) when we both rush to the
bathroom at the same time and tick the "WantIn" box at the same time. Then we
both wait for the other person to retract their intent.

Retract Intent
 - Leads to intersection livelock situation; lockstep WantIn, DontWantIn, etc.
 - One line off breaks us out of this.

Prioritized Retract Intent
 - "Ladies first".
 - Breaks ties with priorities.
 - Breaks rule 5 (starvation). Mary could keep going into the bathroom and I
   wouldn't get in.

Theo Dekker (Netherlands) [210]
 - Djikstra was writing original seminal paper on concurrency.
 - Theo seemingly walked into Djikstra's office and solved a problem, and
   Djikstra published it. Dekker did get full credit though!
 - Theo bought a third blackboard.
   1 & 2: Prioritized Retract Intent (one way rule though)
   3: Whoever was last in the bathroom is low priority.
 - "Dekker's algorithm is fundamental to concurrency, and will be taught for
   thousands of years, long after Jobs and Gates are forgotten."
 - "You don't need a company to achieve immortality. You just need a line of
   code. Dekker's third blackboard."
 - Dekker's algorithm is not read-write safe. Reading before or after bits have
   been changed only; we don't see the dancing bits "scrambling and
   flickering".
   "My clicker has a cheap, 25-cent CPU. It isn't read-write safe."
 - Deckker's can be modified so that there is no w/w scramble or r/w flicker.
   "I was one of the authors on that paper."

Peterson
    me = WantIn;
    ::Last = &me; // Race!
    // If my name is in the variable, I lost the race, my hand is on top of
    // yours.
    while (you == WantIn && ::Last == &me ) {}
    CriticalSection();
    me = DontWantIn;
But, if you run a race by yourself and are a pessimist, you are a loser.
This breaks if we write half of each hand into the variable--it breaks when
writes aren't atomic.
"Dekker's algorithm works. Peterson's cheats!"


Lecture
*MIDTERM? If we give you a question on Dekker's question, chances are we broke
something ;)*

Peterson has bounded overtaking because the race loser doesn't retract intent.
Difference: Dekker doesn't have bounded overtaking.

"Now, let's jump from 1 thread to N threads."
Pretend that you are in residence. The bathroom at the end of the hall has a
blackboard for everyone. When you need to go, put a checkbox at your priority
level, and walk in the HIGH priority direction. If we see a higher priority, we
back down and wait for that higher priority person to finish. Then walk our
right, any lower priority threads will back down, but we need to wait on each
one that we hit for them to retract their intent.

Note: Lower-priority threads that arrive first will enter the bathroom, because
they don't see the higher priority threads.

This breaks rule 5 though, because an influx of high priority threads will
starve low priority threads.

Algorithm that statisfies all 5 rules: Bakery Hehner/Shy (based on Lampert)
You enter the bakery, grab a piece of paper, and ask everyone to hold up
theirs. You right the largest number plus 1 on your piece of paper. Position
breaks ties.
"Requires N bits of storage, though."

Ticket value 0 is special. It's a magic value. It means you are in the process
of selecting the ticket. 0 is the highest priority value, and other people wait
on us while we are getting our ticket.

We can walk by greater and equal tickets on our way to the bathroom (walking
from high priority position forwards to low priority (?)).

But this can break if the integer counter overflows. If the tickets al become
INT_MAX at some point, it automagically resets.

It is "probabalistically correct", since it's so hard to overflow the counter
(two threads can conspire to drive up the counter).

Note: You are assigned your position when you move in, and assign a 0. Then you
scan every ticket. It doesn't matter "where you walk from", you have to scan
them all.

The 19 at the end is lower position, so we win. Ticket is priority! Position is
tie breaker!

Simpler: Tournament
 - Tree of executing threads
 - "Slight amount of unfairness as well as your tree is perfectly balanced."
 - Retract execution in reverse order (red arrow) [221] (bottom-up)
   "You have to go from the bottom up when you leave the bathroom."
 - "Takes something extremely complicated (Bakery, Decker/Peterson?) and
   reduces it to a simple tree walk." 

Arbiter Algorithm
 - "Hire a full-time co-op student to manage (arbitrate) the bathroom."
 - Blackboard of people wanting to go into the bathroom. Arbiter cycles through
   blackboard and puts people into the bathroom as it passes their checkmark.
 - "We've turned mutual exclusion into synchronization by adding another thread.
    We have to decide if hiring someone is worth it, versus figuring out
    priority amongst ourselves in the other algorithms."
 - Sounds like a good idea, but the cost of another thread must be deemed
   worthwhile.

"These algorithms actually work. Some of the earliest cellphones used
Peterson's algorithm."
"That's it for software solutions."

Hardware Solutions (for the critical section problem)
 - eliminates shared information in software, communication among threads, etc.
 - cheat by making assumptions about execution impossible at the software level
 - balance must be struck between "thicker" instructions that don't take too
   much time, "must find the smallest thick instruction that does the most work"

Atomic read and write: Test and Set
Imagine C's ternary operator is atomic:
while(
    Lock == CLOSED ?                // Read!
        CLOSED : (Lock = CLOSED),   // Write!
        OPEN
)

N tasks, 1 bit of storage! An improvement!
In uC++, we have uTestSet

Everyone spinning sees CLOSED, except for one lucky person that sees OPEN and
immediately closes it. Hardware ensures that nobody else sees the OPEN.
Note: Memory bus manages traffic from multiple CPUs to memory. Atomic
instructions acting on memory must notify the bus to not let other CPUs access
it.

Swap
Swap a CLOSED "dummy" with the lock value until we get an OPEN value.
*MIDTERM? "The moment we know we have an atomic read and write, we know there's
a way to do an atomic test and set."*

Fetch and Increment
Increments value between read and write.

    while( FetchInc( Lock != 0 );
    // critical section
    Lock = 0;

"One lucky person sees the 0 and goes in, and everyone else keeps counting it."
Probabalistically correct. Very hard for this to overflow.
Use ticket counter to solve both problems (Bakery).

Note: CompareAndSwap taught earlier is really CompareAndAssign or
CompareAndSet. It doesn't swap values!

We would love to end the course here.
We can create threads, and provide mutual exclusion with flags.
But we can also do it with atomic instructions, with locks!
"An Operating System is just an atomic instruction and a linked list.
Everything else is extra."

Lock Abstraction

Two types of locks:
 - Spinning (busywaits)
   - no yield
   - yield
 - Blocking (no busywaits)
   - synchronization
     - condition
     - barrier
   - semaphore
     - binary
     - counting
   - mutex
     - owners
   - others...

"Underneath all of them is an atomic instruction or peterson's algorithm."

Spin Lock
"Eats up enormous amounts of CPU time."
while( TestSet( Lock ) == CLOSED ); // use up time-slice (no yield)
If instead you gave that time slice to the person in the bathroom, they will
get out faster! So just put yourself on the ready queue. `uThisTask().yield();`

uC++ has locks built in! We just need to use them. uLock (yielding) and
uSpinLock (non-yielding).
*MIDTERM "But if we take the covers off, we should know how the locks work."*

Note: Locks cannot be duplicated! Like a key that says, "DO NOT DUPLICATE".
So locks are always passed by pointer or by reference.

Synchronization pattern: Want one thread to do S1 before S2.

    // Set lock to closed (0) when spawning Tasks.
    S1                 lk.acquire();
    lk.release();      S2

Note: Lock value is closed after lk.acquire() executes. All the atomic
instruction things we looked at had the lock immediately set closed as soon as
it was opened.

Note: Never install a closed lock on a bathroom!

Note: Synchronization starts with the lock closed, one threa does acquire(),
one does release(). Mutual exclusion pattern has one thread do both, and the
lock starts open.

Two different critical sections are dependent if they touch the same variables.
e.g. Two critical sections both modify i,j,k

If two critical sections are independent, we need to pass in two locks.
"Next class, we finish off what you need for the last question on your
assignment."


Lecture
Recall: Spinning locks--either a tight loop with no yield, or a larger loop
with a yield.

Blocking Locks
"Sleeping beauty--you wake up when you get a big wet kiss."
What's in it for me in the bathroom scenario? Assume I don't mind rattling
the door. I get to sleep when I'm not in the bathroom anymore! Cooperation
leads to this shared benefit.

Note: You cannot assume that tasks are unblocked in FIFO order. The only
requirement of locks is to prevent starvation.

Mutex Lock
Used solely to provide mutual exclusion.
Tow kinds: single acquisition, multi acquisition (you hold on to the key and
can re-acquire as much as you want.
"With mult-acquisition, you can duck out of the bathroom if you forgot
 something in the bedroom. You can still get in, because you hold the lock."

Code example: [243] "blue code would not be there in a single-acquisition lock"
"include the blue code and it would be a multi-acquisition lock"
Note: `// RACE` - you begin racing with person you just woke up. "Why are you
waking me up?". Waker can be woken up, check the lock, and it's not available.
A barger! That's why we need a while loop. But when we have a while loop, we
almost always have bargers.

Naive solution: try yieldNoSchedule to preempt after lock.release(). But either
ordering of these two statements leads to problems.

Better solution: Modify yieldNoSchedule so that it takes in a lock. Then the
thread system decides who gets woken up. But this instruction isn't
preemptable.
"You can't interrupt the thing that does the thing!"
"The system is allowed to cheat a little bit."

Recall: Rule 3 says that you CAN control the speed and order of execution while
in the exit/entry blocks, so this is okay.

Alternatively: We hold onto the lock while we're in the bathroom. Then we hold
onto it when we walk out, and conceptually give it to you and wake you up.
"This is what we call passing the lock."
"Through the magic of concurrency, when you wake up, you are holding the flag
 and the lock."
"Coming off the back of the blocked queue is like getting resumed after
 suspending!"
"After waking you up, I am NOT allowed to do anything. I have essentially
 passed control to you."

A.K.A "Baton Passing"

But how do we know if someone is in the critical section?
We keep the person in the critical section on the bench. Then we don't need a
flag.

Note: So we don't get interrupted by exceptions while working with a lock, we
put lock.release() in the finally clause.
Or, we use RAII and release locks in the object's destructor.

Stream Locks
"Used to lock system I/O."
Concurrent use of C++ streams can produce unpredictable results.
uC++ supplies osacquire/isacquire for output and input stream locking.
`osaquire( cout ) << "abc" << endl;`

Note: Cout is buffered output, so writing to it can get muddled more easily
than cerr, which is probably more robust to "abuvwc defxyz" issues.

Synchronization Lock
Used solely to block tasks waiting for synchronization.
 - Acquiring task always blocks (no state to make it conditional)
 - Release is lost when no waiting task
 - Often called a condition lock, with wait/signal(notify) for acquire/release.
 - "A very dumb lock. The only state is a list of blocked tasks."

Usage Pattern:
 - Cannot enter a restaurant if all tables are full.
 - Must acquire a lock to check for empty tables. "Because the state could
   change."
 - If no free table, block on waiting-list until a table becomes available.
"The bench sitting in the front of the restaurant is a synchronization lock."
"What's your favourite sushi restaurant? Sushi 99? I like Sushi 99. Oops, no,
 no ads in this class!"
 - On our way out of the restaurant, we acquire the lock again, reset the flag,
   and wake someone up on the way out.
 - Reccurring issue: if you fall asleep with the lock when denied entry into
   the restaurant, you block everyone in the restaurant!
 - To add safely, we must pass in a mutex lock we must acquire before checking
   the blocking lock. (?)

[243]
This is still a while loop! We are busy waiting.
But now we eliminate while loops--if we are woken up, go straight in. Rely on
cooperation.

*MIDTERM?*
Has the busy wait been removed from the blocking lock?
No! We use a spinlock inside! The only way to build a blocking lock is with
mutual exclusion, and mutual exclusion must be gained fundamentally via a
spinlock.
"I told you you use GOTOs. Remember? If there was a centre of the universe, you
know what would be there? A spinlock!"

Why is a blocking lock so much more complex than spinlocks?
 - We used two instructions to do synchronization with spinning locks.
 - But we use a lot more code [266] to do synchronization with blocking locks.
 + Win: we don't waste time spinning!
Q: But why is blocking faster, if it uses a spinlock inside? Something is wrong
with this picture! Spinlocks waste CPU => blocking locks use spinlocks... but
blocking locks don't waste time?
A: We use spinlocks for a different purpose! Only to acquire the lock.
CPUs 2,3,4 aren't nailed to the wall spinning while CPU 1 goes through a
1000-line critical section. They just spin long enough to check the lock, then
sleep.

Concurrent lock usage example: [slide 266]
A3Q3 is now doable! Use these techniques to build yourself some protection, in
two ways:
 - First with [243] while (...) { lock.acquire(); }, but it has starvation.
 - Then we time it.
 - Then we rebuild it without while loops, as in [247]. But it's tricky to do
   that! We have to use two condition variables.
 - Then run it again, and compare.


## Lecture Oct 25 2016
Barriers
start.block()
...
end.block()
"Controller" task waits for everyone to  be ready to start, then waits for
everyone to end.

uBarrier is a "Monitor" (we'll get to those later). For now, just think of it
as a coroutine. If you just use the block method, the coroutine part doesn't
matter.

Recall: Matrix multiplication tasks by rows. We can use a barrier to collect
all the subtasks in the controller.
 - Set barrier to number of rows.


Binary Semaphores
 - Comes from communication from waving flags up and down on a hill
   ("semaphoring").
 - To acquire a lock, we Pee on it.
 - Higher cost for synchronization if external lock is required.


Counting Semaphores
 - Conceptual jump from simple critical sections (1 thread at a time) to
   complex critical sections (multiple threads at a time).
 - Lock has more than just open and closed states.
 - Imagine a washroom with three stalls, and a sign on the door displays how
   many free stalls there are.
 - Take our class for example. We needed a lock on the door at some point to
   prevent too many people from coming in!
   "I had to turn into the lock and yell at people!"
 - Note: We still need to keep our (binary) locks on the bathroom stall doors,
   to prevent 2-3 people from entering the same stall!
 - We can use our semaphore todo both mutual exclusion and synchronization.

Lock Programming
Precedence Graph
"Now we know the maximum possible speedup I can ever hope for."
Recall: Semi-coroutines form a "latice" (one top route and one bottom route).
We can only improve the performance of [slide 289] by a factor of 2 tops.
Semaphores and COBEGIN/COEND are as powerful as start/wait!
 - Do a line of code, then put a flag up saying, "I'm done!" (V()).
 - Wait your turn for the proper flags to go up.
 - But this will deadlock!
 - I have forgotten one important thing about locks:
   - I have remembered to release the locks.
   - I have remembered to close the locks.
   - But when you acquire a lock, it immediately closes!
     - Two threads will race to acquire L1 and get stuck.
     - If this was a counting semaphore, just call V(L1) twice in the first
       line.
       - By the end, L1 would be 1 and the others would be 0.
       - Just pee on the lock to reset it.
     - If you add another lock, they will all be 0.
     - But both approaches are equally good!

Note: Don't get process graphs and precedence graphs confused!
 - Process graph is the lattice thing on slide 290
 - Precendence graph is the ordering dependence thing.

MIDTERM MATERIAL COVERED UP TO HERE + A1, A2, A3

Lecture
Recall: Precedence graph; things that must occur in a particular order.

Buffering
"I was watching on of those home improvement shows. They had a wallpaper
machine that printed out wallpaper designs on the fly. But there can't be
infinite paper! So they went to the back of the machine, and there was a roll
of paper! Between th epaper and the printer, there were rollers that took up
slack. When the roll ran out and needed changing, the rollers would move in to
take up less slack while the roll was changed/taped in seamlessly!"

[slide 292] Buffer with counting semaphore.
"People desperately want to use if statements with semaphores. But no need!
Semaphores already have if statements inside."

We are safe except for when the only item in the buffer is being partially
inserted by a producer. A consumer could see it and rip it out partially, break
it, etc. before the item is all the way in.

But we handle that case! We don't V on the semaphore until AFTER the item is
completely absolutely inserted in the buffer.

"In concurrency we must NEVER say something has happened until after it has
 happened. Not like going to the Bomber and saying, 'I'm done my 343
 assignment. Yeah!'"

Note: This semaphore adds *synchronization*.

Adding a second "empty" semaphore allows producer to wait for empty slots.
Graphical example of this happeneing: [295]

Goal: Have buffer roughly half full most of the time.
"If your buffer is always full or always empty, you don't need a buffer."
The sum of 'full' and 'empty' always equals the size of the buffer.

We don't need any ifs, checks, or blocks! Just two semaphores! Zen! In the
Bomber!

Note: Some objects can't be copied, so they must be consumed in the buffer.

If a buffer is always full, we could:
 - Throttle the producer (silly)
 - Add more consumers (good!)
If a buffer is always empty, hire more producers.

But this breaks!
There isn't any mutual exclusion on adding and removing items. Consumers could
grab the same item; producers could try to insert into the same position.

So we need to add locks for adding and removing.
One lock or two?
STL data structures can handle simultaneous insert/delete--they handle the
shared structure. But this is system-dependent and case-by-case.
Air on the side of caution.

Lock Techniques
"Over time, concurrent programmers have developed patterns."
Recall: The readers and writers problem.

Two patterns: Split Binary Semaphores and Baton Passing

Split binary semaphore is a collection of semaphores where at most one of the
colleciton has the value 1. This lets us differentiate tasks. "Sit hte girls on
one bench and the boys on another."

Baton passing: There is no baton. It's a conceptual thing. Main goal is to
prevent barging. Acquiring a lock is picking it up. Releasing it is putting it
down. Passing it is signaling a thread that is waiting on the ready queue.

Now the reading/writing problem. Put readers on one "bench" and writers on
another. [slide 305]
One writer can be in at a time. Readers are party time, and we don't control
the order and speed of execution, they can read whatever they want and leave
whenever.
Note: Writers can also read, to make decisions.

Whenever the person leaving has to do more work, it's called cooperation!
A writer leaving must pick up the baton, and wake up another writer, or the
readers.

If this was an assignment question, how would we start?
 - There are three benches: Readers, Writers, and Arrivers.
 - Three benches mean three locks, but we treat them as one.
 - uSemaphore entry_q(1), reader_q(0), writer_q(0); // 3 binary semaphores
 - Counters: How many writers are in the room, how many readers in the room,
   and the size of the reader bench and the writer bench. 4 auxillary counter
   variables. We don't care about the size of the Arrivers bench.
Note: Reader who waited on the bench entered the start of the read_q.P() and
exits out the back of that statement. Cooperation means we don't need mroe
checks there.

This is an example of "daisy chain signalling", where each thread signals.

When you are at a party, you don't want to be the last one there. Because then
you have to help clean up! If you leave 15 minutes earlier, you can go find
another party ;)

If you are the last reader in the room, you must wake up a writer, update
counts, etc.
A writer leaving the room checks for readers, and wakes one up (who then
daisy-chain wake up the others), otherwise you look at the writers bench to
wake one up. If benches are empty, put down the baton. Boom!

This is okay, except for the starvation!
If readers keep coming in, the party is out of control!
If 80% are readers and 20% are writers, and we switch it around so that writers
starve instead, then we might pass the tests!
Changes:
 - Writer checks the writers bench first.
 - Readers give writers priority before entering.
But the writers still starve.

Solution: Add fairness by alternating a pointer to either bench; flip it on
each exit. This is like Dekker's algorithm and the bathroom door! Check who was
last in.
Or: Move the writer's exit-protocol back to favour readers. This adds
fairness/alternation! This is left as an axcercise.

Now, this satisfies all 5 rules of the critical section game.
But you forgot... there's a rule 6!
Temporal ordering--staleness/freshness.
Reader that arrived at 2pm is reading stale data.

If you go into a grocery store, and want to get in the checkout, you
arbitrarily look at a line and think, "that will go fastest". But nope!

At airports, there is a single shared queue. Everyone in front of you came
before you in time, and everyone after came after. We know this without needing
a watch, which is never accurate enough.

So if we collapse our split line into one bench, we don't know who is who!


Lecture

Slide of code that looks important
"These are my solutions to A3"
[REDACTED]

Q1
"There was a page in the notes with two nested loops. You could have expanded
 it to three nested loops here."
Note: Single & works as expected with booleans. BUT it doesn't short-circuit.
Double (&&) is a control structure, and short-circuits.
"There was a 3-4x slowdown versus GOTO / labelled break solutions."

Q2
Matrix multiple solution: Task defined inside matrixmultiply routine "to
prevent namespace polution" [reminds me of modularization in JS - Joey]. The
defined Multiply _Task is instantiated at the bottom of the matrixmultiply
routine and immediately deleted, but not actually because the _Task sticks
around for quite a while.

Q3
Bounded Buffer
"Bargers wait in FIFO order"
"Uses baton passing, like in the readers and writers problem."


Recall: Baton passing problem "airport line"
Bargers can get in the way!

Put Readers and Writers in a "shadow queue"--data structure that allows readers
to check for following readers while going in, but fairly prevents starving the
next writer.

Each task can pop the front node on the shadow queue when it wakes up because
we have to be at the front of the queue when we are unblocked.

OR:
 - Arrivers Queue
 - Readers and Writers Queue
 - Chair
When the last reader in a group wakes up and sees a writer, the writer can't go
in right away! The writer must sit down in a chair. Readers exiting check the
chair before checking the queue.
"The chair is really the front of the queue."
"If a task is woken up incorrectly (e.g. a writer woken up when readers are in
 the room) it sits in the chair."

"But we still have a problem. A temporal race happens between readers and
 readers, and writers and writers."
"When we 1. Put the baton down, and 2. Sit down on the bench, someone can sit
 down before we do."

We are back to the issue of having to free a lock and go to sleep. We just
can't do it without risking being interrupted. So we introduced some magic--a
"thingie" (that's a technical term)--that's built in to the underlying system
that takes a lock, releases it, and puts us to sleep atomically.

Potential solution: Ticket system, where we wake all tasks and allow the one
task with the ticket to stay awake, while the others go to sleep. But we can't
guaruntee that tasks will go back to sleep in the correct order, etc. "It's
madness!"

If we don't have "the magic", we can use private semaphores. A linked list of
tasks each wait on their own semaphores. Kinda like a shadow list. We have n
locks, but it works! We can put this on the Airbus A380.

Optimal solution: [slide 327]
1 queue of arrivers, and one chair.
"Double locking."
"Really hard to uderstand; ad-hoc solution that happens to be optimal."
"Patterns are general. One-off solutions are cool but incomprehensible."


Q: How many of you had barger solutions that ran faster than (mostly)
non-barging solutions?
Class: *lots of raised hands*
"Doesn't that seem counterintuitive?"
"What happens is that bargers go straight through without blocking."
*Walks up* "Ooh, the lock is open! *opens lock* *steals goods* *leaves*

Why don't we allow bargers all the time?
 - It often matters which consumers take the item. Certain problems allow
   barging when it doesn't matter, but others require controlled handing out of
   certain items to specific types of consumers.

Note: You probably saw your code slow waaayyy down when we ran it on a
multi-processor. This is because we have many tasks "pounding on a lock" (high
contention).
"Never believe anything you are told."
"Timing is everything."
"Walk away and write a test program."
Rutherford: "If you can't measure it, it's not science."

Concurrent Errors
"Not everything is a deadlock."

Race Condition
 - Occurs when either synchronization or mutual exclusion are missing.
 - Northeast Blackout 2013--largest power outage in North Americans history was
   caused by a C race condition.
   "It was an amazing day, to see a culture go crazy without their electticity."
 - There are some tool sout there to try and automate testing for race
   conditions, but they aren't 100% reliable. "Should you decide to do a PhD in
   CS, this is a potential topic!"

No Progress
Live Lock
 - Indefinate postponement--"You go first" problem on simultaneous arrival.
 - "Like 4 cars arriving simultaneously at a 4-way stop. Giving right-of-way to
   the person on the right can go in a circle and result in a live lock. You
   could die of exposure!"
Mutual Exclusion Deadlock
 - Failure to acquire a resource--all 4 cars must reverse. But they can't!
 [picture of deadlocked intersection]

[midterm]


Lecture
Note: Peterson's algorithm is essentially a coin toss. Assuming a fair coin,
thie is acceptable.
"Infinitely long starvation is incredibly rare."
"So people tend to ignore starvation."

Starvation
 - Selection algorithm ignores one or more tasks so they are waiting for an
   event that will not occur.

Deadlock
 - Very precise definition!
 - Deadlock is the state when one or more processes are waiting for an event
   that will not occur.
 e.g.
     main() {
         uSemaphore s(0);
         s.P();
     }
"Sitting there, dead."
"If there is a livelock, we are probably consuming a lot of CPU, rather than
appearing dead."

Livelock
 - Mutual exclusion deadlock--failure to acquire a resource protected by mutual
   exclusion.
 - Indefinite postponement
 "Do not proceed into an intersection unless you can clear it."
 "It's an unrecoverable situation. You might as well jack the cars up, build
 basements under them, and rent them out to students. They're done."
 Djikstra: "Deadly embrace"

example:
    uSemaphore L1(1), L2(1); // open
    L1.P()                                L2.P()
        R1                                    R2
        L2.P()                                L1.P()  // both sleep!
            R1 & R2                               R2 & R1

There are 5 conditions that must occur for a set of processes to enter a
mutual exclusion deadlock:
"All 4 of them! Simultaneously!" (5th rule, implied)
...

(And 1 smaller rule for synchronization deadlock, above.)

Deadlock Prevention
 - With static analysis programs. Aviation programs must pass these tests.
 - We just need to eliminate one of the five rules.
1. There exists more than one shared resource requiring mutual exclusion
 - getting rid of this is impossible in many situations
2. no hold and wait: do not give any resources, unless all resources can be
   given
...
3. allow preemption
 - it's dynamic; cannot apply statically :(
"In Washington I saw little golf carts zippin gup and down the sidewalks,
latching onto cars locked in intersections, dragging them on the sidewalk,
ticketing them, snd putting boots on them. That's preemption!"
4. no circular wait
 - sacrifices some performance
 - can apply resource ordering

Ordered resource policy
 - Establish an ordering of resources that need to be required
 - Say thread one has R1 and is waiting for R2. There must be a task in front
   that has what you need. But anything in front of the front task, it can
   always get. Anything behind it already has, and will eventually release.
 - Always ensures we won't have any cycles, even though we are holding and
   waiting
Deadlock Avoidance
 - Oracle coordinates resource access
 - Monitor all lock blocking and resource allocation to detect any formation of
   deadlock.
 - "Everybody stay back! 10m back from the deadlock hole!"
Banker's Algorithm
 - demonstrates a safe sequence of resource allocations that don't lead to a
   deadlock
 - however, requires that a process states its maximum resource needs
 - Total Resources (TR), Maximum needed for execution (M), Currently allocated
   (C)
   - Resource request (T1 wants another R1, from 2 to 3).
   "I computed an intermediary matrix of the difference from our worst case."
   "Is there any task here that can go to its worse case, given the resources
   that are currently available in the system?"
   e.g. current available resources 1,3,2,2
        In the intermediary matrix, does any task have <= 1,3,2,2?
 - Oracle looks for a safe path of execution such that all threads can execute
   with needed resources safely.
 - If T1 asks for a resource and the oracle says, "no", it doesn't have to give
   up its resources, it just needs to come back later. "Because the algorithm
   is so conservative."
 - "If it's working, we will end up with all of our resources free in the end."
Allocation Graphs
 - Check for potential deadlock by graphing processes and resource usage at
   each moment a resource is allocated
 - Box with dot(s): resource instance(s)
 - Circle: task
 - Arrow DOT->CIRCLE "task holds resource"
 - ARROW CIRCLE->BOX "task waits for resource"
 - Oracle distributes resources and manages load
 - Look for cycles in the graph.
   - With multiple resouce instances, cycle != deadlock
   - Break multiple-resource boxes in the cycle into separate boxes, creating
     an isomporphic graph
   - If each resource has one instance, cycle = deadlock
 - Look for any circle that only has arrows coming into it (T4) and pretend
   that that task finishes. Then we can change the direction of one arrow, and
   recurse.
   - Eventually we wind up with all our resources free, as in the banker's
     algorithm.
 - "Rick Holt invented this. His office used to be two doors down from min, and
   students could go shake his hand. These algorithms are invented by real
   people! You can get beers with them! Rick will always grab a beer with you."
Detection and Recovery
 - Oracle needs to prevent starvation
 - "A task that causes a deadlock and gets set aside and removed by the oracle
   will immediately jump back in and cause a deadlock."
 - "What do we do with the person we pull out? We have to speak sternly to
   them."
 - "You have to give back your resources." Or, "We've decided to kill you."
 - BUT you can't just restart programs. That would give people multiple raises
   in salary, repeat work, etc. So we have the notion of checkpointing.
   "I used to checkpoint when I would search for prime numbers for six full
   weeks."
Which Method do we Choose? How do they do it in the real world?
 - Databases use transaction processing
 - Ordered resource policy typically most practical
 - Very open field of research


Lecture
8.1 Critical REgions
Attempt to solve complicated critical sections with the help of the programming
language
 - introduced SHARED statement
 - introduced REGION statement around a shared variable
       REGION v DO
       ...
       END REGION
    - advantage: compile-time errors instead of forgetting to call V() on a
      semaphore
 - but deadlocking can still happen, and we don't have a way of handling
   conditional waiting

solution: 
VAR Q : SHARED QUEUE<int,10>
REGION Q DO
    AWAIT NOT EMPTY( Q )
END REGION

But if the condition is false, we release the region lock, and we try
again--this is busy waiting!
So they went to Norway in the late 60s/early 70s. What did the Norwegians give
us? Object oriented programming!
What was the first OO language? Simula 67.
(SmallTalk was done later at Xerox park--I met most of those people!)

What if we take Objects and Concurrency and put them together?
We get Monitors!
 - Take shared data and put it inside a class
 - Add member routines for the shared regions
   - i.e. One thread in the insert member routine or one thread in the remove
     routine at a time.
 - A mutex member (mutual exclusion member) is one that does NOT begin
   execution if there is another active mutex member.
 - public member functions of a monitor are implicitly mutex members
 - mutex members need a temporary variable to store a result, release a lock,
   and return the temp value. You will make mistakes and mess this up. The
   compiler won't. You could, in theory, write your own Monitor, but don't.
   Leverage the programming language!
 - Note: All public members use the same lock.
But is this a single-acquisition lock or a multi-acquisition lock?
 - With a single-acquisition lock, if we call public member from another public
   member, we would have a mutual exclusion deadlock.
 - Monitors give us multi-acquisition, so we can call inc() from dec() and so
   on. We can re-acquire it as many times as we want before releasing it.
 - Destructor in Monitor is public and mutex and blocks appropriately if the
   executing block ends

Scheduling (Synchronization!)
 - Normally, we do a FIFO schedule on a single lock.
 Two alternate modes:
 - External scheduling: ACCEPT statment
       insert(...){
           if (full) _Accept( remove );
       }
       remove(...){
           if (empty) _Accept( insert );
       }
   - Sitting in the "acceptor" chair gives you priority. This takes care of
     barging!
   - Waits for call to other member function
   - Complex batton passing happens!
   - Removing the if statement can cause a deadlock--"it's a tool, use it
     carefully" 
  - Note that conditions inside _Accept should be OR, not AND
  - Note that the queue is brought in in non-temporal order. _Accept( remove )
    gives the next remove priority over hte next insert.
 - The "acceptor chair" is a stack of waiting tasks. It's LIFO. But since the
   top is always waiting on a condition, that probably satisfies something for
   the next one, etc.

_Nomutex keyword on a member allows multiple members to come in at the same
time.
Risky! You should only be reading.
But it's safe! Assuming our Mutex members follow a contract, say only positive
numbers for a buffer size, without ever leaving it as a negative value.

But note: even with a mutex routine, we do not know if the returned value is
accurate as soon as it returns. We can't rely on shared results to persist.

Internal Scheduling
A condition is a queue of waiting locks.
Recall: Two types of cond locks. Where protection comes from inside, or from
outside.
uCondLocks inside monitors have safety from the mutex outside.
We can wait on a condition, check empty, look at the front, or signal the
condition.
One trick: Waking someone up with a signal, they can't actually execute (yet),
since you're in the monitor. Instead, you move them over to the "signalled
chair". They have priority, and the monitor starts them on the next wait/exit.
"Signalling is deferred."

"We use a chair for internal and external scheduling."
"If you signal an empty condition queue, it's lost."
[boundedbuffer internal scheduling example slide 370]

When you exit, signal the opposite queue. It's okay if it's empty.

Without the chair, we would get barging from the outside.


Lecture
A3 Solution discussion
Q1: Heap leads to contention. It's a shared resource.
"Be very careful with your data structures when concurrent programming."
Note: Vector.at() does bound checking, where operator[] doesn't.
Q2: MC - use signal flag like bounded buffer
SEM - use baton passing to prevent barging
uBarrier: Override last() to sum results and set result.
Printer: Buffer as an array of structures. Use union {Tour, unsigned int} to
save space. If Finished, print end line, flush. Else try to overwrite buffer
slot.

Big takeaway: These things don't require horrendous amounts of code. A couple
screenfulls each. If you are writing tonnes of code, simplify!

Q: Why not work with strings?
A: It's very inefficient/slow. Strings are dynamically allocated. Cout deals
with strings too, so it's also redundant. The "zen" of the cout is to keep
things in internal format as long as possible. Changing things from internal
format to external format (and buffering it, and printing it) is cout's job!

Now back to [slide 370].
General model: External and internal scheduling for a general monitor.
 - external entry queue (FIFO, temporal ordering) and extra mutex queues (for
   efficiency/cooperation, sorted for _Accept clauses)
 - internal condition queues, acceptor/signalled stack "chair"

When to use external vs internal scheduling?
 - External is easier to specify and explain; signalling is done for us,
   doesn't need condition variables, etc. "It's generally leaner and meaner and
   I prefer it."
 - BUT external scheduling can't handle all problems. The moment you need to
   check a condition and "sit down", we must use wait/signal, and then we're
   internal scheduling.

DatingService problem.
 - Compatibility code [0,19]
 - Want a matching compatibility code
 - Once you enter the dating service, if nobody is there that matches you, you
   sit and wait.
 - Boy could potentially Signal or SignalBlock twice, leaving their phone
   number on the board, so two signalled girls come in and get his phone number.
   "kinky"

Readers/Writer
Readers and writers problem (Solution 3, 5 rules but staleness)
_Monitor with uConditions readers, writers.
Counters for each blocked bench are tracked.
Signalling/waiting happens (baton passing).
Note: Single operation, _Monitor::signal(), doesn't need a lock passed in, it
all happens behind the scenes. same for wait().

Alternative interface:
    _Nomutex void read(...) {
        startRead() // acquire mutual exclusion
        // read, no mutual exclusion
        endRead() // release mutual exclusion
    }

Problem with expanding the mutual exclusion: No concurrent reads.
Trick: Using public _Nomutex routines that call private mutex routines, to
simplify the protocol.

Note: Each node in the ready queue has room for a pointer, so we can carry as
much user data as we want.
There is a routine called front() on a condition queue that gives you the user
data on the first node of the queue. You can't access user data further on.
"All this does is give you a little help for building shadow queues."

Solution 8: Use _Accept (endWrite) or _accept (endRead).
"As far as I know, this is the shortest readers/writers solution in the world."
In startWrite() sees n readers in the room, `while(rcnt>0) _Accept(endRead);`

Nested monitor problem: acquire mon X, call mon Y, and wait on condition in Y.
 - You could go to sleep holding the lock for mon X! Potential deadlock.
 - No clean solution.
 - Same issue with locks (nested lock problem).

8.6 Condition, Signal, Wat, vs. Counting Semaphore, V, P
 - wait always blocks; P only blocks if sem=0.
 - if signal occurs before wait, it is lost; V before a P affects the P
 - multiple Vs may start multiple tasks simultaneously, while multiple signals
   only start one task at a time because each task must exit serially through
   the monitor.

It's possible to simulate P and V using a monitor (uCondition semcond
wait/signal). This is internal scheduling. External scheduling wuld be shorter!
(Left as exercise).

"We can now do the first few items on assignment 5."


Lecture
Review: Internal vs External scheduling
External: _Accept
Internal: "Participate" with signalling, conditions, etc.
 - Implicit scheduling can select from the calling (C), signalled (W), and
   signaller (S) queues. "Three places where we can reach out and pull
   something in"
 - We can prioritize these different choices (13 different ways! Greater than,
   less than, and equal priorities--coin flip)
 - Six of the categories are considered useful. Different programming languages
   monitor work in one of those 6 ways.
   1. C = W = S
   C = W < S
   C = S < W
   C < W < S
   ...

Implicit Signalling
    waitUntil (bool condition); // Not in uC++
    - This is similar to Hoare's region statment + await statement, but without
      busywaiting!
    - Not present in uC++, but we could build one out of the constructs that
      exist with uC++ monitors. We do this in A5!
      - Problem: We have to wake everyone up in order to check their
        predicates, which is inifficient.
      - Notes tell us three ways to build a waitUntil monitor. Two of them are
        easy, one of them is fast ;)

We want to get rid of monitors that have starvation.
 - That means setting C equal (=) to something above. A fair coin toss means
   that people won't get starved. But this has barging! Get rid of those
   monitors!
   (Java and others use these though--C=W<S, or "no priority nonblocking")
 - uC++ is derived from the picture on [slide 384], with a stack instead of a
   queue for W
 - Hoare picked the wrong of two choices (C < S < W), because signals almost
   always occur right before a return. We shouldn't have to sit down after
   signalling, and get woken up only to leave.
   We want S to have highest priority, because it handles getting out of the
   monitor quickly after signalling (the most common case).
 - Search notes.pdf for "Ten kinds of useful monitor are suggested"
 - Pros/cons: "Implicit (automatic signal monitors are good for prototyping..."

Coroutine monitors (Cormonitors):
Coroutines with implicit mutual exclusion; i.e. a printer that looks like a
finite state machine on the inside (as opposed to just a Monitor with internal
state).

Java Monitors
Synchronized member has mutual exclusion (meant "mutual exclusion member").
Also said, "a monitor only needs one condition variable", which was used to
solve all problems.
 - Has wait(), notify(), and notifyAll() methods.
Internal scheduling is no-priority nonblocking! This means barging!
 - It will fail eventually!
 - "I would not get on the Airbus A380 if it was running Java!"
Erroneous JAva implementation of barrier:
 - try { wait(); } catch( InterruptedException e ) {} // WHY!?! This is
   useless! Silly Java people!
 - Doesn't work because of "spurious wakeup". But there is no such thing! So we
   require a loop around wait.
Misconception: We can build condition variables in Java with nested monitors.
 - We get the nested monitor problem
"If you have a broken lock with a race condition, you fix the race condition!"
"Some dude woke up and wrote a broken lock and somehow convinced the world that
 it was a performance issue!"
"I would fail someone who wrote a lock with a race condition! I can write lots
 of things with 'good performance' that are broken!"

Indirect Communication
 - Example: Email, messaging, snail mail (100 years ago, there would be 6 post
   deliveries a day!)
Direct Communication
 - Monitors work well for passive objects that require mutual exclusion because
   of sharing.
 - However, communication among tasks with a monitor is indirect.
 - Probem: point to point with reply idirect communication: [diagram]
Task (coroutine whose main routine starts immediately)
 - Has its own execution state
 - A task is like a monitor because it provides mutual exclusion (?)

In general, basic execution properties produce different abstractions:
[important table!]
Thread, stack, Yes synch/mutex or No
 - Things in red do not exist in uC++

One takeaway from this course: Locks are useful, but difficult to use. We want
higher-level abstractions.


Lecture
Recall: Dat important table about key properties / some in red.

_Task BoundedBuffer example
"Not a great idea to have a buffer as a task, but will help us explain"
Recall: Mary and Joe hiring a full-time attendant to manage bathroom
main routine has synchronization code; _When (notFull) _Accept(insert) {
} or _When (notEmpty) _Accept(remove) {
}

 - We've moved _accept statement from members to main so that we can track
   control flow with the task's state.
 - _Accept(c1, c2) S1;
   is equivalent to _Accept(c1) S1; or _Accept(c2) S1;
   is equivalent to _Accept(C1 || C2)
   same situation as if (C1) S1; else if (C2) S1; == if(C1 || C2) S1
 - Pulling it out into two statments lets us modify the statements.
 - Note: You can also put statements after _Accept statements (the curly
   braces).
 - _When controls execution of _Accept statement
   "Allows us to turn off part of the accept statement"
 - Pulling out into long form allows us to have multiple _When clauses
 - Long form also adds precedence--inserts are accepted before removes
   - BUT there is no starvation because when the (finite) buffer fills up,
     remove starts getting accepted.
 - Note: _When is checked once. No busywaiting (as in _WaitUntil), or sleeping
   until the condition is true, etc.
 - Using an if statement instead gets quite messy. _When and _Accept covers
   each condition once, whereas if statements need to cover all combinations of
   conditions manually! 2^N - 1 if statements (by binomial theorem).
 - _Monitor starts with all the doors open, empty inside; _Task starts with the
   main task inside, initializing, all doors shut. [otherwise, same picture as
   monitors]
"But then we get a call from the producer-consumer union. They say they don't
 want to do the counting and front/back setting, since it has nothing to do with
 production/consumption."

So we pull out these lines into the statements following each _Accept
statement.

**I have a hunch that this code will be good to know for the final:

template<typename ELEMTYPE> _Task BoundedBuffer {
    ...
    public:
        ...
            void insert( ELEMTYPE elem ) {
                Elements[back] = elem;
            }
        ELEMTYPE remove() {
            return Elements[front];
        }
    protected:
        void main() {
            for ( ;; ) {
                _When ( count < Size ) _Accept( insert ) {
                    back = (back + 1) % Size;
                    count += 1;
                } or _When ( count > 0 ) _Accept( remove ) {
                    front = (front + 1) % Size;
                    count -= 1;
                }
            }
        }
};

This makes the "full time employee" tasks more worthwhile.
The after-statements occur concurrently with production and consumption. P/C
come and go, then the administrative work is done by the other thread.
Note: The accepted function's code runs *before* the statement following the
_Accept clause.
Note: We still have only one thread inside at a time. Our task is just
carefully keeping certain doors open/closed, on behalf of the P/Cs waiting
outside.

Don't ever put _Else statements on your _Accept clauses, save for two or three
places in your project. It can lead to busywaiting or an infinite loop if no
_Accept clauses are true.

Exception uMutexFAilure::RendezvousFailure turned on by default because it's so
important.

Accepting the Destructor
 - We need to break the infinite loop of _Accept insert, delete.
 - Use an empty stop() {} routine.
 - But the stop routine feels like a flag variable! Bleh!
 - So _Accept the ~Destructor, and then call break in main. (Unless we have
   code we want to run before buffer is gone).
   - But this segfaulted! We can't resume main after deleting memory!
   - So we left it in, but tweaked it, to be the opposite. _Accept(~Destructor)
     works like a signal, destructor call sits on the chair, main breaks and
     runs off the end, and then the _Task becomes an object with mutual
     exclusion--a Monitor! Then the monitor sees the destructor call in the
     chair, runs it, and voila!
 - Recall: _Accepts are like SignalBlocks; Signals are like signals.
 - Note: uMain needs to wait for each producer/consumer to finish and delete
   them so we know the buffer is empty (nobody waiting at a closed door). If we
   do that wrong, it breaks.

Internal Scheduling
 - Additional thread does nothing! _Accept (insert, remove) {}.
 - Threads go back to being in charge of their own synchronization.
 - But wait! combining the two ideas (internal and external) ends up being
   super useful! (A5 and A6).
 - If someone comes into remove, sees count==0, and cannot proceed just yet, it
   goes to sleep. Then _Accept(insert) can wake it up! Complex synchronization
   happens!
    - We must still decide how much work happens in the p/c and how much is
      done by the admin task.
 - Inside main, use SignalBlock to wake up people in member routines, not
   signal!

Note: Never ever ever put a lock in a monitor.
A task is a monitor.
Never ever ever put a lock in a task!
Note: _Accept sits you on the acceptor/signaller stack, and you leave the
monitor. When the monitor is empty for whatever reason, it looks at that stack
first.

"This is all we need to know for A5."

For A6, we will see how we can increase concurrency.
i.e. Direct communication (caller and callee task--or if you want money, caller
is client and callee is the server).
The above pattern of moving code into the admin task is starting us on that
path.


Lecture
Recall: We want the BoundedBuffer Task to use signalBlock so that the thread
leaves the monitor immediately.

Increasing Concurrency
Client (caller) and Server (callee)
 - Ideally, client does the minimal amount of work and then leaves
 - Examples using _Accept have no or little concurrency, as clients can't get
   in and out due to mutual exclusion, and server task blocks while caller does
   work inside.

Internal Buffer
 - Recall: If a buffer is always empty or always full, we don't need a buffer!
 - Relationship is now customer (client), manager (server), and employee
   (worker)
 - "Creating another end on the buffer--server hires workers on one side of the
    buffer; server itself simply manages the buffer."

Administrator
- Key: the administrator does no "real" work; its job is to manage.
- Management means delegating work to others, receiving and checking completed
  work, and passing completed work on.
- Accepts calls but never makes calls (don't want to be put on hold/blocked,
  because then we stop managing).
- There can be many different types of workers (couriers, notifiers, timer
  notifiers, etc.).
  - Timer: "Call me at noon to let me know it's lunch time, then again at 6 for
    dinner."
  - Administrator: Has a fixed or variable pool of workers.
  - (Simple) Worker: Just talks to the administrator. Looks like two calls: to
    ask for work, and then to return work. But we can combine these into one
    step! If there is no work there yet, the Worker blocks inside the
    administrator and sits on a bench, all in a single call still!
  - (Complex) Worker: Can also talk to clients.
  - Courier: An administrator never calls out, so how do they talk to each
    other? A courier! Courier calls in to an admin, asks for messages, sits on
    bench if no messages. When delivering a message, may also sit on a bench if
    the admin needs time to generate a response.
- On our assignment, we will build something that looks like this [slide 424]
  - Will take ~5 administrators, and a bunch of workers
  - Up to us to decide how we use couriers, etc. (We read a fixed pool size of
    them as input).

The concept of an administrator task was invented at UWaterloo, and was part of
the PhD of David R. Cheriton! Used in Port, Harmony, and Thoff (?) operating
systems. They scale very well!

Can build a hierarchy of administrators--super-administrators, etc. Can be
scaled up to very large systems.

This design pattern appears to be deadlock-free, since administrators don't
call/block and so we don't get into hold and wait situations.

"That covers 95% of A6."

Client-Side
Idea: Asynchronous call. "Drop off some information (in the administrator) and
      get away as quickly as possible."
Feature: Asynchronous call buffers these requests, and guaruntees that the
         caller won't block on the server.
         "Asynchronous calls are just more buffers."
uC++ only supports synchronous calls. You can build asynchonous calls from
synchronous calls, but the reverse is harder due to buffer overhead.
"If you have to choose one, implement synchronous calls, then build asynchonous
 calls on top of that later."

But if an asynch call requires eventual returning of a value, that forces us to
split the operation into two calls! This requires a two-step protocol, but
people are bad with protocols!

*accidentally drops presenter*
"Oh no! I've had this presenter for many years! Not actually, but oh well."

Tickets/Tokens
- One form of protocol
- Can be misused (forged, etc.) BUT we typically write the client AND the
  server. When we have trusted clients and servers, it's good. Untrusted
  clients can abuse this.
- Caller assigned a ticket, then returns with a ticket number.
- You can forget to come back with your ticket. Bah! Unreliable protocol!
  "This is why drycleaners have yearly yard sales of forgotten clothes."

Call-Back Routine
- Another protocol
- Register/transmit a routine on the initial call.
- The call-back routine cannot block the server; it can only store the result
  and set an indicator (i.e., V a semaphore) known to the original client.
- Original client must "poll" the indicator or block until the indicator is
  set.

Futures
- A future provides the same asynchrony as above but without an explicit
  protocol
- "Dry cleaner takes your clothes, sweeps them away, and immediately hands them
   back to you. But they actually gave you an empty bag, and they assume you
   won't put your clothes on ASAP, and later sneak into your closet to put your
   clean clothes in your bag."
- You received a "future of clean clothes".
- You will block if you reach for the hanger too early.
- Looks like a normal routine call, but it's "smoke and mirrors".
- REsult must look, smell, act like the result we're expecting (i.e. an int).
- Under the covers, could be using tickets, callbacks, etc.
- Implementing: Make your Future a friend of your server _Task, for setting the
  future state.
- Note that Future objects have extra memory allocations, which must be
  explicitly deleted, unless you're using a garbage-collected language like
  Java.
- So, it's hard to not know you are using a future. You always know.
- uC++ provides two types of Futures
  - Future_ISM<T> automatic allocation/free of storage via GC (use this in
    assignments)
  - Future_ESM<T> (Explicit storage management vs. implicit), deleted explicitly

Client w/ Futures:
    #include <uFuture.h>
    Server server;
    Future_ISM<int> f[10];
    for(int i-0; i<10; i+=1) {
        f[i] = server.perform(i);
    }
    // work asynchronously while server processes requests

    // receiveasync results:
    for(...) {
        osacquire( cout ) << f[i]() << " " // sync on retrieve value, must be
                                         // done once before accesing with []
            << (int)f[i] << " " f[i] << endl; // cast optional but can fix rare
                                              // bugs
    }

Note: We shouldn't change futures, especially when they are shared.
      i.e. f[3] = 3;

Reset future: f[3].reset(); // be careful; must KNOW nobody is still waiting
                            // for it

Cancel future: f[3].cancel(); // tell server to stop wasting time on a task we
                              // no longer need completed

Future pointers: FUTURE_ISM<int *> // future pointer returns pointer to answer,
which may be changed (but pointer itself can't be changed)
"Why am I telling you this? Nudge nudge, wink wink." **ASSIGNMENT/A6: Use this!

Server w/ Futures

struct Work {
    int i;
    Future_ISM<int> result;
    Work( int i ) : i(i) {}
};

Future_ISM<int> Server::perform() {
    Work w = new Work(1);
    work_queue.push_back(w);
    return w->result;
}

Complex Future Access
- _Select statement waits for one or more heterogenous futures based on logical
  selection-criteria
  e.g. _Select( f1 || f2 && f3 );
- Not to be confused wiht _Accept!

A _Select clause can be guarded with a logical expression and have code
executed after the future recieves the value (hust like _Accept).

_Select( f1 )
or _Select( f2 )
and _Select( f3 );
- AND goes when both futures come in; can't assume f2 has executed in f3.
**A6 "All we need for our project is _Select(f1 || f2)."
"We will have complicated _Accept statements, but _Select is easy!"
- If f3 comes first, we do the statment after f3's select statement, then we
  block and wait for an f1 or an f2 then unblock.
- Note: you can't call f2 inside statement 3; only statement 2 is available in
  there!

"The course is effectively done. You can do the assignment. The rest is just
 for 8 short answer questions on the final. DO NOT run off and use the
 following things in your assignment!"

"We will now shift focus from concurrent programming to compilers and hardware."
Optimization
- A computer with infinite memory and speed requires no optimization.
- With finite resources, optimization is useful/necessary to coserve resources
  and achieve good performance.

General forms of optimizations are:
- reordering: data and codea re reordered to increase performance in certain
  contexts
- eliding: removal of unnecessary data, data accesses, and computation "vast
  amounts of your beautiful program is thrown right in the garbage"
- replication: processors, memory, data, code are duplicated because of
  limitations in processing and communication speed (speed of light)
- optimized program must be isomorphic (perform same output for fixed input)

Sequential optimizations
"A lot of programs are not written in optimal order."
- operations occur in program order, so optimization semantics are simpler
- data dependency
- control dependency

Most programs are not written in optimal or minimal form.
- OO, functional, SE are seldom optimal approaches on von Neumann machine.
- "All those lovely precedures, member routines, objects, classes, are
  flattened! Gone! A compiler reduces it all to one big chunk of code. Software
  engineering is for people."

Concurrent optimizations
- reorder disjoint (independent memory addresses) operations (variables have
  different addresses!)
- elide unnecessary operations (transformation/dead code) `x=0;x=3` => `x=3`
  for loop without body => trash
  tail recursion => convert to loop
- execute in parallel if multiple functional-units (adders, floating units,
  pipelines, cache)
  - part of why the compiler reorders things is to take advantage of the
    parallel hardware--start up different ones at different times for maximum
    throughput
  - Keep in mind that replication causes duplication, which adds challenges in
    terms of keeping things consistent
Memory Hierachy
- Duplication between cache, memory, disk
Cache review
- Problem: CPU 100(0)s times faster than memory 10000(0) times faster than
  disk.
- Solution: Move highly-accessed data within a program from memory into
  registers for as long as possible and then back to memory.
  - Takes advantage of locality
- Associative cache: hash table, with keys as memory addresses
- Cache lines pull a buffer worth of data at a time (spatial locality)
- Cache is transparent (programmers don't worry about it)
- Cache aims at reducing latency (vs memory)
- We move things into cache eagerly, and move them back lazily
- Hardware knows how to keep cache and memory consistent, which can be seen by
  everyone; registers are "dark" and invisible to the system; we would love to
  get rid of registers, but they are super useful for programming due to the
  tiny addresses of each register.

Cache Coherence
- There are different levels of caches, typically 3 (L1, L2, L3)
- L1 closest to registers, L3 closest to memory
- Each level must be kept consistent
- Different cache coherence protocols: MOFET, MOSET (AMD/Intel)
- Cache coherence is hardware protocol ensuring update of duplicate data.
- Cache consistency addresses *when* processor sees update--bidirectional
  synchronization.
- Registers' copies of a shared variable may each be "wrong" until someone does
  a store and the cache knows about it.

Cache Thrashing
- When multiple threads repeatedly change a shared variable. "I'll change the
  value in the cache, and tell you about it. Then you change it and tell me
  about it."
- Cache *lines* are invalidated, not variables. Two variables next to each
  other can lead to thrashing.
- Inadvertant cache thrashing can occur when cache lines are overwritten--false
  sharing. This can be resolved by adding padding between variables (depending
  on the compiler).

Concurrent Optimizations
- Recall: Readers/writers problem (can't read when others are writing--can
  cause flickering, reading stale data)
- This is hard to enforce in a cache--it slows us down!
- We control order and speed and execution to resolve this (?).
- Sequential sections accessing private variables can by optimized normally
  (but not across concurrent boundaries--that's when we restrict optimization)
- We restrict sequential optimizations so that lock.acquire() doesn't get moved
  after a critical section, etc.

Eliding
- "Things inside of registers are invisible to the machine."
- Looping on flag in register is silly!
- For hihg-level language, compiler decides when/which variables are loaded
  into registers.
- Volatile keyword forces program to fetch register value from memory (cache)
  each time.

Double-check locking for singleton vars
- Concurrent implementations of a singleton pattern can fail where a pointer's
  storage is allocated before the pointer is assigned; never assume pointers
  are non-null (?).


Lecture

Memory model: ideally, a universe where shared variables are instantly visible
to everyone. But we are bounded by the speed of light; caches can't update
instantly. Information flows through the layers of caches like a wave, not
instantaneously. Staleness is something we must accept.

BElieve it or not, nothing we've done so far has let staleness been a problem.

e.g. The semaphores that we spin on are values that are blocking until their
unblocking open value is propogated to our cache. Spinning on stale values here
doesn't matter.

Set of optimizations defined by CPU manufacturer define a memory model.

[table: Relaxation, W->R, R->W, W->W, Lazy cache update; Model(AT, TSO, ...)]

TSO is most common for us.

In each of these, there are mechanisms to disable optimizations and "claw back"
to sequential consistency in order to write concurrent programs. We would like
atomic consistency, but we know algorithms to deal with sequential consistency.

Preventing Optimization Problems
- All optimization problems result from races on shared variables
- If shared data is protected by a lock (implicit or explicit):
  - locks define sequential/concurrent boundaries
  - boundaries can provide preclude optimizations that affect concurrency
- Called race free because programmer's variables do not have races because
  synchronization and mutual exclusion
- However, race free does not mean there are no races
  - Races to internal locks, etc.
- Magic word volatile
  - Utter witchcraft
  - Insert it in places where it fixes bugs
  - Use cases depend on how certain compilers decide to cache data; may not
    always fix a bug, etc.
- progam order / compiler: disable inlining, asm("" ::: "memory");
- memory order / runtime: fences (mfence, lfence, ...) "stops specific types of
  optimizations from happening"
  - also specifies layer of code where the optimization is disabled
    specifically; underlying code optimizations can "bubble up" to this point.
- platform-specific instructions for cache invalidation
- most mechanisms are compiler/platform specific, and difficult/low-level =>
  tread carefully! "You must become a wizard!"

Code example: Dekker for TSO environment:
(_Task Dekker with a bunch of inline and fencing instructions, cache aligning)

"My team and I build world-class software solutions for these algorithms for a
 living, now matching or outperforming current atomic-instruction algorithms,
 without any atomic instructions!"
"I use 300-600 CPU hours a week running experiments in code like this. An
 experiment is running right now!"

Other Approaches

Atomic (lock-free) data structures
- Initially thought to be a holy grail solution, but now known to be just
  another tool
- Data structures that don't directly need classical locks (locking is done
  indirectly)
- Recall: Ad-hoc solution to readers-writers problem was very specific, can't
  be generalized; very small set of well-designed data structures can be called
  lock-free
- If they are lock free and don't have starvation, they are called wait free

Compare and Assign/Set/Swap(erroneous) Instruction
- Atomic CAA; doesn't always do a write (reading a false value stops short).
- As soon as we know we can atomically read and write, we can use it to solve
  mutual exclusion

Lock-Free Stack (trying to build a lock-free data structure)
- Stack class with top pointer, Nodes with data and next Node pointer 

*clicker dies*
"Wow. Turn it off, weight 5 to 8 seconds turn it back on... no sausage!"
*realizes laptop is frozen*
*presses keyboard shortcut to bring up TTY interface, signs in as root, issues
 shutdown now command*

- push operation:
  
    newnode.next = top; // link new node
    top = newnode;      // update top with CAA

    // CAA failed, top != n.next, so no assignment
    // must have been an intervening push
    // try again

    void Stack::push( Node &n ) {
        for( ;; ) { // busy wait :(
            n.next = top;
            if ( CAA( top, n.next, &n ) ) break;
        }
    }

- pop operation

    t = top // copy top node
    // CAA failed, top != t, so no assignment
    // Someone took the top while we were setting up ours!
    // try again
    // CAA succeeds, top == t so ...

    for ( ;; ) {
        t = top;
        if (t == NULL) break;
        if( CAA( top, t, t->next ) ) break;
    }

But these don't work, because of the ABA problem.
- "Know this for your Google interview!"
- Push works, but pop fails.
- Given a stack top -> x -> y -> z, thread 1 pops x, gets interrupted, thread 2
  pops x and y then pushes x back on, then thread 1 does the CAA and sees x!
  Then our stack gets corrupted!
- To fix this,  we throw more hardware at the problem: double-wide CAVD
  instruction, compares and assigns 64-bit (128-bit) values. Then we add a
  ticket counter.

- push:
    CAVD( n.next, n.next, h );
    for ( ;; ) {
        if( CAVD( h, n.next, (Header){ &n, n,next.count + 1} ) ) break;
    }

- header is (top, #pushes)
  - compare (x,3) above, but we see (x,4)

If we attempt to apply this to a queue or a doubly-linked list, it goes
downhill fast. It would take three weeks of lectures. Extremely ocmplicated!
- Ends up with dangling pieces of storage; you end up building yourself a
  garbage collector of sorts (can't simply use malloc and free!)

Summary: locks vs lock-free
- lock free has no lock, doesn't ever contribute to deadlock
- performance gains of lock free are unclear
- lock free can only be used in specific ways
- locks can protect arbitrarily complicated critical sections :)

Exotic Atomic Instructions
- VAX (Russian Airforce) has linked-list update instructions in the hardare :O
- MIPS processor has load locked (LL) and store conditional (SC) instructions;
  system has an "oracle" that watches memory locations indicated by LL and lets
  you know if it's changed when you do SC

  e.g. Implement test-and-set with LL/SC:

  testSet:
    ll  $2,($4)
    or  $8,$2,1
    sc  8,($4)
    beq $8,0,testSet
    j   $31

  - Does not suffer from ABA problem, as we have a full-time Oracle in MIPS
    watching data for us.
  - BUT it's hard to implement at the hardware level, and tends to have lots of
    caveats/error cases.
  - Can't build SPARC's atomic swap, because that requires watching 2 memory
    locations (but LL/SC only watches 1).

- AMD64 a few years ago added multiple watchpoint instructions, but never got
  into production.
  - It's like data base transactions, where changes are either committed fully
    or rolled back and restarted
  - SPECULATE to start a speculative region, next instruction checks for abort
    and branches to retry; also LOCK and COMMIT instructions
  - Hardware transacitonal memory allows N (2,4,8) watchpoints--this is the
    future! It lets us do anything!
- Software Transactional Memory (STM) allows reservations of arbitrary size;
  super difficult to implement, often bad performance.
  "This is like implementing a web browser inside a database. It doesn't
   scale..."

General Purpose GPU (GPGPU)
- coprocessor
- Single-instruction multiple-data environment (SIMD), wheras CPUs are
  typically MIMD
  - SIMD executes the same instruction many times in lockstep; memory is GPU
    memory
  - e.g. A conditional: all threads check a condition, some get t/f, hardware
    masks (stops) false threads (like the _When clause). Then all the trues get
    masked, and all the false ones start up again for the else clause, and all
    continue.
- GPU uses control units (warps) of 32 "threads" executing in lockstep.
- Warps may be combined into blocks executing the same code, kernel executing
  multiple blocks
- Very bizarre programming environment
- Herds and herds of registers further complicate things
- Parellel operations dump to an array of subtotals
- Code example: cudaMalloc memory for subtotals, matrix, then copy those
  buffers out to system


Lecture

Concurrent Languages

Ada
- Monitors: implicit signalling
- Tasks: where uC++ Tasks are based, minus object oriented style (accept ... do
  code-normally-in-member-routine)

Wait/signal is magic because it's like suspend/resume in coroutines--you go to
sleep and wake back up, and the universe is the same around you. With no
waiting, Ada is less desirable.

SR/Concurrent C++
- "One attempt to make external scheduling more powerful."
- Downside of accept: can't look at arguments
- When/by clause on accept statement to look at calling "codes" (like dating
  service compatibility codes)--lets us examine things waiting outside.
  - By cluase calculates for each truthy when clause, and the minimum by clause
    is accepted (e.g. shortest person by height).
- Never as powerful as internal scheduling--we can't compare two or more
  qaiting queues, etc. Internal scheduling gives us the whole programming
  language to make decisions, rather than contrived external information.
- I feel like advancing the accept statement is an unsolveable problem. But
  prove me wrong!

Java
- Concurrency constructs are largely derived from Modula-3
- Thread is like uC++ uBaseTask, and all tasks must explicitly inherit from it.
- Threads must be explicity started.
- Forgetting the "start" could result in the right answer from your program,
  but with no concurrency! "The best protocol would be a protocol with one
  step."
  - This allows inheritance, though (constructor runs completely before start).
  - uC++ allows inheritance automagically (uAction extra argument added during
    compilation, etc. figures out when most derived constructor is done
    running).
- Note: All Java objects must be heap-allocated (but stack is your friend in
  C++).
- Like uC++, when the task's thread terminates, it becomes an object, hence
  allowing the call to result to retrieve a result.
- Java has barging! "Lean over and tell a coworker this, and email me what
  happens ;)"

**Final potential q: uC++ Cormonitors and Tasks turn into *Monitors* (objects)

Go
- Prof worked on Go team for 6 months in 2013
- Threads are called goroutines "I have no idea why Rob picked that name"
- Threads cannot be referenced by a name (are anonymous and run off on their
  own).
- If main program starts threads then runs off the end, it kills everything!
  Common newbie mistake. Carefully stop main thread and have it woken up by
  another thread later.
- Threads interact through bounded buffers (channels, CSP). This is a paradigm
  shift from routine calls.
- Channel is a type shared buffer with 0 to N elements; facilitate asynchronous
  calls; very powerful, can have channel of channel of strings, etc.
- Special message sent in a channel to tell main thread to stop.
- Has conds, mutex, once (singleton-pattern lock!), atomic instructions, etc.
- Go is not C! It's garbage collected, etc etc.o

C++11
- Thread creation: start/wait (fork/join) approach.
- Compile with g++ -std=c++11 ...
- Any entity that is callable (functor), including lambdas, can be started
- Thread starts at implicit point of declaration
- thread can detach() and run independently, but watch for dangling pointers to
  local variables that are later deallocated.
- It's an error to deallocate thread object before join or detach.
- Must put all waits in while loops because barging (and probably spurious
  wakeup LOL)
- async calls, futures, atomic operations

Concurrency Models

Actors (popular, particularly used in Scala and Erlang)
- Developed by Hewitt & Agha
- An actor is an administrator, accepting messages, handles it, or sends it to
  a worker in a fixed or open pool
- However, communication is via an untyped queue of messages--a mailbox (a
  bounded buffer!)
- Actor is like a piece of work that gets perform()'ed; gets pulled off queue
  by workers.
- Actor also has a recieve routine (where message arrives); actor can change
  itself.
  - Recieve routines can be swapped, say, depending on states in a finite state
    machine; "recieve state 2", etc.
- Mailbox is polymorphic in message types => dynamic type checking
- Message send is usually asynchronous.
- Advantages in Scala over uC++: GC allows detached threads where uC++ must
  join
- "As of Tuesday (two days ago, Nov 29) we have a complete actor model
   working, and it's 10 times faster than scala!"

Linda
- Will sound like madness!
- Recall: our swimming pool idea
- Tuplespace is our swimming pool--things can be thrown in, read, taken out
- Tuplespace is accessed atomically and by content
- In and out are backwards (out means write, in means take something out)
- How do we write a concurrent program with this? Start with a semaphore!

    out( "semaphore" ); // open
    ...
    in( "semaphore" );
    // critical section
    out( "semaphore" );

- For a bounded buffer, do `out( "bufferSem" )` ten times (these are slots).
  Producer calls in("bufferSem") to wait for empty slot.

OpenMP
- Recall: Three different types of concurrent systems--program finds
  concurrency in sequential program, or programming language helps programmer,
  or programmer does all the low-level stuff.
- OpenMP is the second level
- Uses #pragma to do things
- #pragma omp parallel sections num_sections(4) is like COBEGIN, followed by
  `#pragma omp section` is like each statement in a COBEGIN/COEND in uC++.
- `#pragma omp parralel for` is like COFOR
- Also barrier

Threads and Locks Library
- java.util.concurrent (More extensive than shitty Java monitors)
- Doug built this. Doug is pretty cool.
- Doug and I disagree on this: Doug has barging (already existed in Java)
- Apparently you could build non-barging things, ut you would have to dig
  deep--there be dragons.
- Doug added executors and futures
  - executors are like workers in the Actor model
  - "A swimming pool full of workers threads, with work units thrown in, with
     Futures as answers."
  - uC++ now has executors :)

Pthreads (in C)
- Just like basic Java stuff, but not type safe

Multiple Address-Spaces won't be on the exam.

Two sample finals will be posted! Subsections of those that are relavent will
be posted.

"If you want to be a good programmer, you must understand the human side and
 the computer side. Avoiding busywaiting, etc. will have unbelievable
 mechanical advantages, so we beat you over the head with them. You just do the
 right things now. But some tradeoffs are aceptable for readability--occasional
 flag variable, etc."

"I have made you concurrently dangerous. There is more to learn."
