====================================================
QUnitでハマったところ
====================================================

ハマりどころについて書いておきます。

.. contents:: 目次
    :local:

-----------------------------
qunit-fixture
-----------------------------
+++++++++++++++++
事象
+++++++++++++++++

こんなjsのテストを書こうとして

    .. code-block:: javascript
        
        $(function () {
            SvcMng.deviceBasic = SvcMng.deviceBasic || {};
            SvcMng.deviceBasic.completeController = new SvcMng.CompleteController(
                {
                    top: '.'
                },
                $('#research-param')
            );

            $('#back-btn').click(function () {
                SvcMng.deviceBasic.completeController.research();
            });
        });

こんなテストを書きました。

    .. code-block:: javascript
    
        module('端末基本情報完了画面処理のテスト', {
          setup: function() {
            sinon.config.useFakeTimers = false;
          }
        });

        test('端末基本情報完了画面コントローラオブジェクトの初期化確認', function() {
          equal('.' , SvcMng.deviceBasic.completeController.urls.top, 'TOP画面URLが正しく設定されている');
          deepEqual($('#research-param')[0] , SvcMng.deviceBasic.completeController.$researchParam[0], '再検索パラメータ要素が正しく設定されている');
        });

        test('端末基本情報完了画面の要素に対するイベントバインドの確認', function() {
          var mock = sinon.mock(SvcMng.deviceBasic.completeController);
          mock.expects('research').once();
          $('#back-btn').trigger('click');
          ok(mock.verify(), '戻るボタンのclickイベントに再検索処理の関数がバインドされている');
        });

htmlはこれで

    ::
    
        <!doctype html>
        <html lang="en">
        <head>
            <meta charset="UTF-8">
            <title>complete Test</title>
            <link rel="stylesheet" type="text/css" href="./css/qunit.css">
            <script type="text/javascript" src="./js/qunit.js"></script>
            <script type="text/javascript" src="./js/sinon-1.9.0.js"></script>
            <script type="text/javascript" src="./js/sinon-qunit-1.0.0.js"></script>
            <script type="text/javascript" src="./js/jquery-1.10.1.min.js"></script>
            <script type="text/javascript" src="./js/main.js"></script>
            <script type="text/javascript" src="./js/complete.js"></script>
            <script type="text/javascript" src="./js/complete-test.js"></script>
        </head>
        <body>
            <div id="qunit"></div>
            <div id="qunit-fixture">
                <div id="research-param">research-param</div>
                <button id="back-btn">back-button</button>
            </div>
        </body>
        </html>

| で、実行すると以下のようにエラーになりました。
| ``: Expected research([...]) once (never called)`` ですって？？？


    .. image:: ../img/qunit/fixture_error.jpg
        :width: 600px

+++++++++++++++++
原因と解決
+++++++++++++++++

