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
