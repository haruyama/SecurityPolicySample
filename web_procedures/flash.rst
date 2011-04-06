Flashの手順
=================

XSS対策
----------------

ActionScriptから外部サイトに移動するために,

* AS2: getURL("移動先URL")
* AS3: navigateToURL(new URLRequest("移動先URL"))

などとする. このとき, 「移動先URL」に任意のURLの挿入を許すと, XSSが可能となる.
この対策として, すくなくともURLが 「http://」 ないし 「https://」 から始まることを保証しなければならない. ホスト名などをより厳しく制限できるならばそうしたほうがよい.