| 「qunit-fixture」以下に置いた要素は、テスト毎にクリア→再生成されるそうです。
| 今回はdocument.readyのタイミングで「#back-btn」にイベントバインドしているのですが、
| 1つ目のテストが終わった時点でその要素が
| 削除 → 再生成
| されているため、イベントバインドも外れて当然という結果でした。
| 
| 参考ページ(http://reiare.net/blog/2011/10/27/qunit) にも書いてありますが、戻されて不都合なものはここに置くべきではないんです。
|
| ということで、以下のように要素配置を変更して解決しました。

    ::
    
        <body>
            <div id="qunit"></div>
            <div id="qunit-fixture">
                <div id="research-param">research-param</div>
            </div>
            <button id="back-btn">back-button</button>
        </body>


---------------------------------------
asyncTestでstartしてもテストが動かない
---------------------------------------
+++++++++++++++++
事象
+++++++++++++++++

こんな感じでasyncTestを書いてみたとき、

    .. code-block:: javascript

        module('complete Test', {
          setup: function() {
          }
        });

        asyncTest('オブジェクトにセットされた値のテスト', function() {
          equal('.' , SvcMng.deviceBasic.completeController.urls.top, 'TOP画面URLが正しく設定されている');
          deepEqual($('#research-param')[0] , SvcMng.deviceBasic.completeController.$researchParam[0], '再検索パラメータ要素が正しく設定されている');
          setTimeout(function () {
            ok(true, 'asyncTest OK')
            start();
          }, 100);
        });

        test('イベントバインドのテスト', function() {
          mock = sinon.mock(SvcMng.deviceBasic.completeController);
          mock.expects('research').exactly(1);
          $('#back-btn').trigger('click');
          ok(mock.verify(), '戻るボタンのclickイベントに再検索関数がバインドされている');
        });

| startが実行されたにも関わらず、後続のテストが動きません。
| デバッグしてみると分かるのですが、setTimeoutにセットした関数が実行されていないことが原因のようです。

    .. image:: ../img/qunit/asynctest_not_run.jpg
        :width: 600px

+++++++++++++++++
原因と解決
+++++++++++++++++

| `Sinon.js <http://sinonjs.org/>`_ を使っているのですが、一緒に噛ませている「qunit-sinon」がSinonの偽装Timer機能を
| 使うことをデフォルトで設定してしまうらしく、これによって意図したsetTimeoutが発火しなくなっていました。
|
| 参考ページ(https://github.com/cjohansen/sinon-qunit/issues/3)に書いてありますが、
| ``sinon.config.useFakeTimers = false;`` として、偽装Timer機能を明示的にオフにする必要があります。
|
| 今回はsetupに入れて解決しました。

    .. code-block:: javascript

        module('端末基本情報完了画面処理のテスト', {
          setup: function() {
            sinon.config.useFakeTimers = false;
          }
        });

---------------------------------------
Sinon.jsのfake serverで404応答
---------------------------------------
+++++++++++++++++
事象
+++++++++++++++++

こんな感じでfake serverを使ったテストを書いてみたとき、

    .. code-block:: javascript

        server.respondWith('GET', '/search', [200, {'Content-Type': 'text/html', 'Content-Length': 2 }, 'OK']);
        SvcMng.Test = new SvcMng.BaseController();
        var spy = sinon.spy();
        SvcMng.Test.ajax('GET', 'html', null, '/search', spy);
        server.respond();

| fake serverへの疑似リクエストは飛んでいるが、404が応答されていました。
| リクエストオブジェクト(FakeXMLHttpRequest)をキャプチャしてみると、どうも指定したURL(/search)がマッチングしていないようです。

    .. image:: ../img/qunit/fakeserver_404.jpg
        :width: 650px
        
+++++++++++++++++
原因と解決
+++++++++++++++++

テスト対象のSvcMng.Test.ajaxは、以下のような実装をしていました。

    .. code-block:: javascript

        ajax: function (type, dataType, params, targetUrl, callback, errorCallback, redirectUrl) {
            var self = this;
            $.ajax({
                type: type,
                dataType: dataType,
                data: params,
                cache: false,
                url: targetUrl,
                success: function (data, text, xhr) {
                    var ajaxRedirect = xhr.getResponseHeader("AjaxRedirect");
                    if (ajaxRedirect) {
                        self.redirect(redirectUrl);
                    } else {
                        callback(data, text, xhr);
                    }
                },
                error: errorCallback
            });
        },

    | cache: falseなので、JQuery側でキャッシュクリアパラメータをURLに付与します。
    | 先程のURLだと「/search?_1234567789...」という形式でパラメータが付与されており、これによってURLがマッチングしていませんでした。
    |
    | fake serverのURLには正規表現が使用できるとのことだったので、今回は正規表現でマッチングをかけるようにして対応しました。

    .. code-block:: javascript

        // $.ajaxをcache:falseでcallするためURLにキャッシュクリアパラメータを含めてマッチングさせる
        server.respondWith('GET', /\/search\?_\=[0-9]+/, [200, {'Content-Type': 'text/html', 'Content-Length': 2 }, 'OK']);
        SvcMng.Test = new SvcMng.BaseController();
        var spy = sinon.spy();
        SvcMng.Test.ajax('GET', 'html', null, '/search', spy);
        server.respond();
