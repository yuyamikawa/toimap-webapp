# うごく問いの地図 — Firebase（Firestore）でリアルタイム共有

> Supabase無料は「7日アクセスがないと一時停止 → 放置で削除」されますが、**Firebase（Firestore）は放置でもデータが消えません**。
> Blazeプラン（従量課金）でも、無料枠の範囲なら**実質0円**で運用できます。所要 15〜20分。

`D-02_うごく問いの地図.html` は Firebase / Supabase / ローカルを自動で切替えます。
**`FB.apiKey` を入れれば Firebase 優先**、空なら Supabase、どちらも空ならローカル保存。

---

## STEP 1 — Firebaseプロジェクトを作る（3分）

1. https://console.firebase.google.com → **プロジェクトを追加**。
2. Google アナリティクスは「オフ」でOK（不要）。

---

## STEP 2 — Firestore を有効化（2分）

1. 左メニュー **構築 → Firestore Database → データベースの作成**。
2. ロケーションは **asia-northeast1（東京）** を推奨。
3. モードは一旦「本番」でOK（ルールは STEP 4 で設定）。

---

## STEP 3 — Blaze（従量課金）に上げる＋予算アラート（5分）

> Firestore自体は無料枠（1日あたり 読取5万・書込2万・削除2万、保存1GiB）が大きく、今回の用途なら**ほぼ0円**ですが、Blazeにすると上限解放＋将来も安心です。

1. 左下 **プランをアップグレード → Blaze** を選択（請求先＝クレジットカード登録が必要）。
2. **必ず予算アラートを設定**：Google Cloud Console → 「お支払い」→「予算とアラート」→ 低額（例 ¥500）でアラートを作成。万一の暴走課金を早期に検知できます。

> 100人・1日のイベント程度では無料枠内に収まり、実費はほぼ発生しません。

---

## STEP 4 — セキュリティルール（誰でも閲覧・投稿だけ）（2分）

**Firestore Database → ルール** を以下にして「公開」：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /voices/{id} {
      allow read:   if true;     // 誰でも閲覧
      allow create: if true;     // 誰でも投稿
      allow update, delete: if false;  // 改ざん・削除は不可
    }
  }
}
```

---

## STEP 5 — 接続情報をHTMLに入れる（2分）

1. Firebase Console **プロジェクトの設定（歯車）→ 全般 → マイアプリ → ウェブアプリ（</>）を追加** → 表示される `firebaseConfig` を控える。
2. `D-02_うごく問いの地図.html` 先頭の `FB` を書き換え：

```js
const FB = {
  apiKey:"AIza...",
  authDomain:"xxxx.firebaseapp.com",
  projectId:"xxxx",
  appId:"1:xxxx:web:xxxx"
};
```

> これで起動時に Firestore から読み込み、投影はリアルタイム購読、スマホは投稿のみ（接続を張らない）で動きます。

---

## STEP 6 — 公開する（5分）

どちらでもOK：

- **手持ちのレンタルサーバー**：HTMLを置くだけ（**HTTPS必須**）。データはFirestoreなので、ホスティングはレン鯖のままで問題ありません。
- **Firebase Hosting（無料・HTTPS・CDN・容量気にせず）**：
  ```bash
  npm i -g firebase-tools
  firebase login
  firebase init hosting     # 公開ディレクトリにHTMLを index.html として配置
  firebase deploy
  ```

---

## STEP 7 — 投影とスマホで出し分け（QR）

| 用途 | URL | 表示・接続 |
|------|-----|-----------|
| 投影PC | `…/?view=board` | 地図のみ・**ライブ購読あり** |
| 参加者スマホ | `…/` または `…/?view=submit` | フォームのみ・**投稿だけ（常時接続なし）** |
| 手元PC（両方） | `…/` | 地図＋フォーム |

- スマホは画面幅で自動的に「投稿のみ」レイアウト＋接続を張らない動作になります（地図・全画面ボタン・英タイトルは非表示）。
- スマホ用URLを **QRコード**化して配布。投影は `?view=board` を全画面（F キー）。

---

## STEP 8 — 本番前チェック（3分）

- スマホ2台で投稿 → 投影（`?view=board`）に**即反映**されるか。
- 投稿は名前・所属が必須。地図上の声に**ホバーで本文**が出るか。
- 同名で複数投稿 → **同色アイコン**で並ぶか。

---

## 運用メモ

- **放置してもデータは残ります**（Firestoreは無操作でも削除されません）。会期後もそのまま保管 → Console からエクスポート、または部会のNotion/Slackへ転記。
- **通信**：会場のWi‑Fi/回線が必要（Firestore接続とフォント読込のため）。100台一斉接続は回線側がボトルネックになりやすいので、当日の回線確認を推奨。
- **コスト管理**：予算アラートを必ず設定。無料枠内ならほぼ0円。
- **匿名性**：apiKey はフロント公開前提（Firebaseの設計どおり。ルールで読・投稿のみに制限済み）。本文に個人情報を書かない旨を一言添えると安心。
- **荒れ対策（任意）**：合言葉・表示期間制御・簡易モデレーション等が必要なら実装できます。
