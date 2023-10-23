# concurrent.futures 平行任務處理
Overview
---
-  Python在執行時，通常採同步的處理模式。而Threading採用「執行緒」的方式，運用多個執行緒，在同一時間內處理多個任務 ( 非同步 )

**ThreadPoolExecutor** :
------------------------------
- ThreadPoolExecutor透過Thread的方式建立多個Executor(執行器)，執行並處理多個task，以下為其參數:

|參數               |說明                          |
|------------------|-----------------------------|
|max_workers       |Thread 的數量，預設 5 (每個 CPU 可以處理 5 個 Thread)，數量越多，速度越快，如果小於等於 0 會發生錯誤。|
|thread_name_prefix|Thread的名稱，預設''                       |
|initializer       |每個Thread啟動時調用的可調用對象，預設None           |
|initargs          |傳遞給初始化程序的參數，使用tuple，預設()|


使用ThreadPoolExecutor後，就能使用Executor的Method:

|Method  |參數              |說明      |
|--------|-----------------|--------- |
|submit  |fn,*args,**kwargs|執行某個函式|
|map     |func,*iterables  |使用 map 的方式，使用某個函式執行可迭代的內容。|
|shutdown|wait             |完成執行後return信號，釋放正在使用的資源，wait預設True會在所有對象完成後才回傳信號，若設定False則會在執行後立刻回傳|

- 下方code執行後，會按照順序顯示數字，前一個task未處理完則不會執行後續工作
```
import time 
def test(n):
    for i in range(n):
        print(i , end= ' ')
        time.sleep(0.2)
test(2)
test(3)
test(4)
```
- 修改成ThreadPoolExecutor方式，3個函式會一起進行(如果執行的函式>5，則須在設定max_workers的數值)
```
import time 
from concurrent.futures import ThreadPoolExecutor
def test(n):
    for i in range(n):
        print(i , end= ' ')
        time.sleep(0.2)
executor = ThreadPoolExecutor() #設定一個執行Thread的執行器
test(2)
test(3)
test(4)
executor.shutdown() #關閉器動器(如果沒有使用，則啟動器會在鎖住的狀態而無法繼續)
```
同理修正成with..as的方式，使程式簡潔
```
import time 
from concurrent.futures import ThreadPoolExecutor
def test(n):
    for i in range(n):
        print(i , end= ' ')
        time.sleep(0.2)
with ThreadPoolExecutor() as executor:
    executor.submit(test,2)
    executor.submit(test,3)
    executor.submit(test,4)
```
或是map
```
import time 
from concurrent.futures import ThreadPoolExecutor
def test(n):
    for i in range(n):
        print(i , end= ' ')
        time.sleep(0.2)
with ThreadPoolExecutor() as executor:
    executor.map(test,[2,3,4])
```

**輸入文字，使函式停止執行**
- 透過平行處理方式，我們希望做到"輸入特定文字，使函式停止"
Run 如果不使用平行任務處理，在其後方的程式都無法運作，而keyin是具有input指令的函式，如果不使用平行任務處理，其後方程式也無法運作，因此使用concurrent.futures搭配全域變數，讓兩個函式同時運作，並輸入特定指令，停止運作

```
import time
from concurrent.futures import ThreadPoolExecutor
a = True               
def run():
    global a           # 定義 a 是全域變數
    while a:           # 如果 a 為 True
        print(123)     # 不斷顯示 123
        time.sleep(1)  # 每隔一秒
def keyin():
    global a           # 定義 a 是全域變數
    if input() == 'a':
        a = False      # 如果輸入的是 a，就讓 a 為 False，停止 run 函式中的迴圈
executor = ThreadPoolExecutor()
e1 = executor.submit(run)
e2 = executor.submit(keyin)
executor.shutdown()
```
**ProcessPoolExecutor**:
------------------------------
- ProcessPoolExecutor透過Process的方式建立多個Executors，執行及處理多個程序，以下為其參數:
- 使用ProcessPoolExecutor後，也可以使用Executor的Method

|參數               |說明                                                   |
|------------------|------------------------------------------------------|
|max_workers       |Process的數量，預設為機器的CPU數量，<= 0 或 >= 61會發生錯誤。|
|thread_name_prefix|Thread的名稱，預設''                                    |
|initializer       |每個Thread啟動時調用的可調用對象，預設None                  |
|initargs          |傳遞給初始化程序的參數，使用tuple，預設()                    |



- ProcessPoolExecutor 的用法和 ThreadPoolExecutor 很像，但 ProcessPoolExecutor 主要會用做處理比較需要運算的程式，ThreadPoolExecutor  會使用於等待輸入和輸出 ( I/O ) 的程式
兩者執行後也會有些差別，ProcessPoolExecutor 執行後最後是顯示運算結果，而 ThreadPoolExecutor 則是顯示過程。

```
import time
from concurrent.futures import ProcessPoolExecutor

def test(n):
    for i in range(n):
        print(i, end=' ')
        time.sleep(0.2)
    print()

with ProcessPoolExecutor() as executor:
    executor.map(test, [4,5,6])
```

