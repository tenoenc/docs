## 제어의 역전과 의존성 주입

### 프레임워크 설계 철학
#### 제어의 역전 (IoC)
#### 의존성 주입 (DI)
#### 서비스 로케이터 패턴 비교
#### 의존성 부패와 할리우드 원칙

### BeanFactory 인터페이스 계층
#### 레지스트리 기반 구조
##### AliasRegistry
##### SimpleAliasRegistry
##### SingletonBeanRegistry
##### DefaultSingletonBeanRegistry
#### 팩토리 핵심 규격
##### BeanFactory
##### HierarchicalBeanFactory
##### ListableBeanFactory
##### ConfigurableBeanFactory
##### AutowireCapableBeanFactory
#### 완성형 엔진 구현체
##### DefaultListableBeanFactory

### ApplicationContext 인터페이스 구조
#### 다중 상속 기반 확장 기능
##### MessageSource
##### ApplicationEventPublisher
##### ResourcePatternResolver
#### 설정 관리 및 생명주기 제어
##### ConfigurableApplicationContext
##### AbstractApplicationContext
#### 런타임 컨텍스트 구현체
##### GenericApplicationContext
##### AnnotationConfigApplicationContext
##### ClassPathXmlApplicationContext

### 빈 설정 메타데이터 (BeanDefinition)
#### 메타데이터 추상화 규격
##### BeanDefinition 인터페이스
##### AttributeAccessor
##### BeanMetadataElement
#### 빈 정의 핵심 구현체
##### AbstractBeanDefinition
##### RootBeanDefinition
##### ChildBeanDefinition
##### GenericBeanDefinition
#### 설정 정보 로딩 아키텍처
##### BeanDefinitionReader
##### AnnotatedBeanDefinitionReader
##### XmlBeanDefinitionReader
##### BeanDefinitionDocumentReader

### 의존성 주입 매커니즘
#### 주입 유형 및 특성
##### 생성자 주입 (Constructor Injection)
##### 수정자 주입 (Setter Injection)
##### 필드 주입 (Field Injection)
##### 메서드 주입 (Method Injection)
#### 의존성 해결 규약
##### 타입 기반 매칭 (By Type)
##### 이름 기반 매칭 (By Name)
##### 우선순위 결정 규칙

### 자동 와이어링 및 한정 기법
#### 오토와이어링 전략 모드
##### Autowire.NO
##### Autowire.BY_NAME
##### Autowire.BY_TYPE
##### Autowire.CONSTRUCTOR
#### 의존성 선택 한정자
##### @Primary (우선권 지정)
##### @Qualifier (한정자 식별)
##### @Priority (표준 우선순위)
#### 컬렉션 및 옵셔널 주입
##### List/Map 인터페이스 주입
##### ObjectProvider/ObjectFactory
##### Optional 기반 주입

### 컴포넌트 스캔 및 어노테이션 프로세싱
#### 스캐닝 메커니즘
##### ClassPathBeanDefinitionScanner
##### ComponentIndexRepository
#### 스캔 필터링 규약
##### TypeFilter 인터페이스
##### AnnotationTypeFilter
##### AssignableTypeFilter
#### 설정 클래스 파싱 아키텍처
##### ConfigurationClassPostProcessor
##### ConfigurationClassParser
##### ConfigurationClassBeanDefinitionReader
#### 핵심 어노테이션 프로세서
##### AutowiredAnnotationBeanPostProcessor
##### CommonAnnotationBeanPostProcessor

### 순환 참조 및 지연 주입
#### 순환 의존성 처리 메커니즘
##### 3단계 싱글톤 캐시 계층
##### 얼리 빈 레퍼런스 (Early Bean Reference)
#### 지연 로딩 전략
##### @Lazy 프록시 기반 주입
##### ProxyFactory 기반 타겟 지연

### 빈 스코프 관리 (Bean Scopes)
#### 표준 스코프 전략
##### Singleton (공유 인스턴스)
##### Prototype (독립 인스턴스)
#### 웹 특화 스코프
##### Request 스코프
##### Session 스코프
##### Application 스코프
#### 스코프 프록시 메커니즘
##### ScopedProxyMode
##### ScopedProxyFactoryBean
#### 스코프 확장 인터페이스
##### Scope 인터페이스
##### CustomScopeConfigurer

