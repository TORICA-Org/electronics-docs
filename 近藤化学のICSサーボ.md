# 近藤化学のICSサーボ

使い方がまとめてあるサイトがあまりないのでまとめます。

## ICS通信
UARTの送受信（TXとRX）を1本の通信線でおこなう、半二重通信です。

## 構成例
一番簡単なのは、[ICS変換基板](https://kondo-robot.com/product/03121)と
[ICS Library for Arduino](https://kondo-robot.com/faq/ics-library-a3)
を利用することです。

近藤化学の解説サイトがありました。   
↓↓↓   
<https://kondo-robot.com/faq/ics_board_-tutorial2-2>

上記サイトに対する補足です。
### ICS変換基板との接続
下記サイトに詳しい説明があります。   
↓↓↓   
<https://kondo-robot.com/faq/ics_board_-tutorial1-2>

Raspberry Pi Picoの場合、`IOREF`は`3V3`に接続します。

### 「■プログラムの書き込みと実行」について
> サンプルプログラムの「KrsServo2」を書き込んで実行してみます。ICS変換基板のスイッチが「実行」になっているか確認してください。
> ※Arduino UNO R4などUARTとUSBのSerialが別の場合、ICS変換基板のスイッチは「実行」のまま切り替える必要はありません。

Raspberry Pi Picoなどの場合でも、ICS変換基板のスイッチは「実行」のまま切り替える必要はありません。

## 注意点
[ICS Library for Arduino](https://kondo-robot.com/faq/ics-library-a3)
の内部ではUART通信が初期化されています。

その際、マイコン初学者にとっては理解し難いオプションが指定されています。
