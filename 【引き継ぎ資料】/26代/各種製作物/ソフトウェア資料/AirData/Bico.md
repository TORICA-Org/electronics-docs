# AirDataソフトウェアの簡単な解説

## 26th_Air_Bico.ino

### ライブラリインクルード部分
[L8-L12](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L8-L12)
```c
#include <Arduino.h>
#include "parameters.h"
#include "Bico_config.h"
#include "pico/multicore.h"
#include "pico/mutex.h"
```

[L28-L36](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L28-L36)
```c
// 各ファイル読み込み
#include "calculate_altitude.h"
#include "calculate_airspeed.h"
#include "AS5600.h"
#include "BMP3xx.h"
#include "SDP31.h"
#include "UARTHelper_Bico.h"
#include "GPSHelper.h"
#include "sensor_reader.h"
```

- `Arduino.h`はArduino環境でプログラムを書いているため、読み込む必要がある。
- `parameters.h`は全基板共通の変数（たとえば`data_air_sdp_differential_Pressure_hPa`）をまとめたファイル。これにより指示しているものは一緒なのに、プログラムごとに変数が異なって混乱してしまうことが防止できる。
- `Bico_config.h`はBicoのピン番号をまとめたファイルである。これにより使用するピンが突然変わっても、これを書き換えるだけで変更できるため柔軟に対応できる。
- `pico/multicore.h`や`pico/mutex.h`はRaspberry Pi Picoに搭載されているチップRP2040固有の機能を使うためにインクルードしている。具体的にはCPUのコア間でデータをやり取りするためのFIFOや、データを安全に受け渡しするためのmutexと呼ばれる機能を使うためにインクルードした。（FIFOやmutexについては後述）
- 28行目から36行目については、センサーの値を読むための関数をまとめたファイルである。

---

### 100Hzタイマー
- 正確に100Hz周期で実行するために、RP2040についているハードウェアタイマーを使用した。
  - ハードウェアタイマーを用いなくても、以下のような方法で100Hz周期で実行できるが、処理の遅延などで正確性に欠けてしまう、また計算リソースの節約にもなるのではと考えた。
  ```c
  // ハードウェアタイマーを用いずに100Hz周期で動かす場合
  unsigned long last_run_time = 0;
  void loop(){
    if (millis() - last_run_time >= 10){ // この10は10ms(=100Hz)を表す
      last_run_time = millis();
      run_task(); // 100Hzで実行するタスク
    }
  }
  ```

> [!NOTE]
> RP2040には64bitのタイマーが1個あり、1MHz(=1μs刻み)でカウントアップし続ける。アラームは4個まで設定できる。

