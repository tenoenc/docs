### 선택 표현과 처리 (enum과 when)

#### enum 클래스 정의

```kotlin
enum class Color {
    RED, ORANGE, YELLOW, GREEN, BLUE, INDIGO, VIOLET
}
```

코틀린에서는 `enum class`를 사용하지만 자바에서는 `enum`을 사용한다. 코틀린에서 `enum`은 소프트 키워드라 부르는 존재다. `enum`은 `class` 앞에 있을 때는 특별한 의미를 지니지만 다른 곳에서는 이름에 사용할 수 있다.

자바와 마찬가지로 `enum`은 단순히 값만 열거하지 않는다. `enum` 클래스 안에 프로퍼티나 메서드를 정의할 수 있다.

```kotlin
enum class Color(
    val r: Int, val g: Int, val b: Int
) {
    RED(255, 0, 0), ORANGE(255, 165, 0), YELLOW(255, 255, 0),
    GREEN(0, 255, 0), BLUE(0, 0, 255), INDIGO(75, 0, 130),
    VIOLET(238, 130, 238);

    fun rgb() = (r * 256 + g) * 256 + b
}

fun main() {
    println(Color.BLUE.rgb()) // 255
}
```

`enum` 상수를 지정해야 할 때에는 그 상수에 해당하는 프로퍼티 값을 지정해야 한다. 또한, `enum` 클래스 안에 메서드를 정의하는 경우 반드시 `enum` 상수 목록과 메서드 정의 사이에 세미콜론(`:`)을 넣어야 한다.

#### when으로 enum 클래스 다루기

```kotlin
fun getMnemonic(color: Color) =
    when (color) {
        Color.RED -> "Richard"
        Color.ORANGE -> "Of"
        Color.YELLOW -> "York"
        Color.GREEN -> "Gave"
        Color.BLUE -> "Battle"
        Color.INDIGO -> "In"
        Color.VIOLET -> "Vain"
    }

fun main() {
    println(getMnemonic(Color.BLUE)) // Battle
}
```

`when`은 자바와 달리 각 분기의 끝에 `break`를 넣지 않아도 된다.

한 분기 안에서 여러 값을 매치 패턴으로 사용할 수도 있다. 그럴 경우 값 사이를 콤파(`,`)로 분리한다.

```koltin
fun getWarmth(color: Color) = when(color) {
    Color.RED, Color.ORANGE, Color.YELLOW -> "warm"
    Color.GREEN -> "neutral"
    Color.BLUE, Color.INDIGO, Color.VIOLET -> "cold"
}

fun main() {
    println(getWarmth(Color.ORANGE)) // warm
}
```

상수 값을 임포트하면 `enum` 클래스 수식자 없이 `enum`을 사용할 수 있다.

```kotlin
import ch02.ex11.Color
import ch02.ex11.Color.*

fun getWarmth(color: Color) = when(color) {
    RED, ORANGE, YELLOW -> "warm"
    GREEN -> "neutral"
    BLUE, INDIGO, VIOLET -> "cold"
}
```

#### when과 임의의 객체를 함께 사용

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(RED, YELLOW) -> ORANGE
        setOf(YELLOW, BLUE) -> GREEN
        setOf(BLUE, VIOLET) -> INDIGO
        else -> throw Exception("Dirty color")
    }

fun main() {
    println(mix(BLUE, YELLOW)) // GREEN
}
```

`when` 식은 인자 값과 매치하는 조건 값을 찾을 때까지 각 분기를 검사한다. 위 코드에서는 `Set`이라는 객체 사이를 매치할 때 동등성을 사용한다. 다른 객체에 대해서도 동일하게 동등성을 비교한다. `when`의 분기 조건 부분에 식을 넣을 수 있기 때문에 많은 경우 코드를 더 간결하게 작성할 수 있다.

#### 인자 없는 when 사용

앞의 코드에서는 `when`의 분기 조건에 있는 객체의 동등성을 비교하기 위해 인스턴스를 생성한다. 보통의 경우에는 문제 없지만, 만약 해당 함수가 자주 호출된다면 불필요한 가비지 객체가 늘어나게 된다. 인자 없는 `when`을 사용하면 불필요한 객체 생성을 막을 수 있다. 코드의 가독성은 떨어지지만 성능을 더 향상시키기 위해 감수해야 하는 경우도 있다.

```kotiln
fun mixOptimized(c1: Color, c2: Color) =
    when {
        (c1 == RED && c2 == YELLOW) ||
        (c1 == YELLOW && c2 == RED) ->
            ORANGE
        (c1 == YELLOW && c2 == BLUE) ||
        (c1 == BLUE && c2 == YELLOW) ->
            GREEN
        (c1 == BLUE && c2 == VIOLET) ||
        (c1 == VIOLET && c2 == BLUE) ->
            INDIGO
        else -> throw Exception("Dirty color")
    }

