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
	    # Disable cache
            add_header 'Cache-Control' 'no-cache';

            # CORS setup
            add_header 'Access-Control-Allow-Origin' '*' always;
            add_header 'Access-Control-Expose-Headers' 'Content-Length';

            # allow CORS preflight requests
            if ($request_method = 'OPTIONS') {
                add_header 'Access-Control-Allow-Origin' '*';
                add_header 'Access-Control-Max-Age' 1728000;
                add_header 'Content-Type' 'text/plain charset=UTF-8';
                add_header 'Content-Length' 0;
                return 204;
            }

            types {
                application/dash+xml mpd;
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
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
## 測試
- 測試網址：http://127.0.0.1:8080/webserver/streamTemp/ch3.m3u8 ，有下載影片代表成功。
- video.js網頁直播，參考來源---->https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>JW7 Player test</title>
    <!-- peer5 client library -->
    <script src="https://api.peer5.com/peer5.js?id=YOUR_PEER5_CUSTOMER_ID"></script>

    <!-- videojs5 scripts - You can change to your self hosted scripts -->
    <link rel="stylesheet" href="https://api.peer5.com/videojs5/assets/video-js.min.css">
    <script src="https://api.peer5.com/videojs5.js"></script>

    <!-- peer5 plugin for videojs5 -->
    <script src="https://api.peer5.com/peer5.videojs5.hlsjs.plugin.js"></script>
</head>
<body>

<video id="player" class="video-js vjs-default-skin" height="360" width="640" controls preload="none">
    <source src="http://127.0.0.1:8080/webserver/streamTemp/ch3.m3u8" type="application/x-mpegURL" />
</video>
<script>
    var player = videojs('#player');
</script>
</body>
</html>
```
## 高機率問題
1. m3u8檔案IIS無法識別--->[source_1](https://stackoverflow.com/questions/5513469/iis-server-m3u8-extension-not-opening),[source_2](https://docs.microsoft.com/en-us/iis/configuration/system.webserver/staticcontent/mimemap)
