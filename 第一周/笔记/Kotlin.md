# Kotlin

val不可变

var可变

$表引用，{}拿来在引用时当括号使用

## 1.数字

与一些其他语言不同，Kotlin 中的数字没有隐式拓宽转换。 例如，具有 `Double` 参数的函数只能对 `Double` 值调用，而不能对 `Float`、 `Int` 或者其他数字值调用：

```kotlin
fun main() {
    fun printDouble(d: Double) { print(d) }

    val i = 1    
    val d = 1.0
    val f = 1.0f 

    printDouble(d)
//    printDouble(i) // 错误：类型不匹配
//    printDouble(f) // 错误：类型不匹配
}
```

如需将数值转换为不同的类型，请使用[显式转换](https://book.kotlincn.net/text/numbers.html#显式数字转换)。

### 数字字面常量

数值常量字面值有以下几种:

- 十进制:

   

  ```
  123
  ```

  - Long 类型用大写 `L` 标记: `123L`

- 十六进制: `0x0F`

- 二进制: `0b00001011`

> Kotlin 不支持八进制。
>
> <svg width="24" height="24" fill="#4dbb5f" viewBox="0 0 24 24"><path d="M21 12a9 9 0 1 1-9-9 9 9 0 0 1 9 9zM10.5 7.5A1.5 1.5 0 1 0 12 6a1.5 1.5 0 0 0-1.5 1.5zm-.5 3.54v1h1V18h2v-6a.96.96 0 0 0-.96-.96z"></path></svg>

Kotlin 同样支持浮点数的常规表示方法:

- 默认 double：`123.5`、`123.5e10`
- Float 用 `f` 或者 `F` 标记: `123.5f`

你可以使用下划线使数字常量更易读(可参与运算)：

```kotlin
val oneMillion = 1_000_000
val creditCardNumber = 1234_5678_9012_3456L
val socialSecurityNumber = 999_99_9999L
val hexBytes = 0xFF_EC_DE_5E
val bytes = 0b11010010_01101001_10010100_10010010
```

### 引用数据表示

Java中的比如`resourse<Integer>`变为`resourse(Int?)`其他类似

### 显式数字转换

小的类型**不能** **隐式转换**为较大的类型。 这意味着把 `Byte` 型值赋给一个 `Int` 变量必须显式转换：

```java
val b: Byte = 1 // OK, 字面值会静态检测
// val i: Int = b // 错误
val i1: Int = b.toInt()
```

很多情况都不需要显式类型转换，因为类型会从上下文推断出来， 而算术运算会有重载做适当转换，例如：

```kotlin
val l = 1L + 3 // Long + Int => Long
```

### 整数除法

对于任何两个整数类型之间的除法总是返回整数。会丢弃任何小数部分。如需返回浮点类型，请将其中的一个参数显式转换为浮点类型

### 位运算

Kotlin 对整数提供了一组*位运算*。它们直接使用数字的比特表示在二进制级别进行操作。 位运算有可以通过中缀形式调用的函数表示。只能应用于 `Int` 与 `Long`：

```kotlin
val x = (1 shl 2) and 0x000FF000
```

这是完整的位运算列表：

- `shl(bits)` – 有符号左移
- `shr(bits)` – 有符号右移
- `ushr(bits)` – 无符号右移
- `and(bits)` – 位**与**
- `or(bits)` – 位**或**
- `xor(bits)` – 位**异或**
- `inv()` – 位非

### 浮点数比较

本节讨论的浮点数操作如下：

- 相等性检测：`a == b` 与 `a != b`
- 比较操作符：`a < b`、 `a > b`、 `a <= b`、 `a >= b`
- **区间实例以及区间检测：`a..b`、 `x in a..b`、 `x !in a..`**

当操作数 `a` 与 `b` 都是静态已知的 `Float` 或 `Double` 或者它们对应的可空类型，两数字所形成的操作或者区间遵循 [IEEE 754 浮点运算标准](https://zh.wikipedia.org/wiki/IEEE_754)。（标准规定NaN与谁进行比较都是false，包括自己）

为了支持泛型场景并提供全序支持，对于**并非**静态类型就是浮点数的情况，行为是不同的。

```kotlin
fun main() {
    //sampleStart
    // 静态类型作为浮点数的操作数
    println(Double.NaN == Double.NaN)// false，比较的是NaN，NaN的规则是与谁都是false
    // 静态类型并非作为浮点数的操作数
    // 所以 NaN 等于它本身
    println(listOf(Double.NaN) == listOf(Double.NaN)) // true，比较的是list，运用的是list规定的比较，这里只看括号内一不一样

    // 静态类型作为浮点数的操作数
    println(0.0 == -0.0) // true，比较的是0.0的规则，此时两者相等
    // 静态类型并非作为浮点数的操作数
    // 所以 -0.0 小于 0.0
    println(listOf(0.0) == listOf(-0.0))              // false

    println(listOf(Double.NaN, Double.POSITIVE_INFINITY, 0.0, -0.0).sorted())
    // [-0.0, 0.0, Infinity, NaN]
    //sampleEnd
}
```

