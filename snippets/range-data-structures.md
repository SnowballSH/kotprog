# Range Data Structures

## Fenwick Tree

Implemented: 4/6/2026

Tested (update, query):
* 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_b
```kotlin
/**
 * Maintains an array `a`.
 * `update(i, x)`: `a[i] += x`; O(log n)
 * `query(i)`: returns `sum(a[0, i))`; O(log n)
 */
class FenwickTree(n: Int) {
    val s: LongArray = LongArray(n)

    /**
     * REQUIRES: `0 <= pos < n`
     *
     * ENSURES: N/A
     *
     * Performs `a[pos] += dif`
     *
     * Time Complexity: O(log n)
     */
    fun update(pos: Int, dif: Long) {
        require(0 <= pos && pos < s.size) {
            "`pos` not in [0, n)"
        }
        var pos = pos
        while (pos < s.size) {
            s[pos] += dif
            pos = pos or (pos + 1)
        }
    }

    /**
     * REQUIRES: `0 <= pos <= n`
     *
     * ENSURES: `query(pos) == sum(a[0, pos))`
     *
     * Queries `sum(a[0, pos))`
     *
     * Time Complexity: O(log n)
     */
    fun query(pos: Int): Long {
        require(0 <= pos && pos <= s.size) {
            "`pos` not in [0, n]"
        }
        var pos = pos
        var res: Long = 0
        while (pos > 0) {
            res += s[pos - 1]
            pos = pos and (pos - 1)
        }
        return res
    }

    /**
     * REQUIRES: `0 <= l <= r <= n`
     *
     * ENSURES: `queryExclusiveRange(l, r) == sum(a[l, r))`
     *
     * Queries sum over an interval
     *
     * Time Complexity: O(log n)
     */
    fun queryExclusiveRange(l: Int, r: Int): Long {
        require(l in 0..r && r <= s.size) {
            "Arguments do not satisfy `0 <= l <= r <= n`"
        }
        return query(r) - query(l)
    }

    /**
     * REQUIRES: None
     *
     * ENSURES: if `sum <= 0`, then `lowerBound(sum) == -1`;
     * otherwise, `lowerBound(sum)` is the smallest index `i` such that
     * `a[0] + ... + a[i] >= sum`
     *
     * Time Complexity: O(25)
     */
    fun lowerBound(sum: Long): Int {
        if (sum <= 0) return -1
        var sum = sum
        var pos = 0
        var pw = 1 shl 25
        while (pw > 0) {
            if (pos + pw <= s.size && s[pos + pw - 1] < sum) {
                pos += pw
                sum -= s[pos - 1]
            }
            pw = pw shr 1
        }
        return pos
    }
}
```