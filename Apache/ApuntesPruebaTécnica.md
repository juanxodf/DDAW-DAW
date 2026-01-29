# Tutorial completo: Apache con Virtual Hosts en Mac

Te voy a explicar paso a paso cómo configurar Apache con virtual hosts en tu Mac, basándome en lo que ya tienes hecho con el profe.

## 📋 Estructura del proyecto

Según la imagen, tu proyecto tiene esta estructura:

```
apache-project/
├── ej1/
├── ej2/
│   └── www/
│       ├── delfines/
│       │   └── index.html
│       └── mundoemo/
│           └── index.html
├── docker-compose.yml
├── httpd-vhosts.conf
├── httpd.conf
└── README.md
```

## 🚀 Paso 1: Crear la estructura de carpetas

Abre la Terminal y navega a tu carpeta del proyecto:

```bash
cd ruta/a/tu/apache-project/ej2
```

Crea las carpetas necesarias si no las tienes:

```bash
mkdir -p www/delfines
mkdir -p www/mundoemo
```

## 📝 Paso 2: Crear los archivos HTML

### Archivo: `www/delfines/index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Delfines</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            text-align: center;
            padding: 50px;
        }
        h1 { font-size: 3em; }
    </style>
</head>
<body>
    <h1>🐬 Bienvenido a Delfines 🐬</h1>
    <p>Este es el sitio web de delfines.com</p>
</body>
</html>
```

### Archivo: `www/mundoemo/index.html`

```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mundo Emo</title>
    <style>
        body {
            font-family: 'Courier New', monospace;
            background: #000;
            color: #ff00ff;
            text-align: center;
            padding: 50px;
        }
        h1 { 
            font-size: 3em;
            text-shadow: 2px 2px 4px #ff00ff;
        }
    </style>
</head>
<body>
    <h1>🖤 Mundo Emo 🖤</h1>
    <p>Bienvenido al mundo oscuro de mundoemo.com</p>
</body>
</html>
```

## ⚙️ Paso 3: Archivo docker-compose.yml

Ya lo tienes creado, pero aquí está completo:

```yaml
services:
  apache:
    image: httpd:2.4
    container_name: en_una_tribu_comanche_jaujaujau_tenia_un_servidor_de_apache
    ports: 
      - "80:80"
    volumes:
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf:ro
      - ./www:/usr/local/apache2/htdocs:ro
      - ./httpd-vhosts.conf:/usr/local/apache2/conf/extra/httpd-vhosts.conf:ro 
    restart: unless-stopped
```

## 🔧 Paso 4: Configurar httpd.conf

Necesitas un archivo `httpd.conf` básico. Crea el archivo:

```bash
touch httpd.conf
```

Contenido mínimo del `httpd.conf`:

```apache
ServerRoot "/usr/local/apache2"
Listen 80

LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule dir_module modules/mod_dir.so
LoadModule mime_module modules/mod_mime.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule unixd_module modules/mod_unixd.so
LoadModule vhost_alias_module modules/mod_vhost_alias.so

<IfModule unixd_module>
    User daemon
    Group daemon
</IfModule>

ServerAdmin admin@localhost
ServerName localhost

<Directory />
    AllowOverride none
    Require all denied
</Directory>

DocumentRoot "/usr/local/apache2/htdocs"

<Directory "/usr/local/apache2/htdocs">
    Options Indexes FollowSymLinks
    AllowOverride None
    Require all granted
</Directory>

<IfModule dir_module>
    DirectoryIndex index.html
</IfModule>

ErrorLog /proc/self/fd/2
LogLevel warn

<IfModule log_config_module>
    LogFormat "%h %l %u %t \"%r\" %>s %b" common
    CustomLog /proc/self/fd/1 common
</IfModule>

<IfModule mime_module>
    TypesConfig conf/mime.types
    AddType application/x-compress .Z
    AddType application/x-gzip .gz .tgz
</IfModule>

# Incluir Virtual Hosts
Include conf/extra/httpd-vhosts.conf
```

## 🌐 Paso 5: Configurar httpd-vhosts.conf

Ya tienes este archivo, según la imagen:

```apache
<VirtualHost *:80>
    ServerName delfines.com
    DocumentRoot "/usr/local/apache2/htdocs/delfines"
    
    <Directory "/usr/local/apache2/htdocs/delfines">
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog /proc/self/fd/2
    CustomLog /proc/self/fd/1 combined
</VirtualHost>

