# Reliable & Automated Framework for Testing (RAFT)

## FIX Testing

### Overview

RAFT provides a framework for testing FIX (Financial Information eXchange) messages. This document outlines how to create and execute basic FIX scripts using RAFT.
### Basic FIX Script Structure
A basic RAFT script consists of a series of commands that send and acknowledge FIX messages. The following sections will guide you through creating a simple RAFT script, sending a FIX message, and acknowledging it.

## Example Script (Basic)
```plaintext
FIX_SEND $alias 35=1|112=$message| -d |
FIX_ACK $alias -p 112=<string:$message>

**ANDREY PLEASE EXPLAIN HOW TO GET TO THIS TO WORK**
**ANDREY PLEASE EXPLAIN HOW TO SET THE HOST AND PORT**
**ANDREY PLEASE EXPLAIN HOW TO SET THE DATA DICTIONARY**
```

### Creating a Basic FIX Script
2. **Set Up the FIX Message**: Use the `FIX_SEND` command to construct and send a FIX message. The message should include the necessary tags and values.
3. **Acknowledge the Message**: Use the `FIX_ACK` command to acknowledge the receipt of the FIX message and its individual fields.

### How To Run The Test Using Intellij IDE
1. Create a new 'Application' run configuration
2. Set the main class to `com.bgcgroup.fx.raft.RaftMain`
3. CLI Arguments:
   * -l
   * -d scripts/{folder}}/{script_name}.txt

## Example Script (Advanced)
```plaintext
@SECURITY_TEST_CHECK
$alias
$message=helloworld
FIX_SEND $alias 35=1|112=$message| -d |
FIX_ACK $alias -p 112=<string:$message>
```

### Creating An Advanced FIX Script
1. **Define the Alias**: **ANDREY PLEASE EXPLAIN**
2. **Set Up the FIX Message**: Use the `FIX_SEND` command to construct and send a FIX message. The message should include the necessary tags and values.
3. **Acknowledge the Message**: Use the `FIX_ACK` command to acknowledge the receipt of the FIX message and its individual fields.
4. **Execute the Script**: Finally, run the script to send the FIX message and receive the acknowledgment.

### How To Run The Script
1. Along with running the script through Intellij, you can also run it through a web browser
2. How To Test Through Chrome
 * **ANDREY PLEASE FILL IN**

## Deep Dive Into the Script
```plaintext
@SECURITY_TEST_CHECK
```
**ANDREY -- PLEASE EXPLAIN**

```plaintext
$alias
```
**ANDREY -- PLEASE EXPLAIN**

```plaintext
$message=helloworld
```
This line defines a variable `$message` with the value `helloworld`. This variable can be used in the FIX message to dynamically insert values. It is case-sensitive. 
**ANDREY PLEASE EXPLAIN IF THERE ARE LIMITATIONS**

```plaintext
FIX_SEND $alias 35=1|112=$message| -d |
```
**ANDREY PLEASE EXPLAIN WHAT THE $alias IS DOING**
The actual FIX message is constructed here. The `35=1` indicates the message type (in this case, a logon), and `112=$message` sets the value of tag 112 to the value of the `$message` variable. 
The `-d` flag that allows you to define the delimiter for the FIX message. **ANDREY PLEASE WRITE IF THERE IS A DEFAULT OR ANY CONSTRAINTS**

```plaintext
FIX_ACK $alias -p 112=<string:$message>
```
This line acknowledges the receipt of the FIX message sent earlier. The `-p` flag specifies the tag to acknowledge.
The `<string:$message>` syntax indicates that the acknowledgment should match the value of the `$message` variable. 
The **string** is used to specify that the value should be treated as a string. If the value is not a string, it will cause a compile error.
**ANDREY PLEASE EXPLAIN THE USE OF $alias HERE**

## Available Commands (**ANDREY PLEASE ADD ALL COMMANDS**)
| Command | Description |
|---------|-------------|
| FIX_SEND | Sends a FIX message to the specified session alias. The message must include the necessary tags and values. |
| FIX_ACK | Acknowledges the receipt of a FIX message for the specified session alias. This command confirms that the message was processed correctly. |

## FIX_SEND Parameters (**ANDREY PLEASE ADD ALL PARAMETERS**)
| Parameter | Description                                                                                    |
|-----------|------------------------------------------------------------------------------------------------|
| -d        | Specifies the delimiter for the FIX message. If not provided, a default delimiter may be used. |
| -f        | **ANDREY PLEASE EXPLAIN WHAT THIS DOES**                                                       |

