Ruby利用時の手順
==================================

情報の暗号化
----------------------------

(ECナビ固有の情報のため削除)


パスワードのハッシュ化
--------------------------------------

以下のRailsプラグインは, 適切なパスワードのハッシュ化手法を実装している. ただし, ハッシュ化回数がデフォルトで10回で少ないので

* REST_AUTH_DIGEST_STRETCHES

を変更する必要がある.

`Restful-authentication | AgileWebDevelopment <http://agilewebdevelopment.com/plugins/restful_authentication>`_
