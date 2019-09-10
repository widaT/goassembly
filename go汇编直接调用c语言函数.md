# go 汇编直接调用c语言函数

## 目录结构

```
├── assembly
│   ├── add.go
│   └── add.s
├── c
│   └── go.go
├── go.mod
└── main.go
```

## 文件的add.go 

```go
package assembly

import _ "unsafe"
import _ "cgotest/c"
//
func Add(b,c int64) (d int64)
func Add2(a uintptr, b,c int64 ) (d int64)
```

## 文件add.s
```go
#include "textflag.h"

TEXT ·Add(SB), $0
    MOVQ a+0(FP), DI     
    MOVQ b+8(FP), SI     
    CALL sum(SB)
	MOVQ AX,d+16(FP)
    RET

TEXT ·Add2(SB), $0
	MOVQ a+0(FP),AX
    MOVQ a+8(FP), DI    
    MOVQ b+16(FP), SI   
    CALL AX
	MOVQ AX,d+24(FP)
    RET

```

## 文件 go.go

```go
package c

//int sum(int a, int b) { return a+b; }
import "C"
import "unsafe"
var _ = C.sum  //这里需要加，否则汇编找不到c语言中的sum函数
var FpCsum = uintptr(unsafe.Pointer(C.sum))
```


## 文件mian.go

```go
package main

import (
	"cgotest/assembly"
	"cgotest/c"
)

func main() {
	println(assembly.Add(2,4))
	println(assembly.Add2(c.FpCsum,4,2 ))
}
```


## 运行

```bash
$ go run  main.go
6
6
```