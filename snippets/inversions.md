# Counting Inversions with Merge Sort

Tested:
- 5/9/2026 - various Luogu problems

```kotlin
/**
 * Sorts `a[l, r)` in non-decreasing order and counts inversions in that range.
 *
 * REQUIRES: `0 <= l <= r <= a.size`
 *
 * ENSURES: returns the number of pairs `(i, j)` with `l <= i < j < r` and
 * `a[i] > a[j]` in the original array segment; after the call, `a[l, r)` is sorted
 *
 * Time Complexity: O((r - l) log(r - l))
 */
fun mergeSort(a: Array<Int>, l: Int, r: Int): Long {
    require(l in 0..r && r in 0..a.size) {
        "Arguments do not satisfy `0 <= l <= r <= a.size`"
    }
    if (r - l <= 1) return 0

    val mid = l + (r - l) / 2
    var ans = mergeSort(a, l, mid) + mergeSort(a, mid, r)

    var i = 0
    var lp = l
    var rp = mid
    val b = Array(r - l) { 0 }
    while (i < r - l) {
        if (lp == mid) {
            b[i++] = a[rp++]
        } else if (rp == r) {
            b[i++] = a[lp++]
            ans += r - mid
        } else if (a[lp] <= a[rp]) {
            b[i++] = a[lp++]
            ans += rp - mid
        } else {
            b[i++] = a[rp++]
        }
    }
    for (i in 0 until r - l) {
        a[l + i] = b[i]
    }

    return ans
}
```
