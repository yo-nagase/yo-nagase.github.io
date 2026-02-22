5. 既存の時間記録アプリ全部めんどくさかったので自作した

# Klokka: 個人開発で時間管理アプリを作った話

## はじめに

こんにちは。個人開発でiOSアプリ「Klokka（クロッカ）」をリリースしました。

### 作ろうと思ったきっかけ

普段から複数のプロジェクトを掛け持ちしているのですが、ふと「自分、何にどれだけ時間使ってるんだろう？」と気になりました。体感では均等に時間を割いているつもりでも、実際は特定のプロジェクトに偏っていたり、思ったより進捗が出ていなかったり。

「じゃあ記録しよう」と既存の時間管理アプリをいくつか試したのですが、どれもしっくりこない。

**既存アプリの不満点:**

- プロジェクトの切り替えが面倒（リストから探す、検索する...）
- 切り替え忘れたときの修正がやりづらい
- 「あ、さっきの30分、別のプロジェクトだった」を直すのに何タップも必要

結局「記録が面倒 → 後でまとめてやろう → 忘れる」のループに陥って、どのアプリも3日坊主で終わりました。

「ないなら作るか」と思い立ち、自分が本当に使いたい時間管理アプリを作ることにしました。それがKlokkaです。

**Klokkaで解決したかったこと:**

- スワイプだけでプロジェクトを切り替えられる
- 記録し忘れても、あとから素早く時間を調整できる
- 「うっかり」を前提にした設計

完璧に記録できる人なんていません。だからこそ、**忘れても大丈夫なアプリ**を目指しました。

この記事では、Klokkaの機能紹介と、開発中に苦労した点・工夫した点をお話しします。

## Klokkaの主な機能

### 1. スマートカルーセル - スワイプで直感的にプロジェクト切り替え

![タイマー画面](./images/screenshot-timer.png)

時間記録アプリの最大の敵は「記録が面倒」という心理的ハードルです。

多くのアプリでは、プロジェクトを選ぶのにリストをスクロールしたり、検索したりする必要があります。これが地味にストレスで、「あとで記録しよう」→「忘れた」の悪循環に陥りがちです。

Klokkaでは**スワイプだけでプロジェクトを切り替えられるカルーセルUI**を採用しました。左右にスワイプするだけで次のプロジェクトに移動でき、タップひとつで記録開始。最小限の操作で時間記録を完了できます。

### 2. 忘れても大丈夫 - 素早い時間調整

「あ、1時間前にプロジェクト切り替えるの忘れてた...」

時間記録あるあるですよね。Klokkaでは、こういう「うっかり」を前提に設計しています。

- **開始時刻の調整:** 記録開始後でも、開始時刻を遡って変更可能
- **記録の分割:** 1つの記録を途中で分割して、別のプロジェクトに振り分け
- **ドラッグで直感的に調整:** 時刻をスライダーで動かすだけ

既存アプリだと「記録を削除 → 正しい時刻で新規作成」という手順が必要でしたが、Klokkaなら数タップで修正完了。**完璧に記録できなくても、あとからリカバリーできる**安心感があります。

### 3. ワンタップでPDFレポート出力

![レポート画面](./images/screenshot-reports.png)

記録した時間は、月次レポートとしてPDF出力できます。

- プロジェクト別の作業時間
- 日別の作業時間推移
- 工数（人月）換算

フリーランスの方が請求書と一緒に作業報告書を出したいときや、学生が勉強記録を可視化したいときに便利です。

### 4. 詳細な分析画面

![日別レポート](./images/screenshot-reports-daily.png)

日別・月別で作業時間を可視化。どのプロジェクトにどれだけ時間を使ったか、グラフで一目瞭然です。

### 5. ウィジェット・ライブアクティビティ対応

![ロック画面ウィジェット](./images/screenshot-lockscreen.png)

ホーム画面ウィジェット、ロック画面ウィジェット、ライブアクティビティ（ダイナミックアイランド）に対応。アプリを開かなくても、現在の記録状況を確認できます。

StandByモードにも対応しているので、iPhoneをデスクに置いて卓上時計のように使うこともできます。

### 6. プライバシーファースト

- データは端末内にローカル保存
- iCloud同期はオプション（ユーザーの選択制）
- アカウント登録不要
- 広告なし

「自分の作業時間」という機密性の高いデータを扱うアプリだからこそ、プライバシーを最優先に設計しました。

## 開発の苦労話

### 苦労1: カルーセルUIの実装

スワイプでプロジェクトを切り替えるカルーセルUIは、見た目はシンプルですが実装は意外と大変でした。

**課題:**
- スワイプの速度・距離に応じた慣性スクロール
- 端でのバウンス効果
- 選択中のプロジェクトの拡大表示
- アニメーションのスムーズさ

SwiftUIの`ScrollView`や`TabView`では細かい制御ができなかったため、**ジェスチャーを自前でハンドリング**することにしました。

```swift
// ざっくりしたイメージ
.gesture(
    DragGesture()
        .onChanged { value in
            // ドラッグ中のオフセット計算
            offset = value.translation.width + dragStartOffset
        }
        .onEnded { value in
            // 速度と移動距離から最終位置を計算
            let velocity = value.predictedEndLocation.x - value.location.x
            let targetIndex = calculateTargetIndex(offset: offset, velocity: velocity)
            withAnimation(.spring()) {
                currentIndex = targetIndex
            }
        }
)
```

