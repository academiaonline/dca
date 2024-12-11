1. https://labs.play-with-docker.com/
```
docker swarm init --advertise-addr $( hostname -i )
docker swarm join-token worker
docker swarm join-token manager
docker node ls
```
```
mkdir php/
tee php/index.php 0<<EOF
<?php phpinfo();?>
EOF
docker config create php-config php/index.php
docker service create --config source=php-config,target=index.php --entrypoint php --name php-svc-3 --publish 8080 docker.io/library/php:alpine -f index.php -S 0.0.0.0:8080
docker secret create php-secret php/index.php
docker service create --secret source=php-secret,target=index.php --entrypoint php --name php-svc-4 --publish 8080 docker.io/library/php:alpine -f index.php -S 0.0.0.0:8080
docker service create --secret source=php-secret,target=/index.php --entrypoint php --name php-svc-5 --publish 8080 docker.io/library/php:alpine -f index.php -S 0.0.0.0:8080
```
```
docker node ls
docker service ls
docker service ps php-svc-4
docker service logs php-svc-4
docker ps
docker exec php-svc-4.1.lx3fu84r3mble8snpzbz3fx7k df
```
```
docker service update --secret-rm php-secret php-svc-4
docker service ps php-svc-4
docker ps
docker exec php-svc-4.1.vus6clyku7qfsocjffefiwiwr df
docker service update --secret-add source=php-secret,target=/index.php php-svc-4
docker service ps php-svc-4
docker ps
docker exec php-svc-4.1.253x45roucx7v7utczpl14n2s df
```
```
docker service scale php-svc-3=5
docker service ps php-svc-3
docker service ls
docker ps
docker rm --force php-svc-3.1.boqyufqmrk4rc78ga9w176qa7 ; docker ps
docker service ps php-svc-3
docker restart php-svc-3.1.iujff6vlnhphgzga2dsu4mexe 
docker service ps php-svc-3
docker node update --availability drain node4
docker service ps php-svc-3
docker node update --availability active node4
docker service ps php-svc-3
```
1. https://docs.docker.com/compose/compose-file/compose-file-v3/
```
mkdir -p php/
tee php/index.php 0<<EOF
<?php phpinfo();?>
EOF
```
```
tee php/docker-compose.yaml 0<<EOF
configs:
  php-config:
    external: false
    file: index.php
networks:
  php-network:
    external: false
    internal: false
secrets:
  php-secret:
    external: false
    file: index.php
services:
  php-svc:
    command:
      - -f
      - index.php
      - -S
      - 0.0.0.0:8080
    configs:
      - source: php-config
        target: /my_app/index.php
        uid: '65534'
        gid: '65534'
        mode: 0400
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: '0.2'
          memory: 200M
        reservations:
          cpus: '0.2'
          memory: 200M
    entrypoint:
      # docker exec PHPINFO_php-svc.4.z51xlgdlmt1ifpjmjvemh8z1r which php
      - /usr/local/bin/php
    image: docker.io/library/php:alpine@sha256:6340f86b1dc4325d09cd8311d8c40e36ab54061d0d25ea1491c100578bc50ae1
    networks:
      - php-network
    ports:
      - 8080
    read_only: true
    # docker exec PHPINFO_php-svc.4.z51xlgdlmt1ifpjmjvemh8z1r touch sebastian
    secrets:
      - source: php-secret
        target: /my_app/index-secret.php
        uid: '65534'
        gid: '65534'
        mode: 0400
    user: nobody:nogroup
    # docker exec PHPINFO_php-svc.4.z51xlgdlmt1ifpjmjvemh8z1r id
    # docker exec PHPINFO_php-svc.4.z51xlgdlmt1ifpjmjvemh8z1r whoami
    volumes:
      - type: volume
        source: my_volume
        target: /my_app/data/
        read_only: false
    working_dir: /my_app/
version: '3.8'
volumes:
  my_volume:
    external: false
EOF
```
```
docker stack rm PHPINFO ; sleep 20
docker stack deploy --compose-file php/docker-compose.yaml PHPINFO
docker stack ps PHPINFO --no-trunc
docker stack ls
docker stack services PHPINFO
docker service logs PHPINFO_php-svc
docker ps
docker exec PHPINFO_php-svc.4.z51xlgdlmt1ifpjmjvemh8z1r df
```
