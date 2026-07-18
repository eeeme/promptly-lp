# Promptly LP — 公開手順とローンチ・チェックリスト

この1フォルダに、ローンチに必要な素材が全部入っています。

```
promptly-lp/
├── index.html                LP本体
├── style.css                 共通スタイル
├── legal/
│   ├── terms.html            利用規約
│   ├── privacy.html          プライバシーポリシー
│   └── tokushoho.html        特定商取引法に基づく表記
├── assets/
│   ├── icon-512/128/48/32/16.png   ロゴアイコン各サイズ
│   ├── mark-transparent.png        透過ロゴ
│   └── ogp.png                     SNSシェア画像 (1200x630)
├── store-listing.md          Chrome Web Store 申請文 (コピペ用)
├── スクショ・動画ガイド.md     スクショ5枚+30秒動画の作り方
└── README-公開手順.md         このファイル
```

---

## STEP 0: 差し替え箇所 (公開前に必ず)

以下のプレースホルダーを、エディタの一括置換で埋めてください。

| プレースホルダー | 置き換える内容 | 出現ファイル |
|---|---|---|
| 【氏名を記入】 | Stripe登録と同じ氏名 | terms / privacy / tokushoho |
| 【メールアドレスを記入】 | 専用Gmail | index / terms / privacy / tokushoho |

購入リンクは現在**テスト用Stripeリンク**が入っています
(index.html 内 2箇所)。本番移行時に差し替えます (STEP 4)。

---

## STEP 1: GitHub Pages で公開 (15分)

Stripe審査用に作った仮LPと同じリポジトリを使い回してOKです。

1. リポジトリ `promptly-lp` を開く (なければ Public で新規作成)
2. このフォルダの中身をすべてアップロード
   - GitHubの Web UI なら「Add file → Upload files」にドラッグ&ドロップ
   - 既存の index.html は上書き
3. Settings → Pages → Branch: main / root → Save
4. 数分後、以下で公開される:
   `https://<ユーザー名>.github.io/promptly-lp/`
5. スマホでも表示確認 (レスポンシブ対応済み)

### 独自ドメインを使う場合 (任意、推奨)

1. ドメイン取得: Cloudflare Registrar か お名前.com で
   `promptly.jp` `getpromptly.app` など (年1,000〜3,000円)
2. GitHub Pages の Settings → Pages → Custom domain に入力
3. DNS に CNAME レコード追加 (`<ユーザー名>.github.io` を指す)
4. 「Enforce HTTPS」にチェック

※独自ドメインは必須ではありません。github.io のままでもストア審査・
   販売とも問題なし。後から移行も可能です。

---

## STEP 2: Chrome Web Store 申請 (30分 + 審査1〜3日)

0. (推奨) 拡張機能のアイコンを新ロゴに差し替え:
   このフォルダの `assets/icon-512.png` を
   `01_promptly/assets/icon.png` に**上書きコピー** (ファイル名は icon.png のまま)
   → Plasmoが全サイズを自動生成します
1. スクショ5枚を撮影 (→ スクショ・動画ガイド.md)
2. 拡張機能を本番ビルド:
   `cd 01_promptly && npm run build`
3. `build/chrome-mv3-prod` フォルダの中身を **zip化**
   (フォルダごとではなく、manifest.json が zip 直下に来るように)
4. https://chrome.google.com/webstore/devconsole にアクセス
   → デベロッパー登録 ($5、Google アカウントで)
5. 「新しいアイテム」→ zip をアップロード
6. store-listing.md の内容を各欄にコピペ
   - ストア掲載情報 (名前・説明・カテゴリ)
   - スクショをアップロード
   - プライバシー: 権限の理由、データ収集の申告
   - プライバシーポリシーURL (STEP 1 で公開したURL)
7. 「審査のため送信」

---

## STEP 3: Stripe 本番モードの準備 (審査通過後、30分)

Stripeの本人確認審査が通ったら:

1. ダッシュボード右上を「テスト」→「本番」に切り替え
2. 商品カタログ → 「Promptly Pro ¥3,980 買い切り」を本番でも作成
3. 決済リンクを作成 → 本番リンク (buy.stripe.com/live_... 形式) を控える
4. Webhook を本番でも登録:
   - URL: `https://promptly-api.promptly-dev2026.workers.dev/webhook/stripe`
   - イベント: checkout.session.completed
   - 発行された whsec_ を控える
5. Workers のシークレットを本番用に更新:
   ```
   cd promptly-api
   npx wrangler secret put STRIPE_WEBHOOK_SECRET
   ```
   (本番の whsec_ を入力)

※テスト用Webhookは削除するか、そのままでもOK (署名が合わず無視される)

---

## STEP 4: 本番リンクへの差し替え (15分)

1. **LP**: index.html 内の `buy.stripe.com/test_...` (2箇所) を
   本番リンクに置換 → GitHubにpush
2. **拡張機能**: `01_promptly/src/lib/config.ts` の
   `PURCHASE_URL` を本番リンクに変更
3. 再ビルド → 新しいzipをChrome Web Storeに
   「パッケージ更新」としてアップロード (バージョンを 0.2.1 等に上げる)
4. ストア公開後、index.html の「Chromeに追加 (準備中)」ボタン2箇所を
   ストアURLに差し替え:
   ```html
   <a class="btn btn-primary" href="https://chromewebstore.google.com/detail/xxxx">Chrome に追加</a>
   ```

---

## STEP 5: 本番E2Eテスト (必須、15分)

自分のカードで**本番購入を1回**行い、以下を確認:

- [ ] 決済完了 → ライセンスキーがメールで届く
- [ ] キーで拡張機能がPro化できる
- [ ] Stripe管理画面に売上が計上されている
- [ ] (確認後、Stripeから自分に返金してもOK。手数料分だけ負担)

※本番のResendはテストドメインのままだと自分以外に届きません。
   販売開始前に、独自ドメインを取ったら Resend でドメイン認証
   → promptly-api/src/email.ts の FROM_ADDRESS を変更 → deploy。
   ドメインをまだ取らない場合、Resendの無料枠で
   「検証済みの自分のドメインなし運用」は不可のため、
   ドメイン取得を先に済ませてください (STEP 1 の独自ドメインと共用可)。

---

## STEP 6: ローンチ告知

- [ ] X に投稿 (デモ動画添付、ストアリンク)
- [ ] Zenn 記事「Chrome拡張を個人開発して公開した話」(技術記事は伸びる)
- [ ] AI系の発信者にDM (10〜20名、無料キー進呈と引き換えに紹介依頼)

---

## よくある詰まりどころ

| 症状 | 対処 |
|---|---|
| GitHub Pagesが404 | 反映に数分かかる。Settings→Pagesでbranch指定を再確認 |
| ストア審査リジェクト | 理由メールの文面を共有してください。ほぼ文言修正で通ります |
| 本番購入でメール不達 | Resendのドメイン認証が済んでいるか、FROM_ADDRESSを変えたか確認 |
| LPのOGPが反映されない | X Card Validator でキャッシュ更新。og:image は絶対URLに変更推奨 |
