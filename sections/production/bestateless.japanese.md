# ステートレスのままで、ほぼ毎日サーバーを停止させる

<br/><br/>

### 一段落説明

1台のサーバーに設定やデータの一部が欠落しているという深刻な運用上の問題に遭遇したことはありませんか？ これはおそらく、デプロイメントの一部ではないローカルアセットへの不必要な依存が原因であると思われます。多くの成功しているプロダクトが不死鳥のようにサーバーを扱います – それは何のダメージも受けずに死んで定期的に生まれ変わります。言い換えれば、サーバーとは、コードをしばらくの間実行して、その後に入れ替わるハードウェアの一部に過ぎません。
このアプローチは、

- サーバを動的に追加したり削除したりすることで、副作用のないスケーリングを可能にします。
- 各サーバの状態を評価することから解放されるので、メンテナンスが簡単になります。

<br/><br/>

### コード例: アンチパターン

```javascript
// 典型的な間違い 1: アップロードファイルのローカル保存
const multer = require('multer'); // マルチパートアップロード処理用高速ミドルウェア
const upload = multer({ dest: 'uploads/' });

app.post('/photos/upload', upload.array('photos', 12), (req, res, next) => {});

// 典型的な間違い 2: 認証セッション(パスポート)をローカルファイルまたはメモリに保存する
const FileStore = require('session-file-store')(session);
app.use(session({
    store: new FileStore(options),
    secret: 'keyboard cat'
}));

// 典型的な間違い 3: グローバルオブジェクトの情報を格納
Global.someCacheLike.result = { somedata };
```

<br/><br/>

### 他のブロガーが言っていること

ブログ [Martin Fowler](https://martinfowler.com/bliki/PhoenixServer.html) より:
> ...ある日、私は運用のための認証サービスを始めたいという妄想を抱いた。認定審査は、同僚と私が会社のデータセンターに出向き、野球のバット、チェーンソー、水鉄砲を持って、重要な本番用サーバーを見て回るというものでした。評価は、運用チームがすべてのアプリケーションを再稼働させるまでにどれくらいの時間がかかるかに基づいて行われます。くだらない妄想かもしれませんが、ここには名言のヒントがあります。野球のバットは見送るべきですが、定期的にサーバーを仮想的に焼き尽くすのは良いアイデアです。サーバーは不死鳥のように灰の中から定期的に立ち上がるべきです...

<br/><br/>