## FIX_ACK Parameters (**ANDREY PLEASE ADD ALL PARAMETERS**)
| Parameter | Description                                                                                          |
|-----------|------------------------------------------------------------------------------------------------------|
| -p        | Specifies the tag to acknowledge. The value should match the expected response from the FIX message. |

## Standard Library
### Overview
Once users start to create multiple scripts, they will find that there are common patterns and functions that can be reused across different scripts.
To facilitate this, RAFT provides a standard library of functions and commands that can be used in your scripts. This section outlines the available functions and how to use them effectively.

### How To Define The Standard Library
Create a file called: `stdlib.txt`
***ANDREY PLEASE EXPLAIN IF IT HAS TO BE CALLED THIS OR CAN BE CALLED ANYTHING YOU WANT***

### How To Include The Standard Library
***ANDREY PLEASE EXPLAIN***

## Matching Basics

Given a message `A=a|B=b|C=c|D=d|E=e`

We can perform a variety of pattern matches, the most simple being:
```plaintext
-p C=c
```
This scans the given message for the presence of the tag `C` with the exact value `c`; it is found, so the match is successful. (Note that, for clarity, the fields are separated by `|` here; a custom delimiter can be specified via the `-d` flag.)
Similarly, 
```plaintext
C=x
```
will return failure.
We can also negate patterns with `!`:
```plaintext
-p !C=x
```
is true, while 
```plaintext
!C=c
``` 
is false.

If a value we want to compare contains whitespaces, we have to enclose the ENTIRE pattern in quotes `""`, e.g.
```plaintext
-p "!C=hello world" 
```

We can also match multiple fields in a specific order:
```plaintext
-p C=c|D=d
```
is true; but
```plaintext
-p C=c|D=x and -p C=c|E=e|D=d
```
are both false.

Patterns can also be enclosed in square brackets:
```plaintext
-p [C=c|D=d] or -p ["C=c|D=d"] or -p [["C=c|D=d"]]
```
are valid. Note: a maximum of `[[]]` (two levels of brackets) is supported.

Patterns can be used to form Boolean expressions in sum-of-products (a AND b) OR (c AND d) form.
Patterns in a single level of brackets are considered part of an AND:
```plaintext
-p [C=c E=e]
```
is parsed as `C=c` AND `E=e` -- since both are present in `A=a|B=b|C=c|D=d|E=e`, the comparison is considered successful.

Patterns can be more complicated:
```plaintext
-p [["C=c|D=d"] ["!B=b"]]
```
is **false**: since B=b is present in the message, `C=c|D=d` AND `!B=b` fails.

ORs of patterns can be created via multiple `-p` flags:
```plaintext
-p ["C=c|D=d"] -p ["!B=b"]
```
is successful: the expression above is parsed as `C=c|D=d` OR `!B=b`; while `!B=b` fails, `C=c|D=d` succeeds, so the pattern match passes.

## Skipped Fields

In the example above, all fields are considered directly. Now, consider the message
```plaintext
8=FIX.4.4|A=a|B=b|C=c|D=d|E=e
```
While the BeginString `(8=)` is vital for the correct operation of FIX, it -- alongside other headers, except MsgType -- contains session
information rather than payload data. As such, by default, headers are **NOT** compared even if they are present. To change this, pass
```plaintext
-P IGNORE_NONE
```
to the commandline. To automatically ignore commonly ignored non-header fields (such as timestamps), set
```plaintext
-P IGNORE_ALL
```
Notably, fields can always be ignored manually via the `-i` flag (regardless of policy):
```plaintext
-i B
```
will automatically ignore the B field and
```plaintext
-P IGNORE_NONE -i 8
```
will automatically ignore the BeginString comparison, even though the comparison policy is set (by default) to include headers.

## Wildcards and Type Checks

Patterns support wildcard matching and type checks. In the examples above, all the values have to match exactly; this is not required.
Angle bracket `<...>` syntax allows for more tailored comparison.

Consider the pattern `A=a|B=b|C=c` -- this pattern requires the values a, b, and c to match exactly and in order. To simply check that a field is present, we can use the `<?>` operator:
```plaintext
A=a|B=<?>|C=c
```

Which matches on the tag `B` being present with ANY value. This is different from `A=a|C=c`, since `B` still has to be present in sequence between `A=a` and `C=c`. 
On the other hand, `-p [A=a C=c]` only checks that both `A=a` AND `C=c` are both present in the message, not that they are in sequence.