### 메서드 주입 및 협력 객체 제어
#### 룩업 메서드 주입 (Lookup Method)
##### @Lookup 어노테이션
##### CGLIB 서브클래싱 기반 구현
#### 메서드 교체 (Method Replacement)
##### MethodReplacer 인터페이스
##### 타겟 메서드 런타임 치환

### 커스텀 팩토리 및 프로그래밍적 등록
#### FactoryBean 아키텍처
##### FactoryBean 인터페이스
##### AbstractFactoryBean
##### & 연산자 (Dereferencing)
#### 프로그래밍적 빈 정의 등록
##### AnnotatedBeanDefinitionReader
##### BeanDefinitionCustomizer
##### GenericApplicationContext.registerBean

## 빈 정의와 생명주기 관리

### 생명주기 인터페이스 규약
#### 초기화 콜백 (Initialization)
##### InitializingBean
##### @PostConstruct
##### 커스텀 init-method 명세
#### 소멸 콜백 (Destruction)
##### DisposableBean
##### @PreDestroy
##### 커스텀 destroy-method 명세

### 인식 인터페이스 (Aware Interfaces)
#### 컨테이너 자원 인식 규약
##### BeanNameAware
##### BeanClassLoaderAware
##### BeanFactoryAware
#### 인프라스트럭처 인식 규약
##### ApplicationContextAware
##### EnvironmentAware
##### ResourceLoaderAware
##### ApplicationEventPublisherAware
##### MessageSourceAware

### 빈 포스트 프로세서 (BeanPostProcessor)
#### 객체 초기화 확장 규약
##### postProcessBeforeInitialization (초기화 전 처리)
##### postProcessAfterInitialization (초기화 후 처리)
#### 인스턴스화 단계 제어
##### InstantiationAwareBeanPostProcessor
##### SmartInstantiationAwareBeanPostProcessor
#### 소멸 단계 확장
##### DestructionAwareBeanPostProcessor

### 빈 팩토리 포스트 프로세서 (BeanFactoryPostProcessor)
#### 설정 메타데이터 조작 규약
##### postProcessBeanFactory
#### 빈 정의 등록 확장
##### BeanDefinitionRegistryPostProcessor
##### postProcessBeanDefinitionRegistry

### 실행 우선순위 및 정렬
#### 정렬 규약 인터페이스
##### Ordered
##### PriorityOrdered
#### 어노테이션 기반 정렬
##### @Order
##### @Priority

### 스마트 라이프사이클 및 실행 그룹
#### 객체 생명주기 핵심 규약
##### Lifecycle 인터페이스
##### SmartLifecycle 확장 규약
#### 단계별 실행 제어
##### Phased 인터페이스
##### 단계(Phase) 값에 따른 실행 순서
##### 자동 시작 (isAutoStartup)

### 컨테이너 생명주기 프로세싱
#### 프로세서 아키텍처
##### LifecycleProcessor
##### DefaultLifecycleProcessor
#### 라이프사이클 트리거
##### 컨테이너 시작 (onRefresh)
##### 컨테이너 종료 (onClose)

### 시스템 종료 및 자원 해제
#### 셧다운 매커니즘
##### JVM 셧다운 후크 (Shutdown Hook)
##### Closeable 및 AutoCloseable 처리
#### 우아한 종료 (Graceful Shutdown)
##### 종료 단계별 타임아웃 제어

## 리소스 추상화 및 데이터 바인딩

### 자원 접근 추상화 (Resources)
#### 리소스 핵심 인터페이스
##### Resource
##### InputStreamSource
#### 저장소별 리소스 구현체
##### UrlResource
##### ClassPathResource
##### FileSystemResource
##### BeanDefinitionResource

### 리소스 로딩 및 경로 분석
#### 자원 로더 규약
##### ResourceLoader
##### DefaultResourceLoader
#### 경로 패턴 해석기
##### ResourcePatternResolver
##### PathMatchingResourcePatternResolver
#### 위치 경로 접두사 (Prefix)
##### classpath: 및 classpath*:
##### file: 및 http:

