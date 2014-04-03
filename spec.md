OGDL Specification Version 2.0
==============================

####Rolf Veen     <rolf.veen@gmail.com>
####Hǎiliàng Wáng <hwang.dev@gmail.com>

Copyright (c) 2014, Rolf Veen and Hǎiliàng Wáng. All rights reserved.

This specification is licensed under the Creative Commons Attribution 4.0
International License. To view a copy of this license, visit

    http://creativecommons.org/licenses/by/4.0/

Overview
--------
OGDL (Ordered Graph Data Language) is a simple, readable and extensible data
serialization format.

The data model of OGDL is simply a tree with special node representing cyclic
references and types. It is easy to parse and is flexible enough to be extended
to represent arbitrary data structures.

OGDL has two distinct syntaxes sharing the same data model:

1. Flow syntax: curly braces and comma separated syntax.
2. Block syntax: space indented syntax.

The two syntaxes cannot be used in a mixture.

In the following sections:
* Core: specifies the core definitions of data model and common syntaxes.
* Flow syntax: specifies the definitions specific to flow syntax.
* Block syntax: specifies the definitions specific to block syntax.
* Extension: specifies how to extend OGDL core definitions and extentions of
  standard types.

Core
----
###Data Model
The data model of OGDL is a tree of nodes composed of value, list and link:
* There are two kinds of nodes: value and list.
    - A value is a string value.
    - A list is an ordered list of nodes.
* There are two kinds of links: composition and association.
    - A composition link is the link from a list to each of its element node.
    - An association link is the link from one node to another. A node can has
      at most one incoming and one outgoing association link.
    - Both composition and association links are unidirectional: from parent to
      child.
* When a list node has an association link to another node, the list node can be
  seen as a value encoded in the form of a list.

EBNF:

    node  ::= (value | list) node?
    list  ::= list_start node (list_sep node)* list_sep? list_end
    value ::= string

###Common definitions
There are definitions common to both flow and block syntaxes:

    char_visible    ::= [^0..32]
    char_space      ::= [ \t]
    char_inline     ::= char_visible | char_space
    char_delimiter  ::= [{}(),:]
    char_break      ::= [\r\n]
    char_space      ::= [ \t] | char_break
    new_line        ::= char_break | '\r\n'
    EOF             ::= (end of file)

###Value
While the format of list and link are syntax dependent, the format of value is
shared between flow and block syntaxes.

    quoted_char     ::= (char_inline - '"') | '\\"'
    quoted_string   ::= '"' quoted_char* '"'
    unquoted_char   ::= char_visible - char_delimiter
    unquoted_string ::= unquoted_char+ | ':'
    string          ::= unquoted_string | quoted_string

####Cyclic reference
OGDL can represent a graph by reference IDs. A cyclic reference id is an
unquoted string in the form ^id, where id is a unique ID within an OGDL file.

A referenced node is defined as the only child (descendant) of the reference id,
linked by an association link. It can be referenced as an value in the form ^id
without any children. It should be defined only once but can be referenced
multiple times.

####Typed node
A type is an unquoted string in the form !type. A typed node is defined as the
only child (descendant) of the type, linked by an association link. 

When both a cyclic reference and a type are defined for a node, it doesn't
matter which comes first.

Flow syntax
-----------
In addiction to the shared syntax:

    inline_comment  ::= '//' char_inline* (new_line | EOF)
    list_start      ::= '{'
    list_end        ::= '}'
    list_sep        ::= ','

Notes:
* An association link is represented by space characters char_space+, i.e. two
  nodes separated by spaces are connected with an association link.

Block syntax
------------
TODO

Extension
---------
Core definitions only contain value, list, link, cyclic reference and type name.
Format for other data structures can be extended with these premitives. An OGDL
implementation should provide implementation specific way for users to define
the format of their own types.

In this specification, format for certain types are specified. These types are
intended to be mapped to builtin types or types in standard libraries, and
format of these types are fixed and cannot be overriden.

###nil
The special string nil is used to represent an uninitialized nullable value or
list.

###array
An array is represented with a list of array elements.

An integer array with 3 elements.

    Flow syntax:
        {1, 2, 3}

    Block syntax:
        1
        2
        3

An array of integer array.

    Flow syntax:
        {{1, 2, 3}, {4, 5}, {6}}
    Block syntax:
        -
            1
            2
            3
        -
            4
            5
        -
            6

###map
An map is represented with a list of key-value pairs. Each pair is represented
as a key node, assotiated with an optional child node of colon that is then
assotiated with a child node of the value.

An object with two string fields.

    Flow syntax:
        {FieldX: "a", FieldY: "b"}

    Block syntax:
        FieldX 1
        FieldY 2

A map with string key and integer value.

    Flow syntax:
        {"a": 1, "b": 2}
    Block syntax:
        "a" 1
        "b" 2

A map with struct key and boolean value.

    Flow syntax:
        {{FieldX: "a", FieldY: 1}: true, {FieldX: "b", FieldY: 2}: false}
    Block syntax:
        (FieldX: "a", FieldY: 1) true
        (FieldX: "b", FieldY: 2) false

Note:
* colon is optional for both flow and block syntax.

###Interpreted string
Interpreted string is a double quoted string, that can interpret certain escape
sequences.

    \a    U+0007 alert or bell
    \b    U+0008 backspace
    \f    U+000C form feed
    \n    U+000A line feed or newline
    \r    U+000D carriage return
    \t    U+0009 horizontal tab
    \v    U+000b vertical tab
    \\    U+005c backslash
    \"    U+0022 double quote
    \x    value of two hexadecimal digits followed by \x
    \u    value of exactly 4 hexadecimal digits followed by \u
    \U    value of exactly 8 hexadecimal digits followed by \U

###Boolean value
Boolean value is an unquoted string of either true of false.

###Numeric value
Numeric value is an unquoted string that encode a number.

    sign       ::= '+' | '-'
    decimal    ::= [0-9]
    decimals   ::= decimal+ 

####Integer
    integer    ::= sign? decimals

####Float
Float value is an unquoted string that encode a floating point number:

    exponent   ::= ( 'e' | 'E' ) ( '+' | '-' )? decimals
    float_base ::= (decimals '.' decimal* exponent?) |
                   (decimals exponent) |
                   ('.' decimals exponent?)
    float      ::= sign? float_base

####Complex
    int_float  ::= decimals | float_base
    complex    ::= sign? int_float sign int_float 'i'

###Date/time
A date/time value is a quoted string encoded with RFC3339, e.g.

    "2006-01-02T15:04:05.999999999Z07:00"

###IP address
An IPv4 address value is an unquoted string that encode an IPv4 address. e.g.

    74.125.19.99

An IPv6 address value is a quoted string that encode an IPv6 address. e.g.

    "2001:4860:0:2001::68"

