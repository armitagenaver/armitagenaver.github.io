﻿---
layout: post
title: 개발자 테스트 자동화 도입의 효과
img : Development.jpg
tags: [iOS, testing, automation]
---
(이 글은 qiita.com에 있는 @kuniwak님의 글을 가상 대담 형식을 빌어 제 의견을 같이 넣어 구성한 것임을 밝힙니다. )

#### 조직에서 개발자 테스트 자동화를 하지 못하는 이유는?

* 게부라 : 이번에 참여하신 조직에서 테스트 자동화에 대한 이야기를 해 주신다구요? 
* Kuniwak : 네. 이야기하기 전에 제가 지금까지 본 일반적인 테스트 자동화를 하지 못한 이유를 이야기해야 할 것 같습니다. 적어 보면.. 

	
	1. 시간이 없어서
	2. 테스트 작성 하는 법을 몰라서
	3. 무리하게 테스트를 하다가 크게 데인 적이 있어서

	제가 이번에 조인한 조직에서도 테스트 자동화가 되어 있지 않았는데 분석해 보니 아래와 같은 이유였습니다. 

	1. 자동화 테스트 작성 법을 잘 몰라서
	2. 리뷰가 테스트를 대체하기 때문에..

	새로운 팀원으로 합류를 했을 때 테스트 자동화 경험이 있는 사람은 저 혼자 뿐이었습니다. 정말 막막했죠. 자동화 방향을 제가 정한다 하더라도, 자동화 경험이 없으면 어떻게 해야 할지 모르기 때문입니다. 그러면 자연히 자동화 작업은 뒷전으로 밀리게 됩니다. 
	또한 개발 코드의 리뷰를 통해 최소 품질을 보증하고 있다고 했습니다. 이런 상황에 개발자들이 익숙해져 있었기 때문에 외견상으로는 현장 테스트 자동화가 필요 없었습니다.


* 게부라 : 외견상으로 테스트 자동화가 필요 없었다고 하셨는데 어떤 문제가 발견된 거죠?  
* Kuniwak : 우선 수행중에 있는 리뷰의 효율성에 문제가 있었습니다. 코드 리뷰에 너무 많은 시간을 쓰고 있었던 겁니다. 리뷰에서 오류를 놓쳤다면 그 책임은 전적으로 리뷰어에게로 돌아갑니다. 따라서 리뷰어는 신중하게 리뷰를 할 수 밖에 없고 빠짐없이 코드 전부를 리뷰를 하게 됩니다. 어떻게 보면 비효율적인 일이라 할 수 있습니다. 그리고 리뷰어의 책임은 또다른 문제를 야기합니다. 책임의 무게에 따라 심리적 부담이 커지며 리뷰의 착수가 늦어지는 일이 발생합니다. 안그래도 늦어지는 리뷰가 더더욱 늦어지게 되는 거죠<sup>※</sup>.  

	또 발견되는 문제는 테스트 자동화가 되어 있지 않은 조직에서 자주 발견되는 문제인데 파멸적 코드 설계 였습니다. 파멸적 설계라는 건 예를들어 복잡하고 모듈간 밀접하게 연관되어 있거나, 불안전하게 링킹이되어 설치되어 있다던지 하는 설계를 의미합니다. 이러한 설계는 디버그 시간을 늘리며 코드의 유지보수를 어렵게 합니다. 얼핏보면 테스트 자동화와 파멸적 설계는 관계가 없는 것으로 보입니다만 관계가 분명 있기 때문에 나중에 뒤에서 말씀드리겠습니다.

* 게부라 : 그럼 이 조직에서는 리뷰와 코드의 설계 상태가 문제였다는 말이네요…?  
* Kuniwak :  네. 제가 수행한 대응 활동은 나중에 설명드리고, 방금 언급드린 파멸적 코드 설계에 대하여 좀 더 이야기를 해 보겠습니다. 파멸적 설계로서의 한 예는 악성 싱글턴 패턴이라는 게 있습니다. 이런 악성 싱글턴 패턴은 테스트 자동화에 있어서 최대의 적입니다. 이의 이해를 위해 우선 테스트 자동화의 흐름을 살펴볼 필요가 있습니다.

