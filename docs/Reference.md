# Dassie Language Reference
**Dassie** is a statically typed high-level programming language designed around conciseness, expressiveness, simplicity and ease of use. Dassie is designed to be run on the .NET platform, and as such, can be JIT- or AOT-compiled to run on a wide range of systems. As a result, this reference deals with certain .NET-specific constructs and requires basic knowledge of .NET and general programming principles. It is not geared towards complete programming beginners.

This reference is not a formal specification, it is merely designed to be a practical introduction to Dassie and to give an overview over Dassie syntax, semantics and language features. This document describes the full feature set of version 1.0 of the Dassie programming language.

|**Table of contents**|
|--|

# 1. Program structure
A Dassie source file (``*.ds``) can have one of the two following structures:
- **Full file:**
  1. Import and export directives
  2. Type definitions

  If this file structure is chosen and the application is executable (that is, not a library), one method needs to be marked as the application entry point using the ``<EntryPoint>`` attribute.

- **Top-level code:**
  1. Import and export directives
  2. Function-level code elements (expressions and non-expression elements as per [section 3](#3-non-expression-code-elements))

  Only one such file can exist per project. It serves as the default entry point of the application, disallowing any other method to be declared as an ``<EntryPoint>``.

A set of multiple source files that form a Dassie program are referred to as a **project**. A project always produces exactly one **assembly**, which is an executable or library file. A software that is made up of multiple projects and assemblies is referred to as a **project group**.

Type definitions as well as expressions are covered in the latter sections of this document, this section focuses only on fundamental concepts that are used throughout the language.

## 1.1 Comments
Comments are ignored by the compiler and have no effect on the program. They are merely meant to provide complementary remarks about the source code to developers.

````dassie
# This is a comment that spans a single line.
#[
  This is a comment that
  spans multiple lines.
]#
````

### 1.1.1 Documentation comments
**XML Documentation comments** are used to document code elements which are part of a public API, allowing IDEs (Integrated Development Environments) to display additional information about those code elements. For more information, including a list of valid tags, visit [this page](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/xmldoc/).

In Dassie, documentation comments are defined using two hash symbols at the start of every line of the comment.
````dassie
## <summary>
## Computes the sum of two integers.
## </summary>
## <param name="a">The first summand.</param>
## <param name="b">The second summand.</param>
## <returns>The sum of <paramref name="a"/> and <paramref name="b"/>.</returns>
Add (a: int, b: int) = a + b
````

## 1.2 Identifiers
**Identifiers** are the names used to access code elements such as variables, functions, parameters, types along with their members and namespaces.
````dassie
x = 10
````
In the above example, the value ``10`` is assigned to the local variable ``x``, where "x" is the identifier of the variable.

### 1.2.1 Naming rules
- An identifier must start with a letter or underscore (``_``).
- Identifiers may only contain Unicode letters (classes Lu, Ll, Lt, Lm, and Nl), decimal digits (Nd), connecting characters (Pc), combining characters (Mn, Mc) and formatting characters (Cf).

### 1.2.2 Backtick identifiers
Enclosing an identifier in **backticks** allows it to contain arbitrary Unicode characters. This also makes it possible for identifiers to be the same as reserved words. The backtick itself cannot be used in the identifier.
````dassie
`Print to standard output` (message: string) = println message
`Print to standard output` "Hello World!"
````

## 1.3 Namespaces
**Namespaces** provide a way of organizing types with related functionality into named containers.

Namespace are defined through *export directives*. An export defines the namespace of the entire file; a Dassie source file can contain at most one namespace.

````dassie
export Application.Tools
module Example = {}
````
In the above example, the module ``Example`` is contained in the namespace ``Application.Tools``. Consequently, the full name of the module is now ``Application.Tools.Example``.

### 1.3.1 Imports
By **importing** a namespace, all containing types and modules can be accessed by identifier only, instead of having to fully specify the namespace.

Dassie provides two ways of importing a namespace:
- **Import directives** at the start of a source file
````dassie
# Local imports
# Imports one or more namespaces in the current file.

import System
import System.Text, System.Linq

# Global imports
# Imports one or more namespaces in all files of the current project.

!import System.IO
!import System.Reflection, System.Reflection.Metadata
````
- The **intrinsic functions** **``localImport``** and **``globalImport``**

#### 1.3.1.1 Module imports
Import directives are also valid on [modules](#413-modules), which makes their members globally accessible by name.
````dassie
import System.Console
WriteLine "Hello World!"
````

### 1.3.2 Aliases
Using import directives, an **alias** can be set for namespaces and types. The original name is not replaced and can still be used.
````dassie
import io = System.Console
io.WriteLine "Hello World!"
````

#### 1.3.2.1 Default aliases
Dassie provides a set of **default aliases** for commonly used .NET types, as shown in the following table.
|.NET type|Alias|
|---|---|
|``System.SByte``|``int8``|
|``System.Byte``|``uint8``|
|``System.Int16``|``int16``|
|``System.UInt16``|``uint16``|
|``System.Int32``|``int32``, ``int``|
|``System.UInt32``|``uint32``, ``uint``|
|``System.Int64``|``int64``|
|``System.UInt64``|``uint64``|
|``System.Single``|``float32``|
|``System.Double``|``float64``|
|``System.Decimal``|``decimal``|
|``System.IntPtr``|``native``|
|``System.UIntPtr``|``unative``|
|``System.Boolean``|``bool``|
|``System.String``|``string``|
|``System.Char``|``char``|
|``System.Void``|``null``|
|``System.Object``|``object``|

# 2. Expressions
**Expressions** are code elements that represent or produce a **value**. In Dassie, almost all code elements inside of a function body are expressions.

## 2.1 Literals
**Literals** are constant **values** known at compile-time.

### 2.1.1 Numeric literals
**Numeric literals** represent integral as well as floating-point numbers of various sizes.

The character ``'`` can be placed anywhere in the literal and is ignored by the compiler. Its purpose is to increase readability for large numbers such as ``1'000'000`` (as opposed to ``1000000``).

#### 2.1.1.1 Integer literals
**Integer literals** are made up of an optional sign, the numeric value and an optional suffix specifying the data type of the literal. For integer literals, Dassie also supports **binary literals** (starting with ``0b``) and **hexadecimal literals** (starting with ``0x``). Here are a few examples for valid integer literals:
````dassie
10
-5
0x2B
-4l
3'000'000
0b10110
````

Integer literals map to the following .NET types:
|Type alias|.NET type|Size|Minimum value|Maximum value|
|---|---|---|---|---|
|``int8``|``System.SByte``|1 byte|-127|128|
|``uint8``|``System.Byte``|1 byte|0|255|
|``int16``|``System.Int16``|2 bytes|-32,768|32,767|
|``uint16``|``System.UInt16``|2 bytes|0|65,535|
|``int32``|``System.Int32``|4 bytes|-2,147,483,648|2,147,483,647|
|``uint32``|``System.UInt32``|4 bytes|0|4,294,967,295|
|``int64``|``System.Int64``|8 bytes|-9,223,372,036,854,775,808|9,223,372,036,854,775,807|
|``uint64``|``System.UInt64``|8 bytes|0|18,446,744,073,709,551,615|

#### 2.1.1.2 Real literals
**Real literals** only support base 10 (decimal). They are allowed to start with a leading period and support scientific notation.
````dassie
2.5
.3s
-10m
2.4e3
````

Real literals map to the following .NET types:
|Type alias|.NET type|Size|Accuracy|Approximate range|
|---|---|---|---|---|
|``float32``|``System.Single``|4 bytes|~6-9 digits|±1.5 x 10⁻⁴⁵ to ±3.4 x 10³⁸|
|``float64``|``System.Double``|8 bytes|~15-17 digits|±5.0 × 10⁻³²⁴ to ±1.7 × 10³⁰⁸|
|``decimal``|``System.Decimal``|16 bytes|28-29 digits|±1.0 x 10⁻²⁸ to ±7.9228 x 10²⁸|

#### 2.1.1.3 Type suffixes
The following table lists all type suffixes supported by numeric literals:
|Type alias|.NET type|Suffix|
|---|---|---|
|``int8``|``System.SByte``|``sb``|
|``uint8``|``System.Byte``|``b``|
|``int16``|``System.Int16``|``s``|
|``uint16``|``System.UInt16``|``us``|
|``uint32``, ``uint``|``System.UInt32``|``u``|
|``int64``|``System.Int64``|``l``|
|``uint64``|``System.UInt64``|``ul``|
|``native``|``System.IntPtr``|``n``|
|``unative``|``System.UIntPtr``|``un``|
|``float32``|``System.Single``|``s``|
|``float64``|``System.Double``|``d``|
|``decimal``|``System.Decimal``|``m``|

### 2.1.2 Boolean literals
**Boolean literals** are of type ``System.Boolean`` and are supported through the keywords **``true``** and **``false``**.

### 2.1.3 Character literals
A **character literal** represents a single UTF-16 code unit as an instance of ``System.Char``. The character is enclosed in single quotes (``'``). Character literals can contain [escape sequences](#2141-escape-sequences). Empty character literals are not allowed.
````dassie
letter = 'A'
````

### 2.1.4 String literals
A **string literal** represents a sequence of zero or more UTF-16 characters as an instance of ``System.String``. The string is enclosed in double quotes (``"``) and is **immutable**.
````dassie
message = "Hello World!"
````

#### 2.1.4.1 Escape sequences
**Escape sequences** start with ``^`` and are replaced with a specific character. They provide an easier way of adding commonly used special characters such as line breaks to a string. The following table lists all escape sequences and their replacement character.

|Escape sequence|Replacement character|Unicode encoding|
|---|---|---|
|``^^``|Caret|0x005E|
|``^"``|Double quote|0x0022|
|``^'``|Single qoute|0x0027|
|``^0``|Null|0x0000|
|``^a``|Alert|0x0007|
|``^b``|Backspace|0x0008|
|``^e``|Escape|0x001B|
|``^f``|Form feed|0x000C|
|``^n``|New line|0x000A|
|``^r``|Carriage return|0x000D|
|``^t``|Horizontal tab|0x0009|
|``^v``|Vertical tab|0x000B|
|``^u``|Unicode escape sequence (UTF-16)|``^uHHHH`` (range: 0000 - FFFF; example: ``^u00E7`` = "ç")|
|``^U``|Unicode escape sequence (UTF-32)|``^U00HHHHHH`` (range: 000000 - 10FFFF; example: ``^U0001F47D`` = "👽")|
|``^x``|Unicode escape sequence (variable length)|``^xH[H][H][H]`` (range: 0 - FFFF; example: ``^x00E7`` or ``^x0E7`` or ``^xE7`` = "ç")|
|``^{...}``|See chapter [String interpolation](#21411-string-interpolation)||


> [!WARNING]  
> When using the ``^x`` escape sequence and specifying less than 4 hex digits, if the characters that immediately follow the escape sequence are valid hex digits (i.e. 0-9, A-F, and a-f), they will be interpreted as being part of the escape sequence. For example, ``^xA1`` produces "¡", which is code point U+00A1. However, if the next character is "A" or "a", then the escape sequence will instead be interpreted as being ``^xA1A`` and produce "ਚ", which is code point U+0A1A. In such cases, specifying all 4 hex digits (for example, ``^x00A1``) prevents any possible misinterpretation.

##### 2.1.4.1.1 String interpolation
The special escape sequence ``^{}`` is used to directly embed Dassie expressions into strings.
````dassie
name = "John"
age = 22

display = "Name: ^{name}, Age: ^{age}"
println display
# -> "Name: John, Age: 22"
````

##### 2.1.4.1.2 Verbatim string literals
By prefixing a string literal with ``^``, all escape sequences are disabled and treated literally. To escape quotation marks, use ``""``.
````dassie
text = ^"This string contains no newlines.^n It is a ""verbatim string""."
````

### 2.1.5 Empty literal
The **empty literal** ``()`` represents a .NET null reference. By default, the type of the literal is ``System.Object``.

## 2.2 Basic expressions
These are miscellaneous expressions that don't fit into any other category.

### 2.1.1 Placeholders
**Placeholders** represent no value, but can be used in places where expressions are used. They are commonly used to set the body of a function that is not yet implemented. Placeholders are defined using the "``.``" (dot) symbol.
````dassie
Add (a: int, b: int) = .
````

### 2.2.2 Blocks
**Blocks** are used to group together multiple expressions. The value of the block is the value of the last expression inside of it. Code blocks are delimited using braces (``{ }``).
````dassie
println {
  a = 2
  b = 2
  a + b
}

# Prints "4"
````

A secondary function of blocks is **scoping** of local variables. A local declared in a block is only accessible inside of that block.
````dassie
{
  var x = 10
}

x = 20 # Error: 'x' is out of scope.
````

### 2.2.3 Wildcards
**Wildcards** are used in pattern matching to specify that the value of a pattern is irrelevant. They are declared using an underscore ``_``. For more information about pattern matching, visit chapter [2.5 Pattern matching](#25-pattern-matching).

### 2.2.4 Full range expression
The expression ``..`` with no operands specifies a ``System.Range`` encompassing all elements of a collection, as opposed to the unary and binary **range operators** seen in [2.3 Operators](#23-operators).

## 2.3 Operators
Dassie provides built-in behavior for arithmetic, comparison, logical, bitwise and equality operators as well as the ability to **overload** them, that is to define custom operator behavior for user-defined types.

### 2.3.1 Unary operators
**Unary operators** are operators with one operand. The following table lists all unary operands with decreasing precedence.
|Name|Operator|Category|Description|
|---|---|---|---|
|Complement|``~a``|Bitwise|Computes the ones complement of its operand.|
|Logical negation|``!a``|Boolean logic|Negates its operand.|
|Reference|``&a``||Takes the reference (memory location) of its operand.|
|Delegate|``func a``||Creates a managed delegate of type ``System.Action`` or ``System.Func`` that encapsulates its operand.|
|Function pointer|``func& a``||Returns the raw function pointer of its operand.|
|Type of|``^\a``|Type system|Returns the data type of its operand as a ``System.Type`` object. Only valid on identifiers.|
|Name of|``$\a``||Returns the name (identifier) of its operand.|
|Open-ended unary range operator|``a..``||Creates an object of type ``System.Range`` representing a range from index ``a`` to the end of the collection.|
|Closed-ended unary range operator|``..a``||Creates an object of type ``System.Range`` representing a range from the start of the collection to the specified index.|
|Grouping operator|``(a)``||Used to change the default operator precedence.|
|Expression separator|``a;``||Denotes the end of an expression.|

### 2.3.2 Binary operators
**Binary operators** are operators with two operands. The following table lists all binary operands with decreasing precedence.
|Name|Operator|Category|Description|
|---|---|---|---|
|Power|``a ** b``|Arithmetic|Computes the value of ``a`` raised to the power ``b``.|
|Multiplication|``a * b``|Arithmetic|Computes the product of its operands.|
|Division|``a / b``|Arithmetic|Computes the quotient of its operands.|
|Remainder|``a % b``|Arithmetic|Computes the remainder of an integer division of its operands.|
|Modulus|``a %% b``|Arithmetic|Performs the modulus operation, where ``a %% b = (a % b + b) % b``.|
|Addition|``a + b``|Arithmetic|Computes the sum of its operands.|
|Concatenation|``a + b``||Overloaded for strings and lists. Concatenates its operands into a new string or list.|
|Subtraction|``a - b``|Arithmetic|Computes the difference of its operands.
|Left shift|``a << b``|Bitwise|Shifts ``a`` to the left by ``b`` bits.|
|List append|``a << b``|Lists|Returns a new list corresponding to ``a`` with element ``b`` added at the last index.|
|Right shift|``a >> b``|Bitwise|Shifts ``a`` to the right by ``b`` bits.|
|Equality|``a == b``|Equality|Checks wheter its operands are equal.|
|Inequality|``a != b``|Equality|Checks wheter its operands are inequal.|
|Less than|``a < b``|Comparison|Checks wheter ``a`` is less than ``b``.|
|Less than or equal|``a <= b``|Comparison|Checks wheter ``a`` is less than or equal to ``b``.|
|Greater than|``a > b``|Comparison|Checks wheter ``a`` is greater than ``b``.|
|Greater than or equal|``a >= b``|Comparison|Checks wheter ``a`` is greater than or equal to ``b``.|
|Type compatibility check|``a :? b``|Type system|Checks wheter ``a`` compatible with type ``b``.|
|Bitwise conjunction|``a & b``|Bitwise|Computes the conjunction of its operands.|
|List intersection|``a & b``|Lists|Returns a new list containing all elements contained in both of its operands.|
|Logical conjunction|``a && b``|Boolean logic|Computes the logical conjunction of its operands.|
|Bitwise disjunction|``a \| b``|Bitwise|Computes the disjunction of its operands.|
|List union|``a \| b``|Lists|Same as list concatenation, except that duplicate elements are filtered out.|
|Logical disjunction|``a \|\| b``|Boolean logic|Computes the logical disjunction of its operands.|
|Exclusive OR|``a ^ b``|Bitwise|Computes the exclusive OR (XOR) of its operands.|
|Symmetric difference|``a ^ b``|Lists|Computes the symmetric difference of two lists, that is a list containing all elements that are contained in either, but not both of its operands.|
|Conversion|``a <: b``|Type system|Converts ``a`` to data type ``b``. Throws an exception at runtime if unsuccessful.|
|Safe conversion|``a <?: b``|Type system|Converts ``a`` to data type ``b``. Returns ``()`` (a null reference) if unsuccessful.|
|Index|``a::b``||Retrieves the element of ``a`` at index ``b``.|
|Assignment|``a = b``||Assigns the value of ``b`` to ``a``.|
|Right pipe|``a -> b``||Calls function ``b`` with argument ``a``.|
|Left pipe|``a <- b``||Calls function ``a`` with argument ``b``.|
|Range|``a..b``||Creates an object of type ``System.Range`` with the specified parameters.|

All operators except the exponentiation operator ``**`` are left-associative.

#### 2.3.3 Compound assignment expressions
The binary operators ``+``, ``-``, ``*``, ``/``, ``**``, ``%``, ``<<``, ``>>``, ``|``, ``||``, ``&``, ``&&`` and ``^`` can be combined with the assignment operator ``=`` to directly assign the value of the operation to its left operand.
````dassie
a += 5
# is equal to
a = a + 5
````

### 2.3.3 Specifics of arithmetic operators
When arithmetic operators are applied to operands of different types, the result will always have the type of the first operand.
````dassie
a = 2.5
b = 3

c = a + b # 5.5
d = b + a # 5, since a is truncated before adding
````

## 2.4 Control flow
**Control flow** is used to dictate the order in which instructions are executed in a program. They allow the program to make decisions, repeat certain actions, and manage the flow of execution based on specific conditions. Dassie provides several control flow constructs, including conditionals and loops.

The symbol ``=`` has the double meaning of serving as the assignment operator as well as separating the logical segments of control flow expressions, error handling constructs as well as type and member definitions.

### 2.4.1 Conditionals
**Conditional expressions** are used to execute code only if a specified condition is met. To achieve this, Dassie provides support for **'if' expressions** as well as **'unless' expressions**.

#### 2.4.1.1 'If' expressions
This conditional expression executes a piece of code only **if** the specified condition is met. If the condition is not met, the **else** branch is executed instead. 'If' expressions can be chained together to form *else if* branches.

The **conditional operator** **``?``** is used to associate a condition with an expression. The colon (**``:``**) is used to define the *else if* and *else* branches.
````dassie
age = rdint "Enter your age: "
println ? age >= 18 = "Welcome!"
: = "You are too young to use this service. :("
````

Conditional expressions can also be applied in **postfix form**.
````dassie
age = rdint "Enter your age: "
{ println "Welcome!" } ? age >= 18
````

#### 2.4.1.2 'Unless' expressions
Prefixing the conditional operator ``?`` with an exclamation mark (**``!``**) negates the condition, making it only execute if the condition evaluates to ``false``.
````dassie
age = rdint "Enter your age: "
{ println "Welcome!" } !? age < 18
````

### 2.4.2 Loops
**Loops** are used to execute a block of code **multiple times**. There are three basic variants of loops:
- *Count loops* that execute code a set number of times
- *Conditional loops* that execute code as long as a condition is met
- *Iteration loops* that execute code for every element of a collection

Unlike traditional imperative languages, Dassie provides no mechanisms to end the execution of a loop early (such as the ``break`` and ``continue`` keywords found in many C-like languages).

In Dassie, the **looping operator ``@``** is used to create a loop. It can be applied both in prefix as well as in postfix form. The **value** of a loop expression is an object of type ``System.Collections.Generic.List[T]`` containing the values of each iteration.

#### 2.4.2.1 Count loops
**Count loops** execute a block of code a set number of times. The below code will print "Hello World!" exactly 10 times.
````dassie
@ 10 = {
  println "Hello World!"
}
````

#### 2.4.2.2 Conditional loops
**Conditional loops** execute a block of code as long as a specified condition is met. They are also known as **while loops**.
````dassie
var count = 0
@ count < 10 = {
  println count
  count += 1
}
````
Like with conditional expressions, prepending ``!`` to the loop operator will negate the condition, making the loop execute **until** a condition is met. This is called **until loop**.

#### 2.4.2.3 Iteration loops
**Iteration loops** execute a block of code for each element in a collection, providing access to each element through a so called **iteration variable**. The following code uses an iteration loop to list all command-line arguments passed to the application:
````dassie
@ arg :> args = {
  println arg
}
````
In this code, ``args`` represents the collection to loop over and ``arg`` is the iteration variable.

## 2.5 Pattern matching
**Pattern matching** is used to classify and transform an input expression based on specified rules, so-called **patterns**. In their most simple form, match expressions act similar to ``switch`` statements known from C-like languages. The **match operator ``$``** is used to initiate a pattern matching block:
````dassie
x = rdint "Enter a number: "
name = $ x = {
  ? 0 = "Zero"
  : 1 = "One"
  : 2 = "Two"
  : 3 = "Three"
  : 4 = "Four"
  : 5 = "Five"
  : 6 = "Six"
  : 7 = "Seven"
  : 8 = "Eight"
  : 9 = "Nine"
  : 10 = "Ten"
  : = "Unknown number"
}
````
More complex patterns are not yet supported.

## 2.6 Local values and variables
Through **local values** and **variables**, a value can be assigned to a name, allowing it to be reused multiple times.

**Local values** are assigned using the keyword ``val``, which is optional. They are **immutable**, meaning they cannot be modified after creation.
````dassie
val x = 5
y = 10

x = 10 # Error
````

**Local variables** are defined using the keyword ``var``. They are **mutable**, which means they can be reassigned after creation. Reassignment uses no keyword.
````dassie
var x = 5 # Declaration
x = 10 # Reassignment
````

Dassie requires locals to be initialized when created, declaration without initialization is not possible.

As the name implies, locals are bound to the **local scope**, which is either a code block or the current function.

### 2.6.1 Type inference and explicit typing
The Dassie compiler automatically **infers** the data type of a variable when it is assigned. Dassie is statically typed, so the data type cannot be changed after initialization.

In some contexts, it might be necessary or preferable to manually specify the type of a local. This is accomplished by adding a colon (``:``) after the name.
````dassie
x: int = 5
text: string = "Hello World!"
````

### 2.6.2 Reference and value semantics
Variables whose data type is a **reference type** (``ref type``) store a **reference** to the underlying instance, as opposed to the value itself. When a second variable is initialized to the value of a reference, the reference is copied, but the underlying instance remains the same.
````dassie
import System.Text

sb1 = StringBuilder
sb2 = sb1

sb2.Append "A"
println sb1.ToString
# -> "A"
````

Variables of **value types** (``val type``, including all primitive types) store the value directly. When such a variable is referenced by another one, the value is **copied**. The *reference operator* **``&``** can be used to retrieve a reference to such a value.
````dassie
var x = 10
var y = x

y = 20
println x
# -> 10
````

## 2.7 Function expressions (Lambda expressions, Anonymous functions)
**Lambda expressions** provide a way of quickly defining small helper functions that are only used once. They have no name and are commonly used as part of LINQ queries. Anonymous functions are defined using the **arrow operator ``=>``** with a parameter list on the left and an expression on the right side.

Lambda expressions can **capture** variables from their enclosing scope, forming a **closure**.

The value of a lambda expression is an unmanaged function pointer. The keyword **``func``** is used to convert this function pointer to a managed delegate type, namely ``Action[T]``. These delegates can then be executed using their ``Invoke`` method.

The following code example uses a lambda expression that captures the variable ``x`` from the containing scope and adds a value to it.
````dassie
var x = 10
add = (func y: int => x + y)
println add.Invoke 5
x = 20
println add.Invoke 5

# Output:
15
25
````

### 2.7.1 Local functions
Functions can be nested. Nested functions are called **local functions** and are compiled the same as lambda expressions. They behave the same as anonymous functions, including being able to capture variables from their enclosing scope.

For more information about functions in general, visit chapter [4.3.2 Methods](#432-methods).

## 2.8 Type conversions
The **conversion operator ``<:``** is used to change the data type of an expression. This operation is commonly called **casting**, or when converting value types to and from ``object``, **boxing** and **unboxing**.
````dassie
x = 2.5
y = x <: int
println x
println y

# Output:
2.5
2
````
When performing arithmetic operations on primitive types, conversions are performed implicitly.

The conversion operator ``<:`` throws an exception of type ``System.InvalidCastException`` at runtime when a conversion is invalid. For more safety, the **safe conversion operator ``<?:``** might be preferred, which instead returns a null reference (``()``) if it fails.

## 2.9 Pipes
**Pipes** are a useful way of chaining function calls. There are two kinds of pipes:
- The **left pipe ``f <- x``** calls the function on its left side with the argument on its right side. This behavior is the same as an ordinary function call.
- The **right pipe ``x -> f``** calls the function on its right side with the argument on its left side.

## 2.10 Collection expressions
The term **collection** refers to data structures that can hold multiple values. Dassie provides special syntax for creating different kinds of collections, namely **arrays**, **lists**, **dictionaries** and **tuples**.

### 2.10.1 Arrays
**Arrays** are the simplest form of collection. They have a fixed length and can only contain elements of the same type. To create them, the **``@[...]``** syntax is used:
````dassie
nums = @[1, 2, 3, 4, 5]
````
The data type of the above expression is ``int@[]``.

#### 2.10.1.1 Array dimensions
Initialization of arrays with dimensions greater than one is not yet supported. To refer to their type, the ``T@[,,,]`` syntax is used, where the amount of commas is equal to the dimension of the array.

### 2.10.2 Lists
**Lists** are instances of the generic type ``System.Collections.Generic.List[T]``. They are similar to arrays, with the important addition of being dynamically resizable; elements can be added and removed from them using the ``Add`` and ``Remove`` methods.
````dassie
numberList = [1, 2, 3, 4, 5]
numberList.Add 6
numberList.Remove 2
````

### 2.10.3 Dictionaries
**Dictionaries** are lists of key-value-pairs. They are instances of the generic type ``System.Collections.Generic.Dictionary[T]``.
````dassie
people = {
  {"John", 22},
  {"Alex", 26},
  {"Marcus", 18}
}
````
In the current version of the Dassie compiler, dictionary literals are not yet supported.

### 2.10.4 Tuples
**Tuples** are immutable and can contain values of different types. They are instances of the generic type ``System.ValueTuple[T1, T2, ...]``.
````dassie
data = ("John", 22, 1.82)
````
The data type of the above expression is ``System.ValueTuple[string, int, float64]`` or ``(string, int, float64)``.

# 3. Non-expression code elements
While most code elements in Dassie produce a value, there are certain constructs that are not realistically able to and cannot be used as an expression.

## 3.1 Exception handling
Dassie supports .NET exceptions through the keywords **``raise``** to raise exceptions and **``try``**, **``catch``** and **``finally``** to handle them.
````dassie
try = {
  raise NullReferenceException
}
catch nre: NullReferenceException = {
  println "NullReferenceException thrown"
}
catch = {
  println "Other exception thrown"
}
finally = {
  println "End of block"
}
````

# 4. Types
#### Type names
The following table lists all possible kinds of type names:
|Name|Syntax|Description|
|---|---|---|
|Regular type|``T``|A regular, non-generic type.|
|Generic type|``T[U]``|A type with a generic type argument.|
|By-Ref type|``T&``|A type with reference semantics.|
|Vector (SZ array)|``T@[]``|A single-dimension, zero-indexed array.|
|Multidimensional array|``T@[,]``|A multidimensional array, determined by the amount of commas.|
|Tuple|``(T1, T2, ...)``|A tuple type.|
|Function pointer|``func*[T1, T2, ..., TReturn]``|A function pointer type.|

## 4.1 Kinds of types
There are two kinds of types in Dassie: **reference types** and **value types**. Variables of reference types store references to their data (objects), while variables of value types directly contain their data. With reference types, two variables can reference the same object; therefore, operations on one variable can affect the object referenced by the other variable. With value types, each variable has its own copy of the data, and it's not possible for operations on one variable to affect the other (unless when using byref-types and the reference operator ``&``).

### 4.1.1 Reference types
All **reference types** implicitly inherit from ``System.Object`` and are defined using ``ref type`` (or just ``type`` for short).
````dassie
type Example = {
  # Members ...
}
````

### 4.1.2 Value types
**Value types** implicitly inherit from ``System.ValueType`` and are defined using ``val type``:
````dassie
val type Point = {
  X: float64
  Y: float64
}
````

#### 4.1.2.1 Immutable value types
When declaring a type using ``val! type``, the compiler ensures that the type contains no mutable members. This is useful in high-performance scenarios, as it allows for specialized optimizations.

#### 4.1.2.2 Byref-like types
A type declared as a ``val& type`` (called **byref-like type**) is always allocated on the stack and guaranteed to never escape to the heap. For more information, see [this page](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/ref-struct) for detailed information regarding ``ref struct`` types in C#, which are functionally equivalent to ``val& type`` in Dassie.

The ``&`` and ``!`` type modifiers can also be combined to form immutable byref-like types.

### 4.1.3 Modules
**Modules** are a special kind of type where all members are automatically marked **static** (see section [static methods](#43221-static-methods)) and can be referred to by the type name only. Modules cannot be instantiated. Modules can also be [imported](#1311-module-imports).
````dassie
module Helpers = {
  Add (a: int, b: int) = a + b
  Sub (a: int, b: int) = a - b
}
````
The above module can be used like so:
````dassie
x = Helpers.Add 2, 3
y = Helpers.Sub 4, x
````

### 4.1.4 Templates
A **template** (equivalent to ``interface`` types in C#) defines a contract. Any value type or reference type that *implements* that contract must provide an implementation of the members defined in the template. A template may define a default implementation for members.
````dassie
template SampleTemplate = {
  abstract SampleMethod (): null
}

type ImplementationType: SampleTemplate = {
  SampleMethod () = {
    # Implementation ...
  }
}
````
Templates cannot contain instance fields.

### 4.1.5 Literal types (Enumerations)
**Enumerations** are supported primarily for .NET interop scenarios. They are defined through the special attribute ``<Enumeration[T]>``, which instructs the compiler to:
- Set the base type of the enumeration to ``System.Enum``.
- Define a field of type ``T`` called ``value__``.

The only valid type arguments for ``<Enumeration[T]>`` are ``uint8``, ``int8``, ``uint16``, ``int16``, ``uint32``, ``int32``, ``uint64`` and ``int64``. The ``<Enumeration>`` attribute can also be used without type parameters, in which case it defaults to ``int32``.

All enumeration members have to be marked as ``literal``.

Here is an example that defines an enumeration type containing the days of the week:
````dassie
<Enumeration>
type Weekday = {
    literal = {
        Monday = 0
        Tuesday = 1
        Wednesday = 2
        Thursday = 3
        Friday = 4
        Saturday = 5
        Sunday = 6
    }
}
````

### 4.1.6 Union types
This is a placeholder for an upcoming description of union types. Currently, compiler support for union types is limited.

### 4.1.7 Delegates
**Delegates** are currently not supported through specialized syntax, but can be manually constructed. Here is an example that defines a delegate for a function of type ``(int, int) -> int``:
````dassie
type IntBinaryOperationDelegate: MulticastDelegate = {
    <RuntimeImplemented>
    <HideBySig>
    IntBinaryOperationDelegate (obj: object, method: native)
    
    <RuntimeImplemented>
    <HideBySig>
    <NewSlot>
    Invoke (a: int, b: int): int
}
````
The concurrency-related methods ``BeginInvoke`` and ``EndInvoke`` are optional.

## 4.2 Type modifiers

### 4.2.1 Access modifiers
The **access modifiers** **``global``** and **``internal``** are used to restrict access to types. Types marked as ``global`` (which is the default) can be used from anywhere inside the current assembly as well as any referencing assemblies. Types marked ``internal`` can only be accessed in the current assembly.
````dassie
type A = {}
global type B = {}
internal type C = {}
````
**Nested types** additionally support the access modifier ``local``, which allows only the parent type to access it.

### 4.2.2 Specialized modifiers
The **``open``** modifier is used to allow a type to be inherited from. By default, a type cannot be extended, it is **sealed**.
````dassie
type A = {}
type B: A = {} # Invalid, since A is sealed

open type A = {}
type B: A = {} # Valid
````

## 4.3 Members
**Type members** define storage locations and executable code of a type. Members are divided into *fields*, *constants*, *properties*, *methods*, *constructors* and *events*.

Members support the **access modifiers** ``global`` (member can be accessed from anywhere), ``internal`` (member can be accessed from anywhere in the current assembly), ``protected`` (member can be accessed from child types) and ``local`` (member can be accessed only from the containing type). The default access modifier is ``global``.

Instead of specifying access modifiers for each member individually, access modifiers can be grouped, like the following example shows:
````dassie
FirstName: string
LastName: string
local Id: int
local Password: string
local BirthDate: DateTime

# - is equivalent to -

FirstName: string
LastName: string

local = {
  Id: int
  Password: string
  BirthDate: DateTime
}
````

### 4.3.1 Fields
**Fields**, also called **member variables**, are used to store data within a type. They can be *instance fields*, which are specific to each instance of a type, or *static fields* (denoted using the keyword ``static``), which are shared among all instances of a type.

As opposed to local variables, fields are **mutable** by default. To make them immutable, the keyword ``val`` is used, which allows them to be modified only inside of constructors. Fields are not required to be initialized with a value, altough the data type needs to be explicitly declared in this case.

````dassie
X: int # Mutable field of type int
val Id: Guid # Immutable field of type Guid
Result = 0 # Mutable field initialized with value 0 (type int).
````

#### 4.3.1.1 Compile-time constants (``literal`` fields)
Fields declared using the **``literal``** modifier are required to be initialized with a compile-time constant value that cannot change. The constant value is inlined by the compiler, the field itself is never actually accessed.
````
literal PI = 3.141592653589793
literal E = 2.718281828459045
````

### 4.3.2 Methods
**Methods**, also called **member functions**, are functions that are defined within a type. They are used to perform operations on the data contained within the object or to manipulate the object's state. Methods can access and modify the object's members and can be called on instances of the class.

Here is an example method that adds two integers:
````dassie
Add (a: int, b: int) = a + b
````
The return type of the method is inferred from the expression, the parameter types need to be declared explicitly.

A method body consists of exactly one expression. Commonly, **block expressions** are used as the body. Early returns like in C-like languages are not supported.

#### 4.3.2.1 Inlined methods
Methods can be inlined with the **inline** modifier, which can improve performance in some cases.
````dassie
type T = {
  inline Add (a: int, b: int) = a + b
}
````

#### 4.3.2.2 Modifiers
**Modifiers** are placed before the name of the method to change their behavior.

##### 4.3.2.2.1 Static methods
Marking a method with the modifier **static** makes it independent of a specific instance of its containing type. As such, it has no access to instance members. Instead, it can be called on the type name itself. Overloaded operators are always **static**, as are members of **modules**.
````dassie
type Example = {
  PrintInstance (msg: string) = println msg
  static PrintStatic (msg: string) = println msg
}
````
These methods would be called as follows:
````dassie
inst = Example
inst.PrintInstance "Hello World!"

Example.PrintStatic "Hello World!"
````

##### 4.3.2.2.2 Virtual methods
**Virtual methods** can be **overriden** from inherited types. Methods of an ``open type`` are always virtual, unless when marked with ``closed``. Methods of a closed type can never be virtual.

#### 4.3.2.3 Parameters
**Parameters**, like local values, are always read-only, unless marked with the ``var`` modifier. Currently, Dassie does not support specifying default values of parameters, neither does it support a variable number of parameters (*varargs*).

##### 4.3.2.3.1 Pass-by-reference
**Pass-by-reference** is supported by declaring the type of a parameter as a *byref type* (denoted using a ``&``). In order to modify the value, the modifier ``var`` also needs to be prepended to the parameter name:
````dassie
Increment (var x: int&) = ignore x += 1
````

##### 4.3.2.3.2 Parameter constraints
**Parameter constraints** can be appended to a parameter and are enclosed in braces. The compiler will throw an error at compile time if a constraint is violated (**checked constraint**). If a question mark is prepended to the constraint, an exception is thrown at runtime instead (**unchecked constraint**).

````dassie
# Checked constraint: setting parameter b to 0 results in an error.
Divide (a: int, b: int { != 0 }) = a / b
````

#### 4.3.2.4 Constructors
A **constructor** is a special kind of method that is called when an instance of a type is created. It is used to initialize the data of the type and pass in configuration.

A constructor is defined as a method with the same name as the enclosing type. The only allowed return type is ``null`` (``System.Void``).
````dassie
type T = {
  local _x: int

  T (x: int) = ignore {
    _x = x
    println "New instance created!"
  }
}

t = T 20

# Output:
New instance created!
````

##### 4.3.2.4.1 Primary constructors
**Primary constructors** allow for a more concise way of declaring simple data types:
````dassie
val! type Point (X: float64, Y: float64)
````
In the above example, the compiler automatically creates instance properties for ``X`` and ``Y`` as well as a constructor. Additional functionality can be defined as normal:
````dassie
val! type Point (X: float64, Y: float64) = {
    Distance (other: Point) = {
        xs = Math.Pow (X - other.X), 2.0
        xy = Math.Pow (Y - other.Y), 2.0
        Math.Pow xs + xy, 0.5
    }
}
````

##### 4.3.2.4.2 Static constructors
Static constructors are not yet supported.

###### 4.3.2.4.2.1 Module initializers
Dassie supports the .NET attribute ``<ModuleInitializer>`` (namespace ``System.Runtime.CompilerServices``) which instructs the compiler to call the decorated method inside of the module initializer (``<Module>..ctor()``). Module initializers need to be non-generic, static methods with no parameters and no return value.
````dassie
<ModuleInitializer>
Init () = {
  println "Hello from <Module>..ctor()!"
}
````

#### 4.3.2.5 Finalizers
**Finalizers** are used to clean up unmanaged resources when objects are collected by the **garbage collector** (**GC**). Finalizers are defined like constructors with a prepended tilde (**``~``**). They cannot have parameters and they cannot return values.
````dassie
type Example = {
  ~Example () = {
    # Perform cleanup...
  }
}
````
Finalizers are only supported by **reference types**. For more general information on finalizers, visit [this page](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/finalizers).

Currently, the above syntax is not supported by the Dassie compiler.

#### 4.3.2.6 Extension methods
Dassie supports **extension methods** like in C# through the ``<Extension>`` attribute applied to a static method. Here is an example:
````dassie
<Extension>
static Double (num: int) = num * 2

x = 10
y = x.Double
````

### 4.3.3 Properties
As of the time of writing, **properties** are not yet fully supported by the Dassie compiler.

### 4.3.4 Events
.NET **events** are not yet supported by Dassie.

### 4.3.5 Operators
**Operators** can be overloaded by defining a method with the same name as the operator to overload. When overloading operators that are only one character long, the name needs to be encased in parentheses. Operators are automatically **static**. Here is an example that overloads the ``+`` (addition) and ``-`` (subtraction) operators for a custom type:
````dassie
type T = {
  Value: int
  (+) (a: T, b: T) = a.Value + b.Value
  (-) (a: T, b: T) = a.Value - b.Value
}
````

#### 4.3.5.1 Custom operators
Dassie also supports the definition of **custom operators**. When using only the symbols ``!$%&*+-./<=>?@^|~``, the syntax remains the same as when overloading default operators. Custom operator cannot take less than one (prefix unary operator) or more than two (infix binary operator) arguments.
````dassie
# In this case, the parentheses are optional
(>>?) (a: int, b: int) = a > (b * 100)

a = 10
b = 200
c = a >>? b
````
In addition, when delimiting the operator name in slashes (``/ /``), arbitrary Unicode characters can be used:
````dassie
/Σ/ (a: int, b: int) = a + b
x = 10 /Σ/ 20
````
This is very similar to methods with [backticked identifiers](#122-backtick-identifiers), except for supporting infix notation for binary operators.

## 4.4 Parametric polymorphism
**Parametric polymorphism** is supported through **generics**. Generics allow types and methods to be reused for different data types, specified throught **type parameters**, which can be used in place of types inside of a type or method. A commonly used generic type is ``List[T]``, which has one type parameter ``T`` that specifies the type of the list items.

Here is an example generic type:
````dassie
type Example[T] = {
  Value: T
}
````

Unlike C#, Dassie currently does not support overloading type parameter lists on types with the same name (as is done for ``Func[TRet]``, ``Func[T1, TRet]``, ``Func[T1, T2, TRet]``, ...).

### 4.4.1 Variance
Type parameters of templates support **variance modifiers**. The following three modifiers are supported:
- ``template Example[=T]``: **Invariance**, equivalent to ``[T]``.
- ``template Example[+T]``: **Covariance**: Permits a method to have a more derived return type than that defined by the generic type parameter of the template.
- ``template Example[-T]``: **Contravariance**: Permits a method to have argument types that are less derived than that specified by the generic parameter of the interface

For more information on variance, see [this page](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-generic-interfaces) about variance in C# interfaces.

### 4.4.2 Generic methods
**Methods** support generic type parameters just like types.

### 4.4.3 Type parameter constraints
Type parameters can be **constrained** to only accept types with specific properties. The following constraints are supported:
|Name|Syntax|Description|
|---|---|---|
|Reference type|``ref T``|Only accepts reference types.|
|Value type|``val T``|Only accepts value types.|
|Default constructor|``default T``|Only accepts types with a parameterless constructor.|
|Base type|``T: TBase``|Only accepts types inheriting from ``Base``.|
|Template|``T: TTemplate``|Only accepts types implementing ``Template``. Multiple, comma-separated templates can be specified.|

Here is an example that shows how these constraints are defined:
````dassie
type Example[default ref T, val U, V: IDisposable] = {}
````

## 4.5 Inheritance
Reference types, when marked as ``open``, can **inherit** from exactly one base type. Reference types, value types and templates can **implement** any number of templates.
````dassie
import System

open type Base = {}

# Inherits from 'Base', implements 'IDisposable'
type Child: Base, IDisposable = {
  # ...
}

template Template: IDisposable = {}

# Implements 'Template', which in turn implements 'IDisposable'
val type Example: Template = {
  # ...
}
````

Currently, overriding methods requires no special syntax apart from defining a method with the same signature that is to be overriden.

Apart from that, compiler support for inheritance is currently primitive.

# 5. Attributes
**Attributes** are used to add custom metadata to code elements such as types, members and parameters. Attributes are types inheriting from ``System.Attribute``. Applying an attribute is done using angle brackets (``<Attribute>``).

Compiler support for attributes is limited, currently only parameterless attributes on types and methods are suppoorted.

## 5.1 Application entry point
The **application entry point** is a method marked using the special attribute ``<EntryPoint>``. Only one method per project can be marked with this attribute. When using top-level code, a type named ``Program`` is automatically created, containing the entry point named ``Main``.