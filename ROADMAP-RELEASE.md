# MoneyFlow アプリリリース ロードマップ

## 現状

- PWA（単一HTML + manifest.json + Service Worker）
- GitHub Pages でホスティング
- 「ホーム画面に追加」で擬似アプリとして動作中
- 外部ライブラリ不使用、localStorage ベース

---

## リリース戦略

PWAをネイティブラッパーで包んでストアに出す方式が最もコスパが良い。
コードの書き直し不要で、既存のWeb資産をそのまま活用できる。

### 推奨ツール: **Capacitor**（by Ionic）

- Web アプリをそのまま iOS / Android のネイティブアプリに変換
- ネイティブAPIへのアクセスも可能（通知、生体認証など将来対応）
- 無料・OSS、メンテナンスも活発
- 他の選択肢（PWABuilder, TWA）より拡張性が高い

---

## Phase 0: リリース前準備

### アプリの品質整備
- [x] アプリ名・説明文の確定（日本語 / 英語）→ STORE_LISTING.md
- [x] スクリーンショットフレーム作成 → screenshots.html（実機キャプチャは別途）
- [x] アプリアイコン整備（192/512/1024 全サイズあり、manifest.json に登録済み）
- [x] スプラッシュスクリーン作成 → splash.html
- [x] プライバシーポリシー更新 → privacy.html（通知・SW・ストア審査対応済み）
- [x] 利用規約ページ更新 → terms.html（課金・年齢制限・管轄対応済み）

### 開発者アカウント登録
- [ ] **Google Play Console** — 登録料 $25（一回のみ）
- [ ] **Apple Developer Program** — 年額 $99（毎年更新）

---

## Phase 1: Capacitor プロジェクト化

```bash
# 1. Node.js プロジェクト初期化
npm init -y
npm install @capacitor/core @capacitor/cli

# 2. Capacitor 初期化
npx cap init MoneyFlow com.shourix.moneyflow --web-dir=.

# 3. プラットフォーム追加
npm install @capacitor/android @capacitor/ios
npx cap add android
npx cap add ios

# 4. ビルド＆同期
npx cap sync
```

### 確認ポイント
- [ ] Safe Area の挙動（ノッチ・ダイナミックアイランド対応済みか）
- [ ] スワイプナビゲーションが iOS の戻るジェスチャーと競合しないか
- [ ] localStorage がアプリ内 WebView で正常に動作するか
- [ ] ダークモード切替がOS設定と連動するか

---

## Phase 2: Android リリース（Google Play）

### ビルド
```bash
npx cap open android   # Android Studio で開く
```
- Android Studio で署名付き AAB（Android App Bundle）を生成
- minSdkVersion: 24（Android 7.0+）推奨

### Google Play Console 設定
- [ ] アプリ情報（タイトル・説明・カテゴリ: ファイナンス）
- [ ] スクリーンショット（スマホ + 7インチタブレット）
- [ ] フィーチャーグラフィック（1024x500）
- [ ] コンテンツレーティング質問票に回答
- [ ] プライバシーポリシー URL 設定
- [ ] データセーフティ申告（データ収集なし・サーバー送信なし）

### 審査
- 内部テスト → クローズドテスト → オープンテスト → 製品版
- 初回審査: 通常 1〜3日
- 主な注意点: 「最低限の機能」ポリシー（WebViewだけのアプリは却下リスクあり → Capacitor経由ならOKなケースが多い）

---

## Phase 3: iOS リリース（App Store）

### 前提
- Mac が必要（Xcode でのビルド・署名が必須）
- Mac がない場合: GitHub Actions + Mac ランナー or クラウドビルドサービス（Ionic Appflow等）

### ビルド
```bash
npx cap open ios   # Xcode で開く
```
- Xcode で Archive → App Store Connect にアップロード
- Deployment Target: iOS 15+ 推奨

### App Store Connect 設定
- [ ] アプリ情報（名前・サブタイトル・カテゴリ: ファイナンス）
- [ ] スクリーンショット（6.7インチ + 5.5インチ、iPad任意）
- [ ] App Privacy（データ収集なし）
- [ ] 年齢制限設定

### 審査
- 審査期間: 通常 1〜2日
- 主な却下理由と対策:
  - **4.2 Minimum Functionality**: Webサイトをラップしただけと判断される → ネイティブ要素を1つ以上追加（通知、ウィジェットなど）
  - **5.1.1 Data Collection**: プライバシーポリシーの不備 → 事前に整備
  - **2.1 Performance**: クラッシュ・遅延 → 実機テスト必須

---

## Phase 4: リリース後の運用

### 自動化
- [ ] GitHub Actions で CI/CD（テスト → ビルド → ストアアップロード）
- [ ] バージョン番号管理（package.json で一元管理）
- [ ] OTA アップデート（WebView内のHTML更新はストア審査不要）

### ストア最適化（ASO）
- キーワード: 家計管理, 家計簿, 貯金, 予測, シミュレーター
- ユーザーレビューへの返信

### 将来のネイティブ機能追加（Capacitor プラグイン）
- [ ] プッシュ通知（毎日の支出記録リマインダー）
- [ ] 生体認証（Face ID / 指紋でアプリロック）
- [ ] ウィジェット（今日使える金額をホーム画面に表示）
- [ ] iCloud / Google Drive バックアップ

---

## タイムライン目安

| フェーズ | 内容 | 備考 |
|---------|------|------|
| Phase 0 | リリース準備 | アカウント登録は先にやっておく |
| Phase 1 | Capacitor化 | 既存コードほぼそのまま |
| Phase 2 | Android リリース | 審査がiOSより通りやすい。先に出す |
| Phase 3 | iOS リリース | Mac 必須。審査はやや厳しめ |
| Phase 4 | 運用・改善 | OTAでWebView部分は即時更新可能 |

---

## コスト

| 項目 | 費用 | 頻度 |
|------|------|------|
| Google Play 開発者登録 | $25 | 一回のみ |
| Apple Developer Program | $99 | 毎年 |
| GitHub Pages ホスティング | 無料 | — |
| Capacitor | 無料（OSS） | — |

**最低コスト: 約 $124 で両ストアにリリース可能**
