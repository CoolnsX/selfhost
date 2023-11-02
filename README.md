# selfhost

This repo contains the docker compose of each and every services that is self-hosted by me and already tuned to my preferences

### Nextcloud and yacht together

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

  yacht:
    container_name: yacht
    restart: unless-stopped
    ports:
      - 8000:8000
    volumes:
      - yacht:/config
      - /var/run/docker.sock:/var/run/docker.sock
    image: selfhostedpro/yacht:latest

volumes:
  nextcloud_aio_mastercontainer:
    name: nextcloud_aio_mastercontainer
  yacht:
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
      - "extra_params=--o:ssl.enable=false --o:ssl.termination=true"
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

### prowlarr - cause yacht default json doesn't have it :(

```yaml
services:
  prowlarr:
    image: lscr.io/linuxserver/prowlarr:latest
    container_name: prowlarr
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Kolkata
    volumes:
      - ./config:/config
    ports:
      - 9696:9696
    restart: unless-stopped
```

## Now nginx config for each and every services I make public to make them more optimized by enabling gzip compression

### nextcloud

```nginx
client_body_buffer_size 100m;
proxy_read_timeout 86400s;
client_max_body_size 0;
output_buffers 4 64k;
proxy_hide_header Upgrade;
proxy_hide_header X-Powered-By;
add_header Content-Security-Policy "upgrade-insecure-requests";
add_header X-Frame-Options "SAMEORIGIN";
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Cache-Control "no-transform" always;
add_header Referrer-Policy no-referrer always;
add_header X-Robots-Tag none;
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml ;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```

### collabora

```nginx
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml ;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```


### jellyfin

```nginx
client_max_body_size 20m;
output_buffers 4 64k;
proxy_hide_header Upgrade;
proxy_hide_header X-Powered-By;
add_header Content-Security-Policy "upgrade-insecure-requests";
add_header X-Frame-Options "SAMEORIGIN";
add_header X-XSS-Protection "1; mode=block" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Cache-Control "no-transform" always;
add_header Referrer-Policy no-referrer always;
add_header X-Robots-Tag none;
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml ;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```

### yacht

```nginx
client_max_body_size 20m;
output_buffers 4 64k;
gzip on;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml ;
gzip_min_length 1024;
gzip_comp_level 6;
gzip_buffers 16 8k;
gzip_proxied no-cache no-store private expired auth;
gunzip on;
gzip_static on;
```
