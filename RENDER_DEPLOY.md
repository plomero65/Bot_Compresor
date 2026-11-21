# Despliegue como Web Service en Render

Este proyecto está configurado como **Web Service con Docker**. El servidor HTTP escucha en `0.0.0.0:$PORT`, que es el puerto asignado por Render, mientras Pyrogram ejecuta el bot en el hilo principal. El servidor HTTP se mantiene en un hilo independiente.

## 1. Subir a GitHub

Sube el contenido de esta carpeta al repositorio. Deben estar incluidos `Dockerfile`, `render.yaml`, `requirements.txt`, `main.py` y `ThreadOptimizer.py`.

## 2. Crear el Web Service

1. En Render, selecciona **New + → Web Service**.
2. Conecta el repositorio de GitHub.
3. Selecciona **Docker** como entorno si Render no lo detecta automáticamente.
4. Usa la raíz del repositorio como Docker Context y `./Dockerfile` como Dockerfile Path.
5. Render puede leer el `render.yaml` mediante Blueprint; en ese caso selecciona **New → Blueprint** y confirma que el recurso creado sea de tipo `web`.

El Dockerfile arranca el servicio con `python main.py`. No configures un comando de inicio alternativo.

## 3. Variables de entorno

Configura en **Environment**:

- `API_ID`: API ID numérico de https://my.telegram.org
- `API_HASH`: API Hash de https://my.telegram.org
- `BOT_TOKEN`: token obtenido de @BotFather
- `OWNER_ID`: tu ID numérico de Telegram
- `DATABASE_CHANNEL_ID`: opcional; déjalo vacío si no utilizas el canal de registro
- `DATA_DIR`: `/var/data` si añades un disco persistente; de lo contrario puedes dejar el valor predeterminado `.`

No subas tokens ni claves al repositorio.

## 4. Health check y puerto

Render comprueba `GET /health`. La aplicación escucha automáticamente en `0.0.0.0` usando la variable `PORT` de Render. La ruta `/` muestra una página sencilla de estado.

## 5. Persistencia

El sistema de archivos de un Web Service puede ser efímero. Si quieres conservar `admins.json`, `channels.json` y `user_languages.json` entre despliegues, añade un **Persistent Disk** con punto de montaje `/var/data` y establece `DATA_DIR=/var/data`. Los archivos temporales de descarga y compresión no deben considerarse almacenamiento permanente.

## 6. FFmpeg

El `Dockerfile` instala FFmpeg en la imagen. El código usa `imageio-ffmpeg` para localizar el ejecutable incluido por esa dependencia; FFmpeg del sistema también queda disponible.

## Notas

- El bot y el servidor HTTP corren dentro del mismo servicio. Si el proceso principal se detiene, ambos se detienen.
- La disponibilidad, memoria, CPU y límites de transferencia dependen del plan de Render y de las restricciones de Telegram.
- Un Web Service gratuito puede suspenderse por inactividad según las condiciones de Render; no se debe asumir disponibilidad permanente.
