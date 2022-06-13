# Debugging with ipdb

## Learning Goals

- Explain how `ipdb` is a more flexible REPL than the Python shell.
- Install `ipdb` via Pipfile
- Debug a program using `ipdb.set_trace()` within the body of your file

## Introduction

We'll cover `ipdb`, a type of REPL, and discuss how to install and use it to debug
a program.

## What Is a REPL?

You've already been introduced to REPLs through using the Python Shell. REPL
stands for _Read, Evaluate, Print, Loop_. It is an interactive programming
environment that takes a user's input, evaluates it and returns the result to
the user.

Python installs with its own REPL, which is the Python shell that you've already
been using. Every time you type `python` into your terminal, you're entering
into a REPL.

## What Is `ipdb`?

`ipdb` is another Python REPL with some added functionality. It is built on
top of `pdb`, a REPL in Python's standard library, and provides helpful features such as
tab completion, syntax highlighting, and better tracebacks. When you enter
`ipdb`, youare entering a brand new interactive environment. For any code you
want to play with in the Python shell, you have to copy and paste or write your
code in the PYthon shell itself. ipdb, on the other hand, is like a REPL that
you can inject into your program.

ipdb is far more flexible than the Python shell. Once you install the ipdb library (via this
lesson's `Pipfile`), you can use `ipdb.set_trace()` anywhere in your code.

`ipdb.set_trace()` gives you similar functionality to using `debugger` in a
JavaScript application, in that it lets you set a **breakpoint** in your code
that will pause the execution of your program at a certain point so you can
inspect the variables, functions, and other context available at a specific place
in your code.

So when you place the line `ipdb.set_trace()` in your code, that line will get
interpreted at runtime (as your program is executed). When the interpreter hits
that line, your program will actually _freeze_ and your terminal will turn into
a REPL that exists right in the middle of your program, wherever you added the
`ipdb.set_trace()` line.

Let's take a look. In this repository, you'll see a file called
`ipdb_is_awesome.py`.

## Using `ipdb.set_trace()`

First, let's make sure we have installed the packages from our `Pipfile`,
including `ipdb`.

```console
> pipenv install
> pipenv shell
```

> NOTE: This will only install `ipdb` in your **virtual environment**. If you
> want to install `ipdb` system-wide, you must exit your virtual environment
> and run the following command in your local environment:
>
> ```console
> > pip install ipdb
> ```

Look at the code in `lib/ipdb_is_awesome.rb`. You should see the following code:

```py
import ipdb

def tracing_the_function():
    inside_the_function = "We're inside the function"
    print(inside_the_function)
    print("We're about to stop because of ipdb!")
    ipdb.set_trace()
    this_variable_hasnt_been_interpreted_yet = \
        "The program froze before it could read me!"
    print(this_variable_hasnt_been_interpreted_yet)

tracing_the_function()
```

At the top of the file, `import ipdb` is a statement that loads the `ipdb`
library in this file when our application runs, very similar to `import`
statements in JavaScript.

> We will elaborate on Python `import` statements later in Phase 3, but note
> now that Python `import` statements **do not** use quotes or semicolons.

After importing `ipdb`, _which you must do to use ipdb_, we define a function, and
then call that function.

In the directory of this repo, in your terminal, run the file by typing
`python lib/ipdb_is_awesome.py`. Now, look at your terminal. You should see
something like this:

```txt
> python lib/ipdb_is_awesome.py
We're inside the function
We're about to stop because of ipdb!
> /lib/ipdb_is_awesome.py(11)tracing_the_function()
     10     this_variable_hasnt_been_interpreted_yet = \
---> 11         "The program froze before it could read me!"
     12     print(this_variable_hasnt_been_interpreted_yet)

ipdb>
```

You have frozen your program _as it executes_ and are now inside a REPL _inside
your program_. You basically just stopped time! How cool is that?

In the terminal, in your pry console, type the variable name `inside_the_function`
and hit enter. You should see a return value of `"We're inside the function"`

```txt
ipdb> inside_the_function
```

You are able to explore the data _inside_ the function in which you've placed your
binding. Any variables or functions that are available from the line you placed
your `ipdb.set_trace()` will be available to you to explore from the terminal.

Now, in the terminal, in your pry console, type the variable name
`this_variable_hasnt_been_interpreted_yet`. You should see a return value of
`nil`. That's because the binding you placed on line 7 actually froze the
program on line 7 and the variable you just called hasn't been interpreted yet.
Consequently, our REPL doesn't know about it.

Now, in the terminal, type `exit`, and you'll leave your pry console and the
program will continue to execute.

## Using Pry to Debug

In addition to _exploring_ code inside Pry, you can also manipulate variables
and try code out. This is where Pry really becomes helpful for debugging. If you
have a function that isn't doing what it's supposed to do, instead of making
changes in your text editor and running the tests over and over until you get it
working, you can put a binding in your code and try things out. Once you've
figured out how to fix the problem, you then update the code in your text editor
accordingly.

Let's walk through an example together. In this repository, you'll see a `spec`
folder containing a file `pry_debugging_spec.rb`. This is a test for the file
`lib/pry_debugging.rb`.

In `pry_debugging.rb`, we have a broken function. Run `learn test` to see the
failing test. You should see the following:

```txt
  1) #plus_two takes in a number as an argument and returns the sum of that number and 2
     Failure/Error: expect(plus_two(3)).to eq(5)

       expected: 5
            got: 3

       (compared using ==)
     # ./spec/pry_debugging_spec.rb:6:in `block (2 levels) in <top (required)>'
```

So what's happening? In the second line (the line starting with
`Failure/Error`), we can see that the test is calling the `plus_two` function and
passing in `3` as an argument. Below that we can see that the test is expecting
`5` to be returned, but that `3` is being returned instead. Remember that the
return value of a function in Ruby is generally the value of the last line of the
function, in this case, `num`:

```ruby
def plus_two(num)
    num + 2
    num
end
```

So while our function is adding 2 to `num` on the second line, it appears that it
is not _updating_ `num`. We have Pry required at the top of our
`spec/pry_debugging_spec.rb` file so we can use it to verify this. Let's place a
`ipdb.set_trace()` in our code, right after that line:

```ruby
def plus_two(num)
    num + 2
    ipdb.set_trace()
    num
end
```

Now, run the test suite again and drop into your Pry console. Your terminal
should look like this:

```txt
    1: def plus_two(num)
    2:  num + 2
    3:  ipdb.set_trace()
 => 4:  num
    5: end

[1] pry(#<RSpec::ExampleGroups::PlusTwo>)>
```

Let's check our current return value by typing `num` at the Pry prompt. You
should see something like this:

```txt
[1] pry(#<RSpec::ExampleGroups::PlusTwo>)> num
=> 3
[2] pry(#<RSpec::ExampleGroups::PlusTwo>)>
```

By checking the value of the variable inside our pry console, we can confirm
that `num` is still equal to `3` and, as a result, the function is returning `3`.

How can we modify the code on line 2 so that the function behaves in the expected
way? We need to _update_ the value of our `num` variable so that it's equal to
the sum of itself and 2. Play around inside your Pry console: try code that you
think will update `num` as needed, then check the value of `num` to see if it
worked. Once you figure it out you can type `exit` in your terminal to get out
of Pry, update the code in your text editor, and rerun the test to verify it's
passing. Be sure to remove the `ipdb.set_trace()`!

It can take a little while to get the hang of using Pry, so don't worry if it's
still a little confusing. As you start working with more complex functions and
data structures, you'll find it can be a very helpful tool.

## Resources

- [ipdb documentation](https://pypi.org/project/ipdb/)
- [Python Debugging with PDB](https://realpython.com/python-debugging-pdb/)
