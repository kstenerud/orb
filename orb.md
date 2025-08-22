ORB - Object Representation in Binary
=====================================

ORB is the binary representation of [ORT](ort.md).

ORB is an extension of [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md) that adds support for the most requested things missing from JSON (and by extension, BONJSON):

 * Floating point NaN and infinity values are allowed.
 * [Timestamp](#timestamp) type
 * [Identifier](#identifier) (UUID) type
 * [Typed arrays](#typed-array) (including a byte array type)

Any BONJSON document can also be read as an ORB document.



WIP DRAFT
---------

This is currently a work-in-progress, and subject to change.



Contents
--------

- [ORB - Object Representation in Binary](#orb---object-representation-in-binary)
  - [WIP DRAFT](#wip-draft)
  - [Contents](#contents)
  - [Terms and Conventions](#terms-and-conventions)
  - [Structure](#structure)
  - [Encoding](#encoding)
    - [Type Codes](#type-codes)
  - [New Types](#new-types)
    - [Timestamp](#timestamp)
    - [Identifier](#identifier)
    - [Typed Array](#typed-array)
  - [Formal Grammar](#formal-grammar)
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



Structure
---------

ORB has the same general structure as BONJSON and JSON, with more types.

Except where this document specifies otherwise, ORB follows the same rules and structure as the [BONJSON specification](https://github.com/kstenerud/bonjson/blob/main/bonjson.md).

**Document**:

    ──[value]──>

**Value**:

    ──┬─>─[string]──────┬─>
      ├─>─[number]──────┤
      ├─>─[object]──────┤
      ├─>─[array]───────┤
      ├─>─[boolean]─────┤
      ├─>─[null]────────┤
      ├─>─[timestamp]───┤
      ├─>─[identifier]──┤
      ╰─>─[typed array]─╯

**Object**:

    ──[begin object]─┬─>────────────────────┬─[end container]──>
                     ├─>─[string]──[value]──┤
                     ╰─<─<─<─<─<─<─<─<─<─<──╯

**Array**:

    ──[begin array]─┬─>──────────┬─[end container]──>
                    ├─>─[value]──┤
                    ╰─<─<─<─<─<──╯

**Typed Array**:

    ──[begin typed array]─┬─>────────────┬─[end (implied)]──>
                          ├─>─[element]──┤
                          ╰─<─<─<─<─<─<──╯



Encoding
--------


### Type Codes

Here is the complete table of type codes (including those defined in [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md)):

| Type Code | Payload                      | Type      | Description                 | New |
| --------- | ---------------------------- | --------- | --------------------------- | --- |
| 00 - 64   |                              | Number    | Integers 0 through 100      |     |
| 65        | 64-bit nanoseconds           | Time      | [Timestamp](#timestamp)     |  Y  |
| 66        | 128-bit Identifier           | Number    | [Identifier](#identifier)   |  Y  |
| 67        | Type, length, contents       | Container | [Typed array](#typed-array) |  Y  |
| 68        | Arbitrary length string      | String    | Long String                 |     |
| 69        | Arbitrary length number      | Number    | Big Number                  |     |
| 6a        | 16-bit bfloat16 binary float | Number    | 16-bit float                |     |
| 6b        | 32-bit ieee754 binary float  | Number    | 32-bit float                |     |
| 6c        | 64-bit ieee754 binary float  | Number    | 64-bit float                |     |
| 6d        |                              | Null      | Null                        |     |
| 6e        |                              | Boolean   | False                       |     |
| 6f        |                              | Boolean   | True                        |     |
| 70 - 77   | Unsigned integer of n bytes  | Number    | Unsigned Integer            |     |
| 78 - 7f   | Signed integer of n bytes    | Number    | Signed Integer              |     |
| 80 - 8f   | String of n bytes            | String    | Short String                |     |
| 90 - 99   |                              |           | RESERVED                    |     |
| 9a        |                              | Container | Array start                 |     |
| 9b        |                              | Container | Object start                |     |
| 9c - ff   |                              | Number    | Integers -100 through -1    |     |



New Types
---------

### Timestamp

A timestamp is a 64-bit unsigned integer representing the number of nanoseconds elapsed since 00:00 UTC January 1st, 1900 (disregarding any leap seconds, or the fact that UTC didn't exist in 1900).

The actual anchor point of the timestamp is 00:00 UTC January 1st, 1970 (UNIX epoch) with a bias applied. Subtracting the bias value of 2208988800000000000 seconds `(70 years * 365 days + 17 leap days) * 24 hours * 60 minutes * 60 seconds * 1000000000 nanoseconds` from the timestamp yields a [UNIX epoch](https://en.wikipedia.org/wiki/Unix_time) timestamp (in nanoseconds).

Nanosecond-precision values are supported from the year 1900 to the year 2484.

**Example**:

    TODO


### Identifier

An identifier is a 128-bit universally unique identifier, represented as binary per [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html).

**Example**:

    TODO


### Typed Array

A typed array is a more compact representation of an array, where all elements are of the same type and size, and are written without type codes. This allows for efficient copying of data to/from a platform-native array.

Typed arrays are encoded in chunks, like how BONJSON encodes [long strings](https://github.com/kstenerud/bonjson/blob/main/bonjson.md#long-string):

The "typed array" type code is followed by the element's type code, and then chunks of elements.

    [typed array] [element type code] [chunk] ...
        0x67              ...           ...   ...

Each chunk consists of a chunk header (containing a length field), followed by that many elements (not necessarily bytes) of data, until the end of a chunk where the continuation bit is 0:

    [chunk header] [chunk data]

The following type codes may be used as element types:

| Type Codes     | Payload                            | Type   | Description               |
| -------------- | ---------------------------------- | ------ | ------------------------- |
| 66             | 64 bit nanoseconds                 | Time   | [Timestamp](#timestamp)   |
| 67             | 128 bit identifier                 | Number | [Identifier](#identifier) |
| 6a, 6b, 6c     | 16, 32, 64 bit float               | Number | Floating point            |
| 70, 71, 73, 77 | 8, 16, 32, 64 bit unsigned integer | Number | Unsigned integer          |
| 78, 79, 7b, 7f | 8, 16, 32, 64 bit signed integer   | Number | Signed integer            |

**Examples**:

    TODO



Formal Grammar
--------------

```dogma
dogma_v1 utf-8
- identifier  = orb_v1
- description = Object Representation in Binary, version 1
- dogma       = https://github.com/kstenerud/dogma/blob/master/v1/dogma_v1.0.md

document          = byte_order(lsb, ordered_document);
ordered_document  = value;

value             = string | number | boolean | null | object | array | timestamp | uuid | record | typed_array | reference;
key               = string;

# Types

array             = u8(0x99) & value* & end_container;
object            = u8(0x9a) & (string & value)* & end_container;
end_container     = u8(0x9b);

number            = int_small | int_unsigned | int_signed | float_16 | float_32 | float_64 | big_number;
int_small         = i8(-100~100);
int_unsigned      = u4(7) & u1(0) & u3(var(count, ~)) & ordered(uint((count+1)*8, ~));
int_signed        = u4(7) & u1(1) & u3(var(count, ~)) & ordered(sint((count+1)*8, ~));
float_16          = u8(0x6a) & f16(~);
float_32          = u8(0x6b) & f32(~);
float_64          = u8(0x6c) & f64(~);
big_number        = u8(0x69)
                  & var(header, big_number_header)
                  & [
                        header.sig_length > 0: ordered(sint(header.exp_length*8, ~))
                                             & ordered(uint(header.sig_length*8, ~))
                                             ;
                    ]
                  ;
big_number_header = u5(var(sig_length, ~)) & u2(var(exp_length, ~)) & u1(var(sig_negative, ~));
timestamp         = u8(0x66) & u64(~);
uuid              = u8(0x67) & id128(~); # Note: Big endian
marker            = u8(0x91) & identifier;
reference         = u8(0x92) & identifier;
record            = u8(0x97) & identifier & value* & end_container;
record_definition = u8(0x98) & identifier & key* & end_container;

boolean           = true | false;
false             = u8(0x6e);
true              = u8(0x6f);

null              = u8(0x6d);

typed_array       = u8_array | u16_array | u32_array | u64_array
                  | s8_array | s16_array | s32_array | s64_array
                  | f16_array | f32_array | f64_array
                  | ts_array
                  | id_array
                  ;
u8_array          = array_type(0x70, u8(~));
u16_array         = array_type(0x71, u16(~));
u32_array         = array_type(0x73, u32(~));
u64_array         = array_type(0x77, u64(~));
s8_array          = array_type(0x78, s8(~));
s16_array         = array_type(0x79, s16(~));
s32_array         = array_type(0x7b, s32(~));
s64_array         = array_type(0x7f, s64(~));
f16_array         = array_type(0x6a, f16(~));
f32_array         = array_type(0x6b, f32(~));
f64_array         = array_type(0x6c, f64(~));
ts_array          = array_type(0x66, u64(~));
id_array          = array_type(0x67, id128(~)); # Note: Big endian
array_type(elem_type, elem) = u8(0x90) & elem_type & elem_chunk(elem, 1)* & elem_chunk(elem, 0);
elem_chunk(elem, hasNext) = chunked(var(count, ~), hasNext) & elem{count};

string                = string_short | string_long;
string_short          = u4(8) & u4(var(count, ~)) & sized(count*8, char_string*);
string_long           = u8(0x68) & string_chunk(1)* & string_chunk(0);
string_chunk(hasNext) = chunked(var(count, ~), hasNext) & sized(count*8, char_string*);

chunked(len, hasNext) = length(len * 2 + hasNext);
length(l)             = ordered([
                                    l >=                 0 & l <=               0x7f: uint( 7, l) & uint(1, 0x01);
                                    l >=              0x80 & l <=             0x3fff: uint(14, l) & uint(2, 0x02);
                                    l >=            0x4000 & l <=           0x1fffff: uint(21, l) & uint(3, 0x04);
                                    l >=          0x200000 & l <=          0xfffffff: uint(28, l) & uint(4, 0x08);
                                    l >=        0x10000000 & l <=        0x7ffffffff: uint(35, l) & uint(5, 0x10);
                                    l >=       0x800000000 & l <=      0x3ffffffffff: uint(42, l) & uint(6, 0x20);
                                    l >=     0x40000000000 & l <=    0x1ffffffffffff: uint(49, l) & uint(7, 0x40);
                                    l >=   0x2000000000000 & l <=   0xffffffffffffff: uint(56, l) & uint(8, 0x80);
                                    l >= 0x100000000000000 & l <= 0xffffffffffffffff: uint(64, l) & uint(8, 0x00);
                                ]);

# Primitives

u1(v)             = uint(1, v);
u2(v)             = uint(2, v);
u3(v)             = uint(3, v);
u4(v)             = uint(4, v);
u5(v)             = uint(5, v);
u8(v)             = uint(8, v);
u16(v)            = ordered(uint(16, v));
u32(v)            = ordered(uint(32, v));
u64(v)            = ordered(uint(64, v));
i8(v)             = sint(8, v);
i16(v)            = ordered(sint(16, v));
i32(v)            = ordered(sint(32, v));
i64(v)            = ordered(sint(64, v));
f16(v)            = ordered(sized(16, float(32, v))); # bfloat16
f32(v)            = ordered(float(32, v));
f64(v)            = ordered(float(64, v));
id128(v)          = byte_order(msb, uint(128, v));
char_string       = unicode(C,L,M,N,P,S,Z);
```



License
-------

Copyright (c) 2025 Karl Stenerud. All rights reserved.

Distributed under the [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/legalcode) ([license deed](https://creativecommons.org/licenses/by/4.0)).
