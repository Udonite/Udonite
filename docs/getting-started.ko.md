[English](getting-started.md) | [日本語](getting-started.ja.md) | [简体中文](getting-started.zh-CN.md) | 한국어

# 시작하기

## 설치

1. VRChat Creator Companion을 열고 **Settings → Packages → Add Repository**로 이동합니다.
2. `https://udonite.github.io/vpm/index.json`을 붙여넣고 **Add**를 누릅니다.
3. 월드 프로젝트를 열고 패키지 목록에서 **Udonite**를 추가합니다.

Udonite에는 VRChat Worlds SDK 3.10 이상과 Unity 2022.3이 필요합니다. 의존하는 C# 컴파일러 라이브러리는 함께 들어 있으므로, 그 밖에 설치할 것은 없습니다.

## behaviour 작성하기

`UdoniteBehaviour`를 상속하고, Unity에서 평소 쓰던 그대로 C#을 작성합니다.

```csharp
using Udonite.Package.Runtime;
using UnityEngine;

public class Door : UdoniteBehaviour
{
    public Transform hinge;
    public float openAngle = 90f;
    public float speed = 2f;

    private bool open;
    private float current;

    public override void Interact()
    {
        open = !open;
    }

    public override void Update()
    {
        float goal = open ? openAngle : 0f;
        current = Mathf.MoveTowards(current, goal, speed * 90f * Time.deltaTime);
        hinge.localRotation = Quaternion.Euler(0f, current, 0f);
    }
}
```

Unity와 VRChat의 이벤트(`Start`, `Update`, `Interact`, `OnPlayerJoined`, `OnPickup` 등)는 `UdoniteBehaviour`의 `virtual` 메서드입니다. 필요한 것만 `override`하세요. public 필드는 `MonoBehaviour`와 똑같이 인스펙터에 표시되고, 그 값이 월드로 넘어갑니다.

평범한 `MonoBehaviour`도 컴파일되며 VRChat 이벤트도 동작합니다. `OnPlayerJoined(VRCPlayerApi player)`라는 이름의 메서드는 `UdoniteBehaviour`를 상속하지 않아도 플레이어가 들어올 때 호출됩니다. 차이는 `override`할 대상이 없다는 점입니다. 즉 이름과 매개변수 목록을 직접 정확히 맞춰야 하고, 오타가 나면 "그냥 호출되지 않는 메서드"가 되며 컴파일러는 알려 주지 않습니다. `UdoniteBehaviour`는 모든 이벤트를 `virtual` 메서드로 만들기 때문에 `override`가 그 실수를 잡아 주고, 동기화 필드 어트리뷰트와 VRChat 헬퍼 메서드까지 함께 제공합니다.

## 컴파일과 부착

Udonite는 Unity가 스크립트를 다시 컴파일할 때, 씬을 저장할 때, 그리고 Play를 누를 때 알아서 실행됩니다. 프로젝트 안의 각 루트 behaviour에는 같은 GameObject에 Udon 프로그램이 붙습니다. 콘솔에는 컴파일마다 한 줄이 나오고, 마지막에 `Compiled 12 scripts.` 같은 요약이 나옵니다.

Unity 메뉴 바의 **Udonite → Compile and Attach**로 직접 실행할 수도 있습니다. 이미 컴파일된 스크립트를 인스펙터에서 GameObject로 끌어다 놓은 경우에는, 씬 편집만으로는 재컴파일이 일어나지 않으므로 이 작업이 필요합니다.

Play 모드에서는 C# 컴포넌트가 비활성화되고 Udon 프로그램이 돌아갑니다. 에디터에서 테스트한 것이 곧 월드에서 도는 것이라는 뜻입니다. ClientSim은 로컬 플레이어, 픽업, 인터랙트는 재현하지만 직렬화는 실행하지 않습니다. 그 콜백들은 ClientSim에서 조용합니다. 네트워크 코드는 **Build & Test**로 확인하고, 실제로 네트워크를 건너야 하는 것은 클라이언트 두 개로 테스트하세요.

## 거부되었을 때

Udon이 실행할 수 없는 것은 이름을 짚어 거부됩니다. 콘솔 줄에는 해당 문법과 `UDN` 코드, 그리고 대신 무엇을 쓰면 되는지가 나옵니다.

```
Door.cs(14,9): error UDN0011: Udonite does not support 'UnityEngine.GameObject.CompareTag': Udon does not expose 'tag', so there is nothing to compare in a world. Compare 'gameObject.layer' or 'gameObject.name', or hold a reference to the object you mean.
```

거부된 behaviour는 Udon 프로그램을 갖지 못하지만, 프로젝트의 다른 behaviour는 평소대로 컴파일되어 부착됩니다. 프로그램 내부의 무언가가 조용히 사라지는 일은 없습니다. 전체 코드 목록은 [진단 코드](diagnostics.md)(영어)에 있습니다.

## 스크립트 제외하기

컴파일하고 싶지 않은 스크립트(에디터용 헬퍼, 월드에서는 돌지 않을 외부 컴포넌트 등)는 제외할 수 있습니다. 제외 항목은 `ProjectSettings/UdoniteExclusions.txt`에 한 줄에 하나씩 기록되므로 버전 관리에서도 깔끔하게 병합됩니다. 읽을 수 없는 줄은 `UDN0007`로, 제외한 스크립트를 여전히 호출하는 behaviour는 `UDN0008`로 보고됩니다.

## 업로드

평소대로 VRChat SDK 컨트롤 패널에서 빌드하고 업로드하세요. 월드에 실리는 것은 Udonite가 부착한 Udon 프로그램입니다. C#은 다음 편집을 위해 프로젝트에 남습니다.

---

한국어판은 README와 이 페이지뿐입니다. 나머지 페이지는 갱신이 잦고, 오래된 번역은 영어보다 더 사람을 헷갈리게 하기 때문에 일부러 영어로 두었습니다. 문제 제보는 한국어로 하셔도 괜찮습니다.