Angle brackets allow for type comparison with `<type>` syntax. Currently supported types are:
1. Character (char)
2. String (str)
3. Boolean (bool)
4. Integer (int)
5. Float (flt)
6. Timestamp (ts) (case-insensitive).

`Integer` and `Float` types default to 64-bit (Double and Long respectively).

The following example requires tag B to be present and readable as an integer.
```plaintext
A=a|B=<int>|C=c
```

Setting `B=<bool>`, `B=<string>`, or `B=<float>` works similarly.

Notably, `Boolean` values can be parsed from `y/n`, `yes/no`, `t/f`, `true/false`, `1/0` (case-insensitive).

Constraints can be passed in `<type:constraint>` format.

Boolean constraints (such as `<Boolean:true>`) require the actual value to match the provided truth value:
```plaintext
A=a|B=<Boolean:true>|C=c
```

Requires field `B` to contain a value readable as `true`.

`Integer` and `float` constraints take a pattern separated by `&&` and `||` in sum-of-products format (much like patterns themselves); literals separated by `&&` are treated as `AND`, and parsed as a single expression, `OR'ed` with the next sequence of `AND's`. Valid operators include 

**ANDREY WHAT IS GOING ON HERE**
�<�, �>�, �<=�, �>=�, and �==�; one side of the operation MUST contain a �?� and the other, a literal value.
**ANDREY WHAT IS GOING ON HERE**

For example, the following requires an integer value greater than 5:
```plaintext
B=<int:?>5>
```

The following requires an integer between 5 and 7:
```plaintext
B=<int:?>5&&?<7>
```

The following requires an integer between 5 and 7 exclusive, or 10 and 20 `inclusive`:
```plaintext
B=<int:?>5&&?<7||?=>10&&?<=20>
```

The only exception to `?` being required is the `==` operator: literal values can be passed in `?==value`, `value==?`, or value format. That is:
Requires an integer value of 5, 6, or 7. Floats operate in the same way; the passed value, however, must be interpretable as a float:
```plaintext
B=<int:?==5||6==?||7>
```

Timestamps operate in a similar manner, but with a few additional considerations. Timestamps can be passed in as an exact value (as interpretable by quickfixj) or as one of the following functions:

1. `NOW()` (the current system time)
2. `PLUS()` (time + delta)
3. `MINUS()` (time + delta)

The first argument to `PLUS()` and `MINUS()` must be interpretable as a timestamp **ANDREY WHAT IS THIS** � either an exact value or `NOW()`. Following the first input, time units (YEARS, MONTHS, WEEKS, DAYS, HOURS, MINUTES, SECONDS, MILLIS, MICROS, NANOS) can be passed in to add or subtract from the initial time. `e.g. MINUS(NOW(),SECONDS(15))` is the current system time minus 15 seconds and `PLUS(NOW(),HOURS(5),SECONDS(15)` is the current system time plus 5 hours 15 seconds.
NOTE: RAFT settings default to UTC unless otherwise specified (which can cause unexpected timestamp mismatches).

Characters support the `||` operator `(e.g. <char:a||b>)`, which ensures that the target matches the character a or the character b.

String comparison works by standard Java regex parsing (see https://docs.oracle.com/javase/6/docs/api/java/util/regex/Pattern.html).

`B=<string:regex>` will use standard pattern matching (where regex is a valid regex).

One important note, however, is that Java uses `[]` and **ANDREY WHAT IS THIS** � as special characters; unfortunately, the parser does as well. In order to pass `[, ]`, or **ANDREY WHAT IS THIS** �, their HTML-like value must be passed instead:
```plaintext
&lbrack;  
&rbrack;
&quot;
```

For `[, ]`, and **ANDREY WHAT IS THIS** � respectively.

The example below matches any alphanumeric string with exactly 5-7 characters:
```plaintext
B=<string:^&lbrack;a-zA-Z0-9&rbrack;{5,7}$>
```
and
```plaintext
<&lbrack;\&quot;&rbrack;>
```

Corresponds to the pattern **ANDREY WHAT IS THIS**`[\�]`.

Additionally, certain shorthand patterns are permitted: `<==value1,value2,value3==>` and `<type:==value1,value2,value3==>`

These correspond to a direct lookup of enumerable values (bypassing and performing a type check respectively). Allowed values must be comma-separated.

Patterns can also be inverted with a `!` prefix: `A=!<constraint>` returns false if A matches the constraint provided and true otherwise.