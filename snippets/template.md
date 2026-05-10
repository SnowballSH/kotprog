# Template

Main Contest File

Last updated: 4/6/2026

```kotlin
import java.util.*

/** IO */
private val INPUT = System.`in`

private val OUTPUT = System.out

private const val BUFFER_SIZE = 1 shl 16

private val _buffer = ByteArray(BUFFER_SIZE)

private var _bufferPt = 0

private var _bytesRead = 0

private tailrec fun readChar(): Char {
    if (_bufferPt == _bytesRead) {
        _bufferPt = 0
        _bytesRead = INPUT.read(_buffer, 0, BUFFER_SIZE)
    }
    return if (_bytesRead < 0) Char.MIN_VALUE
    else {
        val c = _buffer[_bufferPt++].toInt().toChar()
        if (c == '\r') readChar() else c
    }
}

private fun readLine(): String? {
    var c = readChar()
    return if (c == Char.MIN_VALUE) null
    else buildString {
        while (c != '\n' && c != Char.MIN_VALUE) {
            append(c)
            c = readChar()
        }
    }
}

private fun readLn() = readLine()!! // string line
private fun readInt() = readLn().toInt() // single int
private fun readLong() = readLn().toLong() // single long
private fun readDouble() = readLn().toDouble() // single double
private fun readStrings() = readLn().split(" ") // list of strings
private fun readInts() = readStrings().map { it.toInt() } // list of ints
private fun readLongs() = readStrings().map { it.toLong() } // list of longs
private fun readDoubles() = readStrings().map { it.toDouble() } // list of doubles

private fun nextInt(): Int {
    var c = readChar()
    while (c != Char.MIN_VALUE && c <= ' ') {
        c = readChar()
    }
    if (c == Char.MIN_VALUE) throw NoSuchElementException()
    var sign = 1
    if (c == '-') {
        sign = -1
        c = readChar()
    }
    var res = 0
    while (c != Char.MIN_VALUE && c > ' ') {
        res = res * 10 + (c - '0')
        c = readChar()
    }
    return res * sign
}

private fun nextLong(): Long {
    var c = readChar()
    while (c != Char.MIN_VALUE && c <= ' ') {
        c = readChar()
    }
    if (c == Char.MIN_VALUE) throw NoSuchElementException()
    var sign = 1L
    if (c == '-') {
        sign = -1L
        c = readChar()
    }
    var res = 0L
    while (c != Char.MIN_VALUE && c > ' ') {
        res = res * 10L + (c - '0').toLong()
        c = readChar()
    }
    return res * sign
}

fun main() {
    val t = readInt()
    repeat(t) {
        // do something
    }
}
```
