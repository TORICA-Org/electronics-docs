# 4 - LEDの明るさを変える
`analogWrite()`を使います。

`analogWrite() `は
ピンから出す電圧の強さを0~255の段階で調整する関数です。

0を指定すると：完全にOFF

255を指定すると：完全にON(3.3Vか5V)

127などを指定すると：その中間の明るさとなります。

Aruduinoは2.5Vのような中間の電圧を直接出すことはできません。その代わりに、高速でスイッチをON/OFFして、平均的な電圧の調整をしています。これをPWMと呼びます。

スイッチがON/OFFされているので、LEDが点滅していることになりますが、人間の目には早すぎて「暗くなった」ように感じます。イメージとしては、ある時間内に、どれだけ電気の蛇口を全開にしているかという感じ。


## 必要なもの
- Raspberry Pi Pico
- ブレッドボード
- マイコンとPCを接続するためのUSBケーブル
- Arduino IDEがインストールされたPC
 - ボードマネージャー`Raspberry Pi Pico/RP2040/RP2350`がインストールされていること

 ## やり方
 ### 1. ブレッドボード上で配線する
 以下の図のように配線します。しっかりさしましょう。


 ### 2. プログラムを準備する
 Arduino IDEに以下のプログラムをコピーしましょう．
     
 ```cpp
 const int led = 0;

 void setup(){
   pinMode(led, OUTPUT);
 }

 void loop(){
   for (int i = 0; i <= 255; i++){
     analogWrite(led, i);
     delay(10);
   }
   for (int i = 255; i >= 0; i--){
     analogWrite(led, i);
     delay(10);
   }
 }
```
　analogWrite()はArduino IDEでだけ使える特別な関数です．
   [このページ](https://www.musashinodenpa.com/arduino/ref/)で確認してみてください．
  
### 3. プログラムをマイコンに書き込む  
PCとマイコンをUSBケーブルでつなぎます．
Arduino IDEでボードの種類とCOMポートを選択します．

### 4. 動作確認
LEDの明るさが変わっていたら完成です‼‼   

## 練習問題
### 1. 三角関数を用いて，LEDの明るさを変化させなさい．
[ヒント] 
- `sin()`関数または`cos()`関数があります．
- `sin()`や`cos()`の引数はラジアンです．
- `PI`で円周率が使えます．

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
const int led = 0;

void setup() {
  pinMode(led, OUTPUT);
}

float rad = 0.0;
void loop() {
  rad += 0.1;
  float offsetted_sin = sin(rad) + 1;
  int value = (int)(255.0*offsetted_sin/2);
  analogWrite(led, value);
}
```
</details>

### 2. しなさい．
[ヒント]
- ます

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
const int led = 5;

void setup() {
  pinMode(led, OUTPUT);
}

void loop() {
}
```
</details>
