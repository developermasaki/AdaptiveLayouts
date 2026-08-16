# About Adaptive layouts
&nbsp;&nbsp;&nbsp; 以下は、Adaptive layoutsについての備忘録である。

## Overview
&nbsp;&nbsp;&nbsp; アダプティブレイアウトとは、単純に画面のサイズに合わせてUIを伸縮させるのではなく、ウィンドウサイズ・画面密度・フォントサイズ・折りたたみの変化などに基づいて、レイアウトを変更するものである。

## Importance
&nbsp;&nbsp;&nbsp; 以下の点から、アダプティブレイアウトの実装は重要だと言える。

- Google Playのランキングは、[大画面アプリの品質ガイドライン](https://developer.android.com/docs/quality-guidelines/adaptive-app-quality)に基づいている
- 近年、折りたたみデバイスや、デスクトップ画面、スマホ画面、タブレット画面、マルチウィンドウモードというように、ユーザーがアプリを利用する状況が幅広くなっており、従来の全画面表示では対応し切れない

## How to start
&nbsp;&nbsp;&nbsp; 以下の手順でアダプティブレイアウトの実装を進める

### 1. ウィンドウサイズクラスとコンテンツペインの理解<br>
&nbsp;&nbsp;&nbsp; ウィンドウサイズとは、デバイス画面のサイズとは異なり、そのアプリが表示されているアプリウィンドウの大きさのことである。アプリのウィンドウだけを考慮することで、ウィンドウサイズやアスペクト比、向きという問題を、以下の３つに単純化できる<br>

* Compact
  * 幅が600dp以下
  * 対象：縦向きのスマホ
* Medium
  * 幅が600dp以上、840dp以下
  * 対象：縦向きのタブレット、展開された縦型の折りたたみスマホ
* Expanded
  * 幅が840dp以上
  * 対象：横向きのタブレット、展開された横型の折りたたみスマホ

&nbsp;&nbsp;&nbsp; ウィンドウサイズクラスは、幅や高さに基づくが、基本、幅によって判断することが多い。<br><br>
&nbsp;&nbsp;&nbsp; ペインとは、リストと詳細のように、コンテンツごとの画面を指す。単純に画面だと、現在のように１つの画面に複数のコンテンツを表示させる状況を説明できないからである。<br><br>
&nbsp;&nbsp;&nbsp; ウィンドウサイズを取得する方法は次の通りである。
```
val windowSizeClass = currentWindowAdaptiveInfo().windowSizeClass

when {
    windowSizeClass.isWidthAtLeastBreakpoint(WIDTH_DP_EXPANDED_LOWER_BOUND) -> {
        // ウィンドウサイズクラスがExpandedのとき
    }
    windowSizeClass.isWidthAtLeastBreakpoint(WIDTH_DP_MEDIUM_LOWER_BOUND) -> {
        // ウィンドウサイズクラスがMediumのとき
    }
    else -> {
        // ウィンドウサイズクラスがCompactのとき
    }
}
```

### 2. Canonical-layout<br>
&nbsp;&nbsp;&nbsp; Jetpack Composeには、アダプティブレイアウトを手軽に実装できる、3つの関数がすでに用意されている。

1. [NavigationSuiteScaffold](https://developer.android.com/develop/ui/compose/layouts/adaptive/build-adaptive-navigation)<br>
&nbsp;&nbsp;&nbsp; 幅や高さがCompact、あるいは折りたたみデバイスが卓上姿勢の場合、ナビゲーションバーを表示し、それ以外は、ナビゲーションレールを表示する。

2. [ListDetailPaneScaffold](https://developer.android.com/develop/ui/compose/layouts/adaptive/list-detail)<br>
&nbsp;&nbsp;&nbsp; CompactやMediumでは、リストまたは詳細のどちらかを表示し、Expandedでは、リストと詳細を並べて表示する。

3. [SupportingPaneScaffold](https://developer.android.com/develop/ui/compose/layouts/adaptive/build-a-supporting-pane-layout) <br>
&nbsp;&nbsp;&nbsp; CompactやMediumでは、メインコンテンツペインが表示され、Expandedでは、メインコンテンツペインと補助コンテンツペインが表示される。

### 3. 折りたたみデバイスの状態変化を検知
&nbsp;&nbsp;&nbsp; 折りたたみデバイスの姿勢を検知する方法は、次の通りである。<br>
1. まず、`WindowAdaptiveInfo`を取得し、`Posture`を取得する。<br><br>
```
val windowAdaptiiveInfo = currentWindowAdaptiveInfoV2()
val posture = windowAdaptiveInfo.windowPosture
```
2. 例えば、次にヒンジのリストを取得し、折りたたみデバイスの姿勢がどうであるか検出する。
```
// 
val hingeList = posture.hingeList

if (hingeList.any{ it.isFlat == true && it.Vertical }) {
    // 折りたたみデバイスが開かれて、かつ縦型の状態であるとき
}
```

### 4. タッチ以外の入力
&nbsp;&nbsp;&nbsp; Jetpack Composeを使用することで、キーボードやマウス、スタイラスペンへの対応は自動でサポートされる。キーボードショートカットヘルパーの実装は、[以下のリンク](https://developer.android.com/reference/kotlin/android/view/Window.Callback#onprovidekeyboardshortcuts)で、参照できる。

## アダプティブアプリのテスト方法
&nbsp;&nbsp;&nbsp; [アダプティブアプリの品質ガイドライン](https://developer.android.com/docs/quality-guidelines/adaptive-app-quality)では、アダプティブレイアウトへの対応度合いを３つのレベルに分類し、各レベルの達成基準を確認できる。また、Previewでは、[DeviceConfigurationOverride](https://developer.android.com/reference/kotlin/androidx/compose/ui/test/DeviceConfigurationOverride)を使用して、デバイス構成のフォントサイズを変更できたり、`@PreviewScreenSizes` `@PreviewFontScale` `@PreviewLightDar` で画面設定を変えることもできたりする。

## Reference
* [Get started with adaptive apps](https://developer.android.com/develop/ui/compose/layouts/adaptive/get-started-with-adaptive-apps)
* [Use window size classes](https://developer.android.com/develop/ui/compose/layouts/adaptive/use-window-size-classes)
