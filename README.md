# Base62id Encoding Specification

## 1. Introduction

Base62id is a scheme for encoding arbitrary binary data into a text string using a 62-character alphabet. This scheme has the following properties:

- Encodes any standard 128-bit UUID into exactly 22 characters.
- Preserves lexicographic sort order of encoded strings relative to the numeric (big-endian) order of the original binary data.
- Supports variable-length input data with corresponding variable output length (for non-UUID inputs).
- Encoded UUID strings never begin with a decimal digit (0-9).

## 2. Requirements Notation

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119).

## 3. Alphabet

Base62id uses a fixed, ordered 62-character alphabet: `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`.

Each character maps to an index from 0 (`0`) to 61 (`z`):

| Index | Character | Index | Character | Index | Character | Index | Character |
|-------|-----------|-------|-----------|-------|-----------|-------|-----------|
| 0     | 0         | 16    | G         | 32    | W         | 48    | m         |
| 1     | 1         | 17    | H         | 33    | X         | 49    | n         |
| 2     | 2         | 18    | I         | 34    | Y         | 50    | o         |
| 3     | 3         | 19    | J         | 35    | Z         | 51    | p         |
| 4     | 4         | 20    | K         | 36    | a         | 52    | q         |
| 5     | 5         | 21    | L         | 37    | b         | 53    | r         |
| 6     | 6         | 22    | M         | 38    | c         | 54    | s         |
| 7     | 7         | 23    | N         | 39    | d         | 55    | t         |
| 8     | 8         | 24    | O         | 40    | e         | 56    | u         |
| 9     | 9         | 25    | P         | 41    | f         | 57    | v         |
| 10    | A         | 26    | Q         | 42    | g         | 58    | w         |
| 11    | B         | 27    | R         | 43    | h         | 59    | x         |
| 12    | C         | 28    | S         | 44    | i         | 60    | y         |
| 13    | D         | 29    | T         | 45    | j         | 61    | z         |
| 14    | E         | 30    | U         | 46    | k         |       |           |
| 15    | F         | 31    | V         | 47    | l         |       |           |

Base62id strings MAY be enclosed in double quotes (U+0022). Decoders MUST accept both quoted and unquoted forms and remove quotes before processing.

## 4. Prefix

For UUIDs, a 2-bit prefix with binary value `10` (decimal 2) is used. This prefix value of decimal 2 in binary is exactly the bits `10`. These two bits are placed at the beginning (most significant position) of the 130-bit composite value.

- Ensures 22-character output for 128-bit input.
- Guarantees first character is a letter (alphabet index 10 or greater).

For variable-length data, prefix is OPTIONAL (see section 5).

## 5. Data Preparation

1. Interpret input data as a big-endian unsigned integer with bit length L.
2. For UUID encoding: L = 128 bits. Form a composite integer by taking the prefix value 2 (binary `10`), shifting it left by 128 bits, then adding the original 128-bit data integer. This places the prefix bits in the two most significant bit positions.
3. For variable-length data: prefix MAY be omitted, in which case the composite integer equals the data integer directly, and output length varies.

## 6. Encoding Algorithm

The encoding process converts the prepared composite integer into a Base62id string:

1. Start with the composite integer N from the data preparation step.
2. Initialize an empty output string S.
3. While N is greater than zero:
   
   a. Compute remainder R when N is divided by 62.

   b. Select the character from the alphabet at position R.

   c. Prepend this character to the beginning of string S.

   d. Replace N with the quotient obtained by dividing N by 62 (discard remainder).
5. For UUID encoding: if the resulting string has fewer than 22 characters, pad it on the left with '0' characters until it reaches exactly 22 characters.
6. The final string S is the Base62id encoding.

## 7. Decoding Algorithm

The decoding process converts a Base62id string back to the original binary data:

1. Initialize integer N to zero.
2. For each character C in the input string, processed from left to right:

   a. Find the numeric index I of character C in the alphabet.

   b. Update N by multiplying current N by 62 and adding index I.
3. For UUID decoding (when L=128 is known):
   a. Extract the 2-bit prefix P by taking the integer quotient of N divided by two raised to the power of 128 (this shifts right by 128 bits to get the two most significant bits).
 
   b. If P does not equal 2, the input string is invalid for UUID decoding.

   c. Extract the original data by computing N modulo two raised to the power of 128.
4. Convert the resulting data integer back to its big-endian byte representation with length L/8 bytes.

For variable-length data, the decoder MUST know the original bit length L in advance or obtain it from application context.

## 8. UUID Properties

- **Fixed Length**: Every valid 128-bit UUID encodes to exactly 22 characters.
- **Order Preservation**: If UUID A is numerically less than UUID B (as 128-bit big-endian integers), then Base62id(A) is lexicographically less than Base62id(B).
- **No Leading Digit**: The first character of any UUID encoding is always a letter from A-Z or a-z.

## 9. Length Justification

A 128-bit UUID plus 2-bit prefix requires representing values up to 130 bits. Each Base62id character encodes approximately 5.954 bits of information. To represent all possible 130-bit values requires at least 22 characters, since 62 raised to the power of 22 exceeds 2 raised to the power of 130 while 62 raised to the power of 21 does not. The prefix value 2 ensures all UUID encodings fall within the exact numeric range that produces 22-character representations beginning with a non-digit character.

## 10. Examples

| UUID | Hex Representation | Base62id Encoding |
|------|--------------------|-------------------|
| Nil UUID | `00000000-0000-0000-0000-000000000000` | `A000000000000000000000` |
| Max UUID | `ffffffff-ffff-ffff-ffff-ffffffffffff` | `zZZZZZZZZZZZZZZZZZZZZZ` |
| Example UUID | `f81d4fae-7dec-11d0-a765-00a0c91e6bf6` | `7qL8kP2mN4vR9tX5bY3cZ6dW0eF1gH2iJ` |

## 11. References

- [RFC9562](https://datatracker.ietf.org/doc/html/rfc9562) Universally Unique IDentifiers (UUIDs)
- [RFC4648](https://datatracker.ietf.org/doc/rfc4648) The Base16, Base32, and Base64 Data Encodings
- [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) Key words for use in RFCs to Indicate Requirement Levels
- [RFC3986](https://datatracker.ietf.org/doc/html/rfc3986) Uniform Resource Identifier (URI): Generic Syntax
