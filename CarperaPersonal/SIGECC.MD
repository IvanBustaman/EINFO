# Documentación Técnica: Sistema de Gestión de Evaluaciones de Control de Confianza (SIGECC)

**Versión del Modelo:** 4.1 (Final)
**Fecha:** 25 de noviembre de 2025
**Contexto:** Secretaría de Marina - Centro de Control de Confianza

---

## 1. Descripción General
El **SIGECC** es una plataforma centralizada diseñada para gestionar el flujo de solicitudes de evaluación de control de confianza. El sistema permite a los Mandos y Unidades Navales solicitar evaluaciones para su personal, dando seguimiento puntual al estatus de cada elemento, eliminando la duplicidad de registros y asegurando la integridad de la información jerárquica.

---

## 2. Decisiones de Arquitectura y Normalización

Durante el diseño se implementaron las siguientes reglas de negocio para garantizar la robustez del dato:

### A. Gestión de Identidad (Personal vs. Usuarios)
* **Decisión:** Se eliminó la tabla separada de `Usuarios`.
* **Justificación:** Todo actor del sistema (Administrador, Solicitante, Analista) es, a su vez, personal de la institución.
* **Implementación:** La autenticación se realiza directamente contra la tabla `tb_personal` utilizando credenciales (RFC/CURP + Contraseña) y un sistema de permisos granular.

### B. Jerarquía y Rangos (Grado + Complemento)
* **Decisión:** Se implementó una tabla intermedia `tb_cat_rangos_validos`.
* **Justificación:** Evitar combinaciones inexistentes (ej. "Almirante Oficinista").
* **Implementación:** El personal no se enlaza directamente a un grado, sino a una combinación válida predefinida de `Grado + Complemento`.

### C. Sistema de Permisos
* **Decisión:** Relación Muchos a Muchos (N:M).
* **Justificación:** Un solo elemento puede requerir múltiples roles (ej. puede ser "Evaluador" y "Solicitante" simultáneamente).
* **Implementación:** Se utiliza la tabla `tb_personal_permisos` para asignar N permisos a un elemento.

---

## 3. Diccionario de Datos (Esquema Final)

### I. Estructura Organizacional
| Tabla | Descripción | Claves Principales |
| :--- | :--- | :--- |
| **`tb_mandos`** | Catálogo de Regiones Navales y Mandos Superiores. | **PK:** `id_mando` |
| **`tb_unidades`** | Catálogo de unidades operativas (Batallones, Buques, etc.). | **PK:** `id_unidad`<br>**FK:** `fk_id_mando` |

### II. Catálogos de Jerarquía
| Tabla | Descripción | Claves Principales |
| :--- | :--- | :--- |
| **`tb_cat_grados`** | Lista de grados (Marinero, Teniente, Capitán...). | **PK:** `id_grado` |
| **`tb_complemento`** | Lista de especialidades (Cuerpo General, IM, Servicios...). | **PK:** `id_complemento` |
| **`tb_cat_rangos_validos`** | **Tabla pivote** que define qué combinaciones Grado-Complemento existen. | **PK:** `id_rango_valido`<br>**FK:** `fk_id_grado`<br>**FK:** `fk_id_complemento` |

### III. Gestión de Personal (Núcleo)
| Tabla | Descripción | Claves Principales |
| :--- | :--- | :--- |
| **`tb_personal`** | Registro único de personas. Incluye datos biométricos, jerárquicos y de acceso (password). | **PK:** `id_personal`<br>**FK:** `fk_id_unidad_pertenencia`<br>**FK:** `fk_id_rango_valido` |
| **`tb_permisos`** | Catálogo de permisos del sistema (Crear, Editar, Ver). | **PK:** `id_permiso` |
| **`tb_personal_permisos`** | **Tabla pivote** para asignar permisos al personal. | **PK:** `id_personal_permiso`<br>**FK:** `fk_id_personal`<br>**FK:** `fk_id_permiso` |

