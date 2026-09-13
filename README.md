# kmykprn.github.io

ユーザーサイト。**`/__/auth/` に Firebase Authentication のサインイン用ヘルパーを自前で配信する**ためにある。

## なぜここに要るのか

roomplanner-web（`kmykprn.github.io/roomplanner-web/`）の Google ログインは Firebase Auth を使う。
Firebase の既定では認証の戻り先が `project-….firebaseapp.com` になり、アプリとは**別オリジン**。
Safari 16.1+ / Firefox 109+ / Chrome 115+ はサードパーティの保存領域を分断するため、
別オリジンだと iOS Safari でログイン結果をアプリに持ち帰れない（実機で再現済み）。

Firebase が公式に案内する回避策のうち、静的配信だけで済むのが
「ヘルパーを自分のドメインで配る」方法。SDK は `https://<authDomain>/__/auth/handler` を
**ドメイン直下**に見に行くので、プロジェクトサイトではなくユーザーサイトに置く。
https://firebase.google.com/docs/auth/web/redirect-best-practices

## 中身

`__/auth/` の 7 ファイルは `https://project-db31f07b-2895-48b8-8bb.firebaseapp.com/__/auth/` から
取得したものをそのまま置いている。**手を加えていない。**

`/__/firebase/init.json` は置いていない。ハンドラは設定を URL クエリで受け取るので不要で、
Google 側の配信でも 404 のまま動いている。API キーをこのリポジトリに入れないためでもある。

`.nojekyll` は必須。無いと GitHub Pages の Jekyll が `_` で始まるディレクトリを落とす。

## 更新

Google 側のヘルパーは予告なく更新される。**ここは取得時点のスナップショット**なので、
定期的（目安: 四半期）に取り直す。差分が出たらそのまま置き換えて push する。

```sh
S=https://project-db31f07b-2895-48b8-8bb.firebaseapp.com/__/auth
for f in handler handler.js experiments.js iframe iframe.js links links.js; do
  curl -sS -o __/auth/$f $S/$f
done
git diff --stat
```

## 対応しないもの

Apple サインインと SAML はこの方法では動かない（Firebase の注記）。使っていない。
