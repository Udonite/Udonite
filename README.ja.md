[English](README.md) | 日本語 | [简体中文](README.zh-CN.md) | [한국어](README.ko.md)

# Udonite

**VRChat のワールド制作を、普通の Unity C# で。**

`MonoBehaviour` を書けば、Udonite がそれを Udon にコンパイルします。覚えるべき方言も、使えないキーワードの一覧もありません。Unity で普段書いている C# が、そのままワールドの中で動きます。

Udonite は無料です。ネットワーク機能も含みます。

ワールドから [Udonite のホスティングサービス](https://admin.udonite.com)を呼び出すこともできます。ワールドから Discord や Pushover にメッセージを送ったり、カウンターやスコア表、共有スイッチを全員が退出したあとも残したりできます。GitHub アカウントがあれば無料で試せます。

**[ドキュメント](https://docs.udonite.com/compiler?lang=ja)** · **[インストール](https://udonite.com/?lang=ja)**

## 使い方

`MonoBehaviour` を書いて Play を押すだけです。Udonite はプロジェクト内のすべてのビヘイビアをコンパイルし、同じ GameObject に Udon プログラムを付け、コンパイルごとに 1 行ログを出します。Play モードで動くのは Udon プログラムそのものなので、エディタで見えているものがそのままアップロードされます。

`UdoniteBehaviour` を継承する方法もあります。必須ではありませんが、少し楽です。VRChat のイベントが `virtual` メソッドになるため `override` でき、イベント名を打ち間違えても「何も呼ばれないメソッド」ではなくコンパイルエラーになります。同期フィールドの属性や VRChat 用のヘルパーメソッドも付いてきます。

## 書けるもの

- **サブセットではなく、言語そのもの。** 制約付きジェネリクス、既定実装を持つインターフェース、レコード、構造体、入れ子クラス、拡張メソッド、タプルと分解、ローカル関数、`params`、省略可能引数。
- **現代的な C# の構文。** switch 式、パターンマッチング（`is > 5 and < 10`、プロパティパターン、`is not null`）、`?.`、`??=`、書式指定子付きの文字列補間、範囲とインデックス。
- **コレクションと LINQ。** `List<T>`、`Dictionary<K,V>`、`HashSet<T>`、`Queue<T>`、`Stack<T>`、多次元配列、そして LINQ のチェーン（`Where`、`Select`、`OrderBy`、`GroupBy`、`Zip` など）。ラムダは呼び出し箇所に展開されます。
- **デリゲートとイベント。** `Action` と `Func` のフィールド、メソッドグループ、マルチキャストデリゲートの `+=` と `-=`、`?.Invoke` を伴う C# の `event` 宣言、値としてのラムダ。
- **時間の扱いも普通に。** `yield return new WaitForSeconds(...)` を使うコルーチン、`await Task.Delay(...)` を使う `async` メソッド、`Invoke` と `InvokeRepeating`。
- **ペイロード付きのネットワーク処理。** 変更コールバック付きの同期フィールド、名前を指定するカスタムイベント、そして型付きネットワークイベント。フィールドを持つクラスを定義し、`Network.Send(evt, receiver)` で送り、そのイベント型を引数に取るメソッドで受け取ります。位置・回転・スケールを同期する `NetworkTransform` コンポーネントも同梱です。
- **null 許容型、`TryGetComponent`、`StringBuilder`、`Array.FindAll`** など、意識せずに手が伸びる細かいものも一通り。

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

## できないこと

Udon には実際の制約があります。例外なし、書き換え可能な static なし、`Awake` なし、タグなし。Udonite はそれらを名指しで拒否します。拒否はコンソールに 1 行出て、`UDN` コードとともに「何が拒否されたか」と「代わりに何を書けばよいか」を示します。黙って無視されるものはありません。拒否されたビヘイビアはプログラムを持ちませんが、プロジェクト内の他のビヘイビアは通常どおりコンパイルされます。詳細は[対応状況](https://docs.udonite.com/compiler/language-support?lang=ja)（英語）を参照してください。

## Udonite を支援する

Udonite コンパイラは無料で、これからもずっと無料です。Udonite は一人で作り、VRChat SDK の更新で何かが壊れるたびに一人で直しています。

支援はその作業を支えるものですが、いまはワールドで実際に使えるものも付いてきます。Udonite にはワールドから URL で呼び出せるホスティングサービスがあります。ワールドから Discord や Pushover にメッセージを送り、カウンターやスコア表、共有スイッチを全員が退出したあとも残る形で保存できます。どちらも無料で試せて、支援すると上限は事実上気にならないところまで上がります。

コンパイラはその対象外です。誰にとっても無料のままで、SDK の更新で壊れた箇所の修正は全員に同じ日に届きます。

[支援する](https://buy.polar.sh/polar_cl_fiMBXRqzbvm0c8qt6xqKKBGbaB0FOQmPMddgn06PRsC) · [サービスの内容](https://udonite.com/?lang=ja)

## 不具合の報告

不具合、コンパイルできるはずなのに拒否された構文、SDK の更新による破損は [Issues](https://github.com/Udonite/Udonite/issues) へお願いします。日本語で構いません。コンソールに出た拒否の行をそのまま貼り付けてください。どの構文かが正確に分かります。

Udonite は有志による非公式ツールであり、VRChat Inc. とは関係ありません。
