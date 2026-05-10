# Floor Sums and Like-Euclidean Algorithms

Source: https://oi-wiki.org/math/number-theory/euclidean/

## Like-Euclidean Calculator

Tested (rawF):
- 4/6/2026 - https://atcoder.jp/contests/practice2/tasks/practice2_c

```kotlin
/**
 * Represents the configuration for which sums to compute.
 *
 * @property computeRawF Whether to compute the unmodded result of the floor sum.
 * @property computeF Whether to compute the modded result of the floor sum.
 * @property computeG Whether to compute the modded result of the linear floor sum.
 * @property computeH Whether to compute the modded result of the squared floor sum.
 */
data class FloorSumConfig(
    val computeRawF: Boolean = true,
    val computeF: Boolean = false,
    val computeG: Boolean = false,
    val computeH: Boolean = false,
)

/**
 * Represents the results of the Like Euclidean (Floor Sum) algorithm.
 * Uncomputed values will default to 0L.
 *
 * @property rawF The unmodded result of $\sum_{i=0}^{n} \lfloor \frac{a \cdot i + b}{c} \rfloor$
 * @property f The result of $\sum_{i=0}^{n} \lfloor \frac{a \cdot i + b}{c} \rfloor \pmod{mod}$
 * @property g The result of $\sum_{i=0}^{n} i \lfloor \frac{a \cdot i + b}{c} \rfloor \pmod{mod}$
 * @property h The result of $\sum_{i=0}^{n} \lfloor \frac{a \cdot i + b}{c} \rfloor^2 \pmod{mod}$
 */
data class FloorSumResult(
    val rawF: Long,
    val f: Long,
    val g: Long,
    val h: Long,
)

/**
 * Solves floor-sum families over `sum_{i=0}^{n} floor((a * i + b) / c)`.
 *
 * REQUIRES: `mod > 0` and `gcd(mod, 6) == 1`
 *
 * `solve(a, b, c, n)`: computes the requested floor-sum variants over `i in [0, n]`; O(log(a + c))
 */
class FloorSumCalculator(private val mod: Long = 998244353L) {
    init {
        require(mod > 0L) {
            "`mod` must be positive"
        }
        require(mod % 2L != 0L && mod % 3L != 0L) {
            "`mod` must be coprime with 6"
        }
    }

    private val i2: Long = (mod + 1) / 2L
    private val i6: Long = (mod + 1) / 6L

    /**
     * REQUIRES: `a >= 0`, `b >= 0`, `c > 0`, `n >= 0`
     *
     * ENSURES: returns the requested sums over `i in [0, n]`;
     * fields disabled by `config` are set to `0L`
     *
     * Evaluates floor-sum expressions and related weighted variants
     *
     * Time Complexity: O(log(a + c))
     */
    fun solve(
        a: Long, b: Long, c: Long, n: Long,
        config: FloorSumConfig = FloorSumConfig(),
    ): FloorSumResult {
        require(a >= 0L) {
            "`a` must be non-negative"
        }
        require(b >= 0L) {
            "`b` must be non-negative"
        }
        require(c > 0L) {
            "`c` must be positive"
        }
        require(n >= 0L) {
            "`n` must be non-negative"
        }
        val calcGH = config.computeG || config.computeH
        val calcF = config.computeF || calcGH
        val calcRawF = config.computeRawF

        return solveInternal(a, b, c, n, calcRawF, calcF, calcGH)
    }

    private fun solveInternal(
        a: Long, b: Long, c: Long, n: Long,
        calcRawF: Boolean, calcF: Boolean, calcGH: Boolean,
    ): FloorSumResult {
        val nMod = n % mod
        val n2 = if (calcF || calcGH) (nMod + 1) * nMod % mod * i2 % mod else 0L
        val n3 = if (calcGH) (2 * nMod + 1) * (nMod + 1) % mod * nMod % mod * i6 % mod else 0L

        if (a >= c || b >= c) {
            val tmp = solveInternal(a % c, b % c, c, n, calcRawF, calcF, calcGH)

            val aa = a / c
            val bb = b / c
            val aaMod = aa % mod
            val bbMod = bb % mod

            val rawF = if (calcRawF) {
                val rawN2 = if (n % 2L == 0L) (n / 2L) * (n + 1) else n * ((n + 1) / 2L)
                tmp.rawF + aa * rawN2 + bb * (n + 1)
            } else 0L

            val f = if (calcF) {
                (tmp.f + aaMod * n2 % mod + bbMod * (nMod + 1) % mod) % mod
            } else 0L

            val g = if (calcGH) {
                (tmp.g + aaMod * n3 % mod + bbMod * n2 % mod) % mod
            } else 0L

            val h = if (calcGH) {
                (tmp.h +
                        2 * bbMod % mod * tmp.f % mod +
                        2 * aaMod % mod * tmp.g % mod +
                        aaMod * aaMod % mod * n3 % mod +
                        bbMod * bbMod % mod * (nMod + 1) % mod +
                        2 * aaMod % mod * bbMod % mod * n2 % mod) % mod
            } else 0L

            return FloorSumResult(rawF, f, g, h)
        }

        val m = (a * n + b) / c

        if (m == 0L) {
            return FloorSumResult(0L, 0L, 0L, 0L)
        }

        val tmp = solveInternal(c, c - b - 1, a, m - 1, calcRawF, calcF, calcGH)
        val mMod = m % mod

        val rawF = if (calcRawF) {
            n * m - tmp.rawF
        } else 0L

        val f = if (calcF) {
            (mMod * nMod % mod - tmp.f + mod) % mod
        } else 0L

        val g = if (calcGH) {
            (mMod * n2 % mod + (mod - tmp.f) * i2 % mod + (mod - tmp.h) * i2 % mod) % mod
        } else 0L

        val h = if (calcGH) {
            (nMod * mMod % mod * mMod % mod - tmp.f - (tmp.g * 2 % mod) + 3 * mod) % mod
        } else 0L

        return FloorSumResult(rawF, f, g, h)
    }
}
```
