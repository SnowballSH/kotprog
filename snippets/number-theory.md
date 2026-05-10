# Number Theory

## Fast Modular Exponentiation

Tested:
- 5/9/2026 - https://www.luogu.com.cn/problem/P1226

```kotlin
/**
 * Computes `a^b mod p`.
 *
 * REQUIRES: `b >= 0`; `p > 0`
 *
 * ENSURES: returns the residue of `a^b` modulo `p` in `[0, p)`
 *
 * Time Complexity: O(log b)
 */
fun binpow(a: Long, b: Long, p: Long): Long {
    require(b >= 0L) {
        "`b` must be non-negative"
    }
    require(p > 0L) {
        "`p` must be positive"
    }
    var a = ((a % p) + p) % p
    var b = b
    if (b == 0L) return 1 % p
    var ans = 1L
    while (b > 0) {
        if (b and 1 == 1L) ans = ans * a % p
        a = a * a % p
        b = b shr 1
    }
    return ans
}
```

Implemented: 4/6/2026

## Extended GCD

```kotlin
/**
 * Extended Euclidean algorithm.
 *
 * REQUIRES: `a >= 0`, `b >= 0`, and at least one of them is positive
 *
 * ENSURES: returns `(x, y, d)` such that `a * x + b * y == d` and `d == gcd(a, b)`
 *
 * Time Complexity: O(log(max(a, b)))
 */
fun euclid(a: Long, b: Long): Triple<Long, Long, Long> {
    require(a >= 0L) {
        "`a` must be non-negative"
    }
    require(b >= 0L) {
        "`b` must be non-negative"
    }
    require(a != 0L || b != 0L) {
        "At least one of `a` or `b` must be non-zero"
    }
    if (b == 0L) {
        return Triple(1L, 0L, a)
    }
    val (x1, y1, d) = euclid(b, a % b)
    return Triple(y1, x1 - (a / b) * y1, d)
}
```

## Modular Arithmetic

```kotlin
/**
 * Global modulus used by [ModLong].
 *
 * REQUIRES: `MOD > 0`
 */
const val MOD: Long = 17L

/**
 * Lightweight modular integer wrapper under the global modulus [MOD].
 *
 * REQUIRES: `MOD > 0`
 */
@JvmInline
value class ModLong @PublishedApi internal constructor(val x: Long) {
    operator fun plus(b: ModLong) = ModLong((this.x + b.x) % MOD)
    operator fun minus(b: ModLong) = ModLong((this.x - b.x + MOD) % MOD)
    operator fun times(b: ModLong) = ModLong((this.x * b.x) % MOD)

    /**
     * REQUIRES: `gcd(x, MOD) == 1`
     *
     * ENSURES: returns the multiplicative inverse of this value modulo [MOD]
     *
     * Time Complexity: O(log MOD)
     */
    fun invert(): ModLong {
        val (x, _, g) = euclid(this.x, MOD)
        check(g == 1L) { "Value ${this.x} is not coprime with modulus $MOD" }
        return ModLong((x % MOD + MOD) % MOD)
    }

    operator fun div(b: ModLong) = this * b.invert()

    /**
     * REQUIRES: `e >= 0`
     *
     * ENSURES: returns `this^e`
     *
     * Time Complexity: O(log e)
     */
    fun pow(e: Long): ModLong {
        require(e >= 0L) {
            "`e` must be non-negative"
        }
        var res = ModLong(1L)
        var base = this
        var exp = e
        while (exp > 0L) {
            if ((exp and 1L) != 0L) res *= base
            base *= base
            exp = exp shr 1
        }
        return res
    }

    override fun toString(): String = x.toString()
}

/**
 * Normalizes a signed integer into `[0, MOD)`.
 *
 * REQUIRES: `MOD > 0`
 *
 * ENSURES: returns the canonical representative of this residue class
 */
fun Long.toModLong(): ModLong = ModLong((this % MOD + MOD) % MOD)
```

## Prime Sieve

```kotlin
import java.util.BitSet

/**
 * Maximum supported exclusive upper bound for [eratosthenes].
 */
const val MAX_PR = 5_000_000

/**
 * Shared sieve workspace. After `eratosthenes(lim)`, entries in `[0, lim)` reflect primality.
 */
val isPrimeSieve = BitSet(MAX_PR)

/**
 * Returns all primes in `[2, lim)`.
 *
 * REQUIRES: `0 <= lim <= MAX_PR`
 *
 * ENSURES: returns the primes below `lim` in increasing order and updates [isPrimeSieve]
 * on the same range
 *
 * Time Complexity: O(lim log log lim)
 */
fun eratosthenes(lim: Int): ArrayList<Int> {
    require(lim >= 0) { "Limit must be non-negative" }
    require(lim <= MAX_PR) { "Limit $lim exceeds MAX_PR bound of $MAX_PR" }

    isPrimeSieve.set(0, lim)

    if (lim > 0) isPrimeSieve.clear(0)
    if (lim > 1) isPrimeSieve.clear(1)

    for (i in 4 until lim step 2) {
        isPrimeSieve.clear(i)
    }

    var i = 3
    while (i * i < lim) {
        if (isPrimeSieve.get(i)) {
            for (j in i * i until lim step i * 2) {
                isPrimeSieve.clear(j)
            }
        }
        i += 2
    }

    val primes = ArrayList<Int>(maxOf(10, lim / 10))
    if (lim > 2) primes.add(2)
    for (p in 3 until lim step 2) {
        if (isPrimeSieve.get(p)) {
            primes.add(p)
        }
    }
    return primes
}
```

