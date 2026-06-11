# Universal Links — eticket.rocketbase.co.jp (GitHub Pages 構成)

このディレクトリの内容を GitHub Pages にデプロイし、サブドメイン `eticket.rocketbase.co.jp` でアクセスできるようにする。

## ファイル構成

```text
aasa/
├── .well-known/
│   └── apple-app-site-association   # Universal Links 用 AASA
├── .nojekyll                         # Jekyll 無効化（.well-known を配信するため必須）
├── CNAME                             # GitHub Pages のカスタムドメイン設定
├── index.html                        # アプリ未インストール時のフォールバックページ
└── README.md
```

## GitHub Pages へのデプロイ手順

### 1. 新規リポジトリを作成

例: `rocketeticket-pages`（名前は何でも OK）。Private でも Public でもどちらも可。

### 2. ファイルを push

```bash
cd /Users/isao/work/rokcet/rocket_eticket/aasa
git init
git add -A
git commit -m "Initial AASA"
git branch -M main
git remote add origin git@github.com:<YOUR_USERNAME>/<REPO_NAME>.git
git push -u origin main
```

### 3. GitHub Pages を有効化

リポジトリ > Settings > Pages:

- **Source**: Deploy from a branch
- **Branch**: `main` / root (`/`)
- 保存

数十秒〜数分でビルド完了。

### 4. カスタムドメインを設定

リポジトリ > Settings > Pages > Custom domain:

- `eticket.rocketbase.co.jp` を入力 → Save
- 「Enforce HTTPS」にチェック（DNS 反映後に有効化できる）

`CNAME` ファイルが既にリポジトリにあるので、GitHub 側で自動的にこれを認識。

### 5. DNS に CNAME レコードを追加

`rocketbase.co.jp` の DNS 管理画面で:

```text
タイプ: CNAME
名前: eticket
値:   <YOUR_USERNAME>.github.io
TTL:  300 など短めから
```

DNS の伝播後（数分〜数十分）、`https://eticket.rocketbase.co.jp/` が GitHub Pages に向く。
GitHub が証明書を自動発行するので、HTTPS も使えるようになる。

### 6. 動作確認

```bash
# AASA が返ってくるか
curl https://eticket.rocketbase.co.jp/.well-known/apple-app-site-association

# ステータスコード確認（リダイレクトしていないこと、200 OK であること）
curl -I https://eticket.rocketbase.co.jp/.well-known/apple-app-site-association

# フォールバックページ確認
curl -I https://eticket.rocketbase.co.jp/
```

Apple/Branch の AASA バリデータ: [branch.io/resources/aasa-validator/](https://branch.io/resources/aasa-validator/)

## AASA の中身（参考）

```json
{
  "applinks": {
    "details": [
      {
        "appIDs": ["WK649QF382.rocketbase.Rocket-eTicket"],
        "components": [
          {
            "/": "/ticket/*"
          }
        ]
      }
    ]
  }
}
```

- `WK649QF382`: Apple Developer Team ID
- `rocketbase.Rocket-eTicket`: アプリの Bundle ID
- `/ticket/*`: このパスのみアプリで開く

## 注意点

1. **Content-Type**: GitHub Pages は `apple-app-site-association` を `application/octet-stream` 等で配信することがあるが、iOS は実質受け入れる。気になる場合は別のホスティングへ
2. **キャッシュ**: 一度 AASA を取得した端末は数時間〜数日キャッシュする。設定を変えた場合、即時反映は期待しない（端末再起動 + アプリ再インストールで確実）
3. **App Store 公開後の動作**: 開発中はキャッシュやプロビジョニングの影響でうまく動かないこともある。実機 + リリース署名 + AASA 配信が揃って初めて完全動作

## 関連するアプリ側の設定（既に対応済み）

- `Rocket_eTicket.entitlements`: `applinks:eticket.rocketbase.co.jp` 追加
- `CalendarManager`: カレンダーイベントの URL を `https://eticket.rocketbase.co.jp/ticket/{UUID}` に
- `Rocket_eTicketApp`: `onContinueUserActivity` で Universal Link を受け取り
- `ContentView`: handleDeepLink で https と旧 `rocketeticket://` の両方をパース
