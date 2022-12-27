# 1. WirignPiのインストールができない
今回遊ぶ上で参考にしたテキストではGPIOを操作するコマンドとして`WiringPi`というライブラリのものを使用しています。しかし、これは2019年に開発元が開発停止を表明していたようで今後は非推奨のようです(参考にしたページに貼ってあるURLにとんでもページは見つかりませんが)。
そこで今回はいくつかるGPIO操作ライブラリのうち`pigpio`というものを使用することにしました。

# 2. pigpioのインストール
このライブラリを使用するとシェルからGPIOを叩いたりPythonやCからGPIOを操作できるようです。

aptコマンドでインストールできます。
```shell
sudo apt install pigio
```
また、このライブラリを使用してGPIOにアクセスするためにはpigpiodというものを起動する必要があるようなのでそのために以下のコマンドを実行します。
```shell
sudo pigpiod
```

# 3. 使い方

# 4. pigpio (C interface)の一部関数の紹介
[公式ページ](http://abyz.me.uk/rpi/pigpio/cif.html#gpioInitialise)を参考にしながらまとめた。
1. SetUp関連

    |関数     |  説明|
    |---------|---|
    |`int gpioInitialize()`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;| ライブラリの初期化を行う。正常に動作すればpigpioのversion番号を返す。pigpioライブラリを用いる場合、まずはじめにこの関数を呼び出さなければならない。|
    |`void Terminate()`|ライブラリを終了する。プログラムを終了する前に呼び出す。この関数は使用済みDMAチャンネルのリセット、メモリの開放、そして実行中のスレッドを終了させる。|

2. 使用頻度の高い関数

    |関数|説明|
    |--------|-----|
    |`int` <br> `gpioSetMode(unsigned gpio, unsigned mode)`|GPIO番号とモードを設定する。典型例でいうとinputかoutputかを指定する。<br>※注1参照|
    |`int gpioGetMode(unsigned gpio)`|GPIO番号を取得する。|
    |`int` <br>`gpioSetPullUpDown(unsigned gpio, unsigned pud)`|GPIO番号を指定し、oudにPUD_UP(プルアップ抵抗付き)、PUD_DOWN(プルダウン抵抗付き)、PUD_OFF(抵抗なし)の中から選択する。<br>※注2参照|
    |`int gpioRead(unsigned gpio)`|GPIO番号を指定し、入力された信号がonかoffかを返します。|
    |`int` <br>`gpioWrite(unsigned gpio, unsigned level)`|GPIO番号を指定し、onまたはoff(0 or 1)を選択します。|

    注1
    ```
    gpio : 0 - 53  (GPIO番号のこと)
    mode : 0 - 7   (GPIOピンはAlternate function(代替機能)を選択して、ピンに割り当てる機能を変更することができる。詳細は公式でドキュメントのp102)
    ```
    使用例は以下のとおりである。
    ```C
    gpioSetMode(17, PI_INPUT);  //GPIO17を入力としてセットする
    gpioSetMode(18, PI_OUTPUT); //GPIO18を出力としてセットする
    gpioSetMode(22, PI_ALT0);   //GPIO22を代替機能0としてセットする
    ```
    注2
    ```C
    gpioSetPullUpDown(17, PI_PUD_UP);   //Sets a pull-up
    gpioSetPullUpDown(18, PI_PUD_DOWN); //Sets a pu;;-down
    gpioSetPullUpDown(23, PI_PUD_OFF);  //Clear any pull-ups/down
    ```
3. PWM関連
    |関数|説明|
    |--------|-----|
    |`int`<br> `gpioPWM(unsigned user_gpio, unsigned dutycycle)`|GPIOでPWMを扱う。dutycycleは0(off)からrange(full on)の範囲である。デフォルトではrange = 255になっている<br>注3参照|
    
    注3
    ```
    user_gpio : 0-31
    dutycycle : 0-range
    ```
    使用例
    ```C
    gpioPWM(17,255); //Sets GPIO17 full on
    gpioPWM(18,128); //Sets GPIO17 half on
    gpioPWM(23,0);   //Sets GPIO23 full off
    ```

# 5. 補足事項
- `プルアップ、プルダウン`
    ...とりあえずは次のサイトを参照のこと([プルアップ抵抗・プルダウン抵抗とは？](https://voltechno.com/blog/pullup-pulldown/))。 あとで時間を見つけて自分の言葉で説明をかくことにします。
- `PWM = Pulse Width Modulation`
    ...パルス幅変調と呼ばれ、パルス波形のON/OHHの比率(=デューティー比)を変化させて信号を変調させる。LEDにPWMを使うことでLEDの点滅、調光が簡単に扱える様になる。ラズパイには2チャンネル(PWM0とPWM1)のハードウェアPWM回路がある。それぞれのチャンネルは2つのGPIOが接続しており、出力先を選択できる。1つのチャンネルから(2つのGPIO)からPWM信号を出力させられるが同一のPWM信号になる。
