# Treap Data Structures

## Implicit Treap

Implemented: 6/28/2026

Tested:
- 6/28/2026 - Verification of point/range queries, point/range updates, and range reversals.

```kotlin
import kotlin.random.Random

/**
 * Maintains an array of elements using a randomized binary search tree (Treap).
 * Supports range queries, range updates (lazy propagation), and range reversals.
 *
 * `insert(pos, x)`: inserts `x` at index `pos`; O(log n)
 * `erase(pos)`: erases the element at index `pos`; O(log n)
 * `get(pos)`: returns the element at index `pos`; O(log n)
 * `set(pos, x)`: sets the element at index `pos` to `x`; O(log n)
 * `query(l, r)`: returns the range aggregate over `[l, r)`; O(log n)
 * `apply(l, r, f)`: applies the lazy action `f` to the range `[l, r)`; O(log n)
 * `reverse(l, r)`: reverses the subarray in range `[l, r)`; O(log n)
 *
 * REQUIRES: `op` is associative with identity `e`; `id` is an identity lazy action;
 * `mapping(f, x)` applies `f` to an aggregate `x`; and `composition(f, g)` represents
 * applying `f` after `g`
 */
class ImplicitTreap<S, F>(
    val e: S,
    val id: F,
    val op: (S, S) -> S,
    val mapping: (F, S) -> S,
    val composition: (F, F) -> F,
    seed: Long = 0x20260628L
) {
    private val rng = Random(seed)

    inner class Node(
        var value: S,
        var priority: Int,
    ) {
        var left: Node? = null
        var right: Node? = null
        var size: Int = 1
        var agg: S = value
        var revAgg: S = value
        var lazy: F = id
        var rev: Boolean = false
    }

    private var root: Node? = null

    val size: Int
        get() = root?.size ?: 0

    // Reusable fields to eliminate Pair object allocation churn during splits
    private var splitLeft: Node? = null
    private var splitRight: Node? = null

    private fun update(node: Node) {
        node.size = 1 + (node.left?.size ?: 0) + (node.right?.size ?: 0)

        var agg = node.value
        node.left?.let { agg = op(it.agg, agg) }
        node.right?.let { agg = op(agg, it.agg) }
        node.agg = agg

        var revAgg = node.value
        node.right?.let { revAgg = op(it.revAgg, revAgg) }
        node.left?.let { revAgg = op(revAgg, it.revAgg) }
        node.revAgg = revAgg
    }

    private fun allApply(node: Node?, f: F) {
        if (node == null || f == id) return
        node.value = mapping(f, node.value)
        node.agg = mapping(f, node.agg)
        node.revAgg = mapping(f, node.revAgg)
        node.lazy = composition(f, node.lazy)
    }

    private fun allReverse(node: Node?) {
        if (node == null) return
        val tmp = node.left
        node.left = node.right
        node.right = tmp
        val tmpAgg = node.agg
        node.agg = node.revAgg
        node.revAgg = tmpAgg
        node.rev = !node.rev
    }

    private fun push(node: Node) {
        if (node.lazy != id) {
            allApply(node.left, node.lazy)
            allApply(node.right, node.lazy)
            node.lazy = id
        }
        if (node.rev) {
            allReverse(node.left)
            allReverse(node.right)
            node.rev = false
        }
    }

    private fun split(node: Node?, k: Int) {
        if (node == null) {
            splitLeft = null
            splitRight = null
            return
        }
        push(node)
        val leftSize = node.left?.size ?: 0
        if (leftSize >= k) {
            split(node.left, k)
            val l = splitLeft
            val r = splitRight
            node.left = r
            update(node)
            splitLeft = l
            splitRight = node
        } else {
            split(node.right, k - leftSize - 1)
            val l = splitLeft
            val r = splitRight
            node.right = l
            update(node)
            splitLeft = node
            splitRight = r
        }
    }

    private fun merge(l: Node?, r: Node?): Node? {
        if (l == null) return r
        if (r == null) return l
        if (l.priority > r.priority) {
            push(l)
            l.right = merge(l.right, r)
            update(l)
            return l
        } else {
            push(r)
            r.left = merge(l, r.left)
            update(r)
            return r
        }
    }

    /**
     * O(n) Linear Time Builder from an initial list
     */
    constructor(
        values: List<S>,
        e: S,
        id: F,
        op: (S, S) -> S,
        mapping: (F, S) -> S,
        composition: (F, F) -> F,
        seed: Long = 0x20260628L
    ) : this(e, id, op, mapping, composition, seed) {
        val stack = ArrayList<Node>()
        for (value in values) {
            val node = Node(value, rng.nextInt())
            var lastPopped: Node? = null
            while (stack.isNotEmpty() && stack.last().priority < node.priority) {
                lastPopped = stack.removeAt(stack.size - 1)
                update(lastPopped)
            }
            node.left = lastPopped
            if (stack.isNotEmpty()) {
                stack.last().right = node
            }
            stack.add(node)
        }
        while (stack.isNotEmpty()) {
            update(stack.removeAt(stack.size - 1))
        }
        root = stack.firstOrNull()
    }

    fun insert(pos: Int, value: S) {
        require(pos in 0..size) { "Index $pos out of bounds for size $size" }
        split(root, pos)
        val left = splitLeft
        val right = splitRight
        val newNode = Node(value, rng.nextInt())
        root = merge(merge(left, newNode), right)
    }

    fun erase(pos: Int) {
        require(pos in 0 until size) { "Index $pos out of bounds for size $size" }
        split(root, pos)
        val left = splitLeft
        val right = splitRight

        split(right, 1)
        val right2 = splitRight
        root = merge(left, right2)
    }

    fun get(pos: Int): S {
        require(pos in 0 until size) { "Index $pos out of bounds for size $size" }
        split(root, pos)
        val left = splitLeft
        val right = splitRight

        split(right, 1)
        val mid = splitLeft
        val right2 = splitRight

        val res = mid!!.value
        root = merge(left, merge(mid, right2))
        return res
    }

    fun set(pos: Int, value: S) {
        require(pos in 0 until size) { "Index $pos out of bounds for size $size" }
        split(root, pos)
        val left = splitLeft
        val right = splitRight

        split(right, 1)
        val mid = splitLeft
        val right2 = splitRight

        mid!!.value = value
        update(mid)
        root = merge(left, merge(mid, right2))
    }

    fun query(l: Int, r: Int): S {
        require(l in 0..r && r in 0..size) { "Arguments $l, $r do not satisfy 0 <= l <= r <= size ($size)" }
        if (l == r) return e
        split(root, l)
        val left = splitLeft
        val right = splitRight

        split(right, r - l)
        val mid = splitLeft
        val right2 = splitRight

        val res = mid?.agg ?: e
        root = merge(left, merge(mid, right2))
        return res
    }

    fun apply(l: Int, r: Int, f: F) {
        require(l in 0..r && r in 0..size) { "Arguments $l, $r do not satisfy 0 <= l <= r <= size ($size)" }
        if (l == r || f == id) return
        split(root, l)
        val left = splitLeft
        val right = splitRight

        split(right, r - l)
        val mid = splitLeft
        val right2 = splitRight

        allApply(mid, f)
        root = merge(left, merge(mid, right2))
    }

    fun reverse(l: Int, r: Int) {
        require(l in 0..r && r in 0..size) { "Arguments $l, $r do not satisfy 0 <= l <= r <= size ($size)" }
        if (l == r) return
        split(root, l)
        val left = splitLeft
        val right = splitRight

        split(right, r - l)
        val mid = splitLeft
        val right2 = splitRight

        allReverse(mid)
        root = merge(left, merge(mid, right2))
    }

    fun toList(): List<S> {
        val res = ArrayList<S>(size)
        fun traverse(node: Node?) {
            if (node == null) return
            push(node)
            traverse(node.left)
            res.add(node.value)
            traverse(node.right)
        }
        traverse(root)
        return res
    }
}
```

