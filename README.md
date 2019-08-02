# Portainer - Docker monitoring

### Create file `stack.yml`

``` yml
version: '3.5'

services:
  agent:
    image: portainer/agent
    environment:
      AGENT_CLUSTER_ADDR: tasks.agent
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - portainer-net
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]
  portainer:
    image: portainer/portainer
    command: -H tcp://tasks.agent.portainer-net:9001 --tlsskipverify
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    networks:
      - portainer-net
    ports:
      - target: 9000
        published: 9000
        protocol: tcp
        mode: host
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
  alpine:
    image: alpine:3.8
    command: tail -f /dev/null
    networks:
      - portainer-net
    deploy:
      mode: global

networks:
  portainer-net:
   attachable: true
   name: portainer-net

volumes:
  portainer_data:
```

Run

``` console
$ docker stack deploy -c stack.yml portainer
```

and visit

[`http://127.0.0.1:9000`](http://127.0.0.1:9000)