### 검증 및 데이터 바인딩
#### 데이터 유효성 검증
##### Validator 인터페이스
##### Errors 및 BindingResult
##### SmartValidator 확장
#### 빈 속성 조작 및 바인딩
##### BeanWrapper 인터페이스
##### BeanWrapperImpl 구현체
##### DataBinder 아키텍처
##### PropertyValues 및 MutablePropertyValues

### 타입 변환 시스템 (Type Conversion)
#### 기존 프로퍼티 에디터
##### PropertyEditor 인터페이스
##### PropertyEditorSupport
#### 현대적 타입 변환 SPI
##### Converter 인터페이스
##### ConverterFactory
##### GenericConverter 및 ConditionalConverter
#### 변환 서비스 레지스트리
##### ConversionService 인터페이스
##### GenericConversionService
##### DefaultConversionService

### 포맷팅 SPI (Formatting)
#### 필드 포맷팅 규약
##### Formatter 인터페이스
##### Printer 및 Parser
#### 어노테이션 기반 포맷팅
##### AnnotationFormatterFactory
##### @DateTimeFormat 처리 매커니즘
##### @NumberFormat 처리 매커니즘
#### 통합 변환 서비스
##### FormatterRegistry
##### FormattingConversionService

## 관점 지향 프로그래밍 (AOP)

### AOP 핵심 추상화 및 용어
#### 조인 포인트 (Join Point) 규약
##### JoinPoint 인터페이스
##### ProceedingJoinPoint
#### 어드바이스 (Advice) 모델
##### MethodBeforeAdvice
##### AfterReturningAdvice
##### ThrowsAdvice
##### MethodInterceptor (Around)
#### 포인트컷 (Pointcut) 설계
##### Pointcut 인터페이스
##### ClassFilter
##### MethodMatcher
#### 어드바이저 (Advisor) 아키텍처
##### Advisor 인터페이스
##### PointcutAdvisor
##### IntroductionAdvisor

### 프록시 생성 및 구성 아키텍처
#### 프록시 팩토리 계층
##### ProxyFactory
##### ProxyFactoryBean
##### AspectJProxyFactory
#### 저수준 프록시 생성 전략
##### AopProxy 인터페이스
##### JdkDynamicAopProxy
##### ObjenesisCglibAopProxy
#### 프록시 설정 및 타겟 제어
##### ProxyConfig
##### Advised 인터페이스
##### TargetSource 추상화

### 자동 프록시 생성 아키텍처
#### 자동 프록시 생성기 (Auto-Proxy Creator)
##### AbstractAutoProxyCreator
##### DefaultAdvisorAutoProxyCreator
##### AnnotationAwareAspectJAutoProxyCreator
#### 선언적 구성 및 활성화
##### @EnableAspectJAutoProxy
##### AspectJAutoProxyRegistrar
#### 프록시 노출 제어
##### AopContext (ThreadLocal 기반 프록시 참조)

### AspectJ 통합 및 포인트컷 해석
#### AspectJ 어노테이션 아키텍처
##### @Aspect 매커니즘
##### AspectJAdvisorFactory
##### ReflectiveAspectJAdvisorFactory
#### 포인트컷 표현식 엔진
##### PointcutExpression 인터페이스
##### AspectJExpressionPointcut
##### 포인트컷 문법 파싱 (PointcutParser)
#### 어스펙트 인스턴스 팩토리
##### AspectInstanceFactory
##### MetadataAwareAspectInstanceFactory

### 인트로덕션 및 런타임 인터셉션
#### 인트로덕션 아키텍처
##### IntroductionAdvisor
##### IntroductionInterceptor
##### DelegatingIntroductionInterceptor
#### 어드바이스 체인 관리
##### AdvisorChainFactory
##### DefaultAdvisorChainFactory
##### MethodInvocation 규약
##### ReflectiveMethodInvocation

## 스프링 표현 언어 (SpEL)

### SpEL 핵심 API 아키텍처
#### 표현식 파서 규약
##### ExpressionParser
##### TemplateAwareExpressionParser
##### SpelExpressionParser
#### 표현식 데이터 모델
##### Expression 인터페이스
##### SpelExpression
##### ParserContext (템플릿 파싱 전략)

