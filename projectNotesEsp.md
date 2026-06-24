# Izarbide - Notas de Arquitectura y Diseño

## Objetivo

Construir una plataforma de gestión de activos georreferenciados basada en el concepto de gemelo digital capaz de escalar desde pequeños inventarios urbanos hasta grandes infraestructuras distribuidas.

La arquitectura debe ser suficientemente flexible para soportar múltiples dominios:

* Entornos urbanos.
* Infraestructuras aeroportuarias.
* Entornos naturales.
* Infraestructuras industriales.
* Futuros sectores aún no definidos.

---

# Stack tecnológico

## Base de datos

### PostgreSQL + PostGIS

La base de datos principal será PostgreSQL utilizando la extensión PostGIS.

Motivos:

* Excelente soporte geoespacial.
* Escalable.
* Open Source.
* Compatible con GIS profesionales.
* Consultas espaciales optimizadas.
* Capacidad para realizar búsquedas geográficas complejas.

Ejemplos:

* Activos dentro del área visible del mapa.
* Activos cercanos a una ubicación.
* Activos dentro de una zona definida.
* Cálculo de distancias.

---

## Backend

### Opción elegida: Python + Flask

Ventajas:

* Ligero.
* Muy flexible.
* Fácil integración futura con IA y Computer Vision.
* Ecosistema científico enorme.
* Curva de desarrollo rápida.

Alternativa considerada:

### FastAPI

Ventajas frente a Flask:

* Mejor rendimiento.
* Validación automática.
* Documentación Swagger integrada.
* Tipado moderno.
* Muy utilizada en APIs modernas.

Posible decisión:

Si se empieza desde cero, FastAPI puede ser una mejor opción que Flask.

---

## Frontend

### React + TypeScript

Mantener la misma aproximación que proyectos anteriores.

Motivos:

* Ecosistema enorme.
* Buen soporte para mapas.
* Componentes reutilizables.
* Facilidad de integración con APIs.

---

## Almacenamiento de imágenes

### No utilizar AWS inicialmente

Para un proyecto de portfolio:

* Coste innecesario.
* Mayor complejidad.

Alternativas:

#### Opción 1

Guardar imágenes localmente:

```text
uploads/
 ├─ assets/
 ├─ defects/
 └─ tasks/
```

Base de datos:

```sql
image_path VARCHAR
```

#### Opción 2 (futuro)

MinIO

Ventajas:

* Compatible con S3.
* Gratuito.
* Puede migrarse a AWS posteriormente sin apenas cambios.

Posible elección futura:

Docker + MinIO.

---

# Modelo conceptual

## Project

Representa una instalación o cliente.

Ejemplos:

* Zarautz Urban Assets.
* Bilbao Airport.
* Forest Monitoring.

Un usuario puede acceder a uno o varios proyectos.

---

## AssetGroup

Agrupación lógica de activos.

Ejemplos:

* Urban Infrastructure.
* Airport Infrastructure.
* Natural Environment.

---

## AssetType

Define el tipo concreto de activo.

Ejemplos:

* Traffic Sign.
* Street Light.
* Bench.
* Tree.
* Nest.
* Runway Marker.

---

## Asset

Elemento físico geolocalizado.

Campos:

* id
* project_id
* asset_type_id
* geometry
* status
* installation_date
* metadata

---

## Defect

Problemas asociados a activos.

Un activo puede tener múltiples defectos.

Campos:

* id
* asset_id
* severity
* description
* status

---

## Task

Acciones asociadas a defectos.

Campos:

* id
* defect_id
* assigned_user
* due_date
* priority
* status

---

# Gestión de iconos

No almacenar iconos en Base64.

Guardar únicamente:

```text
icons/
 ├─ traffic-sign.svg
 ├─ street-light.svg
 ├─ tree.svg
 └─ runway-marker.svg
```

Base de datos:

```sql
icon_path
```

Ventajas:

* Menos peso.
* Más rápido.
* Fácil mantenimiento.

---

# Problema detectado en la versión anterior

## Situación

Existían aproximadamente 5000 activos.

Todos los activos se cargaban inicialmente en el mapa.

Consecuencias:

* Renderizado lento.
* Consumo elevado de memoria.
* Mala experiencia de usuario.

---

# Nueva estrategia de visualización

La solución NO debe consistir en ocultar activos manualmente.

Debe basarse en la zona visible del mapa.

---

## Estrategia 1 - Bounding Box

Cuando el usuario mueve el mapa:

Frontend:

```text
Obtiene límites visibles
↓
Envía coordenadas
↓
Backend
↓
Devuelve únicamente activos visibles
```

Ejemplo:

```http
GET /assets?bbox=minLng,minLat,maxLng,maxLat
```

Resultado:

Solo se cargan los activos presentes en pantalla.

---

## Estrategia 2 - Nivel de zoom

Dependiendo del zoom:

### Zoom bajo

Mostrar únicamente:

* Contadores.
* Clusters.

### Zoom medio

Mostrar grupos.

### Zoom alto

Mostrar activos individuales.

---

## Estrategia 3 - Clustering

Agrupar marcadores cercanos.

Ejemplo:

```text
[245]
```

al acercarse:

```text
[15] [32] [21]
```

al acercarse más:

```text
📍 📍 📍 📍 📍
```

Esto elimina gran parte de los problemas de rendimiento.

---

## Estrategia 4 - Filtros dinámicos

Panel lateral:

Project
↓
Asset Group
↓
Asset Type
↓
Status

Solo entonces se solicitan los activos.

No cargar nunca el inventario completo.

---

# Casos de prueba iniciales

## Caso 1

Zarautz

Objetivo:

* Señales.
* Farolas.
* Bancos.
* Papeleras.

---

## Caso 2

Bilbao Airport

Objetivo:

* Señalización.
* Balizamiento.
* Elementos de pista.

---

## Caso 3

Bosque / Reserva natural

Objetivo:

* Árboles.
* Nidos.
* Puntos biológicos.
* Seguimiento ambiental.

---

# Integración futura con IA

Servicio independiente.

Arquitectura:

```text
Vehicle Camera
       ↓
Detection Service
       ↓
PostgreSQL
       ↓
Izarbide API
       ↓
Frontend
```

El sistema de visión artificial nunca debe formar parte del backend principal.

Debe ser un microservicio independiente que alimente la base de datos.

---

# Principios de diseño

1. No cargar nunca todos los activos.
2. Todo debe ser filtrable.
3. Todo debe ser multi-proyecto.
4. Todo debe ser geoespacial.
5. El sistema de IA debe ser independiente.
6. Las imágenes deben desacoplarse de la base de datos.
7. El frontend solo debe solicitar lo que necesita mostrar.
