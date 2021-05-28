# Why SC?

A reasonable question to ask might be: why SC and not JSON, YAML, etc?

There are various other file formats available, however they all have issues that make them not
ideal for writing configuration files.

### JSON

JSON is a data serialization language first and foremost. It is intended for sharing structured
data between programs. JSON is great and accomplishes this very well. However, it is not ideal
for writing configuration files due to it being a little too simple and lacking some key features.
The most notible feature missing is comments which are crucial for documenting configuration files.
JSON is also overly verbose, requiring explicit quotes everywhere and commas.

All this is ok though. JSON is not intended for writing configuration files and it shouldn't be.
Let JSON continue doing what it does well, and have another format for configuration files.

Further reading:

- JSON spec: https://datatracker.ietf.org/doc/html/rfc8259

### YAML

YAML is quite common for writing configuration files. However, YAML has a lot of issues and
there is no shortage of posts describing all these issues. Without going into to much detail,
YAML is way too complicated. It has unclear semantics that are difficult to reason about, and no
shortage of gotchas that have caused lots of issues.

Further reading:

- YAML spec: https://yaml.org/spec/1.2/spec.html
- List of YAML issues with further links: https://noyaml.com/
- Issues with using YAML for configuration files: https://www.arp242.net/yaml-config.html

### TOML

TOML seems great on the surface and for simple configuration files it is ok. However, once
files become more complicated and you introduce nested structures it becomes hard to read and
understand. The syntax for things like tables and arrays of tables is confusing. While TOML
is a lot simpler than YAML, it is still far too complicated and unintuitive.

Further reading:

- TOML Spec: https://toml.io/en/v1.0.0

### JSON5

JSON5 is a proposed extension to JSON that adds some useful features such as comments,
unquoted object keys, multiline strings, etc. JSON5 seems pretty good actually and solves
a lot of the issues that make JSON not idea for configuration files. The biggest issue with JSON5
is its name. By having JSON in the name, it opens the possibility for confusion and that then
opens the possibility for bugs. A new language with different features should be distinct and not
associate itself with an existing language.

Further reading:

- JSON5: https://json5.org/

### Dhall

Dhall is a configuration language that has features such as functions, types and imports.
Having these features in a configuration language is a mistake. It creates a language that is
way too complicated to be a configuration language, but not powerful enough to be a programming language.
If you truly need these features then it's likely a sign you should just use a programming language.
Or as an alternative, create a tool to generate configuration files.

- Dhall: https://dhall-lang.org/#

## Now SC

All the languages described above have at least one of two main issues:

- The language is not ideal for writing configuration files.
- The language is too complicated.

## A Configuration Language for Configuration Files

SC was designed first and foremost for writing configuration files. It is not a general purpose
data serialization language. You should not be building a REST API that uses SC. SC contains futures
to make handling configuration easier, such as variables to make it easy to have dynamic configuration values.

### Simplicity is Key

SC was also designed with simplicity in mind. SC should be as simple as possible for both humans and machines.

Since a lot of configuration files are written by hand, SC should be easy to write by limiting needless verbosity
or unnecessary syntax. SC should be easy to read, since generally code is read a lot more than it is written.
There should not be questions asked like "what does this mean?" or "why didn't this do what I thought it would?".

Since configuration files will be read by machines, SC should be easy to parse. This makes it easy to write implementations
for it in various programming languages. It also makes it easy to write tools that can operate on SC files.

### Familiarity is Key

The SC syntax is quite similar to JSON. This is intentional, but not for the reason you might thing. SC is not a JSON superset,
nor does it ever intend to be. It was not designed to be compatible with JSON. The syntax was designed organically from the ground up.
However, it is beneficial to have syntax that is familiar and similar to other languages that already exist. This makes SC easy
to understand and quick to pick up. Someone who has never heard of SC should be able to glance over an SC file and quickly get a rough idea
of what is going on.

Some key highlights:

- The data types SC supports are common to most programming languages. i.e. booleans, numbers, strings, lists, dictionaries
- The list and dictionary syntax, using square and curly brackets, is pretty common in a lot of programming languages, not to mention JSON.
- The comment syntax, line and block, are common to many programming languages.
- The variable syntax is similar to the variable expansion syntax used in many Unix shells.