### IV. Módulo de Evaluaciones (Transaccional)
| Tabla | Descripción | Claves Principales |
| :--- | :--- | :--- |
| **`tb_solicitudes`** | El "oficio" o paquete que agrupa varias evaluaciones. | **PK:** `id_solicitud`<br>**FK:** `fk_id_mando_solicitante`<br>**FK:** `fk_id_personal_creador` |
| **`tb_cat_motivos`** | Catálogo de razones (Promoción, Ingreso, Permanencia). | **PK:** `id_motivo` |
| **`tb_evaluaciones`** | Registro individual de evaluación de una persona. | **PK:** `id_evaluacion`<br>**FK:** `fk_id_solicitud`<br>**FK:** `fk_id_personal`<br>**FK:** `fk_id_motivo` |
| **`tb_historial_estatus`** | Bitácora de auditoría de cambios de estatus. | **PK:** `id_historial`<br>**FK:** `fk_id_evaluacion`<br>**FK:** `fk_id_personal_modificacion` |

---

## 4. Diagrama Entidad-Relación (Código Fuente)

Puede generar el diagrama visual pegando el siguiente código en [Mermaid Live Editor](https://mermaid.live).

```mermaid
erDiagram
    %% --- ENTIDADES ESTRUCTURALES ---
    tb_mandos {
        int id_mando PK
        string nombre_mando UK
    }
    tb_unidades {
        int id_unidad PK
        string nombre_unidad UK
        int fk_id_mando FK
    }

    %% --- ENTIDADES DE JERARQUÍA ---
    tb_cat_grados {
        int id_grado PK
        string nombre_grado UK
    }
    tb_complemento {
        int id_complemento PK
        string nombre_complemento UK
    }
    tb_cat_rangos_validos {
        int id_rango_valido PK
        int fk_id_grado FK
        int fk_id_complemento FK
    }

    %% --- NÚCLEO DE PERSONAL ---
    tb_personal {
        int id_personal PK
        int fk_id_unidad_pertenencia FK
        int fk_id_rango_valido FK
        string curp UK
        string rfc UK
        string matricula_guerra UK
        string nombre
        string apellido_paterno
        string contrasena "Hash"
    }

    %% --- SISTEMA DE PERMISOS ---
    tb_permisos {
        int id_permiso PK
        string nombre_permiso UK
    }
    tb_personal_permisos {
        int id_personal_permiso PK
        int fk_id_personal FK
        int fk_id_permiso FK
    }

    %% --- MÓDULO DE SOLICITUDES ---
    tb_solicitudes {
        int id_solicitud PK
        string folio_solicitud UK
        int fk_id_mando_solicitante FK
        int fk_id_personal_creador FK
        datetime fecha_creacion
    }
    tb_cat_motivos {
        int id_motivo PK
        string nombre_motivo UK
    }
    tb_evaluaciones {
        int id_evaluacion PK
        int fk_id_solicitud FK
        int fk_id_personal FK
        int fk_id_motivo FK
        string estatus_actual
        datetime fecha_ultimo_estatus
    }
    tb_historial_estatus {
        int id_historial PK
        int fk_id_evaluacion FK
        int fk_id_personal_modificacion FK
        string estatus_anterior
        string estatus_nuevo
        datetime fecha_cambio
    }

    %% --- RELACIONES ---
    tb_mandos ||--o{ tb_unidades : contiene
    tb_unidades ||--o{ tb_personal : pertenece
    
    tb_cat_grados ||--o{ tb_cat_rangos_validos : define
    tb_complemento ||--o{ tb_cat_rangos_validos : define
    tb_cat_rangos_validos ||--o{ tb_personal : posee

    tb_personal ||--o{ tb_personal_permisos : tiene
    tb_permisos ||--o{ tb_personal_permisos : asigna

    tb_mandos ||--o{ tb_solicitudes : genera
    tb_personal ||--o{ tb_solicitudes : crea
    
    tb_solicitudes ||--o{ tb_evaluaciones : agrupa
    tb_personal ||--o{ tb_evaluaciones : es_evaluado
    tb_cat_motivos ||--o{ tb_evaluaciones : define_motivo
    
    tb_evaluaciones ||--o{ tb_historial_estatus : registra
    tb_personal ||--o{ tb_historial_estatus : modifica
