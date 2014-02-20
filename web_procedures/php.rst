PHP利用時の手順
==================================


公式に推奨されていない機能・関数の利用禁止
-------------------------------------------------

`PHP: PHP 5.3.x で推奨されない機能 - Manual <http://jp2.php.net/manual/ja/migration53.deprecated.php>`_ や `PHP: PHP 5.4.x で推奨されなくなる機能 - Manual <http://jp.php.net/manual/ja/migration54.deprecated.php>`_ に上げられている機能や関数は利用してはならない.

escapeshellcmd()の利用禁止
-------------------------------------------------

escapeshellcmd()関数は利用してはならない.

* `PHPのescapeshellcmdの危険性 - 徳丸浩の日記 <http://www.tokumaru.org/d/20110101.html#p01>`_
* `escapeshellcmdの危険な実例 - 徳丸浩の日記 <http://www.tokumaru.org/d/20110104.html#p01>`_



PHPコードを評価する関数・機能の利用禁止
-------------------------------------------------
PHPコードを評価する,

* eval()
* preg_replace()の'e'修飾子

は, 利用してはならない. ただし, 利用が社内に限られているソフトウェアについては, eval()は利用してもよい. preg_replaceの'e'修飾子については preg_replace_callback()を利用すること.


クロスサイトスクリプティング(XSS)対策
-------------------------------------------------

通常
#########

htmlに文字列を出力する前に, htmlspecialchars()関数を第3引数にエンコーディングを指定してエスケープを行なう. エスケープは, コントーラがテンプレートに引数を渡す箇所かテンプレートで行なわなければならない.

以下のようなヘルパー関数を定義して用いることを推奨する.::

   function h($s)
   {
       return htmlspecialchars($s, ENT_QUOTES, 'UTF-8');
   }

URL
#########

URLを出力する場合には, 文字列が「http://」 ないし 「https://」, 「//」 で開始されることをチェックする.::

    function checkUrl($s)
    {
        return preg_match('!\A(?:https?:)?//[-_.\!~*\'();/?:@&=+\$,%#a-zA-Z0-9]*\z!u', $s) === 1;
    }



エンコーディングのチェック
########################################

入力パラメータを文字列として扱う場合には, mb_check_encoding()関数でエンコーディングが期待通りかどうかチェックしなければならない. もしくは, mb_convert_encoding()で, 強制的にエンコーディングの変換を行なわなければならない.

注: 現在のmb_check_encoding()は, 5byte, 6byteのUTF-8を有効と判断する. `最近の mbstring 動向について(PHP 5.4～) - t_komuraの日記 <http://d.hatena.ne.jp/t_komura/20110812/1313125578>`_ によると PHP 5.4 では無効と判断するようになる.::

  function checkUtf8($s)
  {
      return mb_check_encoding($s, 'UTF-8');
  }



SQLインジェクション対策
-------------------------------------------------

PDO利用時の設定
########################################

`gist: 459499 - GitHub <http://gist.github.com/459499>`_
のどちらかで, PDOオプジェクトを生成しなければならない. 以下にも転載する::

   //PDOからよばれるreal_escape_stringで文字コードを考慮させたい場合は
   $dbh = new PDO('mysql:host=localhost;dbname=sandbox;charset=cp932', 'sandbox', 'sandbox', array(
       PDO::MYSQL_ATTR_READ_DEFAULT_FILE => '/etc/mysql/my.cnf',
           PDO::MYSQL_ATTR_READ_DEFAULT_GROUP => 'client',
       ));

   //もしくはserver-side-prepareを使う場合(文字コード気にしなくておｋ)
   $dbh = new PDO('mysql:host=localhost;dbname=sandbox;charset=cp932', 'sandbox', 'sandbox', array(
       PDO::ATTR_EMULATE_PREPARES => false,
   ));


以上はcp932の場合であることに注意. mysqlの場合dsnでのcharset指定はPHP 5.3.6以降でのみサポートされている. また, 前者の場合は /etc/mysql/my.cnf を適切に設定しなければならない.


PHP 5.3.6以降では, dsnでcharset指定すればよい. utf8の場合は以下のようにする.::

   $dbh = new PDO('mysql:host=localhost;dbname=sandbox;charset=utf8', 'sandbox', 'sandbox', array(
       PDO::ATTR_EMULATE_PREPARES => false,
   ));

`PHP5.3.6からPDOの文字エンコーディング指定が可能となったがWindows版では不具合（脆弱性）あり - 徳丸浩の日記 <http://www.tokumaru.org/d/20110322.html#p01>`_ によると, Windows版のPHPでは十分に対応していないとのことなので注意すること. また, サーバ側のプリペアドステートメントを利用しない場合は, 予防として MySQL の設定ファイルを読み込ませる方式を取ること.

補足: 文字エンコーディングを利用した SQLインジェクションが可能となるのは cp932 などの文字エンコーディングの場合は, euc-jpやutf8の場合は問題とはならない.しかし, 文字エンコーディングを正しく設定しておいたほうが問題が起きる可能性を少なくできる.

セッション設定
-------------------------------------------------

PHP 5.3以降で, PHPのセッション機構を利用する場合以下のように設定することを推奨する.::

  session.use_cookies = 1
  session.use_only_cookies = 1
  session.cookie_httponly = 1
  session.entropy_file = /dev/urandom
  session.entropy_length = 32
  session.hash_function = sha256
  session.hash_bits_per_character = 5