## 64-bit Primality Test

```kotlin
private val MILLER_RABIN_BASES = arrayOf(
    2uL, 325uL, 9_375uL, 28_178uL, 450_775uL, 9_780_504uL, 1_795_265_022uL,
)

/**
 * Deterministic Miller-Rabin primality test for 64-bit unsigned integers.
 *
 * ENSURES: returns whether `n` is prime
 *
 * Time Complexity: O(log n)
 */
@OptIn(ExperimentalUnsignedTypes::class)
fun isPrime(n: ULong): Boolean {
    if (n < 2uL) return false
    if (n == 2uL || n == 3uL) return true
    if ((n and 1uL) == 0uL) return false

    val d0 = n - 1uL
    val s = d0.countTrailingZeroBits()
    val d = d0 shr s

    for (a in MILLER_RABIN_BASES) {
        if (a % n == 0uL) continue

        var x = modPow(a % n, d, n)
        if (x == 1uL || x == d0) continue

        var witnessFound = true
        for (r in 1 until s) {
            x = modMul(x, x, n)
            if (x == d0) {
                witnessFound = false
                break
            }
        }

        if (witnessFound) return false
    }
    return true
}

/**
 * Computes `(a * b) mod m` without overflowing 64-bit unsigned arithmetic.
 *
 * REQUIRES: `m > 0`
 *
 * ENSURES: returns the residue of `a * b` modulo `m`
 *
 * Time Complexity: O(log b)
 */
fun modMul(a: ULong, b: ULong, m: ULong): ULong {
    require(m > 0uL) {
        "`m` must be positive"
    }
    var x = a % m
    var y = b
    var result = 0uL

    while (y != 0uL) {
        if ((y and 1uL) != 0uL) {
            result = addMod(result, x, m)
        }
        x = addMod(x, x, m)
        y = y shr 1
    }

    return result
}

/**
 * Computes `base^exp mod m`.
 *
 * REQUIRES: `m > 0`
 *
 * ENSURES: returns the residue of `base^exp` modulo `m`
 *
 * Time Complexity: O(log exp * log m)
 */
fun modPow(base: ULong, exp: ULong, m: ULong): ULong {
    require(m > 0uL) {
        "`m` must be positive"
    }
    if (m == 1uL) return 0uL

    var res = 1uL
    var b = base % m
    var e = exp
    while (e != 0uL) {
        if ((e and 1uL) != 0uL) res = modMul(res, b, m)
        b = modMul(b, b, m)
        e = e shr 1
    }
    return res
}

private fun addMod(a: ULong, b: ULong, mod: ULong): ULong {
    val threshold = mod - b
    return if (a >= threshold) a - threshold else a + b
}
```

## Factorization

```kotlin
/**
 * Greatest common divisor over unsigned integers.
 *
 * ENSURES: returns `gcd(a, b)`
 *
 * Time Complexity: O(log(max(a, b)))
 */
tailrec fun gcd(a: ULong, b: ULong): ULong {
    return if (b == 0uL) a else gcd(b, a % b)
}

/**
 * Pollard-Rho factor search.
 *
 * REQUIRES: `n > 1` and `n` is composite
 *
 * ENSURES: returns a non-trivial factor of `n`
 *
 * Time Complexity: expected sublinear in `n`; practical for 64-bit inputs
 */
fun pollard(n: ULong): ULong {
    require(n > 1uL) {
        "`n` must be greater than 1"
    }
    if (n % 2uL == 0uL) return 2uL

    var x = 0uL
    var y = 0uL
    var t = 30
    var prd = 2uL
    var i = 1uL
    var q: ULong

    fun f(v: ULong): ULong = addMod(modMul(v, v, n), i % n, n)

    while (true) {
        val skipGcd = (t % 40 != 0)

        t++
        if (!skipGcd) {
            val g = gcd(prd, n)
            if (g != 1uL) return g
        }

        if (x == y) {
            i++
            x = i
            y = f(x)
        }

        val diff = if (x > y) x - y else y - x
        q = modMul(prd, diff, n)
        if (q != 0uL) prd = q

        x = f(x)
        y = f(f(y))
    }
}

/**
 * Factorizes `n` into prime factors.
 *
 * REQUIRES: `n > 0`
 *
 * ENSURES: appends the prime factors of `n` to `dest` and returns `dest`;
 * factors are not guaranteed to be sorted
 *
 * Time Complexity: expected near `O(n^(1/4))` per non-trivial split for 64-bit inputs
 */
fun factor(n: ULong, dest: ArrayList<ULong> = ArrayList()): ArrayList<ULong> {
    require(n > 0uL) {
        "`n` must be positive"
    }
    if (n == 1uL) return dest
    if (isPrime(n)) {
        dest.add(n)
        return dest
    }
    val x = pollard(n)
    factor(x, dest)
    factor(n / x, dest)
    return dest
}
```
