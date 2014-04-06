OGDL Specification Version 2.0
==============================

####Rolf Veen     <rolf.veen@gmail.com>
####Hǎiliàng Wáng <hwang.dev@gmail.com>

Copyright (c) 2014, Rolf Veen and Hǎiliàng Wáng. All rights reserved.

This specification is licensed under the Creative Commons Attribution 4.0
International License. To view a copy of this license, visit
    http://creativecommons.org/licenses/by/4.0/

Introduction
------------
OGDL (Ordered Graph Data Language) is a simple, extensible, text-based data
serialization format.

The data model of OGDL is a tree of string nodes. It is simple but flexible
enough to be extended to represent a rich set of data structures, including
cyclic graphs and application-defined types.

OGDL has two syntax styles: flow and block. They share the same data model and
extensions, but cannot be mixedly used.

The specification is structured as below:
* Core
    + Data model
    + Common syntax
    + Two syntax styles
        - Flow syntax
        - Block syntax
* Extensions
    + Representations of common data types.

Notation
--------
The syntax is specified using a variant of Extended Backus-Naur Form (EBNF),
based on [W3C XML EBNF](http://www.w3.org/TR/2006/REC-xml11-20060816/#sec-notation),
which is extended with the following definitions:
* **U+XXXX** matches Unicode code point 0xXXXX.
* **EOF** matches the end of the file.
* **A{n}** matches exactly n occurrences of A.
* **A{,m}** matches zero to m occurrences of A.
* **A{n,}** matches n or more than n occurrences of A.
* **A{n,m}** matches n to m occurrences of A.

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
    - Both composition and association links are unidirectional: from the parent
      to the child.

Parser EBNF:

    node  ::= (value | list) node?
    list  ::= list_start (node (list_sep node)* list_sep?)? list_end
    value ::= string

Note:
* Two node connected by an association link is represented by two consecutive
  nodes.
* When a list node has an association link to another node, the list node can be
  seen as a value encoded in the form of a list.

###Common syntax

An OGDL text is a sequence of [Unicode](http://unicode.org/) code points encoded
in UTF8.

Except \t (U+0009), \n (U+000A) and \r (U+000D), code points less than U+0032 are
invalid and should not appear in an OGDL text.

    char_visible    ::= [^0..32]
    char_space      ::= [ \t]
    char_inline     ::= char_visible | char_space
    char_break      ::= [\r\n]
    char_space      ::= [ \t] | char_break
    char_any        ::= char_inline | char_break
    char_invalid    ::= ^char_any

Line ending (newline) could either be \r, \n or \r\n.

    newline         ::= char_break | '\r\n'

While the format of list is syntax dependent, the format of string value is
shared between flow and block syntaxes.

    char_delimiter  ::= [{}(),]
    quoted_char     ::= (char_inline - '"') | '\\"'
    quoted_string   ::= '"' quoted_char* '"'
    unquoted_char   ::= char_visible - char_delimiter
    unquoted_string ::= unquoted_char+
    string          ::= unquoted_string | quoted_string

OGDL can represent a cyclic graph by using reference IDs. A cyclic reference id
is an unquoted string in the form ^id, where id is a unique ID within an OGDL
file.

A referenced node is defined as the only child (descendant) of the reference id,
linked by an association link. It can be referenced by the reference id alone.
It should be defined only once but can be referenced multiple times.

    ref_id          ::= '^' unquoted_char+
    referenced_node ::= ref_id node

A type is an unquoted string in the form !type. A typed node is defined as the
only child (descendant) of the type, linked by an association link. 

    type            ::= '!' unquoted_char+
    typed_node      ::= type node

When both a cyclic reference and a type are defined for a node, it doesn't
matter which comes first. e.g.

    either
        ref_id type node
    or
        type ref_id node

###Flow syntax
Flow syntax style is a curly braces and comma separated syntax.

    inline_comment  ::= '//' char_inline* (newline | EOF)
    list_start      ::= '{'
    list_end        ::= '}'
    list_sep        ::= ','

###Block syntax
Block syntax style is a line-based, space indented syntax.

**TODO**

Extensions
----------
Core definitions only include value, list, link, cyclic reference and type name.
Format for other data structures can be extended with these premitives. An OGDL
implementation should provide implementation specific way for users to define
the format of their own types.

In this specification, format for common types are specified. These types are
intended to be mapped to builtin types or types in standard libraries, and
format of these types are fixed and cannot be overriden.

###nil
The special string nil is used to represent an uninitialized nullable value or
list.

    nil             ::= 'nil'

###array
An array is represented with a list of array elements.

    array_element   ::= node
    array           ::= list

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
as a key node, assotiated with a child node of the value.

    key_value       ::= node node
    map             ::= list

An object with two string fields.

    Flow syntax:
        {FieldX "a", FieldY "b"}

    Block syntax:
        FieldX 1
        FieldY 2

A map with string key and integer value.

    Flow syntax:
        {"a" 1, "b" 2}
    Block syntax:
        "a" 1
        "b" 2

A map with struct key and boolean value.

    Flow syntax:
        {{FieldX "a", FieldY 1} true, {FieldX "b", FieldY 2} false}
    Block syntax:
        (FieldX "a", FieldY 1) true
        (FieldX "b", FieldY 2) false

###Interpreted string
Interpreted string is a double quoted string, that can interpret certain escape
sequences.

    interpreted_string ::= quoted_string

Escape sequences:

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

    boolean    ::= 'true' | 'false'

###Numeric value
Numeric value is an unquoted string that encode a number.

    sign       ::= '+' | '-'
    decimals   ::= [1-9] [0-9]*

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
A date/time value is an unquoted string encoded with
[RFC3339](http://www.rfc-editor.org/rfc/rfc3339.txt), e.g.

    2006-01-02T15:04:05.999999999Z07:00

###IP address
An IP address is either an IPv4 or IPv6 address.

    ip         ::= ipv4 | ipv6

An IPv4 address value is an unquoted string encoded with dot-decimal notation:

    ipv4       ::= decimals ('.' decimals){3}

e.g.

    74.125.19.99

An IPv6 address value is an unquoted string encoded with
[RFC5952](http://www.rfc-editor.org/rfc/rfc5952.txt), e.g.

    2001:4860:0:2001::68