session.hash_bits_per_character は 6 でもよい.

PHP 5.2 では,  以下を推奨する.::

  session.use_cookies = 1
  session.use_only_cookies = 1
  session.cookie_httponly = 1
  session.entropy_file = /dev/urandom
  session.entropy_length = 20
  session.hash_function = 1
  session.hash_bits_per_character = 5

以下の項目は要件に合わせた設定を行なうこと.

* session.cookie_lifetime

クロスサイトリクエストフォージェリ(CSRF)対策
-----------------------------------------------------------------

HTMLテンプレートにて次のように「type="hidden"」でvalue属性値がセッションIDのハッシュ値であるinput要素を生成し, 処理側でその値がセッションIDのハッシュ値と等しいことを検証する.::

  <input type="hidden" name="csrf_token" value="<?php echo h(hash('sha256', session_id())); ?>">

セッション固定攻撃対策
-------------------------------------

ログイン前からセッションを維持する場合には, session_regenerate_id()を利用してセッションIDを変更する. このとき session_regenerate_id(true) として古いセッションIDを破棄すること.

ファイルを開く際の注意点
---------------------------------------

リモートファイル読み込み攻撃対策
##################################################################

allow_url_fopen が有効になっている場合に fopen()やfile()などの関数でファイル名を任意に外部から設定できると,  任意のファイルを読み込ませることができる. このとき, ファイルを開く場合には, 必ず特定のディレクトリ以下のファイルのみを開くようにしなければならない.

ディレクトリトラバーサル対策
#################################################

以下の2つの対策のうちどちらかを実施しなければならない.

* 開けるファイル名のリストを作り, リストにあるファイル名のみを許可する.
* ファイル名に「..」が含まれるかチェックし含まれていないファイル名のみを許可する.

ヌルバイト攻撃対策
#################################################

以下の2つの対策のうちどちらかを実施しなければならない.

* 開けるファイル名のリストを作り, リストにあるファイル名のみを許可する.
* ファイル名にヌルバイトが含まれるかチェックし含まれていないファイル名のみを許可する.


まとめ
#################################################

* 特定のディレクトリ以下のファイルのみを開くよう設定する
* ファイル名が特定できる場合はそのリストを作成し, リストにあるファイル名のみを許可する
* リストが作成できない場合は, 「..」やヌルバイトを含むファイル名をエラーとする

より柔軟なファイル名の扱いをしたい場合には, 情報セキュリティ委員会にレビューを依頼しなければならない.

OSコマンドインジェクション攻撃対策
-----------------------------------------------

外部に公開するサービスについては, 可能な限りOSのコマンドを利用しないようにしなければならない.

利用する場合は, 可能な限りコマンド・コマンド引数を限定しなければならない.

どうしても任意の引数・コマンドを利用しなければならない場合は, escapeshellarg() を利用してエスケープを行なわなければならない. また情報セキュリティ委員会にレビューを依頼しなければならない.

メールの第三者中継攻撃対策
-----------------------------------------

* mail()やmb_send_mail()の第1引数は可能ならば固定とすること. 固定にできない場合は, メールアドレスのフォーマットチェックを行ない, 1通のみしか送信できないようにしなければならない.
* mail()やmb_send_mail()の第4引数を指定する場合, 任意のメールヘッダを挿入されないように改行文字のチェックを行なわなければならない. 参考: `Email ヘッダ・インジェクション(Email header injection):PHP と Web アプリケーションのセキュリティについてのメモ <http://www.asahi-net.or.jp/~wv7y-kmr/memo/php_security.html#PHP_Email_header_injection>`_

情報の暗号化
--------------------------

mcrypt拡張を利用すること.

(ECナビ固有の情報のため削除)

パスワードのハッシュ化
--------------------------------

(ECナビ固有の情報のため削除)

HTTPヘッダ・インジェクション
-----------------------------------------------

PHP 5.4.0 より前の PHPのheader()関数は, ラインフィード(0x0A)はチェックし複数のヘッダが送信できないが, キャリッジリターン(0x0D)はチェックしない. キャリッジリターンにより Internet Explorer, Google Chrome, Operaでは, HTTPヘッダ・インジェクションが可能である. このため安全なリダイレクトのためには, キャリッジリターンが含まれていないことをチェックしたあとで header()関数を利用しなければならない.::


    function redirect($url, $code = 302)
    {
        if (strpos($url, "\x0d") === false) {
            header('Location: ' . $url, $code);
        }
        error_log('redirect: ' . $url);
        exit(1);
    }

以上の例では キャリッジリターンの有無のみチェックしている.  `体系的に学ぶ 安全なWebアプリケーションの作り方 <http://www.hash-c.co.jp/wasbook/>`_ では, URLの文字種もチェックしている.

また, URLのパラメータなどを動的に生成する場合は, パラメータ値にURLエンコードを行なえばキャリッジリターンは変換されるので, HTTPヘッダ・インジェクションは起こらない. パラメータの場合は urlencode(), パスなどの場合は rawurlencode()を用いること.

PHP 5.4.0 以降では, header() をそのまま利用しても問題ない. 

なお, 広告サイトなどで任意のURLへのリダイクレトを行ないたい場合を除いて, 任意のURLに対するリダイレクトを行なうべきではない(オープンリダイレクタ脆弱性).  リダイレクト先を自サイトドメインなどに限定するコードを別途実装し利用しなければならない.
