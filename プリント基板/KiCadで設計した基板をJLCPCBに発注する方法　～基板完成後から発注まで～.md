# KiCadで設計した基板をJLCPCBに発注する方法　～基板完成後から発注まで～

本記事ではKiCadで設計した基板をJLCPCBに発注する方法を解説します．

前提条件：KiCadをインストールしている＆完成した基板データを持っている

## １．前準備（プラグインインストール．初回のみ）
KiCadにJLCPCB発注用のプラグインをインストールします．KiCadを起動し，上のツールバーから"Tools" -> "Plugin and Content Manager"の順にクリックします（図１）．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig1.png" width="50%">
</p>

<!--![](../assets/kicad-jlcpcb-tools/fig1.png)-->
<p align="center">
  図1　Plugin and Content Managerの起動
</p>

<br>

"Plugin and Content Manager"が起動するので検索窓に「jlc」と入力（図２）．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig2.png" width="70%">
</p>
<p align="center">
  図2　Fabrication Toolkitのインストール（１）
</p>

<br>

入力したら「Fabrication Toolkit」というプラグインが出てくるので"Install"をクリックします．クリックしたらウィンドウの右下"Apply Pending Changes"をクリックして変更を適用します．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig3.png" width="70%">
</p>
<p align="center">
  図3　Fabrication Toolkitのインストール（２）
</p>

<br>

これで"Fabrication Toolkit"のインストールが完了しました．

## ２．発注用ファイル（ガーバーデータ）の作成
ここでは完成した基板データを発注用ファイル（ガーバーデータ）に変換する方法を扱います．例としてMain 1層目の基板データを使います．

まず完成した基板データ（ここでは"PCB_Main1.kicad_pcb"）をKiCadで開きます（図4）．
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig4.png" width="70%">
</p>
<p align="center">
  図4　基板データ内のフォルダ構造
</p>
<br>

次にKiCadのツールバーの一番右にある"Fabrication Toolkit"をクリックします（図5）．
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig5.png" width="50%">
</p>
<p align="center">
  図5　Fabrication Toolkitの起動
</p>
<br>

起動したら図6のような画面が起動します．
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig6.png" width="50%">
</p>
<p align="center">
  図6　Fabrication Toolkitによるガーバーデータの生成（１）
</p>
<br>

起動したら"Generate"をクリック．いろいろなオプションがありますが，基本的には初期設定のままで大丈夫です．クリックしたら基板データを保存しているフォルダ内に"production"というフォルダーが生成され，その中にガーバーデータ（ここでは"PCB_Main1.zip"）が保存されます（図7）．これでガーバーデータの出力が完了しました．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig7.png" width="70%">
</p>
<p align="center">
  図7　Fabrication Toolkitによるガーバーデータの生成（２）
</p>
<br>


## ３．JLCPCBに発注
ガーバーデータが完成したら次はいよいよ発注です．[ここ](https://jlcpcb.com/jp/)をクリックしてJLCPCBにアクセスしてください．アクセスし，ログインすると右上の「発注する」というボタンをクリックします（図8）．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig8.png" width="70%">
</p>
<p align="center">
  図8　ガーバーデータのアップロード（１）
</p>
<br>

クリックしたら「ガーバーファイルを追加」をクリックし，２．発注用ファイル（ガーバーデータ）の作成　で生成したガーバーデータ（ここではPCB_Main1.zip）を選択し，アップロードします（図9）．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig9.png" width="70%">
</p>
<p align="center">
  図9　ガーバーデータのアップロード（２）
</p>
<br>

アップロードし，「ガーバービューアー」をクリックすると図10のように作成した基板の2D/3Dイメージが出ますので，設計通り出力されているか確認してください（図10）．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig10.png" width="70%">
</p>
<p align="center">
  図10　ガーバーデータの確認
</p>
<br>

アップロードできたら，オプションの設定を行います．基本的に数量の設定以外はデフォルト設定のままで大丈夫です．デフォルト設定を図11～13に載せておきます．

<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig11.png" width="70%">
</p>
<p align="center">
  図11　発注オプションの設定（１）
</p>
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig12.png" width="70%">
</p>
<p align="center">
  図12　発注オプションの設定（１）
</p>
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig13.png" width="70%">
</p>
<p align="center">
  図13　発注オプションの設定（１）
</p>
<br>

次に配送方法を選択します（図14）．お好きな方法でどうぞ．またクーポンの選択もここで行いましょう．
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig14.png" width="70%">
</p>
<p align="center">
  図14　配送方法の選択
</p>
<br>

配送会社を選択出来たら，「カートに保存」をクリックします．そしてカートボタンを押し，注文を確認します（図15）．
<p align="center">
  <img src="../assets/kicad-jlcpcb-tools/fig15.png" width="70%">
</p>
<p align="center">
  図15　注文の確認
</p>
<br>

確認できたら画面右側の「安全な決済」をクリックし，支払方法や配送先を入力し，決済を完了させて下さい．これで発注完了です．