### 평가 컨텍스트 및 리졸버 아키텍처
#### 실행 컨텍스트 규약
##### EvaluationContext
##### StandardEvaluationContext
##### SimpleEvaluationContext (보안 제한형)
#### 자원 식별 및 해석기
##### PropertyAccessor
##### MethodResolver
##### ConstructorResolver
##### TypeLocator

### 추상 구문 트리 및 파싱 엔진
#### 구문 트리 구성 요소
##### SpelNode 인터페이스
##### 연산자 및 리터럴 노드 계층
#### 내부 파서 메커니즘
##### InternalSpelExpressionParser
##### Tokenizer 및 Scanner

### 확장 해석 및 성능 최적화
#### 외부 자원 리졸버
##### FunctionResolver (정적 함수 등록)
##### BeanResolver (컨테이너 빈 참조)
#### 실행 성능 최적화
##### SpelCompiler (런타임 바이트코드 생성)
##### SpelCompilerMode (OFF/IMMEDIATE/MIXED)

## 환경 추상화 및 외부 설정

### 환경 추상화 (Environment)
#### 환경 정보 핵심 규약
##### Environment 인터페이스
##### ConfigurableEnvironment
#### 실행 환경별 구현체
##### StandardEnvironment
##### StandardServletEnvironment

### 프로파일 (Profiles)
#### 프로파일 제어 규약
##### 활성 프로파일 (Active Profiles)
##### 기본 프로파일 (Default Profiles)
#### 조건부 등록 매커니즘
##### @Profile 어노테이션
##### ProfilePredicate

### 프로퍼티 소스 및 해석 아키텍처
#### 프로퍼티 소스 규약
##### PropertySource 추상 클래스
##### EnumerablePropertySource
##### MapPropertySource
##### CompositePropertySource (복합 소스 관리)
#### 프로퍼티 해석기 계층
##### PropertyResolver 인터페이스
##### ConfigurablePropertyResolver
##### PropertySourcesPropertyResolver
#### 설정값 보간 및 플레이스홀더
##### PropertyPlaceholderHelper
##### StringValueResolver

### 소스 관리 및 우선순위 규약
#### 소스 컬렉션 관리
##### MutablePropertySources
##### 우선순위 기반 탐색 순서
#### 어노테이션 기반 소스 등록
##### @PropertySource
##### PropertySourceFactory 인터페이스
##### DefaultPropertySourceFactory

### 설정값 주입 및 바인딩 아키텍처
#### 플레이스홀더 처리기
##### PropertySourcesPlaceholderConfigurer
##### BeanDefinitionValueResolver
#### 런타임 값 해석
##### EmbeddedValueResolverAware
##### @Value 어노테이션 프로세싱

## 컴포넌트 이벤트 및 메시지 통지

### 애플리케이션 이벤트 규약 (Events)
#### 이벤트 객체 추상화
##### ApplicationEvent
##### PayloadApplicationEvent (제네릭 데이터 기반)
#### 이벤트 리스너 인터페이스
##### ApplicationListener
##### SmartApplicationListener (조건부 리스너)
##### GenericApplicationListener

### 이벤트 발행 및 멀티캐스팅
#### 이벤트 발행 규약
##### ApplicationEventPublisher
#### 이벤트 방송 매커니즘
##### ApplicationEventMulticaster
##### SimpleApplicationEventMulticaster
#### 비동기 이벤트 실행 전략
##### TaskExecutor 기반 전파

### 어노테이션 기반 이벤트 처리
#### 리스너 등록 프로세서
##### EventListenerMethodProcessor
##### EventListenerFactory
#### 선언적 리스너 제어
##### @EventListener
##### @TransactionalEventListener

### 국제화 및 메시지 처리 (MessageSource)
#### 메시지 해석 규약
##### MessageSource
##### HierarchicalMessageSource
#### 메시지 소스 구현체
##### ResourceBundleMessageSource
##### ReloadableResourceBundleMessageSource

## 컨테이너 확장 및 프레임워크 전략

### 애플리케이션 컨텍스트 초기화 규약
#### 컨텍스트 초기화 인터페이스
##### ApplicationContextInitializer
##### DelegatingApplicationContextInitializer
#### 팩토리 로딩 매커니즘
##### SpringFactoriesLoader (SPI 확장 모델)

