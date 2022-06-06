# GOPATH/bin下的可执行文件的用法

<!-- GFM-TOC -->
* [GOPATH/bin下的可执行文件的用法](#GOPATH/bin下的可执行文件的用法)
    * [dlv](#dlv)
        * [dlv attach](#dlv-attach)
        * [示例](#示例)
<!-- GFM-TOC -->

## dlv
go的调试工具
[GO delve(dlv)调试工具笔记及实操](https://zhuanlan.zhihu.com/p/425645473)

### dlv attach
跟踪一个正在运行的程序

这个命令将使Delve控制一个已经运行的进程，并开始一个新的调试会话。 当退出调试会话时，你可以选择让该进程继续运行或杀死它。

### 示例

示例代码

```
package main

import (
	"fmt"
	"log"
	"math/rand"
	"net/http"
	"time"
)

func count(i, j int) int {
	yz := 5
	result := (i + j) * yz
	return result
}

func randHandler(w http.ResponseWriter, r *http.Request) {
	rand.Seed(time.Now().Unix())
	var (
		i = rand.Intn(20)
		j = rand.Intn(20)
	)
	result := count(i, j)
	_, _ = w.Write([]byte(fmt.Sprintf("%d", result)))
}

func main() {
	http.HandleFunc("/rand", randHandler)
	err := http.ListenAndServe(":8080", nil)
	if err != nil {
		log.Fatalf("start server fail: %v", err)
	}
}
```

启动dlv

```
go build -o rand.exe main.go
start rand.exe
tasklist|grep rand
dlv attach 912
```

进入dlv后

```
# b: break的缩写，设置断点
b randHandler
# 请求一次，进入调试
curl http://localhost:8080/rand
# c: continue的缩写，继续执行到一个断点或者程序结束
c
```