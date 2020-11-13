---
layout: post
title: Basic Python Cheatsheet
description: "First-time programmers are often introduced through the Python language, due to its readability and simplicity. This cheatsheet was developed as a means for a friend to get a handle on the fundamentals of programming, and describes most of the basic functionalities of high-level programming languages through Python."
---
1. [Variable types](#variable-types)
    1. [Value types](#value-types)
        1. [Booleans](#booleans)
        1. [Integers](#integers)
        1. [Floats](#floats)
    1. [Complex types](#complex-types)
        1. [Strings](#strings)
        1. [Lists](#lists)
        1. [Dicts](#dicts)
1. [Operators](#operators)
1. [Functions](#functions)
1. [Flow control](#flow-control)
    1. [Condition](#condition)
    1. [Iteration](#iteration)
        1. [While](#while)
        1. [For](#for)
1. [Input and output](#input-and-output)
1. [Classes](#classes)

# Variable types

A variable has a value (that can change). Variables can be modified, in many different ways.

## Value types

Value, or basic, types only have a single value. There is nothing more to them than the basic value they hold.
It is generally possible to convert between basic types if it makes sense, e.g. `str(5)` returns the value `5`.

### Booleans

A **boolean** is a true or false value. Convert to it using the function `bool(x)`, e.g. `bool(0)` is `False` while any other number is `True`.

```python
x = True

print(x) # prints "True"
print(x == False) #prints "False"
```

### Integers

An **integer** is a whole number value, either positive or negative. Convert to it using the function `int(x)`, e.g. `int("128")` is `128`.

```python
x = 5

print(x + 5) # prints "10"
print(x - 11) # prints "-6"
print(x * 6) # prints "30"
print(x / 2) # prints "2" ... note that dividing an integer makes another integer
print(float(x) / 2) #prints "2.5" (correct)
```

### Floats

A **float** is a number with a fraction part. Convert to it using the function `float(x)`, e.g. `float("195.23")` is `195.23`.

```python
x = 4.5
print(x + 5) # prints "9.5"
...
print(x / 2) # prints "2.25"
```

## Complex types

Complex types have more to them, as they have variables and methods within them that you can access. These types are defined by a [class](#Classes), and specific instances of these are called **objects**.

### Strings
A **string** is a collection of characters, also known as text. Convert to it using the function `str(x)`, e.g. `str(965.333)` = `"965.333"`

You can also think of a string as a **read only** [list](#Lists), so you can iterate over it using a [`for` loop](#For), but you can't assign new values.

Strings are created using single `''` or double `""` quotes. There is no difference between using either of them, but the convention is to generally use double quotes. 

If you want to use the type of quote you're using *inside* the string, just put a backslash `\` before it and Python will know it's part of the text and not the end of the string.

```python
greeting = "hello"
name = "Josh"

print(greeting + " there, " + name  + "!") # prints "hello there, Josh!"
```

String objects have a wide range of useful functions that make life easier.

```python
fav_food = input("What is your favourite food?")
fav_food = fav_food.lower()

if fav_food == "beans":
    print("correct")
else:
    print("incorrect")
```

Strings can be **concatenated** (added) together, but to make it more readable you can also **interpolate** strings. This is achieved using an `f""` syntax instead of the normal `""`.

```python
greeting = "hello"
name = "Josh"

print(f"{greeting} there, {name}!") # prints "hello there, Josh!"
```


### Lists

A **list** is an ordered set. Convert to it using the function `list(x)`. 

You can access a value in a list by its index using square brackets `[]` after the variable name.
Lists can be **iterated over** (gone through) using a [`for` loop](#For).

```python
a_list = ["apple", "banana", "pear"]
a_list[0] # has the value "apple"
a_list[1] # has the value "banana"
a_list[2] # has the value "pear"

for fruit_element in a_list:
    print(fruit_element)

# prints:
# "apple"
# "banana"
# "pear"
```

### Dicts

A **dict** is a map of keys to values. Think of it like an actual dictionary, where you can look up a definition (value) using the word (key).
You can access a value in a dict by its key using square brackets `[]` after the variable name.

You can't use `in` for dicts, because it doesn't make sense, but you can check whether a value is a key or value using a dict's `.keys()` and `.values()`
functions to get a list of keys and a list of values, and checking them.

```python
banana_info = {"color": "yellow", "amount": 5, "is_fruit": True}
banana_info["color"] # has the value "yellow"
banana_info["amount"] # has the value 5
banana_info["is_fruit"] # has the value True

banana_info.keys() # returns a list of all keys in the dictionary
banana_info.values() # returns a list of all values in the dictionary
```

### And more!

Python allows you to define your own types through the class system. Go to [Classes](#classes) for more.

# Operators

Some operators make something happen.

| Symbol | Name           | Description                                                     | Syntax                     | Compatible types   |
| ------ | -------------- | --------------------------------------------------------------- | -------------------------- | ------------------ |
| `=`    | Assignment     | Assigns the variable on the LHS to the value on the RHS.        | `my_var = 5`               | all types          |

Some operators 'return', or evaluate to, a value.

| Symbol | Name           | Description                                                     | Syntax                     | Evaluates to   | Compatible types   |
| ------ | -------------- | --------------------------------------------------------------- | -------------------------- | -------------- | ------------------ |
| `==`   | Equality       | Returns whether the left value is equal to the right value.     | `my_var == 5`              | `boolean`      | all types          |
| `+`    | Addition       | Adds the left value and the right value.                        | `5 + 2` or `"hel" + "lo"`  |`float` or `int`| int, float, string |
| `-`    | Subtraction    | Subtracts the left value by the right value.                    | `17 - 12`                  |`float` or `int`| int, float         |
| `*`    | Multiplication | Multiplies the left value and the right value.                  | `6 * 10`                   |`float` or `int`| int, float         |

# Functions

Functions are blocks of code that can be 'called' (run) using parentheses `()` after the function name. Functions are actually just like any other kind of
variable, but they're defined differently.

Functions should generally be a black box: it only looks at the inputs it's given, and then it returns an output if necessary. You are doing functions wrong if a function refers to a variable outside it (constants are okay, but we haven't got into that yet).

```python
def say_beans():
    print("I say: beans")

say_beans() # prints "I say: beans"
```

Values can be 'passed in' through the parentheses into the function code.

```python
def say(phrase):
    print("I say: " + phrase)

say("hello") # prints "I say: hello"
say("kill me now") # prints "I say: kill me now"
```

Functions can also 'return' a value, which means that the function ends up as a value after it's called.

```python
def add_two_numbers(x,y):
    return x + y

a = 2
b = 5

add_two_numbers(a,b) # does nothing
c = add_two_numbers(a,b) # assigns the return value to the variable 'c'

print(c) # prints "7"
```

Variables can 'have' functions and variables inside them, which can be accessed with `.` after the variable name.

In VSCode you can often look at the functions available through pressing `CTRL-Space` once you have typed the `.`

```python
my_string = "hey there!"
my_string_uppercase = my_string.upper()

print(my_string) # prints "hey there!"
print(my_string_uppercase) # prints "HEY THERE!"
```

# Flow control

## Condition

Programming conditionals take the following form.

1. if *condition*, do this
2. else if *condition*, do this
3. else, do this

```python
if x:
    # runs if x is True
elif y:
    # runs if x is False and y is True
elif z:
    # runs if x and y are False and z is True
else:
    # runs if x,y,z are all False
```

## Iteration

### While

The `while` structure runs a block over and over again, while the condition is True.

```python
x = 0

while x < 5:
    print(x)
    x = x + 1

# prints:
# "0"
# "1"
# "2"
# "3"
# "4"
```


### For

The `for` structure **iterates** (goes over) the elements of a list one at a time.

```python
xlist = [2,4,6,8,10,12]

for number in xlist:
    print(number)

# prints:
# "2"
# "4"
# "6"
# "8"
# "10"
# "12"
```

# Input and output

Use the `print(text)` function to output text. Note that it tries to automatically convert all other values to text.

```python
print("wow I'm output text")
print(56)

# prints:
# "wow I'm output text"
# "56"
```

Use the `input(prompt_text)` function to ask for input using the prompt. It returns the text input by the user (as a string).

```python
favorite_num_string = input("Type your favourite number: ")
print("Your favourite number is " + favorite_num_string)

# prints:
# "Type your favourite number: " (and waits for input, let's assume the number '56' is input)
# "Your favourite number is 56"
```

# Classes

Classes are definitions of a new custom type. A class definition defines what functions and variables a specific type of object has.

A function that belongs to a class is called a **method**.

Class methods in Python always take a `self` parameter as the first argument. `self` contains all the information about the specific instance, and it's where you should store all the information you want to persist once the method is done.

```python
class Tile:
    def __init__(self,x,y,is_bomb):
        self.x = x
        self.y = y
        self.is_bomb = is_bomb

    def get_position(self):
        return [self.x,self.y]
```

You can create an **instance** of a class (a specific object) by calling the class name like it's a function. When you do this, Python calls the `__init__` method and passes in all the arguments you put in, so `Tile(5,4,False)` maps to `__init__` with `x = 5`, `y = 4`, `is_bomb = False`.

Note that you do not have to specify the `self` argument, Python does this automatically.
Internally, `my_tile_var.get_position()` is changed to something like `Tile.get_position(my_tile_var)`, filling in the `self` automatically.

```python
my_tile_var = Tile(5,4,False)

my_tile_var.get_position() # returns [5,4]
my_tile_var.is_bomb        # returns True
```