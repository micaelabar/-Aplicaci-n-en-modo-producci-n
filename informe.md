#  Aplicación en modo producción.
## 1. Tiempo de duración:
Aproximadamente 3 horas.
## 2. Fundamentos y conocimientos previos
- Conocimientos básicos de Docker y Docker Compose.
- Conceptos de contenedores y virtualización ligera.
- Manejo básico de React para frontend y Node.js/Express para backend.
- Conceptos básicos de redes en Docker (puertos y comunicación entre contenedores).
- Uso básico de PostgreSQL como base de datos.
## 3. Objetivo a alcanzar
- Crear un contenedor para construir la aplicación frontend con Node.js (proceso build).
- Crear un contenedor para servir el frontend en producción usando Nginx con los archivos estáticos generados.
- Configurar el backend y la base de datos en contenedores separados.
- Integrar y orquestar los tres servicios mediante Docker Compose para facilitar el despliegue completo y la comunicación entre ellos.
## 5. Equipo necesario
- Computadora con sistema operativo Windows, macOS o Linux.
- Docker Desktop instalado (incluye Docker Engine y Docker Compose).
- Editor de código (VSCode recomendado).
## 6. Material de apoyo:
- Documentación oficial de Docker: https://docs.docker.com/
- Documentación oficial de React: https://reactjs.org/
- Documentación oficial de Nginx: https://nginx.org/en/docs/
- Documentación de Node.js y Express: https://nodejs.org/ y https://expressjs.com/
- Tutorial básico de PostgreSQL con Docker: https://hub.docker.com/_/postgres
## 7. Procedimiento:
### Paso 1:Preparar el frontend
```
FROM node:18-alpine AS build

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build
````
### Evidencia:
<imag!![1](https://github.com/user-attachments/assets/6c67b17d-09dd-417d-a504-b3ed277338fc)

### Paso 2: Preparar el contenedor Nginx para servir frontend
- En la carpeta nginx, crea un archivo Dockerfile para servir los archivos estáticos generados por React:
```
FROM nginx:stable-alpine

COPY default.conf /etc/nginx/conf.d/default.conf
COPY --from=build-stage /app/build /usr/share/nginx/html
````
### Evidencia:
<imag!![2](https://github.com/user-attachments/assets/4dbea271-c35f-46e9-ab4f-3b7b2af2c830)

- El archivo default.conf puede tener esta configuración básica:
```
server {
    listen 80;
    server_name localhost;

    root /usr/share/nginx/html;
    index index.html index.htm;

    location / {
        try_files $uri /index.html;
    }
}
````
### Evidencia:
<imag!![3](https://github.com/user-attachments/assets/488bd59b-98ea-44d2-a656-8be1647b9ee8)

### Paso 3: Preparar backend
En la carpeta backend crea Dockerfile:
```
FROM node:18-alpine

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000
CMD ["node", "index.js"]
````
### Evidencia:
<imag![4](https://github.com/user-attachments/assets/767b0558-00b3-466e-8267-20e07321eae5)

### Paso 4: Crear archivo docker-compose.yml en la raíz del proyecto
```
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - DB_HOST=db
      - DB_USER=postgres
      - DB_PASSWORD=postgres
      - DB_NAME=mydb
    depends_on:
      - db

  db:
    image: postgres:15-alpine
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  frontend:
    build:
      context: ./frontend
      target: build-stage
    volumes:
      - ./frontend/build:/usr/share/nginx/html
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  postgres-data:
````
### Evidencia:
<imag!![5](https://github.com/user-attachments/assets/d650962c-1e79-4416-aeb4-c668699a8a6d)

### Paso 5: Ejecutar todo
- Desde la raíz del proyecto, correr:
```
docker-compose up --build
````
### Evidencia:
<imag!![6](https://github.com/user-attachments/assets/75893ef5-30c0-429e-9e11-ab1345eab2bf)

- Abrir navegador y entrar a http://localhost:3000 para ver la app React servida desde Nginx.
  ### Evidencia:
<imag!![fron](https://github.com/user-attachments/assets/f96e6235-9f59-4a09-b24e-4ee8f8f2e189)

- Backend disponible en http://localhost:5000.
### Evidencia:
<imag!![backend](https://github.com/user-attachments/assets/d66b7a49-7811-4a1f-b550-7cedc6c62663)

## 8. Resultados esperados:
- Contenedores para frontend, backend y base de datos funcionando y comunicándose correctamente.
- Frontend React en modo producción servido por Nginx.
- Backend Node.js/Express disponible y consumible desde el frontend.
- Persistencia de datos en la base PostgreSQL.
## 9. Conclusión: 
El uso de contenedores Docker permite crear entornos aislados para cada componente de la aplicación, facilitando la construcción, despliegue y escalabilidad. Separar el frontend en una etapa de build y servirlo con Nginx mejora el rendimiento en producción. Docker Compose simplifica la orquestación y el manejo de dependencias entre servicios.
## 10. Bibliografía:
- Docker Docs: https://docs.docker.com/
- React Docs: https://reactjs.org/docs/getting-started.html
- Nginx Docs: https://nginx.org/en/docs/
- Node.js Docs: https://nodejs.org/en/docs/
- PostgreSQL Docker Hub: https://hub.docker.com/_/postgres
  
