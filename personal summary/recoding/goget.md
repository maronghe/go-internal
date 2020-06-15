go get 失败问题



> 国内解决`go get` 下载不下来的问题

解决方案：配置代理  参照 https://github.com/goproxy/goproxy.cn



### macOS 或 Linux

打开你的终端并执行

```
$ export GO111MODULE=on
$ export GOPROXY=https://goproxy.cn
```

或者

```
$ echo "export GO111MODULE=on" >> ~/.profile
$ echo "export GOPROXY=https://goproxy.cn" >> ~/.profile
$ source ~/.profile
```