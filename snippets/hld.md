# Heavy-Light Decomposition (HLD)

## Heavy-Light Decomposition

Implemented: 7/4/2026

Tested:
- Vertex Path Sum & Update: https://library.yosupo.jp/problem/vertex_add_path_sum
- Vertex Set Path Composite (Directed/Non-commutative): https://library.yosupo.jp/problem/vertex_set_path_composite
- Subtree Sum & Update: https://library.yosupo.jp/problem/vertex_add_subtree_sum

```kotlin
/**
 * Heavy-Light Decomposition (HLD) on a tree.
 * Decomposes a tree of size `n` into a set of heavy paths.
 * Maps nodes to a contiguous range `[0, n)` in DFS order. This allows querying
 * and updating paths or subtrees using range data structures (like SegmentTree
 * or FenwickTree) in O(log^2 n) for paths and O(log n) for subtrees.
 *
 * Vertices are 0-indexed: `0, 1, ..., n - 1`.
 */
class HLD(
    val adj: List<List<Int>>,
    val root: Int = 0
) {
    val n = adj.size
    
    // parent[i] is the parent of node i (-1 for the root)
    val parent = IntArray(n) { -1 }
    
    // depth[i] is the distance from root to node i
    val depth = IntArray(n)
    
    // sz[i] is the size of the subtree of node i
    val sz = IntArray(n)
    
    // heavy[i] is the heavy child of node i (-1 if leaf)
    val heavy = IntArray(n) { -1 }
    
    // head[i] is the head of the heavy path containing node i
    val head = IntArray(n)
    
    // inTime[i] is the DFS entry time of node i (0-indexed position in range data structures)
    val inTime = IntArray(n)
    
    // outTime[i] is the DFS exit time of node i
    val outTime = IntArray(n)
    
    // order[t] is the node at DFS entry time t
    val order = IntArray(n)

    private var time = 0

    init {
        require(n > 0) { "Tree must have at least one node" }
        dfsSz(root, -1, 0)
        dfsHld(root, root)
    }

    private fun dfsSz(u: Int, p: Int, d: Int) {
        parent[u] = p
        depth[u] = d
        sz[u] = 1
        var maxSz = 0
        for (v in adj[u]) {
            if (v != p) {
                dfsSz(v, u, d + 1)
                sz[u] += sz[v]
                if (sz[v] > maxSz) {
                    maxSz = sz[v]
                    heavy[u] = v
                }
            }
        }
    }

    private fun dfsHld(u: Int, h: Int) {
        head[u] = h
        inTime[u] = time
        order[time] = u
        time++

        if (heavy[u] != -1) {
            dfsHld(heavy[u], h)
        }
        for (v in adj[u]) {
            if (v != parent[u] && v != heavy[u]) {
                dfsHld(v, v)
            }
        }
        outTime[u] = time
    }

    /**
     * REQUIRES: `0 <= u < n` and `0 <= v < n`
     *
     * ENSURES: returns the Lowest Common Ancestor (LCA) of `u` and `v`
     *
     * Lowest Common Ancestor using heavy paths
     *
     * Time Complexity: O(log n)
     */
    fun lca(u: Int, v: Int): Int {
        require(u in 0 until n && v in 0 until n) {
            "Vertices must be in range [0, n)"
        }
        var curU = u
        var curV = v
        while (head[curU] != head[curV]) {
            if (depth[head[curU]] > depth[head[curV]]) {
                curU = parent[head[curU]]
            } else {
                curV = parent[head[curV]]
            }
        }
        return if (depth[curU] < depth[curV]) curU else curV
    }

    /**
     * REQUIRES: `0 <= u < n` and `0 <= v < n`
     *
     * ENSURES: returns the distance (number of edges) between `u` and `v`
     *
     * Queries the number of edges on the path between `u` and `v`
     *
     * Time Complexity: O(log n)
     */
    fun dist(u: Int, v: Int): Int {
        require(u in 0 until n && v in 0 until n) {
            "Vertices must be in range [0, n)"
        }
        return depth[u] + depth[v] - 2 * depth[lca(u, v)]
    }

    /**
     * REQUIRES: `0 <= u < n` and `0 <= v < n`
     *
     * ENSURES: returns true if `u` is an ancestor of `v` (a node is considered an ancestor of itself)
     *
     * Checks if `u` is an ancestor of `v` in O(1)
     *
     * Time Complexity: O(1)
     */
    fun isAncestor(u: Int, v: Int): Boolean {
        require(u in 0 until n && v in 0 until n) {
            "Vertices must be in range [0, n)"
        }
        return inTime[u] <= inTime[v] && outTime[v] <= outTime[u]
    }

    /**
     * REQUIRES: `0 <= u < n`, `0 <= v < n`
     *
     * ENSURES: Decomposes the path between `u` and `v` into O(log n) intervals.
     * Each interval is represented as `[l, r)` (exclusive right endpoint) in the segment tree.
     * The intervals are passed to the `block` lambda.
     *
     * Note: This function assumes the path query/update operation is commutative.
     * The order in which intervals are processed and their direction do not matter.
     *
     * Time Complexity: O(log n) invocations of `block`
     */
    fun forEachPathSegment(u: Int, v: Int, includeLca: Boolean = true, block: (l: Int, r: Int) -> Unit) {
        require(u in 0 until n && v in 0 until n) {
            "Vertices must be in range [0, n)"
        }
        var curU = u
        var curV = v
        while (head[curU] != head[curV]) {
            if (depth[head[curU]] > depth[head[curV]]) {
                block(inTime[head[curU]], inTime[curU] + 1)
                curU = parent[head[curU]]
            } else {
                block(inTime[head[curV]], inTime[curV] + 1)
                curV = parent[head[curV]]
            }
        }
        if (depth[curU] > depth[curV]) {
            val l = if (includeLca) inTime[curV] else inTime[curV] + 1
            val r = inTime[curU] + 1
            if (l < r) {
                block(l, r)
            }
        } else {
            val l = if (includeLca) inTime[curU] else inTime[curU] + 1
            val r = inTime[curV] + 1
            if (l < r) {
                block(l, r)
            }
        }
    }

    /**
     * REQUIRES: `0 <= u < n`, `0 <= v < n`
     *
     * ENSURES: Decomposes the directed path from `u` to `v` into O(log n) intervals.
     * Each interval is represented as `[l, r)` (exclusive right endpoint) in the segment tree.
     * The intervals are passed to the `block` lambda as `(l, r, isAscending)`.
     *
     * - `isAscending` is true: The path goes UP the tree (towards LCA). The range should be
     *   processed from right to left (e.g. from index `r - 1` down to `l`).
     * - `isAscending` is false: The path goes DOWN the tree (away from LCA). The range should be
     *   processed from left to right (e.g. from index `l` to `r - 1`).
     *
     * This is useful for non-commutative operations (e.g. matrix multiplication).
     * The intervals are guaranteed to be processed in order from `u` to `v`.
     *
     * Time Complexity: O(log n) invocations of `block`
     */
    fun forEachPathSegmentDirected(
        u: Int,
        v: Int,
        includeLca: Boolean = true,
        block: (l: Int, r: Int, isAscending: Boolean) -> Unit
    ) {
        require(u in 0 until n && v in 0 until n) {
            "Vertices must be in range [0, n)"
        }
        var curU = u
        var curV = v
        val leftSegments = ArrayList<Pair<Int, Int>>()
        val rightSegments = ArrayList<Pair<Int, Int>>()
        while (head[curU] != head[curV]) {
            if (depth[head[curU]] > depth[head[curV]]) {
                leftSegments.add(Pair(inTime[head[curU]], inTime[curU] + 1))
                curU = parent[head[curU]]
            } else {
                rightSegments.add(Pair(inTime[head[curV]], inTime[curV] + 1))
                curV = parent[head[curV]]
            }
        }
        if (depth[curU] > depth[curV]) {
            val l = if (includeLca) inTime[curV] else inTime[curV] + 1
            val r = inTime[curU] + 1
            if (l < r) {
                leftSegments.add(Pair(l, r))
            }
        } else {
            val l = if (includeLca) inTime[curU] else inTime[curU] + 1
            val r = inTime[curV] + 1
            if (l < r) {
                rightSegments.add(Pair(l, r))
            }
        }

        // Left segments go up the tree (from u towards LCA)
        for (seg in leftSegments) {
            block(seg.first, seg.second, true)
        }
        // Right segments go down the tree (from LCA towards v)
        // They were collected in bottom-up order, so we process them in reverse order
        for (i in rightSegments.size - 1 downTo 0) {
            val seg = rightSegments[i]
            block(seg.first, seg.second, false)
        }
    }

    /**
     * REQUIRES: `0 <= u < n`
     *
     * ENSURES: Returns the range `[l, r)` representing the subtree of `u`.
     * If `includeRoot` is false, it excludes `u` itself, returning the range for its children's subtrees.
     *
     * Time Complexity: O(1)
     */
    fun subtreeRange(u: Int, includeRoot: Boolean = true): Pair<Int, Int> {
        require(u in 0 until n) {
            "Vertex must be in range [0, n)"
        }
        val l = if (includeRoot) inTime[u] else inTime[u] + 1
        val r = outTime[u]
        return Pair(l, r)
    }
}
```

