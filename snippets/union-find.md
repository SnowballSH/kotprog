# Union Find Data Structures

## Union Find

Time: O(alpha(N))

Implemented: 4/6/2026

Tested:
* 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_a
```kotlin
class UnionFind(n: Int) {
    val dat: ArrayList<Int> = List(n) { -1 } as ArrayList<Int>

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

Time: O(log(N))

Implemented: 4/6/2026
```kotlin
class UnionFindRollBack(n: Int) {
    val dat: ArrayList<Int> = List(n) { -1 } as ArrayList<Int>
    val st: Stack<Pair<Int, Int>> = Stack()

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

    fun time(): Int {
        return st.size
    }

    fun rollback(t: Int) {
        for (i in time() downTo t) {
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