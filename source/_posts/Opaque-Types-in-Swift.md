---
title: Opaque Types in Swift
subtitle: some, 넌 누구니...? - SwiftUI를 공부하다가
date: 2019-12-12 20:59:12
tags: ["Opaque Type", "some"]
category: ["Swift", "SwiftUI"]
---

## Opaque Types in Swift

오늘은 Swift 5.1에서 처음 소개된 **불투명 타입(Opaque Type)**에 대해 공부해보고 기록해보려 한다. 사실 SwiftUI를 본격적으로 공부하려다 `some` 키워드에 막혀 이게 무엇인지 알아보려다 여기까지 오게 되었다. 

>  이렇게 SwiftUI는 다시금 뒤로 미뤄졌다.🙄 진짜 제대로 시작해야 하는데..

불투명 타입을 공부하면서 명확하게 이해가 된 부분도 있지만, 아직은 불투명하게 다가온 부분도 있었다. 그 이유는 아무래도 어느 문법을 공부해도 마찬가지겠지만, 직접 프로젝트에 사용하여 필요성을 느껴보지 못해서 그런 것 같다. 

> @escaping 클로저를 처음 배웠을 때도 이와 비슷한 느낌이었다. 

오늘은 단순히 문법적 이해를 위한 기록으로 작성하고 추후 기회가 된다면 직접 사용해보고 이 문법이 왜 도입되었고, 언제 사용하면 좋을지 다른 사람들이 소개한 케이스가 아닌 내가 경험한 케이스를 소개해보려 한다. 

------

### Type Identity

불투명 타입을 소개하기 전 가장 먼저 본인이 불투명 타입을 공부하면서 헷갈렸던 부분을 먼저 소개해보려 한다. 스위프트 공식 문서나 여러 해외 블로그들을 읽다 보면 불투명 타입을 설명하는 데 있어 **정체성(Identity)**를 자주 언급하는 것을 확인할 수 있다. 정확하게 말하면 **타입 정체성(Type Identity)**이다.

타입 정체성이란 무엇일까? 몇 개의 글과 함께 예제 코드를 보고서야 조금은 감을 잡을 수 있었다. 내가 이해한 타입 정체성은 다음과 같다. 

우린 스위프트에서 프로토콜을 타입으로써 사용할 수 있다. 

```swift
protocol Analytics {
  func log()
}

class FirebaseAnalytics: Analytics { 
  func log() { /* ... */ }
}

class FlurryAnalytics: Analytics { 
  func log() { /* ... */ }
}

class AnalyticsManager { 
  private let analytics: Analytics
  
  init(_ analytics: Analytics) { 
    self.analytics = analytics
  }
  
  func sendLog() { 
    analytics.log()
  }
}

let manager1 = AnalyticsManager(FirebaseAnalytics())
let manager2 = AnalyticsManager(FlurryAnalytics())
```

> 전적으로 예제를 위한 예제 코드다. 

`AnalyticsManager`는 어떤 애널리틱스 툴을 사용해도 로그만 전송하면 된다. 우리는 보통 이런 경우 위와 같이 프로토콜을 만들고 행위를 정의한다. 그리고 행위를 구현한 구현체를 프로토콜 타입으로써 주입해준다. 

`AnalyticsManager`가 `sendLog`를 호출할 때 `AnalyticsManager`는 `FirebaseAnalytics`나 `FlurryAnalytics`가 아닌 단지 `Analytics` 타입으로 `analytics` 프로퍼티를 사용한다. 이렇게 프로토콜 타입은 사용되는 시점에 자신의 실제 타입에 대한 정체성을 잃게 된다.

"잃게 된다"라는 어감(?) 때문인지 부정적으로 다가올 수 있지만 이게 프로토콜 타입을 사용하는 이유다.

> 추상화의 의미를 생각해보면 정체성을 잃는다는 것을 보다 쉽게 받아들일 수 있다.

그리고 이런 프로토콜 타입은 프로그램에 유연성을 제공해주기 때문에 의존성 주입 등 코드 작성 전반에 걸쳐 유용하게 사용된다. 

