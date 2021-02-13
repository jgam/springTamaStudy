# springTamaStudy

## 주입 방법
의존관계 주입은 크게 4가지 방법이 있다.
1. 생성자 주입
2. 수정자 주입(setter주입)
3. 필드 주입
4. 일반 메서드 주입

## 생성자 주입
```
@Component
public class OrderServiceImpl implements OrderService{
  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;
  
  //but one constructore you can omit Autowired annotation
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy){
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
```

## 수정자 주입(setter주입)
```
@Component
public class OrderServiceImpl implements OrderService {
  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;
  
  @Autowired
  public void setMemberRepository(MemberRepository memberRepository) {
    this.memberRepository = memberRepository;
  }
  
  @Autowired
  public void setDiscountPolicy(DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
  }
}
```

## 필드 주입
```
@Component
public class OrderServiceImpl implements OrderService{
  @Autowired
  private MemberRepository memberRepository;
  
  @Autowired
  private DiscountPolicy discountPolicy;

}
```

## 일반 메서드 주입
```
@Component
public class OrderServiceImpl implements OrderService{
  private MemberRepository memberRepository;
  private DiscountPolicy discountPolicy;
  
  @Autowired
  public void init(MemberRepository memberRepository, DiscountPolicy discountPolicy){
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

# 옵션처리!
@autowired만 사용하게될때 required= true일때 자동주입대상이 없다... 다른말로 하면 오류발생

1. required=false
2. org.springframework.lang.@Nullable
3. Optional<>

결론....생성자 주입 선택!
di framework는 생성자 주입을 권장함.
- 불변
- 의존관계는 어플리케이션 종료전까지 변하면 안됨. 목적자체가 그렇다하면 생성자 주입을 하는게 맞음.
- 누락
- 프레임워크 없이 순수한 자바코드를 단위테스트 하는경우에 필요하다. Null point exception발생하게됨
- final keyword
- 한번 생성할때 정해지면 안바뀐다. 생성자 혹은 초기값세팅 가능. 그래서 컴파일때에 누락된 설정값 캐치가 가능해짐

## Conclusion
1. 순수한 자바언어의 특징을 잘 살림
2. 생성자 주입 사용후 수정자 주입방식옵션으로 부여 가능 (동시에 사용가능)

# 롬복과 최신 트렌드
생성자 주입은 코드가 굉장히 많음...
```
@Component
public class OrderServiceImpl implements OrderService}
  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;
  
  @Autowired
  public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy){
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
  }
}
```

위코드가...
```

@Component
@RequiredArgsConstructor
public class OrderServiceImpl implements OrderService {

  private final MemberRepository memberRepository;
  private final DiscountPolicy discountPolicy;

}
```

롬복을 추가하는 방법은 플러그인 설치후 annotation processors에서 enable annotation processing 체크


## Two same types of beans
@autowired를 사용하면 타입으로 조회하게된다.

빈2개를 만든다
```
@Component
public class FixdiscountPolicy implements DiscountPolicy()...

@Component
public class RateDiscountPolicy...
```

그리고 의존관계 자동주입 실행!
```
@Autowired
private DiscountPolicy discountPolicy
```

당연히 no unique에러가 발생하게된다.

어떻게 처리할까?
1. @autowired 필드명 매칭
@autowired타입매칭을 시도하고 빈있으면 필ㄷ이름 패라미터이름으로 빈이름을 추가 매칭!

```
@Autowired
private DiscountPolicy discountPolicy
...

@Autowired
private DiscountPolicy rateDiscountPolicy

```
타입매칭 -> 결과2개이상일때는, 필드명,패러미터 명으로 빈이름 매칭

2. @Qualifier 사용
추가 구분자를 붙여주는 방법 단, 빈이름 변경은 아님
```
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy()
```

qualifier끼리 매칭 -> 빈이름 매칭

3. @Primary사용
우선순위를 정하는 방법! @Autowired시에 여러빈이 매칭되면 @Primary가 우선권을 가진다...! 어떤빈에 우선권을 주는것인데.... qualifier와의 우선권은?

annotation 직접 만들기
@Qualifier("mainDiscountPolicy")는 컴파일시 타입체크가 안된다. 그래서 애노테이션을 만들어서 문제를 해결 할 수 있다.
```
//생성자 자동 주입
@Autowired
public OrderServiceImpl(MemberRepository memberRepository, @MainDiscountPolicy DiscountPolicy discountPolicy) {
  this.memberRepository = memberRepository;
  this.discountPolicy = discountPolicy;
}
//수정자 자동 주입
@Autowired
public DiscountPolicy setDiscountPolicy(@MainDiscountPolicy DiscountPolicy discountPolicy) {
  return discountPolicy;
}
```

조회한 빈이 모두 필요할때 List, Map
ㅡ map 에다가 필요한 빈들을 전부 넣어주고 원할때 꺼내쓸수있음

```
public DiscountService(Map<String, DiscountPolicy> policyMap,
List<DiscountPolicy> policies) {
  this.policyMap = policyMap;
  this.policies = policies;
  System.out.println("policyMap = " + policyMap);
  System.out.println("policies = " + policies);
 }
 public int discount(Member member, int price, String discountCode) {
  DiscountPolicy discountPolicy = policyMap.get(discountCode);
  System.out.println("discountCode = " + discountCode);
  System.out.println("discountPolicy = " + discountPolicy);
  return discountPolicy.discount(member, price);
 }
```

- policyMap에 dsciountPolicy를 주입받을것이고 그안에는 fix ratediscountPolicy 가있다.
- discountCode에 따라서 맞는 스프링빈을 찾아서 실행한다.


# 자동,수동의 올바른 실무 운영기준

1. 편리한 자동기능을 기본으로 사용!
- 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원하고 스프링 부트는 컨포넌트 스캔을 기본으로 사용한다.
- 자동빈등록을 통해서도 OCP,DIP를 지킬 수 있다.

2. 수동빈 등록은 언제 사용?
- 업무로직빈: 웹을 지원한는 컨트롤러, 핵심 비지니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리등이 모두 업무 로직 -> 비슷한 로직에 자동기능 적극 사용! 어디서 무제생겼는지 파악이 쉽다
- 기술 지원 빈: 기술적인 문제가 공통 관심사처리에 사용 -> 수가 적고 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미친다. 수동빈사용!

## conclusion
편리한자동기능을 기본으로 사용!
직접등록 하는 기술지원객체는 수동!
