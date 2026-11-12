# Viva Preparation: UTF-8 & Unicode Case-Mapping

**Student:** Rajpal ([Rajpal14](https://github.com/Rajpal14))  
**Course:** CS202 - Systems / Software Engineering  
**Institution:** IIT Gandhinagar  
**Project:** `sheredom/utf8.h` - PR #158  

---

## Part 1: Core Theoretical Foundations

### 1. What is Unicode?
Unicode is a universal character encoding standard maintained by the Unicode Consortium. It assigns a unique numeric value (called a **Code Point**) to every character, glyph, and symbol across virtually all writing systems in the world, plus emojis and mathematical symbols. Unicode currently defines over 149,000 characters spanning 17 planes (0 to 16).

### 2. What is a Code Point?
A Code Point is an integer value assigned to a character in Unicode. It is traditionally written as `U+` followed by four to six hexadecimal digits:
- `U+0041` = 'A' (Latin Capital Letter A)
- `U+0500` = 'Ԁ' (Cyrillic Capital Letter Komi De)
- `U+1F600` = '😀' (Grinning Face)
The valid range of Unicode code points is `U+0000` to `U+10FFFF`.

### 3. What is UTF-8?
UTF-8 (8-bit Unicode Transformation Format) is a variable-width character encoding capable of encoding all $1{,}112{,}064$ valid character code points in Unicode using one to four 8-bit bytes. It is backward compatible with 7-bit ASCII and is the dominant encoding of the World Wide Web and modern Unix operating systems.

### 4. How UTF-8 Encodes 1, 2, 3, and 4-Byte Characters

| Codepoint Range (Hex) | UTF-8 Byte Format (Binary) | Total Bytes | Notes |
|:---|:---|:---:|:---|
| `U+0000` .. `U+007F` | `0xxxxxxx` | 1 byte | Standard 7-bit ASCII |
| `U+0080` .. `U+07FF` | `110xxxxx 10xxxxxx` | 2 bytes | Greek, Arabic, Cyrillic, Hebrew |
| `U+0800` .. `U+FFFF` | `1110xxxx 10xxxxxx 10xxxxxx` | 3 bytes | Most Asian scripts, Georgian, Armenian |
| `U+10000` .. `U+10FFFF` | `11110xxx 10xxxxxx 10xxxxxx 10xxxxxx` | 4 bytes | Emojis, historical scripts |

### 5. What are Continuation Bytes?
Every byte in a multi-byte UTF-8 sequence after the leading byte must be a continuation byte.
- Continuation bytes always have the binary pattern `10xxxxxx` (hex range `0x80` to `0xBF`).
- This prefix (`10` in bits 7 and 6) allows a parser to immediately distinguish leading bytes (`0xxxxxxx`, `110xxxxx`, `1110xxxx`, `11110xxx`) from continuation bytes, making UTF-8 **self-synchronizing**.

---

## Part 2: Issue #131 & Implementation Deep-Dive

### 6. What was Issue #131?
Issue #131 pointed out that `utf8.h` was missing case-mapping tables for several Unicode blocks that feature bicameral (upper/lower case) alphabetic characters, specifically:
- Cyrillic Supplement (`U+0500`–`U+052F`)
- Latin Extended-C (`U+2C60`–`U+2C76`)
- Latin Extended-D (`U+A722`–`U+A7C3`)
- Cyrillic Extended-C (`U+1C80`–`U+1C88`)

Without these mappings, `utf8lwrcodepoint`, `utf8uprcodepoint`, `utf8casecmp`, `utf8lwr`, and `utf8upr` left these characters unchanged, breaking case-insensitive search and text normalization for texts in those scripts.

### 7. How does Case-Mapping Work in `utf8.h`?
`utf8.h` implements two core pure functions:
```c
utf8_int32_t utf8lwrcodepoint(utf8_int32_t cp);
utf8_int32_t utf8uprcodepoint(utf8_int32_t cp);
```
These inspect the 32-bit integer code point `cp`. If `cp` falls in a range of uppercase letters with an arithmetic offset, it applies the offset or bitwise operation. Otherwise, special cases are handled via a `switch` statement.

### 8. Explain the Bitwise Parity Trick (`cp |= 0x1` and `cp &= ~0x1`)
In many Unicode alphabets (such as the Cyrillic Supplement `U+0500..U+052F`):
- All uppercase characters are at **even** code points (`U+0500`, `U+0502`, `U+0504`, ...)
- Their corresponding lowercase letters are at the immediately adjacent **odd** code point (`U+0501`, `U+0503`, `U+0505`, ...)
- To convert uppercase to lowercase: setting the lowest bit (`cp |= 0x1`) changes even $2k$ to $2k+1$. If `cp` is already odd ($2k+1$), it remains unchanged!
- To convert lowercase to uppercase: clearing the lowest bit (`cp &= ~0x1`) changes odd $2k+1$ to $2k$. If `cp` is already even ($2k$), it remains unchanged!

### 9. Why can't the Bitwise Trick be used for all blocks?
Some blocks violate the "even = upper, odd = lower" rule:
- In Latin Extended-C, `U+2C67` is **odd** but uppercase (Latin Capital Letter H with Descender), and its lowercase counterpart `U+2C68` is **even**.
- Naively running `cp |= 0x1` on `0x2C67` does nothing (`0x2C67 | 1 = 0x2C67`).
- Naively running `cp &= ~0x1` would give `0x2C66`, which is completely incorrect!
- Therefore, blocks with inverted or non-contiguous parity require explicit `switch` cases.

### 10. Why are Cyrillic Extended-C characters (`U+1C80..U+1C88`) special?
`U+1C80..U+1C88` are historic Old Church Slavonic lowercase forms (e.g., `U+1C80` Cyrillic Small Letter Rounded Ve).
- Their uppercase mapping maps to standard Cyrillic Capital Letters (e.g., `U+1C80` -> `U+0412` Capital Ve).
- However, standard Cyrillic Capital Ve (`U+0412`) downcases to standard Small Ve (`U+0432`), **not** `U+1C80`.
- Thus, `utf8lwrcodepoint(0x1C80)` must return `0x1C80` (it is already lowercase), while `utf8uprcodepoint(0x1C80)` returns `0x0412`. This is a non-bijective (one-way) mapping.

---

## Part 3: 35 Viva Questions and Answers

### Q1: What is the main design goal of `utf8.h`?
**A:** To provide a dependency-free, ANSI C/C++ compatible, single-header library for common UTF-8 string manipulation operations with zero dynamic memory allocation when possible.

### Q2: What license is `utf8.h` released under?
**A:** The Unlicense (Public Domain dedication).

### Q3: What is the maximum number of bytes a UTF-8 character can use according to RFC 3629?
**A:** 4 bytes. (Earlier specifications allowed up to 6 bytes, but RFC 3629 restricted UTF-8 to the valid Unicode code point range `U+0000` to `U+10FFFF`).

### Q4: Why is ASCII a proper subset of UTF-8?
**A:** For code points `U+0000` to `U+007F`, UTF-8 uses a single byte with bit 7 set to `0`. This is bit-for-bit identical to standard 7-bit ASCII.

### Q5: What is the significance of the leading byte in UTF-8?
**A:** The leading byte indicates both the start of a character and the total length of the sequence based on the number of leading `1` bits:
- `0xxxxxxx`: 1-byte sequence
- `110xxxxx`: 2-byte sequence
- `1110xxxx`: 3-byte sequence
- `11110xxx`: 4-byte sequence

### Q6: Can a continuation byte ever be confused with an ASCII byte in UTF-8?
**A:** No. Continuation bytes always begin with `10xxxxxx` (decimal 128–191), whereas ASCII bytes always begin with `0xxxxxxx` (decimal 0–127).

### Q7: What is case folding?
**A:** Case folding is the process of converting strings to a canonical form where case differences are eliminated, enabling case-insensitive comparisons regardless of language or script.

### Q8: What does `utf8casecmp` do?
**A:** It compares two UTF-8 strings lexicographically, ignoring case, by decoding each string code-point-by-code-point and converting them via `utf8lwrcodepoint`.

### Q9: What is the difference between `utf8len` and `strlen`?
**A:** `strlen` returns the number of raw bytes up to the null terminator. `utf8len` returns the number of Unicode code points (characters), which can be significantly fewer than the byte count in multibyte strings.

### Q10: What does `utf8nlen` do?
**A:** It counts the number of UTF-8 code points within at most `n` bytes of input, preventing buffer over-reads on untrusted or non-null-terminated buffers.

### Q11: What is an overlong UTF-8 sequence?
**A:** An invalid encoding where a code point is encoded using more bytes than necessary (e.g., encoding ASCII 'A' as `11000001 10000001` instead of `01000001`). UTF-8 security standards strictly reject overlong sequences.

### Q12: Why are surrogate code points (`U+D800..U+DFFF`) forbidden in UTF-8?
**A:** Surrogates are reserved exclusively for UTF-16 to encode characters above `U+FFFF`. In UTF-8, characters above `U+FFFF` are encoded directly as 4-byte sequences; encoding surrogate code points is invalid in UTF-8.

### Q13: What Unicode blocks did your contribution add?
**A:** Cyrillic Supplement (`U+0500..U+052F`), Latin Extended-C (`U+2C60..U+2C76`), Latin Extended-D (`U+A722..U+A7C3`), and Cyrillic Extended-C (`U+1C80..U+1C88`).

### Q14: In Cyrillic Supplement, which code points are uppercase?
**A:** Even code points (`U+0500`, `U+0502`, ..., `U+052E`).

### Q15: In Cyrillic Supplement, which code points are lowercase?
**A:** Odd code points (`U+0501`, `U+0503`, ..., `U+052F`).

### Q16: How does `utf8lwrcodepoint` transform `U+0500`?
**A:** `0x0500 | 0x1 = 0x0501` (Cyrillic Small Letter Komi De).

### Q17: How does `utf8uprcodepoint` transform `U+0501`?
**A:** `0x0501 & ~0x1 = 0x0500` (Cyrillic Capital Letter Komi De).

### Q18: What is `U+2C67` and why was it tricky?
**A:** `U+2C67` is Latin Capital Letter H with Descender. Its code point is odd (`0x2C67`), but its lowercase counterpart `U+2C68` is even (`0x2C68`), reversing the common parity.

### Q19: What is `utf8_constexpr14` macro in `utf8.h`?
**A:** A macro defined as `constexpr` in C++14 and later, allowing functions like `utf8lwrcodepoint` to be evaluated at compile-time when called with constant expressions.

### Q20: What is `utf8_int8_t` in C++20 vs C?
**A:** In C and earlier C++, `utf8_int8_t` is `char`. In C++20 (when `__cpp_char8_t` is defined), it is `char8_t`.

### Q21: Why were string literal tests placed inside `#if !defined(__cplusplus)` in `test_shared.h`?
**A:** In C++20, string literals like `"hello"` have type `const char[6]`, which cannot implicitly convert to `const char8_t*` without reinterpret casts. Placing them in C mode ensures standard string literal compatibility.

### Q22: What happens if `utf8lwrcodepoint` is passed an ASCII letter like 'A' (`0x0041`)?
**A:** It hits the ASCII check `((0x0041 <= cp) && (0x005a >= cp))` and adds 32, returning `0x0061` ('a').

### Q23: What happens if `utf8lwrcodepoint` is passed a character with no lowercase equivalent (e.g., a digit '5')?
**A:** It falls through all checks and the `default` switch branch, returning the code point unchanged (`0x0035`).

### Q24: What is AddressSanitizer (ASan)?
**A:** A compiler-instrumented tool that detects memory errors including out-of-bounds accesses, use-after-free, double-free, and stack buffer overflows at runtime.

### Q25: What is UndefinedBehaviorSanitizer (UBSan)?
**A:** A tool that catches undefined behavior such as integer overflow, null pointer dereference, misaligned memory access, and bit-shift out of range.

### Q26: What is ThreadSanitizer (TSan)?
**A:** A tool designed to detect data races and deadlocks in multi-threaded code.

### Q27: Did all tests pass with sanitizers enabled?
**A:** Yes, 100% of test suites passed with ASan, UBSan, and TSan enabled with zero warnings or errors.

### Q28: What is in-place string modification in `utf8lwr`?
**A:** `utf8lwr(str)` modifies the input buffer directly in place. Because every mapped uppercase/lowercase pair in the added blocks has the same UTF-8 byte length (e.g. both 2 bytes, or both 3 bytes), in-place modification never overflows or truncates the buffer.

### Q29: Do all Unicode lowercase mappings have the same byte length as their uppercase mappings?
**A:** In simple 1-to-1 case mapping (as used by `utf8.h`), yes for almost all characters. The German lowercase `ß` (sharp s) historically had uppercase `SS` (two characters) or capital `ẞ` (`U+1E9E`). `utf8.h` uses single-codepoint mapping for deterministic buffer safety.

### Q30: How is backwards compatibility maintained for older C compilers?
**A:** Functions use portable ANSI C types with conditional macro definitions for `constexpr`, `inline`, and attributes like `nonnull` and `pure`.

### Q31: How did you ensure your PR did not collide with other contributors' work?
**A:** Issue #131 addresses Unicode case mappings, completely distinct from issue #118 (which addresses four-byte sequence validation in `utf8makevalid`). The work was developed on an independent fork and feature branch.

### Q32: What git command was used to push the feature branch?
**A:** `git push origin add-missing-unicode-case-mappings`

### Q33: How does GitHub link a PR to an issue automatically?
**A:** By including keywords like `Closes #131` or `Resolves #131` in the PR description.

### Q34: What is a "pure" function in C compiler terminology (`__attribute__((pure))`)?
**A:** A function whose return value depends only on its parameters and global memory, with no side effects. This enables compiler optimizations like common subexpression elimination.

### Q35: What is the difference between `utf8cmp` and `utf8casecmp`?
**A:** `utf8cmp` performs a case-sensitive byte-by-byte unsigned comparison; `utf8casecmp` converts both strings code-point-by-code-point to lowercase before comparing, ignoring case differences.
