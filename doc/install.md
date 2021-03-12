# 环境安装

Go目前支持三大平台的安装，`Linux`，`Mac`，`Windows`。具体方法如下

## Linux

1. 下载压缩包[go1.16.2.linux-amd64.tar.gz](https://golang.org/dl/go1.16.2.linux-amd64.tar.gz)，

	然后执行命令解压到`/usr/local/go`目录下

    ```bash
    rm -rf /usr/local/go && tar -C /usr/local -xzf go1.16.2.linux-amd64.tar.gz
    ```

	!> 注意：如果不是`root`用户，请带上`sudo`

2. 设置环境变量

	在你的环境变量文件`($HOME/.profile 或者 /etc/profile)`追加如下内容：

    ```bash
    export PATH=$PATH:/usr/local/go/bin
    ```

3. 测试运行命令

	到此，`go`已安装完毕，可以执行命令测试一下

    ```bash
    go version
    ```

	如果出现`go`的版本信息，则说明安装成功

    ```bash
    # go版本信息
    go version go1.16.2 linux/amd64
    ```

## Windows

1. 下载安装包[go1.16.2.windows-amd64.msi](https://golang.org/dl/go1.16.2.windows-amd64.msi)

2. 直接双击安装包进行安装

3. 打开*CMD*命令行执行命令

   ```bash
   go version
   ```

   如果出现`go`版本信息，则说明安装成功

## Mac

!> 参照Linux安装方法