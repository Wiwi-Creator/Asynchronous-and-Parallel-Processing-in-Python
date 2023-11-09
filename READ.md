# Threading
###### tags: `Threading` `Lock` `release`

## 前言
進入同步,異步處理前需要先了解下列專有名詞:

- process(進程) : 獨立執行的一個程式實例。

    - 特點: Process間的記憶體空間是獨立的，不能訪問彼此的變數。
    - 資源: 每個Process都有自己的系統資源分配，如文件句柄、註冊表鍵等。
    - 由於Process有自己的記憶體空間，所以不受GIL（限制，使得Multi-Process在Python中適合CPU密集型任務。

- Thread(線程) : 是Process內的一個單獨的執行單元。
    - 共享記憶體: 由於線程共享相同的記憶體空間，這意味著一個線程可以直接訪問另一個線程的數據，但這也意味著必須使用同步機制（如鎖）來避免數據競爭。

    - 開銷: 相對於進程，創建新線程的開銷較小。

    - GIL: 由於GIL，Multi-Thread可能不適合CPU密集型任務，但適合I/O密集型任務。

- GLI(Global Interpreter Lock):全局解釋器鎖，用於同步的Multi-Thread，確保同一時間只有一個 Thread 執行python.

Overview
---
-  一般我們透過Python發出Request後，主執行緒(main thread)都會被暫停，我們在等待回應(Response)的期間什麼事情都不能做，只能靜靜等待回應。 -> 而這是所謂的同步。

而要達到異步處理，有以下方法:

- asyncio:
Python的一個標準庫，用於編寫單線程的異步代碼。
它使用 async/await 語法來定義異步函數。
主要用於I/O密集型任務，如網絡操作、文件操作等。
它不適用於CPU密集型任務，因為它在單一線程中運行。

- concurrent.futures:

    這是Python的一個標準庫，提供了高級接口來異步運行可調用對象。

    它有兩個主要的執行器：
    - ThreadPoolExecutor 使用多線程，
    - ProcessPoolExecutor 使用多進程。

- multi-thread (多線程):
線程是程序中的單一順序控制流程。
Python中的多線程由於全局解釋器鎖（GIL）的存在，並不總是真正的並行。對於CPU密集型任務，多線程可能不會帶來性能上的提升。
但對於I/O密集型任務，多線程可以提高性能。

- multi-process (多進程):
進程是操作系統分配資源（如CPU時間）的基本單位。
每個進程都有自己的記憶體空間和資源。
使用多進程可以真正地並行運行多個任務，特別是在多核CPU上。
由於每個進程都有自己的記憶體空間，因此進程之間的通信比線程之間的通信要複雜。


## 同步((Synchronous)) v.s 異步((Asynchronous))

同步(Synchronous): 接到一個任務後，需要等到它完成，才能繼續執行下一個任務。

非同步(Asynchronous): 平行處理，無需等待第一個任務完成，即可執行其它的任務，只要第一個任務完成了，就回來處理。

非同步適合用在io密集型的任務，ex:網路端的操作、爬蟲…

## Threading 基本用法:

## 總結：

如果您的任務是I/O密集型的，asyncio 或多線程可能是一個好選擇。
如果您的任務是CPU密集型的，多進程可能是更好的選擇。
concurrent.futures 提供了一個簡單的方式來使用多線程或多進程。
選擇哪種方法取決於您的具體需求和應用場景。