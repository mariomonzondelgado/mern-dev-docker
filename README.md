# mern-dev-docker

Vamos a crear un contenedor base con Node.js. Luego, dentro de este contenedor, podrás instalar las dependencias específicas de tus proyectos MERN (MongoDB, Express, React, Node, npm, y las librerías de JavaScript y TypeScript que necesites). Este enfoque te da flexibilidad para tener diferentes proyectos con diferentes versiones de dependencias si lo necesitas.

**Paso 1: Crea un Directorio para tu Proyecto Docker**

Primero, crea un directorio donde guardarás los archivos relacionados con tu configuración de Docker. Puedes llamarlo como quieras, por ejemplo: `mern-dev-docker`.

```bash
mkdir mern-dev-docker
cd mern-dev-docker
```

**Paso 2: Crea el Dockerfile**

Dentro del directorio `mern-dev-docker`, crea un archivo llamado `Dockerfile` (sin extensión). Abre este archivo con tu editor de texto y pega el siguiente contenido:

```dockerfile
# Especifica la imagen base de Node.js. Elegimos una versión LTS (Long Term Support)
FROM node:20-alpine

# Establece el directorio de trabajo dentro del contenedor
WORKDIR /app

# Copia los archivos package.json y package-lock.json (si existe) al directorio de trabajo
COPY package*.json ./

# Instala las dependencias de Node.js
RUN npm install

# Copia el resto de los archivos de la aplicación al directorio de trabajo
COPY . .

# Expone el puerto en el que correrá tu aplicación (por defecto para Express)
EXPOSE 3000

# Comando para iniciar tu aplicación Node.js (ajústalo según tu archivo principal)
CMD [ "npm", "start" ]
```

**Explicación del Dockerfile:**

* `FROM node:20-alpine`: Esta línea indica que vamos a usar la imagen oficial de Node.js versión 20 basada en Alpine Linux. Alpine es una distribución ligera, lo que resulta en imágenes de Docker más pequeñas.
* `WORKDIR /app`: Establece `/app` como el directorio de trabajo dentro del contenedor. Todos los comandos posteriores se ejecutarán dentro de este directorio.
* `COPY package*.json ./`: Copia los archivos `package.json` y `package-lock.json` (si lo tienes) desde tu máquina local al directorio `/app` dentro del contenedor. Esto se hace primero para aprovechar la caché de Docker. Si solo cambian los archivos de la aplicación y no las dependencias, Docker no tendrá que reinstalar todo.
* `RUN npm install`: Ejecuta el comando `npm install` dentro del contenedor para instalar todas las dependencias listadas en `package.json`.
* `COPY . .`: Copia el resto de los archivos de tu proyecto desde tu máquina local al directorio `/app` dentro del contenedor.
* `EXPOSE 3000`: Indica que la aplicación dentro del contenedor escuchará en el puerto 3000. Esto es útil para la documentación y para el enrutamiento cuando ejecutas el contenedor.
* `CMD [ "npm", "start" ]`: Especifica el comando que se ejecutará cuando se inicie el contenedor. Aquí asumimos que tienes un script `start` en tu `package.json` que inicia tu aplicación Node.js (por ejemplo, `node server.js` o `nodemon server.js`).

**Paso 3: Crea un Archivo `.dockerignore` (Opcional pero Recomendado)**

Para evitar copiar archivos innecesarios a tu imagen de Docker (como `node_modules`, archivos de registro, etc.), crea un archivo llamado `.dockerignore` en el mismo directorio que tu `Dockerfile` y añade las siguientes líneas:

```
node_modules
.git
.env
```

**Paso 4: Construye la Imagen de Docker**

Abre tu terminal, navega al directorio `mern-dev-docker` y ejecuta el siguiente comando para construir la imagen de Docker:

```bash
docker build -t mern-dev .
```

* `docker build`: Es el comando para construir una imagen de Docker.
* `-t mern-dev`: Asigna la etiqueta (nombre) `mern-dev` a tu imagen. Puedes elegir el nombre que prefieras.
* `.`: Indica que el contexto de la construcción (los archivos que Docker puede usar) es el directorio actual.

**Paso 5: Ejecuta el Contenedor Docker**

Una vez que la imagen se haya construido exitosamente, puedes ejecutar un contenedor basado en ella con el siguiente comando:

```bash
docker run -p 3000:3000 mern-dev
```

* `docker run`: Es el comando para ejecutar un contenedor a partir de una imagen.
* `-p 3000:3000`: Mapea el puerto 3000 de tu máquina local al puerto 3000 del contenedor. Esto te permitirá acceder a tu aplicación Express desde tu navegador en `http://localhost:3000`.
* `mern-dev`: Especifica la imagen que se utilizará para crear el contenedor.

**Integrando MongoDB**

