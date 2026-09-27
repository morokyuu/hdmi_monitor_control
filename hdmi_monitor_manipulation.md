
# HDMIモニターの制御をする(on linux)

最初に、ddcutilを入れる
```
hoge@desk.local:~/ $ sudo apt install ddcutil
```

こんな風に、モニターが認識されているか確認する。
```
hoge@desk.local:~/ $ sudo ddcutil detect
Display 1
   I2C bus:  /dev/i2c-2
   DRM connector:           card0-HDMI-A-1
   EDID synopsis:
      Mfg id:               MTC - Mars-Tech Corporation
      Model:                HDMI
      Product code:         9347  (0x2483)
      Serial number:        00101
      Binary serial number: 269488144 (0x10101010)
      Manufacture year:     2012,  Week: 20
   VCP version:         2.2
```

使えるコマンド一覧を取得する。
```
hoge@desk.local:~/ $ sudo ddcutil capabilities
Model: RTK
MCCS version: 2.2
Commands:
   Op Code: 01 (VCP Request)
   Op Code: 02 (VCP Response)
   Op Code: 03 (VCP Set)
   Op Code: 07 (Timing Request)
   Op Code: 0C (Save Settings)
   Op Code: E3 (Capabilities Reply)
   Op Code: F3 (Capabilities Request)
VCP Features:
   Feature: 02 (New control value)
   Feature: 04 (Restore factory defaults)
   Feature: 05 (Restore factory brightness/contrast defaults)
   Feature: 06 (Restore factory geometry defaults)
   Feature: 08 (Restore color defaults)
   Feature: 0B (Color temperature increment)
   Feature: 0C (Color temperature request)
   Feature: 10 (Brightness)
   Feature: 12 (Contrast)
   Feature: 14 (Select color preset)
      Values:
         01: sRGB
         02: Display Native
         04: 5000 K
         05: 6500 K
         06: 7500 K
         08: 9300 K
         0b: User 1
   Feature: 16 (Video gain: Red)
   Feature: 18 (Video gain: Green)
   Feature: 1A (Video gain: Blue)
   Feature: 52 (Active control)
   Feature: 60 (Input Source)
      Values:
         01: VGA-1
         03: DVI-1
         0f: DisplayPort-1
         11: HDMI-1
   Feature: 87 (Sharpness)
   Feature: AC (Horizontal frequency)
   Feature: AE (Vertical frequency)
   Feature: B2 (Flat panel sub-pixel layout)
   Feature: B6 (Display technology type)
   Feature: C6 (Application enable key)
   Feature: C8 (Display controller type)
   Feature: CA (OSD/Button Control)
   Feature: CC (OSD Language)
      Values:
         01: Chinese (traditional, Hantai)
         02: English
         03: French
         04: German
         06: Japanese
         0a: Spanish
         0d: Chinese (simplified / Kantai)
   Feature: D6 (Power mode)
      Values:
         01: DPM: On,  DPMS: Off
         04: DPM: Off, DPMS: Off
         05: Write only value to turn off display
   Feature: DF (VCP Version)
   Feature: FD (Manufacturer specific feature)
   Feature: FF (Manufacturer specific feature)
```


コマンド実行の例
・Brightnessの取得
```
hoge@desk.local:~/ $ sudo ddcutil getvcp 10
VCP code 0x10 (Brightness                    ): current value =     0, max value =   100
```

・Contrastの取得

```
hoge@desk.local:~/ $ sudo ddcutil getvcp 12
VCP code 0x12 (Contrast                      ): current value =    47, max value =   100
```

・入力ソースの取得

```
hoge@desk.local:~/ $ sudo ddcutil getvcp 60
VCP code 0x60 (Input Source                  ): DisplayPort-1 (sl=0x0f)
```

・電源モードの取得
```
hoge@desk.local:~/ $ sudo ddcutil getvcp D6
VCP code 0xd6 (Power mode                    ): DPM: On,  DPMS: Off (sl=0x01)
```

電源モードで使えるのはこのDPM:On/Offでそれぞれ01/04。
05っていうのもある。これは電源ランプも消える。

```
   Feature: D6 (Power mode)
      Values:
         01: DPM: On,  DPMS: Off
         04: DPM: Off, DPMS: Off
         05: Write only value to turn off display
```

01,04は、実行してみると、毎回エラーが出る。
echo $?も１になっている。しかし電源モードをgetvcpで確認したらモードは切り替わっているので、
スクリプトで実行するときは
```
sudo ddcutil setvcp D6 01 || true
```
などとして実行結果を無視するか、コマンドでモードの切り替わりを確認するようにしないといけない。

```
hoge@desk.local:~/ $ sudo ddcutil setvcp D6 04
Verification failed for feature d6

hoge@desk.local:~/ $ sudo ddcutil getvcp D6
VCP code 0xd6 (Power mode                    ): DPM: Off, DPMS: Standby (sl=0x02)


hoge@desk.local:~/ $ sudo ddcutil setvcp D6 01
Setting value failed for feature d6, rc=DDCRC_RETRIES(-3007): maximum retries exceeded
    Try errors: DDCRC_RETRIES

hoge@desk.local:~/ $ sudo ddcutil getvcp D6
VCP code 0xd6 (Power mode                    ): DPM: On,  DPMS: Off (sl=0x01)
```


05だとエラーが出ない。電源ランプも消灯する。
多分これが電源OFFの確実なやり方。
```
hoge@desk.local:~/ $ sudo ddcutil setvcp D6 05
```