하지만 이러한 유연성이 때로는 코드 복잡성을 야기할 수 있다. 일반적인 프로토콜 타입에는 해당하지 않지만 `associatedtype`이나 `Self`를 사용하는 프로토콜 타입은 타입으로써 반환되거나, 파라미터로 전달, 프로퍼티로 사용될 수 없다. 만일 그렇게 사용하면 우린 굉장히 익숙한 에러 메세지를 볼 수 있다. 

```
Protocol 'SomeProtocol' can only be used as a generic constraint because it has Self or associated type requirements
```

너무 돌아왔다. 결국 **프로토콜 타입은 타입 정체성을 잃고 유연성을 제공**한다는 것에 주목하자!

------

### Opaque Types 

불투명 타입의 사용법은 간단하다. 반환되는 프로토콜 타입 앞에 `some` 키워드를 붙여주면 끝이다. 하지만 단순히 문법의 사용법만 가지곤 문법을 활용할 수 없다. 언제 불투명 타입을 사용할 수 있고, 불투명 타입을 왜 사용하는지에 대해 알아보았다. 

이를 위해 불투명 타입이 없던 Swift 5.1 이전으로 돌아가 보자. 결제수단을 제공하는 프레임워크를 작성한다고 가정해보자. 

```swift
public func favoriteCreditCard() -> CreditCard { 
  return getLastUsedCreditCard()
}
```

위와 같은 방식으로 사용자가 최근에 사용한 카드를 사용자가 선호하는 카드로 반환해줄 수 있다. 하지만 이러한 과정에서 우리는 굳이 사용자가 알 필요 없고, 알아서는 안되는 `CreditCard`라는 타입을 노출시킨다. 

우리는 이런 문제를 프로토콜을 사용하여 해결하려 한다.  

```swift
protocol PaymentType { }
struct CreditCard: PaymentType { }
struct ApplePay: PaymentType { }

func favoritePaymentType() -> PaymentType { 
  if likesApplyPay { 
    return ApplePay()
  }
  return getLastUsedCreditCard()
}
```

얼핏 보면 해결된 것처럼 보일 수 있지만 사용자가 `PaymentType`을 비교해야 한다면 얘기는 달라진다.  

```swift
protocol PaymentType: Equatable { }
```

> `Equatable` 타입은 내부적으로 `Self`를 사용한다.

이렇게 결제수단의 비교를 위해 `Equatable`을 사용하면 타입 정체성에서 언급한 에러 메세지를 보게 된다. 

```
Error: Protocol 'PaymentType' can only be used as a generic constraint because it has Self or associated type requirements
```

물론 이런 문제를 제네릭과 타입 제거 기법(type erasure techniques)을 통해 해결할 수 있다. 하지만 이는 프레임워크의 사용을 더욱 어렵게 만들며 역시 의도치 않게 내부 타입을 노출시킬 수 있다. 

불투명 타입이 이런 문제를 해결할 수 있다. 방법은 매우 간단하다. 

```swift
public func favoriteCreditCard() -> some PaymentType { 
  return getLastUsedCreditCard()
}
```

물론 방법이 매우 간단한 만큼 한계도 존재한다. 불투명 타입을 반환하는 함수는 오로지 하나의 타입만 반환할 수 있다. 위의 코드에서 함수의 반환 타입은 `CreditCard`다. 하지만 컴파일러가 이를 `PaymentType`인 것 마냥 다룬다. 내부적으론 실제 타입을 반환하는 것이기 때문에 실제 타입으로 할 수 있는 모든 것들은 할 수 있다. 즉 **타입 정체성이 보장**된다는 것이다. 타입의 정체성이 보장되기 때문에 `associatedtype`이나 `Self`를 사용하는 프로토콜도 반환 타입으로 바로 사용할 수 있다.

물론 불투명 타입의 한계도 존재한다. 이런 타입의 정체성을 보장하기 위해 하나의 타입만을 반환할 수 있다. 

```swift
func favoritePaymentType() -> some PaymentType { 
  if likesApplyPay { 
    return ApplePay()
  }
  return getLastUsedCreditCard()
}
```

