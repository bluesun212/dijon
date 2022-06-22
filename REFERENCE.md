# Dijon Reference Version 1.0

## Numbers
The simplest datatype in Dijon is the number.  There are two built-in numbers, 0 and 1, but only 1 is strictly necessary.  All other numbers may be created by dividing and subtracting these numbers in clever ways.  Numbers are represented by fractions in Dijon, and when two numbers are divided or subtracted, the corresponding fractional methods will be used, and the resulting fraction will be stored in simplified form.

## Variables
Variables are supported in Dijon, and are essential for data storage and control flow.  In a general sense, variables are containers that have a value and contain zero or more child variables.  The path from the base variable to one of its sub-children is referred to a variable tree, and is referenced by joining each variable with a period from the base to the end.  Variable names are made up of lowercase letters a-z, the underscore character _, and numbers 0-9.  Unlike many modern languages, variable names may start with a number, or event be completely comprised of numbers.  As an example, consider a variable named `foo`, and  a child variable named `bar`. `bar` may be accessed by the string `foo.bar`.

Some examples of variable names:
- `foo`
- `foo.bar.baz`
- `10`
- `i.0`
- `array.1234`

## Scopes and Triggers
The code in each file can be organized into triggers, which are the sections of a file starting with `:varname` and ending with the matching `;`.  In fact, the whole file is wrapped in its own root trigger, which contains all code outside other triggers.  The code in each trigger is run before retrieving the value of the variable `varname` so long that the scope that the trigger is part of is accessible from the current scope.  After the trigger is finished running, the code will continue to execute where it left off.  The scope refers to a collection of currently defined variables and triggers starting with the current trigger and ending with the file's root trigger.  When the program begins interpreting from either the beginning of a file, or the start of a trigger, a new scope is created.  To explain these concepts more succinctly, consider the following snippet:
```
:baz ... ;

:foo
    :bar baz ;
    
    bar
;

foo 
```

When the file is initially imported, the interpreter will record that there are two triggers, one being `baz`, and the other being `foo`, which has a child trigger called `bar`.  When the code is run, it will start with the outermost scope, which has access to both triggers `foo` and `baz` and their associated variables.  When the variable `foo` is accessed at the end, it will begin execution in a new scope starting inside the `foo` trigger.  Note that trying to access `foo.bar` will not run the internal trigger as it is outside the scope.  Inside the `foo` trigger, the code has access to its parent scope, as well as the new trigger `bar`.  When `bar` is accessed, its trigger will run, where it will access `baz`, then execute the corresponding trigger from the outermost scope.  It is possible to use triggers to recreate branching, looping, and recursion in Dijon.

## The Interpreter and the Stack
When a file or trigger is executed, the interpreter will begin enumerating through the code symbols.  Code symbols are comprised of "names" and "operations".  Names are strings with refer to the name of variables, or a sub-variable, and obey variable naming conventions.  There are a couple of reserved names which will be mentioned later.  Operations are single characters that cause some operation to be performed on the stack.  Currently, there are 6 operations in Dijon.  Line comments are also allowed with the `%` character.  Any characters aside from these are ignored.  When a name symbol is encountered, it is immediately pushed onto the stack.  The name may not refer to an actual variable, and the reference to an actual variable (if any) is lazily evaluated when the reference is "de-referenced".  This is when the interpreter looks for the requested variable matching this name.  If found, it will also run any triggers if they exist, finally pushing the variable's value onto the stack.  

#### Resolved references and anonymous variables
A name that is not yet evaluated is called an unresolved reference.  An unresolved reference may be transformed into a resolved reference by "referencing" it.  This will record the variable's location and scope for future use by pushing a reference to the variable onto the stack.  In addition to this, an anonymous variable may be created by referencing a non-reference or a resolved reference.  This will push a reference to the original value onto the stack and create a variable that isn't part of any scope.  Examples of these concepts will be shown below.

