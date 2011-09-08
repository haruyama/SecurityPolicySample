Javascriptの手順
====================================


AjaxでのJSON
-----------------

AjaxでのJSONは通常はクロスドメインで読み取られないが, 環境によってはクロスドメインで読み取られる可能性がある. また, JSONをHTMLとブラウザに解釈させ, XSSを行なう攻撃がある.

* `PHPのイタい入門書を読んでAjaxのXSSについて検討した(3)～JSON等の想定外読み出しによる攻撃～ - ockeghem(徳丸浩)の日記 <http://d.hatena.ne.jp/ockeghem/20110907/p1>`_

AjaxでJSONでデータを提供する場合は, 以下の対策を行なわなければならない(上記記事より).

* リクエストの X-Requested-With ヘッダの値が 「XMLHttpRequest」かチェックする.
* レスポンスの Content-Type ヘッダを, 「application/json; charset=UTF-8」と指定する.

また, 以下の対策を行なうことが望ましい.

* JSON生成ライブラリで設定できる最大限のエスケープの実施する


以前の版では GET での JSON の取得を制限し POST でのみ取得できるようにするよう記述していたが, この対策には機密情報の漏洩は対策できるがXSSの防止には問題があった.

* `Twitter / @ockeghem: @hasegawayosuke 罠サイトに（なんなら ... <https://twitter.com/#!/ockeghem/statuses/111587725739704320>`_

参考:

* `ここが危ない！Web2.0のセキュリティ：第4回　Flash，JSONでのクロスドメインアクセス｜gihyo.jp … 技術評論社 <http://gihyo.jp/dev/serial/01/web20sec/0004?page=2>`_
* `［さらに気になる］JSONの守り方 − ＠IT <http://www.atmarkit.co.jp/fcoding/articles/webapp/05/webapp05a.html>`_

JSONP
----------

JSONPはクロスドメインでのアクセスが可能なので, JSONPで提供するデータに機密情報を含めてはならない.
