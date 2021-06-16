# SC

This repository contains the [official specification](spec.md) for the Simple Config (SC) Language.

## Goals

SC is designed and intended for writing configuration files. SC favours simplicity and readability first and foremost.

SC should be easy to read and write since it is intended to be written by humans. It should also
be simple to parse, making it easy to write parsers in any programming language.

## Overview

SC supports the following value types: null, booleans, numbers, variables, strings, lists, and dictionaries.

Some key highlights:

- Line comments (`//`) and block comments (`/* */`).
- Variables allow dynamic values in config files that can be expanded when the file is parsed.
- Variables specified and expanded inside strings.
- Raw strings enclosed in backticks (``) that are treated literally and can contain newlines.
- Dictionary keys can be unquoted if they contain only letters and digits.
- Commas can be omitted after a list element or dictionary member if it is followed by a newline.

## Example

```sc
{
  /* Block comment
  over multiple lines
  */

  // Key does not need to be quoted
  container: {
    name: "service"
    // Variable that will be expanded during parsing
    label: ${value}
    memory: 256
    start: true
    // Variable within string
    image: "ubuntu:${version}-latest"
    ports: [
      8080
      8081
    ]
  }
  description: `raw string
over multiple lines
without escapes \n\t\"`
  // Quoted key due to space
  "secret value": null
}
```

## Why SC?

Learn more about [why SC exists](why.md).
