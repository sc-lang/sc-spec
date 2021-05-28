# The SC Language Specification

### Table of Contents

- [Introduction](#introduction)
- [Notation](#notation)
- [File Extension](#file-extension)
- [Souce Text](#source-text)
- [Comments](#comments)
- [Whitespace](#whitespace)
- [Commas](#commas)
- [SC Document](#sc-document)
- [Values](#values)
- [Null](#null)
- [Booleans](#booleans)
- [Numbers](#numbers)
- [Variables](#variables)
- [Strings](#strings)
- [Lists](#lists)
- [Dictionaries](#dictionaries)

## Introduction

The Simple Config (SC) Language is a configuration language designed for writing configuration files.

SC should be easy to read and write since it is intended to be written by humans. It should also
be simple to parse, making it easy to write parsers in any programming language.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT",
"RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in
[RFC 2119](https://tools.ietf.org/html/rfc2119).

## Notation

The grammar is specified using Extended Backus-Naur Form (EBNF). It is similar to the version
of EBNF used by the W3C for the [XML grammar](https://www.w3.org/TR/REC-xml/#sec-notation).

A production has the form:

```
production_name ::= expression
```

Terminals are enclosed in double quotes "" or backticks `` (if the terminal contains a double quote).
Non-terminals are alphanumeric identifiers.

The following operators are used in order of precedence:

- `*` matches zero or more times
- `+` matches one or more times
- `?` matches zero or one times (optional)
- `()` grouping
- `|` alteration (or)
- `/* */` comment

An ellipsis (...) indicates a range of values not explicitly specified for brevity.
For example, `"0" | ... | "3"` is equivalent to `"0" | "1" | "2" | "3"`.

## File Extension

SC files SHOULD have a `.sc` file extension.

## Source Text

SC source code MUST be UTF-8 encoded Unicode text. This document will use the term _character_ to
refer to a Unicode code point in the source text.

The following productions are used to denote Unicode character classes:

```
newline        ::= /* the Unicode code point U+000A */
unicode_char   ::= /* any Unicode code point except newline */
unicode_letter ::= /* any Unicode code point classified as "Letter" */
unicode_digit  ::= /* any Unicode code point classified as "Number, decimal digit" */
```

The supported Unicode categories for "Letter" are "Letter, uppercase (Lu)", "Letter, lowercase (Ll)",
"Letter, titlecase (Lt)", "Letter, modifier (Lm)", "Letter, other (Lo)" and the supported category
for "Number" is "Number, decimal digit (Nd)".

The underscore character `_` (U+005F) is considered a letter.

The following productions are used to classify letters and digits:

```
letter        ::= unicode_letter | "_"
decimal_digit ::= "0" | ... | "9"
hex_digit     ::= "0" | ... | "9" | "A" | ... | "F" | "a" | ... | "f"
```

## Comments

Comments can be used to document SC files. There are two types of comments supported:

1. _Line comments_ which start with "//" and stop at the end of the line.
2. _Block comments_ which start with "/\*" and stop with the first subsequent "\*/".

A comment cannot start inside a string or a comment. A block comment containing no newlines
functions as a space. A line comment or a block comment containing newlines functions as a newline.

```sc
// This is a line comment

/* this is
a block comment
over multiple lines
*/

/* this is a block comment on a single line */

"// this is not a comment"
```

## Whitespace

Whitespace is formed from spaces (U+0020), horizontal tabs (U+0009), carriage returns (U+000D),
and newlines (U+000A). Whitespace is ignored, except for newlines that may trigger the insertion of
a [comma](#commas).

## Commas

The formal grammar uses commas "," to separate [dictionary](#dictionaries) members and [list](#lists) elements.
SC files MAY omit commas using the following rule:

When the input is tokenized during lexical analysis, a comma is automatically inserted into the
token stream immediately after a newline if the last token was a [value](#values).

Example:

```sc
{
  automatic: 1 // no comma, automatically inserted
  explicit: 2, // explicit comma
  multiline:   // not a value, no comma inserted
    3          // value, comma inserted
  list: [      // not a value, no comma inserted
  ]            // value, comma inserted
}
```

Commas are required if the elements are on the same line.

```sc
[1, 2, 3] // Same line, commas required
```

## SC Document

The top level [value](#values) of an SC document MUST be a [dictionary](#dictionarys).

```
sc ::= dictionary
```

## Values

An SC value is any of the following: [null](#null), [boolean](#booleans), [number](#numbers),
[variable](#variables), [string](#strings), [list](#list), or [dictionary](#dictionaries).

```
value ::= null | boolean | number | variable | string | list | dictionary
```

## Null

The token "null" is used to represent the absence of a value.

```
null ::= "null"
```

Example:

```sc
{
  noValue: null
}
```

## Booleans

The tokens "true" and "false" are used to represent boolean values. Implementations MUST NOT
use any other tokens to represent booleans, such as "True", "False", "yes", "no".

```
boolean ::= "true" | "false"
```

Example:

```sc
{
  isTrue: true
  isFalse: false
}
```

## Numbers

A number is a numeric value represented in base 10. Numbers can be integers or floating point values.
It is up to an implementation to decide how to how to represent a number. If a number cannot be
represented in an implementation, an error MUST be communicated somehow.

A number consists of an optional minus sign, an integer component followed by an optional
faction component and/or exponent component.

```
number   ::= "-"? decimal_digit+ ("." decimal_digit+)? exponent?
exponent ::= ("e" | "E") ("+" | "-")? decimal_digit+
```

Example:

```sc
{
  integer: 123
  negativeInteger: -456
  withFraction: 123.456
  withExponent: 123e456
  withFractionAndExponent: 123.456e-789
}
```

## Variables

A variable is a placeholder for a value that will be provided when the SC file is parsed.
There are cases where values in a configuration file must be dynamic. Variables allow
values to be set with a concrete value by an implementation during parsing.

```
variable   ::= "${" identifier "}"
identifier ::= letter (letter | unicode_digit)*
```

Example:

```sc
{
  var: ${abc}
  alsoAllowed: ${_THIS_IS_4110w3d}
}
```

## Strings

A string is a sequence of characters. There are two types of strings: raw strings and interpolated strings.

A raw string is a sequence of characters between backticks, such as \`foo\`. Within the backticks,
any character may appear except a backtick. The value of a raw string is the string composed of
the characters between the backticks. No characters inside the raw string have any special meaning,
and the string may contain newlines.

A interpolated string is a sequence of characters between double quotes, such as "foo". Within the
quotes, any character may appear except a newline or an unescaped double quote. A backslash is
interpreted as an escape sequence. Variables may also occur inside the string. During parsing,
an implementation can replace the variable with its value inside the string.

The following escape sequences are supported:

<table>
  <tr><th>Escape Sequence</th><th>Description</th><th>Code point(s)</th></tr>
  <tr><td>\b</td><td>backspace</td><td>U+0008</td></tr>
  <tr><td>\f</td><td>form feed</td><td>U+000C</td></tr>
  <tr><td>\n</td><td>newline</td><td>U+000A</td></tr>
  <tr><td>\r</td><td>carriage return</td><td>U+000D</td></tr>
  <tr><td>\t</td><td>horizontal tab</td><td>U+0009</td></tr>
  <tr><td>\\</td><td>backslash</td><td>U+005C</td></tr>
  <tr><td>\"</td><td>quote</td><td>U+0022</td></tr>
  <tr><td>\${</td><td>literal ${</td><td>U+0024 U+007B</td></tr>
  <tr><td>\uXXXX</td><td>unicode</td><td>U+XXXX</td></tr>
</table>

```
string              ::= raw_string | interpolated_string
raw_string          ::= "`" (unicode_char | newline)* "`"
interpolated_string ::= `"` (unicode_value | variable)* `"`

unicode_value       ::= unicode_char | little_u_value | escaped_char
little_u_value      ::= "\u" hex_digit hex_digit hex_digit hex_digit
escaped_char        ::= "\" ("b" | "f" | "n" | "r" | "t" | "\" | `"` | "${")
```

Example:

```sc
{
    raw: `foo`
    multiline: `\n
\t`                                 // same as "\\n\n\\t"
    unicode: "\u00E0"               // "à"
    withEscapes: "\"\n\t"
    var: "Hello ${name}"
    escapedVar: "literal \${hello}" // not a variable
}
```

## Lists

A list is an ordered sequence of values. Lists are specified using square brackets with values inside.
Values are separated by commas. Trailing commas are allowed.

```
list     ::= "[" elements? "]"
elements ::= value ("," value)* ","?
```

Example:

```sc
{
  nums: [1, 2, 3]
  nested: [
    [1, 2]                  // automatic comma
    [4, 5,],                // trailing commas allowed
  ]
  mixed: [1, null, "hello"] // any type of value is allowed
}
```

## Dictionaries

A dictionary is a collection of key/value pairs. Each key/value pair is referred to as a member.
Dictionaries are specified using curly brackets with members inside. Members are separated by commas.
Trailing commas are allowed.

Members are not guaranteed to be in any specific order. Implementations MAY choose to make
the order of members visible to callers.

Member keys MUST always be strings. Keys can be quoted with backticks `` or double quotes ""
just like strings. Keys quoted with backticks function like raw strings. Keys quoted with
double quotes function like interpolated strings, with the exception that variables are not allowed
within the key. If a variable is encountered in a key, an error MUST be reported.
If a key contains only letters or numbers, quotes MAY be omitted.

```
dictionary ::= "{" members? "}"
members    ::= member ("," member)* ","?
member     ::= key ":" value

key        ::= identifier | raw_string | key_string
key_string ::= `"` unicode_value* `"`
```

Example:

```sc
{
  empty: {}                        // key does not need quotes
  inline: { first: 1, second: 2, } // trailing comma allowed
  nested: {
      v1: {
          foo: "bar"
    }
    v2: {
        foo: "baz"
    }
  }
  `raw key
with newline`: true                // raw key, similar to raw string
  "needs quoting": "yes"           // must be quoted since there is a space
  "\${foo}": "error"               // key is literal ${foo}
  "${foo}": "error"                // this will cause an error
}
```