### 설정 및 등록 확장 전략
#### 동적 설정 주입 (Import)
##### @Import 매커니즘
##### ImportSelector (조건부 선택)
##### DeferredImportSelector (그룹화 및 지연 선택)
#### 프로그래밍적 빈 레지스트리 제어
##### ImportBeanDefinitionRegistrar
#### 설정 오버라이딩 전략
##### 빈 정의 오버라이딩 규약 (AllowBeanDefinitionOverriding)

### 컴포넌트 인덱싱 및 필터링 최적화
#### 인덱스 기반 스캔 전략
##### CandidateComponentsIndex
##### CandidateComponentsIndexLoader
#### 메타-어노테이션 엔진
##### MergedAnnotations API
##### AnnotationFilter 규약

### 조건부 설정 아키텍처
#### 실행 조건 판정 규약
##### Condition 인터페이스
##### ConfigurationCondition
##### ConditionEvaluator
#### 빈 등록 제어 전략
##### @Conditional 매커니즘

## 테스트 컨텍스트 인프라스트럭처

### 테스트 프레임워크 핵심 추상화
#### 테스트 컨텍스트 관리 규약
##### TestContextManager
##### TestContext 인터페이스
#### 컨텍스트 로딩 전략
##### ContextLoader
##### SmartContextLoader
##### AbstractContextLoader
##### AnnotationConfigContextLoader

### 테스트 실행 리스너 및 확장
#### 리스너 규약 및 공통 구현
##### TestExecutionListener
##### AbstractTestExecutionListener
#### 핵심 기능 리스너
##### DependencyInjectionTestExecutionListener
##### DirtiesContextTestExecutionListener
##### TransactionalTestExecutionListener
##### SqlScriptsTestExecutionListener

### 컨텍스트 캐싱 및 성능 최적화
#### 컨텍스트 캐시 메커니즘
##### ContextCache
##### DefaultContextCache
#### 캐시 키 생성 규약
##### MergedContextConfiguration
##### HierarchyMode (컨텍스트 계층 모드)

### 모킹 및 가상 환경 인프라
#### 가상 서블릿 환경 추상화
##### MockServletContext
##### MockHttpServletRequest
##### MockHttpServletResponse
#### 모킹 통합 프로세서
##### MockitoPostProcessor
##### MockDefinition 및 SpyDefinition

### 테스트 설정 및 프로퍼티 전략
#### 동적 환경 구성
##### @ActiveProfiles 매커니즘
##### @TestPropertySource 해석기
##### DynamicPropertyRegistry (동적 프로퍼티 등록)

### 테스트 트랜잭션 및 자원 제어
#### 테스트 전용 트랜잭션 관리
##### TestTransaction (상태 제어 API)
##### TransactionalTestExecutionListener
#### 데이터 스크립트 실행 매커니즘
##### @Sql 및 @SqlConfig 해석기
##### SqlScriptsTestExecutionListener

## Ahead-of-Time 및 네이티브 최적화

### AOT 컴파일 및 코드 생성 엔진
#### AOT 처리 확장 규약
##### BeanRegistrationAotProcessor
##### BeanFactoryAotProcessor
#### 소스 코드 생성 및 기여 매커니즘
##### GeneratedClass (생성 클래스 추상화)
##### GeneratedMethod (정적 빈 등록 로직)
##### RuntimeHints (네이티브 힌트 수집기)

### 런타임 힌트 및 정적 분석 (Runtime Hints)
#### 네이티브 메타데이터 규약
##### RuntimeHints
##### RuntimeHintsRegistrar (사용자 정의 등록)
#### 항목별 접근 명세
##### ReflectionHints (리플렉션 허용 항목)
##### ProxyHints (JDK 동적 프록시 허용 항목)
##### ResourceHints (번들 및 리소스 포함 항목)
##### SerializationHints (Java 직렬화 허용 항목)

### 네이티브 실행 및 빌드 통합
#### AOT 실행 환경 감지
##### AotDetector (AOT 모드 판별기)
#### 빌드 시점 기여 (Contributions)
##### BeanFactoryInitializationAotContribution
##### ProcessedBindableMethod (바인딩 최적화)
#### 네이티브 구성 전략
##### NativeConfiguration
##### GraalVM Reachability Metadata 통합