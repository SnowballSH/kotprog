# Union Find Data Structures

## Union Find

Implemented: 4/6/2026

Tested:
- 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_a

```kotlin
/**
 * Maintains a partition of vertices `[0, n)`.
 * `find(x)`: returns the representative of the set containing `x`; amortized O(alpha(n))
 * `size(x)`: returns the size of the set containing `x`; amortized O(alpha(n))
 * `sameSet(a, b)`: returns whether `a` and `b` are in the same set; amortized O(alpha(n))
 * `join(a, b)`: merges the sets containing `a` and `b`; amortized O(alpha(n))
 */
class UnionFind(n: Int) {
    val dat: IntArray = IntArray(n) { -1 }

    /**
     * REQUIRES: `0 <= x < n`
     *
     * ENSURES: returns the representative of the set containing `x`
     *
     * Finds the representative of the set containing `x`
     *
     * Time Complexity: amortized O(alpha(n))
     */
    fun find(x: Int): Int {
        require(x in dat.indices) {
            "`x` not in [0, n)"
        }
        if (dat[x] < 0) {
            return x
        } else {
            dat[x] = find(dat[x])
            return dat[x]
        }
    }

    /**
     * REQUIRES: `0 <= x < n`
     *
     * ENSURES: returns the size of the set containing `x`
     *
     * Queries the size of the set containing `x`
     *
     * Time Complexity: amortized O(alpha(n))
     */
    fun size(x: Int): Int {
        require(x in dat.indices) {
            "`x` not in [0, n)"
        }
        return -dat[find(x)]
    }

    /**
     * REQUIRES: `0 <= a < n`, `0 <= b < n`
     *
     * ENSURES: returns whether `a` and `b` are in the same set
     *
     * Checks whether two vertices belong to the same set
     *
     * Time Complexity: amortized O(alpha(n))
     */
    fun sameSet(a: Int, b: Int): Boolean {
        require(a in dat.indices) {
            "`a` not in [0, n)"
        }
        require(b in dat.indices) {
            "`b` not in [0, n)"
        }
        return find(a) == find(b)
    }

    /**
     * REQUIRES: `0 <= a < n`, `0 <= b < n`
     *
     * ENSURES: if the sets were distinct, merges them and returns `true`;
     * otherwise, leaves the structure unchanged and returns `false`
     *
     * Merges the sets containing `a` and `b`
     *
     * Time Complexity: amortized O(alpha(n))
     */
    fun join(a: Int, b: Int): Boolean {
        require(a in dat.indices) {
            "`a` not in [0, n)"
        }
        require(b in dat.indices) {
            "`b` not in [0, n)"
        }
        var a = find(a)
        var b = find(b)
        if (a == b) return false
        if (dat[a] > dat[b])
            a = b.also { b = a }
        dat[a] += dat[b]
        dat[b] = a
        return true
    }
}
```

## Union Find with Rollback

Implemented: 4/6/2026

Tested: not tested

```kotlin
/**
 * Maintains a partition of vertices `[0, n)` with rollback support.
 * `find(x)`: returns the representative of the set containing `x`; O(log n)
 * `size(x)`: returns the size of the set containing `x`; O(log n)
 * `sameSet(a, b)`: returns whether `a` and `b` are in the same set; O(log n)
 * `time()`: returns the current rollback timestamp; O(1)
 * `rollback(t)`: restores the state at timestamp `t`; O(time() - t)
 * `join(a, b)`: merges the sets containing `a` and `b`; O(log n)
 */
class UnionFindRollBack(n: Int) {
    val dat: IntArray = IntArray(n) { -1 }
    val st: Stack<Pair<Int, Int>> = Stack()

    /**
     * REQUIRES: `0 <= x < n`
     *
     * ENSURES: returns the representative of the set containing `x`
     *
     * Finds the representative of the set containing `x`
     *
     * Time Complexity: O(log n)
     */
    fun find(x: Int): Int {
        require(x in dat.indices) {
            "`x` not in [0, n)"
        }
        var curr = x
        while (dat[curr] >= 0) {
            curr = dat[curr]
        }
        return curr
    }

    /**
     * REQUIRES: `0 <= x < n`
     *
     * ENSURES: returns the size of the set containing `x`
     *
     * Queries the size of the set containing `x`
     *
     * Time Complexity: O(log n)
     */
    fun size(x: Int): Int {
        require(x in dat.indices) {
            "`x` not in [0, n)"
        }
        return -dat[find(x)]
    }

    /**
     * REQUIRES: `0 <= a < n`, `0 <= b < n`
     *
     * ENSURES: returns whether `a` and `b` are in the same set
     *
     * Checks whether two vertices belong to the same set
     *
     * Time Complexity: O(log n)
     */
    fun sameSet(a: Int, b: Int): Boolean {
        require(a in dat.indices) {
            "`a` not in [0, n)"
        }
        require(b in dat.indices) {
            "`b` not in [0, n)"
        }
        return find(a) == find(b)
    }

    /**
     * REQUIRES: None
     *
     * ENSURES: returns the current rollback timestamp
     *
     * Returns the current rollback timestamp
     *
     * Time Complexity: O(1)
     */
    fun time(): Int {
        return st.size
    }

    /**
     * REQUIRES: `0 <= t <= time()`
     *
     * ENSURES: restores the data structure to the state when `time() == t`
     *
     * Rolls the data structure back to timestamp `t`
     *
     * Time Complexity: O(time() - t)
     */
    fun rollback(t: Int) {
        require(t in 0..st.size) {
            "`t` not in [0, time()]"
        }
        while (st.size > t) {
            val (a, b) = st.pop()
            dat[a] = b
        }
    }

    /**
     * REQUIRES: `0 <= a < n`, `0 <= b < n`
     *
     * ENSURES: if the sets were distinct, merges them and returns `true`;
     * otherwise, leaves the structure unchanged and returns `false`
     *
     * Merges the sets containing `a` and `b`
     *
     * Time Complexity: O(log n)
     */
    fun join(a: Int, b: Int): Boolean {
        require(a in dat.indices) {
            "`a` not in [0, n)"
        }
        require(b in dat.indices) {
            "`b` not in [0, n)"
        }
        var a = find(a)
        var b = find(b)
        if (a == b) return false
        if (dat[a] > dat[b])
            a = b.also { b = a }
        st.push(Pair(a, dat[a]))
        st.push(Pair(b, dat[b]))
        dat[a] += dat[b]
        dat[b] = a
        return true
    }
}
```
