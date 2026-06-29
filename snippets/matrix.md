# Matrix

## Abstract Matrix

Implemented: 5/21/2026

Usage:
- `val a = ModMatrix.zero(n, n, MOD)`
- `a[i, j] = x`
- `val b = ModMatrix.of(arrayOf(longArrayOf(1, 1), longArrayOf(1, 0)), MOD)`
- `val c = b.pow(k)`
- `val raw = a.rawData()` for direct row-major mutation when every nanosecond matters

```kotlin
/**
 * Mutable dense row-major matrix over a user-defined algebra.
 *
 * `get(r, c)`: returns the value at row r and column c; O(1)
 * `set(r, c, value)`: sets the value at row r and column c; O(1)
 * `times(other)`: returns this * other; cache-friendly tiled multiplication.
 * `pow(e)`: returns this^e by binary exponentiation.
 *
 * REQUIRES:
 * - [zero] is the additive identity and absorbing for [multiply]
 * - [one] is the multiplicative identity
 * - [add] is associative
 * - [multiply] distributes over [add]
 *
 * Multiplication Time Complexity: O(rows * cols * other.cols)
 * Exponentiation Time Complexity: O(n^3 log e) for an n x n matrix
 * Space Complexity: O(rows * cols)
 */
abstract class Matrix<M : Matrix<M>> protected constructor(
    val rows: Int,
    val cols: Int,
    protected val data: LongArray,
) {
    protected abstract val zero: Long
    protected abstract val one: Long
    protected open val blockSize: Int = 32

    init {
        require(rows > 0 && cols > 0) {
            "Matrix dimensions must be positive"
        }
        require(data.size == rows * cols) {
            "`data.size` must equal rows * cols"
        }
    }

    protected abstract fun add(a: Long, b: Long): Long
    protected abstract fun multiply(a: Long, b: Long): Long
    protected open fun normalize(x: Long): Long = x
    protected abstract fun construct(rows: Int, cols: Int, data: LongArray): M

    operator fun get(r: Int, c: Int): Long {
        require(r in 0 until rows && c in 0 until cols) {
            "Index out of bounds"
        }
        return data[r * cols + c]
    }

    operator fun set(r: Int, c: Int, value: Long) {
        require(r in 0 until rows && c in 0 until cols) {
            "Index out of bounds"
        }
        data[r * cols + c] = normalize(value)
    }

    fun fill(value: Long) {
        data.fill(normalize(value))
    }

    fun copy(): M = construct(rows, cols, data.copyOf())

    fun toLongArray(): LongArray = data.copyOf()

    /**
     * Exposes the backing array for performance-sensitive contest code.
     *
     * Values are stored row-major: index `(r, c)` is `r * cols + c`.
     * Mutating this array bypasses [normalize].
     */
    fun rawData(): LongArray = data

    /**
     * REQUIRES: `cols == other.rows`
     *
     * ENSURES: returns the matrix product under [add] and [multiply]
     *
     * Time Complexity: O(rows * cols * other.cols)
     */
    operator fun times(other: M): M {
        require(cols == other.rows) {
            "Matrix dimensions do not match for multiplication"
        }

        val result = LongArray(rows * other.cols) { zero }
        val bs = blockSize
        require(bs > 0) {
            "`blockSize` must be positive"
        }

        var ii = 0
        while (ii < rows) {
            val iEnd = minOf(ii + bs, rows)
            var kk = 0
            while (kk < cols) {
                val kEnd = minOf(kk + bs, cols)
                var jj = 0
                while (jj < other.cols) {
                    val jEnd = minOf(jj + bs, other.cols)

                    for (i in ii until iEnd) {
                        val aRow = i * cols
                        val cRow = i * other.cols
                        for (k in kk until kEnd) {
                            val aik = data[aRow + k]
                            if (aik == zero) continue

                            val bRow = k * other.cols
                            for (j in jj until jEnd) {
                                val bkj = other.data[bRow + j]
                                if (bkj == zero) continue

                                val idx = cRow + j
                                result[idx] = add(result[idx], multiply(aik, bkj))
                            }
                        }
                    }

                    jj += bs
                }
                kk += bs
            }
            ii += bs
        }

        return construct(rows, other.cols, result)
    }

    /**
     * REQUIRES: `rows == cols` and `e >= 0`
     *
     * ENSURES: returns this matrix raised to the power `e`
     *
     * Time Complexity: O(rows^3 log e)
     */
    fun pow(e: Long): M {
        require(rows == cols) {
            "Matrix exponentiation requires a square matrix"
        }
        require(e >= 0L) {
            "`e` must be non-negative"
        }

        var exp = e
        var base = copy()
        var ans = identity(rows)

        while (exp > 0L) {
            if ((exp and 1L) != 0L) ans = ans * base
            base = base * base
            exp = exp shr 1
        }

        return ans
    }

    /**
     * ENSURES: returns the n x n identity matrix
     *
     * Time Complexity: O(n^2)
     */
    fun identity(n: Int): M {
        require(n > 0) {
            "`n` must be positive"
        }
        val result = LongArray(n * n) { zero }
        for (i in 0 until n) {
            result[i * n + i] = one
        }
        return construct(n, n, result)
    }
}

/**
 * Example: matrix over integers modulo [mod].
 *
 * REQUIRES: products `a * b` do not overflow Long before taking `% mod`.
 */
class ModMatrix private constructor(
    rows: Int,
    cols: Int,
    data: LongArray,
    private val mod: Long,
) : Matrix<ModMatrix>(rows, cols, data) {
    override val zero: Long = 0L
    override val one: Long = 1L % mod

    override fun normalize(x: Long): Long {
        val y = x % mod
        return if (y < 0L) y + mod else y
    }

    override fun add(a: Long, b: Long): Long {
        return if (a >= mod - b) a - (mod - b) else a + b
    }

    override fun multiply(a: Long, b: Long): Long = a * b % mod
    override fun construct(rows: Int, cols: Int, data: LongArray): ModMatrix {
        return ModMatrix(rows, cols, data, mod)
    }

    companion object {
        fun zero(rows: Int, cols: Int, mod: Long): ModMatrix {
            require(mod > 0L) {
                "`mod` must be positive"
            }
            return ModMatrix(rows, cols, LongArray(rows * cols), mod)
        }

        fun identity(n: Int, mod: Long): ModMatrix {
            val result = zero(n, n, mod)
            for (i in 0 until n) {
                result[i, i] = 1L
            }
            return result
        }

        fun of(rows: Int, cols: Int, values: LongArray, mod: Long): ModMatrix {
            require(mod > 0L) {
                "`mod` must be positive"
            }
            require(values.size == rows * cols) {
                "`values.size` must equal rows * cols"
            }
            val result = ModMatrix(rows, cols, LongArray(values.size), mod)
            for (i in values.indices) {
                val x = values[i] % mod
                result.rawData()[i] = if (x < 0L) x + mod else x
            }
            return result
        }

        fun of(values: Array<LongArray>, mod: Long): ModMatrix {
            require(values.isNotEmpty() && values[0].isNotEmpty()) {
                "`values` must be non-empty"
            }
            val rows = values.size
            val cols = values[0].size
            val result = zero(rows, cols, mod)
            for (i in 0 until rows) {
                require(values[i].size == cols) {
                    "All rows must have the same length"
                }
                for (j in 0 until cols) {
                    result[i, j] = values[i][j]
                }
            }
            return result
        }
    }
}
```
