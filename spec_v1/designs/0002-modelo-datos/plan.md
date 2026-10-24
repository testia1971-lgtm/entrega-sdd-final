---
title: "Plan de Modelo de Datos MongoDB"
tags: [data-model, mongodb, json-schema, seed]
status: draft
created: 2026-10-21
---

# Plan de Modelo de Datos MongoDB

Este documento detalla el mapeo de las entidades del modelo de datos (`designs/0002-modelo-datos.md`) a colecciones en MongoDB, la definición de índices necesarios para las consultas de la API y los tableros, el `$jsonSchema` para validación de documentos, y el proceso de carga del seed.

## 1. Mapeo Entidad → Colección

| Entidad | Colección MongoDB | Notas |
|---|---|---|
| Persona | `personas` | |
| Rol | `roles` | `GLOBAL` vs `PROYECTO` |
| Proyecto | `proyectos` | |
| Asignacion | `asignaciones` | |
| CargaEstimada | `cargas_estimadas` | |
| Hito | `hitos` | |
| PersonaHistorial | `personas_historial` | Append-only |
| Ausencia | `ausencias` | |
| SaldoVacaciones | `saldos_vacaciones` | |
| Competencia | `competencias` | |
| PersonaCompetencia | `personas_competencias` | N:M |
| Herramienta | `herramientas` | |
| PersonaHerramienta | `personas_herramientas` | N:M |
| ConvenioBecario | `convenios_becario` | 1:1 con Persona |
| Propuesta | `propuestas` | |
| PropuestaItem | `propuestas_items` | |
| BitacoraEvento | `bitacora_eventos` | Audit Log |
| Feriado | `feriados` | |
| CapacidadPersonaPeriodo | `capacidades_persona_periodo` | Materializada |
| Logro | `logros` | Catálogo de logros (gamificación) |
| LogroPersona | `logros_persona` | Logros desbloqueados por persona |
| SolicitudVacaciones | `solicitudes_vacaciones` | Autogestión de vacaciones |

## 2. Definición de Índices

Los índices se definen para optimizar las consultas de los tableros (proyectos y personas 360) y para hacer cumplir invariantes.

### Colección: `personas`
- `id`: `_id` (default)
- `email`: `unique: true`
- `rolContratualBaseId` (para búsquedas de personas por rol)
- `fechaIngreso` (para antigüedad, y ordenación)
- `estado` (para filtrar activas/inactivas)

### Colección: `roles`
- `id`: `_id` (default)
- `nombre`: `unique: true`
- `familia`
- `alcance`, `proyectoId` (compuesto para roles de proyecto)

### Colección: `proyectos`
- `id`: `_id` (default)
- `codigo`: `unique: true`
- `estado` (para filtrar proyectos activos/en pausa/cerrados)
- `fechaInicio`, `fechaFinPrevista` (compuesto para línea de tiempo)

### Colección: `asignaciones`
- `id`: `_id` (default)
- `personaId`, `fechaDesde`, `fechaHasta`: compuesto para consultas de solapamiento y timeline de personas.
- `proyectoId`, `estado`: compuesto para consultas de asignaciones de un proyecto.
- `tipo`, `estado`: compuesto para filtrar por tipo y estado (ej. préstamos activos).
- `personaId`, `proyectoId`, `rolId`: índice único parcial para `estado: "ACTIVA"` (Invariante I-6).

### Coleción: `cargas_estimadas`
- `proyectoId`, `periodoSemanaInicio`, `rolId`: compuesto para las métricas de carga por proyecto y rol.
- `origen`

### Colección: `hitos`
- `proyectoId`, `fecha`: compuesto para timeline del proyecto.
- `fecha` (para timeline general)

### Colección: `personas_historial`
- `personaId`, `fecha`: compuesto para historial 360 de la persona, ordenado cronológicamente.
- `entidad`, `entidadId`: para trazabilidad (qué evento de qué entidad generó el historial).

### Colección: `ausencias`
- `personaId`, `fechaDesde`, `fechaHasta`: compuesto para consultas de solapamiento y timeline de ausencias.
- `estado`

### Colección: `saldos_vacaciones`
- `personaId`, `anio`: único compuesto para el saldo anual de vacaciones.

### Colección: `personas_competencias`
- `personaId`, `competenciaId`: único compuesto.

### Colección: `personas_herramientas`
- `personaId`, `herramientaId`: único compuesto.

### Colección: `convenios_becario`
- `personaId`: único.

### Colección: `propuestas`
- `proyectoId`, `createdAt`: para listar propuestas por proyecto, ordenadas.
- `estado`

### Colección: `propuestas_items`
- `propuestaId`: para obtener los ítems de una propuesta.
- `personaId`, `rolId`, `proyectoId`

### Colección: `bitacora_eventos`
- `occurredAt`: para auditoría temporal.
- `entidad`, `entidadId`: para filtrar eventos de una entidad específica.
- `actorId`

