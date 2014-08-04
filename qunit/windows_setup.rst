====================================================
ローカルPCでQUnitを動かしてみる
====================================================

ローカルPCでのサンプル実行を行った際のメモ書きです。

.. warning:: 
   | Windows環境を前提とした説明になっています。
   | 他のOSを使用している場合は、本ページを参考に自分でアレンジして進めてください。
   
.. contents:: 目次
    :local:

-----------------------------
QUnitを取得する
-----------------------------
| GitHubにいます。
| https://github.com/jquery/qunit

+++++++++++++++++
とりあえずClone
+++++++++++++++++

#. `GitHub for Windows <http://windows.github.com/>`_ を入れておきましょう。
#. ブラウザからQUnitのリポジトリを表示して(GitHubにログインしておいてください)、「Clone in Desktop」を押します。

    .. image:: ../img/qunit/clone_desktop.jpg
        :width: 600px
        
    プログラムの選択を求められますので、そのまま「GitHub」を選択するとCloneが実行されます。
    
    .. image:: ../img/qunit/clone_desktop_application.jpg
        :width: 200px
    
    ローカルリポジトリにCloneされました。
    
    .. image:: ../img/qunit/clone_complete.jpg
        :width: 600px

    | ローカルリポジトリのWindows上でのパスは、初期設定通りであれば
    | ``「C:\Users\ユーザ名\Documents\GitHub\」``
    | になっているはずです。

++++++++++++++++++++++
サンプルを動かしてみる
++++++++++++++++++++++

| Cloneしたリポジトリの「test」ディレクトリにそれっぽいhtmlがいたので、ブラウザで開いてみました。
| 、、、が「test markup」と表示されるだけで、何も起きません。

FireBugを見ていると、Scriptエラーを吐いていました。

    .. image:: ../img/qunit/qunit_not_defined.jpg
        :width: 450px

HTMLで読み込んでいる

    ::

        <script src="../dist/qunit.js"></script>

が無いことが原因のようです。

リポジトリの内容をよく見てみると、トップディレクトリに「Gruntfile.js」があって、こんな設定がされていました。

    .. code-block:: javascript

        grunt.initConfig({
        	pkg: grunt.file.readJSON( "package.json" ),
        	concat: {
        		"src-js": {
        			options: { process: process },
        			src: [
        				"src/intro.js",
        				"src/core.js",
        				"src/test.js",
        				"src/assert.js",
        				"src/equiv.js",
        				"src/dump.js",
        				"src/diff.js",
        				"src/export.js",
        				"src/outro.js"
        			],
        			dest: "dist/qunit.js"
        		},

Gruntでコンパイルしないと使えないみたいです。

-----------------------------
ビルドジョブを流してみる
-----------------------------
++++++++++++++++++++++++++++
とりあえずGrunt入れてみる
++++++++++++++++++++++++++++
| このあたりを参考にやってみます。
| http://geckotang.tumblr.com/post/29117250620/windows7-node-js-grunt-js
| http://qiita.com/kzhrk/items/85ae4c363ae553c45fc5


#. Node入れる

    | `Node公式ページ <http://nodejs.org/>`_ のINSTALLボタンを押すだけです。
    | OSに対応したインストーラがDLされるので、これを実行します。
    
    .. image:: ../img/qunit/node_install.jpg
        :width: 450px
    
    | インストーラに色々聞かれますが、全てデフォルトで問題ありません。
    | インストーラによって、Node本体とそのパッケージ管理ツールであるnpmがインストールされます。
    
    | 全て完了したら、コマンドプロンプトから以下のようにバージョンを確認してみましょう。
    | 表示されたらインストール完了です。
    

    .. image:: ../img/qunit/node_install_complete.jpg
        :width: 650px

#. Grunt入れる

    コマンドプロンプトから
    
    :: 
    
        npm install -g grunt
    
    | を実行します。
    | 完了すると、以下のように表示されます。

    .. image:: ../img/qunit/grunt_install_complete.jpg
        :width: 650px
    
    .. note:: 
        | と思ったら、コマンドラインからの実行ができなかったため、``npm install -g grunt-cli`` も実行しました。
        | もしかしたら、こっちだけやれば依存関係解決してくれて全てインストールしてくれるかも。
        

    これで、コマンドプロンプトから ``grunt`` コマンドが実行できるようになりました。

++++++++++++++++++++++++++++
QUnitビルドしてみる
++++++++++++++++++++++++++++

#. Qunitのローカルリポジトリ ``「C:\Users\ユーザ名\Documents\GitHub\qunit」`` に移動します。
#. | package.jsonが用意されているので、``npm install`` コマンドを実行して、必要なモジュールを取り込みます。
   |
   | あれ、Phantom.jsも入ったかも。

    .. image:: ../img/qunit/npm_install_qunit1.jpg
        :width: 650px

    .. note:: 
        | どうも`このモジュール <https://www.npmjs.org/package/grunt-lib-phantomjs-istanbul>`_ を使っているらしく、こいつに引っ張られて以下に入りました。
        | C:\Users\n-enokido\Documents\GitHub\qunit\node_modules\grunt-qunit-istanbul\node_modules\grunt-lib-phantomjs-istanbul\node_modules\phantomjs\lib\phantom
        | 自分で使いたい場合は、別途インストールした方がよさそうです。
        
   | そして何やらエラーも出る。。
   
    .. image:: ../img/qunit/npm_install_qunit2.jpg
        :width: 650px

   | GitHubのリポジトリ内で実行したからマズいのか？
   | 状況としてはcommitpleaseというモジュールがインストール失敗しただけみたいなので、先に進めます。
   

#. Gruntfile.jsが用意されているので、``grunt`` コマンドを実行するとビルドされます。

    .. image:: ../img/qunit/qunit_grunt.jpg
        :width: 650px

#. ビルドが完了したら、testディレクトリの配下にいる適当なHTMLを開くと、QUnitが実行されるのを確認できます。

-----------------------------
と、ここで気付いたこと
-----------------------------

GitHubじゃない `QUnit公式ページ <http://qunitjs.com/>`_ からなら、ビルド済みのファイルDLできましたね。。。
