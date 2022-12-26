# Intelligent Noise Preventer 
# 降噪智慧旗手


###### Made by Sammy Chen

## Overview

#####     身為一名住校學生，在沒有課的早晨，常因為宿舍外草地上的噪音失去睡意而被迫早起，因此做為發想。利用鋁擠、步進馬達、超音波感測器搭建出旗手，在旗手偵測到有人過於靠近時，旗手會揮動手臂與其連動的警示旗，提醒路過行人鄰近宿舍區需降低音量，設計出讓美夢不被中斷的幫手。

1. 透過超音波感測器偵測距離
2. 在距離(20cm)內，透過步進馬達牽動旗子，以提醒行經者
3. 如若有人在其手前停留太久(5cm)，旗手會透過E-mail提醒管理者檢查裝置是否有被移動


## Required Components
### Software
* Python 3.9

### Hardware
* Raspberry Pi 4B
* SD Card 32GB
* Aluminum Extrusion (3030) 100cm , 30cm x 2 , 20cm x2
* Aluminum Extrusion Corner Brackets(3030) x10
* US-100 Supersonic Sensor
* 5V Stepping Motor x2
* ULN2003 Stepping Motor Controller  x2

### Tools
* Carton 20cm x 30cm x 8cm
* Glue gun
* Tape
* Straw
* Cardboard

## Shape of Intelligent Noise Preventer