```
// 악성 싱글턴 패턴의 예 
class ExampleService {
    static let shared = ExampleService()    
    //이 내부변수가 상태가 된다 
    private var number: Int = 0
    func getCount() {
        return self.number
    }
    func increment() {
        self.number += 1
    }
    func reset() {
        self.number = 0
    }
    private init() {}
}
```

테스트 자동화의 기본 패턴은 적당한 입력값을 넣고 그의 출력값을 검증하는 것입니다. 따라서, 테스트 코드에 의한 입력의 제어가 매우 중요합니다. 이때 악성 싱글턴 코드가 입력에 있다면, 테스트코드에서 의도한 싱글턴 부분이 같이 입력값으로 들어가야 합니다. 그러면 테스트 코드의 실행 전 싱글턴 부분에 대한 경우에 따라 분기를 시켜야 하는 복잡한 상황이 발생하게 됩니다. 이것은 어떻게보면 테스트 수행에 있어서 불필요한 코드가 발생하는 것으로 코드가 늘어나기도 하고 테스트 코드의 가독 및 예측도 어렵게 됩니다.

```
//악성 싱글턴을 입력으로 가지고 있는 컴포넌트
class ExampleServiceUser {
    func doSomething() -> Bool {
        //악성 싱글턴의 상태에 의존하여 처리되고 있다 
        guard ExampleService.shared.getCount() == 5 { return }
        // 테스트를 하려는 코드 부분 
        // NOTE: 싱글턴에 뭔가 처리를 하지 않으면 테스트 하려는 코드는 동작하지 않는다. 
    }
}
```

첫번째 코드는 싱글턴 패턴으로 작성되어 있고 두번째 코드는 이를 가져다 쓰고 있습니다. 두번째 코드에서 만일 ExampleService.shared.getCount() 가 5가 아닌 다른 값으로 메모리상에 있다면 위 코드는 실행을 못하고 에러가 날 것입니다. 이런 악성 싱글턴 패턴을 가지고 테스트 코드를 짜려면 아래와 같이 짜야 합니다.

```
import XCTest
// 위 컴포넌트의 테스트 코드
class ExampleServiceTest {
    func testIncrement() {
        // 싱글턴의 상태가 기대대로 되지 않는 경우에 대한 코드 처리 
        if ExampleService.shared.getCount() != 5 {
            ExampleService.shared.reset()
            for _ in 0..<5 { ExampleService.shared.increment() }
        }
        let example = ExampleServiceUser()
        let result = example.doSomething()
        // 결과 검증을 한다 
        XCTAssertTrue(result)        
        // NOTE: 여기서 악성 싱글턴 코드의 입력이 늘어나면 코드의 가독 및 예측이 더 어려워진다 
    }
}
```

이 테스트 코드에서는 아래 XCTAssertTrue() 를 수행하기 위해 앞단에서 shared변수가 5인지를 확인하여 5가 아니면 이를 중가시키는 코드를 별도로 넣었습니다. 이렇게 하지 않으면 shared변수가 5가 아닌 상태에서는 테스트 doSomething()에사 에러가 발생하겠죠. 테스트와 무관한 코드가 들어간 셈입니다.

* 게부라 : 싱글턴 패턴 코드가 다 나쁜건 아닐 것 같은데요… ?
* Kuniwak : 네.. 하지만 싱글턴 패턴에서 이러한 오류가 발생한 가능성이 크기 때문에 가능하면 싱글턴 패턴을 지양하라고 하는 거죠.
* 게부라 : 네 알겠습니다. 그럼 실제로 어떻게 이런 사항들을 팀 내에 전개하셨는지 소개해 주실 수 있을지요.
* Kuniwak : 제가 프로젝트에 테스터로서 조인하여 수행한 활동은 아래 6가지 입니다. 


	1. 리뷰 목적의 정정(訂定)
	2. 팀의 테스트 자동화에 대한 목적의 설정 
	3. 테스트 자동화의 지식 공유
	4. 테스트 자동화의 환경 셋팅
	5. 테스트 용이성을 위한 설계 방법의 전파
	6. “파멸적” 설계의 예방 

