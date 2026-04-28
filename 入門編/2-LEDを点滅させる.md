# 2 - LEDを点滅させる

マイコンを使うことで，電気が流れる／流れないを制御することができます．

ここでは，LEDのON/OFFを切り替えて，LEDを点滅させてみましょう！

## 必要なもの
- Raspberry Pi Pico，またはそれに準ずるマイコン
- ブレッドボード
- マイコンとPCを接続するためのUSBケーブル
- Arduino IDEがインストールされたPC
  - ボードマネージャ`Raspberry Pi Pico/RP2040/RP2350`がインストールされていること
 
## やり方
### 1. ブレッドボード上で配線する
以下の図のように配線します．**奥まで差し込むこと！**

配線図はパワポとかで作る．

### 2. プログラムを準備する
Arduino IDEの画面の一番上（メニューバー）から，サンプルプログラム（プログラムの作成例）を開きましょう．

`File` > `Examples` > `Blink` を探してください．

クリックすると，以下のようなプログラムが出るはずです．

```cpp
/*
  Blink

  Turns an LED on for one second, then off for one second, repeatedly.

  Most Arduinos have an on-board LED you can control. On the UNO, MEGA and ZERO
  it is attached to digital pin 13, on MKR1000 on pin 6. LED_BUILTIN is set to
  the correct LED pin independent of which board is used.
  If you want to know what pin the on-board LED is connected to on your Arduino
  model, check the Technical Specs of your board at:
  https://docs.arduino.cc/hardware/

  modified 8 May 2014
  by Scott Fitzgerald
  modified 2 Sep 2016
  by Arturo Guadalupi
  modified 8 Sep 2016
  by Colby Newman

  This example code is in the public domain.

  https://docs.arduino.cc/built-in-examples/basics/Blink/
*/

// the setup function runs once when you press reset or power the board
void setup() {
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(LED_BUILTIN, OUTPUT);
}

// the loop function runs over and over again forever
void loop() {
  digitalWrite(LED_BUILTIN, HIGH);  // change state of the LED by setting the pin to the HIGH voltage level
  delay(1000);                      // wait for a second
  digitalWrite(LED_BUILTIN, LOW);   // change state of the LED by setting the pin to the LOW voltage level
  delay(1000);                      // wait for a second
}

```

ここに出てきた`pinMode()`や`digitalWrite()`はArduino IDEでだけ使える特別な関数です．
[このページ](https://www.musashinodenpa.com/arduino/ref/)で確認してみてください．

しかし，このままではマイコンに内蔵されたLEDが光るだけで，せっかく繋いだLEDは光りません．
プログラムを以下のように修正する必要があります．

### 3. プログラムをマイコンに書き込む
PCとマイコンをUSBケーブルでつなぎます．
Arduino IDEでボードの種類とCOMポートを選択します．

### 4. 動作確認
1秒ごとにON/OFFが切り替わる，つまり2秒ごとに点滅していれば完成です！

## 練習問題
### 1. LEDが0.2秒点灯，0.8秒消灯を繰り返すプログラムを作成し，動作を確認しなさい．
[ヒント] 
- `delay()`関数はミリ秒単位で設定します．

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(200);
  digitalWrite(LED_BUILTIN, LOW);
  delay(800);  
}
```
</details>

### 2. LEDの配線場所を`GPIO5`に変更し，プログラムを修正したうえで動作を確認しなさい．
[ヒント]
- あれや
- これや

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
void setup() {
  pinMode(LED_BUILTIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(200);
  digitalWrite(LED_BUILTIN, LOW);
  delay(800);  
}
```
</details>
