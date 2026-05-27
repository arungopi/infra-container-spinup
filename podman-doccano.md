```
podman container create --name doccano \
  -e "ADMIN_USERNAME=admin" \
  -e "ADMIN_EMAIL=arungopi@cdac.in" \
  -e "ADMIN_PASSWORD=pass@Word1" \
  -v doccano-db:/data \
  -p 8000:8000 docker.io/doccano/doccano
```
