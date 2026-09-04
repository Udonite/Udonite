[English](README.md) | [日本語](README.ja.md) | [简体中文](README.zh-CN.md) | 한국어

# Udonite

**평범한 Unity C#으로 만드는 VRChat 월드 스크립팅.**

`MonoBehaviour`를 작성하면 Udonite가 그것을 Udon으로 컴파일합니다. 따로 익혀야 할 방언도, 외워야 할 금지 키워드 목록도 없습니다. Unity에서 이미 쓰고 있는 그 C#이, 월드 안에서 그대로 돌아갑니다.

Udonite는 무료이며, 네트워크 기능도 포함되어 있습니다.

## 설치

1. VRChat Creator Companion을 열고 **Settings → Packages → Add Repository**로 이동합니다.
2. `https://udonite.github.io/vpm/index.json`을 붙여넣고 **Add**를 누릅니다.
3. 월드 프로젝트를 열고 패키지 목록에서 **Udonite**를 추가합니다.

또는 [리스팅 페이지](https://udonite.github.io/vpm/)에서 **Add to VCC**를 누르세요.

VRChat Worlds SDK 3.10 이상과 Unity 2022.3이 필요합니다. 그 밖에 설치할 것은 없습니다. 컴파일러가 의존하는 라이브러리는 패키지에 함께 들어 있습니다.

## 사용법

`MonoBehaviour`를 작성하고 Play를 누르면 됩니다. Udonite는 프로젝트 안의 모든 behaviour를 컴파일해 같은 GameObject에 Udon 프로그램을 붙이고, 컴파일마다 한 줄씩 로그를 남깁니다. Play 모드에서 실제로 도는 것은 Udon 프로그램 자체이므로, 에디터에서 보고 있는 것이 곧 업로드되는 것입니다.

`UdoniteBehaviour`를 상속하는 것은 선택이지만 조금 더 편합니다. VRChat 이벤트가 `override` 가능한 `virtual` 메서드가 되므로, 이벤트 이름을 잘못 쓰면 "호출되지 않는 메서드"가 아니라 컴파일 오류가 됩니다. 동기화 필드 어트리뷰트와 VRChat 헬퍼 메서드도 함께 딸려 옵니다.

## 무엇을 쓸 수 있나

- **부분집합이 아니라 언어 그 자체.** 제약 조건이 붙은 제네릭, 기본 구현이 있는 인터페이스, 레코드, 구조체, 중첩 클래스, 확장 메서드, 튜플과 분해, 지역 함수, `params`와 선택적 매개변수.
- **현대적인 C# 문법.** switch 식, 패턴 매칭(`is > 5 and < 10`, 속성 패턴, `is not null`), `?.`, `??=`, 형식 지정자를 포함한 문자열 보간, 범위와 인덱스.
- **컬렉션과 LINQ.** `List<T>`, `Dictionary<K,V>`, `HashSet<T>`, `Queue<T>`, `Stack<T>`, 다차원 배열, 그리고 LINQ 체인(`Where`, `Select`, `OrderBy`, `GroupBy`, `Zip` 등). 람다는 호출 지점에 그대로 펼쳐져 컴파일됩니다.
- **델리게이트와 이벤트.** `Action`과 `Func` 필드, 메서드 그룹, 멀티캐스트 델리게이트의 `+=`와 `-=`, `?.Invoke`를 쓰는 C# `event` 선언, 값으로 다루는 람다.
- **시간도 평소처럼.** `yield return new WaitForSeconds(...)`를 쓰는 코루틴, `await Task.Delay(...)`를 쓰는 `async` 메서드, `Invoke`와 `InvokeRepeating`.
- **페이로드가 있는 네트워크 처리.** 변경 콜백이 붙은 동기화 필드, 이름으로 호출하는 커스텀 이벤트, 그리고 타입이 있는 네트워크 이벤트. 필드를 가진 클래스를 정의하고 `Network.Send(evt, receiver)`로 보낸 뒤, 그 이벤트 타입을 매개변수로 받는 메서드에서 받습니다. 위치·회전·크기를 동기화하는 `NetworkTransform` 컴포넌트도 기본 제공됩니다.
- **널 허용 값, `TryGetComponent`, `StringBuilder`, `Array.FindAll`** 등 별생각 없이 손이 가는 자잘한 것들까지.

```csharp
using System.Collections.Generic;
using System.Linq;
using Udonite.Package.Runtime;
using UnityEngine;

public class ScoreEvent : NetworkEvent
{
    public int player;
    public int points;

    public void Serialize(ByteWriter writer) => writer.WriteInt32(player).WriteInt32(points);
    public void Deserialize(ByteReader reader) { player = reader.ReadInt32(); points = reader.ReadInt32(); }
}

public class Scoreboard : UdoniteBehaviour
{
    public event System.Action<int> Changed;
    private readonly Dictionary<int, int> totals = new Dictionary<int, int>();

    public void Award(int player, int points) => Udonite.Network.Send(new ScoreEvent { player = player, points = points }, this);

    [NetworkEventHandler]
    public void OnScore(ScoreEvent evt)
    {
        totals.TryGetValue(evt.player, out int current);
        totals[evt.player] = current + evt.points;
        Changed?.Invoke(evt.player);
    }

    public string Leader() => totals.OrderByDescending(pair => pair.Value).Select(pair => pair.Key.ToString()).FirstOrDefault() ?? "nobody";
}
```

## Udonite가 하지 않는 것

Udon에는 실제로 존재하는 제약이 있습니다. 예외 없음, 변경 가능한 static 없음, `Awake` 없음, 태그 없음. Udonite는 그런 것들을 이름을 짚어 거부합니다. 모든 거부는 콘솔의 한 줄로 나오며, `UDN` 코드와 함께 무엇이 거부되었는지와 대신 무엇을 쓰면 되는지를 알려 줍니다. 조용히 사라지는 것은 없습니다. 거부된 behaviour는 프로그램을 갖지 못하지만, 프로젝트의 나머지는 평소대로 컴파일됩니다. 자세한 내용은 [언어 지원](docs/language-support.md)(영어)을 참고하세요.

## 문서

**[udonite.dajno.com](https://udonite.dajno.com/?lang=ko)** 에 한국어 문서가 모두 있고 검색도 됩니다.

- [시작하기](docs/getting-started.ko.md)(한국어)
- [언어 지원](docs/language-support.md)(영어)
- [네트워크](docs/networking.md)(영어)
- [UdonSharp에서 이전하기](docs/migrating-from-udonsharp.md)(영어)
- [진단 코드](docs/diagnostics.md)(영어)

한국어판은 README와 「시작하기」 두 페이지뿐입니다. 나머지 페이지는 갱신이 잦고, 오래된 번역은 영어보다 더 사람을 헷갈리게 하기 때문에 일부러 영어로 두었습니다.

## Udonite 후원하기

Udonite 컴파일러는 무료이며, 앞으로도 계속 무료입니다.

Udonite는 한 사람이 만들고 유지보수합니다. VRChat SDK가 갱신되면서 무언가 깨질 때마다 고치는 일도 마찬가지입니다. 후원은 그 작업과 그 주변에서 만들어지는 서비스를 지원합니다.

후원해도 돌아오는 것은 없습니다. 후원자 등급도, 비공개 패키지도, 제한된 기능도 없으며 어느 쪽이든 전부 무료입니다. 이 작업이 계속되기를 바라신다면 후원해 주세요. 그렇지 않다면 컴파일러를 그냥 쓰시면 됩니다.

[후원하기](https://buy.polar.sh/polar_cl_fiMBXRqzbvm0c8qt6xqKKBGbaB0FOQmPMddgn06PRsC)

## 문제 제보

버그, 컴파일되어야 한다고 생각하는데 거부된 문법, SDK 업데이트로 인한 문제는 [Issues](https://github.com/Udonite/Udonite/issues)로 보내 주세요. 한국어로 적으셔도 괜찮습니다. 콘솔에 나온 거부 줄을 그대로 붙여 주시면 어떤 문법인지 정확히 알 수 있습니다.

Udonite는 서드파티 도구이며 VRChat Inc.와는 관계가 없습니다.
