# Pull Request Submission Details

**Upstream Repository:** [sheredom/utf8.h](https://github.com/sheredom/utf8.h)  
**Pull Request Link:** https://github.com/sheredom/utf8.h/pull/158  
**Pull Request Number:** #158  
**Target Branch:** `main`  
**Head Branch:** `Rajpal14:add-missing-unicode-case-mappings`  
**Author:** Rajpal ([Rajpal14](https://github.com/Rajpal14))  

---

## Pull Request Title
```
Add Unicode case mappings for Cyrillic Supplement, Latin Extended-C/D, and Cyrillic Extended-C
```

---

## Pull Request Body
```markdown
## Summary

This PR adds missing Unicode uppercase and lowercase case mappings to `utf8lwrcodepoint` and `utf8uprcodepoint` (resolving #131).

### Added Unicode Blocks & Ranges:
1. **Cyrillic Supplement (U+0500..U+052F)**:
   - Alternating upper/lower pairs (even = uppercase, odd = lowercase).
   - Handled via offset/parity checks in `utf8lwrcodepoint` and `utf8uprcodepoint`.
2. **Latin Extended-C (U+2C60..U+2C76)**:
   - Handled even=upper pairs (`U+2C60`, `U+2C72`).
   - Handled odd=upper pairs (`U+2C67`, `U+2C69`, `U+2C6B`, `U+2C75`).
   - Individual mappings for non-contiguous pairs (`U+2C62`->`U+026B`, `U+2C63`->`U+1D7D`, `U+2C64`->`U+027D`, `U+2C6D`->`U+0251`, `U+2C6E`->`U+0271`, `U+2C6F`->`U+0250`, `U+2C70`->`U+0252`).
3. **Latin Extended-D (U+A722..U+A7C3)**:
   - Added alternating pairs (`U+A722..U+A72F`, `U+A732..U+A76F`, `U+A77E..U+A787`, `U+A790..U+A793`, `U+A796..U+A7A9`, `U+A7B4..U+A7C3`).
   - Added odd=upper pairs (`U+A779`, `U+A77B`, `U+A78B`).
   - Individual pairs (`U+A77D`->`U+1D79`, `U+A78D`->`U+0265`, `U+A7AA`..`U+A7AE`, `U+A7B0`..`U+A7B3`).
4. **Cyrillic Extended-C (U+1C80..U+1C88)**:
   - Historic lowercase Cyrillic forms mapped to standard Cyrillic uppercase letters (`U+1C80`->`U+0412`, `U+1C81`->`U+0414`, `U+1C82`->`U+041E`, `U+1C83`->`U+0421`, `U+1C84`/`U+1C85`->`U+0422`, `U+1C86`->`U+042A`, `U+1C87`->`U+0462`, `U+1C88`->`U+A64A`).

### Testing:
- Added comprehensive unit tests in `test/test_shared.h`:
  - `utf8lwrcodepoint` and `utf8uprcodepoint` tests across all newly supported ranges.
  - String-level tests for `utf8lwr`, `utf8upr`, and `utf8casecmp`.
- Verified 100% pass across all C and C++ test suites with AddressSanitizer, UndefinedBehaviorSanitizer, and ThreadSanitizer.

Closes #131
```
