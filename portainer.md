```yaml
services:
  portainer:
    container_name: portainer
    image: portainer/portainer-ce:sts
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data
    ports:
      - "127.0.0.1:19443:9443"
      - "127.0.0.1:8000:8000"  # Remove if you do not intend to use Edge Agents

volumes:
  portainer_data:
    name: portainer_data

networks:
  default:
    name: portainer_network
```

start : `docker compose -f portainer-compose.yaml up -d`  
remove : `docker compose -f portainer-compose.yaml down --volumes --rmi all --remove-orphans`
