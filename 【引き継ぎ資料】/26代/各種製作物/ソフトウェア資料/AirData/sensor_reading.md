# センサーの読み取り（エアデータ部分）

## 

## GPS
### GPSモジュールの設定変更
GPSモジュールの設定はUART通信で変更する．多くの製品はNMEAフォーマットに対応しており，それに従って設定を行う．

#### 26代の場合
Teseo LIV3FLを用いたため，それに従って説明する．設定に必要な情報は[公式のソフトウェアマニュアル](https://www.st.com/resource/en/user_manual/um2229-teseoliv3-gnss-module--software-manual-stmicroelectronics.pdf)を参照．
- 基本的には，`$PSTMSETPAR,<どこを変更するか>,<変更する項目のID>,<値>,<checksum>\n`のフォーマットである．`$PSTMSETPAR`の使い方は，[公式のソフトウェアマニュアル](https://www.st.com/resource/en/user_manual/um2229-teseoliv3-gnss-module--software-manual-stmicroelectronics.pdf)のp.71 `10.3.1 $PSTMSETPAR`に掲載されている．

  ```c:Teseo_setup.ino
  #include <Arduino.h>

  SerialPIO Serial_GPS(14, 13, 4096); // GPSモジュールを接続するピンを設定．RP2040を想定．
  TinyGPSPlus gps;

  void setup() {
    Serial.begin(921600);
    Serial_GPS.begin(9600); // GPSモジュールの通信速度を設定
    
    delay(500); // 一応delayあったほうがいいと思う．GPSモジュールの起動時間を考慮．
    
    Serial_GPS.println("$PSTMSETPAR,1102,0xA*10"); // Baudrateを115200bpsに変更

    delay(500);
    Serial_GPS.println("$PSTMSETPAR,1303,0.10*05"); // 周期10Hzに変更
    
    Serial_GPS.println("$PSTMSAVEPAR*58"); // 設定をモジュール内部のフラッシュメモリに保存
    delay(500);
    
    Serial_GPS.println("$PSTMSRR*49"); // GPSモジュールを再起動
  }

  void loop() {
  }
  ```