## Explicit Treap

Implemented: 6/28/2026

Tested:
- 6/28/2026 - Verification of set/multiset binary search tree operations.

```kotlin
import kotlin.random.Random

/**
 * Maintains a sorted set/multiset of elements using a randomized binary search tree (Treap).
 * Supports standard BST operations and order statistics queries.
 *
 * `insert(key)`: inserts `key` into the Treap; O(log n)
 * `erase(key)`: removes one instance of `key` from the Treap; returns whether it existed; O(log n)
 * `contains(key)`: returns whether `key` exists in the Treap; O(log n)
 * `count(key)`: returns the number of occurrences of `key`; O(log n)
 * `rank(key)`: returns the number of elements strictly less than `key`; O(log n)
 * `kth(k)`: returns the k-th smallest element (0-indexed); O(log n)
 * `lowerBound(key)`: returns the smallest element `>= key`, or `null` if none; O(log n)
 * `upperBound(key)`: returns the smallest element `> key`, or `null` if none; O(log n)
 */
class ExplicitTreap<K : Comparable<K>>(
    seed: Long = 0x20260628L
) {
    private val rng = Random(seed)

    class Node<K>(
        var key: K,
        var priority: Int,
    ) {
        var left: Node<K>? = null
        var right: Node<K>? = null
        var size: Int = 1
    }

    private var root: Node<K>? = null

    val size: Int
        get() = root?.size ?: 0

    // Shared global state handles inside the instance to stop GC allocation leaks during recursion
    private var splitLeft: Node<K>? = null
    private var splitRight: Node<K>? = null

    private fun update(node: Node<K>) {
        node.size = 1 + (node.left?.size ?: 0) + (node.right?.size ?: 0)
    }

    private fun splitByKey(node: Node<K>?, key: K) {
        if (node == null) {
            splitLeft = null
            splitRight = null
            return
        }
        if (node.key < key) {
            splitByKey(node.right, key)
            val l = splitLeft
            val r = splitRight
            node.right = l
            update(node)
            splitLeft = node
            splitRight = r
        } else {
            splitByKey(node.left, key)
            val l = splitLeft
            val r = splitRight
            node.left = r
            update(node)
            splitLeft = l
            splitRight = node
        }
    }

    private fun splitByKeyGreater(node: Node<K>?, key: K) {
        if (node == null) {
            splitLeft = null
            splitRight = null
            return
        }
        if (node.key <= key) {
            splitByKeyGreater(node.right, key)
            val l = splitLeft
            val r = splitRight
            node.right = l
            update(node)
            splitLeft = node
            splitRight = r
        } else {
            splitByKeyGreater(node.left, key)
            val l = splitLeft
            val r = splitRight
            node.left = r
            update(node)
            splitLeft = l
            splitRight = node
        }
    }

    private fun splitBySize(node: Node<K>?, k: Int) {
        if (node == null) {
            splitLeft = null
            splitRight = null
            return
        }
        val leftSize = node.left?.size ?: 0
        if (leftSize >= k) {
            splitBySize(node.left, k)
            val l = splitLeft
            val r = splitRight
            node.left = r
            update(node)
            splitLeft = l
            splitRight = node
        } else {
            splitBySize(node.right, k - leftSize - 1)
            val l = splitLeft
            val r = splitRight
            node.right = l
            update(node)
            splitLeft = node
            splitRight = r
        }
    }

    private fun merge(l: Node<K>?, r: Node<K>?): Node<K>? {
        if (l == null) return r
        if (r == null) return l
        if (l.priority > r.priority) {
            l.right = merge(l.right, r)
            update(l)
            return l
        } else {
            r.left = merge(l, r.left)
            update(r)
            return r
        }
    }

    private fun getMin(node: Node<K>?): K? {
        var curr = node ?: return null
        while (curr.left != null) {
            curr = curr.left!!
        }
        return curr.key
    }

    private fun getMax(node: Node<K>?): K? {
        var curr = node ?: return null
        while (curr.right != null) {
            curr = curr.right!!
        }
        return curr.key
    }

    fun insert(key: K) {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight
        val newNode = Node(key, rng.nextInt())
        root = merge(merge(l, newNode), r)
    }

    fun erase(key: K): Boolean {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight
        val firstKey = getMin(r)
        return if (firstKey == key) {
            splitBySize(r, 1)
            val r2 = splitRight
            root = merge(l, r2)
            true
        } else {
            root = merge(l, r)
            false
        }
    }

    fun contains(key: K): Boolean {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight
        val firstKey = getMin(r)
        val ans = firstKey == key
        root = merge(l, r)
        return ans
    }

    fun count(key: K): Int {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight

        splitByKeyGreater(r, key)
        val mid = splitLeft
        val r2 = splitRight

        val ans = mid?.size ?: 0
        root = merge(l, merge(mid, r2))
        return ans
    }

    fun rank(key: K): Int {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight
        val ans = l?.size ?: 0
        root = merge(l, r)
        return ans
    }

    fun kth(k: Int): K {
        require(k in 0 until size) { "Index $k out of bounds for size $size" }
        splitBySize(root, k)
        val l = splitLeft
        val r = splitRight

        splitBySize(r, 1)
        val mid = splitLeft
        val r2 = splitRight

        val ans = mid!!.key
        root = merge(merge(l, mid), r2)
        return ans
    }

    fun lowerBound(key: K): K? {
        splitByKey(root, key)
        val l = splitLeft
        val r = splitRight
        val ans = getMin(r)
        root = merge(l, r)
        return ans
    }

    fun upperBound(key: K): K? {
        splitByKeyGreater(root, key)
        val l = splitLeft
        val r = splitRight
        val ans = getMin(r)
        root = merge(l, r)
        return ans
    }

    fun first(): K? = getMin(root)

    fun last(): K? = getMax(root)

    fun toList(): List<K> {
        val res = ArrayList<K>(size)
        fun traverse(node: Node<K>?) {
            if (node == null) return
            traverse(node.left)
            res.add(node.key)
            traverse(node.right)
        }
        traverse(root)
        return res
    }
}
```
