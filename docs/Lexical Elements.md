BSV has the same basic lexical elements as Verilog.

### Whitespace and comments

Spaces, tabs, newlines, formfeeds, and carriage returns all constitute whitespace. They may be used
freely between all lexical tokens.

A _comment_ is treated as whitespace (it can only occur between, and never within, any lexical token).
A one-line comment starts with `//` and ends with a newline. A block comment begins with `/*` and
ends with `*/` and may span any number of lines.

Comments do not nest. In a one-line comment, the character sequences `//`, `/*` and `*/` have no special
significance. In a block comment, the character sequences//and/*have no special significance.

### Identifiers and keywords

An identifier in BSV consists of any sequence of letters, digits, dollar signs$and underscore
characters (_). Identifiers are case-sensitive: `glurph` , `gluRph` and `Glurph` are three distinct identifiers.
The first character cannot be a digit.

BSV currently requires a certain capitalization convention for the first letter in an identifier. 
Identifiers used for package names, type names, enumeration labels, union members and type classes must
begin with a capital letter. In the syntax, we use the non-terminal _Identifier_ to refer to these. Other
identifiers (including names of variables, modules, interfaces, etc.) must begin with a lowercase letter
and, in the syntax, we use the non-terminal _identifier_ to refer to these.

As in Verilog, identifiers whose first character is `$` are reserved for so-called _system_ tasks and functions
(see Section [12.8](#Section_12.8)).

If the first character of an instance name is an underscore, (_), the compiler will not generate
this instance in the Verilog hierarchy name. This can be useful for removing submodules from the
hierarchical naming.

There are a number of _keywords_ that are essentially reserved identifiers, i.e., they cannot be used by
the programmer as identifiers. Keywords generally do not use uppercase letters (the only exception
is the keyword `valueOf`). BSV includes all keywords in SystemVerilog. All keywords are listed in
Appendix [A](#Appendix_A).

The types `Action` and `ActionValue` are special, and cannot be redefined.

### Integer literals

Integer literals are written with the usual Verilog and C notations:

```ebnf
intLiteral ::= ’0|’
| sizedIntLiteral
| unsizedIntLiteral
sizedIntLiteral ::= bitWidth baseLiteral
unsizedIntLiteral ::= [sign]baseLiteral
| [sign]decNum
baseLiteral ::= (’d|’D)decDigitsUnderscore
| (’h|’H)hexDigitsUnderscore
| (’o|’O)octDigitsUnderscore
| (’b|’B)binDigitsUnderscore
decNum ::=decDigits[decDigitsUnderscore]
bitWidth ::= decDigits
sign ::=+|-
decDigits ::={ 0 ... 9 }
decDigitsUnderscore ::={ 0 ... 9 ,_}
hexDigitsUnderscore ::={ 0 ... 9 ,a...f,A...F,_}
octDigitsUnderscore ::={ 0 ... 7 ,_}
binDigitsUnderscore ::={ 0 , 1 ,_}
```

An integer literal is a sized integer literal if a specific _bitWidth_ is given (e.g., `8’o255`). There is no
leading _sign_ (`+` or `-`) in the syntax for sized integer literals; instead we provide unary prefix `+` or `-`
operators that can be used in front of any integer expression, including literals (see Section 9).
An optional sign (`+` or `-`) is part of the syntax for unsized literals so that it is possible to construct
negative constants whose negation is not in the range of the type being constructed (e.g. `Int#(4) x = -8 ;`
since 8 is not a valid `Int#(4)`, but `-8` is).

Examples:

```bsv
’h48454a
32’h48454a
8’o
12’b
32’h_FF_FF_FF_FF
```
#### 2.3.1 Type conversion of integer literals

Integer literals can be used to specify values for various integer types and even for user-defined
types. BSV uses its systematic overloading resolution mechanism to perform these type conversions.
Overloading resolution is described in more detail in Section 14.1.

An integer literal is a sized literal if a specific _bitWidth_ is given (e.g., `8’o255`), in which case
the literal is assumed to have type `bit [w− 1 :0]`. The compiler implicitly applies the function
`fromSizedInteger` to the literal to convert it to the type required by the context. Thus, sized
literals can be used for any type on which the overloaded function `fromSizedInteger` is defined,
i.e., for the types `Bit`, `UInt` and `Int`. The function `fromSizedInteger` is part of the
`SizedLiteral` typeclass, defined in Section B.1.5.

If the literal is an unsized integer literal (a specific _bitWidth_ is _not_ given), the literal is assumed
to have type `Integer`. The compiler implicitly applies the overloaded function `fromInteger` to the
literal to convert it to the type required by the context. Thus, unsized literals can be used for any
type on which the overloaded function `fromInteger` is defined. The function `fromInteger` is part
of the `Literal` typeclass, defined in Section B.1.3.

The literal `’0` just stands for 0. The literal `’1` stands for a value in which all bits are 1 (the width
depends on the context).



### Real literals

Real number literals are written with the usual Verilog notation:

```ebnf
realLiteral ::=decNum[.decDigitsUnderscore]exp[sign]decDigitsUnderscore
| decNum.decDigitsUnderscore
sign ::=+|-
exp ::=e|E
decNum ::=decDigits[decDigitsUnderscore]
decDigits ::={ 0 ... 9 }
decDigitsUnderscore ::={ 0 ... 9 ,_}
```
There is no leading sign (`+` or `-`) in the syntax for real literals. Instead, we provide the unary prefix
+and-operators that can be used in front of any expression, including real literals (Section 9).

If the real literal contains a decimal point, there must be digits following the decimal point. An
exponent can start with either anEor ane, followed by an optional sign (`+` or `-`), followed by digits.
There cannot be an exponent or a sign without any digits. Any of the numeric components may
include an underscore, but an underscore cannot be the first digit of the real literal.

Unlike integer literals, real literals are of limited precision. They are represented as IEEE floating
point numbers of 64 bit length, as defined by the IEEE standard.

Examples:

```bsv
1.
0.
2.4E10 // exponent can be e or E
5e-
325.761_452_e-10 // underscores are ignored
9.2e+
```
#### 2.4.1 Type conversion of real literals

Real literals can be used to specify values for real types. By default, real literals are assumed to
have the typeReal. BSV uses its systematic overloading resolution mechanism to perform these type
conversions. Overloading resolution is described in more detail in Section 14.1. There are additional
functions defined for `Real` types, provided in the `Real` package (Section C.5.1).

The function `fromReal` (Section B.1.4) converts a value of type `Real` into a value of another datatype.
Whenever you write a real literal in BSV (such as3.14), there is an implied `fromReal` applied to it,
which turns the real into the specified type. By defining an instance of `RealLiteral` for a datatype,
you can create values of that type from real literals.

The type `FixedPoint`, defined in the `FixedPoint` package, defines a type for representing fixed
point numbers. The `FixedPoint` type has an instance of `RealLiteral` defined for it and contains
functions for operating on fixed-point real numbers.

### String literals

String literals are written enclosed in double quotes `"···"` and must be contained on a single source
line.

```ebnf
stringLiteral ::= "···string characters···"
```
Special characters may be inserted in string literals with the following backslash escape sequences:


```
\n newline
\t tab
\\ backslash
\" double quote
\v vertical tab
\f form feed
\a bell
\OOO exactly 3 octal digits (8-bit character code)
\xHH exactly 2 hexadecimal digits (8-bit character code)
```


Example - printing characters using form feed.

```bsv
module mkPrinter (Empty);
    String display_value;
    display_value = "a\nb\nc";  // prints a
                                //        b
                                //        c  repeatedly
    rule every;
        $display(display_value);
    endrule
endmodule
```
#### 2.5.1 Type conversion of string literals

String literals are used to specify values for string types. BSV uses its systematic overloading
resolution mechanism to perform these type conversions. Overloading resolution is described in
more detail in Section 14.1.

Whenever you write a string literal in BSV there is an implicit `fromString` applied to it, which
defaults to typeString.

### Don’t-care values

A lone question mark `?` is treated as a special don’t-care value. For example, one may return `?`
from an arm of a case statement that is known to be unreachable.

Example - Using `?` as a don’t-care value

```bsv
module mkExample (Empty);
    Reg#(Bit#(8)) r <- mkReg(?); // don’t-care is used for the reset value of the Reg
    rule every;
        $display("value is %h", r); // the value of r is displayed
    endrule
endmodule
```
### Compiler directives

The following compiler directives permit file inclusion, macro definition and substitution, and
conditional compilation. They follow the specifications given in the Verilog 2001 LRM plus the extensions
given in the SystemVerilog 3.1a LRM.

In general, these compiler directives can appear anywhere in the source text. In particular, they do
not need to be on lines by themselves, and they need not begin in the first column. Of course, they
should not be inside strings or comments, where the text remains uninterpreted.



#### 2.7.1 File inclusion: `` `include`` and `` `line``

```ebnf
compilerDirective ::= ‘include "filename"
| ‘include <filename>
| ‘includemacroInvocation
```
In an‘includedirective, the contents of the named file are inserted in place of this line. The
included files may themselves contain compiler directives. Currently there is no difference between
the"..."and<...>forms. AmacroInvocationshould expand to one of the other two forms. The
file name may be absolute, or relative to the current directory.

```ebnf
compilerDirective ::= ‘linelineNumber"filename"level
lineNumber ::= decLiteral
level ::= 0 | 1 | 2
```
A ``‘line`` directive is terminated by a newline, i.e., it cannot have any other source text after the _level_.
The compiler automatically keeps track of the source file name and line number for every line of
source text (including from included source files), so that error messages can be properly correlated to
the source. This directive effectively overrides the compiler’s internal tracking mechanism, forcing
it to regard the next line onwards as coming from the given source file and line number. It is
generally not necessary to use this directive explicitly; it is mainly intended to be generated by other
preprocessors that may themselves need to alter the source files before passing them through the
BSV compiler; this mechanism allows proper references to the original source.

The _level_ specifier is either 0, 1 or 2:

- 1 indicates that an include file has just been entered
- 2 indicates that an include file has just been exited
- 0 is used in all other cases

#### 2.7.2 Macro definition and substitution: ‘define and related directives

```ebnf
compilerDirective ::= ‘definemacroName[ (macroFormals) ]macroText
macroName ::= identifier
macroFormals ::= identifier{ ,identifier }
```
The ``‘define`` directive is terminated by a bare newline. A backslash (`\`) just before a newline
continues the directive into the next line. When the macro text is substituted, each such continuation
backslash-newline is replaced by a newline.

The _macroName_ is an identifier and may be followed by formal arguments, which are a list of
comma-separated identifiers in parentheses. For both the macro name and the formals, lower and
upper case are acceptable (but case is distinguished). The _macroName_ cannot be any of the compiler
directives (such as `include`, `define`, ...).

The scope of the formal arguments extends to the end of themacroText.

ThemacroTextrepresents almost arbitrary text that is to be substituted in place of invocations of
this macro. ThemacroTextcan be empty.

One-line comments (i.e., beginning with `//`) may appear in themacroText; these are not considered
part of the substitutable text and are removed during substitution. A one-line comment that is not
on the last line of a `‘define` directive is terminated by a backslash-newline instead of a newline.

A block comment (`/*...*/`) is removed during substitution and replaced by a single space.

The _macroText_ can also contain the following special escape sequences:



- `‘"` Indicates that a double-quote (`"`) should be placed in the expanded text.
- `‘\‘"` Indicates that a backslash and a double-quote (`\"`) should be placed in the expanded
    text.
- `‘‘` Indicates that there should be no whitespace between the preceding and following
    text. This allows construction of identifiers from the macro arguments.

A minimal amount of lexical analysis of _macroText_ is done to identify comments, string literals,
identifiers representing macro formals, and macro invocations. As described earlier, one-line 
comments are removed. The text inside string literals is not interpreted except for the usual string
escape sequences described in Section 2.5.

There are two define macros in the define environment initially; `‘bluespec` and `‘BLUESPEC`.

Once defined, a macro can be invoked anywhere in the source text (including within other macro
definitions) using the following syntax.

```ebnf
compilerDirective ::= macroInvocation
macroInvocation ::= ‘macroName[ (macroActuals) ]
macroActuals ::= substText{ ,substText }
```
The _macroName_ must refer to a macro definition available at expansion time. The _macroActuals_,
if present, consist of substitution text _substText_ that is arbitrary text, possibly spread over multiple
lines, excluding commas. A minimal amount of parsing of this substitution text is done, so that
commas that are not at the top level are not interpreted as the commas separating _macroActuals_.
Examples of such “inner” uninterpreted commas are those within strings and within comments.

```ebnf
compilerDirective ::= ‘undefmacroName
| ‘resetall
```

The `‘undef` directive’s effect is that the specified macro (with or without formal arguments) is no
longer defined for the subsequent source text. Of course, it can be defined again with `‘define` in the
subsequent text. The `‘resetall` directive has the effect of undefining all currently defined macros,
i.e., there are no macros defined in the subsequent source text.

#### 2.7.3 Conditional compilation: ‘ifdef and related directives

```ebnf
compilerDirective ::= ‘ifdefmacroName
| ‘ifndefmacroName
| ‘elsifmacroName
| ‘else
| ‘endif
```
These directives are used together in either an `‘ifdef`-`endif` sequence or an `ifndef`-`endif` sequence.
In either case, the sequence can contain zero or more `elsif` directives followed by zero or one `else`
directives. These sequences can be nested, i.e., each `‘ifdef` or `ifndef` introduces a new, nested
sequence until a corresponding `endif`.

In an `‘ifdef` sequence, if the _macroName_ is currently defined, the subsequent text is processed until
the next corresponding `elsif`, `else` or `endif`. All text from that next corresponding `elsif` or `else`
is ignored until the `endif`.

If the _macroName_ is currently not defined, the subsequent text is ignored until the next corresponding
`‘elsif`, `‘else` or `‘endif`. If the next corresponding directive is an‘elsif, it is treated just as if it
were an `‘ifdef` at that point.

If the `‘ifdef` and all its corresponding `‘elsif`s fail (macros were not defined), and there is an `‘else`
present, then the text between the ``‘else`` and ``‘endif`` is processed.



An `‘ifndef` sequence is just like an `‘ifdef` sequence, except that the sense of the first test is
inverted, i.e., its following text is processes if the _macroName_ is _not_ defined, and its `‘elsif` and
`‘else` arms are considered only if the macro _is_ defined.

Example using `‘ifdef` to determine the size of a register:

```bsv
‘ifdef USE_16_BITS
    Reg#(Bit#(16)) a_reg <- mkReg(0);
‘else
    Reg#(Bit#(8)) a_reg <- mkReg(0);
‘endif
```