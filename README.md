Object Representation in Binary / Text
======================================

ORB and ORT are extensions of [BONJSON](https://github.com/kstenerud/bonjson) and [JSON](https://www.rfc-editor.org/rfc/rfc8259) that support more data types and fix some annoyances with JSON:

 * Floating point NaN and infinity values are allowed
 * Timestamp type
 * Identifier (UUID) type
 * Typed arrays (including a byte array type)
 * Comments
 * Hexadecimal notation
 * The comma character (`,`) is now considered "whitespace"
 * A new Unicode escape sequence that supports the entire codepoint range



Specifications
--------------

 * [ORB: Object Representation in Binary](orb.md)
 * [ORT: Object Representation in Text](ort.md)



License
-------

Copyright (c) 2025 Karl Stenerud. All rights reserved.

Distributed under the [Creative Commons Attribution License](https://creativecommons.org/licenses/by/4.0/legalcode) ([license deed](https://creativecommons.org/licenses/by/4.0)).
