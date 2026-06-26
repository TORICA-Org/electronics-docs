# 近藤化学のICSサーボ

鳥科では、これまでの安定した稼働の実績から、
近藤化学の高トルクサーボモータ
「[KRS-4034HV ICS](https://kondo-robot.com/product/krs-4034hv-ics)」
を採用しています。

解説してくれているサイトをまとめました。

## 仕様書
[KRS-403xHVseries](https://kondo-robot.com/w/wp-content/uploads/KRs-403xManual.pdf)

## コマンドリファレンス
[ICS3.5/3.6 ソフトウェアマニュアルコマンドリファレンス](https://kondo-robot.com/w/wp-content/uploads/ICS3.5_3.6_SoftwareManual_2_9.pdf)

## ICS通信
UARTの送受信（TXとRX）を1本の通信線でおこなう、半二重通信です。

## 構成例
一番簡単なのは、[ICS変換基板](https://kondo-robot.com/product/03121)と
[ICS Library for Arduino](https://kondo-robot.com/faq/ics-library-a3)
を利用することです。

近藤化学の解説サイトがありました。このサイトに従って、サンプルスケッチを書き込んでみると良いでしょう。   
↓↓↓   
<https://kondo-robot.com/faq/ics_board_-tutorial2-2>

以下、上記サイトに対する補足です。
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
（`Serial.begin(115200);`などというコードと同等のものです。見覚えがある人が多いと思います。）

その際、マイコン初学者にとっては理解し難いオプションが指定されています。
```cpp
bool IcsHardSerialClass::begin()
{
  if (icsHardSerial == nullptr)
  {
    return false;
  }

  icsHardSerial->begin(baudRate,SERIAL_8E1); // この行がSerial.begin()に相当します。
  icsHardSerial->setTimeout(timeOut);
  pinMode(enPin, OUTPUT);
  enLow();
  
	
  return true;
}
```

`SERIAL_8E1`は、未指定の場合`SERIAL_8N1`となっています。
最後の`E1`はパリティと呼ばれる、誤り訂正ビットの指定です。   
これについては、各自調べてみてください。

設定できる項目については、下記サイトに詳しい説明があります。   
↓↓↓   
<https://garretlab.web.fc2.com/arduino.cc/docs/language-reference/ja/functions/communication/serial/begin/>
