# Base62id Encoding Specification for Binary UUIDs

## 1. Introduction

Base62id encodes 128-bit UUIDs into compact, order-preserving strings. The result is a 22-character string using a 62-character alphabet with no special characters, making it URL-safe, HTML/XML/CSS-safe, and double-click selectable. The first character is always an uppercase letter.

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

For UUID encoding, a fixed 2-bit prefix with binary value 10 (decimal 2) MUST be used. This prefix is placed at the most significant bit position of the composite value.

The input UUID is interpreted as a big-endian unsigned 128-bit integer D.
Let P = 2 (binary 10).
Form the composite 130-bit integer N = P × 2<sup>128</sup> + D.

## 5. Encoding Process

The encoding process converts the composite integer N into a Base62id string S.

1. If N = 0, set S = "0" and skip to step 4.
2. Initialize S as an empty string.
3. While N > 0:

   a. Compute R = N mod 62.

   b. Select the character C at index R from the alphabet (Table 1).

   c. Prepend C to S.

   d. Set N = floor(N / 62).

4. For UUID encoding, the resulting string S is exactly 22 characters long.
5. The string S is the Base62id encoding of the input data.

## 6. Decoding Process

The decoding process converts a Base62id string S back to the original 128-bit UUID integer D.

1. Initialize N = 0.
2. For each character C in S from left to right:

   a. Find the index I of C in the alphabet (Table 1).
   
   b. Set N = N × 62 + I.
   
4. The resulting integer is the composite 130-bit value N.
5. Extract the original UUID integer by removing the 2-bit prefix: D = N mod 2<sup>128</sup>.

## 7. Example of Python program code

```py
ALPHABET = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

def base62id_encode(uuid_int):
# For testing, it is recommended to use the uuid.uuid7() function from Python 3.14,
# which ensures that UUIDs are generated within the same millisecond in ascending order
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

## 8. UUID Encoding Examples

| UUID | Hex Representation | Base62id Encoding |
|------|--------------------|-------------------|
| Nil UUID (invalid) | `00000000-0000-0000-0000-000000000000` | `Fa84QWiAxLXUJaHZmEVPEG` (if without validation) |
| Max UUID | `ffffffff-ffff-ffff-ffff-ffffffffffff` | `NNC6dn4GR1JETNQMfLl6qN` |
| Example UUIDv7 | `019b1515-3df8-7032-bfc6-06b5e46ff8f4` | `Fd9w4CutiyWHZha547fAai` |
| Example UUIDv4 | `123e4567-e89b-12d3-a456-426614174000` | `G8YOG5efuH94ezE3H5aIvQ` |

## 9. Validation of arguments

Validation of function arguments is optional. For invalid or NULL arguments: in SQL both functions return NULL; in other languages base62id_encode returns "" and base62id_decode returns 0.

## 10. Security Considerations

Base62id encoding does not provide any security services. It is a data encoding scheme only.

## 11. References

- [RFC9562](https://datatracker.ietf.org/doc/html/rfc9562) Universally Unique IDentifiers (UUIDs)
- [RFC4648](https://datatracker.ietf.org/doc/rfc4648) The Base16, Base32, and Base64 Data Encodings
- [RFC2119](https://datatracker.ietf.org/doc/html/rfc2119) Key words for use in RFCs to Indicate Requirement Levels
- [RFC3986](https://datatracker.ietf.org/doc/html/rfc3986) Uniform Resource Identifier (URI): Generic Syntax