#### Appearance
![](https://i.imgur.com/PcTNdAf.jpg)


#### Closer Look Inside
![](https://i.imgur.com/hTqCLEq.jpg)


#### Raspberry Pi Setup
![](https://i.imgur.com/VPddvxC.jpg)



#### Stepping Motors
![](https://i.imgur.com/i2lgULe.jpg)

![](https://i.imgur.com/Hg96SIk.jpg)



#### Supersonic Sensor Setup
![](https://i.imgur.com/pMU8rgd.jpg)

![](https://i.imgur.com/VwFazUN.jpg)



## Circuit Diagram of Intelligent Noise Preventer
![](https://i.imgur.com/JcIyVVO.jpg)
* Supersonic Sensor
![](https://i.imgur.com/0TyZAeU.jpg)




## Crafting
1. A functioning Raspberry Pi 4B with 32GB SD Card

2. Format SD card
2-1. Download SD Card Formatter
[https://www.sdcard.org/downloads/formatter_4/](https://)
2-2. Plug the SD card into computer, open SD card formatter and format the SD card

3. Install Raspberry Pi OS on SD Card
3-1.Download Raspberry Pi OS(32-bit) with desktop and recommended software
[https://www.raspberrypi.org/downloads/raspberry-pi-os/](https://)
![](https://i.imgur.com/BTnWQGc.jpg)

3-2. Unzip the zip and get .img file 

3-3. Open config.txt inside .img file uncomment(remove #) hdmi_group and set to 2, uncomment hdmi_mode and set to 69, uncomment hdmi_group and set group 0 to 2, and set group 1 to 2, set hdmi_drive to 2
![](https://i.imgur.com/5M9ygr5.jpg)

3-4. Save file 

3-5. Use balenaEtcher to Flash raspberry pi OS to Sd card
[https://www.balena.io/etcher/](https://)
![](https://i.imgur.com/RWIrHJv.jpg)

3-6. Unmount SD card

3-7. Plug SD card into Raspaberry Pi. First, connect to monitor, mouse and keyboard. Then, connect to power cable.

3-8. Set Country, Language, Timezone

3-9. Choose a Wi-Fi to connect
![](https://i.imgur.com/DoCkm12.jpg)

3-10. Enable Webcam, VNC, SSH
![](https://i.imgur.com/0KJ77oN.jpg)
4. Install software

4-1. Update all packages
```
sudo apt-get update
```
4-2. Download python 3
```
sudo apt-get install python3
```
4-3. Download Visual Studio Code 
```
sudo apt install code
```
4-4. Setting up Visual Studio Code Environment
According to this tutorial videos
[https://youtu.be/QJTe9weVhSw](https://)
5. Write Code

5-1. Open Pycharm

5-2. Press new and choose create python file

5-3. Programming
```
import threading
from threading import Thread
from datetime import datetime
import time
import email.message
import smtplib
import queue
# from random import choice
# import keyboard
import RPi.GPIO as GPIO


GPIO.setwarnings(False)
GPIO.setmode(GPIO.BCM)
GPIO.cleanup()

PIN1 = [2, 3, 4, 17]
PIN2 = [12, 16, 20, 21]
MODE = {0: [0, 0, 0, 0],
        1: [1, 0, 0, 0],
        2: [1, 1, 0, 0],
        3: [0, 1, 0, 0],
        4: [0, 1, 1, 0],
        5: [0, 0, 1, 0],
        6: [0, 0, 1, 1],
        7: [0, 0, 0, 1],
        8: [1, 0, 0, 1]}

trigger_pin = 23
echo_pin = 24

# WAIT_TIME = 200
#
# global motor_flag
# global stop_program
motor_flag = 0     # start=1 stop=0
sensor_flag = 1    # start on=1, off=0
stop_program = 0
thread_flag = 0    # thread ok=1, nok=0

def mySetPin(m,PIN):
    mode = MODE[m]
    for idx, pin in enumerate(PIN):
        GPIO.output(pin, mode[idx])
        # print(pin, mode)

def motor1():
    global motor_flag
    global stop_program
    global thread_flag

    for pin in PIN1:
        GPIO.setup(pin, GPIO.OUT)

    mySetPin(0,PIN1)
    time.sleep(1)

    while True:
        if stop_program == 1:
            mySetPin(0, PIN1)
            print("motor1 stop")
            return None

        # wait thread ok
        # if thread_flag == 0:
        #     time.sleep(1)
        #     continue

        if motor_flag == 0:
            time.sleep(0.01)
            mySetPin(0, PIN1)
            continue

        mySetPin(0, PIN1)
        for i in [1, 2, 3, 4, 5, 6, 7, 8]:
            mySetPin(i,PIN1)
            time.sleep(0.01)
        mySetPin(0,PIN1)
    print("return motor1")

def motor2():
    global motor_flag
    global stop_program
    global thread_flag

    for pin in PIN2:
        GPIO.setup(pin, GPIO.OUT)
    mySetPin(0,PIN2)
    time.sleep(1)
    
    while True:
        if stop_program == 1:
            mySetPin(0, PIN2)
            print("motor2 stop")
            return None

        # wait thread ok
        # if thread_flag == 0:
        #     time.sleep(1)
        #     continue

        if motor_flag == 0:
            time.sleep(0.01)
            mySetPin(0, PIN2)
            continue

        # run motor
        mySetPin(0, PIN2)
        for i in [1, 2, 3, 4, 5, 6, 7, 8]:
            mySetPin(i, PIN2)
            time.sleep(0.01)
        mySetPin(0, PIN2)
    print("return motor2")



# V=331+0.6T (T：攝氏溫度)
# T=20 V=343
v = 343  # (331 + 0.6*20)
def measure():
    # GPIO.output(trigger_pin, GPIO.LOW)
    # time.sleep(0.00001)  # 10uS
    # print("measure0")
    # output: tx: HIGH => LOW
    GPIO.output(trigger_pin, GPIO.HIGH)
    time.sleep(0.0001)  # 10uS
    GPIO.output(trigger_pin, GPIO.LOW)
    pulse_start = None
    pulse_end = None

    # input: rx: HIGH => LOW
    i=0
    while GPIO.input(echo_pin) == GPIO.LOW:
        pulse_start = time.time()
        i=i+1
        if i > 10000:
            return 0
    # print("measure1",i)
    if i == 0:
        pulse_start = time.time()

    i=0
    while GPIO.input(echo_pin) == GPIO.HIGH:
        pulse_end = time.time()
        i=i+1
        if i > 10000:
             return 0

    # print("measure2", i)
    if i == 0:
        pulse_end = time.time()
    t = pulse_end - pulse_start
    d = t * v
    d = d / 2
    return d * 100

def measure_average():
    # print("measure_average")
    d1 = measure()
    time.sleep(0.05)
    d2 = measure()
    time.sleep(0.05)
    d3 = measure()
    # print(d1,":",d2,":",d3)
    if d1 == 0 and d2 == 0 and d3 == 0 :
        return 0
    n = 3
    if d1 == 0:
        n=n-1
    if d2 == 0:
        n=n-1
    if d3 == 0:
        n=n-1
    if n == 0:
        return 0
    distance = (d1 + d2 + d3) / n
    return distance

def pir():
    global motor_flag
    global stop_program
    global thread_flag
    # GPIO.setmode(GPIO.BCM)
    GPIO.setup(trigger_pin, GPIO.OUT)
    GPIO.setup(echo_pin, GPIO.IN)
    GPIO.output(trigger_pin, GPIO.LOW)
    i=0
    while True:
        if stop_program == 1:
            print("sensor stop")
            return None

        # wait thread ok
        # if thread_flag == 0:
        #     time.sleep(1)
        #     continue

        if sensor_flag == 0:
            time.sleep(0.5)
            continue

        # measure object distance
        measure_cm = measure_average()
        if measure_cm <=0 :
            GPIO.output(trigger_pin, GPIO.LOW)
            time.sleep(0.05)
            continue

        if measure_cm <= 5:
            print("run monitor cm=%.2f" % measure_cm)
            if motor_flag == 0:  # stop => start
                motor_flag = 1
                i = 0

            if motor_flag == 1 :
                i=i+1

            if (i % 10) == 0 :
                if not bq.full() :
                    bq.put(round(measure_cm,2))

        if measure_cm >= 10 :
            print("stop monitor cm=%.2f" % measure_cm)
            if motor_flag == 1:
                motor_flag = 0

        time.sleep(0.2)

    print("return pir")

def sendmail():
    global stop_program
    global thread_flag

    while True:
        if stop_program == 1:
            print("sendmail stop")
            return None

        # wait thread ok
        if thread_flag == 0:
            time.sleep(1)
            continue

        if bq.empty():
            continue

        print(threading.current_thread().name + "有 ", bq.qsize())
        server = smtplib.SMTP_SSL("smtp.gmail.com", 465)
        server.login("ncumistest536@gmail.com", "mwinarhhunbdzyyu")

        while bq.qsize() > 0 :
            distance = bq.get()
            print("sendmail for cm:",str(distance) )
            now = datetime.now()
            msg = email.message.EmailMessage()
            msg["From"] = "ncumistest536@gmail.com"
            msg["To"] = "skzhyunlixforever@gmail.com"

            subject = "時間:" + now.strftime("%Y/%m/%d %H:%M:%S")+ " 距離:" +str(distance) +"公分"
            msg["Subject"] = subject
            content = "請檢查!!"
            msg.set_content(content)

            print("傳送郵件 ok:"+subject)
            # print("login ")
            # server.login("ncumistest536@gmail.com","nwjkpllgkjoirrdt")
            server.send_message(msg)
            time.sleep(0.02)

        server.close()


        import keyboard

def keypress():
    global stop_program
    global sensor_flag
    global motor_flag
    global thread_flag
    try:
       while True:
           # wait thread ok
           # if thread_flag == 0:
           #     time.sleep(1)
           #     continue

           choice = input()
           if choice == "r":
               sensor_flag = 1
               print("Press r,run sensor")
           if choice == "s":
                motor_flag = 0
                sensor_flag = 0
                print("Press s,stop sensor and stop motor")
           if choice == "m":
               sensor_flag = 0
               motor_flag =1
               print("Press m,stop sensor and run motor")
           if choice == "q":
                stop_program = 1
                motor_flag = 0
                sensor_flag = 0
                print("Press q,stop program")

    except KeyboardInterrupt:
        stop_program = 1
        print("Exception: KeyboardInterrupt")
    finally:
        print("finally")




# 建立一個容量為1的Queue
bq = queue.Queue(maxsize=100)
k1 = Thread(target = keypress)
m1 = Thread(target = motor1)
m2 = Thread(target = motor2)
p1 = Thread(target = pir)
s1 = Thread(target= sendmail)

tasks =[]
k1.start()
m1.start()
m2.start()
p1.start()
s1.start()

tasks.extend([k1,p1,m1,m2,s1])
print("thead start")

###############################################
thread_flag = 1
###############################################
for task in tasks:
    task.join()
print("thead joined")
GPIO.cleanup()
time.sleep(3)
```
## Working Demo
1. [https://youtu.be/CE6WE4mKVlg](https://youtu.be/CE6WE4mKVlg)
2. [https://youtu.be/62ie74W4NRA](https://youtu.be/62ie74W4NRA)
3. 示警E-mail截圖
![](https://i.imgur.com/y4N62VS.jpg)



## Reference
* Raspberry Pi 4 樹莓派遊戲機實作[https://github.com/raspberrypi-tw/gpio-game-console](https://github.com/raspberrypi-tw/gpio-game-console)
* 超音波測距離 [https://atceiling.blogspot.com/2014/03/raspberry-pi_18.html](https://atceiling.blogspot.com/2014/03/raspberry-pi_18.html)
* 步進馬達 [http://hophd.com/raspberry-pi-stepper-motor-control/](https://hophd.com/raspberry-pi-stepper-motor-control/)
* Python Email 發送電子郵件 [https://youtu.be/YQboCnlOb6Y](https://youtu.be/YQboCnlOb6Y)
* Gmail 的 IMAP、SMTP、POP3 各項 port 及安全性協定 [https://j7.lb168.tw/2021/03/gmail-%E7%9A%84-imap%E3%80%81smtp%E3%80%81pop3-%E5%90%84%E9%A0%85-port-%E5%8F%8A%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A/](https://j7.lb168.tw/2021/03/gmail-%E7%9A%84-imap%E3%80%81smtp%E3%80%81pop3-%E5%90%84%E9%A0%85-port-%E5%8F%8A%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A/)
* OAuth 2.0 Mechanism [https://developers.google.com/gmail/imap/xoauth2-protocol](https://developers.google.com/gmail/imap/xoauth2-protocol)
* Python 獲取鍵盤輸入 [https://www.itread01.com/content/1549587608.html](https://www.itread01.com/content/1549587608.html)




