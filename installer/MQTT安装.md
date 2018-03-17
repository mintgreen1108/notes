#安装MQTT

* apt-get update
* apt-get install wget
* wget http://mosquitto.org/files/source/mosquitto-1.4.5.tar.gz
* tar zvxf mosquitto-1.4.5.tar.gz
* cd mosquitto-1.4.5
* apt-get install make
* make 前置条件
  * apt-get install gcc
  * apt-get install libssl-dev
  * apt-get install libc-ares-dev
  * apt-get install libc-ares2
  * apt-get install g++
  * apt-get install uuid-dev
* make 
* make install  