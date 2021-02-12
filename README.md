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
