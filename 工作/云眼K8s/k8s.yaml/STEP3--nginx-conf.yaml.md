```yaml
apiVersion: v1

kind: ConfigMap

metadata:

  name: nginx-configuration

  namespace: yucekj

data:

  cloud-eyes.conf: |

    server {

        listen 80;

        client_max_body_size 5000M;

        proxy_connect_timeout 300;

        proxy_send_timeout 600;

        proxy_read_timeout 600;

  

        client_header_buffer_size 512k;

        large_client_header_buffers 4 512k;

  

        gzip on;

        gzip_min_length 1k;

        gzip_buffers 4 16k;

        gzip_http_version 1.1;

        gzip_comp_level 9;

        gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php application/javascript application/json;

        gzip_disable "MSIE [1-6]\.";

        gzip_vary on;

  

        location / {

            gzip on;

            if ($request_filename ~* ^.*?.(html|htm)$) {

                add_header Cache-Control "no-cache";

            }

            if (!-e $request_filename) {

                add_header Cache-Control "no-cache";

            }

  

            alias /opt/yuce/cloud-eyes/dist/;

            try_files $uri /index.html;

            index  index.html index.htm;

        }

        location /api/ {

            if ($request_method = 'OPTIONS') {

                add_header 'Access-Control-Allow-Origin' '*';

                add_header 'Access-Control-Allow-Credentials' 'true';

                add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';

                add_header 'Access-Control-Allow-Headers' 'x-authorization,DNT,web-token,app-token,Authorization,Accept,Origin,Keep-Alive,User-Agent,X-Mx-ReqToken,X-Data-Type,X-Auth-Token,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range';

                add_header 'Access-Control-Expose-Headers' 'Content-Length,Content-Range';

                add_header 'Access-Control-Max-Age' 1728000;

                add_header 'Content-Type' 'text/plain; charset=utf-8';

                add_header 'Content-Length' 0;

                return 204;

            }

            proxy_pass http://cloud-eyes-svc.yucekj:7001/;

        }

    }
```