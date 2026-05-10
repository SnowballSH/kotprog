# Range Data Structures

## Fenwick Tree

Implemented: 4/6/2026

Tested (update, query):
- 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_b

```kotlin
/**
 * Maintains an array `a`.
 * `update(i, x)`: `a[i] += x`; O(log n)
 * `query(i)`: returns `sum(a[0, i))`; O(log n)
 * `lowerBound(sum)`: returns the smallest index `i` such that `sum(a[0, i + 1)) >= sum`, or `null`; O(25)
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
        require(pos in s.indices) {
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
        require(pos in 0..s.size) {
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
        require(l in 0..r && r in 0..s.size) {
            "Arguments do not satisfy `0 <= l <= r <= n`"
        }
        return query(r) - query(l)
    }

    /**
     * REQUIRES: None
     *
     * ENSURES: if `sum <= 0`, or if no index `i` satisfies `a[0] + ... + a[i] >= sum`,
     * then returns `null`; otherwise, returns the smallest such index `i`
     *
     * Time Complexity: O(25)
     */
    fun lowerBound(sum: Long): Int? {
        if (sum <= 0) return null
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
        return pos.takeIf { it < s.size }
    }
}
```

## Segment Tree

Implemented: 4/6/2026

Tested:
- 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_j

```kotlin
/**
 * Maintains an array `a`.
 * `update(i, x)`: sets `a[i] = x`; O(log n)
 * `query(l, r)`: returns the aggregate over `a[l, r)`; O(log n)
 * `findFirst(l, p)`: returns the smallest `r` in `[l, n)` such that `p(a[l] op ... op a[r])`, or `null`; O(log n)
 *
 * REQUIRES: `f` is associative and `unit` is an identity element for `f`
 */
class SegmentTree<T>(
    val n: Int,
    val unit: T,
    defaultValue: T = unit,
    val f: (T, T) -> T,
) {
    val size: Int
    val s: MutableList<T>

    init {
        require(n >= 0) {
            "`n` must be non-negative"
        }
        var size = 1
        while (size < n) {
            size *= 2
        }
        this.size = size
        s = MutableList(2 * size) { unit }
        for (i in 0 until n) {
            s[size + i] = defaultValue
        }
        for (i in size - 1 downTo 1) {
            s[i] = f(s[i * 2], s[i * 2 + 1])
        }
    }

    /**
     * REQUIRES: None
     *
     * ENSURES: `n == values.size`, and initializes `a[i] == values[i]` for each `i`
     *
     * Builds a segment tree from an existing array
     *
     * Time Complexity: O(n)
     */
    constructor(values: List<T>, unit: T, f: (T, T) -> T) : this(values.size, unit, unit, f) {
        for (i in values.indices) {
            s[size + i] = values[i]
        }
        for (i in size - 1 downTo 1) {
            s[i] = f(s[i * 2], s[i * 2 + 1])
        }
    }

    /**
     * REQUIRES: `0 <= pos < n`
     *
     * ENSURES: `a[pos] == value`, and all other entries of `a` are unchanged
     *
     * Performs a point assignment
     *
     * Time Complexity: O(log n)
     */
    fun update(pos: Int, value: T) {
        require(pos in 0 until n) {
            "`pos` not in [0, n)"
        }
        var pos = pos + size
        s[pos] = value
        while (pos > 1) {
            pos /= 2
            s[pos] = f(s[pos * 2], s[pos * 2 + 1])
        }
    }

    /**
     * REQUIRES: `0 <= l <= r <= n`
     *
     * ENSURES: returns the aggregate of `a[l, r)`; if `l == r`, returns `unit`
     *
     * Queries the aggregate over an interval
     *
     * Time Complexity: O(log n)
     */
    fun query(l: Int, r: Int): T {
        require(l in 0..r && r in 0..n) {
            "Arguments do not satisfy `0 <= l <= r <= n`"
        }
        var l = l + size
        var r = r + size
        var resLeft = unit
        var resRight = unit
        while (l < r) {
            if (l % 2 == 1) {
                resLeft = f(resLeft, s[l])
                l++
            }
            if (r % 2 == 1) {
                r--
                resRight = f(s[r], resRight)
            }
            l /= 2
            r /= 2
        }
        return f(resLeft, resRight)
    }

    /**
     * REQUIRES: `0 <= l <= n`; `predicate(unit) == false`; and
     * `predicate` is monotone with respect to extending the segment aggregate
     *
     * ENSURES: returns the minimum `r` such that `l <= r < n` and
     * `predicate(query(l, r + 1))` holds; returns `null` if no such `r` exists
     *
     * Finds the first position at or after `l` where the aggregate satisfies `predicate`.
     * For example, with `f = max`, `findFirst(x) { it >= v }` returns the minimum
     * `j >= x` such that `a[j] >= v`, or `null` if no such index exists.
     *
     * Time Complexity: O(log n)
     */
    fun findFirst(l: Int, predicate: (T) -> Boolean): Int? {
        require(l in 0..n) {
            "`l` not in [0, n]"
        }
        require(!predicate(unit)) {
            "`predicate(unit)` must be false"
        }
        return findFirstRec(1, 0, size, l, unit, predicate).first?.takeIf { it < n }
    }

    private fun findFirstRec(
        node: Int,
        segL: Int,
        segR: Int,
        queryL: Int,
        prefix: T,
        predicate: (T) -> Boolean,
    ): Pair<Int?, T> {
        if (segR <= queryL) {
            return Pair(null, prefix)
        }
        if (queryL <= segL) {
            val combined = f(prefix, s[node])
            if (!predicate(combined)) {
                return Pair(null, combined)
            }
            if (segR - segL == 1) {
                return Pair(segL, prefix)
            }
        }
        if (segR - segL == 1) {
            return Pair(null, prefix)
        }
        val mid = (segL + segR) / 2
        val left = findFirstRec(node * 2, segL, mid, queryL, prefix, predicate)
        if (left.first != null) {
            return left
        }
        return findFirstRec(node * 2 + 1, mid, segR, queryL, left.second, predicate)
    }
}
```

