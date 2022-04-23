---
date: '2021-05-15'
title: '[java] ThreadLocal에 관하여'
categories: ['java', 'korean']
summary: 'Java의 ThreadLocal에 대해서 알아보도록 합시다.'
thumbnail: './images/java_icon.png'
---

안녕하세요. 오늘은 ThreadLocal에 대해서 알아보도록 하겠습니다.

## 변수를 공유하는 방법

객체는 Heap 또는 Stack 메모리 영역에 배치시킬 수 있습니다. Heap 영역은 일반적으로 모든 thread에서 접근 할 수 있으며 stack은 thread 하나당 만들어 지는 메모리 영역으로 thread간 접근이 불가능한 것으로 알려져 있습니다.


아래 코드의 UserRepository 변수는 **Heap 영역에 만들어진 객체를 가리키고 있으며 다른 곳에서도 해당 객체를 바로 접근**할 수 있습니다. 함께 공유해서 사용하기 때문에 여러 thread에서 사용할 때 공유된 정보로써 제공할 수 있습니다. 따라서 만약 UserRepository가 설정 정보를 가지고 있고 이를 변경한다면 사용하고 있는 모든 곳에서 영향을 받게 됩니다.

```java
public class UserService {

  private final UserRepository userRepository;

  public UserServiceV1(UserRepository userRepositoryV1) {
      this.userRepository = userRepository;
  }

  @Transactional
  public void signUp(SignUpRequest request) {
    userRepository.save(request.toUser());
}
```

반면 아래처럼 한 메서드 안에서 변수를 사용한다면 Stack 메모리에 올라갈 것이며 이 변수의 값은 thread에 안전한 변수로 사용될 수 있을 것입니다. 왜냐하면 하나의 thread에 한정적으로 사용할 수 있기 때문이죠. 아래예제를 보도록 하겠습니다. `SignInRequest` 파라미터를 받아서 String 변수인 email, password에 할당한 후 `SignInResponse` 객체를 생성하여 리턴합니다. 이경우 email, password 객체에 대해서는 외부에서 접근을 할 수 있는 방법이 없기 때문에 외부 thread에 안전한 변수로 사용할 수 있는 것입니다. **그렇기 때문에 변수 공유를 위해서는 파라미터로 받아서 사용해야하며 자신의 변수를 다른 곳에서 사용하게 하기 위해서는 리턴값으로 제공해야하는 단점이 있습니다.**

```java
public SignInResponse signIn(SignInRequest request) {

    String email = request.getEmail();
    String password = request.getPassword();

    return new SignInResponse(email, pasword);
}
```

## ThreadLocal 이란

ThreadLocal을 정의하기전에 변수의 선언에 대해서 메모리 관점으로 알아보았습니다. 그 이유는 바로 **ThreadLocal은 한 thread 안에서 파라미터 또는 리턴 값으로 정보를 제공하는 것이 아닌 다른 방법으로 thread 안에서의 값을 공유하는 방법을 제공해주기 때문입니다.** 이렇게 Stack 영역에 변수를 선언하는 것에 대해서 단점을 해소해줄 수 있습니다.

**ThreadLocal의 내부는 thread 정보를 key로 하여 값을 저장해두는 Map 구조**를 가지고 있습니다. 기본적인 사용에는 `get`, `set` 메서드를 이용합니다. 아래의 코드를 통해 한번 사용해보도록 하겠습니다.

### 기본 사용 ( 선언, get, set )

```java
public class UserTermService {

  public static ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>(); // ThreadLocal 사용을 위한 선언

  public UserTermService(Integer value) {
    threadLocalValue.set(value); // ThreadLocal에 set 값을 세팅
  }

  public void threadLocal_1() {
    System.out.println("threadLocalValue : " + threadLocalValue.get()); // ThreadLocal에 있는 값을 가져오기
  }
}
```

테스트 코드를 아래와 같이 작성하였고 예상하는 값은 `threadLocalValue : 5` 이며 정상적으로 호출 된 것을 확인할 수 있었습니다.

```java
public class ThreadLocalTest {

    @Test
    public void threadLocalTest() {
        UserTermService userTermService = new UserTermService(5);
        userTermService.threadLocal_1(); // threadLocalValue : 5 출력 확인
    }
}
```

### 추가적인 메서드

추가적으로 ThreadLocal은 아래처럼 `withInitial` 메서드를 이용하면 lambda funciton을 파라미터로하여 기본값을 설정해 줄 수 있습니다.

```java
public static ThreadLocal<Integer> threadLocalValue = ThreadLocal.withInitial(() -> 5);
```

**또한 `remove` 메서드를 이용하여 설정된 값을 삭제할 수 있습니다.**

```java
threadLocalValue.remove();
```

### 주의사항

ThreadLocal을 사용할 때 반드시 명심해야 할 부분이 있습니다. ThreadLocal은 Thead의 정보를 key로 하여 Map의 형식으로 데이터를 저장한 후 사용할 수 있는 자료구조를 가지고 있습니다. 따라서 **만약 ThreadPool을 사용하여 thread를 재활용한다면 동일한 이전에 세팅했던 ThreadLocal의 정보가 남아있어 원치않는 동작을 할 수 있습니다. 따라서 ThreadPool을 사용하는 경우에는 반드시 모두 사용 후 THreadLocal의 값을 remove 메서드를 사용하여 값을 제거해주는것이 필요합니다.**

## 활용

그렇다면 이러한 ThreadLocal은 어디서 사용하는 걸까요 ? 이는 **Spring Security에서 사용자 인증 정보를 사용할 때 확인**할 수 있었습니다. Spring Security를 사용하여 사용자 인증정보를 가져올 때 아래 코드를 통해서 가져올 수 있습니다. 해당 코드를 이용하면 방금 요청된 정보의 인증정보를 내가 만들고 있는 비즈니스 코드에서 이용할 수 있는 것입니다.

```java
SecurityContextHolder.getContext().getAuthentication();
```

이 코드를 내부로 들어가면 아래와 같은 코드를 발견할 수 있었습니다. getContext() 메서드 내부가 ThreadLocal을 통해 구현되어 있었으며 Thread 별로 인증정보를 다르게 가지고 있는 것을 알 수 있었습니다.

```java
final class ThreadLocalSecurityContextHolderStrategy implements
        SecurityContextHolderStrategy {
    // ~ Static fields/initializers
    // =====================================================================================

    private static final ThreadLocal<SecurityContext> contextHolder = new ThreadLocal<>();

    // ~ Methods
    // ========================================================================================================

    public void clearContext() {
        contextHolder.remove();
    }

    public SecurityContext getContext() {
        SecurityContext ctx = contextHolder.get();

        if (ctx == null) {
            ctx = createEmptyContext();
            contextHolder.set(ctx);
        }

        return ctx;
    }

    public void setContext(SecurityContext context) {
        Assert.notNull(context, "Only non-null SecurityContext instances are permitted");
        contextHolder.set(context);
    }

    public SecurityContext createEmptyContext() {
        return new SecurityContextImpl();
    }
}
```

## 마무리

오늘은 이렇게 ThreadLocal을 사용하기 위해 관련된 내용과 코드, 그리고 활용되고 있는 코드들에 대해서 알아보는 시간을 가져보았습니다.

어떠한 덧글이든 언제나 환영합니다.

감사합니다.

## 참고

멀티 코어를 100% 활용하는 자바 병렬 프로그래밍

[baeldung\_java-threadlocal](https://www.baeldung.com/java-threadlocal)