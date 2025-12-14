# Base62id Encoding Specification

## 1. Introduction

This specification defines the Base62id encoding scheme for representing binary data in ASCII string form using 62 characters. Base62id encoding is primarily designed for encoding UUIDs into compact, order-preserving strings.

## 2. Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119.

## 3. Alphabet

Base62id uses a fixed, ordered 62-character alphabet: `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`.

Each character maps to an index from 0 (`0`) to 61 (`z`):

| Index | Character | Index | Character | Index | Character | Index | Character |
|------:|-----------|------:|-----------|------:|-----------|------:|-----------|
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

For UUIDs, a 2-bit prefix with binary value `10` (decimal 2) MUST be used. This prefix is placed at the most significant bit position of the composite value.

For variable-length data, the use of a prefix is OPTIONAL.

## 5. Data Preparation

The input data is interpreted as a big-endian unsigned integer D of bit length L.

For UUID encoding, where L = 128:
   - Let P = 2 (binary `10`)
   - Form the composite integer N = P × 2^128 + D

For variable-length data:
   - If a prefix is used, form the composite integer N = P × 2^L + D
   - If no prefix is used, let N = D

Special case: if D = 0 and no prefix is used, then N = 0.

## 6. Encoding Process

The encoding process converts the composite integer N into a Base62id string S.

1. If N = 0, set S = "0" and skip to step 4.

2. Initialize S as an empty string.

3. While N > 0:

   a. Compute R = N mod 62.

   b. Select the character C at index R from the alphabet (Table 1).

   c. Prepend C to S.

   d. Set N = floor(N / 62).

4. For UUID encoding, the resulting string S will be exactly 22 characters long.

5. The string S is the Base62id encoding of the input data.

## 7. Decoding Process

The decoding process converts a Base62id string S back to the original binary data.

1. If S is empty, the decoded value is 0. Proceed to step 4.

2. Initialize N = 0. For each character C in S from left to right:

   a. Find the index I of C in the alphabet (Table 1).
 
   b. Set N = N × 62 + I.

3. The resulting integer is the composite integer N.

4. Extract the original data:

   a. Compute P = floor(N / 2^128).
 
   b. If P ≠ 2, the input string is invalid for UUID decoding.

   c. Compute D = N mod 2^128.

5. Convert the integer D to a big-endian byte string of length ceil(L/8) bytes.

## 8. UUID Properties

When encoding UUIDs with the prescribed prefix:

- The encoded string is exactly 22 characters in length.
- Lexicographic ordering of encoded strings corresponds to numeric ordering of the original UUIDs when compared as 128-bit big-endian integers.
- The first character of the encoded string is always a letter (A-Z or a-z), never a digit (0-9).

## 9. Length Considerations

The 130-bit composite value formed by a 128-bit UUID and a 2-bit prefix requires exactly 22 Base62id characters, as 62²¹ < 2¹³⁰ ≤ 62²².

The prefix value 2 ensures the first encoded character is always a letter. Mathematically:

- N = 2 × 2¹²⁸ + D, where 0 ≤ D ≤ 2¹²⁸ − 1
- Thus 2¹²⁹ ≤ N ≤ 2¹²⁹ + 2¹²⁸ − 1

The first character's index is floor(N / 62²¹). Calculating bounds:
- Minimum: floor(2¹²⁹ / 62²¹) ≈ floor(15.7) = 15
- Maximum: floor((2¹²⁹ + 2¹²⁸ − 1) / 62²¹) ≈ floor(23.6) = 23

Since 15 ≤ index ≤ 23, and alphabet indices 15–23 are uppercase letters F–N, the first character is guaranteed to be an uppercase letter.

## 10. Example of Python program code

```py
ALPHABET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

def base62id_encode(uuid_int):
    value = (0b10 << 128) | uuid_int  # add prefix
    chars = ["0"] * 22
    
    for i in range(21, -1, -1):
        chars[i] = ALPHABET[value % 62]
        value //= 62
    
    return "".join(chars)

def base62id_decode(encoded):
    value = 0
    for char in encoded:
        value = value * 62 + ALPHABET.index(char)
    
    return value & ((1 << 128) - 1)  # remove prefix
```

## 11. Data Examples

| UUID | Hex Representation | Base62id Encoding |
|------|--------------------|-------------------|
| Nil UUID | `00000000-0000-0000-0000-000000000000` | `Fa84QWiAxLXUJaHZmEVPEG` |
| Max UUID | `ffffffff-ffff-ffff-ffff-ffffffffffff` | `NNC6dn4GR1JETNQMfLl6qN` |
| Example UUIDv7 | `019b1515-3df8-7032-bfc6-06b5e46ff8f4` | `Fd9w4CutiyWHZha547fAai` |
| Example UUIDv4 | `123e4567-e89b-12d3-a456-426614174000` | `G8YOG5efuH94ezE3H5aIvQ` |

## 12. Security Considerations

Base62id encoding does not provide any security services. It is a data encoding scheme only.

## 13. References

- [RFC9562](https://datatracker.ietf.org/doc/html/rfc9562) Universally Unique IDentifiers (UUIDs)
- [RFC4648](https://datatracker.ietf.org/doc/rfc4648) The Base16, Base32, and Base64 Data Encodings
- [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) Key words for use in RFCs to Indicate Requirement Levels
- [RFC3986](https://datatracker.ietf.org/doc/html/rfc3986) Uniform Resource Identifier (URI): Generic Syntax
