---
title: Design Pattern - Facade
date: 2019-04-09 23:38:41
tags: ["Design Pattern", "Facade"]
category: ["Design Pattern"]
---

이번에 다룰 디자인 패턴은 **퍼사드(Facade) 패턴**이다. 일단 다른 패턴들과 달리 단어를 몰랐기 때문에 이름만으로 와닿지 않았다. 먼저 단어의 의미부터 알아보고 들어가자.

Facade란?

*1. (건물의) 정면 [앞면]*

*2. (실제와는 다른) 표면, 허울*

뭔가 느낌이 오는가?! 전혀 오지 않는다 😅 그럼 퍼사드 패턴의 정의를 함께 살펴보자.



### Facade Pattern

---

위키피디아에서 퍼사드 패턴을 다음과 같이 정의하고 있다. 

**"퍼사드 패턴은 객체 지향 프로그래밍에서 흔히 사용되는 소프트웨어 디자인 패턴이다. 건축물의 외형과 유사하게 퍼사드는 복잡한 시스템 뒤단은 숨기고 간단한 인터페이스를 제공하는 객체다. "**

건축물에 비유했는데 여기에 살을 좀 더 붙여보자면, 우리는 그 건축물의 복잡한 내부 구조는 알 필요 없다. 내부 철근 구조, 파이프 구조를 모르고도 우린 건물 안으로 들어갈 수 있으며 건물의 층계를 엘리베이터 혹은 계단을 통해 오를 수 있다.

퍼사드 패턴의 구조를 살펴보자. 

![](https://ehdrjsdlzzzz.github.io/2019/04/09/Design-Pattern-Facade/facade.png)



퍼사드를 하나의 거대하고 복잡한 시스템이라 가정하자. **Dependency**는 시스템이 서비스를 제공하는데 내부적으로 필요한 객체들을 의미한다. 퍼사드는 이러한 내부 객체들이 서로 상호작용하는 것을 숨기고 그 결과물만을 반환하는 인터페이스를 제공한다. 

클라이언트가 이렇게 복잡한 시스템에 접근하게 되면 구현, 변경, 테스트 그리고 재사용이 굉장히 힘들어진다. 이러한 관계는 퍼사드에 숨기고 클라이언트는 퍼사드만을 갖고 있으면 된다. 그리고 그 퍼사드가 제공하는 인터페이스를 사용만 하면 된다.

> 우리가 사용하는 Cocoapod 라이프러리들이 일종의 이런 퍼사드가 아닐까?

이를 시퀀스 다이어그램으로 표현하면 다음과 같다. 

![](http://java.dzone.com/sites/all/files/facade_seq.PNG)

클라이언트는 퍼사드의 인터페이스를 통해서만 원하는 기능을 사용하고 값을 돌려받는다.

어떤 객체든 그 객체와 상호작용하는 클래스 개수와 그 방법에 주의해야 한다. 그만큼 의존성이 생기기 때문이다. 

퍼사드를 공부하면서 느낀 것은 객체 지향의 **은닉화**와 그 개념이 많이 유사하다는 생각이 들었다. 물론 퍼사드는 그 내부에서 여러 객체들이 상호작용한다는 것이 다른 점이다. 

또한 퍼사드는 하나의 거대한 시스템을 담기보단 그 거대한 시스템을 이루는 작은 시스템, 즉 좀 더 작은 퍼사드들로 이루어져 있는 것이 좀 더 올바른 모습이라고 생각한다.

> 모든지 하나의 거대한 덩어리는 옳지 않다고 생각이 든다. 음.. 그만큼 재사용하기가 어렵고.. 테스트 코드를 작성하기 힘들고…!! 

그럼 간단하게 코드 예제를 살펴보자. 예제의 기능에 필요한 기능과 요소는 다음과 같다.

**기능**

- 4등분으로 나뉘어진 뷰의 왼쪽 상단 뷰에 그림을 그리면 그 그림은 데칼코마니 형식으로 오른쪽, 아래, 그리고 대각선 오른쪽 아래의 나머지 3개의 뷰에 반사된 모습으로 그려진다.  
- 공유 버튼을 누르면 우리는 나뉘어진 뷰의 전체 모습을 하나의 이미지의 형태로 `UIActivityViewController`를 통해 다른 앱에 공유할수 있다. 

**필요한 객체**

- 그림이 그려지는 `inputDrawing` , 위의 기능 설명에서 왼쪽 상단 뷰가 여기에 해당된다. 
- 그려진 데칼코마니를 담고 있는 `entireDrawing`
- 뷰 위에서 사용자가 그린 형태를 이미지로 변환시켜주는 `imageRenderer



![](https://ehdrjsdlzzzz.github.io/2019/04/09/Design-Pattern-Facade/draw.png)

![](https://ehdrjsdlzzzz.github.io/2019/04/09/Design-Pattern-Facade/share2.png)

퍼사드는 대략 다음과 같은 모습을 보인다.

```swift
class ShareFacade {
    public unowned var entireDrawing: UIView
    public unowned var inputDrawing: UIView
    public unowned var parentViewController: UIViewController
    public var imageRenderer: ImageRenderer()
    
    init(entireDrawing: UIView, inputDrawing: UIView, parentViewController: UIViewController) { 
    	// ...

    }

    func presentShareActivity() { 
        // ...
        parentViewController.present( ... )
    }
    
    private func convert() -> UIImage { 
        // ...
        let image = imageRenderer.convert( ... )
        // ...
        
        return image
    }
}
```

내부적으로 그림이 그려진 뷰와 그 결과를 담고 있는 뷰, 이미지로의 변환을 담당하는 객체 그리고 이 뷰들을 갖고 있는 뷰 컨트롤러(*`UIAcitivtyViewController`를 띄어줄 뷰 컨트롤러*)를 갖고 뷰로부터 이미지를 만들어내고 그 이미지를 이용해 `UIActivityViewController`를 띄운다.

그리고 클라이언트는 퍼사드를 아래와 같이 갖고 있는다. 

```swift
public class ViewController: UIViewController {
    lazy var shareFacade: ShareFacade = ShareFacade(entireDrawing: drawViewContainer, inputDrawing: inputDrawView, parentViewController: self)
    
    ...
    
	@IBAction public func sharePressed(_ sender: Any) {
		shareFacade.presentShareActivity()
	}
}
```

이렇게 뷰 컨트롤러는 이미지가 변환되고 `UIActivityViewController`가 올라오는 과정은 모른 채 `presentShareActivity` 메소드만 호출하면 원하는 기능을 사용할 수 있다.