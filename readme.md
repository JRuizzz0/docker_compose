# Arquitectura Docker Compose: frontend, backend y PostgreSQL


## Preguntas y respuestas

### 1. ¿Por qué el frontend no necesita conectarse a PostgreSQL?
Porque solo habla con el backend. El frontend consume la API y es el backend
quien accede a los datos. Así la base de datos no queda expuesta a la capa más
cercana al usuario.

### 2. ¿Por qué el backend pertenece a dos redes?
Porque actúa de puente: está en la red del frontend (para recibir sus
peticiones) y en la de la base de datos (para consultarla). Es el único
servicio que necesita ver ambos lados.

### 3. ¿Qué servicio necesita publicar un puerto en el anfitrión?
El **frontend** (y, según el diseño, el backend si se accede a la API
directamente desde fuera). **PostgreSQL no debe publicar ningún puerto.**

### 4. ¿Por qué publicar PostgreSQL hacia fuera no es la solución adecuada para un fallo de comunicación interno?
Porque un fallo interno se arregla dentro de la red de Docker (nombres, redes,
credenciales, orden de arranque), no abriendo puertas. Publicar el 5432 no
corrige nada y expone la base de datos a la red del anfitrión o a internet,
aumentando la superficie de ataque.

### 5. ¿Qué ventaja ofrece Docker Compose respecto a ejecutar los tres contenedores manualmente?
Todo se define de forma declarativa en un solo archivo (`compose.yaml`):
servicios, redes, volúmenes, variables y dependencias. Se levanta y se tumba
todo con un comando, es reproducible y versionable, y Compose crea las redes y
la resolución de nombres automáticamente, en lugar de lanzar tres `docker run`
largos y propensos a errores.

### 6. ¿Por qué hemos dividido la arquitectura en dos redes?
Para aislar y aplicar el principio de mínimo privilegio. El frontend no puede
alcanzar la base de datos aunque se vea comprometido; solo el backend tiene
acceso a ella.

### 7. ¿Por qué el backend utiliza `db` como dirección de PostgreSQL?
Porque Docker incorpora un DNS interno que resuelve el nombre del servicio
(`db`) a la IP actual del contenedor dentro de la red compartida. Las IP pueden
cambiar; el nombre no.

### 8. ¿Qué diferencia existe entre detener un proyecto y eliminar sus volúmenes?
| Comando | Efecto |
|---|---|
| `docker compose down` | Elimina contenedores y redes, **conserva los volúmenes**: los datos siguen ahí. |
| `docker compose down -v` | Borra además los volúmenes: los datos persistentes se pierden **de forma irreversible**. |

### 9. ¿Qué consecuencias tendría para la empresa perder el volumen de PostgreSQL?
- Pérdida de datos de negocio: clientes, pedidos, historial, etc.
- Interrupción del servicio.
- Coste de recuperación (si hay copias de seguridad) o pérdida definitiva (si no las hay).
- Posibles problemas legales (RGPD) y daño reputacional.

De ahí la importancia de mantener **copias de seguridad** periódicas y probadas.