# The Dassie Programming Language
**Dassie** is a statically typed high-level programming language designed around conciseness, expressiveness, simplicity and ease of use. Dassie is designed to be run on the .NET platform, and as such, can be JIT- or AOT-compiled to run on a wide range of systems.

## Getting started
To get started, install a Dassie compiler like the reference implementation at [loschsoftware/dc](https://github.com/loschsoftware/dc). To try out the language, you can also use the online editor on [RyuGod](https://ryugod.com/pages/ide/dassie).

Assuming you have downloaded the Dassie compiler and registered it under the ``%PATH%`` environment variable (or the equivalent in other systems), the command ``dc`` should be available from the terminal. It is used to compile Dassie source files and manage Dassie projects using the configuration file ``dsconfig.xml`` that is at the root of every project.

To start programming in Dassie, create a file ending in ``.ds``. Here is the famous "Hello World" program written in Dassie:
````dassie
println "Hello World!"
````
This prints to the console using a built-in function called ``println``. Since Dassie uses .NET, you can also use its ``Console`` module like this, which should look familiar to .NET developers:
````dassie
import System
Console.WriteLine "Hello World!"
````
Assuming the above code is located in a file called ``hello.ds``, it can be compiled using the command ``dc hello.ds``, yielding an executable called ``hello.exe``. Alternatively, the command ``dc build`` can be used to compile all Dassie source files in the current directory and all subdirectories.

## Design principles
- All code must be contained in a type. Like C#, Dassie allows for one file per project to have *top-level code* that is part of an auto-generated type and is declared as the entry point of the application implicitly.
- Everything from variable assignment to conditionals and loops is an **expression**. A loop returns an array of the return values of each iteration.
- Dassie uses **operators** for control flow. For example, the operator ``?`` is used for conditionals instead of the commonly used keyword ``if``. Loops use the operator ``@``. Both conditionals and loops are available in a negated form, the so called *unless* and *until* expressions, represented by the operators ``!?`` and ``!@``. Conditional and loop expressions are usable in **prefix** and **postfix** form.

## Further information
A detailed documentation of the language syntax is available in the wiki of this repository. Code examples can be found in the **examples** directory.