## Lazy Segment Tree

Implemented: 4/6/2026

Tested:
- 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_l

```kotlin
/**
 * Maintains an array `a`.
 * `set(i, x)`: sets `a[i] = x`; O(log n)
 * `get(i)`: returns `a[i]`; O(log n)
 * `apply(l, r, f)`: applies the lazy action `f` to each entry of `a[l, r)`; O(log n)
 * `query(l, r)`: returns the aggregate over `a[l, r)`; O(log n)
 *
 * REQUIRES: `op` is associative with identity `e`; `id` is an identity lazy action;
 * `mapping(f, x)` applies `f` to an aggregate `x`; and `composition(f, g)` represents
 * applying `f` after `g`
 */
class LazySegmentTree<S, F>(
    val n: Int,
    val e: S,
    val id: F,
    defaultValue: S = e,
    val op: (S, S) -> S,
    val mapping: (F, S) -> S,
    val composition: (F, F) -> F,
) {
    val size: Int
    val height: Int
    val d: MutableList<S>
    val lz: MutableList<F>

    init {
        require(n >= 0) {
            "`n` must be non-negative"
        }
        var size = 1
        var height = 0
        while (size < n) {
            size *= 2
            height++
        }
        this.size = size
        this.height = height
        d = MutableList(2 * size) { e }
        lz = MutableList(size) { id }
        for (i in 0 until n) {
            d[size + i] = defaultValue
        }
        for (i in size - 1 downTo 1) {
            updateNode(i)
        }
    }

    /**
     * REQUIRES: None
     *
     * ENSURES: `n == values.size`, and initializes `a[i] == values[i]` for each `i`
     *
     * Builds a lazy segment tree from an existing array
     *
     * Time Complexity: O(n)
     */
    constructor(
        values: List<S>,
        e: S,
        id: F,
        op: (S, S) -> S,
        mapping: (F, S) -> S,
        composition: (F, F) -> F,
    ) : this(values.size, e, id, e, op, mapping, composition) {
        for (i in values.indices) {
            d[size + i] = values[i]
        }
        for (i in size - 1 downTo 1) {
            updateNode(i)
        }
    }

    private fun updateNode(node: Int) {
        d[node] = op(d[node * 2], d[node * 2 + 1])
    }

    private fun allApply(node: Int, f: F) {
        d[node] = mapping(f, d[node])
        if (node < size) {
            lz[node] = composition(f, lz[node])
        }
    }

    private fun push(node: Int) {
        allApply(node * 2, lz[node])
        allApply(node * 2 + 1, lz[node])
        lz[node] = id
    }

    /**
     * REQUIRES: `0 <= pos < n`
     *
     * ENSURES: `a[pos] == value`, and all other entries of `a` are unchanged
     *
     * Performs a point assignment
     *
     * Time Complexity: O(log n)
     */
    fun set(pos: Int, value: S) {
        require(pos in 0 until n) {
            "`pos` not in [0, n)"
        }
        val pos = pos + size
        for (i in height downTo 1) {
            push(pos shr i)
        }
        d[pos] = value
        while (pos > 1) {
            pos /= 2
            updateNode(pos)
        }
    }

    /**
     * REQUIRES: `0 <= pos < n`
     *
     * ENSURES: returns `a[pos]`
     *
     * Queries a single entry
     *
     * Time Complexity: O(log n)
     */
    fun get(pos: Int): S {
        require(pos in 0 until n) {
            "`pos` not in [0, n)"
        }
        var pos = pos + size
        for (i in height downTo 1) {
            push(pos shr i)
        }
        return d[pos]
    }

    /**
     * REQUIRES: `0 <= l <= r <= n`
     *
     * ENSURES: returns the aggregate of `a[l, r)`; if `l == r`, returns `e`
     *
     * Queries the aggregate over an interval
     *
     * Time Complexity: O(log n)
     */
    fun query(l: Int, r: Int): S {
        require(l in 0..r && r in 0..n) {
            "Arguments do not satisfy `0 <= l <= r <= n`"
        }
        if (l == r) return e
        var l = l + size
        var r = r + size
        for (i in height downTo 1) {
            if ((l shr i) shl i != l) {
                push(l shr i)
            }
            if ((r shr i) shl i != r) {
                push((r - 1) shr i)
            }
        }
        var leftResult = e
        var rightResult = e
        while (l < r) {
            if (l % 2 == 1) {
                leftResult = op(leftResult, d[l])
                l++
            }
            if (r % 2 == 1) {
                r--
                rightResult = op(d[r], rightResult)
            }
            l /= 2
            r /= 2
        }
        return op(leftResult, rightResult)
    }

    /**
     * REQUIRES: `0 <= pos < n`
     *
     * ENSURES: applies `f` to `a[pos]`, and all other entries of `a` are unchanged
     *
     * Performs a point lazy update
     *
     * Time Complexity: O(log n)
     */
    fun apply(pos: Int, f: F) {
        require(pos in 0 until n) {
            "`pos` not in [0, n)"
        }
        var pos = pos + size
        for (i in height downTo 1) {
            push(pos shr i)
        }
        d[pos] = mapping(f, d[pos])
        while (pos > 1) {
            pos /= 2
            updateNode(pos)
        }
    }

    /**
     * REQUIRES: `0 <= l <= r <= n`
     *
     * ENSURES: applies `f` to each entry of `a[l, r)` and leaves all other entries unchanged
     *
     * Performs a range lazy update
     *
     * Time Complexity: O(log n)
     */
    fun apply(l: Int, r: Int, f: F) {
        require(l in 0..r && r in 0..n) {
            "Arguments do not satisfy `0 <= l <= r <= n`"
        }
        if (l == r) return
        var l = l + size
        var r = r + size
        val l0 = l
        val r0 = r
        for (i in height downTo 1) {
            if ((l shr i) shl i != l) {
                push(l shr i)
            }
            if ((r shr i) shl i != r) {
                push((r - 1) shr i)
            }
        }
        while (l < r) {
            if (l % 2 == 1) {
                allApply(l, f)
                l++
            }
            if (r % 2 == 1) {
                r--
                allApply(r, f)
            }
            l /= 2
            r /= 2
        }
        for (i in 1..height) {
            if ((l0 shr i) shl i != l0) {
                updateNode(l0 shr i)
            }
            if ((r0 shr i) shl i != r0) {
                updateNode((r0 - 1) shr i)
            }
        }
    }
}
```

Example for binary-array inversion queries with range bit-flips:
```kotlin
data class InversionNode(
    val zeroCount: Long,
    val oneCount: Long,
    val inversionCount: Long,
)

val values = a.map { bit ->
    if (bit == 0) InversionNode(1, 0, 0) else InversionNode(0, 1, 0)
}

val segTree = LazySegmentTree(
    values = values,
    e = InversionNode(0, 0, 0),
    id = false,
    op = { left, right ->
        InversionNode(
            zeroCount = left.zeroCount + right.zeroCount,
            oneCount = left.oneCount + right.oneCount,
            inversionCount = left.inversionCount + right.inversionCount + left.oneCount * right.zeroCount,
        )
    },
    mapping = { flip, node ->
        if (!flip) {
            node
        } else {
            InversionNode(
                zeroCount = node.oneCount,
                oneCount = node.zeroCount,
                inversionCount = node.zeroCount * node.oneCount - node.inversionCount,
            )
        }
    },
    composition = { newFlip, oldFlip -> newFlip xor oldFlip },
)

segTree.apply(l, r, true)
val inversionCount = segTree.query(l, r).inversionCount
```
