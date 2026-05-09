# Counting Inversions with Merge Sort

Tested (rawF):
* 5/9/2026 - various Luogu problems
```kotlin
fun mergeSort(a: Array<Int>, l: Int, r: Int): Long {
    if (r - l == 1) return 0

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