####  1. 리뷰 목적의 정정(訂定)

가장 큰 목적은 비효율적인 리뷰의 방지입니다. 이를 위해 리뷰의 목적을 재정의 하였습니다. 원래 지금까지의 리뷰 목적은 버그의 사전 검출이었읍니다만, 내부적으로 재정리 작업을 한 결과 지식 공유와 가독성의 검사 로 바꾸었습니다. 그래서 리뷰어가 빠르게 보면서 의미가 잘 와닿지 않은 부분들, 좀 더 개선 할 수 있는 부분들만 지적해도 될 정도로 부담을 줄였습니다. 그 결과 리뷰어들의 코드 리뷰 부담이 줄어들었고 더 짧은 기간에 리뷰를 수행할 수 있었습니다. 

그럼 이런 의문이 들 수도 있습니다 “리뷰 자체가 가지고 있는 기능을 빼버리면, 품질은 어떻게 보증할 수 있나?” 

이에 대한 답은 개발자 자신에게 있습니다. 코드 품질 보증의 주체를 리뷰어에서 개발자로 옮긴 겁니다.
실은 리뷰의 목적을 재정의 할 때 추가한 항목이 스모크 테스트의 수행 입니다. 버그투성이의 코드를 개발자가 만들어 냈다면 이에 대한 책임은 스모크 테스트를 통해 개발자가 집니다. 이를 통해 자연스럽게 코드 품질 보증 주체를 개발자로 옮길 수 있었고 리뷰 코스트를 줄일 수 있었습니다 . 

하지만 개발자 입장에서는 개발 이외에 테스트라는 또다른 부담을 안게 됩니다. 자기가 짠 코드가 제대로 동작을 하는지를 확인해야 하니까요. 이제까지의 방법으로 이러한 동작을 확인하는 데에도 시간이 걸렸읍니다. 이를 단축하기 위해 테스트 자동화를 제안하게 됩니다.

####  2. 팀의 테스트 자동화에 대한 목적의 설정

테스트 자동화의 목적을 ‘개발속도 향상’ 으로 잡고 테스트 자동화를 바로 적용할 수 있는 부분들을 골라 이를 도입하기로 결정했습니다. 
하지만 ‘개발속도 향상’ 을 위해 어떻게 테스트 자동화를 할 것인가.. 고민끝에 ‘디버깅 시간을 단축시켜주는 테스트 자동화’ 로 포커스를 잡았습니다. 

일반적으로 디버깅시간은 문제 원인 부분을 찾는데 대부분의 시간을 쓰고 있습니다. 그래서 찾는 범위가 작을수록 원인을 찾는데 걸리는 시간은 줄어들게 됩니다. 

자, 문제 원인 부분(버그)가 유입되는 타이밍은 바로 개발 직후 입니다. 예를 들어 API호출 함수를 개발한다고 하면 개발 직후 동작확인에 이상이 발견되면 바로 관련 함수에서 이상동작을 파악할 수 있습니다. 하지만 이를 수동테스트로 확인한다고 해 봅시다. 이의 확인을 위해 어느정도 관련 부분이 UI로 구현이 되어야 확인이 가능합니다. 결국 개발이후 시간이 경과 할 수록 원인부분을 찾을 범위가 UI때문에 늘어나게 되는 것입니다. 

테스트 자동화를 하게 되면 UI구축까지 필요 없으며 테스트 코드레벨에서 바로 원인을 찾게 되므로 범위는 줄어들게 됩니다. 더 효율적이라는 이야기 입니다. 이를 통해 개발 속도를 향상시키는 효과를 가져오게 됩니다. 

하지만 모든 곳에 테스트 자동화 코드를 작성한다고 효율이 올라가는 것은 아닙니다. 예를 들어 UI 의 버그는 테스트 자동화 코드로 발견하기 어려운 것이 많습니다. 이와 같이 어떤 부분에 테스트 자동화를 하면 더 효율이 올라갈 지 판단하는 일은 일정한 경험이 필요합니다.


