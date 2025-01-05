# The Dassie Programming Language
**Dassie** is a statically typed high-level programming language designed around conciseness, expressiveness, simplicity and ease of use. Dassie is designed to be run on the .NET platform, and as such, can be JIT- or AOT-compiled to run on a wide range of systems.

## Getting started
To get started, install a Dassie compiler like the reference implementation at [loschsoftware/dc](https://github.com/loschsoftware/dc).

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

## Documentation
The [docs](./docs) directory of this repository contains a thorough language reference. Example programs can be found in the [examples](./examples) directory.
