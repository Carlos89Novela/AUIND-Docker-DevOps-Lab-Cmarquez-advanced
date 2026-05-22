# AUIND-Docker-DevOps-Lab-Cmarquez-advanced

#Autor: Carlos Marquez 12:16PM 22May26

#Con este nuevo repositorio vamos a hacer lo mismo que en el anterior pero lo haremos mas limpio y sin tantas carpetas:

    Estructura:
        
            laboratorio-docker-cmarquez/
            │
            ├─ docker-compose.yml
            ├─ dynamic.yml
            ├─ certs.yml
            ├─ index.html
            └─ certs/
                ├─ cert.pem
                └─ key.pem


    Primero crearemos un carpeta llamada certs y aqui guardaremos nuestros certificados que creamos y quedarian asi:

        cert.pem
        key-pem

    Despues creamos el docker-compose.yml con el siguiente bloque de codigo:

        services:
            traefik:
                image: traefik:v2.10
                container_name: traefik

                command:
                - "--api.dashboard=true"
                - "--api.insecure=false"

                - "--entrypoints.web.address=:80"
                - "--entrypoints.websecure.address=:443"

                # SOLO file provider
                - "--providers.file.filename=/etc/traefik/dynamic.yml"

                - "--log.level=DEBUG"

                ports:
                - "80:80"
                - "443:443"

                volumes:
                - ./dynamic.yml:/etc/traefik/dynamic.yml:ro
                - ./certs:/certs:ro

                networks:
                - proxy

            web:
                image: nginx:latest
                container_name: web

                volumes:
                - ./index.html:/usr/share/nginx/html/index.html:ro

                ports:
                - "8088:80"

                networks:
                - proxy

            mailhog:
                image: mailhog/mailhog
                container_name: mailhog

                ports:
                - "1025:1025"
                - "8025:8025"

                networks:
                - proxy

            portainer:
                image: portainer/portainer-ce:latest
                container_name: portainer

                command: -H unix:///var/run/docker.sock

                volumes:
                - /var/run/docker.sock:/var/run/docker.sock
                - portainer_data:/data

                ports:
                - "9000:9000"

                networks:
                - proxy

            redis:
                image: redis:7-alpine
                container_name: redis

                ports:
                - "6379:6379"

                volumes:
                - redis_data:/data

                networks:
                - proxy

            volumes:
            portainer_data:
            redis_data:

            networks:
            proxy:
                driver: bridge
==========================================================================================================================================


