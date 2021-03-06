MultiProcessing
====================

MultiProcessingはRubyにおいてプロセス間同期とプロセス間通信の機能を提供します（することを目指しています）．
各クラスはRubyの標準添付ライブラリthreadで提供されているクラスのような動作をすることを目指しています．

MultiProcessing includes classes for inter-process synchronization and communication. 
The classes can be used like ones in ruby standard library for thread.

現状で使用できるクラスは以下です．
各クラスはMultiProcessingモジュールの下に作成されています．

* Mutex
* ConditionVariable
* Semaphore
* Queue
* ExternalObject

また，IO.named\_pipe が追加されます

いずれのクラスもプロセス間通信にパイプ（IO.pipe）を使用しています．
また，複数のプロセスで1つの同期オブジェクトを共有するためにforkを使用する事を想定しているため，
Unix系OSのみでのみ使用する事ができます．

（Windowsするためには，名前付き共有セマフォなどの同期機能を使用する必要がありそうですね・・・）

Install
----------------

    gem install multiprocessing

Mutex
----------------

標準添付ライブラリthreadのMutexと同様の使い方をします．

    m = MultiProcessing::Mutex.new
    fork do
      m.synchronize do
        # some critical work
      end
    end
    m.synchronize do
      # come critical work
    end
    Process.waitall

lockした後unlockする前にforkすると（synchronize中でforkした場合も）そのクリティカルセクションは並列に動作してしまうので注意してください．
（forkしたときに子プロセスでは子スレッドが全て殺されるため，別のスレッドでforkすれば問題ないと思います．）

ConditionVariable
----------------

標準添付ライブラリthreadのConditionVariableと同じ使い方をします．

      m = MultiProcessing::Mutex.new
      cond = MultiProcessing::ConditionVariable.new
      fork do
        m.synchronize do
          cond.wait(m)
          # something to do
        end
      end
      sleep 1
      # on condition changed
      cond.signal
      Process.waitall

Semaphore
----------------

Semaphoreは標準添付ライブラリに含まれていませんが作りました．

    s = MultiProcessing::Semaphore.new 2

コンストラクタにリソース量の初期値を与えてください．
Pでリソースのロック，Vで解放をします．

Pはlock，Vはunlockという名前のエイリアスが用意してあります．

Queue
----------------

Queueも標準threadライブラリのQueueと同様の使い方をします．

    q = MultiProcessing::Quque.new
    fork do
      q.push :nyan
      q.close.join_thread
    end
    p q.pop

プロセス間の通信にはパイプを使っています．Queue#pushをすると，パイプにデータを書き込むバックグラウンドスレッドが起動します（pythonのmultiprocessingを参考にしました）．
プロセスが終了する際，バックグラウンドスレッドがパイプにデータを書き込み終わるまで待たないと，Queueが書き込みMutexをロックしたままになる場合があります．
終了時は

    q.close.join_thread

として，書き込みスレッドの終了を待ってください．close後にキューにデータをpushすると例外が発生します．

オブジェクトのシリアライズにMarshal.dumpを使用しているので，Marshal.dumpできないオブジェクトをpushできません．

その他
----------------

### 関係ありそうなライブラリ

* https://github.com/pmahoney/process_shared

### TODO

* ExternalObject, IO.named\_pipe のSpecを書く
* ドキュメントを書く

ライセンス
----------------

Author: clicube

MIT License

