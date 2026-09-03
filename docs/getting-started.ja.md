[English](getting-started.md) | 日本語 | [简体中文](getting-started.zh-CN.md) | [한국어](getting-started.ko.md)

# はじめかた

## インストール

1. VRChat Creator Companion を開き、**Settings → Packages → Add Repository** を選びます。
2. `https://udonite.github.io/vpm/index.json` を貼り付けて **Add** を押します。
3. ワールドのプロジェクトを開き、パッケージ一覧から **Udonite** を追加します。

Udonite には VRChat Worlds SDK 3.10 以降と Unity 2022.3 が必要です。依存する C# コンパイラのライブラリは同梱しているため、他に入れるものはありません。

## ビヘイビアを書く

`UdoniteBehaviour` を継承して、Unity で普段書いているとおりに C# を書きます。

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

Unity と VRChat のイベント（`Start`、`Update`、`Interact`、`OnPlayerJoined`、`OnPickup` など）は `UdoniteBehaviour` の `virtual` メソッドです。必要なものだけ `override` してください。public フィールドは `MonoBehaviour` と同じようにインスペクタに表示され、その値がワールドに持ち込まれます。

素の `MonoBehaviour` もコンパイルできますし、VRChat のイベントも動きます。`OnPlayerJoined(VRCPlayerApi player)` という名前のメソッドは、`UdoniteBehaviour` を継承していなくてもプレイヤーの参加時に呼ばれます。違いは `override` する対象がないことです。つまり名前と引数を自分で正確に一致させる必要があり、打ち間違えると「呼ばれないだけの普通のメソッド」になり、コンパイラは気づきません。`UdoniteBehaviour` はすべてのイベントを `virtual` メソッドにするので `override` が誤りを捕まえますし、同期フィールドの属性や VRChat 用のヘルパーメソッドも付いてきます。

## コンパイルとアタッチ

Udonite は、Unity がスクリプトを再コンパイルしたとき、シーンを保存したとき、そして Play を押したときに自動で動きます。プロジェクト内の各ルートビヘイビアには、同じ GameObject に Udon プログラムがアタッチされます。コンソールにはコンパイルごとに 1 行出て、最後に `Compiled 12 scripts.` のような要約が出ます。

Unity のメニューバーの **Udonite → Compile and Attach** から手動で実行することもできます。コンパイル済みのスクリプトをインスペクタから GameObject にドラッグした場合は、シーンの編集だけでは再コンパイルが起きないため、この操作が必要です。

Play モードでは C# のコンポーネントは無効化され、Udon プログラムのほうが動きます。エディタで試したものが、そのままワールドで動くということです。ClientSim はローカルプレイヤーやピックアップ、インタラクトは再現しますが、シリアライズのコールバックは発火しません。ネットワーク処理の確認には **Build & Test** を使ってください。

## 拒否されたとき

Udon で動かせないものは、名指しで拒否されます。コンソールの行には対象の構文と `UDN` コード、そして代わりに何を書けばよいかが出ます。

```
Door.cs(14,9): error UDN0011: Udonite does not support 'UnityEngine.GameObject.CompareTag': Udon does not expose 'tag', so there is nothing to compare in a world. Compare 'gameObject.layer' or 'gameObject.name', or hold a reference to the object you mean.
```

拒否されたビヘイビアは Udon プログラムを持ちませんが、プロジェクト内の他のビヘイビアは通常どおりコンパイルされアタッチされます。プログラムの中身が黙って削られることはありません。コードの一覧は[診断コード](diagnostics.md)（英語）にあります。

## スクリプトを除外する

コンパイルしたくないスクリプト（エディタ用のヘルパーや、ワールドでは動かない外部コンポーネントなど）は除外できます。除外は `ProjectSettings/UdoniteExclusions.txt` に 1 行 1 件で記録されるため、バージョン管理でも素直にマージされます。読み取れない行は `UDN0007`、除外したスクリプトをまだ呼んでいるビヘイビアは `UDN0008` として報告されます。

## アップロード

いつもどおり VRChat SDK のコントロールパネルからビルドしてアップロードしてください。ワールドに乗るのは Udonite がアタッチした Udon プログラムです。C# は次の編集のためにプロジェクトに残ります。

---

日本語版は README とこのページのみです。他のページは更新が多く、古い翻訳は誤解のもとになるため、あえて英語のままにしています。不具合の報告は日本語で構いません。
