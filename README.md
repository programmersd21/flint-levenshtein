# levenshtein

A small, dependency-free Levenshtein distance and fuzzy-matching library written entirely in Flint.

No C extensions. No Rust backend. Just Flint source.

## Features

- Exact Levenshtein edit distance using two-row dynamic programming.
- Threshold-limited distance using a diagonal band.
- Normalized similarity scores.
- Nearest-candidate ranking with stable tie-breaking.
- No external dependencies.

## Requirements

Flint **0.13.2 or newer**.

## Install as a package

From a Flint project, add this repository as a dependency:

```sh
flint pkg add https://github.com/programmersd21/flint-levenshtein.git
```

Then import the module:

```flint
import levenshtein

print(levenshtein.distance("kitten", "sitting"))  # 3
print(levenshtein.similarity("flint", "flint"))    # 1
print(levenshtein.distance_bounded("kitten", "sitting", 2))  # 3
```

The exact `flint pkg add` source syntax depends on the URL form supported by your Flint installation; a local path also works:

```sh
flint pkg add ../flint-levenshtein
```

## API

### `distance(a, b)`

Returns the exact edit distance. An insertion, deletion, or substitution costs one.

```flint
levenshtein.distance("kitten", "sitting")  # 3
levenshtein.distance("", "flint")          # 5
levenshtein.distance("same", "same")       # 0
```

Time complexity is `O(m*n)`. Auxiliary space is `O(min(m, n))`.

### `distance_bounded(a, b, limit)`

Returns the exact distance when it is at most `limit`; otherwise returns `limit + 1`. A negative limit returns `-1`.

The thresholded implementation evaluates only the diagonal band that could still produce a result within the limit.

```flint
levenshtein.distance_bounded("kitten", "sitting", 3)  # 3
levenshtein.distance_bounded("kitten", "sitting", 2)  # 3 (over limit)
```

### `similarity(a, b)`

Returns a normalized score from `0` to `1`:

`1 - distance(a, b) / max(len(a), len(b))`

Two empty strings have similarity `1`.

```flint
levenshtein.similarity("flint", "flint")  # 1
levenshtein.similarity("", "")             # 1
```

### `closest(query, candidates, k)`

Returns up to `k` records, nearest first. Each record has `value`, `distance`, and `similarity` fields. Equal distances preserve candidate input order.

```flint
import levenshtein

let matches = levenshtein.closest("flnt", [
    "flint",
    "float",
    "print",
    "python",
], 3)

for match in matches {
    print(match.value + ": " + str(match.distance))
}
```

## String semantics

Flint strings are byte sequences: `len()` and indexing operate on bytes, not decoded Unicode code points. This library intentionally follows that rule. ASCII text behaves as expected; a multibyte UTF-8 character contributes multiple bytes to the distance.

## Run tests

From this directory:

```sh
flint tests/test.fl
```

Run the example:

```sh
flint examples/spellcheck.fl
```

## License

MIT. See `LICENSE`.
