# Counting Inversions with Merge Sort

Tested:
- 5/9/2026 - various Luogu problems

```kotlin
/**
 * Sorts `a[l, r)` in non-decreasing order and counts inversions in that range.
 *
 * REQUIRES: `0 <= l <= r <= a.size` and `b.size >= a.size`
 *
 * ENSURES: returns the number of pairs `(i, j)` with `l <= i < j < r` and
 * `a[i] > a[j]` in the original array segment; after the call, `a[l, r)` is sorted
 *
 * Time Complexity: O((r - l) log(r - l))
 */
fun mergeSort(a: Array<Int>, b: Array<Int>, l: Int, r: Int): Long {
    require(l in 0..r && r in 0..a.size) {
        "Arguments do not satisfy `0 <= l <= r <= a.size`"
    }
    require(b.size >= a.size) {
        "Scratch buffer must satisfy `b.size >= a.size`"
    }
    if (r - l <= 1) return 0

    val mid = l + (r - l) / 2
    var ans = mergeSort(a, b, l, mid) + mergeSort(a, b, mid, r)

    var lp = l
    var rp = mid
    var bp = l
    while (lp < mid && rp < r) {
        if (a[lp] <= a[rp]) {
            b[bp++] = a[lp++]
            ans += rp - mid
        } else {
            b[bp++] = a[rp++]
        }
    }
    while (lp < mid) {
        b[bp++] = a[lp++]
        ans += r - mid
    }
    while (rp < r) {
        b[bp++] = a[rp++]
    }
    for (i in l until r) {
        a[i] = b[i]
    }

    return ans
}
```
