# RTSP to HLS
## FFmpeg
#### 將RTSP轉成RTMP
```
start ffmpeg -re -i "rtmp://192.168.1.196:1935/live" -vcodec libx264 -preset:v ultrafast -tune:v zerolatency -acodec copy -f flv "rtmp://192.168.1.196:1935/hls/ch3 -loglevel quiet"
```
## Nginx
#### 打開nginx設定RTMP與http
  - RTMP
  ```
  rtmp {
    server {
        listen 1935;

        application live {
            live on;
        }
		
        application hls {
            live on;
            hls on;  
            hls_path D:/webServer/streamTemp;  
            hls_fragment 8s;  
            dash on;  
            dash_path D:/webServer/streamTemp2;  
            dash_fragment 8s;  
        }
    }
}
```
  - http
  ```
http {
    server {
        listen      8080;
		
        location / {
            root D:/webServer/;
        }
		
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }

        location /stat.xsl {
            root html;
        }
		
        location /hls {  
            #server hls fragments  
            types{  
                application/vnd.apple.mpegurl m3u8;  
                video/mp2t ts;  
            }  
            alias temp/hls;  
            expires -1;  
        }
        location /dash {
            types {
                application/dash+xml mpd;
                video/mp4 mp4;
            }
            alias temp/dash;  
            expires -1;  

        }  
    }
}
```
- 最後測試網址：http://127.0.0.1/webserver/streamTemp/ch3.m3u8 ，有下載影片代表成功。
