# Build Code
This library provides a structured way to build source code from strings. It is intended for use with the D programming language, but can be utilized for other languages as well. The key piece specific to D is the automatically injected line/file insertions.

When D mixes in a string for further compilation, errors compiling that string will start pointing to source code which does not exist. By utilizing this library to build out the code structure, the compiler will point to the source line which inserted the piece of code. This will hide where the mixin took place, but direct your attention to the offending source structure.

## Why use a Code Builder
When generating source code, variables and strings are concatenated together to build out new code. This can become very ugly, be it using a string formatter or plain old concatenation. Code Builder tries to allow for structuring code similar to how you'd write it in a file if you were writing the source code by hand. (Don't be fooled, it still isn't the same.)

This library actually comes out of work done for the ProtocolBuffer tool that generates D code from proto files. (At the time of writing I'm still working on modifying ProtocolBuffer to utilize this library).

https://github.com/opticron/ProtocolBuffer

## Utilizing Code Builder
Creating your source code store:

```
import codebuilder.structure;
auto indentCount = 0;
auto code = CodeBuilder(indentCount);
```

The indentCount allows building code to be broken out into functions which manage their own CodeBuilder. That is one of the benefits of utilizing this library is that it will work with you to manage indentation. The library defaults to utilizing tabs for indentation, modify codebuilder.structure.indentation with the desired indentation string.

```
code.put("void main() {\n", Indent.open);
code.push("}\n");
code.put("int a = 5;\n");
code.put("multiply(a);\n");
code.pop();
```

CodeBuilder does not attempt to manage when new lines are inserted, this means that each desired new line must be included in the string added to the code.

Indicators are provided for when it is desired to increase indentation or decrease it.

The CodeBuilder also provides a stack for pushing code that can and will be popped off later. In the future we will see that we use code.finalize() to get the final string, and this will make sure the stack has been completely popped. It is also the main reason to provide a function with its own CodeBuilder to perform a specific code structure task.

By default, code that is pushed on to the stack will close indentation. Specifying Indent.none can be done to prevent this, but the general intention for the stack is closing scope of some kind.

```
code.put("\nvoid multiply(int v) {\n", Indent.open);
code.push("}\n");
code.put("try {\n", Indent.open);
auto catchblock = CodeBuilder(1);
catchblock.put("} catch(Exception e) {\n", Indent.close | Indent.open);
catchblock.put("import std.stdio;\n");
catchblock.put("writeln(`Exception is bad but I don't care.`);\n");
catchblock.put("}\n", Indent.close);
code.push(catchblock);

code.put("return v * ");
code.rawPut(76.to!string ~ ";\n");
import std.stdio;
writeln(code.finalize());
```

Sometimes the closing code can be a lot more than a single line, for this Code Builder can push other code onto its stack. This allows the code to be written in a linear fashion instead of needing to reverse the source code order, as you would when using code.push().

Since this specific code block will close indentation as the first operation, it needs to be created with at least on indentation level, if it will be closing more than that then it's indentation must be increased as the library verifies that indentation does not drop below 0;

A single line of code can be broken into multiple insertions by using rawPut(), it will not add additional indentation before the provided code.
