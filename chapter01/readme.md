### 객체의 생성과 삭제
객체를 만들어야 하는 시점과 그 방법, 객체 생성을 피해야 하는 경우와 그 방법, 적절한 순간에 객체가 삭제되도록 보장하는 방법, 그리고 삭제 전에 반드시 이루어져야 하는 청소 작업들

#### 생성자 대신 정적 패터리 메서드를 사용할 수 없는지 생각해보라.
일반적으로 클래스를 통해 객체를 만드는 방법은 public으로 선언된 생성자를 이용하는 것이다. 그러나 이 방법 외에도 클래스에 public으로 선언된 정적 팩터리 메서드를 추가하는 것이다.

```Java
    public static Boolean valuOf(boolean b){
        return b ? Boolean.TRUE : Boolean.FALSE;
    }
```

위 코드는 Boolean 클래스에 대한 간단한 예제이다 기본 타입 boolean의 값을 래퍼 클래스 Boolean 객체에 대한 참조로 변환한다.
클래스를 정의할 때 생성자 대신 정적 팩터리 메서드를 제공할 수 있다.이로써 얻는 장점은 다음과 같다.
>>>첫번째 장점은 생성자와는 달리 정적 팩터리 메서드에는 이름이 있다.
생성자에 전달되는 인자들은 어떤 객체가 생성되는지를 설명하지 못하지만, 정적 팩터리 메서드는 이름을 잘 짓기만 한다면 사용하기도 쉽고, 클라이언트 코드의 가독성도 높아진다.클래스에는 시그니처 별로 하나의 생성자만 넣을 수 있다. 이 제약을 피하는 한 가지 방법은 인자의 순서를 바꾸는 것이다. 하지만 이 방법은 정말로 끔찍하다. 그런 API 사용자는 각각의 생성자 용도를 절대로 기억하지 못 할 것이며, 결국 실수로 엉뚱한 생성자를 호출하게 될 것이다. 반면 정적 팩터리 메서드에는 이름이 있으므로 그런 문제는 생기지 않는다.

>>> 두번째 장점은, 생성자와는 달리 호출할 때마다 새로운 객체를 생성할 필요는 없다는 것이다.
변경 불가능 클래스라면 이미 만들어 둔 객체를 활용할 수도 있고, 만든 객체를 캐시 해놓고 재사용하여 같은 객체가 불필요하게 거듭 생성되는 일을 피할 수도 있다. 위 코드는 이 기법을 활요한 좋은 예시이다. 또한 이 기법은 FlyWeight 패턴과 유사하다. 동일한 객체가 요청되는 일이 잦고, 특히 객체를 만드는 비용이 클 때 적용하면 성능을 크게 개선할 수 있다. 또한 정적 팩터리 메서드를 사용하면 같은 객체를 반복해서 반환할 수 있으므로 어떤 시점에 어떤 객체가 얼마나 존재할지를 정밀하게 제어할 수 있다. 개체 수를 제어하면 싱글턴 패턴을 따르도록 할 수 있고 객체 생성이 불가능한 클래스를 만들 수도 있다.

>>> 세 번째 장점은, 생성자와는 달리 반환값 자료형의 하위 자료형 객체를 반환할 수 있다는 것이다. 따라서 반환되는 객체의 클래스를 훨씬 유연하게 결정할 수 있다. 이 유연성을 활용하면 public으로 선언되지 않은 클래스의 객체를 반환하는 API를 만들 수 있다.


```Java
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

interface Service{
    // Do Something...
}

interface Provider{
    Service newService();
}

public class Services{
    private Services() {} // 객체 생성 방지

    private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
    public static final String DEFAULT_PROVIDER_NAME = "<def>";

    // 제공자 등록 API
    public static void registerDefaultProvider(Provider p){
        registerProvider(DEFAULT_PROVIDER_NAME, p);
    }
    public static void registerProvider(String name, Provider p){
        providers.put(name, p);
    }

    //서비스 접근 API
    public  static Service newInstance(){
        return newInstance(DEFAULT_PROVIDER_NAME);
    }
    public static Service newInstance(String name){
        Provider p = providers.get(name);
        if(p == null)
            throw new IllegalArgumentException(
                    "No provider registered with name: " + name);

        return p.newService();
    }
}

```