<VirtualHost *:80>
    ServerName mundoemo.com
    DocumentRoot "/usr/local/apache2/htdocs/mundoemo"
    
    <Directory "/usr/local/apache2/htdocs/mundoemo">
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog /proc/self/fd/2
    CustomLog /proc/self/fd/1 combined
</VirtualHost>
```

## 🖥️ Paso 6: Configurar el archivo hosts en Mac

Para que tu Mac reconozca los dominios locales, necesitas editar el archivo `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Agrega estas líneas al final del archivo:

```
127.0.0.1    delfines.com
127.0.0.1    mundoemo.com
```

Guarda con `Ctrl + O`, Enter, y sal con `Ctrl + X`.

## 🐳 Paso 7: Levantar Docker

Primero, asegúrate de que Docker Desktop está corriendo en tu Mac.

Luego, desde la carpeta `ej2`:

```bash
# Levantar los contenedores
docker-compose up -d

# Ver los logs
docker-compose logs -f

# Para detener
docker-compose down
```

## 🧪 Paso 8: Probar los Virtual Hosts

Abre tu navegador y visita:

- **http://delfines.com** → Debería mostrar la página de delfines
- **http://mundoemo.com** → Debería mostrar la página de mundo emo

## 🔍 Solución de problemas comunes

### Si no funciona:

1. **Verificar que Docker está corriendo:**
```bash
docker ps
```

2. **Ver logs del contenedor:**
```bash
docker logs en_una_tribu_comanche_jaujaujau_tenia_un_servidor_de_apache
```

3. **Verificar la sintaxis de Apache:**
```bash
docker exec en_una_tribu_comanche_jaujaujau_tenia_un_servidor_de_apache httpd -t
```

4. **Reiniciar el contenedor:**
```bash
docker-compose restart
```

## 📚 Resumen para el examen

**Conceptos clave:**

1. **Virtual Hosts**: Permiten servir múltiples sitios web desde un mismo servidor Apache
2. **ServerName**: Define el dominio que responde cada virtual host
3. **DocumentRoot**: Carpeta donde están los archivos de cada sitio
4. **archivo hosts**: Mapea dominios a IPs localmente (127.0.0.1)
5. **Puerto 80**: Puerto estándar HTTP

**Flujo de trabajo:**
1. Crear estructura de carpetas
2. Crear archivos HTML
3. Configurar `httpd.conf`
4. Configurar `httpd-vhosts.conf`
5. Editar `/etc/hosts`
6. Levantar con `docker-compose up -d`
7. Probar en el navegador


# Todo lo que debes saber

## ✅ Resumen final para la prueba

### Qué he visto y tocado (por prácticas)

**EJ1 — Apache básico con Docker**
- Docker + Apache 2.4 con `docker-compose.yml` (puerto host 8080 → contenedor 80).
- Volúmenes: `./www` montado en `/usr/local/apache2/htdocs` y `httpd.conf` personalizado.
- `httpd.conf` con configuración base: `ServerName`, `DocumentRoot`, `<Directory>` y `DirectoryIndex`.
- Contenido de prueba: `www/index.html` y `www/test/hola.html` para comprobar rutas.

**EJ2 — Virtual Hosts + Alias**
- `httpd-vhosts.conf` con dos vhosts:
  - `pierdedinero.com` → `/usr/local/apache2/htdocs/apache1`
  - `fckmagisterio.com` → `/usr/local/apache2/htdocs/apache2`
- `httpd.conf` mínimo con `Include conf/extra/httpd-vhosts.conf`.
- Uso de `Alias "/web" "/data"` y su `<Directory "/data">` (para mapear rutas fuera del `DocumentRoot`).
- Opciones en `<Directory>`: `Options -Indexes` y `Options Indexes -FollowSymLinks` (pruebas de listado y symlinks).
- Edición de `/etc/hosts` para resolver dominios locales a `127.0.0.1`.

**EJ3 — DirectoryIndex, Indexes y FollowSymLinks**
- `Options Indexes FollowSymLinks` en `<Directory "/usr/local/apache2/htdocs">`.
- `DirectoryIndex index.html` para definir el archivo por defecto.
- `httpd-vhosts.conf` con:
  - `pierdedinero.com` → `/usr/local/apache2/htdocs/apache1`
  - `fckmagisterio.com` → `/usr/local/apache2/htdocs/apache2`
  - `localhost` → `/usr/local/apache2/htdocs`
