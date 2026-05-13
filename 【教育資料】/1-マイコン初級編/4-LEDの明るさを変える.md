# 4 - LEDの明るさを変える

analogWrite() は
LEDに流れる平均的な電力を変えている と考えるとわかりやすいです。

`analogWrite()`を使う

## 必要なもの
・　Raspberry Pi Pico
・　ブレッドボード
・　マイコンとPCを接続するためのUSBケーブル
・  Arduino IDEがインストールされたPC
　　　◦ボードマネージャーRaspberry Pi Pico/RP2040/RP2350がインストールされていること

   # やり方
　 ### 1. ブレッドボード上で配線する
   以下の図のように配線します。しっかりさしましょう。


   ### 2. プログラムを準備する
       Arduino IDEに以下のプログラムをコピーしましょう．

       ```cpp
       const int led = 0;

       void setup()
       {
         pinMode(led, OUTPUT);
       }

       void loop()
       {
           for (int i = 0; i <= 255; i++)
           {
                analogWrite(ledPin, i);
                 delay(10);
            }

 
           for (int i = 255; i >= 0; i--) 
           {
             analogWrite(ledPin, i);
             delay(10);
           }
        }

　　analogWrite()はArduino IDEでだけ使える特別な関数です．
    [このページ](https://www.musashinodenpa.com/arduino/ref/)で確認してみてください．

  ### 3. プログラムをマイコンに書き込む  
    PCとマイコンをUSBケーブルでつなぎます．
    Arduino IDEでボードの種類とCOMポートを選択します．

### 4. 動作確認
     LEDの明るさが変わっていたら完成です‼‼   
   
   
