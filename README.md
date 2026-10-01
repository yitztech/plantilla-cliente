# Plantilla de web de cliente

Repositorio plantilla de **yitztech**. El alta de cada cliente
([infra-ionos-vps](https://github.com/yprevot/infra-ionos-vps), playbook `cliente-alta`, fase `repo`)
crea el repositorio del cliente a partir de este, y la fase `web` lo conecta con Coolify.

Se despliega en modo **imágenes**: GitHub Actions construye y publica en GHCR, y Coolify solo descarga y
arranca. El servidor nunca compila.

```
compose.prod.yml          # producción en Coolify: gateway + web (+ PostgreSQL comentado)
gateway/                  # nginx delante de todo: IP real, cabeceras, /version.json, Umami
web/                      # página "Próximamente": sustituir por la aplicación del cliente
.github/workflows/deploy.yml   # publicar en GHCR → webhook de Coolify → comprobar el SHA servido
```

## Reglas de la plataforma

- El dominio va **solo** al servicio `gateway`, en el puerto 80. El resto de servicios no publica puertos.
- **Todo servicio lleva `mem_limit`**: varios clientes comparten servidor. El alta se detiene si falta.
- La configuración va dentro de las imágenes; el compose no monta archivos del repositorio.
- `/version.json` devuelve el commit desplegado: el workflow espera a verlo para dar el despliegue por bueno.

## Después de crear el repo

1. **Paquetes de GHCR**: el primer push a `main` publica `<repo>-gateway` y `<repo>-web`. GitHub los crea
   **privados** aunque el repo sea público. Con repo público, hacerlos públicos (organización → Packages →
   paquete → Package settings → Change visibility). Con repo privado, el alta hace `docker login` en el servidor.
2. **`TRUSTED_PROXY_CIDR`**: tras el primer arranque, medir la subred de Traefik en la red de la app y ponerla en
   la ficha del cliente (infra-ionos-vps, `docs/OPERACION.md` §4.3).
3. **La aplicación**: sustituir `web/` por la aplicación real. Si escucha en otro puerto o hay más servicios
   (API, panel), ajustar los `upstream` y las `location` de `gateway/default.conf.template`. Las variables que
   pone el alta (`SMTP_*`, `MAIL_FROM`, `LISTMONK_*`, `UMAMI_*`, `SITE_URL`) se añaden al `environment` del
   servicio que las use. Los secretos (`POSTGRES_PASSWORD`, claves de la app) se declaran en la ficha con
   `cliente_web_env_secretos` y el alta los genera.

## Probar en local

```bash
docker build -f gateway/Dockerfile -t prueba-gateway .
docker build -f web/Dockerfile -t prueba-web .
docker network create prueba
docker run -d --rm --name web --network prueba prueba-web
docker run --rm -p 8080:80 --network prueba -e TRUSTED_PROXY_CIDR=127.0.0.1/32 prueba-gateway
# http://localhost:8080
```
