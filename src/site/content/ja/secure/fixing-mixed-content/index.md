---
layout: post
title: 混在コンテンツの修正
authors:
  - johyphenel
  - rachelandrew
date: 2019-09-07
updated: 2020-09-23
description: ユーザーを保護し、すべてのコンテンツを確実に読み込めるよう、ウェブサイト上で発生する混在コンテンツのエラーを修正する方法について解説します。
tags:
  - security
  - network
  - privacy
  - html
  - css
  - javascript
  - images
  - media
---

ウェブサイトでHTTPSをサポートすることは、サイトとユーザーを攻撃から保護するための重要なステップですが、コンテンツが混在していると、その保護の効果が損なわれてしまいます。[What is mixed content? (混合コンテンツとは？)](/what-is-mixed-content)でも説明があるように、安全でない混在コンテンツはブラウザによってブロックされます。

このガイドでは、混在コンテンツに関する既存の問題を修正し、新しい問題の発生を防ぐための手法とツールをご紹介します。

## 自身のサイトにアクセスして混在コンテンツを見つける

Google ChromeでHTTPSのページにアクセスすると、ブラウザは、混在コンテンツある場合、JavaScriptコンソール内でそれをエラーおよび警告として表示します。

[What is mixed content? (混在コンテンツとは？)](/what-is-mixed-content)では、数多くの例に加え、Chrome DevToolsで問題が報告される仕組みを紹介しています。

