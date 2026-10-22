## 1. Overview

`top_words(text: str) -> list[tuple[str, int]]` returns up to 10 most frequent words and their occurrence counts.

Processing order:

**Validation → NFC normalization → case folding → NFC normalization → tokenization → stopword filtering → counting → sorting → top 10.**

## 2. Input Validation

- Accept `str` and its subclasses.
- Raise `TypeError` for non-string inputs.
- Raise `ValueError` if the input contains lone surrogate code points (U+D800–U+DFFF).
- Return `[]` for empty input or when no tokens remain.
- Do not modify the input.
- Do not impose undocumented input-size limits.

Validation must precede text processing. Processing must not depend on overridden methods of `str` subclasses.

## 3. Normalization

Apply these operations in order:

1. Unicode NFC normalization.
2. Python's `str.casefold()`.
3. Unicode NFC normalization again.

Use the Unicode database provided by Python. Results may differ between Unicode database versions.

## 4. Tokenization

A letter is any Unicode character whose General Category starts with `L`. Combining marks (`M` categories) may follow a letter or other marks attached to that letter but cannot start a token.

The only permitted internal separators are:

- `-` (U+002D)
- `‐` (U+2010)
- `'` (U+0027)
- `’` (U+2019)

Rules:

- Tokens consist of letter groups, optionally joined by single internal separators.
- A separator is preserved only between letter groups.
- Consecutive separators terminate tokens and are discarded.
- Digits, whitespace, punctuation, symbols, and control characters are boundaries.
- Unattached combining marks are discarded and act as boundaries.
- Leading and trailing separators are discarded.
- Preserve the normalized, case-folded spelling of each token.

Examples:

| Input | Tokens |
|---|---|
| `state-of-the-art` | `state-of-the-art` |
| `hello--world` | `hello`, `world` |
| `can'’t` | `can`, `t` |
| `abc123def` | `abc`, `def` |
| `hello_world` | `hello`, `world` |
| `café` | `café` |

## 5. Stopword Filtering

Remove tokens matching any entry in the fixed, combined English, Russian, and Kazakh stopword lists defined in this specification.

The original three lists are unchanged and form the complete stopword set. Matching is exact and uses normalized, case-folded tokens.

Filtering is language-independent. No language detection, stemming, lemmatization, substring matching, or additional stopword removal is permitted.

## 6. Counting and Sorting

Count all remaining token occurrences and sort by:

1. Frequency, descending.
2. Token string, ascending Unicode code-point order.

Use ordinary Python string ordering, not locale-aware sorting.

Return the first 10 `(word, count)` tuples. Return fewer if fewer than 10 distinct tokens remain.

## 7. Reference Tests

```python
top_words("Cat dog cat. Собака кошка собака. Мысық ит мысық.")
# [('cat', 2), ('мысық', 2), ('собака', 2),
#  ('dog', 1), ('ит', 1), ('кошка', 1)]

top_words("The quick brown fox jumps over the lazy dog.")
# [('brown', 1), ('dog', 1), ('fox', 1), ('jumps', 1),
#  ('lazy', 1), ('over', 1), ('quick', 1)]

top_words("state-of-the-art AI model's performance")
# [('ai', 1), ("model's", 1), ('performance', 1),
#  ('state-of-the-art', 1)]

top_words("Straße STRASSE")
# [('strasse', 2)]

top_words("café cafe\u0301")
# [('café', 2)]

top_words("hello--world abc123def")
# [('abc', 1), ('def', 1), ('hello', 1), ('world', 1)]

top_words("banana apple cherry banana apple cherry")
# [('apple', 2), ('banana', 2), ('cherry', 2)]

top_words("")
# []

top_words("the and of мы и")
# []

top_words(123)
# Raises TypeError

top_words("\ud800")
# Raises ValueError
```

Additional tests must cover `str` subclasses, combining marks, mixed separators, non-ASCII tie-breaking, and inputs containing more than 10 distinct tokens.

## 8. Implementation Contract

The implementation must follow this specification exactly, use no locale-dependent behavior, preserve the input, and return no more than 10 results. Identical inputs must produce identical results under the same Python and Unicode database versions.