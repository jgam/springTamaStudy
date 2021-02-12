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
불변
- 의존관계는 어플리케이션 종료전까지 변하면 안됨. 목적자체가 그렇다하면 생성자 주입을 하는게 맞음.
누락
- 프레임워크 없이 순수한 자바코드를 단위테스트 하는경우에 필요하다.
final keyword
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



