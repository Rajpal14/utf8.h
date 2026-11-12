# Course Contribution Evidence

**Student:** Rajpal  
**Roll / Username:** [Rajpal14](https://github.com/Rajpal14)  
**Institution:** Indian Institute of Technology Gandhinagar (IIT Gandhinagar)  
**Course:** CS202 - Systems / Open Source Contribution  
**Evaluation Weight:** 10% Course Grade  
**Date of Submission:** November 12, 2026  

---

## 1. Upstream Repository Details

- **Repository:** [`sheredom/utf8.h`](https://github.com/sheredom/utf8.h)
- **Repository Description:** Single header utf8 string functions for C and C++
- **License:** Unlicense (Public Domain)
- **Star Count:** 2,004+ stars
- **Forks:** 148+ forks
- **Primary Language:** C / C++
- **Recent Activity:** Actively maintained with recent commits in 2025/2026 improving ASCII fast paths, warning fixes, and Unicode compliance.

---

## 2. Issue Addressed

- **Issue Title:** Missing Unicode blocks
- **Issue URL:** https://github.com/sheredom/utf8.h/issues/131
- **Issue Number:** #131
- **Issue Scope:** Added missing case-folding/mapping rules for Latin Extended-C, Latin Extended-D, Cyrillic Supplement, and Cyrillic Extended-C scripts.

---

## 3. Contribution & Pull Request Evidence

- **Contributor GitHub Account:** [https://github.com/Rajpal14](https://github.com/Rajpal14)
- **Personal Public Fork:** [https://github.com/Rajpal14/utf8.h](https://github.com/Rajpal14/utf8.h)
- **Feature Branch:** `add-missing-unicode-case-mappings`
- **Commit Hash:** `573dc05568bec81f78cbb7e266260b556c0a5cfb`
- **Pull Request URL:** https://github.com/sheredom/utf8.h/pull/158
- **Pull Request Number:** #158
- **PR Status:** Open / Ready for Maintainer Review
- **Base Branch:** `sheredom:main`
- **Head Branch:** `Rajpal14:add-missing-unicode-case-mappings`

---

## 4. Verification & Testing Evidence

- **Total Test Suites Executed:** 8
  1. `utf8_test` (Passed)
  2. `utf8_no_malloc_test` (Passed)
  3. `utf8_address_test` [AddressSanitizer] (Passed)
  4. `utf8_no_malloc_address_test` [AddressSanitizer] (Passed)
  5. `utf8_undefined_test` [UndefinedBehaviorSanitizer] (Passed)
  6. `utf8_no_malloc_undefined_test` [UndefinedBehaviorSanitizer] (Passed)
  7. `utf8_thread_test` [ThreadSanitizer] (Passed)
  8. `utf8_no_malloc_thread_test` [ThreadSanitizer] (Passed)
- **Test Pass Rate:** 100% (8/8 test suites, 829 tests each)
- **Sanitizer Errors:** 0
