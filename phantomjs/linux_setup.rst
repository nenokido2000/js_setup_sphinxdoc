====================================================
CentOSでPhantomJSを動かしてみる
====================================================

CentOSでのサンプル実行を行った際のメモ書きです。

.. warning:: 
   | CentOS環境を前提とした説明になっています。
   | 他のOSを使用している場合は、本ページを参考に自分でアレンジして進めてください。
   
.. contents:: 目次
    :local:

-----------------------------
マシン
-----------------------------
| サービス管理のビルドサーバを使用します。

``uname -a`` の情報はこれ。

::

    Linux dev-mg-f01 2.6.18-308.el5 #1 SMP Tue Feb 21 20:06:06 EST 2012 x86_64 x86_64 x86_64 GNU/Linux


``/etc/redhat-release`` の内容はこれ。

::

    CentOS release 5.8 (Final)

-----------------------------
PhantomJSを取得する
-----------------------------
| 以下の公式ページにバイナリがあります。
| http://phantomjs.org/download.html

::

    Linux

    For 64-bit system, download phantomjs-1.9.7-linux-x86_64.tar.bz2 (12.6 MB).
    
| を落とせば大丈夫です。
| This package is built on CentOS 5.8.って書いてあるし。

-----------------------------
とりあえず入れてみる。
-----------------------------
| DLしたファイルを、CentOS上の適当な場所に解凍します。
| 今回は ``/opt/`` 以下に解凍しました。
| 特にディレクトリ名も変更しないのであれば、以下のディレクトリにバイナリが入ります。

::

    /opt/phantomjs-1.9.7-linux-x86_64/bin

| このディレクトリにbash_profileとかでPATHを通せばOKです。
| 今回は「/etc/profile」に追加しました。

::

    export PHANTOMJS_HOME=/opt/phantomjs-1.9.7-linux-x86_64
    export PATH=$PATH:$PHANTOMJS_HOME/bin

-----------------------------
テストケースを動かしてみる
-----------------------------


基本コマンドはWindowsの場合と一緒です。
Jenkinsに食わせているコマンドの一例を挙げておきます。

    ::
        
        > /opt/phantomjs-1.9.7-linux-x86_64/bin/phantomjs runner.js main.html > ./report/main_report.xml
        

    .. warning::
        
        残念ながら、現時点では
            
            #. QUnit PhantomJS Runnerがconsoleにメッセージを吐かないように、ソースを修正。
            #. その結果、qunit-reporter-junitが吐き出すXMLだけがconsoleに出力される。
            #. consoleに出力されたXMLをリダイレクトでファイルに入れる。
            #. そのファイルをJenkinsにJunitテスト結果として集計させる
        
        | という超原子的な方法でCIジョブを回しています。
        | grunt + 何か(QUnitのリポジトリを参考にするのであれば `istanbul <https://github.com/asciidisco/grunt-qunit-istanbul>`_ )で、実行と結果(カバレッジも)収集を行う方法に変更したいです。


