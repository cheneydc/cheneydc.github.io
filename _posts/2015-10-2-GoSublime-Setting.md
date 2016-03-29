---
layout: post
title: Sublime Text中GoSublime的设置
category: 工具 
---

由于不是土豪，下个破解的Sublime Text，配置下GoSublime，把env设置在了“Setting Default"里面，结果总是失败，后来发现下载的破解版别人已经在“Setting User”里面设置过了，系统是先读取这个了，所以更改下User里面的配置就可以了。

```
    "env": {
        "GOARCH":"amd64", //系统变量里面的 GOHOSTARCH ，386为32位平台,amd64为 64位平台
        "GOOS":"windows", //系统里面的GOOS
        "GOROOT":"C:\\go",
        "GOPATH":"C:\\Users\\dongc\\workspace\\GO",
        "PATH":"%GOROOT%\\bin;"
    }
```
