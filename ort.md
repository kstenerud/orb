ORT - Object Representation in Text
===================================

ORT is the textual representation of [ORB](orb.md).

ORT is an extension of [JSON](https://www.rfc-editor.org/rfc/rfc8259) that adds support for the most requested things missing from JSON:

 * Floating point NaN and infinity values are allowed
 * [Timestamp](#timestamp) type
 * [Identifier](#identifier) (UUID) type
 * [Typed arrays](#typed-array) (including a byte array type)
 * Comments
 * Hexadecimal notation
 * The comma character (`,`) is now considered "whitespace"
 * A new Unicode escape sequence that supports the entire codepoint range

Any JSON document following the stricter rules of [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md) (no duplicate keys, UTF-8 only, no invalid UTF-8 characters) can also be read as an ORT document.



WIP DRAFT
---------

This is currently a work-in-progress, and subject to change.



Contents
--------

- [ORT - Object Representation in Text](#ort---object-representation-in-text)
  - [WIP DRAFT](#wip-draft)
  - [Contents](#contents)
  - [Terms and Conventions](#terms-and-conventions)
  - [Structure](#structure)
  - [New Types](#new-types)
    - [Timestamp](#timestamp)
    - [Identifier](#identifier)
    - [Typed Array](#typed-array)
  - [Other Modifications](#other-modifications)
    - [Comment](#comment)
    - [Unicode Codepoint Escape Sequence](#unicode-codepoint-escape-sequence)
    - [New literals](#new-literals)
    - [Quality of Life](#quality-of-life)
      - [The Comma is Whitespace](#the-comma-is-whitespace)
    - [Hexadecimal Notation](#hexadecimal-notation)
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

ORT has the same general structure as JSON, with more types.

Except where this document specifies otherwise, ORT follows the textual format of [JSON](https://www.rfc-editor.org/rfc/rfc8259) and the enhanced structural and security rules of [BONJSON](https://github.com/kstenerud/bonjson/blob/main/bonjson.md).

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

    ──[begin typed array]─┬─>────────────┬─[end container]──>
                          ├─>─[element]──┤
                          ╰─<─<─<─<─<─<──╯



New Types
---------

### Timestamp

Timestamps are represented as text per [RFC 3339](https://www.rfc-editor.org/rfc/rfc3339), with a subsecond precision supported to the nanosecond.

**Note**: Only timestamps in the years ranging from 1900 to 2484 can be represented in [ORB](orb.md).

**Example**: `1985-04-12T23:20:50.521422010Z`


### Identifier

An identifier is a 128-bit universally unique identifier, represented as text per [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html).

**Example**: `2489E9AD-2EE2-8E00-8EC9-32D5F69181C0`


### Typed Array

A typed array is a more compact (in binary) representation of an array, where all values are of the same type and size.

Typed arrays look like regular JSON array notation, except that they are prepended with type information like so: `@some_type[ ... ]`

The following types are supported:

 * Signed Integers: `i8`, `i16`, `i32`, `i64`
 * Unsigned Integers: `u8`, `u16`, `u32`, `u64`
 * Timestamp: `ts`
 * Identifier: `id`
 * Floating Point: `f16`, `f32`, `f64`

All elements **MUST** fit into the array's element type without data loss (other than the normal loss of floating point precision that is to be expected when converting from decimal string representation to binary floating point).

**Examples**:

 * `@i16[1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024]`
 * `@f32[1 1.5 89.91 12.412225e20]`
 * `@ts[2020-01-18T21:05:44.985929934Z 2020-01-18T21:05:46.995254234Z 2020-01-18T21:05:49.004576523Z]`



Other Modifications
-------------------

### Comment

Comments are treated like whitespace, and aren't preserved when converting to ORB format.

A single-line comment (`//`) continues until the end of the current line.

A multiline comment (`/* */`) runs until the corresponding end-delimiter sequence (`*/`) is encountered. Multiline comments **CAN** be nested, but care must be taken: if a multiline comment end-delimiter is encountered in what would normally be a different context (such as a string), it will be picked up and incorrectly used; no attempt is made to deduce the correct context of comment end-delimiter sequences during multiline comment processing.


### Unicode Codepoint Escape Sequence

While JSON already has a Unicode escape sequence (`\uXXXX`), it only supports codepoints within the basic multilingual plane. ORT adds a second Unicode escape sequence that supports the full range of Unicode characters, as well as any future additions to the Unicode standard.

The escape sequence begins with a backslash (`\`) character, followed by an opening square brace (`[`), followed by 1 to 8 hexadecimal digits representing the codepoint, and finally terminated by a closing square brace (`]`). Leading zeroes **CAN** be used for stylistic purposes (e.g. `\[0020]`), but encoders **MUST** by default not output leading zeroes.

**Examples**:

| Sequence   | Digits | Codepoint | Character     |
| ---------- | ------ | --------- | ------------- |
| `\[c]`     | 1      | u+000c    | Form Feed     |
| `\[df]`    | 2      | u+00df    | ß  (Eszett)   |
| `\[101]`   | 3      | u+0101    | ā  (a macron) |
| `\[2191]`  | 4      | u+2191    | ↑  (up arrow) |
| `\[1D11E]` | 5      | u+1d11e   | 𝄞  (G clef)   |
| `\[1f415]` | 5      | u+1f415   | 🐕 (dog)      |

`"gro\[df]e"` = `"große"`


### New literals

The following floating point literal values are supported in ORT:

| Literal  | Meaning                  |
| -------- | ------------------------ |
| `inf`    | Infinity                 |
| `qnan`   | Not a number (quiet)     |
| `snan`   | Not a number (signaling) |

**Note**: ORT does not distinguish NaN payload data other than the quiet bit. Any other NaN payload information will be lost when converting from a binary floating point value.


### Quality of Life

#### The Comma is Whitespace

In JSON, the comma only serves to separate elements, and is otherwise just an annoyance. Commas are therefore treated as whitespace in ORT.

The following arrays are equivalent:

* `[1, 2, 3, 4]`
* `[1,,, 2,, 3,,,,,,   4]`
* `[1 2 3 4]`
* `[1/**/2/**/3/**/4]`

Whitespace is only required in the following situations:

* Between elements in an array
* Between key-value pairs in an object


### Hexadecimal Notation

Numeric types can be written using hexadecimal notation.

**Examples**:

```ort
{
  "integer": 123
  "hex integer": 0x7b
  "float": 1.2345e-13
  "hex float": 0x1.15fc14727b686p-43
}
```



Formal Grammar
--------------

```dogma
dogma_v1 utf-8
- identifier  = ort_v1
- description = Object Representation in Text, version 1
- dogma       = https://github.com/kstenerud/dogma/blob/master/v1/dogma_v1.0.md

document               = value;

value                  = WSC* & (string | number | boolean | null | object | array | typed_array | timestamp | id) & WSC*;
key                    = WSC* & string & WSC*;

array                  = container('[', value, ']');
object                 = container('{', key_value, '}');
boolean                = true | false;
id                     = digit_0_f{8} & '-'
                       & digit_0_f{4} & '-'
                       & digit_0_f{4} & '-'
                       & digit_0_f{4} & '-'
                       & digit_0_f{12}
                       ;
string                 = '"' & (char | escape_sequence)* & '"';
escape_sequence        = '\\' & ( '"' | '\\' | '/' | 'b'
                                | 'f' | 'n' | 'r' | 't'
                                | ('u' & digit_0_f{4})
                                | ('[' & digit_0_f{1~8} & ']')
                                )
                       ;

number                 = '-'? & (ufloat_dec | ufloat_hex | inf | qnan | snan);
ufloat_dec             = uinteger_dec & fractional_dec? & exponent_dec?;
ufloat_hex             = uinteger_hex & fractional_hex? & exponent_hex?;
integer                = '-'? & uinteger;
uinteger               = uinteger_dec | uinteger_hex;
uinteger_dec           = '0' | (digit_1_9 & digit_0_9*);
uinteger_hex           = '0' | (digit_1_f & digit_0_f*);
fractional_dec         = '.' & digit_0_9+;
fractional_hex         = '.' & digit_0_f+;
exponent_dec           = ('e' | 'E') & ('+' | '-')? & uinteger_dec;
exponent_hex           = ('p' | 'P') & ('+' | '-')? & uinteger_dec;

timestamp              = year & '-' & month & '-' & day & 'T' & hour & ':' & minute & ':' & second & subsecond? & 'Z';
year                   = string_num_4(1900~2484);
month                  = string_num_2(1~12);
day                    = string_num_2(1~31);
hour                   = string_num_2(0~23);
minute                 = string_num_2(0~59);
second                 = string_num_2(0~60);
subsecond              = fractional_dec;

typed_array            = integer_array | uinteger_array | float_array | timestamp_array | id_array;
integer_array          = container('@i' & ('8' | '16' | '32' | '64') & '[', integer, ']');
uinteger_array         = container('@u' & ('8' | '16' | '32' | '64') & '[', uinteger, ']');
float_array            = container('@f' & ('16' | '32' | '64') & '[', number, ']');
timestamp_array        = container('@ts' & '[', timestamp, ']');
id_array               = container('@id' & '[', id, ']');

comment                = comment_single_line | comment_multi_line;
comment_single_line    = "//" & (char* ! LINE_END) & LINE_END;
comment_multi_line     = "/*" & ((char* ! "/*") | comment_multi_line) & "*/";

container(begin, type, end) = begin & WSC* & items(type)? & WSC* & end;
items(type)            = type & (WSC+ & type)*;
key_value              = key & ':' & value;

string_num_4(value)    = ('0' + (value/1000) % 10) & ('0' + (value/100) % 10) & ('0' + (value/10) % 10) & ('0' + value % 10);
string_num_2(value)    = ('0' + (value/10) % 10) & ('0' + value % 10);

true                   = 'true';
false                  = 'false';
null                   = 'null';
qnan                   = 'qnan';
snan                   = 'snan';
inf                    = 'inf';
digit_0_f              = '0' | digit_1_f;
digit_1_f              = digit_1_9 | 'a'~'f' | 'A'~'F';
digit_0_9              = '0' | digit_1_9;
digit_1_9              = '1'~'9';
WSC                    = WS | comment;
WS                     = HT | SP | LINE_END | ',';
LINE_END               = CR? & LF;
HT                     = '\[9]';
LF                     = '\[a]';
CR                     = '\[d]';
SP                     = '\[20]';
char_identifier_first  = unicode(L,N) | '_';
char_identifier_next   = unicode(Cf,L,M,N) | '_' | '.' | '-';
char                   = unicode(C,L,M,N,P,S,Z);
```



License
-------

Copyright (c) 2025 Karl Stenerud. All rights reserved.

Distributed under the [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/legalcode) ([license deed](https://creativecommons.org/licenses/by/4.0)).
