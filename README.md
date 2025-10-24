# Proyecto Flask con Docker y pre-commit

Este proyecto es un ejemplo básico de una aplicación **Flask** que se ejecuta dentro de un contenedor **Docker**.
Además, incluye un entorno de desarrollo con **tests automáticos** y un **hook pre-commit** que comprueba el formato del código antes de hacer commits.

---

## Cómo levantar la aplicación

1. Construye la imagen:

   ```bash
   docker build -t mi-app:1.0 .
   ```

2. Ejecuta el contenedor:

   ```bash
   docker run --rm -p 8000:8000 mi-app:1.0
   ```

3. Abre el navegador en [http://localhost:8000](http://localhost:8000)

---

## Entorno de desarrollo

Para entrar al entorno de desarrollo dentro de Docker:

```bash
docker compose run --rm -T dev bash
```

Desde ahí puedes ejecutar comandos como `pytest` o `black`.

---

## Ejecutar tests

Para comprobar que todo funciona correctamente:

```bash
docker compose run --rm -T dev pytest
```

---

## Formatear el código

Para revisar el formato del código:

```bash
docker compose run --rm -T dev black --check .
```

Si hay errores de formato, puedes arreglarlos automáticamente con:

```bash
docker compose run --rm -T dev black .
```

---

## Pre-commit

Cada vez que hagas un commit, se ejecutará automáticamente un **hook pre-commit**.
Este hook:

* Revisa que el código esté bien formateado con **Black**.
* Ejecuta los **tests** para comprobar que todo funciona.

Si algo falla, el commit se bloquea hasta que se corrija.

