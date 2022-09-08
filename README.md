# nginx-vod-kaltura


https://facsiaginsa.com/nginx/build-adaptive-bitrate-vod-server-nginx
apt update && apt install build-essential git libpcre3-dev libssl-dev zlib1g-dev ffmpeg libxml2-dev (1059 MB)
wget http://nginx.org/download/nginx-1.20.2.tar.gz
tar -zxvf nginx-1.20.2.tar.gz
wget https://github.com/kaltura/nginx-vod-module/archive/refs/tags/1.29.tar.gz
tar -zxvf 1.29.tar.gz
cd nginx-1.20.2

./configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/run/nginx.pid \
    --sbin-path=/usr/sbin/nginx \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_stub_status_module \
    --with-http_realip_module \
    --with-file-aio \
    --with-threads \
    --with-stream \
    --with-cc-opt="-O3 -mpopcnt" \
    --add-module=../nginx-vod-module-1.29

make && make install
nginx -V
cd /etc/nginx
mv nginx.conf nginx.conf.old
nano nginx.conf


  GNU nano 4.8                                                            nginx.conf                                                                      
user www-data;
worker_processes auto;
worker_rlimit_nofile 8192;
pid /run/nginx.pid;

events {
    worker_connections 4096;
}

http {
    server {
        listen 80;

        # vod mode
        vod_mode local;

        # vod caches
        vod_metadata_cache metadata_cache 512m;
        vod_response_cache response_cache 128m;
        vod_mapping_cache mapping_cache 5m;

        # gzip manifests
        gzip on;
        gzip_types application/vnd.apple.mpegurl;

        # file handle caching
        open_file_cache          max=1000 inactive=5m;
        open_file_cache_valid    2m;
        open_file_cache_min_uses 1;
        open_file_cache_errors   on;

        location ^~ /video/ {
            alias /etc/nginx/vod/;
            autoindex on;
            vod hls;

            add_header Access-Control-Allow-Headers '*';
            add_header Access-Control-Expose-Headers 'Server,range,Content-Length,Content-Range';
            add_header Access-Control-Allow-Methods 'GET, HEAD, OPTIONS';
            add_header Access-Control-Allow-Origin '*';
            expires 100d;
        }
    }
}
