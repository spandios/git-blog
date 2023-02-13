# 빈 생명주기

빈의 생명주기에 대한 콜백을 지원한다. 빈의 초기화와 종료할 때의 훅을 지원함으로써 부가적인 로직을 추가 할 수 있다.



## 스프링 빈의 이벤트 라이프 사이클&#x20;

스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백 -> 사용 -> 소멸 전 콜백 -> 스프링 종료



## @PostConstruct, @PreDestroy 애노테이션 지원

```java
@Component
public class AppServiceTest {
    private final MemberService memberService;

    @Autowired
    public AppServiceTest(MemberService memberService) {
        this.memberService = memberService;
    }
    
    // 초기화 콜백 
    @PostConstruct
    public void postConstruct() throws Exception {

    }
    
    // 소멸 전 콜백
    @PreDestroy
    public void postDestroy() throws Exception {

    }
}

```

* 스프링이 권장하는 방법애노테이션만 붙이면 편리
* 스프링에 종속적이지 않음
* 컴퍼넌트 스캔과 잘 어울린다

