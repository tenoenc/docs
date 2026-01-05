### 링커와 로더

우리가 작성한 소스 코드가 실제로 실행되기 위해서는 일련의 변환 과정을 거쳐야 한다. 운영체제는 프로그램을 실행하기 위해 디스크에 있는 이진 파일(Binary)을 메모리로 가져오고, 필요한 라이브러리를 연결해준다.

#### 프로그램의 빌드 및 실행 과정

프로그램은 일반적으로 다음과 같은 단계를 거쳐 실행 가능한 형태가 된다.



1. **소스 파일 (Source File):** 개발자가 작성한 고수준 언어 파일 (예: `main.c`).
2. **컴파일러 (Compiler):** 소스 코드를 기계어 코드로 변환하여 **오브젝트 파일(Object File)**을 생성한다. 이때 생성된 코드는 **재배치 가능(Relocatable)**하지만, 외부 함수나 라이브러리의 주소는 아직 알지 못한다.
3. **링커 (Linker):** 여러 개의 오브젝트 파일과 필요한 라이브러리 함수를 묶어서 하나의 **실행 파일(Executable File)**로 만든다.
4. **로더 (Loader):** 실행 파일을 메모리에 적재(Load)하고, CPU가 실행할 수 있도록 준비시킨다.

#### 링킹 (Linking)

링킹은 흩어진 코드 조각들을 하나로 모으는 과정이다.

- **정적 링킹 (Static Linking):** 프로그램이 사용하는 라이브러리 코드를 실행 파일 안에 그대로 포함시킨다.
    - **장점:** 실행 시 별도의 라이브러리 파일이 필요 없다.
    - **단점:** 실행 파일의 크기가 커지고, 라이브러리가 업데이트되면 프로그램을 다시 컴파일해야 한다.
- **동적 링킹 (Dynamic Linking):** 실행 파일에는 라이브러리를 호출할 위치 정보만 포함하고, 실제 라이브러리 코드는 실행 시(Runtime)에 메모리에 로드하여 연결한다.
    - **공유 라이브러리 (Shared Library):** 윈도우의 `.dll`, 리눅스의 `.so` 파일이 대표적이다. 여러 프로그램이 하나의 라이브러리 인스턴스를 메모리에서 공유하므로 메모리를 절약할 수 있다.

#### 로딩 (Loading)

로더는 디스크에 있는 이진 실행 파일을 메모리로 가져와 프로세스로 변환한다.

- **재배치 (Relocation):** 디스크에 저장된 코드는 논리적인 주소만 가지고 있다. 로더는 이 코드를 물리 메모리의 특정 위치에 올리면서, 코드 내의 주소들을 실제 물리 주소에 맞게 수정한다.
- **동적 로딩 (Dynamic Loading):** 프로그램의 모든 루틴을 한 번에 메모리에 올리지 않고, **해당 루틴이 호출될 때** 메모리에 적재한다. 사용되지 않는 코드는 메모리를 차지하지 않으므로 메모리 효율이 높다.

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
        // 편의상 내부 클래스를 사용하지만, 외부 .class 파일이어도 동일하게 동작함
        String className = "DynamicLoadingTest$DynamicPlugin"; 

        try {
            // [핵심] 이 시점에 JVM은 클래스 로더에게 해당 클래스를 찾아 메모리에 적재(Load)하라고 명령함
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

    // 테스트를 위해 내부에 static 클래스로 정의
    // 하지만 main 메서드에서는 이 클래스를 직접 참조(new DynamicPlugin)하지 않음!
    public static class DynamicPlugin {
        public DynamicPlugin() {
            System.out.println("   [System] DynamicPlugin Constructor Called! (Memory Allocated)");
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

> **백엔드 관점: C/C++의 링킹 vs Java의 ClassLoader**
>
> 일반적인 OS의 링킹/로딩 과정과 Java의 실행 방식은 결정적인 차이가 있습니다. 이를 비교하여 정리하는 것이 중요합니다.
>
> 1. **컴파일 결과물:** C언어는 OS가 바로 실행할 수 있는 `Native Code`로 컴파일되지만, Java는 `Bytecode(.class)`로 컴파일됩니다.
> 2. **링킹 시점:** C언어는 주로 실행 전(Build Time)에 링킹이 완료되지만, Java는 **런타임(Runtime)**에 링킹과 로딩이 일어납니다.
> 3. **ClassLoader의 마법:**
>     - Java는 `ClassLoader`가 필요한 클래스를 그때그때 동적으로 메모리(Method Area)에 로드합니다(**Dynamic Loading**).
>     - **장점:** 이 덕분에 Java 애플리케이션은 멈추지 않고도 새로운 기능을 추가(Plugin 시스템)하거나, 구현체를 갈아끼우는 유연성을 가집니다.
>     - **Spring Framework:** 스프링 빈(Bean)이 생성되고 의존성이 주입되는 과정도 결국 이 동적 로딩 메커니즘 위에서 동작하는 것입니다.
{: .block-tip }