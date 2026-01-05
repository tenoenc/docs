### 운영체제 설계 및 구현

운영체제는 매우 크고 복잡한 시스템이기 때문에, 설계 단계에서부터 **변경에 유연하게 대처할 수 있는 구조**를 잡는 것이 중요하다. 그중 가장 중요한 원칙이 바로 **정책과 메커니즘의 분리**다.

#### 정책(Policy)과 메커니즘(Mechanism)의 분리

이 원칙은 시스템을 설계할 때 **'무엇을(What)'** 할 것인지와 **'어떻게(How)'** 할 것인지를 명확히 구분하는 것이다.



##### 메커니즘 (Mechanism): 어떻게 할 것인가? (How)

어떤 기능을 수행하기 위한 **도구**나 **구현 그 자체**를 말한다. 하드웨어와 직접적으로 상호작용하거나 저수준의 연산을 수행하는 부분이다. 메커니즘은 자주 변하지 않는다.

- **예시**:
    - **문맥 교환(Context Switch):** CPU 레지스터 값을 저장하고 복구하는 기능 자체.
    - **타이머 인터럽트:** 일정 시간마다 신호를 보내는 하드웨어적 기능.
    - **메모리 보호:** 기준 레지스터와 한계 레지스터를 비교하여 접근을 차단하는 기능.

##### 정책 (Policy): 무엇을 할 것인가? (What)

주어진 메커니즘을 사용하여 **어떤 결정을 내릴지 정하는 규칙**이다. 사용자 요구사항이나 시스템 부하 상황에 따라 자주 변경될 수 있다.

- **예시**:
    - **CPU 스케줄링:** "문맥 교환(메커니즘)"을 이용해, "처리량이 많은 프로세스를 먼저 실행할 것인가, 응답 시간이 중요한 프로세스를 먼저 실행할 것인가?"를 결정.
    - **페이지 교체:** "페이지 교체(메커니즘)" 기능을 이용해, "LRU(가장 오래된 것)를 버릴 것인가, FIFO(먼저 들어온 것)를 버릴 것인가?"를 결정.

**분리의 장점**

정책과 메커니즘이 분리되어 있다면, 메커니즘(커널 코드)을 다시 작성하지 않고도 정책(파라미터 설정이나 모듈 교체)만 변경하여 시스템을 튜닝할 수 있다. 반대로 두 가지가 결합되어 있다면 정책을 바꿀 때마다 커널 전체를 다시 컴파일해야 하는 비효율이 발생한다.

#### [Practice] Dynamic Loading (Java) vs Static Linking (C)

C언어(Static Linking)는 `printf` 같은 라이브러리 코드를 실행 파일 안에 꽉 채워 넣습니다. 반면, Java는 프로그램이 실행되는 도중(Runtime)에 필요한 클래스 파일을 찾아서 메모리에 올리는 **동적 로딩(Dynamic Loading)** 방식을 사용합니다.

##### 1. 검증 코드 (Java)

`Class.forName()`을 사용하여, 컴파일 시점에는 존재조차 몰랐던 클래스(문자열로 이름 전달)를 런타임에 로딩하고 인스턴스화하는 과정을 확인합니다.

```java
import java.lang.reflect.Constructor;

public class DynamicLoadingTest {
    public static void main(String[] args) {
        System.out.println("=== 1. App Started (Plugin not loaded yet) ===");
        
        // 시나리오: 설정 파일이나 사용자 입력에서 클래스 이름을 문자열로 받음
        // 컴파일 타임에는 이 클래스의 존재를 모름 (import 안 함)
        String className = "DynamicPlugin"; 

        try {
            // 이 시점에 JVM은 파일시스템에서 'DynamicPlugin.class'를 찾아 메모리에 적재(Load)함
            System.out.println(">>> Loading class: " + className);
            Class<?> clazz = Class.forName(className);

            // 인스턴스 생성 (Reflection)
            Constructor<?> constructor = clazz.getDeclaredConstructor();
            Object plugin = constructor.newInstance();

            System.out.println("=== 2. Class Loaded & Instantiated ===");
            System.out.println("Instance: " + plugin.toString());

        } catch (ClassNotFoundException e) {
            System.err.println("Class not found! (Check classpath)");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 테스트를 위해 내부에 static 클래스로 정의 (실제로는 별도 파일로 존재할 수 있음)
    // 컴파일 시점에는 main 메서드가 이 클래스를 직접 참조하지 않음!
    public static class DynamicPlugin {
        public DynamicPlugin() {
            System.out.println("   [System] DynamicPlugin Constructor Called!");
        }

        @Override
        public String toString() {
            return "I am a dynamically loaded plugin!";
        }
    }
}
```

##### 2. 실행 결과 및 해석

-   **지연 로딩 (Lazy Loading):** `Class.forName()`이 호출되기 전까지 `HeavyPlugin` 클래스는 메모리에 존재하지 않습니다. 즉, **사용하지 않는 코드는 메모리를 점유하지 않는다**는 OS의 효율적 메모리 관리 원칙을 따릅니다.
-   **유연성:** 코드를 수정하고 재컴파일하지 않아도, 설정 파일(String)만 바꾸면 실행되는 객체를 교체할 수 있습니다. (예: JDBC Driver 로딩).

> **백엔드 관점: Spring Bean과 동적 로딩**
>
> 우리가 사용하는 **Spring Framework**의 근간이 바로 이 기술입니다. `@Component`가 붙은 클래스들을 스캔해서 동적으로 메모리에 올리고, 의존성 주입(DI)을 해줍니다. 만약 Java가 C언어처럼 정적 링킹만 지원했다면, 빈 하나를 추가할 때마다 서버 전체를 다시 빌드하고 링킹하는 고통을 겪었을 것입니다.
{: .block-tip }

> **백엔드 관점: 좋은 아키텍처의 황금률**
>
> 이 챕터는 백엔드 개발자가 **객체지향 설계(OOD)**를 할 때 가장 중요한 원칙과 맞닿아 있습니다.
>
> 1. **전략 패턴 (Strategy Pattern):** 변하지 않는 전체 흐름(Context/Mechanism)에서 알고리즘(Strategy/Policy)만 인터페이스로 분리하여 갈아 끼우는 패턴입니다.
> 2. **의존성 주입 (DI):** Spring Framework에서 `@Service` 로직(Policy)은 `@Repository` 구현체(Mechanism)가 무엇이든(JPA든 JDBC든) 상관없이 동작해야 합니다.
>
> **실무 적용:** "할인을 적용한다"는 기능(메커니즘)은 그대로 두고, "VIP는 10%, 신규 유저는 5%"라는 정책만 변경했을 때 코드를 얼마나 적게 수정해도 되는지가 좋은 설계의 척도입니다. OS 설계자가 고민했던 유연함의 미학을 우리 코드에도 적용해 보세요.
{: .block-tip }