## Operations
The following table is a short reference of the operators in Dijon.  For each operation, `a` refers to the value at top of the stack, and `b` refers to the value under `a` (if applicable).  A long description along with the caveats for each operator is listed below the table.  

<table>
<tr><th>Operator</th> <th>Name</th> <th>Short description</th></tr>
<tr><td><code>-</code></td> <td>Subtract</td> <td>Pops a and b, then pushes b-a</td></tr>
<tr><td><code>/</code></td> <td>Divide</td> <td>Pops a and b, then pushes b/a</td></tr>
<tr><td><code>#</code></td> <td>Concatenate</td> <td>Pops a and b, then pushes their concatenated reference</td></tr>
<tr><td><code>$</code></td> <td>Dereference</td> <td>Pops a, then pushes the value of a</td></tr>
<tr><td><code>&</code></td> <td>Reference</td> <td>Pops a, then pushes a reference to a</td></tr>
<tr><td><code>~</code> or <code>\n</code></td> <td>Shortcut</td> <td>Accesses a, then sets b to the value of a, if applicable</td></tr>
</table>

**Subtract**:
If the stack has two or more values, pop `a` and `b`.  If either are references, then dereference each.  If either is not a number, throw an exception.  Finally, push `b-a` onto the stack.  If there were otherwise fewer than two values on the stack, push the value `0` onto the stack.

**Divide**:
If the stack has fewer than two values, push `1` onto the stack.  Otherwise, pop `a` and `b`.  If either are references, then dereference each.  Then, if either is not a number, throw an exception.  Finally, push `b/a` onto the stack.

**Concatenate**:
If the stack has fewer than two values, throw an exception.  Otherwise, pop `a` and `b`.  If `b` is not a reference, raise an exception.  If `a` is an unresolved reference, then extend `b` with `a`'s name.  If `a` is a non-negative integer, then extend `b` with the string representation of `a`.  Use this string to push a new reference to `b`'s child with that name.

**Dereference**:
If the stack is empty, throw an exception.  Otherwise, if `a` is a reference, pop it, dereference it, then push the variable's value, otherwise do nothing.

**Reference**:
If the stack is empty, throw an exception.  Otherwise, pop `a`.  If `a` is an unresolved reference, then attempt to resolve it.  If it exists, push a resolved reference to the stack.  Otherwise, create an anonymous variable and set its value to `a`, then push a resolved reference to the anonymous variable.

**Shortcut**:
This runs at the end of every line.  If there is at least one value on the stack, then dereference it.  If there are at least two values on the stack, and the second one is a reference, then get the variable corresponding to the reference, creating it if it doesn't exist, and set the value to the top stack value.  Finally, clear the stack.

## Importing and Reserved Variables
One can take advantage of importing to use variables and triggers from other files, especially the standard library included with Dijon.  A file can be imported directly from the current working directory, the directory containing `dijon.py`, the directory containing the first loaded `.dij` file, or from a subdirectory of the prior options.  Note that a valid Dijon file ends with `.dij` and must be valid variable name containing no periods.  One may specify a file in a subdirectory by delineating the relative path with periods, i.e. importing `test.hello` would refer to `test/hello.dij` in relation to one of the three aforementioned base directories.  A file can be imported at the top of the code with the line `@filename`.  This will add all exported variables and triggers to the current file.  The variables may be used subsequently with the name `filename.varname`.  Alternatively if the file was imported with two at signs like `@@filename`, it is said to be a global import, and all the exports are accessed directly with `varname` in place of `filename.varname`.  

A variable along with any associated triggers may be tagged as an export by using the `export` keyword as shown below.  In addition to this special variable, there is also `out` and `nout`.  When `out` is set, the value is printed to stdout.  More specifically, if the value is a positive integer between 0 and 127, then a character corresponding to that ASCII codepoint is printed out.  If the character is a reference, then the reference's name is printed out.  `nout` is similar but is used to print out numbers.  If the value is a number, then its decimal representation is printed to stdout.  
