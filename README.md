# EXE Lounge 座席管理システム

iPad（中のスタッフ）とPC（入口のスタッフ）で座席の空き状況をリアルタイム共有するシステムです。

## 特徴

- **リアルタイム同期**: iPadでタップした瞬間、PCに即反映（Firebase WebSocket）
- **インストール不要**: ブラウザでURLを開くだけ
- **ネットワーク不問**: iPad（Wi-Fi）とPC（VPN）が別ネットワークでもOK
- **完全無料**: Firebase無料枠 + GitHub Pages
- **設定変更が簡単**: 管理画面から座席の追加・削除・変更が可能（コード変更不要）

## セットアップ手順

### 1. Firebase プロジェクトの作成

1. [Firebase Console](https://console.firebase.google.com/) にアクセス
2. 「プロジェクトを追加」をクリック
3. プロジェクト名を入力（例: `exe-lounge`）
4. Google アナリティクスは無効でOK → 「プロジェクトを作成」

### 2. Realtime Database の作成

1. 左メニューの「構築」→「Realtime Database」をクリック
2. 「データベースを作成」をクリック
3. ロケーション: `asia-southeast1`（シンガポール）を推奨
4. 「**テストモードで開始**」を選択 → 「有効にする」

### 3. セキュリティルールの設定

Realtime Database の「ルール」タブで以下を設定:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

> ⚠️ これは全員が読み書き可能な設定です。URLを知っている人だけがアクセスできるため、社内利用であれば問題ありません。より制限したい場合は後述の「セキュリティ強化」を参照してください。

### 4. Firebase の設定情報を取得

1. Firebase Console → ⚙️（プロジェクト設定）→「全般」
2. 「マイアプリ」セクションで「ウェブアプリを追加」(</> アイコン)をクリック
3. アプリ名を入力（例: `exe-lounge-web`）→「アプリを登録」
4. 表示される設定情報をメモ:
   - `apiKey`
   - `authDomain`
   - `databaseURL`
   - `projectId`

### 5. GitHub Pages でホスティング

1. このリポジトリをGitHubにプッシュ
2. リポジトリの Settings → Pages
3. Source: `Deploy from a branch`
4. Branch: `main` (または `master`) → `/ (root)` → Save
5. 数分後にURLが発行される（例: `https://username.github.io/EXENEW/`）

### 6. 初回設定

1. ブラウザで GitHub Pages のURL を開く
2. Firebase の設定情報入力画面が表示される
3. 手順4でメモした情報を入力して「設定を保存して開始」
4. ※ この設定は各デバイスのブラウザに保存されます（iPad・PCそれぞれで1回だけ必要）

### 7. 座席の登録

1. `?mode=admin` で管理画面を開く（例: `https://.../?mode=admin`）
2. まず「座席タイプ管理」でタイプを追加:
   - 例: ID=`table`, 表示名=`テーブル`, 色=緑, 順序=1
   - 例: ID=`counter`, 表示名=`カウンター`, 色=青, 順序=2
   - 例: ID=`sofa`, 表示名=`ソファ`, 色=オレンジ, 順序=3
3. 次に「座席管理」で座席を追加:
   - 例: ID=`T1`, 表示名=`テーブル1`, タイプ=テーブル, 順序=1
   - 例: ID=`T2`, 表示名=`テーブル2`, タイプ=テーブル, 順序=2
   - 以下、必要な分だけ繰り返す

## 使い方

### iPad（中のスタッフ）

```
https://username.github.io/EXENEW/?mode=staff
```

- 座席ボタンをタップすると「空き ⇔ 使用中」が切り替わる
- 緑 = 空き、赤 = 使用中
- 「全席を空きにする」ボタンで一括リセット（閉店時など）

### PC（入口のスタッフ）

```
https://username.github.io/EXENEW/?mode=entrance
```

- 自動的にリアルタイム更新（操作不要）
- 空き数のサマリーが上部に表示
- 使用時間（「45分」など）が表示される
- 90分以上利用中の席は黄色枠で強調
- 席が空いた時は通知音 + 緑色アニメーション

### 管理画面

```
https://username.github.io/EXENEW/?mode=admin
```

- 座席タイプの追加・削除
- 座席の追加・削除
- ラウンジ名の変更
- Firebase接続設定のリセット

## セキュリティ強化（オプション）

URLを知っている人のみアクセスできますが、さらにセキュリティを高めたい場合:

### 方法1: Database Rules で読み取りのみ許可

```json
{
  "rules": {
    "seats": {
      ".read": true,
      ".write": true
    },
    "seatTypes": {
      ".read": true,
      ".write": true
    },
    "settings": {
      ".read": true,
      ".write": true
    },
    "log": {
      ".read": true,
      ".write": true
    }
  }
}
```

### 方法2: Firebase Authentication を有効にする

Firebase Authentication で匿名認証を有効にし、ルールで認証済みユーザーのみ許可:

```json
{
  "rules": {
    ".read": "auth != null",
    ".write": "auth != null"
  }
}
```

## トラブルシューティング

### 画面が表示されない
- Firebase の設定情報が正しいか確認
- ブラウザのコンソール（F12）でエラーメッセージを確認
- 「Firebase設定リセット」で再設定を試す

### リアルタイム更新されない
- 接続状況バナー（赤い帯）が表示されていないか確認
- VPNで `*.firebaseio.com` がブロックされていないか確認
- ブラウザをリロード

### 通知音が鳴らない
- ブラウザの自動再生ポリシーにより、最初の一回は画面をクリック/タップしてから音が出ます
- Edge の設定でサイトの音声が許可されているか確認

## 技術仕様

- **フロントエンド**: 単一HTMLファイル（バニラJS、フレームワーク不使用）
- **バックエンド**: Firebase Realtime Database（サーバーレス）
- **ホスティング**: GitHub Pages（静的サイト）
- **通信**: Firebase WebSocket（リアルタイム双方向通信）
- **対応ブラウザ**: Edge, Chrome, Safari (iPad)
