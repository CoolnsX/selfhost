# selfhost

This repo contains the docker compose of each and every services that is self-hosted by me and already tuned to my preferences

Note: Please move the env.example in some directory to .env with the preferred values set before proceeding.

### Nextcloud
```yaml
services:
  nextcloud:
    image: nextcloud/all-in-one:latest-arm64
    restart: unless-stopped
    container_name: nextcloud-aio-mastercontainer
    ports:
      - "8080:8080"
    environment:
      - AIO_DISABLE_BACKUP_SECTION=true #remove this line if you want backup
      - APACHE_PORT=11000
      - APACHE_IP_BINDING=0.0.0.0
      - NEXTCLOUD_MOUNT=/media/vault/nextcloud/ #external storage mounted location
      - NEXTCLOUD_ENABLE_DRI_DEVICE=true #enables gpu acceleration on videos stored in nextcloud
      - NEXTCLOUD_UPLOAD_LIMIT=100G #you might want to adjust this
      - NEXTCLOUD_MAX_TIME=3600
      - NEXTCLOUD_MEMORY_LIMIT=1024M #you might want to adjust this
      - SKIP_DOMAIN_VALIDATION=true
      - NEXTCLOUD_KEEP_DISABLED_APPS=true
    volumes:
      - nextcloud_aio_mastercontainer:/mnt/docker-aio-config
      - /var/run/docker.sock:/var/run/docker.sock:ro

volumes:
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer
```
### *arr stack
```yaml
version: '3'
services:
  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=etc/UTC
    volumes:
      - radarr:/config
      - /media/vault/movies:/movies #change accordingly
      - /media/vault/downloads:/downloads #change accordingly
    ports:
      - 7878:7878
    restart: unless-stopped

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=etc/UTC
    volumes:
      - bazarr:/config
      - /media/vault/movies:/movies #change accordingly
    ports:
      - 6767:6767
    restart: unless-stopped

  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=etc/UTC
    volumes:
      - prowlarr:/config
    ports:
      - 9696:9696
    restart: unless-stopped

  transmission:
    image: lscr.io/linuxserver/transmission:latest
    container_name: transmission
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=etc/UTC
    volumes:
      - transmission:/config
      - /media/vault/downloads:/downloads #change accordingly
      - /media/vault/watch:/watch #change accordingly
    ports:
      - 9091:9091
      - 51413:51413
      - 51413:51413/udp
    restart: unless-stopped

volumes:
  bazarr:
  prowlarr:
  radarr:
  transmission:
```


## Below are the services that are put on another server

### Nginx Proxy Manager
```yaml
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      - '80:80'
      - '81:81'
      - '443:443'
    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```

### Collabora (if hosting on another server)
```yaml
services:
  collabora:
    image: collabora/code:latest
    container_name: collabora_code
    ports:
      - "9980:9980"
    environment:
      - "extra_params=--o:ssl.enable=false --o:ssl.termination=true --o:net.post_allow.host[0]=<ip_range_in_regex>" #must configure
      - 'domain=nextcloud.example.com' #point it to your nextcloud domain
    restart: always
    cap_add:
      - MKNOD
```

### Imaginary (if hosing on another server)
```yaml
services:
  imaginary:
    image: nextcloud/aio-imaginary:latest
    environment:
       PORT: 9000
    command: -enable-url-source -cors
    ports:
      - "9000:9000"
    cap_add:
      - SYS_NICE
```

## Now nginx config for each and every services I make public to make them more optimized by enabling gzip compression

### nextcloud
```nginx
client_body_buffer_size 256m;
proxy_read_timeout 86400s;
proxy_buffer_size 4k;
proxy_buffers 100 8k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
client_max_body_size 0;
output_buffers 4 64k;
proxy_hide_header Upgrade;
proxy_hide_header X-Powered-By;
gzip on;
gzip_types text/plain text/css text/javascript text/xml text/calendar text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy application/javascript application/json application/ld+json application/manifest+json application/rdf+xml application/rss+xml application/schema+json application/atom+xml application/xhtml+xml application/xml application/xml+rss application/soap+xml application/font-woff application/font-woff2 application/vnd.ms-fontobject application/x-font-ttf application/x-javascript application/x-web-app-manifest+json application/pdf application/vnd.ms-excel application/msword application/vnd.ms-powerpoint application/zip application/x-shockwave-flash application/xhtml+xml application/xslt+xml application/xml-dtd application/vnd.android.package-archive application/vnd.iphone application/vnd.wap.xhtml+xml application/xhtml+xml application/x-font-opentype application/x-font-truetype application/x-font-ttf application/x-javascript application/x-mpegURL application/x-rar-compressed application/x-shockwave-flash application/x-stuffit application/x-tar application/x-web-app-manifest+json application/xhtml+xml application/zip application/x-7z-compressed font/eot font/opentype image/bmp image/svg+xml image/vnd.microsoft.icon image/x-icon;
gzip_min_length 1024;
gzip_comp_level 9;
gzip_buffers 32 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```

### collabora | jellyfin
```nginx
client_max_body_size 50m;
output_buffers 4 64k;
proxy_hide_header Upgrade; #remove this for collabora in nginx proxy manager
proxy_hide_header X-Powered-By;
proxy_buffer_size 4k;
proxy_buffers 100 8k;
proxy_busy_buffers_size 64k;
proxy_temp_file_write_size 64k;
gzip on;
gzip_types text/plain text/css text/javascript text/xml text/calendar text/vnd.rim.location.xloc text/vtt text/x-component text/x-cross-domain-policy application/javascript application/json application/ld+json application/manifest+json application/rdf+xml application/rss+xml application/schema+json application/atom+xml application/xhtml+xml application/xml application/xml+rss application/soap+xml application/font-woff application/font-woff2 application/vnd.ms-fontobject application/x-font-ttf application/x-javascript application/x-web-app-manifest+json application/pdf application/vnd.ms-excel application/msword application/vnd.ms-powerpoint application/zip application/x-shockwave-flash application/xhtml+xml application/xslt+xml application/xml-dtd application/vnd.android.package-archive application/vnd.iphone application/vnd.wap.xhtml+xml application/xhtml+xml application/x-font-opentype application/x-font-truetype application/x-font-ttf application/x-javascript application/x-mpegURL application/x-rar-compressed application/x-shockwave-flash application/x-stuffit application/x-tar application/x-web-app-manifest+json application/xhtml+xml application/zip application/x-7z-compressed font/eot font/opentype image/bmp image/svg+xml image/vnd.microsoft.icon image/x-icon;
gzip_min_length 1024;
gzip_comp_level 9;
gzip_buffers 32 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```
