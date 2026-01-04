### 클래스와 프로퍼티

시작하기 위해 간단한 자바빈 클래스인 `Person`을 정의하자. `Person`에는 `name`이라는 프로퍼티만 들어있다.

```java
public class Person {
    private final String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

필드가 둘 이상으로 늘어나면 생성자인 `Person(String name)`의 본문에서 파라미터를 이름이 같은 필드에 대입하는 대입문의 수도 늘어난다. 코틀린에서는 이런 필드 대입 로직을 훨씬 더 적은 코드로 작성할 수 있다.

```kotlin
class Person(val name: String)
```

이런 유형의 클래스(코드 없이 데이터만 저장하는 클래스)를 **값 객체**(Value Object)라 부르며, 다양한 언어가 값 객체를 간결하게 기술할 수 있는 구문을 제공한다.

무엇이 바뀌었을까? `public` 가시성 변경자가 사라졌다. 코틀린의 기본 가시성은 `public`이므로 이런 경우 변경자를 생략해도 된다.

#### 프로퍼티

클래스라는 개념의 목적은 **데이터를 캡슐화**하고 캡슐화한 데이터를 다루는 코드를 **한 주체 아래 가두는 것**이다.

자바에서는 필드와 접근자를 한데 묶어 **프로퍼티**(property)라 부른다. 코틀린은 프로퍼티를 언어 기본 기능으로 제공하며, 코틀린 프로퍼티는 자바의 필드와 접근자 메서드를 완전히 대신한다. 클래스에서 프로퍼티를 선언할 떄는 변수를 선언하는 방법과 마찬가지로 `val`이나 `var`를 사용한다.

```kotlin
class Person(
    val name: String,
    var isMarried: Boolean
)
```

코틀린에서 프로퍼티를 선언하는 방식은 프로퍼티와 관련 있는 접근자를 선언하는 것이다. 코틀린은 값을 저장하기 위한 **비공개 필드**와 그 필드에 값을 저장하기 위한 **세터**, 필드의 값을 읽기 위한 **게터**로 이뤄진 간단한 **디폴트 접근자 구현을 제공**한다.

**자바와 코틀린의 실제 코드는 같다**

위 `Person` 클래스 정의 뒤에는 원래의 자바 코드와 똑같은 구현이 숨어있다. `Person`에는 비공개 필드가 들어있고, 생성자가 그 필드를 초기화하며, 게터를 통해 그 비공개 필드에 접근한다. 이는 자바 클래스와 코틀린 클래스를 동일한 방식으로 사용할 수 있다는 뜻이다(**상호운용성**).

```java
Person person = new Person("Bob", true);
System.out.println(person.getName()); // Bob
System.out.println(person.isMarried()); // true
```

**게터 및 세터의 이름 규칙**

자바와 코틀린에서 정의한 `Person` 중 어느 쪽을 사용해도 이 코드를 바꿀 필요가 없다. 게터와 세터의 이름을 정하는 규칙에는 에외가 있다. 이름이 `is`로 시작하는 프로퍼티의 게터에는 `get`이 붙지 않고 원래 이름(`isMarried`)을 그대로 사용하며, 세터에는 `is`를 `set`으로 바꾼 이름(`setMarried`)을 사용한다.

```kotlin
val person = Person("Bob", true)
println(person.name) // Bob
println(person.isMarried) // true
```

> 자바에서 선언한 클래스에 대해 코틀린 문법을 사용해도 된다. 예를 들어 자바가 `isMarried`에 대한 게터, 세터를 제공한다면 그에 상응하는 코틀린 프로퍼티의 이름은 `isMarried`이며 `var` 프로퍼티처럼 사용할 수 있다.
{: .block-tip }

#### 커스텀 접근자

대부분의 프로퍼티에는 그 프로퍼티의 값을 저장하기 위한 필드가 있다. 이를 프로퍼티를 **뒷받침하는 필드**(backing field)라고 부른다. 하지만 원한다면 프로퍼티 값을 그때그때 계산할 수도 있다. 커스텀 게터를 작성하면 그런 프로퍼티를 만들 수 있다.

```kotlin
class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}
```

`isSquare` 프로퍼티에는 자체 값을 저장하는 필드가 필요 없다. 클라이언트가 프로퍼티에 접근할 때마다 게터가 프로퍼티 값을 매번 다시 계산한다. 이 접근자를 자바에서 사용하려면 `isSquare()` 메서드를 호출하면 된다.

**파라미터가 없는 함수 vs 커스텀 게터**

구현이나 성능상의 차이는 없다. 차이가 나는 부분은 가독성뿐이다.

#### 코틀린 소스코드 구조 (디렉터리와 패키지)

모든 코틀린 파일의 맨 앞에 `package`문을 넣을 수 있다. 그러면 그 파일 안에 있는 모든 선언(클래스, 함수, 프로퍼티 등)이 해당 패키지에 들어간다. **같은 패키지**에 속해 있다면 **다른 파일**에서 정의한 선언일지라도 **직접 사용할 수 있다**. 반면 **다른 패키지**에 정의한 선언을 사용하려면 임포트를 통해 **선언을 불러와야 한다**.

```kotlin
package ch02.ex08 // 패키지 선언

import java.util.Random // 표준 자바 라이브러리 클래스 임포트

class Rectangle(val height: Int, val width: Int) {
    val isSquare: Boolean
        get() {
            return height == width
        }
}

fun createRandomRectangle(): Rectangle {
    val random = Random()
    return Rectangle(random.nextInt(), random.nextInt())
}
```

**클래스와 함수의 임포트에 차이가 없다**

코틀린에서는 클랫 임포트와 함수 임포트에 차이가 없으며, 모든 선언을 import 키워드로 가져올 수 있다. 최상위 함수는 그 이름을 써서 임포트할 수 있다.

```kotlin
package ch02.ex09

import ch02.ex08.createRandomRectangle

fun main(args: Array<String>) {
    println(createRandomRectangle().isSquare)
}
```

**모든 선언 임포트**

패키지 이름 뒤에 `.*`을 추가하면 패키지 안의 모든 선언을 임포트할 수 있다.

> 이런 스타 임포트를 사용하면 패키지 안에 있는 모든 클래스뿐 아니라 최상위에 정의된 함수나 프로퍼티까지 불러온다는 점을 주의해야 한다.
{: .block-warning }

**코틀린은 디렉터리 계층 구조에서 자유롭다**

자바에서는 패키지의 구조와 일치하는 디렉터리 계층 구조를 만들고 클래스의 소스코드를 그 클래스가 속한 패키지와 같은 디렉터리에 위치시켜야 한다. 반면 코틀린에서는 여러 클래스를 한 파일에 넣을 수 있고, 파일의 이름도 마음대로 정할 수 있다. 또한, 디스크상의 어느 디렉터리에 소스코드 파일을 위치시키든 상관없다.

**패키지별로 디렉터리를 구성하되, 한 파일에 하나의 클래스만 넣을 필요는 없다**

하지만 대부분의 경우 **자바와 같이 패키지별로 디렉터리를 구성하는 편이 낫다.** 특히 자바와 코틀린을 함께 사용하는 프로젝트에서는 자바의 방식을 따르는 게 중요하다. 자바 클래스를 코틀린 클래스로 마이그레이션할 때 문제가 생길 수도 있기 때문이다. 그렇다고 **여러 클래스를 한 파일에 넣는 것을 주저해서는 안 된다**. 특히 각 클래스를 정의하는 소스코드 크기가 아주 작은 경우 더욱 그렇다.