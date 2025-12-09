# Base62id Encoding Specification

## 1. Overview

Base62id is a scheme for encoding arbitrary binary data into a text string using a sixty-two-character alphabet. This scheme is designed with the following specific properties:
1.  It preserves the lexicographic sort order of encoded strings relative to the numeric (big-endian) order of the original binary data.
2.  It encodes any standard 128-bit UUID into a string of exactly twenty-two characters.
3.  The encoded string is guaranteed never to begin with a decimal digit (0-9).
4.  It can encode binary input of variable length, with the output length being variable for non-UUID inputs.

## 2. Alphabet

The encoding uses a fixed, ordered alphabet of 62 characters, corresponding to their sequence in the ASCII table:
The ten decimal digits: 0, 1, 2, 3, 4, 5, 6, 7, 8, 9.
The twenty-six uppercase Latin letters: A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z.
The twenty-six lowercase Latin letters: a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z.

Each character is assigned a numeric index based on its position in this sequence, starting from 0 for the character '0' up to 61 for the character 'z'.

## 3. Prefix

To satisfy the fixed length and "no leading digit" requirements for UUIDs, a constant prefix is combined with the input data before the base conversion process.
*   **Length:** The prefix is 2 bits long.
*   **Value:** The prefix has a binary value of `10`, which is the integer number 2.
*   **Purpose:** This prefix ensures that the composite value (prefix + data) always results in a 22-character encoding for a 128-bit input and that the first character of the encoded string is a letter (index 10 or greater).

## 4. Data Preparation

Prior to encoding, the input binary data is prepared:
1.  The input data is interpreted as a single unsigned integer, with the first byte being the most significant (big-endian byte order). Let the bit length of this data integer be L.
2.  A new composite integer is formed by shifting the prefix value left by L bits and adding the data integer. Mathematically, this is: Composite Integer = (Prefix Value) × 2ᴸ + Data Integer, where 2ᴸ denotes two raised to the power of L.

## 5. Encoding Algorithm

The process to convert prepared binary data into a Base62id string is defined as follows:
1.  Let N be the composite integer value obtained from the Data Preparation step.
2.  Initialize an empty output string S.
3.  While the integer N is greater than zero, repeat the following steps:

    a. Calculate the remainder R of the division N ÷ 62.

    b. Find the character in the alphabet at the index equal to R.

    c. Prepend this character to the beginning of the string S.

    d. Replace N with the quotient of the integer division N ÷ 62.
4.  If the original input was a 128-bit UUID, the string S will contain exactly twenty-two characters. For other input lengths, the string length will vary accordingly.

## 6. Decoding Algorithm

The process to convert a Base62id string back into its original binary data is defined as follows:
1.  Let N be the integer value zero.
2.  For each character C in the input string, processed from left to right:
    a. Find the numeric index I of the character C in the alphabet.
    b. Update the integer value: N = (N × 62) + I.
3.  The resulting integer N represents the composite value (prefix + original data).
4.  Determine the bit length L of the original data. For a UUID, L = 128.
5.  Extract the 2-bit prefix value P by calculating the integer quotient of N ÷ 2ᴸ.
6.  Validate that the extracted prefix value P equals 2. If not, the input string is invalid for this specification.
7.  Extract the original data integer D by calculating D = N mod 2ᴸ, where mod is the modulo operation.
8.  Convert the integer D into its binary big-endian byte representation. The byte length is L ÷ 8.

## 7. Properties for UUID Encoding

*   **Fixed Length:** The encoding of any 128-bit UUID always results in a string of twenty-two characters.
*   **No Leading Digit:** The first character of any valid encoded UUID string is always a letter (A-Z or a-z).
*   **Order Preservation:** For any two UUIDs A and B, where A is numerically less than B when compared as 128-bit big-endian integers, the Base62id string for A will be lexicographically less than the string for B.

## 8. Length Justification
A standard UUID requires 128 bits of information. The 2-bit prefix adds an overhead of 2 bits, creating a composite value of 130 bits. The number of base-62 digits d required to represent a non-negative integer n is given by the smallest integer d such that 62ᵈ > n. For n = 2¹³⁰ - 1, the smallest integer d satisfying this inequality is 22. Therefore, twenty-two characters are necessary and sufficient to represent any 130-bit composite value. The specific prefix value of 2 ensures the composite integer always falls within the precise numeric range that maps to a 22-digit base-62 representation beginning with a non-zero digit in the alphabet.
