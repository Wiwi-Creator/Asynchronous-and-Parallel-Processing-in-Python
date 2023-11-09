# Threading
###### tags: `Threading` `Lock` `release`

Overview
---
-  Python在執行時，通常採同步的處理模式。而Threading採用「執行緒」的方式，運用多個執行緒，在同一時間內處理多個任務 ( 非同步 )

**Threading 基本用法** :
------------------------------

|Method    |說明                                  |
|----------|--------------------------------------|
|start()   |啟用Thread                             |
|join()    |等待Thread，直到該執行緒完成後才會進行後續動作|
|ident     |取得該Thread的標示符                      |
|native_id |取得該Thread的id                         |
|is_alive()|Thread是否啟用                           |

---
- 下方code執行後，會將aa,bb,cc三個函式分別建立thread，當使用a.start(),b.start()方法啟用後，因為加入了a.join(),b.join()的等待方法，所以c.start()會在aa()和bb()執行完成後才會啟用
```
import threading
import time


def aa():
    i = 0
    while i < 5:
        i = i + 1
        time.sleep(0.5)
        print('A:', i)


def bb():
    i = 0
    while i < 50:
        i = i + 10
        time.sleep(0.5)
        print('B:', i)


def cc():
    i = 0
    while i < 500:
        i = i + 100
        time.sleep(0.5)
        print('C:', i)


a = threading.Thread(target=aa)
b = threading.Thread(target=bb)
c = threading.Thread(target=cc)

a.start()
b.start()
a.join()   # 加入等待 aa() 完成的方法
b.join()   # 加入等待 bb() 完成的方法
c.start()  # 當 aa() 與 bb() 都完成後，才會開始執行 cc()

'''
A: 1
B: 10
A: 2
B: 20
A: 3
B: 30
A: 4
B: 40
A: 5
B: 50
C: 100 <--- A B 都結束後才開始
C: 200
C: 300
C: 400
C: 500
'''
```
**Lock()鎖定** :
------------------------------
- 使用Threading建立執行緒後，使用Lock()建立一個Thread的"鎖"，當Lock建立後，便能使用acquire()方法鎖定，使用release()解除鎖定，如果多個Thread共用一個Lock，則同一時間只會執行第一個鎖定的Thread，其他則要等到解鎖才能執行

- 以下code，使用Lock()建立一個Lock，當aa(),bb()執行時會進行鎖定(accquire())，而因為aa()先執行所以會先鎖定，直到i=2時才解鎖(release())，這時bb()才開始執行
```
import threading
import time

def aa():
    lock.acquire()         # 鎖定
    i = 0
    while i<5:
        i = i + 1
        time.sleep(0.5)
        print('A:', i)
        if i==2:
            lock.release()  # i 等於 2 時解除鎖定

def bb():
    lock.acquire()          # 鎖定
    i = 0
    while i<50:
        i = i + 10
        time.sleep(0.5)
        print('B:', i)
    lock.release()

lock = threading.Lock()         # 建立 Lock
a = threading.Thread(target=aa)
b = threading.Thread(target=bb)

a.start()
b.start()

'''
A: 1
A: 2
B: 10
A: 3
B: 20
A: 4
B: 30
A: 5
B: 40
B: 50
'''
```

**Event()事件處理** :
------------------------------
- 透過事件的方式，使不同的Thread之間溝通，達到"等待A完成某件事，B再執行"

|Method    |說明                          |
|----------|-----------------------------|
|Event()   |註冊一個事件                    |
|set()     |觸發事件                       |
|wait      |等到事件被觸發                  |
|clear()   |清除事件觸發，事件回到未被觸發的狀態|

下方code，會註冊一個event，當aa()執行時使用wait()等待event被觸發，然後設定bb()執行到 i = 30 時觸發event，這時aa()才會開始執行

```
import threading
import time

def aa():
    event.wait()            # 等待事件被觸發
    event.clear()           # 觸發後將事件回歸原本狀態
    for i in range(1,6):
        print('A:',i)
        time.sleep(0.5)

def bb():
    for i in range(10,60,10):
        if i == 30:
            event.set()     # 觸發事件
        print('B:',i)
        time.sleep(0.5)

event = threading.Event()   # 註冊事件
a = threading.Thread(target=aa)
b = threading.Thread(target=bb)

a.start()
b.start()

'''
B: 10
B: 20
B: 30
A: 1
B: 40
A: 2
B: 50
A: 3
A: 4
A: 5
'''
```