- 処理を大雑把にまとめると、タイマーを用意→タイマーを設定→タイマー開始＆タイマーがなったらタイマーがなったことを知らせるフラグを立てる。loop処理では、そのフラグが立っているかどうかで10ms経過したかどうか判別して処理を進める。
- Raspberry Pi公式サイトに使い方が載っている。→[Raspberry Pi Hardware APIs](https://www.raspberrypi.com/documentation/pico-sdk/hardware.html#group_hardware_timer)


[L39-L44](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L39-L44)
```c
// 100Hzタイマー用
#include "pico/stdlib.h"
struct repeating_timer core0_timer;
volatile bool core0_timer_triggered = false;  //100Hz用フラグ
struct repeating_timer core1_timer;
volatile bool core1_timer_triggered = false;  //100Hz用フラグ
```
- ハードウェアタイマーを利用するには、`pico/stdlib.h`のインクルードが必要。このライブラリはArduino IDEでRP2040向けの環境設定を行うと自動でインストールされる。
- この部分でタイマーを宣言し、10ms(=100Hz)経過したかを示すフラグを宣言する。
  - `core0_timer`: タイマーの状態を保持する構造体。
  - `core0_timer_triggered`: タイマーが鳴ったかどうかを示すフラグ。`volatile`をつけることにより、割り込みの中で書き換えられるようにする。つけないとコンパイラにこの変数はループ内で変化しないと判断されて消されてしまい、フラグが反応しなくなることがある。


[L46-L49](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L46-L49)
```c
bool core0_timer_callback(struct repeating_timer *t) {
  core0_timer_triggered = true;
  return true;
}
```
- この部分は10msごとに自動的に呼ばれる関数である。やることは`core0_timer_triggered`を`true`にしてタイマーがなったことを知らせるだけ。
- `return true`でタイマーを続けるという意味。`false`だとタイマーが止まる。


[L137-L138](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L137-L138)
```c
// ハードウェアタイマー起動
add_repeating_timer_ms(-10, core0_timer_callback, NULL, &core0_timer);
```
- setup関数内で10ms周期のタイマーを登録。`add_repeating_timer_ms`の引数について、
  - `-10`: 10ms周期。マイナス符号はコールバック開始から次の開始までの間隔を10msにする指定。ここでは、上で宣言した`core0_timer_callback`関数を開始してから、次もう一度実行するまでの間隔を10msにするという意味。
  - `core0_timer_callback`: タイマーがなったら呼び出す関数。今回はタイマーがなったというフラグを立ててほしいので、この関数を呼ぶ。
  - `MULL`: ↑で呼び出す関数に渡す引数。今回はいらないのでNULL。
  - `&core0_timer`: 最初に宣言したタイマーの情報を保持する構造体のアドレスデータ。


[L151-L154](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L151-L154)
```c
void loop() {
  if (core0_timer_triggered == true) {

    core0_timer_triggered = false;  // タイマーのフラグを戻す
    // 以下、100Hz周期で実行したいことを記述する
```
- タイマーのフラグが立っている、つまり10ms経過したかどうか判断。もし10ms経過していたら、まずは真っ先にタイマーのフラグを戻して処理を進める。

---

### Watchdog
- Watchdogとは、CPUが何らかのエラーでフリーズしていないか確認する機能。もしフリーズしていたら強制的に再起動がかけられる。
> [!NOTE]
> Watchdogは名前のとおり、番犬である。犬には定期的に餌をやらなければならない。しかし餌をやらなければ、犬は鳴く。これと同じで、Watchdogに定期的に信号を送り、生存していることを伝える。もし信号が来なくなったら、フリーズしていると見なせる。

  [L56-L58](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L56-L58)
  ```c
  // Watchdog用
  #include "hardware/watchdog.h"
  volatile bool core1_alive;  // core1の生存確認用フラグ
  ```
- RP2040のWatchdogは一つしかない。しかし、コアは2つあるため、どちらか一方の監視にしか使うことができない。
  - そこで、WatchdogはCore0を監視し、Core0はCore1を監視するようにする。Core1はCore0に定期的に生存していることを伝え、Core0はWatchdogに生存を伝える。そのため`core1_alive`というフラグを設定する。

    [L134-L135](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L134-L135)
    ```c
    watchdog_enable(2000, 1);  // watchdogを有効化．
    /* 2000ms(=2s)経っても反応がない場合，システムが暴走したとみなして強制再起動 */
    ```
    - watchdogを有効化。引数の`2000`は2000ms、つまり2sのこと。`1`はデバッグ時に一時停止した際、カウントするか（`1`ならカウントしない）。なおここでいうデバッグとは、ブレークポイントで止めて変数の状態などを見るデバッグのことである（あまり気にしなくてよい）。


- Core1は自らが生存していることを示すために、`core1_alive`を`true`に設定する。そしてCore0は`core1_alive`が`true`になっていることを確認したら、`false`に戻す。これでCore0がCore1の生存を確認できたことになる。
  - Core1が自らの生存を示す部分
    [（L307行目）](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L307)
    ```c
    core1_alive = true;  // core1生存フラグを立てる
    ```
  - Core0がCore1の生存を確認し、そして自らの生存をWatchdogに示す部分
    
    [L223-L228](https://github.com/torica-org/2026-fm-archives/blob/main/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L223-L228)
    ```c
    / Core1生存確認
    if (core1_alive == true) {
      watchdog_update();  // Watchdogに合図を送る
    
      core1_alive = false;  // core1生存フラグを戻す
    }
    ```

---

### mutexとFIFO
- RP2040は2つのコアがあり，それぞれが演算を行っている．演算を行う際，2つのコアが同時に一つのデータにアクセスして書き込もうとすると，データにアクセスできなかったり，破損したりする．これを防止するために，mutex(排他的制御)という仕組みを使った．

> [!NOTE]
> MUTEXについて

- mutexの初期化

  [L14-L26](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L14-L26)

  ```c
  struct SharedSensorData {
      float air_pressure_hPa;
      float air_temperature_deg;
      float under_pressure_hPa;
      float under_temperature_deg;
      float fslg_pressure_hPa;
      float fslg_temperature_deg;
      float under_urm_altitude_m;
      float sdp_differentialPressure_Pa;
  };

  static SharedSensorData shared_sensor_data;
  static mutex_t sensor_mutex;
  ```
  - 上述したが，`"pico/mutex.h"`と`"pico/multicore.h"`のインクルードが必要．
  - `struct SharedSensorData`で`SharedSensorData`という構造体を宣言．この中にある変数を2つのコア間でやり取りすることになる．
  - 処理の大まかな流れとして，Core0で構造体`SharedSensorData`(インスタンス名：`shared_sensor_data`)にデータを格納→FIFOにデータが格納できたという合図を格納→Core1側でFIFOからデータを取り出す(POP)→データが格納できたという合図が取り出したデータに入っていたら，`SharedSensorData`にアクセスしてセンサー値を取り出す

  [L62](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L62)

  - setup関数内
  ```c
  // Mutexの初期化
  mutex_init(&sensor_mutex);
  ```
  構造体`sensor_mutex`をmutexの対象に入れることを宣言．

  - loop関数内(Core0)
    [L178-L191](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L178-L191)

    ```c
    // Core1へデータを安全に渡すため、Mutexロックを取得して共有領域へコピー
    mutex_enter_blocking(&sensor_mutex);
    shared_sensor_data.air_pressure_hPa = data_air_bmp_pressure_hPa;
    shared_sensor_data.air_temperature_deg = data_air_bmp_temperature_deg;
    shared_sensor_data.under_pressure_hPa = data_under_bmp_pressure_hPa;
    shared_sensor_data.under_temperature_deg = data_under_bmp_temperature_deg;
    shared_sensor_data.fslg_pressure_hPa = data_fslg_bmp_pressure_hPa;
    shared_sensor_data.fslg_temperature_deg = data_fslg_bmp_temperature_deg;
    shared_sensor_data.under_urm_altitude_m = data_under_urm_altitude_m;
    shared_sensor_data.sdp_differentialPressure_Pa = data_air_sdp_differentialPressure_Pa;
    mutex_exit(&sensor_mutex);

    // Core1へ書き込み完了シグナル（値1）を送る
    multicore_fifo_push_blocking(1);
    ```
    - Core0→Core1へデータを引き渡す．
      - `mutex_enter_blocking()`でコアの間で共有するメモリをロックする．
      - ロックしたら，データを共有部分のメモリへコピー．
      - `mutex_exit()`でロック解除
    - Core1へデータの書き込みが完了したことを知らせるために，FIFOにシグナルを送る．


  - loop1関数(Core1)

    [L250-L273](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L250-L273)
    ```c
    if (multicore_fifo_rvalid()) {
        multicore_fifo_pop_blocking(); // シグナルをポップ

        mutex_enter_blocking(&sensor_mutex);
        local_air_press = shared_sensor_data.air_pressure_hPa;
        local_air_temp  = shared_sensor_data.air_temperature_deg;
        local_under_press = shared_sensor_data.under_pressure_hPa;
        local_under_temp  = shared_sensor_data.under_temperature_deg;
        local_fslg_press = shared_sensor_data.fslg_pressure_hPa;
        local_fslg_temp  = shared_sensor_data.fslg_temperature_deg;
        local_under_urm   = shared_sensor_data.under_urm_altitude_m;
        local_sdp_diff   = shared_sensor_data.sdp_differentialPressure_Pa;
        mutex_exit(&sensor_mutex);
      } else {
       // シグナルがない場合は既存の値をそのまま使用
       local_air_press = data_air_bmp_pressure_hPa;
       local_air_temp  = data_air_bmp_temperature_deg;
       local_under_press = data_under_bmp_pressure_hPa;
       local_under_temp  = data_under_bmp_temperature_deg;
       local_fslg_press = data_fslg_bmp_pressure_hPa;
       local_fslg_temp  = data_fslg_bmp_temperature_deg;
       local_under_urm   = data_under_urm_altitude_m;
       local_sdp_diff   = data_air_sdp_differentialPressure_Pa;
      }
    ```
    - `if (multicore_fifo_rvalid()) {`で，
    - Core1で共有データを読む際も，`mutex_enter_blocking()`でCore0からメモリにアクセスしないようロック
    - `else`への分岐→`multicore_fifo_rvalid()`が1ではない，つまりCore0側が共有するデータを書き込めていないということである．その場合，Core1側で保持している古いデータをとりあえず使うことにしている．

---

### loop関数
#### センサー・GPS読み取り
- [L166-L176](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L166-L176)
  ```c
  // 各センサーの値を読み取り、グローバル変数に代入
  update_air_bmp();
  update_air_AS5600();
  update_air_SDP();

  // GPSは10Hzつまり100msに一回読む
  static int gps_counter = 0;
  if (gps_counter >= 10) {
    update_air_gps();
    gps_counter = 0;
  }
  gps_counter++;
  ```
  →ここを参照．関連するファイル：`AS5600.h/cpp`, `BMP3XX.h/cpp`, `GPSHelper.h/cpp`, `SDP31.h/cpp`, `sensor_reader.h/cpp`

---

### loop1関数
#### 通信
- 受信

  [L236-L240](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L236-L240)
  ```c
  // 機体下・胴体桁・ICS・ESP信号読み取り
  receiveUnderLog();
  receiveFslgLog();
  receiveIcsAngle();
  handleEspSignal();
  ```

- 送信

  [L288-L305](https://github.com/torica-org/2026-fm-archives/blob/Software/AirData/26th_Air_Bico/26th_Air_Bico.ino#L288-L305)

  ```c
  // UART送信用カウント変数
  static int transmit_count = 0;

  // UART送信
  transmitLog(transmit_count);
  transmit_count++;
  // 一通り送信(=transmit_countが4以上)したらカウントリセット
  if (transmit_count > 3) {
    transmit_count = 0;
  }

  // 胴体桁送信用カウント変数
  static int transmit_count_fslg = 0;
  transmitLog_for_fslg(transmit_count_fslg);
  transmit_count_fslg++;
  if (transmit_count_fslg > 2){
    transmit_count_fslg = 0;
  }
  ```
→どちらもこちら参照．関連するファイル：`UARTHelper_Bico.h/cpp`

---