# Test Execution & Verification Results

**Contributor:** Rajpal ([Rajpal14](https://github.com/Rajpal14))  
**Repository:** [sheredom/utf8.h](https://github.com/sheredom/utf8.h)  
**Pull Request:** [#158](https://github.com/sheredom/utf8.h/pull/158)  
**Date of Execution:** November 12, 2026  

---

## 1. Build Environment

- **Operating System:** macOS (Darwin arm64)
- **Compiler:** Apple Clang / LLVM (cc / c++)
- **Build System:** CMake 4.4.5 & GNU Make
- **Architecture:** ARM64 (Apple Silicon)

---

## 2. Compilation and Build Commands

```bash
# Create and navigate to build directory
mkdir -p build_test && cd build_test

# Configure CMake with Debug profile and full sanitizer probing
cmake ../test -DCMAKE_BUILD_TYPE=Debug

# Compile all 8 test targets using all CPU cores
make -j$(sysctl -n hw.ncpu)
```

### Compiler Output Summary:
- Zero errors.
- All 8 targets compiled and linked successfully:
  - `utf8_test`
  - `utf8_no_malloc_test`
  - `utf8_address_test`
  - `utf8_no_malloc_address_test`
  - `utf8_undefined_test`
  - `utf8_no_malloc_undefined_test`
  - `utf8_thread_test`
  - `utf8_no_malloc_thread_test`

---

## 3. Test Suite Execution (`ctest --output-on-failure`)

```
Test project /tmp/rajpal_utf8/build_test
    Start 1: utf8_test
1/8 Test #1: utf8_test ........................   Passed    0.45 sec
    Start 2: utf8_no_malloc_test
2/8 Test #2: utf8_no_malloc_test ..............   Passed    0.47 sec
    Start 3: utf8_address_test
3/8 Test #3: utf8_address_test ................   Passed    0.64 sec
    Start 4: utf8_no_malloc_address_test
4/8 Test #4: utf8_no_malloc_address_test ......   Passed    0.46 sec
    Start 5: utf8_undefined_test
5/8 Test #5: utf8_undefined_test ..............   Passed    0.52 sec
    Start 6: utf8_no_malloc_undefined_test
6/8 Test #6: utf8_no_malloc_undefined_test ....   Passed    0.48 sec
    Start 7: utf8_thread_test
7/8 Test #7: utf8_thread_test .................   Passed    0.77 sec
    Start 8: utf8_no_malloc_thread_test
8/8 Test #8: utf8_no_malloc_thread_test .......   Passed    0.55 sec

100% tests passed out of 8

Total Test time (real) =   4.35 sec
```

---

## 4. Specific Regression Tests Added for Issue #131

### A. Cyrillic Supplement Tests:
- `c.utf8lwrcodepoint_cyrillic_supplement`: Iterates through all even code points in `U+0500..U+052E` verifying lowercase transformation (`+1`), and verifies odd code points remain unchanged.
- `c.utf8uprcodepoint_cyrillic_supplement`: Iterates through all odd code points in `U+0501..U+052F` verifying uppercase transformation (`-1`), and verifies even code points remain unchanged.
- `c.utf8casecmp_cyrillic_supplement`: Compares `\xd4\x80` (`U+0500` Ԁ) against `\xd4\x81` (`U+0501` ԁ) and `\xd4\xa6` against `\xd4\xa7`. Result: `0` (Equal).
- `c.utf8lwr_cyrillic_supplement_string`: Verifies in-place lowercase transformation of string `\xd4\x80\xd4\x82\xd4\xa6`.
- `c.utf8upr_cyrillic_supplement_string`: Verifies in-place uppercase transformation of string `\xd4\x81\xd4\x83\xd4\xa7`.

### B. Latin Extended-C Tests:
- `c.utf8lwrcodepoint_latin_extended_c`: Verifies:
  - `U+2C60` -> `U+2C61`
  - `U+2C62` -> `U+026B`
  - `U+2C63` -> `U+1D7D`
  - `U+2C64` -> `U+027D`
  - `U+2C67` -> `U+2C68` (odd-upper parity)
  - `U+2C69` -> `U+2C6A`
  - `U+2C6B` -> `U+2C6C`
  - `U+2C6D` -> `U+0251`
  - `U+2C6E` -> `U+0271`
  - `U+2C6F` -> `U+0250`
  - `U+2C70` -> `U+0252`
  - `U+2C72` -> `U+2C73`
  - `U+2C75` -> `U+2C76`
- `c.utf8uprcodepoint_latin_extended_c`: Verifies exact inverse mappings.

### C. Latin Extended-D Tests:
- `c.utf8lwrcodepoint_latin_extended_d`: Verifies ranges `U+A722..U+A72F`, `U+A732..U+A76F`, `U+A779..U+A77C`, `U+A77E..U+A787`, `U+A790..U+A793`, `U+A796..U+A7A9`, `U+A7B4..U+A7C3` along with irregular mappings (`U+A77D`->`U+1D79`, `U+A78D`->`U+0265`, `U+A7AA`..`U+A7AE`, `U+A7B0`..`U+A7B3`).
- `c.utf8uprcodepoint_latin_extended_d`: Verifies inverse uppercase mappings.
- `c.utf8casecmp_latin_extended_d`: Verifies case-insensitive comparison for Latin Extended-D characters (`\xea\x9c\xa2` vs `\xea\x9c\xa3`).

### D. Cyrillic Extended-C Tests:
- `c.utf8uprcodepoint_cyrillic_extended_c`: Verifies mapping of historic lowercase forms `U+1C80..U+1C88` to canonical uppercase Cyrillic letters (`U+0412`, `U+0414`, `U+041E`, `U+0421`, `U+0422`, `U+042A`, `U+0462`, `U+A64A`).
- `c.utf8lwrcodepoint_cyrillic_extended_c`: Verifies that `U+1C80..U+1C86` are identified as already lowercase and returned unchanged.

---

## 5. Sanitizer Results

| Sanitizer | Target Executed | Status | Findings |
|:---|:---|:---:|:---|
| **AddressSanitizer (ASan)** | `utf8_address_test` | **PASS** | 0 memory leaks, 0 buffer overflows |
| **ASan (no malloc)** | `utf8_no_malloc_address_test` | **PASS** | 0 memory violations |
| **UndefinedBehaviorSanitizer (UBSan)** | `utf8_undefined_test` | **PASS** | 0 integer overflows, 0 UB errors |
| **UBSan (no malloc)** | `utf8_no_malloc_undefined_test` | **PASS** | 0 UB errors |
| **ThreadSanitizer (TSan)** | `utf8_thread_test` | **PASS** | 0 data races, 0 deadlocks |
| **TSan (no malloc)** | `utf8_no_malloc_thread_test` | **PASS** | 0 concurrency issues |

---

## 6. Remaining Limitations

- **Context-Sensitive / Locale Casing:** Like the rest of `utf8.h`, casing is simple 1-to-1 codepoint mapping and does not implement locale-tailored casing (such as Turkish dotted/dotless 'i') or multi-character expansions (German `ß` -> `SS`), which is by design for a lightweight single-header C library.
