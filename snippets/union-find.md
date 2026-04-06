# Union Find Data Structures

## Union Find

Implemented: 4/6/2026

Tested:
* 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_a
```kotlin
class UnionFind(n: Int) {
    val dat: IntArray = IntArray(n) { -1 }

    fun find(x: Int): Int {
        if (dat[x] < 0) {
            return x
        } else {
            dat[x] = find(dat[x])
            return dat[x]
        }
    }

    fun size(x: Int): Int {
        return -dat[find(x)]
    }

    fun sameSet(a: Int, b: Int): Boolean {
        return find(a) == find(b)
    }

    fun join(a: Int, b: Int): Boolean {
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
class UnionFindRollBack(n: Int) {
    val dat: IntArray = IntArray(n) { -1 }
    val st: Stack<Pair<Int, Int>> = Stack()

    fun find(x: Int): Int {
        var curr = x
        while (dat[curr] >= 0) {
            curr = dat[curr]
        }
        return curr
    }

    fun size(x: Int): Int {
        return -dat[find(x)]
    }

    fun sameSet(a: Int, b: Int): Boolean {
        return find(a) == find(b)
    }

    fun time(): Int {
        return st.size
    }

    fun rollback(t: Int) {
        while (st.size > t) {
            val (a, b) = st.pop()
            dat[a] = b
        }
    }

    fun join(a: Int, b: Int): Boolean {
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