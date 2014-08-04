====================================================
ローカルPCでPhantomJSを動かしてみる
====================================================

ローカルPCでのサンプル実行を行った際のメモ書きです。

.. warning:: 
   | Windows環境を前提とした説明になっています。
   | 他のOSを使用している場合は、本ページを参考に自分でアレンジして進めてください。
   
.. contents:: 目次
    :local:

-----------------------------
PhantomJSを取得する
-----------------------------
| 以下の公式ページにバイナリがあります。
| http://phantomjs.org/download.html

-----------------------------
とりあえず入れてみる。
-----------------------------

#. | Windows用のzipをDLして、適当な場所に展開します。
   | 2014/3/20現在の最新バージョンは1.9.7でした。
   | 今回は「C:\phantomjs-1.9.7-windows」に展開しました。
   
#. | ``phantomjs`` コマンドを使いやすいように、Path環境変数にPhantomJSのexeを追加しておきましょう。
   | 今回は「C:\phantomjs-1.9.7-windows\」をPathに追加しました。

#. コマンドプロンプトを立ち上げて、``phantomjs --version`` でバージョン番号が表示されれば、インストール完了です。

    .. image:: ../img/phantomjs/phantomjs_version.jpg
        :width: 600px
        
-----------------------------
サンプルを動かしてみる
-----------------------------


基本コマンドはこれです。

    ::
        
        > phantomjs 実行するjsファイル名
        
    
| インストールディレクトリ直下の「example」にサンプルのjsがあるので、適当に動かしてみましょう。
| 例えば「weather.js」を動かすのであれば、以下の画像のようになります。

    .. image:: ../img/phantomjs/weather_js.jpg
        :width: 600px

| weaher.jsが何をやっているのか知りたければ、実際のソースを見てください。

| 参考にしたページ(http://d.hatena.ne.jp/nullpobug/20130320/1363787735)に倣って、Googleのページキャプチャ画像を取得してみました。

    .. code-block:: javascript

        var page = require('webpage').create();
        var url = 'http://www.google.co.jp/';
        page.open(url, function(status) {
          page.render('google.png');
          phantom.exit();
        });

| 簡単に取れました。
| 素敵です。

-------------------------------------
QUnitのテストケースを動かしてみる
-------------------------------------
| 「example」にある「run-qunit.js」で実行できます。

コマンドはこんな感じになります。

    ::
        
        > phantomjs run-qunit.js 作成したQUnitテストケースのHTMLのパス


自分で作成したテストケースを実行してみました。

    .. image:: ../img/phantomjs/run_qunit_failed.jpg
        :width: 600px

| 結果が貧相ですね。。
| そして、ブラウザでは全てpassしていたのに、1つfailedになってしまいました。。

| 結局のところ、QUnitが吐き出した結果HTMLをパースしてCLIに出力しているだけのようなので、
| このページ(http://d.hatena.ne.jp/daisun/20111014/1318519928)を参考にrun-qunit.jsを拡張してみました。
| が、、テストケース名に日本語を入れている部分が、コマンドプロンプトからの実行だと化けてしまいました。
| (コマンドプロンプトの文字コードをUTF-8にしても改善しませんでした。)

-----------------------------------------
qunit-reporter-junitを入れて動かしてみる
-----------------------------------------
| これ。
| https://github.com/jquery/qunit-reporter-junit

    .. note::

        | 「If you're using Grunt, you should take a look grunt-contrib-qunit.」
        | って書いてありますね。
        | Grunt.js使ったほうが、やっぱり色々と楽そうな気がしてきました。
        
| とりあえずXML形式で出せたので、相変わらずコマンドプロンプトからは文字化けるが、失敗したテストは特定できました。
| これ↓

    .. code-block:: javascript

        asyncTest('画面スクロール　指定された位置へ指定された速度でのスクロールが成される', function () {
            SvcMng.Test = new SvcMng.BaseController();
            SvcMng.Test.animateScroll(10, 'fast');
            setTimeout(function () {
                // TODO durationのAssertができていない
                equal(10, $('html, body').scrollTop(), '指定した位置へスクロールが発生する');
                start();
            }, 1000);
        });

| スクロール幅を大きめ(100)に変更したところ、うまく動作するようになりました。
| 具体的な原因までは分からず。。

    .. code-block:: javascript

        asyncTest('画面スクロール 指定された位置へ指定された速度でのスクロールが成される', function () {
            SvcMng.Test = new SvcMng.BaseController();
            SvcMng.Test.animateScroll(100, 'fast');
            setTimeout(function () {
                // TODO durationのAssertができていない
                equal($(window).scrollTop(), 100, '指定した位置へスクロールが発生する');
                start();
            }, 1000);
        });

-----------------------------------------
console.logで日本語が文字化ける
-----------------------------------------
| 前述のQUnitの実行結果(JUnitのXML形式)の日本語が文字化ける件、以下のように表示されてしまいます。

    .. image:: ../img/phantomjs/run_qunit_encoding.jpg
        :width: 600px

| Phantom.js側のデフォルトはUTF-8だと書いてあるので、問題無さそうだが。。。
|

::

    chcp 65001
    
でコマンドプロンプトをUTF-8にできるが、これで試しても改善しない。。。

| 結局これが怪しいと思う。
| http://fine.ap.teacup.com/hepo/30.html
|
| 現状だとレジストリ変えるのもめんどくさいので、必要に応じてrun-qunit.jsに

::

    phantom.outputEncoding = 'System'


| を追加してしのいでます。。
| 個人PCに依存してしまう部分なので、「Windowsで実行する時はブラウザで」というルール・運用にするかも。

------------------------------------------
run-qunit.jsでテストケースの実行が止まる
------------------------------------------
自作のテストケースを実行させていたら、

::
    
    > 'waitFor()' timeout

| と出て終わってしまう、、、というケースが発生。
| 何が原因かさっぱり分からず。

| 調査も面倒なので、QUnitの公式ページでも紹介されていた、`QUnit PhantomJS Runner <https://github.com/jonkemp/qunit-phantomjs-runner>`_ を使うことにしました。
| こちらは特に問題なく動いたので。