---

### Usage Examples

#### 1. Path Sum Query & Vertex Update (Commutative)
Uses `SegmentTree` from `range-data-structures.md`:

```kotlin
// adj: List<List<Int>> representing the tree
// values: List<Long> representing initial values of vertices
val hld = HLD(adj, root = 0)

// Initialize Segment Tree with values ordered by DFS entry times
val segValues = List(hld.n) { t -> values[hld.order[t]] }
val segTree = SegmentTree(segValues, unit = 0L) { a, b -> a + b }

// Point update: set value of vertex `u` to `valNew`
fun updateVertex(u: Int, valNew: Long) {
    segTree.update(hld.inTime[u], valNew)
}

// Path query: query sum of vertices on the path between `u` and `v`
fun queryPathSum(u: Int, v: Int): Long {
    var sum = 0L
    hld.forEachPathSegment(u, v, includeLca = true) { l, r ->
        sum += segTree.query(l, r)
    }
    return sum
}

// Subtree query: query sum of vertices in the subtree of `u`
fun querySubtreeSum(u: Int): Long {
    val (l, r) = hld.subtreeRange(u, includeRoot = true)
    return segTree.query(l, r)
}
```

#### 2. Range Add on Path & Path Sum Query (Lazy Segment Tree)
Uses `LazySegmentTree` from `range-data-structures.md`:

