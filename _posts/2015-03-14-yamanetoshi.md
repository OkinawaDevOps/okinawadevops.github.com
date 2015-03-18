---
layout: post
tags : [Fab, ]
---
{% include JB/setup %}

### [yamanetoshi](https://yamanetoshi.github.io/)

* [やること宣言](https://github.com/OkinawaDevOps/okinawadevops.github.com/issues/107)

### BMP085

今朝ピンヘッダをはんだ付けして初期動作確認済みです。

- [Raspberry Piに温度/気圧センサーを取り付ける](http://www.roshi.tv/2013/03/raspberry-pi.html)

また、無線ドングル使ってネト接続しようとしているのですが、接続に成功せずハマリ中。以下で接続成功。

    pi@raspberrypi:~$ cat /etc/network/interfaces
    auto lo
    
    iface lo inet loopback
	iface eth0 inet dhcp
	
	allow-hotplug wlan0
	iface wlan0 inet dhcp
	
	wireless-essid aterm-577d46-gw
	wireless-key 9d6a15493975d
	
	iface default inet dhcp

BMP085 の動作確認とか。

    pi@raspberrypi:~$ sudo i2cdetect -y 1
    0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
	00:          -- -- -- -- -- -- -- -- -- -- -- -- --
	10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	30: -- -- -- -- -- -- -- -- -- -- -- UU -- -- -- --
	40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
	70: -- -- -- -- -- -- -- 77

あるいは以下。

    $ cd Adafruit-Raspberry-Pi-Python-Code-master/Adafruit_BMP085
	$ sudo python Adafruit_BMP085_example.py
    Temperature: 24.50 C
    Pressure:    1017.53 hPa
	Altitude:    -36.07
	$

残り 10min で plot.ly と云々。

### plot.ly とのやりとり

以下を発見。

- [Raspberry PI Realtime Streaming with Plot.ly](https://github.com/plotly/raspberrypi)

ええと、まず以下?

    sudo apt-get install python-dev
    wget https://bitbucket.org/pypa/setuptools/raw/bootstrap/ez_setup.py -O - | sudo python
    sudo easy_install -U distribute
    sudo apt-get install python-pip
    sudo pip install rpi.gpio
    sudo pip install plotly

rpi.gpio はスデに導入済み、と言われました。いちおう python で import もできてます。

    $ python
	>>> import plotly.plotly
	>>>

で、これからどーすんの、という終了 30min 前。ええと、Adafruit のサンプルによると気圧だけなら以下で取れるのかな。

    #!/usr/bin/python
	
	from Adafruit_BMP085 import BMP085
	
	bmp = BMP085(0x77)
	
	pressure = bmp.readPressure()

同じディレクトリに Adafruit_BMP085.py というソレがありますね。あるいは Adafruit_I2C.py も必要なのかな。

### plotly のプロジェクト登録

この先どうしたものか、と思っていたら以下リポジトリを発見。

- [plotly/Streaming-Demos](https://github.com/plotly/Streaming-Demos)

ええと、以下を fork すれば良いのかな。

- [streamSimpleSensor](https://plot.ly/~streaming-demos/6/streaming-mock-sensor-data/)

save したら以下ができている。

- [streamSimpleSensor](https://plot.ly/~yamanetoshi/28/streaming-mock-sensor-data/)

とりあえずここにデータをぶち込むにはどうすれば良いのか。あるいは

- データの初期化

とかどうするのとか。あと以下にアクセスしたら

- [Getting started: Plotly for Python](https://plot.ly/python/getting-started/)

api_key とか見れたのですが strem_ids とか一体何かと。

### 設定画面?

確認したらデフォで token が用意されているようなのですが、そろそろ時間切れ。別場所にてもくもく続行したいと思います。

### 自宅にて

tor な無線 AP にデバイス接続して電源投入。一連の設定を盛り込みました。その後、

- Adafruit_BMP085/Adafruit_BMP085.py をコピィ
- Adafruit_I2C.py をコピィ

して以下な plot.py をでっち上げてます。
 # username, api_key, stream_token はダミー

    #!/usr/bin/python
    
    import plotly.plotly as py
    from plotly.graph_objs import Scatter, Layout, Figure
    import time
    from Adafruit_BMP085 import BMP085
    
	# dummy
    username = 'username'
    api_key = 'api_key'
    stream_token = 'stream_token' 
    
    bmp = BMP085(0x77)
    
    py.sign_in(username, api_key)
    
    trace1 = Scatter(
            x=[],
            y=[],
            stream=dict(
                    token=stream_token,
                    maxpoints=200
            )
    )
    
    layout = Layout(
            title='Raspberry PI Streaming BMP Sensor Data'
    )
    
    fig = Figure(data=[trace1], layout=layout)
    
    print py.plot(fig, filename='Raspberry PI Streaming Example Values')
    
    i = 0
    stream = py.Stream(stream_token)
    stream.open()
    
    while True:
            pressure = bmp.readPressure() / 100.0
            stream.write({'x': i, 'y': pressure})
            i+=1
            time.sleep(0.25)

これで streaming なグラフが動いていることを確認。

### 自動起動

面倒なので /etc/rc.local に実行するコマンドを追加。再起動して自動起動していることも確認しております。以下から確認可能です。

- [Raspberry PI Streaming BMP Sensor Data](https://plot.ly/~yamanetoshi/46/)