Para integrar MongoDB en tu entorno de desarrollo Docker, tienes varias opciones:

1.  **Ejecutar un Contenedor de MongoDB Separado:** Esta es la forma más común y recomendada para desarrollo. Puedes usar la imagen oficial de MongoDB desde Docker Hub.

    ```bash
    docker run -d --name mongodb -p 27017:27017 mongo
    ```

    * `-d`: Ejecuta el contenedor en segundo plano (detached).
    * `--name mongodb`: Asigna el nombre `mongodb` al contenedor.
    * `-p 27017:27017`: Mapea el puerto 27017 de tu máquina local al puerto 27017 del contenedor MongoDB (puerto por defecto de MongoDB).
    * `mongo`: Especifica la imagen de MongoDB a usar.

    Luego, tu aplicación Node.js (ejecutándose en el primer contenedor `mern-dev`) podrá conectarse a MongoDB usando `mongodb://localhost:27017` (si las aplicaciones están en la misma red Docker por defecto) o `mongodb://mongodb:27017` si configuras una red Docker personalizada.

2.  **Incluir MongoDB en el Mismo Contenedor (No Recomendado para Desarrollo):** Si bien es posible instalar MongoDB dentro del mismo contenedor que tu aplicación Node.js, no es la práctica recomendada para el desarrollo, ya que dificulta la separación de responsabilidades y la escalabilidad. Sin embargo, para fines de aprendizaje muy básicos, podrías investigar cómo añadir los comandos de instalación de MongoDB a tu `Dockerfile`.

**Consideraciones para el Desarrollo MERN:**

* **Volúmenes para Persistencia de Datos:** Para que los datos de tu base de datos MongoDB persistan incluso si detienes o eliminas el contenedor, considera usar volúmenes de Docker. Al ejecutar el contenedor de MongoDB, puedes montar un volumen a la ubicación donde MongoDB guarda sus datos (`/data/db` dentro del contenedor).

    ```bash
    docker run -d --name mongodb -p 27017:27017 -v mongodb_data:/data/db mongo
    ```

    Aquí, `mongodb_data` es un volumen con nombre. Docker gestionará la ubicación física en tu sistema.

* **Volúmenes para Desarrollo en Vivo (Hot Reloading):** Para que los cambios que realices en el código de tu aplicación React y Node.js se reflejen automáticamente en el contenedor sin necesidad de reconstruir la imagen, puedes usar volúmenes para montar el directorio de tu proyecto local dentro del contenedor.

    Al ejecutar el contenedor `mern-dev`, podrías usar algo como:

    ```bash
    docker run -p 3000:3000 -v $(pwd):/app mern-dev
    ```

    * `-v $(pwd):/app`: Monta el directorio de trabajo actual de tu máquina local (`$(pwd)`) en el directorio `/app` dentro del contenedor.

    Sin embargo, para que la "recarga en caliente" funcione correctamente con React y Node.js (por ejemplo, con `nodemon` para Node.js y los scripts de desarrollo de React), es posible que necesites ajustar los comandos de inicio en tu `package.json` y asegurarte de que estén configurados para escuchar los cambios de archivo dentro del contenedor.

* **Docker Compose:** Para gestionar múltiples contenedores (tu aplicación Node.js, MongoDB, posiblemente un cliente de Redis, etc.) de manera más sencilla, te recomiendo aprender a usar Docker Compose. Docker Compose te permite definir y ejecutar aplicaciones multi-contenedor usando un archivo `docker-compose.yml`.

**Próximos Pasos en tu Aprendizaje:**

1.  **Experimenta con el Dockerfile:** Modifica el `Dockerfile`, añade más comandos, y ve cómo afectan a la imagen y al contenedor.
2.  **Conéctate a MongoDB desde tu Aplicación Node.js:** Escribe código en tu aplicación Express para conectarte al contenedor de MongoDB.
3.  **Crea una Aplicación React:** Añade una interfaz de usuario React a tu proyecto y asegúrate de que se sirva correctamente (posiblemente a través de Node.js en modo de desarrollo o construyendo los estáticos).
4.  **Aprende TypeScript:** Integra TypeScript en tu proyecto, tanto en el backend (Node.js/Express) como en el frontend (React). Asegúrate de configurar los archivos de configuración de TypeScript (`tsconfig.json`).
5.  **Explora Docker Compose:** Define tus servicios (Node.js, MongoDB) en un archivo `docker-compose.yml` para una gestión más fácil.
6.  **Practica con Proyectos:** La mejor manera de aprender Full Stack MERN es construyendo proyectos. Empieza con proyectos pequeños y ve aumentando la complejidad.

¡Este es un excelente punto de partida! No dudes en preguntar si tienes más dudas a medida que avanzas en tu aprendizaje. ¡Mucha suerte con tu aventura Full Stack MERN!