fun main() {
    println(mixOptimized(BLUE, YELLOW)) // GREEN
}
```

#### 스마트 캐스트 (타입 검사 + 타입 캐스트)

```kotlin
interface Expr
class Num(val value: Int): Expr
class Sum(val left: Expr, val right: Expr): Expr
```

`Expr` 인터페이스에는 두 가지 구현 클래스가 존재한다. 따라서 식을 평가하려면 두 가지 경우를 고려해야 한다.
- 어떤 식이 수이면 그대로 반환한다.
- 어떤 식이 합계라면 좌항과 우항의 값을 계산한 다음에 그 두 값을 합한 값을 반환한다.

```kotlin
fun eval(e: Expr): Int {
    if (e is Num) {
        val n = e as Num
        return n.value
    }
    if (e is Sum) {
        return eval(e.left) + eval(e.right)
    }
    throw IllegalArgumentException("Unknown expression")
}

fun main() {
    println(eval(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
}
```

코틀린에서는 `is`를 사용해 변수 타입을 검사한다. `is` 검사는 자바의 `instanceof`와 비슷하다. 하지만 자바에서는 어떤 변수의 타입을 `instanceof`로 확인한 다음에 명시적으로 그 타입에 속한 멤버에 접근하기 위해 변수 타입을 캐스팅해야 한다. 위 코드가 자바 스타일의 예이다.

그런데 타입을 확인한 이상 명시적인 타입 캐스팅은 불필요하다는 것을 알 수 있다. 코틀린에서는 어떤 변수가 원하는 타입인지 `is`로 검사하고 나면 컴파일러가 그 변수에 대해 캐스팅을 수행해준다. 이를 스마트 캐스트(smart cast)라고 부른다.

> 스마트 캐스트는 `is`로 변수에 든 값의 타입을 검사한 다음에 그 값이 바뀔 수 없는 경우에만 작동한다.
{: .block-warning }

#### 리팩토링: if를 when으로 변경

`if`는 식이라는 코틀린의 특성을 사용하면 앞의 함수를 더 간단하게 만들 수 있다.

```kotlin
fun eval(e: Expr): Int =
    if (e is Num) {
        e.value
    } else if (e is Sum) {
        eval(e.left) + eval(e.right)
    } else {
        throw IllegalArgumentException("Unknown expression")
    }

fun main() {
    println(eval(Sum(Num(1), Num(2)))) // 3
}
```

이 코드를 `when`을 사용해 더 다듬을 수 있다.

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.left) + eval(e.right)
        else -> throw IllegalArgumentException("Unknown expression")
    }
```

`when` 식을 앞에서 살펴본 값 동등성 검사가 아닌 다른 기능에도 쓸 수 있다. 앞의 예제와 마찬가지로 `is`로 타입을 검사하고 나면 스마트 캐스트가 이뤄진다.

#### if와 when의 분기에서 블록 사용

`if`나 `when`의 각 분기에서 수행해야 하는 로직이 복잡해지면 분기 본문에 블록을 사용할 수 있다. 각 분기를 블록으로 만들고 블록의 맨 마지막에 그 분기의 결과 값을 위치시키면 된다.

```kotlin
fun evalWithLogging(e: Expr): Int =
    when (e) {
        is Num -> {
            println("Num: ${e.value}")
            e.value
        }
        is Sum -> {
            val left =evalWithLogging(e.left)
            val right =evalWithLogging(e.right)
            println("sum: $left + $right")
            left + right
        }
        else -> throw IllegalArgumentException("Unknown expression")
    }

fun main() {
    println(evalWithLogging(Sum(Sum(Num(1), Num(2)), Num(4)))) // 7
    //Num: 1
    //Num: 2
    //sum: 1 + 2
    //Num: 4
    //sum: 3 + 4
    //7
}
```