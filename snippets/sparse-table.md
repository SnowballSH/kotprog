# Sparse Table

## Sparse Table

Implemented: 5/10/2026

Tested:
- 5/10/2026 - https://www.luogu.com.cn/problem/P2880

```kotlin
/**
 * Maintains a static array `a` and answers idempotent range queries.
 * `query(l, r)`: returns the aggregate over `a[l, r)`; O(1)
 *
 * REQUIRES: `op` is associative and idempotent (e.g., min, max, gcd)
 */
class SparseTable<T>(
    values: List<T>,
    val op: (T, T) -> T,
) {
    val n: Int = values.size
    val maxLog: Int
    val st: MutableList<T>

    init {
        require(n > 0) {
            "Array cannot be empty"
        }
        var log = 0
        while ((1 shl log) <= n) log++
        maxLog = max(1, log)
        st = ArrayList<T>(maxLog * n)
        for (i in 0 until n) {
            st.add(values[i])
        }
        for (j in 1 until maxLog) {
            val len = 1 shl (j - 1)
            for (i in 0 until n) {
                if (i + len < n) {
                    st.add(op(st[(j - 1) * n + i], st[(j - 1) * n + i + len]))
                } else {
                    st.add(st[(j - 1) * n + i])
                }
            }
        }
    }

    /**
     * REQUIRES: `0 <= l < r <= n`
     *
     * ENSURES: returns the aggregate of `a[l, r)`
     *
     * Queries the aggregate over an interval
     *
     * Time Complexity: O(1)
     */
    fun query(l: Int, r: Int): T {
        require(l in 0 until r && r in 0..n) {
            "Arguments do not satisfy `0 <= l < r <= n`"
        }
        val len = r - l
        val k = 31 - len.countLeadingZeroBits()
        return op(st[k * n + l], st[k * n + r - (1 shl k)])
    }
}
```
