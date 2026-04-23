# Escalamiento Horizontal y Vertical con Docker Compose

Este proyecto demuestra una aplicacion Node.js escalable horizontalmente usando Docker Compose y Nginx como balanceador de carga, junto con configuracion de recursos para escalamiento vertical del proxy.

## Requisitos Previos
- [Docker](https://www.docker.com/get-started)
- [Docker Compose](https://docs.docker.com/compose/)

## Ejecución Local con Docker Compose

1. **Construir y levantar los servicios:**
   ```sh
   docker compose up --build -d --scale web=3
   ```
   Esto construira la imagen y levantara Nginx junto con 3 instancias del servicio `web`.

2. **Escalar el servicio a 3 instancias:**
   El archivo `compose.yaml` define `deploy.replicas: 3` para `web`. En Docker Compose local, se recomienda usar `--scale` para asegurar el numero de instancias.
   Si deseas cambiar el numero de instancias, usa:
   ```sh
   docker compose up --build -d --scale web=5
   ```

3. **Acceder al servicio:**
   Nginx expone el puerto `3000` del host hacia el puerto `80` del contenedor. Accede en:
   [http://localhost:3000](http://localhost:3000)

   Cada petición puede ser atendida por un contenedor diferente. Deberías ver una respuesta como:
   ```
   Hola estoy desde <nombre-del-contenedor>!
   ```

4. **Ver los contenedores en ejecución:**
   ```sh
   docker compose ps
   ```

5. **Verificar balanceo horizontal desde terminal:**
   ```sh
   1..10 | ForEach-Object { (curl.exe -s http://localhost:3000) + "`n" }
   ```
   Debes observar hostnames distintos en la salida.

6. **Detener los servicios:**
   ```sh
   docker compose down
   ```

## Escalamiento Vertical en Nginx

El servicio `nginx` incluye recursos en `deploy.resources` para controlar CPU y memoria:

```yaml
deploy:
  resources:
    limits:
      cpus: "1.00"
      memory: 256M
    reservations:
      cpus: "0.50"
      memory: 128M
```

- `limits` define el maximo permitido de recursos.
- `reservations` define la garantia minima de recursos.
- Esta configuracion representa escalamiento vertical del balanceador.

## Notas
- El servicio responde con el nombre del contenedor que atiende la peticion, lo que permite ver el balanceo de carga entre instancias.
- Nginx enruta el trafico HTTP a las instancias del backend.
- Puedes personalizar el numero de replicas usando el flag `--scale`.

## Problemas encontrados y soluciones

### 1) Ruta de acceso incorrecta (8080 vs 3000)

- **Error observado:** al abrir `http://localhost:8080` no cargaba la aplicacion.
- **Causa:** en `compose.yaml` el mapeo real de puertos es `3000:80`, por lo que el puerto publico correcto es `3000`.
- **Solucion aplicada:** se valido el mapeo de puertos en Compose y se accedio a `http://localhost:3000`.

### 2) Pagina en blanco en el navegador al inicio

- **Error observado:** el navegador mostraba una vista vacia/intermitente durante las primeras pruebas.
- **Causa:** la respuesta del servidor Node.js no enviaba de forma explicita el encabezado `Content-Type`.
- **Solucion aplicada:** se agrego en `src/server.js` el encabezado `Content-Type: text/plain; charset=utf-8` antes de `res.end(...)`.