* 게부라 : 그렇군요. 테스트 자동화를 고려할 때에는 ‘효율성’ 을 가장 먼저 생각하고 진행할 필요가 있을 것 같습니다.
* Kuniwak : 네.. 저는 그렇게 생각합니다.  효과를 조직에서 인정해야 하니까요.

#### 3. 테스트 지식 공유

* 게부라 : 테스트 지식 공유는 뭔가요? 
* Kuniwak : 테스트 코드를 제대로 작성하려면 충분한 지식과 경험이 필요합니다. 하지만, 팀 멤버들은 이런 경험이 없었기 때문에 이런 부분을 제가 지원할 필요가 있었습니다. 제가 직접 모델이 될 만한 테스트 코드를 직접 짜기도 하고 코드 리뷰를 통해 테스트에 대한 지식을 전달하고자 했습니다.

	또한 지식의 전달 뿐만 아니라 실제 테스트코드를 작성하게 하여 보는 시간도 가졌었습니다. 그 결과 팀 멤버들의 테스트 스킬이 상당히 향상되었으며 관련하여 내용 정리 및 공식 블로그에 포스팅을 할 정도의 레벨까지 도달하게 되었습니다. 테스트를 배울 수 있는 이번 환경이 아니었더라면 어려웠을 일이었습니다. 

	테스트 지식 공유와 병행하여 테스트 환경 정비도 하였습니다.

#### 4. 자동화 환경 정비 


우선 테스트 자동화가 동작하지 않는 원인 조사에 착수했습니다. 조사 결과 테스트 자동화와 연관된 XCode의 설정 누락이란 것을 알았습니다. 이번 프로젝트 대상인 iOS에서 단위 테스트 관련 설정이 많기 때문에 그 중 몇몇 설정이 사라져 버린 것이었습니다. 이 누락된 설정을 우선 수정을 하고 바로 테스트를 돌릴 수 있도록 정비를 했습니다. 

바로 테스트를 돌릴 수 있다고 하더라도 자동화 환경 정비가 끝난 것은 아닙니다. 수동 테스트 실행환경에 의해 테스트 코드를 방치하기 쉽기 때문입니다. 여기서 자동화를 위해 계속적 통합 시스템(CI)를 도입합니다. 단 제가 iOS상에서 CI구축은 처음이어서 제 힘으로는 구축이 되지 않는 몇몇 부분이 있었습니다. 이 부분은 전문가인 @henteko에게 부탁을 하였고 그 과정을 보면서 저도 많은 것을 배웠습니다.
이렇게 환경을 구축하고 돌려보았습니다만 기존 코드에서 테스트가 어려운 코드들이 상당부분 있었습니다. 이를 해결하기 위해 테스트를 하기 쉽도록 코드 리팩토링 작업에 들어가게 됩니다.

* 게부라 : 리팩토링이요? 상당히 어려운 작업일 텐데요..?
* Kuniwak : 네 이제부터 리팩토링 이야기를 하려고 합니다.

#### 5. 테스트 용이성을 위한 설계방법의 전파

아키텍쳐 레이어에서 테스트 도입에 가치가 있다고 생각되는 부분들 중 하나가 바로 API레이어 입니다. 하지만 초기에는 이런 API를 호출하는 클라이언트가 없기 떄문에 우선 API클라이언트의 준비부터 시작했었습니다. 이전의 API클라이언트 들은 상속을 전제로 한 설계들이 대부분이어서 이 부분들의 리펙토링을 감행하게 되었습니다. 인증 등에서 좀 어려운 부분이 있었지만, 최종적으로는 테스트가 간단하게 가능해진 API 클라이언트 설계가 가능해졌습니다. 그리고 예전 API클라이언트들을 서서히 새로 설계한 것으로 교체하였습니다. 교체 과정에 대한 이야기는 시작하면 이야기가 길어지기 때문에 건너뛰도록 하겠습니다. 

또 악성 싱글턴 패턴에 대해서도 서서히 의존도를 낮추어 3개월만에 이를 완전에 제거할 수 있었습니다. 싱글턴 패턴의 작성은 한순간 이지만 이를 끊어내는데 상당한 시간이 걸렸던 것입니다. 이런 악성 싱글턴 패턴이 다 없어져서야 테스트 용이성이 확보된 설계가 속속들이 전달되기 시작했습니다.