- Pruebas de navegación entre carpetas y comportamientos del listado.

**EJ4 — Configuración completa + repaso**
- `httpd.conf` completo (estilo “default” de Apache) con módulos cargados.
- `DocumentRoot`, `<Directory>`, `DirectoryIndex` y opciones de acceso.
- Sitio de prueba en `www/index.html` y ruta extra en `www/test/hola.html`.

### Cosas clave que tengo claras
- **VirtualHost**: permite varios sitios en el mismo Apache según el `ServerName`.
- **DocumentRoot**: carpeta raíz de cada sitio.
- **DirectoryIndex**: archivo que se sirve por defecto.
- **Options**:
  - `Indexes` → lista de archivos al entrar en carpetas sin index.
  - `FollowSymLinks` → permite seguir enlaces simbólicos.
- **Alias**: sirve contenido fuera del `DocumentRoot`.
- **/etc/hosts**: mapea dominios locales a `127.0.0.1`.
- **Docker**: `docker-compose up -d`, logs y reinicios.

### Comandos que he usado
```bash
docker-compose up -d
docker-compose down
docker-compose logs -f
docker ps
docker exec <container> httpd -t
```

### Archivos importantes que he editado
- `Apache/EJ1/docker-compose.yml`
- `Apache/EJ1/httpd.conf`
- `Apache/EJ1/www/index.html`
- `Apache/EJ1/www/test/hola.html`
- `Apache/EJ2/httpd.conf`
- `Apache/EJ2/httpd-vhosts.conf`
- `Apache/EJ3/httpd.conf`
- `Apache/EJ3/httpd-vhosts.conf`
- `Apache/EJ4/httpd.conf`

## 🧠 Apuntes útiles para el examen (enfocados a responder preguntas)

### 1) Qué significa cada directiva (en una línea)
- **ServerName**: nombre con el que Apache identifica el sitio/virtualhost.
- **DocumentRoot**: carpeta raíz desde la que se sirven los archivos.
- **<Directory>**: reglas de acceso y opciones para una carpeta concreta.
- **DirectoryIndex**: archivo que se sirve por defecto en una carpeta.
- **Options Indexes**: si no hay index, muestra listado de archivos.
- **Options FollowSymLinks**: permite seguir enlaces simbólicos.
- **AllowOverride**: permite o bloquea configuración desde `.htaccess`.
- **Alias**: publica una ruta web que apunta a otra carpeta del sistema.
- **VirtualHost**: varios sitios en el mismo servidor según el dominio.

### 2) Diferencias típicas que suelen preguntar
- **DocumentRoot vs Alias**: DocumentRoot es la raíz del sitio; Alias es un “atajo” a otra carpeta.
- **Indexes activado/desactivado**: con `Indexes` se ve el listado; con `-Indexes` no.
- **FollowSymLinks**: si no está, los enlaces simbólicos no se sirven.
- **AllowOverride None vs All**: con `None` no lee `.htaccess`; con `All` sí.

### 3) Qué haces si algo no funciona (checklist rápido)
1. ¿Docker está levantado? → `docker ps`
2. ¿El puerto está bien? → 8080 en host, 80 en contenedor.
3. ¿El fichero correcto se está montando? → revisa `volumes`.
4. ¿Apache carga la config? → `docker exec <container> httpd -t`
5. ¿Te falta el `Include` de vhosts? → `httpd.conf`
6. ¿/etc/hosts tiene los dominios? → `127.0.0.1 dominio.com`

### 4) Respuestas cortas típicas (para oral o test)
- **“¿Para qué sirve VirtualHost?”**  
  Para tener varios sitios/dominios en el mismo Apache.
- **“¿Qué pasa si no hay DirectoryIndex?”**  
  Apache busca el archivo por defecto; si no existe y `Indexes` está activo, lista el directorio.
- **“¿Por qué no veo mi web?”**  
  Suele ser puerto mal, `DocumentRoot` mal o falta en `/etc/hosts`.
- **“¿Cómo servir un archivo fuera del DocumentRoot?”**  
  Con `Alias` y su `<Directory>`.

### 5) Mini‑chuleta de comandos
```bash
docker-compose up -d      # levantar
docker-compose down       # parar
docker-compose logs -f    # ver logs
docker ps                 # ver contenedores
docker exec <c> httpd -t  # validar config
```

### 6) Estructura mínima que debes recordar
```
Apache/
├── EJx/
│   ├── docker-compose.yml
│   ├── httpd.conf
│   ├── httpd-vhosts.conf (si hay virtual hosts)
│   └── www/ (contenido)
```

