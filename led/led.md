前までにインストールしたWiringPiを使用してLEDを光らせる、いわゆるLチカに挑戦してみます。
<br><br>

# 1. 1つのLEDについて点灯・消灯を繰り返す
まずは赤色LEDについて0.5秒点灯、その後0.5秒消灯を繰り返す動作を行うようにしてみます。回路は以下のようになっています。

(ここに回路図を載せる)
ラズパイ側の電源を落とした上でgpioにジャンプワイヤをつなぎ、接続を確認したうえで電源をつけ、コードを実行します。

また、C言語での実装は次のようになっています。
```C
/*led01.c*/
#include <stdio.h>
#include <stdlib.h>
#include <wiringPi.h>

#define LED0 23 //自身で使用するGPIOの番号

int main(void)
{
    wiringPiSetupGpio();
    pinMode(LED0, OUTPUT);

    for(;;){
        digitalWrite(LED0, HIGH);
        delay(500);
        digitalWrite(LED0, LOW);
        delay(500);
    }
    return EXIT_SUCESS;
}
```
これを
```
user@raspberrypi : ~$ gcc -o led01 led01.c -lwiringPi -lpthread -g -O0
user@raspberrypi : ~$ ./led01
```
として実行します。
