# if-__name__-__main__-
understand python  if__name__=='__main__' 



It's boilerplate code that protects users from accidentally invoking the script when they didn't intend to. Here are some common problems when the guard is omitted from a script:

If you import the guardless script in another script (e.g. import my_script_without_a_name_eq_main_guard), then the second script will trigger the first to run at import time and using the second script's command line arguments. This is almost always a mistake.

If you have a custom class in the guardless script and save it to a pickle file, then unpickling it in another script will trigger an import of the guardless script, with the same problems outlined in the previous bullet.

Long Answer
To better understand why and how this matters, we need to take a step back to understand how Python initializes scripts and how this interacts with its module import mechanism.

Whenever the Python interpreter reads a source file, it does two things:

it sets a few special variables like __name__, and then

it executes all of the code found in the file.

Let's see how this works and how it relates to your question about the __name__ checks we always see in Python scripts.

Code Sample
Let's use a slightly different code sample to explore how imports and scripts work. Suppose the following is in a file called foo.py.

# Suppose this is foo.py.

print("before import")
import math

print("before functionA")
def functionA():
    print("Function A")

print("before functionB")
def functionB():
    print("Function B {}".format(math.sqrt(100)))

print("before __name__ guard")
if __name__ == '__main__':
    functionA()
    functionB()
print("after __name__ guard")
Special Variables
When the Python interpreter reads a source file, it first defines a few special variables. In this case, we care about the __name__ variable.

When Your Module Is the Main Program

If you are running your module (the source file) as the main program, e.g.

python foo.py
the interpreter will assign the hard-coded string "__main__" to the __name__ variable, i.e.

# It's as if the interpreter inserts this at the top
# of your module when run as the main program.
__name__ = "__main__" 
When Your Module Is Imported By Another

On the other hand, suppose some other module is the main program and it imports your module. This means there's a statement like this in the main program, or in some other module the main program imports:

# Suppose this is in some other main program.
import foo
The interpreter will search for your foo.py file (along with searching for a few other variants), and prior to executing that module, it will assign the name "foo" from the import statement to the __name__ variable, i.e.

# It's as if the interpreter inserts this at the top
# of your module when it's imported from another module.
__name__ = "foo"
Executing the Module's Code
After the special variables are set up, the interpreter executes all the code in the module, one statement at a time. You may want to open another window on the side with the code sample so you can follow along with this explanation.

Always

It prints the string "before import" (without quotes).

It loads the math module and assigns it to a variable called math. This is equivalent to replacing import math with the following (note that __import__ is a low-level function in Python that takes a string and triggers the actual import):

# Find and load a module given its string name, "math",
# then assign it to a local variable called math.
math = __import__("math")
It prints the string "before functionA".

It executes the def block, creating a function object, then assigning that function object to a variable called functionA.

It prints the string "before functionB".

It executes the second def block, creating another function object, then assigning it to a variable called functionB.

It prints the string "before __name__ guard".

Only When Your Module Is the Main Program

If your module is the main program, then it will see that __name__ was indeed set to "__main__" and it calls the two functions, printing the strings "Function A" and "Function B 10.0".
Only When Your Module Is Imported by Another

(instead) If your module is not the main program but was imported by another one, then __name__ will be "foo", not "__main__", and it'll skip the body of the if statement.
Always

It will print the string "after __name__ guard" in both situations.
Summary

In summary, here's what'd be printed in the two cases:

# What gets printed if foo is the main program
before import
before functionA
before functionB
before __name__ guard
Function A
Function B 10.0
after __name__ guard
# What gets printed if foo is imported as a regular module
before import
before functionA
before functionB
before __name__ guard
after __name__ guard
Why Does It Work This Way?
You might naturally wonder why anybody would want this. Well, sometimes you want to write a .py file that can be both used by other programs and/or modules as a module, and can also be run as the main program itself. Examples:

Your module is a library, but you want to have a script mode where it runs some unit tests or a demo.

Your module is only used as a main program, but it has some unit tests, and the testing framework works by importing .py files like your script and running special test functions. You don't want it to try running the script just because it's importing the module.

Your module is mostly used as a main program, but it also provides a programmer-friendly API for advanced users.

Beyond those examples, it's elegant that running a script in Python is just setting up a few magic variables and importing the script. "Running" the script is a side effect of importing the script's module.

Food for Thought
Question: Can I have multiple __name__ checking blocks? Answer: it's strange to do so, but the language won't stop you.

Suppose the following is in foo2.py. What happens if you say python foo2.py on the command-line? Why?

# Suppose this is foo2.py.
import os, sys; sys.path.insert(0, os.path.dirname(__file__)) # needed for some interpreters

def functionA():
    print("a1")
    from foo2 import functionB
    print("a2")
    functionB()
    print("a3")

def functionB():
    print("b")

print("t1")
if __name__ == "__main__":
    print("m1")
    functionA()
    print("m2")
print("t2")
      
Now, figure out what will happen if you remove the __name__ check in foo3.py:
# Suppose this is foo3.py.
import os, sys; sys.path.insert(0, os.path.dirname(__file__)) # needed for some interpreters

def functionA():
    print("a1")
    from foo3 import functionB
    print("a2")
    functionB()
    print("a3")

def functionB():
    print("b")

print("t1")
print("m1")
functionA()
print("m2")
print("t2")
What will this do when used as a script? When imported as a module?
# Suppose this is in foo4.py
__name__ = "__main__"

def bar():
    print("bar")
    
print("before __name__ guard")
if __name__ == "__main__":
    bar()
print("after __name__ guard")
Share
Improve this answer
Follow
edited yesterday
answered Jan 7 '09 at 4:26

Mr Fooz
94.4k55 gold badges6565 silver badges9595 bronze badges
21
Out of curiosity: What hapens if I run subprocess.run('foo_bar.py') in a python script? I suppose that foo_bar will be started with __name__ = '__main__' just like when I tipe foo_bar.py in cmd manually. Is that the case? Taking @MrFooz' Answer into account there should not be any problem doing this and having as many "main" modules at a time as I like. Even changing the __name__ value or having several independantly creates instances (or instances that created each other by subprocess) interact with each other should be business as usual for Python. Do I miss something? ??? hajef Feb 18 '19 at 16:09 
21
@hajef You're correct about how things would work with subprocess.run. That said, a generally better way of sharing code between scripts is to create modules and have the scripts call the shared modules instead of invoking each other as scripts. It's hard to debug subprocess.run calls since most debuggers don't jump across process boundaries, it can add non-trivial system overhead to create and destroy the extra processes, etc. ??? Mr Fooz Feb 19 '19 at 16:16
5
i have a doubt in foo2.py example in the food for thought section.what does from foo2.py import functionB do? In my view it just imports foo2.py from functionB ??? user471651 Feb 24 '19 at 13:47
4
One of the modules that may import your code is multiprocessing, in particular making this test necessary on Windows. ??? Yann Vernier Sep 17 '19 at 15:51
3
Extremely minor point, but I believe python actually determines the __name__ of an imported module from the import statement, not from stripping ".py" off the filename. Because python identifiers are case sensitive but file names may not be (e.g. on windows), there isn't necessarily enough information in the filename to determine the correct python module name.
