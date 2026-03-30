# 使用 RuyiSDK 在 Milk-V Duo256m 上编译 zstd 教程

## 1.准备工作

### 下载系统镜像

```Plain Text
https://github.com/milkv-duo/duo-buildroot-sdk-v2/releases/download/v2.0.1/milkv-duo256m-musl-riscv64-sd_v2.0.1.img.zip
```

### 解压系统镜像

```Plain Text
unzip milkv-duo256m-musl-riscv64-sd_v2.0.1.img.zip
```

### 系统镜像烧录进存储卡

```Plain Text
sudo dd if=milkv-duo256m-musl-riscv64-sd_v2.0.1.img of=/dev/sdX bs=1M; sync
```

注意：将 `/dev/sdX` 替换为实际的存储卡设备名（如 `/dev/sdb`），避免误写系统磁盘导致数据丢失。

### 安装依赖包

```Plain Text
sudo apt update; sudo apt install -y wget tar zstd xz-utils git build-essential
```

### 安装ruyi包管理器

```Plain Text
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.45.0/ruyi-0.45.0.amd64
chmod +x ruyi-0.45.0.amd64
sudo cp -v ruyi-0.45.0.amd64 /usr/local/bin/ruyi
```

### 安装GCC工具链

```Plain Text
ruyi update
ruyi install gnu-plct 
```

## 2.获取 zstd 源码并编译（基于官方原生文件）

### 创建并激活ruyi虚拟环境（GCC）

```Plain Text
ruyi venv -t toolchain/gnu-plct milkv-duo venv-zstd
. ~/venv-zstd/bin/ruyi-activate
```

### 验证GCC版本

```Plain Text
riscv64-plct-linux-gnu-gcc -v
```

执行后应显示RISC-V架构的GCC工具链版本信息，确认工具链安装成功且激活生效。

### 克隆 zstd 项目源码

```Plain Text
git clone https://github.com/facebook/zstd.git
cd zstd
```

### 编译官方原生可执行程序（基础功能验证）

zstd 项目采用Makefile构建，需指定RISC-V交叉编译工具链：

```Plain Text
make -C lib CC=riscv64-plct-linux-gnu-gcc
make -C programs CC=riscv64-plct-linux-gnu-gcc LDFLAGS="-static"
```



## 3.将二进制文件传输至开发板并运行验证

### 将编译好的二进制传输至开发板

```Plain Text
scp programs/zstd root@192.168.42.1:~
```

### SSH连接到开发板

```Plain Text
ssh root@192.168.42.1
```

### 运行基础功能验证

#### 1. 查看zstd版本

```Plain Text
# 给程序开运行权限
chmod +x zstd

./zstd --version
```

执行后应输出zstd版本信息，确认程序可正常运行。

#### 2. 压缩/解压缩测试

```Plain Text
# 创建测试文件
echo "Test zstd on Milk-V Duo256m" > test.txt

# 压缩文件
./zstd test.txt -o test.txt.zst

# 解压缩文件
./zstd -d test.txt.zst -o test_uncompressed.txt

# 验证内容一致性
cat test_uncompressed.txt
```
