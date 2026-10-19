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
  * Es la opción estándar más eficiente cuando Apache se coloca como servidor web de backend detrás de un proxy inverso como Nginx.

---

## II. Fase 1 - Backend Apache Multi-Marca

### Estructura de Virtual Hosts
* **`nimbus-tech.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-tech`.
* **`nimbus-legal.local`**: Configurado con `DocumentRoot` en `/var/www/nimbus-legal`.
  * **Protección de recursos:**

- Desactivación del listado de directorios mediante `Options -Indexes`.
- Desactivación de la posibilidad de modificar la configuración mediante `.htaccess` con `AllowOverride None`.
- Restricción del acceso al archivo `vault-credentials.txt`.
- Configuración de un sitio predeterminado para controlar las peticiones que no corresponden a los dominios configurados.

### Comportamiento por defecto
* **Archivo de configuración:** `000-default.conf`
* **Explicación:** Se ha configurado el archivo `000-default.conf` para denegar las peticiones que no coinciden con los sitios web permitidos.

De esta forma, se evita mostrar accidentalmente el contenido de otro sitio cuando se recibe una petición con un nombre de dominio desconocido.


## III. Fase 2 — Proxy Inverso Nginx, TLS y Seguridad

### 1. Terminación TLS y Tipo de Certificado

Se ha generado un certificado autofirmado mediante OpenSSL.

Los archivos se encuentran en:

- Certificado: `nginx/certs/nimbus.crt`
- Clave privada: `nginx/certs/nimbus.key`

Se ha utilizado una clave RSA de 2048 bits.

**Justificacion:** El certificado permite realizar pruebas HTTPS en el entorno local. Al ser autofirmado, los navegadores pueden mostrar una advertencia de seguridad.

### 2. Configuración de Nginx (`/etc/nginx/sites-available/nimbus-proxy.conf`)

Nginx actúa como punto de entrada de las peticiones externas.

La configuración se encuentra en:

`nginx/conf.d/nimbus.conf`

Las peticiones se reenvían al servicio Apache mediante:

`proxy_pass http://apache:80;`, por el puerto 8080.

También se conservan las cabeceras necesarias para identificar el dominio solicitado y transmitir información sobre la petición original.

## 3. Redirección HTTP a HTTPS

Se ha configurado una redirección permanente mediante el código HTTP `301 Moved Permanently`.

Cuando un usuario accede mediante HTTP, Nginx lo redirige a HTTPS utilizando el puerto publicado `8443`.

Esto permite centralizar el acceso web seguro en el proxy inverso.

## 5. Negociación TLS Real

**Salida de la negociación:** 

CONNECTED(00000003)
depth=0 C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
verify error:num=18:self-signed certificate
verify return:1
depth=0 C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
verify return:1
---
Certificate chain
 0 s:C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
   i:C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Oct 19 17:47:12 2026 GMT; NotAfter: Oct 19 17:47:12 2027 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEAzCCAuugAwIBAgIUR8X6Bfaj6SPanubbP2FH8QuLIVYwDQYJKoZIhvcNAQEL
BQAwbTELMAkGA1UEBhMCRVMxDjAMBgNVBAgMBUNhZGl6MQ4wDAYDVQQHDAVKZXJl
ejETMBEGA1UECgwKTmltYnVzRG9jczEMMAoGA1UECwwDSUFXMRswGQYDVQQDDBJu
aW1idXMtbGVnYWwubG9jYWwwHhcNMjYxMDE5MTc0NzEyWhcNMjcxMDE5MTc0NzEy
WjBtMQswCQYDVQQGEwJFUzEOMAwGA1UECAwFQ2FkaXoxDjAMBgNVBAcMBUplcmV6
MRMwEQYDVQQKDApOaW1idXNEb2NzMQwwCgYDVQQLDANJQVcxGzAZBgNVBAMMEm5p
bWJ1cy1sZWdhbC5sb2NhbDCCASIwDQYJKoZIhvcNAQEBBQADggEPADCCAQoCggEB
AKHSmeEgoziK2Kecv8zYwCAh0sSQtS2RcQopvPnpILfY9Rg9MzcKs8ritynlDxJe
PNZZoSXf+MVZ3rVDt+h/K7Qz5XrrR51dgQS7VetWHhmv/5Fa3pO/AcaG9Z62Nm5y
qFbpLcls4hSVFkUBEWF/o3QkhhSz8vQpijahn8IpMKcMx9JFT6WHVhs8ETOdpIaZ
J8Ws7gvyPf9bHhhsLRK72DODTqzBKUqpVGgfmYGjkSZAwTvWoPm13teL4MsYqaj0
kHLrPMsI2G3JFICSDdAR9rtApDqXhzzlysd6G5YpMesrUgRd56cFHOIg8nv/Iznc
gjNUe/kwC/1ckB4F+/KKjs0CAwEAAaOBmjCBlzAdBgNVHQ4EFgQUFkIQJoOLZwZU
1OwR+n7OQyOBWREwHwYDVR0jBBgwFoAUFkIQJoOLZwZU1OwR+n7OQyOBWREwDwYD
VR0TAQH/BAUwAwEB/zBEBgNVHREEPTA7ghJuaW1idXMtbGVnYWwubG9jYWyCEm5p
bWJ1cy1sZWdhbC5sb2NhbIIRbmltYnVzLXRlY2gubG9jYWwwDQYJKoZIhvcNAQEL
BQADggEBAFjqadDDBatyUK7wra1UCm1iTVuyNLWcjwe+YsjjwH3p20Cyo1/IVTcR
+mChVEO1jlE/ZeOCZfV9/Cqni/BFo9pRHUt7zXIcRMX9xEqZFJ5Lfdm9ozQaJIgO
Y0424yTwAtulRf04xJUyxMN9ESqQ17O2d7B3c8TkN53LFrMjwOpWErBUXVwSLqql
wHoU83IqssiBfpplY+3iFtZ3sAO8hz0B9clV0loClPsUTW2Ur2j4Tba3l6WrH++E
2L0iSUYeivWoKfSqIEmc/9i6z6M/KW7gazKdWT7TX7gzzvjJKKTpdiNEN3lH5OrG
1ZqLr4DFyqT5xpx8nTxhH/8M2FOHdnk=
-----END CERTIFICATE-----
subject=C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
issuer=C = ES, ST = Cadiz, L = Jerez, O = NimbusDocs, OU = IAW, CN = nimbus-legal.local
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: RSA-PSS
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 1613 bytes and written 400 bytes
Verification error: self-signed certificate
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 2048 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 18 (self-signed certificate)

### 6. Cabeceras de seguridad

Se han añadido las siguientes cabeceras HTTP:

- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: SAMEORIGIN`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `X-XSS-Protection: 0`

Estas cabeceras ayudan a controlar determinados comportamientos del navegador y a mejorar la seguridad de las páginas web.


### 7. Limitación de peticiones (Rate Limiting)

Se ha configurado un límite de peticiones en Nginx para reducir el impacto de ráfagas excesivas de solicitudes.

La zona de limitación utiliza la siguiente configuración:

`limit_req_zone $binary_remote_addr zone=nimbus_limit:10m rate=10r/s;`

Dentro del bloque `location /` se utiliza:

`limit_req zone=nimbus_limit burst=20 nodelay;`

La configuración establece una tasa de referencia de 10 peticiones por segundo por dirección IP, con una ráfaga permitida de hasta 20 peticiones adicionales.



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
