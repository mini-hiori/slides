---
marp: true

---
<!-- 
theme: default
size: 16:9
paginate: true
style: |
  section {
    background-color: #FFFFFF;
  }
-->
# サーバーレスなRSSリーダーをつくる
#python
#AWS
#Lambda
#Docker
#Github Actions
#VSCode Remote Container

<div style="text-align:right">@mini_koharu</div>

---

# もくじ
- やりたいこと
- やってみる
    - 開発環境
    - RSS取得
    - デプロイ
    - Lambdaをつくる
    - RSS取得先URLを設定する
    - 定期実行

---
# やりたいこと1/3

<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/discord-webhook-example.png" width=60%>

---
# やりたいこと2/3
## **†新しそうなこと全部やる†**
- Lambdaをコンテナで動かす
- Lambdaを定期実行する(Eventbridge)
- コンテナで開発する(VSCode Remote Container)
- CIの実装(Github Actions+ECR)
- クロール先URLをハードコーディングしない  
(Systems Managerパラメータストア)

---

# やりたいこと3/3(構成図)

![](https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/architecture.png)

---

# やってみる:開発環境
- VSCode Remote Containerを使う
    - コンテナの中で開発ができる
    - ローカルにはdocker(-compose)とVSCodeだけ入っていればよい
    - Lambdaで動かすコンテナでそのまま開発できる
      - 実行環境と開発環境が完全に一致

<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/vscode-remote-container.png" width=50%>

---

# やってみる:RSS取得
- RSSをとってくる
  - pythonのfeedparserというライブラリが使える
```
def get_rss(url: str) -> List[RssContent]:
    feed = feedparser.parse(url)
    rss_list: List[RssContent] = []
    for entry in feed.entries:
        if not entry.get("link"):
            continue
        rss_content = RssContent(
            title=entry.title,
            url=entry.link
        )
        rss_list.append(rss_content)
    return rss_list
```

---


# やってみる:デプロイ
- GithubへのpushをトリガーにGithub Actionsが走り、  
自動でコンテナをビルド→ECRへアップロード
  - 参考:[GitHub ActionでDockerコンテナをビルドしてAmazon ECRに保存する](https://dev.classmethod.jp/articles/github-action-ecr-push/)

- Github Actions #とは
  - Github上でCI/CD操作を実行できる機能(設定はyaml)
  - ほぼ無料
  - [AWS公式のActionsが公開されている](https://aws.amazon.com/jp/blogs/opensource/github-actions-aws-fargate/)

<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/github-actions.png" width=40%>


---

# やってみる:Lambdaをつくる
- コンテナ利用の場合もGUIからつくれる
- ソースコードやzipをアップロードする代わりにECRのURIを指定する
<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/lambda-config.png" width=60%>


---

# やってみる:RSS取得先URLを設定する
- Systems Managerパラメータストアを利用する
  - デプロイし直さなくてもRSS取得先URLをGUIから追加/削除できる

<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/ssm-params.png" width=60%>

- パラメータ内の改行は保存される
- パラメータの取得はboto3(python用aws接続ライブラリ)から

---

# やってみる:定期実行
- EventbridgeをLambdaのトリガーとして利用する
<img src="https://raw.githubusercontent.com/mini-hiori/zenn-content/main/images/lambda-rss-reader-bot/lambda-with-eventbridge.png" width=60%>

- 周期はrate(1 hour)
  - 開始タイミングは指定できないらしい

---

# おわり
- おまけ
  - このスライドは[Marp](https://yhatt.github.io/marp/)による
    - markdownからスライドが生成できる 
    - [画像はhtmlタグで埋め込むのがよい](https://qiita.com/no-use-kuro/items/411a6c8a7385b9e7f470)