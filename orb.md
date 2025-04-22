Object Representation in Binary
===============================

Orb is an extension of [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md) that adds many commonly used types.



WIP DRAFT
---------

This is currently in early WIP draft form, and subject to extensive changes.



Contents
--------

- [Object Representation in Binary](#object-representation-in-binary)
  - [WIP DRAFT](#wip-draft)
  - [Contents](#contents)
  - [Terms and Conventions](#terms-and-conventions)
  - [Modifications](#modifications)
    - [Allowed values](#allowed-values)
    - [New Type Codes](#new-type-codes)
    - [JSON-like text format](#json-like-text-format)
  - [Structure](#structure)
  - [Encoding](#encoding)
    - [Type Codes](#type-codes)
  - [Timestamp](#timestamp)
  - [UUID](#uuid)
  - [Record Definition](#record-definition)
  - [Record](#record)
  - [Typed Array](#typed-array)
  - [Marker](#marker)
  - [Reference](#reference)
  - [ORT - Object Representation in Text](#ort---object-representation-in-text)
    - [Timestamp](#timestamp-1)
    - [UUID](#uuid-1)
    - [Record Definition](#record-definition-1)
    - [Record](#record-1)
    - [Typed Array](#typed-array-1)
    - [Comment](#comment)
    - [Marker](#marker-1)
    - [Reference](#reference-1)
    - [New literals](#new-literals)
  - [License](#license)



Terms and Conventions
---------------------

**The following bolded, capitalized terms have specific meanings in this document**:

| Term             | Meaning                                                                                          |
| ---------------- | ------------------------------------------------------------------------------------------------ |
| **MUST (NOT)**   | If this directive is not adhered to, the document or implementation is invalid.                  |
| **SHOULD (NOT)** | Every effort should be made to follow this directive, but it's still conformant if not followed. |
| **MAY (NOT)**    | It is up to the implementation to decide whether to do something or not.                         |
| **CAN**          | Refers to a possibility which **MUST** be accommodated by the implementation.                    |
| **CANNOT**       | Refers to a situation which **MUST NOT** be allowed by the implementation.                       |

-----------------------------------------------------------------------------------------------------------------------



Modifications
-------------

ORB should be considered an extension to BONJSON, adding more types and modifying the rules slightly. The BONJSON specification applies to ORB except where this specification overrides it:


### Allowed values

Floating point NaN and infinity values are allowed in ORB.


### New Type Codes

The following type codes (marked RESERVED in BONJSON) correspond to the new types in ORB:

| Type Code | Payload                      | Type      | Description                                |
| --------- | ---------------------------- | --------- | ------------------------------------------ |
| 66        | 64-bit nanoseconds           | Time      | [Timestamp](#timestamp)                    |
| 67        | 128-bit UUID                 | Number    | [UUID](#uuid)                              |
| 90        | Type, length, contents       | Container | [Typed array](#typed-array)                |
| 91        |                              | Reference | [Marker](#marker)                          |
| 92        |                              | Reference | [Reference](#reference)                    |
| 97        |                              | Container | [Record start](#instance)                  |
| 98        |                              | Container | [Record definition start](#type)           |


### JSON-like text format

ORB's textual form (ORT: Object Representation in Text) is mostly the same as JSON, with the following changes:

* `/* */` and `//` can be used as comment delimiters
* Commas are now considered "whitespace"
* Values must be separated by at least one whitespace (key-value pairs only require a `:` in between them, with extra whitespace allowed)
* Encodings for the new types


Structure
---------

ORB follows the same general structure as JSON, but allows more types, and also supports records and references.

**Document**:

    ──[value]──>

**Value**:

    ──┬─>───────────────┬─┬─>───────────┬─┬─>─[string]──────┬─>
      ├─>─[definition]──┤ ╰─>─[marker]──╯ ├─>─[number]──────┤
      ╰─<─<─<─<─<─<─<─<─╯                 ├─>─[object]──────┤
                                          ├─>─[array]───────┤
                                          ├─>─[boolean]─────┤
                                          ├─>─[null]────────┤
                                          ├─>─[timestamp]───┤
                                          ├─>─[uuid]────────┤
                                          ├─>─[typed array]─┤
                                          ├─>─[record]──────┤
                                          ╰─>─[reference]───╯

**Object**:

    ──[begin object]─┬─>────────────────────┬─[end container]──>
                     ├─>─[string]──[value]──┤
                     ╰─<─<─<─<─<─<─<─<─<─<──╯

**Array**:

    ──[begin array]─┬─>──────────┬─[end container]──>
                    ├─>─[value]──┤
                    ╰─<─<─<─<─<──╯

**Typed Array**:

    ──[begin typed array]─┬─>────────────┬─[end]──>
                          ├─>─[element]──┤
                          ╰─<─<─<─<─<─<──╯

**Type**:

    ──[begin type]─┬─>─────────┬─[end container]──>
                   ├─>─[name]──┤
                   ╰─<─<─<─<─<─╯

**Instance**:

    ──[begin instance]─┬─>──────────┬─[end container]──>
                       ├─>─[value]──┤
                       ╰─<─<─<─<─<──╯



Encoding
--------


### Type Codes

Here is the complete table of type codes (including those defined in [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md)):

| Type Code | Payload                      | Type      | Description                                |
| --------- | ---------------------------- | --------- | ------------------------------------------ |
| 00 - 64   |                              | Number    | Integers 0 through 100                     |
| 65        |                              |           | RESERVED                                   |
| 66        | 64-bit nanoseconds           | Time      | [Timestamp](#timestamp)                    |
| 67        | 128-bit UUID                 | Number    | [UUID](#uuid)                              |
| 68        | Arbitrary length string      | String    | Long String                                |
| 69        | Arbitrary length number      | Number    | Big Number                                 |
| 6a        | 16-bit bfloat16 binary float | Number    | 16-bit float                               |
| 6b        | 32-bit ieee754 binary float  | Number    | 32-bit float                               |
| 6c        | 64-bit ieee754 binary float  | Number    | 64-bit float                               |
| 6d        |                              | Null      | Null                                       |
| 6e        |                              | Boolean   | False                                      |
| 6f        |                              | Boolean   | True                                       |
| 70 - 77   | Unsigned integer of n bytes  | Number    | Unsigned Integer                           |
| 78 - 7f   | Signed integer of n bytes    | Number    | Signed Integer                             |
| 80 - 8f   | String of n bytes            | String    | Short String                               |
| 90        | Type, length, contents       | Container | [Typed array](#typed-array)                |
| 91        |                              | Reference | [Marker](#marker)                          |
| 92        |                              | Reference | [Reference](#reference)                    |
| 93 - 96   |                              |           | RESERVED                                   |
| 97        |                              | Container | [Instance start](#instance)                |
| 98        |                              | Container | [Type start](#type)                        |
| 99        |                              | Container | Array start                                |
| 9a        |                              | Container | Object start                               |
| 9b        |                              | Container | Container end                              |
| 9c - ff   |                              | Number    | Integers -100 through -1                   |



Timestamp
---------

A timestamp is a 64-bit integer representing the number of nanoseconds elapsed since 00:00 UTC January 1st, 1900 (disregarding any human adjustments such as leap seconds). It can support timestamps up to the year 2484.




UUID
----

A UUID consists of 128 bits of data in big endian order, per RFC 9562 section 4.



Record Definition
-----------------

A record definition declares a new type of record, listing what the field names are. A [record](#record) will reference this definition and list out the corresponding values.

A definition can be declared anywhere a value can be declared, and is considered "invisible" to the document structure (it is not considered a value and does not affect the document structure).

A record definition begins with a globally unique (to the document) definition name, followed by a series of field names, and is terminated wuth an end container

    [definition name] [field name] ... [end container]

Definitions are not structurally significant in an of themselves. They are effectively invisible to the document structure, so a definition following an object name will not fulfil the role of the name's corresponding value.

A definition **MUST NOT** contain duplicate field names.
A definition's name **MUST** be unique to the entire document.
A definition **MUST** be declared before any [instances](#instance) that reference it.



Record
------

A record is a data instance that lists out the values corresponding to its [record definition](#record-definition), in the same order as the field names were defined.

    [definition name] [field value] ... [end container]

A record **MUST** contain exactly the same number of values as the number of field names in the [definition](#record-definition) it references (and in the same order).

A record **MUST** not appear before the [definition](#record-definition) it references.



Typed Array
-----------

A typed array is a more compact representation of an array, where all values are of the same type and size. This allows for efficient copying of data into a CPU-native array.

The type code is followed by the element's type code, and then chunks of values.

    [type code] [element type code] [chunk] ...

Each chunk consists of a chunk header (containing a length field), followed by that many elements of data, until the end of a chunk where the continuation bit is 0:

    [chunk header] [chunk data]

The following type codes may be used as element types:

| Type Code | Payload                      | Type      | Description                                |
| --------- | ---------------------------- | --------- | ------------------------------------------ |
| 66        | 64-bit nanoseconds           | Time      | [Timestamp](#timestamp)                    |
| 67        | 128-bit UUID                 | Number    | [UUID](#uuid)                              |
| 6a        | 16-bit bfloat16 binary float | Number    | 16-bit float                               |
| 6b        | 32-bit ieee754 binary float  | Number    | 32-bit float                               |
| 6c        | 64-bit ieee754 binary float  | Number    | 64-bit float                               |
| 70 - 77   | Unsigned integer of n bytes  | Number    | Unsigned Integer                           |
| 78 - 7f   | Signed integer of n bytes    | Number    | Signed Integer                             |

TODO: Allow fixed-length strings (1-15 bytes)?



Marker
------

TODO: Keep this?

A marker marks the value following it with an identifier that can be [referenced](#reference) later.

The marker itself is not a value, and is thus invisible structurally.



Reference
---------

TODO: Keep this?

A reference is a value placeholder that refers to a [previously marked value](#marker).

A reference **MUST** refer to a reference made earlier in the document (no forward referencing).




ORT - Object Representation in Text
-----------------------------------

ORT is the textual representation of ORB.

ORT follows most of the conventions of JSON, with some extra types, and some modifications:
- Values are separated by one or more whitespace characters.
- Commas are considered whitespace.
- Strings that don't contain structural characters or match reserved literals can be used without quotes?

Any valid JSON document can be read as an ORT document.


### Timestamp

Use RFC 3339

### UUID

RFC 9562

### Record Definition

`@my_record<name age score>`

### Record

`@my_record{"bill" 40 85923395}`

### Typed Array

`@type[1 2 3]`

Where type is:
- `u8` to `u64`
- `i8` to `i64`
- `timestamp`
- `uuid`
- `f16`, `f32`, `f64`

### Comment

```
/* */
//
```

### Marker

`&remember_me:{"name": "Mighty Thor" "age": 1000}`


### Reference

`{"Thor": $remember_me}`


Nested comments: Simple string search for */ to terminate, or /* to initiate the next nesting level. If strings contain confounding chars, too bad.

### New literals

- `nan`
- `inf` / `-inf`

So all literals will be:

- `true`
- `false`
- `null`
- `nan`
- `inf`
- `-inf`



License
-------

Copyright (c) 2025 Karl Stenerud. All rights reserved.

Distributed under the [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/legalcode) ([license deed](https://creativecommons.org/licenses/by/4.0).
