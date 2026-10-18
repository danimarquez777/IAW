# IAW

# Práctica 1: Servidores, Proxies y Certificados (NimbusDocs)

## I. Diseño y Requisitos

### 1. Dominios elegidos
Se han seleccionado dos dominios ficticios representativos de las dos marcas de la organización:
* **Marca Pública:** `nimbus-tech.local`
  * **Justificación:** Dominio destinado a la documentación técnica general orientada al público y usuarios externos.
* **Marca Interna:** `nimbus-legal.local`
  * **Justificación:** Dominio reservado para la documentación legal e interna de la empresa.

---

### 2. Elección y justificación del MPM de Apache
* **MPM Seleccionado:** mpm_event
* **Justificación:** 
  * mpm_event es la evolución del modelo de hilos, optimizado para gestionar eficientemente conexiones en estado suspension.
  * Es la opción estándar más eficiente cuando Apache se coloca como servidor web de backend detrás de un proxy inverso como Nginx.

---

## II. Fase 1 - Backend Apache Multi-Marca

### Estructura de Virtual Hosts
* **`nimbus-tech.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-tech`.
* **`nimbus-legal.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-legal`.
  * **Protección de recursos:** Se ha desactivado el listado explícito de directorios y se ha restringido el acceso web directo al fichero de configuración sensible `config-secreto.json`.

### Comportamiento por defecto
* **Archivo de configuración:** `000-default-custom.conf`
* **Explicación:** Apache evalúa las peticiones entrantes comparando la cabecera `Host` con el ServerName de los Virtual Hosts activos. Si la petición utiliza una IP o un dominio no registrado, cae en el primer Virtual Host cargado alfabéticamente, asegurando así un comportamiento controlado que no expone accidentalmente ninguna de las dos marcas.


## III. Fase 2 — Proxy Inverso Nginx, TLS y Seguridad

### 1. Terminación TLS y Tipo de Certificado
- **Tipo de Clave y Curva:** Se han generado certificados SSL/TLS autofirmados utilizando criptografía de curva elíptica.

- **Justificación:** A diferencia de las claves RSA tradicionales, las claves ECDSA ofrecen un nivel equivalente de seguridad con un tamaño de clave significativamente menor. Esto reduce de forma drástica el uso de CPU en la negociación y agiliza el intercambio de claves en peticiones HTTPS de alta concurrencia.

### 2. Configuración de Nginx (`/etc/nginx/sites-available/nimbus-proxy.conf`)
Nginx actúa como punto de entrada único gestionando el tráfico en el puerto 80 (HTTP) y 443 (HTTPS), y reenviando las peticiones válidas al backend Apache en el puerto `8080`.

## 3. Redirección HTTP a HTTPS
**Justificación de la Redirección 301:** Se utiliza el estado 301 Moved Permanently en lugar de un 302 Found porque es una redirección definitiva de la arquitectura. Informa a los navegadores y clientes de que todo el tráfico debe reescribirse a HTTPS.

## 5. Negociación TLS Real

**Salida de la negociación:** 

CONNECTED(00000003)
depth=0 CN = nimbus-tech.local
verify error:num=18:self-signed certificate
verify return:1
depth=0 CN = nimbus-tech.local
verify return:1
---
Certificate chain
 0 s:CN = nimbus-tech.local
   i:CN = nimbus-tech.local
   a:PKEY: id-ecPublicKey, 384 (bit); sigalg: ecdsa-with-SHA256
   v:NotBefore: Oct 18 16:45:18 2026 GMT; NotAfter: Oct 18 16:45:18 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIByjCCAVCgAwIBAgIURCzPW+L3nXoX0mJLOBeORINR61AwCgYIKoZIzj0EAwIw
HDEaMBgGA1UEAwwRbmltYnVzLXRlY2gubG9jYWwwHhcNMjYxMDE4MTY0NTE4WhcN
MjcxMDE4MTY0NTE4WjAcMRowGAYDVQQDDBFuaW1idXMtdGVjaC5sb2NhbDB2MBAG
ByqGSM49AgEGBSuBBAAiA2IABCpgcpxmITF9WG+2z08oP23s3cbxS5s7Ou4IFz7z
ceolbFE0Eh+UM67KIHYHx1exEL7Esh1n9s+dCdBLT5QqoQyGFoOrm/R9McBJjkM1
60Eby5ONBYbC4lxKPua0LBiiSKNTMFEwHQYDVR0OBBYEFExgv6UliKgEfMvOC+OT
5RHBqXmqMB8GA1UdIwQYMBaAFExgv6UliKgEfMvOC+OT5RHBqXmqMA8GA1UdEwEB
/wQFMAMBAf8wCgYIKoZIzj0EAwIDaAAwZQIwCX2aoNKDg4FkvZ0osAevybb3KS99
9fu1LRct312Wvc08+Bj70135nxEg3p1IPyObAjEAgby9Cr8n0/v9UzNP29ZVpUh+
3tetM2IAL9+0LyGQFcBNfUztm3rklb0XHxqXivN5
-----END CERTIFICATE-----
subject=CN = nimbus-tech.local
issuer=CN = nimbus-tech.local
---
No client certificate CA names sent
Peer signing digest: SHA384
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 869 bytes and written 399 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 384 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)

## IV. Fase 3 — Diagnóstico Provocado

**Problema observado:** Apache: Al acceder a https://nimbus-tech.local, se muestra la página genérica Apache2 Ubuntu Default Page en lugar del contenido de la marca.

**Hipótesis inicial:** El proxy Nginx no está reenviando correctamente la cabecera Host original.

**Comando usado:** sudo apache2ctl -S

**Solucion:** Deshabilitar la configuración por defecto ejecutando sudo a2dissite 000-default.conf y recargando Apache.

---- ---- ---- 

**Problema observado:** Nginx: Al simular una ráfaga masiva de peticiones con curl, todas devuelven 200 OK sin aplicar bloqueo por Rate Limiting.

**Hipótesis inicial:** El número de peticiones enviadas en el bucle no es suficiente o la IP local está exenta del límite.

**Comando usado:** sudo nginx -T

**Solucion:** Añadir limit_req zone=nimbus_limit burst=6 nodelay; dentro del bloque location / y reiniciar Nginx.
