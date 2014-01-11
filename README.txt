===========================================================
Line6　X3pro の ASIO コントロール・パネルを操作する UWSC クラス
===========================================================


## 必要なツール(要インストール)

 * UWSC http://www.uwsc.info/


## 配布物内容

 * L6ASIO.uws
 
   Line 6 Driver コントロール・パネルを操作するクラス
   他のスクリプトから Call して読み込むので、ファイルは同じ場所に纏めておいてください。
   
 * L6OpenControlPanel.uws
 
   コントロール・パネルを開くだけのデモ。
   スクリプトを書き始める時のテンプレートとしての利用を想定。
   
 * L6ToggleMonitoring.uws
 
   ダイレクト・モニターリングが 0 の時 100 に、100の時 0 に切り替える。
   
 * L6SelectPreset.uws
 
   プリセット設定を選択して反映させるデモ。
   ユーザ定義の設定を簡単に追加記述出来ます。

 * L6Capture.uws

   設定画面の画面キャプチャを取るサンプル
   複数のタブを順に表示し、それぞれ画像に保存します。
   画像サイズ等は要調整。
   
 
 * README.txt このファイル

 * DEV.txt 他のベンダーのASIOドライバー対応モジュールを書くときのためのメモ
 


## スクリプトの書き方（詳細）

 1.  メモ帳等のテキスト・エディタで新規ファイルを作成。
 
 2.  Call 命令で "L6ASIO" ファイルを呼び出し。
     
 3.  クリップボード内に "L6ASIOControlPanel." という文字列をコピー
     USWC専用のエディタであれば入力補完してくれるだろうけど、
     頻繁に入力することになるので、コピーしておきます。
     エディタでの貼り付けキーボード・ショートカット(大抵は Ctrl+V) 
     をあわせて覚えておくと便利。

 4.  If L6ASIOControlPanel.Open() Then で書き始める。
     コントロール・パネルを開くのに成功すると、If 文の中が実行されます。
     
     この時、オーディオ・インターフェースを接続して、電源を入れておかないと、
     設定項目のない空のコントロール・パネルが開くので、
     Open()に成功しても、他の設定等の操作は失敗します。
     
     ※ !!重要な注意事項!!
     録音や再生中にコントロール・パネルを呼び出す動作は未テスト。
     多分、一時的にオーディオ止まるくらいだと思うけど、
     クラッシュのリスク有り、避けた方が無難。
     
     NINJAM中に呼び出す場合は、事前にReaperの設定画面を開き
     一旦、オーディオを止めてからASIOの設定変更行うと安全。
     

 5.  スクリプトを書く
 
     1. コントロールパネルを開く Open()
     2. 設定変更 Set項目名(値)
     3. 保存＆ダイアログを閉じる Ok()
 
     といった流れになると思います。
 

 6.  ファイルを保存して、USWCから読み込み。（ファイルをドラッグ＆ドロップ）
     USWC側でランチ・メニューに登録すると、以後の呼び出しが簡単になります。
     

## L6DriverControlPanel クラスで提供するメソッド
 
   * コントロール・パネルを開く Open()
 
   * 情報の所得
   
     * GetCurrentDriver()
       
     * GetClientDefaultBufferSize()
       
     * GetClientDefaultBitsDepth()
     
     * GetAudioSignalRouting()
       ※ x3proにはない為、未テスト
       
     スライダーの項は、引数指定で最小値(SLD_MIN)・最大値(SLD_MAX)を所得出来ます。
      ※ @see L6toggleMonitoring.uws
     
     * GetDirectMonitorLevel()
       
     * GetUSBStreamingBufferSize()
       
   * 情報の設定
   
     * SelectDriver(driverName)
       
     * SetClientDefaultBufferSize(size)
       
     * SetClientDefaultBitsDepth(depth)
       数値で指定する。" bits"は不要。
         
     * SetUSBStreamingBufferSize(size)
       1..6 までの 6段階
         
     * SetDirectMonitorLevel(level)
       0..100 まで、10単位で指定
       
     * SetAudioSignalRouting(item)
       ※ x3proにはない為、未テスト
       
     * SetParams(...)
       各パラメータを一括して設定する。
       引数は、後述の L6ASIO_SetParams 関数を参照。

   * ボタンをクリック
     * Ok      設定を反映して閉じる
     * Cancel  設定変更を破棄して閉じる
     * Apply   設定の反映のみ
       
   * TABの変更
     * SetTab(label)

  長い名前ばかりなので、入力補完の効かないエディタでは、
  単語をダブルクリックでメソッド名を選択して、コピペを推奨。


## スクリプトの書き方(簡易版)


  各パラメータを簡単に設定する便利関数を提供します。


  Call "L6ASIO"
  L6ASIO_SetParams(False, -1, -1, -1, -1, 0)
  

  設定例では、ダイレクト・モニターリングの音量を0に設定。
  
  6つの引数は、順に・・・

    * ドライバー名
      複数のLine6ドライバがインストールされている場合に指定。
      一つしかない場合は、Falseで良い。
    * (ASIOクライアント標準の)バッファ・サイズ
    * (ASIOクライアント標準の)ビット・レート
    * USBの通信バッファ・サイズ
    * Audio Signal のルーティング (※ X3Proでは項目確認できず)
    * ダイレクト・モニタリングの音量

  ※ ASIOクライアント標準の・・・というのは、設定の参考値で、
     実際には、ホスト側の設定値が用いられる場合もある。
  
  ※ ドライバ名はFalse値、その他の値は -1 を指定すると、
     その項目の設定はスキップされる。


## プリセットを切り替えるランチャー

  サンプルの L6SelectPreset.uws をカスタマイズして使います。

  記述は、上記の L6ASIO_SetParams で設定した項目を、
  カンマ区切りの文字列で記述します。

  Preset["NINJAM"] = "0,256,16,-1,-1,100"
  Preset["Ohm Studio"] = "0,-1,-1,-1,-1,0"



__EOF__