```kotlin
data class Node(val sum: Long, val len: Int)

val hld = HLD(adj, root = 0)

// Initialize Lazy Segment Tree
val segValues = List(hld.n) { t -> Node(values[hld.order[t]], 1) }
val lazySegTree = LazySegmentTree(
    values = segValues,
    e = Node(0L, 0),
    id = 0L,
    op = { left, right -> Node(left.sum + right.sum, left.len + right.len) },
    mapping = { add, node -> Node(node.sum + add * node.len, node.len) },
    composition = { newAdd, oldAdd -> newAdd + oldAdd }
)

// Range update: add `valAdd` to all vertices on the path between `u` and `v`
fun addPath(u: Int, v: Int, valAdd: Long) {
    hld.forEachPathSegment(u, v, includeLca = true) { l, r ->
        lazySegTree.apply(l, r, valAdd)
    }
}

// Path query: sum of vertices on the path between `u` and `v`
fun queryPathSum(u: Int, v: Int): Long {
    var sum = 0L
    hld.forEachPathSegment(u, v, includeLca = true) { l, r ->
        sum += lazySegTree.query(l, r).sum
    }
    return sum
}
```

#### 3. Edge Weights Queries and Updates
To query/update edge weights on a path/subtree, we store the weight of the edge `(u, parent[u])` in node `u`.
The `root` node does not map to any edge. When querying a path, we exclude the LCA of the endpoints by setting `includeLca = false`.

```kotlin
// edgeWeights[u] is the weight of the edge between `u` and `parent[u]`
val hld = HLD(adj, root = 0)

// Initialize Segment Tree
val segValues = List(hld.n) { t ->
    val u = hld.order[t]
    if (u == hld.root) 0L else edgeWeights[u]
}
val segTree = SegmentTree(segValues, unit = 0L) { a, b -> a + b }

// Update edge (u, parent[u]) weight to `newWeight`
fun updateEdge(u: Int, newWeight: Long) {
    require(u != hld.root) { "Root has no parent edge" }
    segTree.update(hld.inTime[u], newWeight)
}

// Path query: sum of edge weights on the path between `u` and `v`
fun queryPathEdges(u: Int, v: Int): Long {
    var sum = 0L
    hld.forEachPathSegment(u, v, includeLca = false) { l, r ->
        sum += segTree.query(l, r)
    }
    return sum
}
```

#### 4. Directed Path Matrix Product Query (Non-commutative)
For non-commutative operations like matrix multiplication, the order of combination matters. We store both the forward product (`prod`) and reversed product (`revProd`) in the Segment Tree node.

```kotlin
data class Matrix(val a00: Long, val a01: Long, val a10: Long, val a11: Long) {
    operator fun times(other: Matrix): Matrix {
        val MOD = 998244353L
        return Matrix(
            (a00 * other.a00 + a01 * other.a10) % MOD,
            (a00 * other.a01 + a01 * other.a11) % MOD,
            (a10 * other.a00 + a11 * other.a10) % MOD,
            (a10 * other.a01 + a11 * other.a11) % MOD
        )
    }
}

data class MatrixNode(val prod: Matrix, val revProd: Matrix)

val IDENTITY = Matrix(1, 0, 0, 1)
val IDENTITY_NODE = MatrixNode(IDENTITY, IDENTITY)

// segment tree combine function
val combine = { left: MatrixNode, right: MatrixNode ->
    MatrixNode(
        prod = left.prod * right.prod,      // forward product: left * right
        revProd = right.revProd * left.revProd // reversed product: right * left
    )
}

// segTree: SegmentTree<MatrixNode> initialized with MatrixNodes
// Query the ordered product from u to v: M[u] * M[parent[u]] * ... * M[v]
fun queryDirectedPathProduct(u: Int, v: Int): Matrix {
    var res = IDENTITY
    hld.forEachPathSegmentDirected(u, v, includeLca = true) { l, r, isAscending ->
        val node = segTree.query(l, r)
        val segmentMatrix = if (isAscending) node.revProd else node.prod
        res = res * segmentMatrix
    }
    return res
}
```
