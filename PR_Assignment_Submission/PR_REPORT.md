# Open-Source Contribution Report

**Student Name:** Rajpal  
**GitHub Username:** [Rajpal14](https://github.com/Rajpal14)  
**Course:** CS202 - Software Development / Systems  
**Institution:** Indian Institute of Technology Gandhinagar (IIT Gandhinagar)  
**Upstream Repository:** [sheredom/utf8.h](https://github.com/sheredom/utf8.h)  
**Pull Request:** [#158 - Add Unicode case mappings for Cyrillic Supplement, Latin Extended-C/D, and Cyrillic Extended-C](https://github.com/sheredom/utf8.h/pull/158)  
**Issue Resolved:** [#131 - Missing Unicode blocks](https://github.com/sheredom/utf8.h/issues/131)  
**Commit Hash:** `573dc05568bec81f78cbb7e266260b556c0a5cfb`  
**Submission Date:** November 12, 2026  

---

## 1. Project Overview

`utf8.h` is a renowned, widely used single-header C/C++ library designed for processing UTF-8 encoded strings safely and efficiently. It is released into the public domain (Unlicense) and utilized by game engines, systems software, and desktop applications for lightweight Unicode handling without external dependencies like ICU.

Key functionalities include string validation, case conversion (`utf8lwr`, `utf8upr`), case-insensitive comparison (`utf8casecmp`, `utf8ncasecmp`), substring search (`utf8casestr`), and bounded codepoint iteration.

---

## 2. Original Issue Description

**Issue #131: Missing Unicode blocks**
- The library supports case-mapping functions `utf8lwrcodepoint` and `utf8uprcodepoint`.
- While basic Latin, Greek, basic Cyrillic, Armenian, and Cherokee had partial or complete coverage, several significant Unicode blocks containing bicameral letters (with distinct uppercase and lowercase variants) were completely missing:
  1. **Cyrillic Supplement (`U+0500..U+052F`)**: Used for minority Uralic, Caucasian, and Siberian languages (e.g. Komi, Khanty, Chukchi).
  2. **Latin Extended-C (`U+2C60..U+2C76`)**: Contains phonetic and orthographic Latin additions.
  3. **Latin Extended-D (`U+A722..U+A7C3`)**: Contains medievalist, Mayanist, and African linguistic Latin letters.
  4. **Cyrillic Extended-C (`U+1C80..U+1C88`)**: Historic Old Church Slavonic Cyrillic lowercase forms.

Due to their absence, calling `utf8lwr`, `utf8upr`, or `utf8casecmp` on strings containing these characters failed to convert or match them properly.

---

## 3. Root Cause Analysis

In `utf8.h`, case mapping is performed via two pure functions:
- `utf8lwrcodepoint(utf8_int32_t cp)`
- `utf8uprcodepoint(utf8_int32_t cp)`

The implementation uses branch-efficient range checks and bitwise operations (`cp |= 0x1` for even-uppercase/odd-lowercase alternating pairs) and `switch` statements for irregular mappings.

The root cause of Issue #131:
1. **Missing Range Rules:** The range conditions in `utf8lwrcodepoint` and `utf8uprcodepoint` lacked checks for `0x0500..0x052E` (Cyrillic Supplement) and the alternating subsets of Latin Extended-C and Latin Extended-D.
2. **Asymmetric/Non-trivial Parity:** In Latin Extended-C and Latin Extended-D, certain pairs do not follow the standard even=upper/odd=lower convention (e.g., `U+2C67` is odd uppercase, mapping to `U+2C68` even lowercase). These require dedicated switch-case mapping rather than naive bitwise masking.
3. **Historic Cyrillic Forms:** Cyrillic Extended-C contains lowercase variants (`U+1C80..U+1C88`) that map to uppercase standard Cyrillic (`U+0412`, `U+0414`, etc.), with no inverse lowercase mapping from standard Cyrillic. These required explicit mappings in `utf8uprcodepoint` while remaining unchanged in `utf8lwrcodepoint`.

---

## 4. Implementation Details

### Files Modified:
- `utf8.h`: Added range conditions and switch cases to `utf8lwrcodepoint` and `utf8uprcodepoint`.
- `test/test_shared.h`: Added comprehensive unit tests covering individual codepoint transformations and string-level functions (`utf8lwr`, `utf8upr`, `utf8casecmp`).

### Changes in `utf8.h`:
1. **`utf8lwrcodepoint`**:
   - Added ranges for even=upper pairs: `0x0500..0x052E` (Cyrillic Supplement), `0x2C60..0x2C61`, `0x2C72..0x2C73`, `0xA722..0xA72F`, `0xA732..0xA76F`, `0xA77E..0xA787`, `0xA790..0xA793`, `0xA796..0xA7A9`, `0xA7B4..0xA7C3`.
   - Added switch cases for odd=upper pairs: `U+2C67`->`U+2C68`, `U+2C69`->`U+2C6A`, `U+2C6B`->`U+2C6C`, `U+2C75`->`U+2C76`, `U+A779`->`U+A77A`, `U+A77B`->`U+A77C`, `U+A78B`->`U+A78C`.
   - Added switch cases for irregular Latin Extended-C/D mappings: `U+2C62`->`U+026B`, `U+2C63`->`U+1D7D`, `U+2C64`->`U+027D`, `U+2C6D`->`U+0251`, `U+2C6E`->`U+0271`, `U+2C6F`->`U+0250`, `U+2C70`->`U+0252`, `U+A77D`->`U+1D79`, `U+A78D`->`U+0265`, `U+A7AA`..`U+A7AE`, `U+A7B0`..`U+A7B3`.
2. **`utf8uprcodepoint`**:
   - Added ranges for odd=lower pairs: `0x0501..0x052F` (Cyrillic Supplement), `0x2C73`, `0xA722..0xA72F`, `0xA732..0xA76F`, `0xA77E..0xA787`, `0xA790..0xA793`, `0xA796..0xA7A9`, `0xA7B4..0xA7C3`.
   - Added switch cases for even=lower pairs: `U+2C68`->`U+2C67`, `U+2C6A`->`U+2C69`, `U+2C6C`->`U+2C6B`, `U+2C76`->`U+2C75`, `U+A77A`->`U+A779`, `U+A77C`->`U+A77B`, `U+A78C`->`U+A78B`.
   - Added reverse mappings for irregular pairs.
   - Added uppercase mappings for Cyrillic Extended-C (`U+1C80`->`U+0412`, `U+1C81`->`U+0414`, `U+1C82`->`U+041E`, `U+1C83`->`U+0421`, `U+1C84`/`U+1C85`->`U+0422`, `U+1C86`->`U+042A`, `U+1C87`->`U+0462`, `U+1C88`->`U+A64A`).

---

## 5. Test Suite & Verification

The changes were validated against all 8 project test configurations:
1. `utf8_test`: Main test suite (C99, C11, C++98, C++11, C++14, C++17, C++20).
2. `utf8_no_malloc_test`: Verification in memory-constrained / embedded environment.
3. `utf8_address_test`: AddressSanitizer (ASan) memory safety verification.
4. `utf8_no_malloc_address_test`: ASan with no-malloc build.
5. `utf8_undefined_test`: UndefinedBehaviorSanitizer (UBSan) safety verification.
6. `utf8_no_malloc_undefined_test`: UBSan with no-malloc build.
7. `utf8_thread_test`: ThreadSanitizer (TSan) concurrency verification.
8. `utf8_no_malloc_thread_test`: TSan with no-malloc build.

**Result: 100% tests passed (8/8 targets, 829 test cases per test target).**

---

## 6. Learning Outcomes

1. Deep understanding of the Unicode Standard, code points, UTF-8 variable-length byte representation, and casing parity.
2. Experience working with multi-standard C/C++ header-only libraries maintaining compatibility from C89 through C++20.
3. Proficiency with memory sanitizers (ASan, UBSan, TSan) and defensive systems programming in C.
4. Mastery of professional Git workflows: fork management, feature branching, clean atomic commits, upstream PR lifecycle.
