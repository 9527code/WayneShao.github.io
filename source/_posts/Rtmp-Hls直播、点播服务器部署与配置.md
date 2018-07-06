---
title: Rtmp/Hls直播、点播服务器部署与配置
tags:
  - nginx-rtmp
  - 流媒体
  - hls
  - 点播
abbrlink: 13147
date: 2017-02-14 17:38:39
---
测试使用的系统为CentOS7.3、所有文章中涉及到的包打包在以下地址:

链接：http://pan.baidu.com/s/1nuF3gLV 密码：fo8q

 <!-- more -->
 ## Nginx-Rtmp-Module 安装
 1). 安装依赖包
 ```bash
        yum -y install gcc glibc glibc-devel make nasm pkgconfig openssl-devel expat-devel gettext-devel libtool perl-Digest-SHA1.x86_64
```
2). yum 安装相关工具包及 ffmpeg 依赖包
   ```
   yum -y install git zlib pcre openssl
```
3). 手动编译安装工具包和依赖包
    a). **yadmi**
     ```
     tar xzvf yamdi-1.9.tar.gz
     cd yamdi-1.9
     make && make install
     cd ..
     ```
b). **yasm**
    ```
         tar xzvf yasm-1.3.0.tar.gz
         cd yasm-1.3.0
         ./configure
         make && make install
         cd ..
         ```
c). **x264**
    ```
         tar -xjvf x264.tar.bz2
         cd x264-snapshot-20170111-2245
         ./configure --enable-shared 
         make && make install
         cd ..
         ```
d). **lame**
    ```
         tar xzvf lame-3.99.5.tar.gz
         cd lame-3.99.5
        ./configure --enable-nasm
         make && make install
         cd ..
         ```
e). **faad2**
    ```
         tar zxvf faad2-2.7.tar.gz
         cd faad2-2.7
         ./configure
         make && make install
         cd ..
         ```
f). **faac**
```
         tar zxvf faac-1.28.tar.gz
         cd faac-1.28
         ./configure
         make && make install
         cd ..
         ```
g). **xvid**
```
         tar zxvf xvidcore-1.3.3.tar.gz
         cd xvidcore/build/generic
         ./configure
         make && make install
         cd ..
         ```
h). **ffmpeg**
```
         tar -xjvf ffmpeg-3.2.4.tar.bz2
         cd ffmpeg-3.2.4
         ./configure  --prefix=/opt/ffmpeg/ --enable-version3 --enable-libvpx --enable-libmp3lame  --enable-libvorbis --enable-libx264 --enable-libxvid --enable-shared --enable-gpl --enable-postproc--enable-nonfree  --enable-avfilter --enable-pthreads
         make && make install
         cd ..
         ```
i). 修改/etc/ld.so.conf如下:
        ```
        include ld.so.conf.d/*.conf
        /lib
        /lib64
        /usr/lib
        /usr/lib64
        /usr/local/lib
        /usr/local/lib64
        /opt/ffmpeg/lib
        ldconfig
        ```
4). 安装 Nginx
        ```
        tar zxvf nginx-1.9.9.tar.gz
        unzip nginx-rtmp-module-master.zip
        tar zxvf openssl-1.0.2k.tar.gz
        cd nginx-1.9.9
        ./configure --add-module=../nginx-rtmp-module-master --without-http_rewrite_module --with-openssl=../openssl-1.0.2k
        make & make install
        cd ..
         ```
## nginx.conf配置
 ```
     # nginx.conf Start
    worker_processes  1;                    #  nginx对外提供 web 服务时的 worker 进程数

    error_log  logs/error.log debug;        # 错误日志路径

    pid        logs/nginx.pid;                #  pid 文件路径
    worker_rlimit_nofile 51200;                #  worker 进程的最大打开文件数限制

    events {                                #  events 模块中包含 nginx 中所有处理连接的设置。
        use epoll;                                # 设置用于复用客户端线程的轮询方法。
        worker_connections  51200;                #由一个 worker 进程同时打开的最大连接数。
    }

    rtmp_auto_push on;                        # 切换自动推送(多 worker 直播流)模式

    rtmp_auto_push_reconnect 1s;            # 当 worker 被干掉时设置自动推送连接超时时间。默认为 100 毫秒。

    rtmp {                                    # 保存所有 RTMP 配置的块。    
        server {                                # 声明一个 RTMP 实例。
            listen 1935;                            # 监听的端口号
            chunk_size 4096;                        # 流整合的最大的块大小。默认值为 4096。

            application vod {                        # 创建一个 RTMP 应用。
                play /opt/media/nginxrtmp/flv;            # 点播文件路径
            }

            application live {                        # 创建一个 RTMP 应用。
                live on;                                # 是否直播
                hls on;                                    # 是否开启hls
                hls_path /usr/local/nginx/html/live;    # 设置 HLS 播放列表和分段目录。
                hls_fragment 1s;                        # 设置 HLS 分段长度。
                max_connections 1024;                    # 最大连接数    
                hls_playlist_length 30s;                #  HLS 播放列表长度
                hls_sync 100ms;                            #  HLS 时间戳同步阈值
                meta copy;                                # 是否发送元数据到客户端    
                recorder manual {                        # 创建一个录制应用
                    record all manual;                        # 设置录制模式
                    record_suffix %Y-%m-%d-%H_%M_%S.flv;    # 设置录制文件名
                    record_max_size 6200000K;                # 设置录制文件的最大值    
                    record_path /usr/local/nginx/html/Rec;    # 指定录制的 flv 文件存放目录
                }
                #record keyframes;
                #record_path /tmp;
                #record_max_size 128K;
                #record_interval 30s;
                #record_suffix .this.is.flv;

                #on_publish http://localhost:8080/publish;
                #on_play http://localhost:8080/play;
                #on_record_done http://localhost:8080/record_done;
            }
            # application hls {  
            #     live on;  
            #     hls on;  
            #     hls_path /tmp/app;  
            #     hls_fragment 5s;  
            # }

            # application hls{
            #     live on;
            #     hls on;
            #     hls_path /usr/local/nginx/html/hls;
            #     hls_fragment 5s;
            # }
        }
    }

    http {
        server {
            listen      5000;
            keepalive_timeout  65;
            location /stat {
                rtmp_stat all;
                rtmp_stat_stylesheet stat.xsl;
            }

            location /stat.xsl {
                root /opt/nginx-rtmp-server/nginx-rtmp-module/;
            }

            location /control {
                rtmp_control all;
            }

            location /rtmp-publisher {
                root /opt/nginx-rtmp-server/nginx-rtmp-module/test;
            }

            location / {
                root /opt/nginx-rtmp-server/nginx-rtmp-module/test/www;
            }

            location /crplayer {
                root /opt/nginx-rtmp-server/nginx-rtmp-module/test;
            }
             
            location /live {  
               #server hls fragments  
                types{  
                    application/vnd.apple.mpegurl m3u8;  
                    video/mp2t ts;  
                }  
                root html;
                expires -1;  
            } 
        }
    }
    # nginx.conf End
 ```
 ##  运行Nginx服务
 ```
 /usr/local/nginx/sbin/nginx -c /root/nginx/nginx.conf
 ```