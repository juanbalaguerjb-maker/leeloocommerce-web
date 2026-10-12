# LeeLoo Commerce — sitio web (leeloocommerce.com)

Sitio estático (`index.html` + páginas de políticas), servido con nginx en un contenedor Docker.
Migrado desde el hosting compartido de Spaceship al VPS de Contabo (Coolify).

## Archivos

| Archivo | Para qué |
|---|---|
| `index.html` | Página principal |
| `terms.html`, `refund-policy.html`, `privacy.html` | Políticas (las pide Stripe) |
| `Dockerfile` | Imagen nginx que sirve el sitio |
| `nginx.conf` | Compresión gzip y cabeceras de seguridad básicas |

## Probar en local

```bash
docker build -t leeloocommerce-web .
docker run --rm -p 8080:80 leeloocommerce-web
# abrir http://localhost:8080
```

## Desplegar en Coolify

1. **+ New → Application** y elegir el repositorio `juanbalaguerjb-maker/leeloocommerce-web`
   (si el repo es privado, a través de la GitHub App de Coolify).
2. Branch: `main`. Build Pack: **Dockerfile**. Puerto expuesto: **80**.
3. Domains: `https://leeloocommerce.com,https://www.leeloocommerce.com`
4. **Deploy**. Coolify (Traefik) genera el certificado SSL cuando el DNS ya apunta al VPS.

Cada push a `main` se puede redesplegar automáticamente activando *Auto Deploy*.

## Cambio de DNS (en Spaceship)

1. Un día antes: bajar el TTL de los registros `A` a 300 segundos.
2. Cambiar **solo** estos registros para que apunten a la IP del VPS:
   - `A`  `@`   → IP del VPS
   - `A`  `www` → IP del VPS (o `CNAME www → leeloocommerce.com`)
3. **No tocar** los registros `MX` (`mx1.spacemail.com`, `mx2.spacemail.com`) ni los `TXT`
   de SPF/DKIM/DMARC: son del correo en Spacemail, que sigue igual.
4. Verificar que el sitio abre por HTTPS y que el correo sigue llegando.
5. Recién después de una o dos semanas sin problemas, cancelar el **hosting** de Spaceship
   (no Spacemail, no el dominio).