즉 위와 같은 코드는 사용할 수 없다는 뜻이다. 

다시 불투명 타입을 사용하면 어떤 점이 좋은지 이야기해보자. 위에서 언급했듯이 불투명 타입은 모듈 내부의 타입을 노출시키지 않을 수 있다. 이게 단순히 하나의 타입을 노출시키지 않는다는 것을 의미하기도 하지만, 타입이 내부적으로 어떤 타입들로 구성되어 있는지도 숨길 수 있다. 

SwiftUI를 예로 들어보자. SwiftUI 프로젝트를 생성하면 하나의 템플릿 코드를 확인할 수 있다.

```swift
struct ContentView: View { 
  var body: some View { 
    Text("Hello World")
  }
}
```

> 바로 내가 SwiftUI 공부를 멈추고 불투명 타입을 공부하기 시작하게 된 코드다. 

이 코드에서 `some` 키워드를 사용한 이유가 뭘까? `some` 키워드를 지우면 컴파일이 되지 않는 걸까? 그렇지 않다. `some` 키워드가 없이도 충분히 프로그램을 만들 수 있다. 

```swift
struct ContentView: View { 
  var body: Text { 
    Text("Hello World")
  }
}
```

`some` 키워드를 사용하지 않고 SwiftUI로 화면을 만들어보자. SwiftUI에선 `VStack`을 이용해 스택 뷰의 형태를 구현할 수 있다. 그리고 SwiftUI에서 이런 컨테이너 타입들은 모두 제네릭 타입이다. 

```swift
struct ContentView: View {
    var body: VStack<TupleView<(Text, Image)>> {
        VStack {
            Text("Hello World")
            Image(systemName: "video.fill")
        }
    }
}
```

.

.

🤔



자자..! 일단 계속 만들어보자. 하나의 텍스트를 더 추가해보자.

```swift
struct ContentView: View {
    var body: VStack<TupleView<(Text, Text, Image)>> {
        VStack {
          	Text("Hello World")
            Text("My name is Corn")
            Image(systemName: "video.fill")
        }
    }
}
```

좀 더 복잡한 레이아웃을 그려보자!

.

.

![](https://i.ytimg.com/vi/-2Z0Y3Kk8nU/maxresdefault.jpg)

.

.

```swift
struct ContentView: View {
    var body: List<Never, TupleView<(HStack<TupleView<(VStack<TupleView<(Text, Text)>>, Text)>>, HStack<TupleView<(VStack<TupleView<(Text, Text)>>, Text)>>)>> {
      ...
      .....
      ........
    }
}
```

자 완성했다! 근데 중간 어디쯤에 있는 `Text`를 `Image`로 바꾸려 한다! 그러면...🤯

이쯤 되면 왜 불투명 타입이 필요한지 절실히 깨달을 수 있다. 우린 단지 `body`가 *어떤(some)* `View` 타입을 반환한다 정도만 알고 싶은 것이다. 그리고 그 정도의 정보만 필요하다. 

우린 `some` 키워드로 `body`는 단지 `View` 타입을 반환한다는 사실을 명시해줄 수 있다. 

```swift
struct ContentView: View {
    var body: some View {
      ...
      .....
      ........
    }
}
```



------

### 마무리

아직은 내가 직접 작성한 모듈 혹은 프로젝트에서 불투명 타입을 사용해보진 못했다. 하지만 이젠 불투명 타입의 존재와 언제 사용하면 좋을지에 대해 훑어보았기 때문에 다음에 기회가 왔을 때 불투명 타입 사용을 고려해볼 수 있게 되었다. 

> SwiftUI 공부도 다시 시작할 수 있다!

------

**참고 자료**

- [What's this "some" in SwiftUI](https://medium.com/@PhiJay/whats-this-some-in-swiftui-34e2c126d4c4)
- [Understanding Opaque Return Types in Swift](https://swiftrocks.com/understanding-opaque-return-types-in-swift.html)
- [Opaque Types](https://docs.swift.org/swift-book/LanguageGuide/OpaqueTypes.html)



