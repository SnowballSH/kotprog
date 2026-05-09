# Number Theory

## Fast Modular Exponentiation

Tested:
* 5/9/2026 - https://www.luogu.com.cn/problem/P1226

```kotlin
fun binpow(a: Long, b: Long, p: Long): Long {
    var (a, b) = Pair(a, b)
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
fun euclid(a: Long, b: Long): Triple<Long, Long, Long> {
    if (b == 0L) {
        return Triple(1L, 0L, a)
    }
    val (x1, y1, d) = euclid(b, a % b)
    return Triple(y1, x1 - (a / b) * y1, d)
}
```

## Modular Arithmetic

```kotlin
const val MOD: Long = 17L

@JvmInline
value class ModLong @PublishedApi internal constructor(val x: Long) {
    operator fun plus(b: ModLong) = ModLong((this.x + b.x) % MOD)
    operator fun minus(b: ModLong) = ModLong((this.x - b.x + MOD) % MOD)
    operator fun times(b: ModLong) = ModLong((this.x * b.x) % MOD)

    fun invert(): ModLong {
        val (x, _, g) = euclid(this.x, MOD)
        check(g == 1L) { "Value ${this.x} is not coprime with modulus $MOD" }
        return ModLong((x % MOD + MOD) % MOD)
    }

    operator fun div(b: ModLong) = this * b.invert()

    fun pow(e: Long): ModLong {
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

fun Long.toModLong(): ModLong = ModLong((this % MOD + MOD) % MOD)
```

## Prime Sieve

```kotlin
import java.util.BitSet

const val MAX_PR = 5_000_000
val isPrimeSieve = BitSet(MAX_PR)

fun eratosthenes(lim: Int): ArrayList<Int> {
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
    2uL, 325uL, 9_375uL, 28_178uL, 450_775uL, 9_780_504uL, 1_795_265_022uL
)

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

fun modMul(a: ULong, b: ULong, m: ULong): ULong {
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

fun modPow(base: ULong, exp: ULong, m: ULong): ULong {
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
tailrec fun gcd(a: ULong, b: ULong): ULong {
    return if (b == 0uL) a else gcd(b, a % b)
}

fun pollard(n: ULong): ULong {
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

fun factor(n: ULong, dest: ArrayList<ULong> = ArrayList()): ArrayList<ULong> {
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