### Colección: `feriados`
- `fecha`, `ambito`: compuesto para obtener feriados por fecha y ámbito.

### Colección: `capacidades_persona_periodo`
- `personaId`, `periodo`: único compuesto.

### Colección: `logros`
- `id`: `_id` (default)
- `codigo`: `unique: true`

### Colección: `logros_persona`
- `personaId`, `logroId`: único compuesto (una persona no puede desbloquear el mismo logro dos veces).
- `personaId`, `fechaDesbloqueo`: compuesto para listar logros cronológicamente.

### Colección: `solicitudes_vacaciones`
- `personaId`, `fechaDesde`, `fechaHasta`: compuesto para consultas de solapamiento.
- `estado`
- `personaId`, `solicitadoEn`: para historial de solicitudes por persona.

## 3. JSON Schema por Colección

Se definirá un `$jsonSchema` para cada colección para garantizar la forma y los tipos de datos en la base de datos. Esto es un primer nivel de validación antes de las reglas de negocio más complejas.

**(Ejemplo para Persona)**

```json
{
  "$jsonSchema": {
    "bsonType": "object",
    "required": ["_id", "nombre", "email", "tipoContrato", "rolContratualBaseId", "nivelExperiencia", "fechaIngreso", "diasLaborablesSemana", "horasDia", "estado"],
    "properties": {
      "_id": {
        "bsonType": "string",
        "description": "UUID de la persona"
      },
      "nombre": {
        "bsonType": "string",
        "description": "Nombre completo de la persona"
      },
      "email": {
        "bsonType": "string",
        "pattern": "^[\w-\.]+@([\w-]+\.)+[\w-]{2,4}$",
        "description": "Email único de la persona"
      },
      "tipoContrato": {
        "bsonType": "string",
        "enum": ["EMPLEADO", "BECARIO", "CONTRATISTA"],
        "description": "Tipo de contrato laboral"
      },
      "rolContratualBaseId": {
        "bsonType": "string",
        "description": "ID del rol contractual base (FK a roles)"
      },
      "nivelExperiencia": {
        "bsonType": "string",
        "enum": ["JUNIOR", "MID", "SENIOR", "LEAD", "PRINCIPAL"],
        "description": "Nivel de experiencia de la persona"
      },
      "fechaIngreso": {
        "bsonType": "date",
        "description": "Fecha de ingreso a la empresa"
      },
      "fechaSalida": {
        "bsonType": ["date", "null"],
        "description": "Fecha de salida de la empresa (null si está activa)"
      },
      "fechaNacimiento": {
        "bsonType": ["date", "null"],
        "description": "Fecha de nacimiento de la persona"
      },
      "diasLaborablesSemana": {
        "bsonType": "double",
        "minimum": 0,
        "maximum": 7,
        "description": "Días laborables por semana (ej. 5.0)"
      },
      "horasDia": {
        "bsonType": "double",
        "minimum": 0,
        "maximum": 24,
        "description": "Horas laborables por día (ej. 8.0)"
      },
      "estado": {
        "bsonType": "string",
        "enum": ["ACTIVA", "INACTIVA"],
        "description": "Estado laboral de la persona"
      },
      "avatarUrl": {
        "bsonType": ["string", "null"],
        "description": "URL del avatar de la persona"
      }
    },
    "additionalProperties": false
  }
}
```
*(Se incluirán esquemas JSON para todas las colecciones principales siguiendo este formato)*

## 4. Carga del Seed

El seed se cargará utilizando un script Python (`scripts/seed.py`) que consumirá los archivos JSON generados.

El proceso será:
1.  **Generación de JSON:** `scripts/seed.py` generará archivos JSON para cada colección (ej. `seed/personas.json`, `seed/proyectos.json`), siguiendo la estructura definida en `docs/DATASET_SEED.md`.
2.  **Carga en MongoDB:** El mismo `scripts/seed.py` contendrá la lógica para conectarse a MongoDB y realizar `upsert`s de los documentos desde los JSON.
    -   **Idempotencia:** El script garantizará que la carga sea idempotente, es decir, que ejecutarlo múltiples veces deje la base de datos en el mismo estado final, idealmente mediante borrado + inserción o uso de `upsert` sobre claves naturales.
    -   **Validación:** Se utilizarán los `$jsonSchema` definidos para validar los documentos antes de la inserción.
3.  **Verificación:** Un script `scripts/verify_seed.py` validará la integridad del seed cargado contra los invariantes I-1..I-8 y la cobertura de los escenarios S-01..S-23, tal como se describe en `docs/DATASET_SEED.md`.

**(Ejemplo de fragmento de seed.py para Persona)**
```python
import json
from pymongo import MongoClient
from bson.objectid import ObjectId # if using ObjectId as primary key

def load_collection(client, db_name, collection_name, json_file_path):
    db = client[db_name]
    collection = db[collection_name]
    
    with open(json_file_path, 'r', encoding='utf-8') as f:
        data