## 🧩 Guía rápida para resolver ejercicios propuestos

### 1) Pasos base (casi siempre)
1. Crear carpeta del ejercicio y subcarpeta `www/`.
2. Crear `docker-compose.yml` con Apache 2.4 y puerto 8080:80.
3. Crear `httpd.conf` y (si hace falta) `httpd-vhosts.conf`.
4. Crear `index.html` y páginas extra de prueba.
5. Levantar con `docker-compose up -d` y probar en navegador.

### 2) Plantillas que puedes reutilizar

**docker-compose.yml (mínimo)**
```yaml
services:
  apache:
    image: httpd:2.4
    container_name: apache_ejx
    ports:
      - "8080:80"
    volumes:
      - ./www:/usr/local/apache2/htdocs:ro
      - ./httpd.conf:/usr/local/apache2/conf/httpd.conf:ro
    restart: unless-stopped
```

**httpd.conf (mínimo)**
```apache
ServerRoot "/usr/local/apache2"
Listen 80

LoadModule mpm_event_module modules/mod_mpm_event.so
LoadModule authn_core_module modules/mod_authn_core.so
LoadModule authz_core_module modules/mod_authz_core.so
LoadModule authz_host_module modules/mod_authz_host.so
LoadModule dir_module modules/mod_dir.so
LoadModule mime_module modules/mod_mime.so
LoadModule log_config_module modules/mod_log_config.so
LoadModule unixd_module modules/mod_unixd.so
LoadModule alias_module modules/mod_alias.so

ServerName localhost
DocumentRoot "/usr/local/apache2/htdocs"

<Directory "/usr/local/apache2/htdocs">
    Options -Indexes
    AllowOverride None
    Require all granted
</Directory>

DirectoryIndex index.html
TypesConfig conf/mime.types

ErrorLog /proc/self/fd/2
CustomLog /proc/self/fd/1 combined
```

**httpd-vhosts.conf (2 sitios)**
```apache
<VirtualHost *:80>
    ServerName sitio1.com
    DocumentRoot "/usr/local/apache2/htdocs/sitio1"
</VirtualHost>

<VirtualHost *:80>
    ServerName sitio2.com
    DocumentRoot "/usr/local/apache2/htdocs/sitio2"
</VirtualHost>
```

### 3) Ejercicios típicos y cómo responderlos

**Ejercicio: “Crea 2 virtual hosts y muéstralos por dominio”**
1. Crear `www/sitio1` y `www/sitio2` con `index.html`.
2. Definir `httpd-vhosts.conf` con `ServerName` y `DocumentRoot`.
3. Incluir vhosts en `httpd.conf`:
   ```
   Include conf/extra/httpd-vhosts.conf
   ```
4. Editar `/etc/hosts`:
   ```
   127.0.0.1 sitio1.com
   127.0.0.1 sitio2.com
   ```

**Ejercicio: “Desactiva el listado de directorios”**
- En el `<Directory>` correspondiente:
  ```
  Options -Indexes
  ```

**Ejercicio: “Activa listado y sigue enlaces simbólicos”**
- En el `<Directory>`:
  ```
  Options Indexes FollowSymLinks
  ```

**Ejercicio: “Define el index por defecto”**
- Añadir/confirmar:
  ```
  DirectoryIndex index.html
  ```

**Ejercicio: “Crea un alias /web apuntando a /data”**
1. En `httpd.conf`:
   ```
   Alias "/web" "/data"
   <Directory "/data">
       Options None
       Require all granted
   </Directory>
   ```
2. Crear carpeta `/data` en el contenedor (o montar un volumen).

**Ejercicio: “Permite .htaccess”**
- Cambiar en `<Directory>`:
  ```
  AllowOverride All
  ```

### 4) Qué escribir si te piden “explica” (mini‑respuesta)
- “Uso `VirtualHost` para servir varios dominios, cada uno con su `DocumentRoot`.”
- “Con `Options Indexes` habilito el listado; con `-Indexes` lo bloqueo.”
- “`Alias` publica una ruta web a otra carpeta fuera del DocumentRoot.”

### 5) Verificación rápida (antes de entregar)
- `httpd -t` sin errores.
- Navegar a `http://dominio.com:8080` o `http://localhost:8080`.
- Si no se resuelve dominio → revisar `/etc/hosts`.
