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

![alt text](image.png)

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

![alt text](image-1.png)
============================================================================
#Autor: Carlos Marquez 12:27PM 22May26

#Ahora Crearemos el archivo dynamic.yml, ya que sin el no es posible seguir con HTTPS ya que lanzaba una serie de errores y con esto se evitan este archivo que lleva el siguiente bloque de codigo:

        http:
            routers:
                web:
                rule: "Host(`app.midominio.com`)"
                entryPoints:
                    - websecure
                service: web-service
                tls: {}

                mailhog:
                rule: "Host(`mail.midominio.com`)"
                entryPoints:
                    - websecure
                service: mailhog-service
                tls: {}

                portainer:
                rule: "Host(`portainer.midominio.com`)"
                entryPoints:
                    - websecure
                service: portainer-service
                tls: {}

                dashboard:
                rule: "Host(`dashboard.midominio.com`)"
                entryPoints:
                    - websecure
                service: api@internal
                tls: {}

            services:
                web-service:
                loadBalancer:
                    servers:
                    - url: "http://host.docker.internal:8088"

                mailhog-service:
                loadBalancer:
                    servers:
                    - url: "http://host.docker.internal:8025"

                portainer-service:
                loadBalancer:
                    servers:
                    - url: "http://host.docker.internal:9000"

            tls:
            certificates:
                - certFile: /certs/cert.pem
                keyFile: /certs/key.pem
============================================================================
#Autor: Carlos Marquez 12:34PM 22May26

#Ahora seguimos con la creación del archivo index.html, solo para que se muestre una pagina, esta misma la creamos con un poco de ayuda de la IA y aqui se muestra el codigo:

        <!DOCTYPE html>
        <html lang="es">
        <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>Docker + Traefik</title>
        <style>
            body {
            margin: 0;
            font-family: Arial, Helvetica, sans-serif;
            height: 100vh;
            background: linear-gradient(135deg, #2563eb, #7c3aed);
            display: flex;
            justify-content: center;
            align-items: center;
            color: white;
            }

            .card {
            background: rgba(255, 255, 255, 0.12);
            padding: 40px;
            border-radius: 18px;
            text-align: center;
            box-shadow: 0 12px 30px rgba(0, 0, 0, 0.25);
            backdrop-filter: blur(8px);
            max-width: 700px;
            }

            h1 {
            margin: 0 0 15px 0;
            font-size: 2.2rem;
            }

            p {
            font-size: 1.15rem;
            margin: 8px 0;
            }

            .highlight {
            font-weight: bold;
            color: #bfdbfe;
            }
        </style>
        </head>
        <body>
        <div class="card">
            <h1>🚀 Hola Carlos Márquez</h1>
            <p>Estás dentro de <span class="highlight">Docker Compose</span>.</p>
            <p>Tu entorno local está funcionando correctamente con <span class="highlight">Traefik</span>.</p>
            <p>Arquitectura local con <span class="highlight">HTTPS</span>, subdominios y múltiples servicios ✅</p>
        </div>
        </body>
        </html>
============================================================================