特に苦労したのは「いい感じの慣性」の調整です。速くスワイプしたら複数プロジェクトを飛ばせるようにしつつ、ゆっくりスワイプしたら1つずつ移動する。この「ちょうどいい感じ」を出すために、何度もパラメータを調整しました。

### 苦労2: ウィジェットとアプリ本体のデータ同期

ウィジェットはアプリ本体とは別プロセスで動作するため、データの共有が必要です。

**App Groupsを使ったデータ共有:**

```swift
// アプリ本体でデータを保存
let sharedDefaults = UserDefaults(suiteName: "group.com.example.klokka")
sharedDefaults?.set(currentProjectId, forKey: "currentProject")

// ウィジェットで読み取り
let sharedDefaults = UserDefaults(suiteName: "group.com.example.klokka")
let projectId = sharedDefaults?.string(forKey: "currentProject")
```

**課題:**
- アプリ本体で記録を開始/停止したとき、ウィジェットにリアルタイム反映させたい
- ウィジェットからタップでアプリを起動したとき、適切な画面に遷移させたい

`WidgetCenter.shared.reloadAllTimelines()`でウィジェットの更新をトリガーできますが、タイミングによっては反映が遅れることがあり、この辺りの挙動調整に時間がかかりました。

### 苦労3: ライブアクティビティとダイナミックアイランド

iOS 16.1から登場したライブアクティビティは、リアルタイムで更新される情報をロック画面やダイナミックアイランドに表示できる機能です。時間記録アプリとの相性は抜群なのですが、実装はかなり複雑でした。

**課題:**
- ライブアクティビティの開始・更新・終了のライフサイクル管理
- ダイナミックアイランドの展開状態（コンパクト/拡張）ごとのレイアウト
- バックグラウンドでのタイマー更新

```swift
// ライブアクティビティの開始
let attributes = TimerActivityAttributes(projectName: project.name)
let state = TimerActivityAttributes.ContentState(
    startTime: Date(),
    projectColor: project.color
)

do {
    activity = try Activity.request(
        attributes: attributes,
        content: .init(state: state, staleDate: nil),
        pushType: nil
    )
} catch {
    print("ライブアクティビティの開始に失敗: \(error)")
}
```

特にダイナミックアイランドは、端末によってサイズが異なり、展開状態も複数あるため、すべてのパターンでレイアウトが崩れないように調整するのが大変でした。

### 苦労4: PDFレポート生成

「ワンタップでPDF出力」は機能としてはシンプルに見えますが、実装は意外と泥臭い作業でした。

**課題:**
- グラフの描画（円グラフ、棒グラフ）
- 複数ページにまたがる場合のページ区切り
- 日本語フォントの埋め込み
- A4サイズへのレイアウト調整

UIKitの`UIGraphicsPDFRenderer`を使用しましたが、SwiftUIのViewをそのままPDFにする方法がないため、グラフは**Core Graphicsで直接描画**する必要がありました。

```swift
let renderer = UIGraphicsPDFRenderer(bounds: CGRect(x: 0, y: 0, width: 595, height: 842)) // A4サイズ
let data = renderer.pdfData { context in
    context.beginPage()

    // タイトル描画
    let title = "月次レポート - 2024年1月"
    title.draw(at: CGPoint(x: 50, y: 50), withAttributes: titleAttributes)

    // グラフ描画
    drawPieChart(in: context.cgContext, data: projectData, rect: chartRect)

    // ページが足りなくなったら新しいページを開始
    if needsNewPage {
        context.beginPage()
    }
}
```

### 苦労5: 多言語対応（日本語・英語）

グローバルリリースを見据えて、日本語と英語の両方に対応しました。

**課題:**
- 文字列の長さが言語によって大きく異なる
- 日付・時刻のフォーマット
- 「工数」「人月」など、英語に直訳しにくい概念

特に「工数」は英語で適切に表現するのが難しく、"Man-Month"や"Person-Hours"など、どの表現を使うか迷いました。最終的には文脈に応じて使い分ける形にしています。

## 技術スタック

- **言語:** Swift
- **UI:** SwiftUI
- **データ永続化:** Core Data + iCloud同期（CloudKit）
- **ウィジェット:** WidgetKit
- **ライブアクティビティ:** ActivityKit
- **PDF生成:** UIGraphicsPDFRenderer + Core Graphics
- **グラフ:** Swift Charts

## 今後の展望

- Apple Watch対応
- macOS対応
- Siri/ショートカット連携
- 繰り返しタスクのテンプレート機能

## おわりに

「時間記録めんどくさい問題」を解決したくて作り始めたKlokkaですが、開発を通じてWidgetKit、ActivityKit、Core Graphicsなど、普段あまり触らないフレームワークを深く学ぶことができました。

時間管理に課題を感じている方は、ぜひ使ってみてください。

**App Storeでダウンロード:** [Klokka - 時間管理をもっと美しく](https://apps.apple.com/app/klokka)

---

この記事が参考になったら、LGTMをお願いします！質問やフィードバックがあれば、コメント欄でお気軽にどうぞ。
