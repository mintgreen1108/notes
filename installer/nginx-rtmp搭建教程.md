##安装nginx-rtmp

###前提

安装 pcre、openssl、zlib

* pcre 安装教程：
  * wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/
  pcre-8.39.tar.gz 下载
  * tar -zvxf pcre-8.39.tar.gz 解压
  * cd pcre-8.39	.....       make
  * make
  * make check
  * make install 安装

* cc 安装教程
  * apt-get install build-essential

* openssl 安装教程
  * apt-get install openssl
  * apt-get install libssl-dev 


###开始安装

* nginx安装
  *  git clone https://github.com/arut/nginx-rtmp-module.git 
  * wget http://nginx.org/download/nginx-1.12.1.tar.gz
  * tar -xvzf nginx-1.12.1.tar.gz
  * cd nginx-1.12.1
  * ./configure --prefix=/usr/local/nginx --add-module=/var/www/nginx-rtmp-module --with-http_ssl_module --with-debug
  * make
  * make install
  * 检查 ／usr/local/nginx/sbin 下是否有nginx
  * nginx.conf添加相关配置：
   
   ~~~
    rtmp {
   server {
       listen 1935;
       chunk_size 4096;
       application myapp {
           live on;
       }
       application hls {
           live on;
           hls on;
           hls_path /tmp/hls;
       }
   }
   }
~~~