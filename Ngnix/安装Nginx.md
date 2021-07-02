## 安装Nginx服务器

**下载**
官方网址：http://nginx.org/en/download.h

Nginx官网提供了三个类型的版本
- Mainline version：Mainline 是 Nginx 目前主力在做的版本，可以说是开发版
- Stable version：最新稳定版，生产环境上建议使用的版本
- Legacy versions：遗留的老版本的稳定版

**安装依赖**

```shell
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
```

**解压nginx压缩包**
```shell
tar -zxvf nginx-1.18.0.tar.gz
```

**执行C语言编译命令**
```shell
./configure

make

make install
```

**默认安装位置**
```shell
/usr/local/nginx
```

## Nginx的启动和关闭

#### 启动

- **进入安装目录：** `cd /usr/local/nginx/sbin/`
- **启动：** `./nginx`


#### 关闭

- **查询nginx进程：** `ps -ef|grep nginx`
- 会出现两个nginx进程，关闭master进程即可

#### 重启

- **进入安装目录：** `cd /usr/local/nginx/sbin/`
- **重启：** `./nginx -s reload`

#### 指定配置文件启动

不论是启动还是重启都会默认使用conf 目录下的nginx.conf 配置文件
- `./nginx -c [绝对路径]`