[パッシブ混在コンテンツ](https://passive-mixed-content.glitch.me/)の例では、次のような警告が表示されます。ブラウザは、`https` URLでコンテンツを見つけることができれば、それを自動的に更新してからメッセージを表示します。

<figure>{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/Y7b4EWAbSL6BgI07FdQq.jpg", alt="混在コンテンツが検出、更新されたときに Chrom DevToolsで表示される警告", width="800", height="294" %}</figure>

[アクティブな混在コンテンツ](https://active-mixed-content.glitch.me/)はブロックされ、以下のような警告が表示されます。

<figure>{% Img src="image/tcFciHGuF3MxnTr1y5ue01OGLBn2/KafrfEz1adCP2eUHQEWy.jpg", alt="アクティブな混在コンテンツがブロックされたときにChrome DevToolsに表示される警告", width="800", height="304" %}</figure>

サイトの`http://` URLについて、こうした警告が見つかった場合は、サイトのソースで修正する必要があります。こうしたURLをそれぞれが見つかったページと一緒にリストアップしておけば、修正するときに便利です。

{% Aside %}混在コンテンツのエラーと警告は、現在表示しているページに対してのみ表示され、新しいページに移動すると、毎回JavaScriptコンソールはクリアされます。つまり、こうしたエラーを見つけるには、サイトの各ページを表示する必要があります。{% endAside %}

### サイト内の混在コンテンツを見つける

混在コンテンツは、ソースコードの中で直接検索できます。ソースで`http://`を検索し、HTTP URL属性を含むタグを探します。アンカータグ (`<a>`)の`href`に`http://`があっても、それは混在コンテンツの問題ではない場合がしばしばあります。注目すべき例外がいくつかありますが、それらについては後ほど解説します。

サイトがコンテンツ管理システムを使って公開されている場合は、ページが公開された時に安全でないURLへのリンクが挿入されている可能性があります。たとえば、画像に、相対パスではなく、完全なURLが含まれる場合などがあります。これは、CMSコンテンツ内で見つけて修正する必要があります。

### 混在コンテンツの修正

サイトのソースに混在コンテンツを見つけたら、次のステップで修正します。

リソースリクエストがHTTPからHTTPSに自動的に更新されたというコンソールメッセージが表示された場合は、コード内のリソースの`http://`を安全に`https://`に変更できます。また、ブラウザのURLバーの`http://`を`https://`に変更し、そのURLをブラウザのタブで開けば、リソースが利用できるかどうかを安全に確認できます。

リソースが`https://`経由で利用できない場合は、次のいずれかのオプションを検討することをおすすめします。

- 別のホストのリソースを含める (利用可能な場合)。
- コンテンツをダウンロードして、自分のサイトで直接ホスティングする (法的に許可されている場合)。
- サイトからリソースを完全に除外する。

問題を修正したら、最初にエラーを見つけたページを表示して、エラーが表示されなくなったことを確認します。

### タグの非標準的な使い方に注意する

サイトで非標準的な使われ方をしているタグに注意してください。たとえば、アンカー (`<a>`) タグのURLは、ブラウザを新しいページに移動させるものであるため、混在コンテンツのエラーにはなりません。つまり、通常なら、これを修正する必要はありません。ただし、一部の画像ギャラリースクリプトは、`<a>`タグの機能をオーバーライドし、`href`属性が指定するHTTPリソースをページのライトボックスのディスプレイに読み込んでしまうため、混在コンテンツの問題を引き起こします。

## 混在コンテンツを広範に処理する

小規模なウェブサイトであれば、こうした手動による手順でも事足りるでしょう。ただし、サイトが大規模なものであったり、多数の開発チームが関与するものであれば、そのサイトに読み込まれるすべてのコンテンツを追跡するのは困難になります。この場合、コンテンツセキュリティポリシーを使って、ブラウザに混在コンテンツについて通知するよう指示を出せば、安全でないリソースが予期せぬかたちでページに読み込まれることを確実に防げます。

### コンテンツセキュリティポリシー

[Content security policy (コンテンツセキュリティポリシー)](https://developers.google.com/web/fundamentals/security/csp/) (CSP) は、混在コンテンツを大規模に管理する際に便利な多目的ブラウザ機能です。CSPレポートメカニズムを使用すれば、サイトの混在コンテンツを追跡し、混在コンテンツを更新またはブロックすることで、ユーザーを保護するための実施ポリシーを提供できます。

対象となるページでこうした機能を有効化するには、サーバーから送信されるレスポンスに`Content-Security-Policy`または`Content-Security-Policy-Report-Only`ヘッダーを含めます。さらに、ページの`<head>`セクションに`<meta>`タグを使えば、`Content-Security-Policy`を設定できます (`Content-Security-Policy-Report-Only`は**設定できません**)。

{% Aside %}最新のブラウザでは、受信するすべてのコンテンツセキュリティポリシーが実施されます。ブラウザがレスポンスヘッダーまたは`<meta>`要素に受信する複数のCSPヘッダー値は結合され、単一のポリシーとして実施されます。報告ポリシーも同様に結合されます。ポリシーは、各ポリシーの共通部分を取り入れることによって結合されます。つまり、最初のポリシーに続く各ポリシーは、許可されるコンテンツの幅を広げるのではなく、どんどん制限していくということです。{% endAside %}

### コンテンツセキュリティポリシーを使用して混在コンテンツを見つける

コンテンツセキュリティポリシーを使用すれば、サイトで見つかった混在コンテンツのレポートを収集できます。この機能を有効にするには、 `Content-Security-Policy-Report-Only`ディレクティブをサイトのレスポンスヘッダーとして追加し、設定します。

レスポンスヘッダー：

`Content-Security-Policy-Report-Only: default-src https: 'unsafe-inline' 'unsafe-eval'; report-uri https://example.com/reportingEndpoint`

{% Aside %} [report-uri](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-uri)レスポンスヘッダーは近々非推奨となり、代わりに[report-to](https://developer.mozilla.org/docs/Web/HTTP/Headers/Content-Security-Policy/report-to)が優先されます。`report-to`のブラウザサポートは現在、ChromeとEdgeに限定されています。両方のヘッダーを指定できますが、ブラウザが `report-to`をサポートしている場合、`report-to`サポートしている場合、`report-uri`は無視されます。{% endAside %}

ユーザーがサイトのページにアクセスする際に、コンテンツセキュリティポリシーに違反するものがあれば、ブラウザはそれについてJSON形式のレポートを`https://example.com/reportingEndpoint`に送信します。この場合、サブリソースがHTTPを介して読み込まれるときは、毎回レポートが送信されます。このレポートには、ポリシー違反が発生したページのURLとポリシーに違反したサブリソースのURLが含まれます。こうしたレポートをログに記録するためのレポートエンドポイントを設定している場合は、自ら各ページにアクセスしなくても、サイトの混在コンテンツを追跡できます。

注意点が2つあります。

- ユーザーは、CSPヘッダーを理解できるブラウザーでページにアクセスする必要があります。最新のブラウザの多くについて言えることです。
- レポートは、ユーザーがアクセスしたページの分しか取得できません。トラフィックの少ないページがある場合は、サイト全体のレポートを取得するまでに少し時間がかかるかもしれません。

[Content security policy (コンテンツセキュリティポリシー)](https://developers.google.com/web/fundamentals/security/csp/) ガイドに詳しい情報とエンドポイントの例を記載しています。

### CSPを使用したレポートの代替手段

サイトがBloggerなどのプラットフォームでホストされている場合は、ヘッダーを変更したりCSPを追加したりするためのアクセス権を持っていない場合があります。代わりに、実行可能な代案として、[HTTPSChecker](https://httpschecker.net/how-it-works#httpsChecker)や[Mixed Content Scan](https://github.com/bramus/mixed-content-scan)といったウェブサイトクローラーを用いて、サイト全体で問題を探るということができます。

### 安全でないリクエストの更新

ブラウザは、安全でないリクエストを更新したり、ブロックしたりするようになりつつあります。CSPディレクティブを使用すれば、こうしたアセットを自動的に更新したり、ブロックしたりできます。

[`upgrade-insecure-requests`](https://www.w3.org/TR/upgrade-insecure-requests/) CSPディレクティブを使用すると、ネットワーク要求を行う前に、安全でないURLを更新するという指示がブラウザに出されます。

たとえば、ページに`<img src="http://example.com/image.jpg">`といったHTTP URLを持つ画像タグが含まれている場合。

ブラウザは、代わりに、`https://example.com/image.jpg`に対する安全なリクエストを行うため、ユーザーは混在コンテンツから保護されます。

この動作を有効にするには、次のディレクティブを指定した`Content-Security-Policy`を送信します。

```markup
Content-Security-Policy: upgrade-insecure-requests
```

もしくは、`<meta>`要素を使って、ドキュメントの`<head>`セクションの中に同じディレクティブを埋め込みます。

```html
<meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
```

ブラウザによる自動更新と同様に、リソースがHTTPS経由で利用できない場合、更新されたリクエストは失敗となり、リソースも読み込まれません。こうして、ページのセキュリティは維持されます。`upgrade-insecure-requests`ディレクティブは、ブラウザが現在更新していない要求までも更新しようと試みるため、ブラウザによる自動更新以上の働きをしてくれます。

`upgrade-insecure-requests`は、`<iframe>`ドキュメントにカスケードするため、ページ全体の安全が保証されます。

### すべての混在コンテンツをブロックする

ユーザーを保護するための代替の選択肢に、[`block-all-mixed-content`](https://www.w3.org/TR/mixed-content/#strict-checking) CSPディレクティブがあります。このディレクティブは、ブラウザに混在コンテンツを決して読み込まないという指示を出します。アクティブとパッシブの両方を含めたすべての混在コンテンツに対するリソース要求がブロックされます。この選択肢も、`<iframe>`ドキュメントにカスケードしますので、ページには混在コンテンツが一切含まれないことが保証されます。

次のディレクティブを指定して、`Content-Security-Policy`ヘッダーを送信すれば、ページは自らこの動作にオプトインできるようになります。

```markup
Content-Security-Policy: block-all-mixed-content
```

もしくは、`<meta>`要素を使って、同じディレクティブをドキュメントの`<head>`セクションに埋め込みます。

```html
<meta http-equiv="Content-Security-Policy" content="block-all-mixed-content">
```

{% Aside %} `upgrade-insecure-requests`と`block-all-mixed-content`の両方を設定している場合は、まず最初に`upgrade-insecure-requests`が評価、使用されます。ブラウザは要求をブロックしません。したがい、どちらか一方だけを使用しましょう。{% endAside %}
