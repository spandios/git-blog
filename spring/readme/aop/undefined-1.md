# 빈 후처리기

앞서 살펴본 프록시 팩토리 덕에 이제 프록시를 편하게 생성할 수 있으며 부가기능 로직과 적용 대상 로직을 명확히 구분해 이해가 쉽고 유지보수하기 쉬워졌다.&#x20;

하지만 프록시 팩토리만으로는 많은 설정 코드가 필요하다. 각 빈 마다 프록시 팩토리 관련 코드를 중복해서 설정해야 한다.&#x20;

<details>

<summary>프록시 팩토리만을 사용해 빈을 등록하는 코드</summary>

```java
@Slf4j
@Configuration
public class ProxyFactoryConfig {
    @Bean
    public OrderController orderController(LogTrace logTrace) {
        OrderController orderController = newOrderControllerImpl(orderService(logTrace));
        ProxyFactory factory = new ProxyFactory(orderController); // 매번 팩토리 관련 설정 코드가 필요 
        factory.addAdvisor(getAdvisor(logTrace));
        OrderController proxy = (OrderController) factory.getProxy();
        return proxy;
    }
    
    @Bean
    public OrderService orderService(LogTrace logTrace) {
        OrderService orderService = new OrderServiceImpl(orderRepository(logTrace));
        ProxyFactory factory = new ProxyFactory(orderService); // 매번 팩토리 관련 설정 코드가 필요 
        factory.addAdvisor(getAdvisor(logTrace));
        OrderService proxy = (OrderService) factory.getProxy();
        return proxy;
    }
    
    @Bean
    public OrderRepository orderRepository(LogTrace logTrace) {
        OrderRepository orderRepository = new OrderRepositoryImpl();
        ProxyFactory factory = new ProxyFactory(orderRepository); // 매번 팩토리 관련 설정 코드가 필요 
        factory.addAdvisor(getAdvisor(logTrace));
        OrderRepository proxy = (OrderRepository) factory.getProxy();
        return proxy;
    }
        
     private Advisor getAdvisor(LogTrace logTrace) {
          //pointcut
          NameMatchMethodPointcut pointcut = new NameMatchMethodPointcut();
          pointcut.setMappedNames("request*", "order*", "save*");
          //advice
          LogTraceAdvice advice = new LogTraceAdvice(logTrace);
          //advisor = pointcut + advice
          return new DefaultPointcutAdvisor(pointcut, advice);
     }
}
 
```

</details>



이 때 빈 후처리기라는 기술을 통해 문제를 해결 할 수 있다.&#x20;

## 빈 후처리기 (BeanPostProcessor)

빈이 등록할 때의 이벤트를 받는 hook이다. 이곳에서 전달된 원본의 빈을 조작하거나 다른 객체로 바꿔치기해 빈을 등록할 수 있다.&#x20;



```java
class AToBPostProcessor implements BeanPostProcessor {
        @Override
        public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (bean instanceof A) {
            return new B();
        }
        return bean;
}
 
```

그렇다면 이 기술을 통해 위에서 중복되어 나타나는 코드, 프록시 객체를 생성한 것을 원본 객체와 교체해 빈으로 등록하는 코드를 훅에 적용하기만 한다면 중복을 제거할 수 있을 것이다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (13).svg" alt="" class="gitbook-drawing">

다음은 빈 등록 시 프록시 적용 대상이면 프록시 팩토리를 통해 생성한 프록시를 반환해 빈으로 등록하고, 적용 대상이 아니라면 원본 대상을 그대로 빈으로 등록하는 코드이다.

```java
public class PackageLogTraceProxyPostProcessor implements BeanPostProcessor {
    private final String basePackage;
    private final Advisor advisor;

    public PackageLogTraceProxyPostProcessor(String basePackage, Advisor
            advisor) {
        this.basePackage = basePackage;
        this.advisor = advisor;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName)
            throws
            BeansException {
        //프록시 적용 대상 여부 체크
        //프록시 적용 대상이 아니면 원본을 그대로 반환
        String packageName = bean.getClass().getPackageName();
        if (!packageName.startsWith(basePackage)) {
            return bean;
        }
        //프록시 대상이면 프록시를 만들어서 반환
        ProxyFactory proxyFactory = new ProxyFactory(bean);
        proxyFactory.addAdvisor(advisor);
        Object proxy = proxyFactory.getProxy();
        
        return proxy;
    }
}
```



## AutoProxyCreator

위에서는 특정한 pacakge에 있는 조건을 가지고 프록시 적용 대상을 찾았다.&#x20;

그런데 우리는 이미 포인트컷이 메서드 단위 뿐 아니라 클래스도 필터 기능을 가지고 있다는 걸 알고 있다.&#x20;

**포인트 컷 만으로도 프록시 적용 대상 여부를 구별할 수 있으며 프록시 내부 호출 시 어드바이스가 필요한 특정 메서드 또한 구별 할 수 있는 것이다.**&#x20;

그리고 이런 포인트 컷의 기능을 토대로 스프링은 자동으로 프록시를 생성해 주는 빈 후처리기를 제공한다.&#x20;

`spring-boot-starter-aop` 의존성을 추가하기만 하면`AnnotationAwareAspectJAutoProxyCreator`라는 자동 프록시 생성기를 빈으로 추가해준다.

자동 프록시 생성기는 빈 후처리기단계에서 등록된 모든 어드바이저(포인트 컷 + 어드바이스)를 조회하고 어드바이저에 포함된 포인트 컷을 사용해 이 객체가 프록시 적용 대상인지 아닌지를 체크한다.&#x20;

<img src="../../../.gitbook/assets/file.excalidraw (6).svg" alt="" class="gitbook-drawing">





## AspectJExpressionPointcut

이제 포인트 컷은 프록시 내부에서 적용 대상 뿐 아니라 프록시 생성 여부에도 사용된다.

스프링은 영향력이 더 커진 포인트 컷을 위해 복잡한 대상도 표현할 수 있는 AspecjtJ라는 표현식을 제공한다.&#x20;

```java
 public Advisor advisor() {
      AspectJExpressionPointcut pointcut = new AspectJExpressionPointcut();
      pointcut.setExpression("execution(* hello.proxy.app..*(..))");
      Advice advice = new Advice();
      return new DefaultPointcutAdvisor(pointcut, advice);
```

}

pointcut에 표현식을 등록하고 advisor를 등록하는 것을 볼 수 있다. 표현식은 다음 글에서 더 알아보도록 하자.



{% hint style="info" %}
여러 advisor를 만족하는 빈이 있다면?

여러 advisor를 포함한 하나의 프록시만을 생성해 빈으로 등록한다.&#x20;

proxy factory는 advisor를 여러개 등록할 수 있기 때문이다.
{% endhint %}