* 게부라 : 네 … 정리하면 API클라이언트 들을 먼저 리팩토링 하여 상속관계를 끊어 주고 CI구축을 통해 테스트 환경을 만들었네요. 그리고 API들 중 악성 싱글턴 패턴들을 정리하여 테스트 용이성을 확보하였다는 이야기로 이해했습니다. 맞는지요 ? 
* Kuniwak : 네.. 정확합니다. 
* 게부라 : 그리고 리팩토링 등의 결과는 나오기는 했는데 이를 확산하는 과정은 정말 힘이 드셨구요. 
* Kuniwak : 네. 개발자 분들은 잘 돌아 가는(?) 것을 굳이 바꾸지 않으려 하니까요. 
* 게부라 : 이제 최종적인 활동 성과를 이야기 해보도록 하겠습니다. 1.7배 개발 속도가 빨라졌다고 했는데요. 어떻게 빨라졌다고 결론을 내리셨는지요? 
* Kuniwak : 제가 생각한 테스트 자동화 도입의 효과는 3가지 입니다. 

1. LoC베이스 개발 속도가 1.7배 빨라짐
2. 악성 싱글턴 패턴과  FatViewController<sup>※※</sup>를 전부 없앴음. 
3. 급여가 올랐음. 

<img src="//t1.daumcdn.net/thumb/R1280x0/?fname=http://t1.daumcdn.net/brunch/service/user/J8k/image/Wp_rL9fOvpBveHj2s5oVlD2sfTk.png">

일단 LoC 개발속도가 1.7배 정도 빨라졌습니다. 2017년부터 개발자가 2명 –> 3명으로 늘었으니 단순 인적계산으로 본다면 예상 코드생산 증가량은 1.5배정도 입니다 (3/2 = 1.5) 하지만 실제 최종 코드 라인을 측정해 보니 2017년 11월기준으로 약2.5배 정도 늘어났습니다. 테스트 활동으로 인해 1.7배 (2.5/1.5 = 1.7) 가 늘어난 것을 알 수 있습니다. 

그리고 테스트 용이성을 염두한 설계 방식이 정착되어 설계 수준이 상당히 올라간 것도 고무적입니다. 테스트 용이성이 염두된 설계의 특징은 작은 결합/부작용의 최소화 / 단위 레벨의 개발자 책임 이기 때문입니다. 그 결과로 싱글턴 패턴과 FatViewController가 리뷰를 통해 모두 사라졌음을 확인했습니다.


* 게부라 :  LoC 베이스로 측정을 한다는게 상당히 위험하지 않을까요? 정확한 수치일지도 모르겠고
* Kuniwak : LoC는 제가 주장하여 측정한 지표는 아니었습니다. LoC로 진척관리를 하는 것은 더더욱 아니었구요. 개발자들이 자발적인 자기 모니터링을 하는 팩터들 중 LoC가 많이 있어 이를 차용한 겁니다. 그렇기 때문에 LoC관리에서 맹점이라 불리우는 ‘자의적 코드수 늘리기’ 문제는 이 프로젝트에서는 없었다고 보는게 맞을 것 같습니다. 
* 게부라 : 네 그렇군요.. @kuniwak님의 경험이 어떻게 보면 소규모 개발조직(개발자3명정도) 이고 개발자들은 중급 이상의 자기관리가 가능하다는 조건에서 진행하신 거라 보면 되겠지요? 
* Kuniwak : 네.. 그렇습니다. 스타트업이죠... 
* 게부라 : 소규모 조직에서의 테스트 자동화 효과 및 포인트에 대하여 @kuniwak님의 경험이 많은 도움이 될 것 같습니다. 감사합니다. 

---
<sup>※</sup> 일본 조직은 분업이 확실하게 되어 있어 리뷰어들의 책임감이 한국의 그것보다 상당히 크다. 그래서 리뷰의 범위는 어떻게든 철처히 보려 한다
<sup>※※</sup>ViewController가 비대해져서 본기능과 무관한 코드들이 많이 들어간 ViewController를 말한다. 