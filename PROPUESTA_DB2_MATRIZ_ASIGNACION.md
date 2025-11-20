# Propuesta de Base de Datos DB2 para Matriz de Asignación Electrónica

> **Última actualización:** 18 de Noviembre 2025  
> **Versión:** 2.0

## Descripción General

Esta propuesta presenta una estructura de base de datos en DB2 para almacenar y gestionar la configuración de la matriz de asignación electrónica por COPE, categorías y subcategorías.

### Cambios Recientes (v2.0)
- ✅ **Reorganización de columnas:** DIVISION y AREA al inicio de la tabla
- ✅ **Eliminación del campo ACTIVO:** Simplificación de la estructura
- ✅ **Actualización de índices:** Índice único sin filtro WHERE
- ✅ **Procedimientos almacenados actualizados:** Nuevos parámetros en orden correcto
- ✅ **Consultas simplificadas:** Sin referencias a registros activos/inactivos
- ✅ **Tabla de auditoría mejorada:** Sistema de snapshots con ID_REFERENCIA para trazabilidad completa

---

## 1. Tabla Principal: MATRIZ_ASIGNACION_ELECTRONICA

Esta tabla almacena cada selección realizada mediante los radio buttons de la interfaz web.

### Estructura de la Tabla

```sql
CREATE TABLE MATRIZ_ASIGNACION_ELECTRONICA (
    ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    DIVISION VARCHAR(50),
    AREA VARCHAR(50),
    COPE VARCHAR(20) NOT NULL,
    CATEGORIA VARCHAR(30) NOT NULL,
    SUBCATEGORIA VARCHAR(10) NOT NULL,
    TIPO_ORDEN VARCHAR(20) NOT NULL,
    TIPO_RECURSO VARCHAR(20) NOT NULL,
    HABILITADO CHAR(1) NOT NULL DEFAULT 'S',
    USUARIO_REGISTRO VARCHAR(50) NOT NULL,
    FECHA_REGISTRO TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    USUARIO_MODIFICACION VARCHAR(50),
    FECHA_MODIFICACION TIMESTAMP,
    PRIMARY KEY (ID),
    CONSTRAINT CHK_TIPO_ORDEN CHECK (TIPO_ORDEN IN ('ORDENES', 'ORDENES_SF')),
    CONSTRAINT CHK_TIPO_RECURSO CHECK (TIPO_RECURSO IN ('Propios', 'Terceros')),
    CONSTRAINT CHK_HABILITADO CHECK (HABILITADO IN ('S', 'N'))
);
```

### Descripción de Campos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **ID** | INTEGER | Identificador único autogenerado |
| **DIVISION** | VARCHAR(50) | División seleccionada en los filtros (opcional) |
| **AREA** | VARCHAR(50) | Área seleccionada en los filtros (opcional) |
| **COPE** | VARCHAR(20) | Código del COPE (ej: 'COPE 1', 'COPE 2', etc.) |
| **CATEGORIA** | VARCHAR(30) | Categoría principal ('Altas', 'CambioDomicilio', 'Migraciones') |
| **SUBCATEGORIA** | VARCHAR(10) | Subcategoría seleccionada (A0, AT, D1, D2, D3, TS, TV, etc.) |
| **TIPO_ORDEN** | VARCHAR(20) | Tipo de orden asociado: 'ORDENES' o 'ORDENES_SF' |
| **TIPO_RECURSO** | VARCHAR(20) | Tipo de recurso: 'Propios' o 'Terceros' |
| **HABILITADO** | CHAR(1) | Estado del toggle del COPE: 'S' (habilitado) o 'N' (deshabilitado) |
| **USUARIO_REGISTRO** | VARCHAR(50) | Usuario que creó el registro |
| **FECHA_REGISTRO** | TIMESTAMP | Fecha y hora de creación del registro (default: CURRENT_TIMESTAMP) |
| **USUARIO_MODIFICACION** | VARCHAR(50) | Último usuario que modificó el registro |
| **FECHA_MODIFICACION** | TIMESTAMP | Fecha y hora de última modificación |

### Índices

```sql
-- Índice único para evitar duplicados por COPE + Categoría + Subcategoría
CREATE UNIQUE INDEX IDX_MATRIZ_COPE_CAT_SUB 
ON MATRIZ_ASIGNACION_ELECTRONICA(COPE, CATEGORIA, SUBCATEGORIA);

-- Índices para mejorar búsquedas
CREATE INDEX IDX_MATRIZ_COPE ON MATRIZ_ASIGNACION_ELECTRONICA(COPE);
CREATE INDEX IDX_MATRIZ_DIVISION_AREA ON MATRIZ_ASIGNACION_ELECTRONICA(DIVISION, AREA);
CREATE INDEX IDX_MATRIZ_CATEGORIA ON MATRIZ_ASIGNACION_ELECTRONICA(CATEGORIA);
```

### Comentarios en Columnas

```sql
COMMENT ON TABLE MATRIZ_ASIGNACION_ELECTRONICA 
IS 'Almacena la configuración de asignación electrónica por COPE y categoría';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.COPE 
IS 'Código del COPE (ej: COPE 1, COPE 2, etc.)';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.HABILITADO 
IS 'Estado del toggle del COPE: S (habilitado) o N (deshabilitado)';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.CATEGORIA 
IS 'Categoría principal (Altas, CambioDomicilio, Migraciones)';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.SUBCATEGORIA 
IS 'Subcategoría seleccionada (A0, AT, D1, D2, D3, TS, TV, etc.)';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.TIPO_ORDEN 
IS 'Tipo de orden: ORDENES o ORDENES_SF';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.TIPO_RECURSO 
IS 'Tipo de recurso: Propios o Terceros';
```

---

## 2. Tabla de Auditoría: MATRIZ_ASIGNACION_AUDITORIA

Tabla que registra todos los cambios realizados en la configuración mediante un sistema de **snapshots completos**. Cada registro es una fotografía completa del estado de la configuración en ese momento, enlazada al estado anterior mediante `ID_REFERENCIA`.

### Estructura de la Tabla

```sql
CREATE TABLE MATRIZ_ASIGNACION_AUDITORIA (
    ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    ID_REFERENCIA INTEGER,  -- Apunta al ID del registro anterior (NULL para el primero)
    
    -- Campos de identificación de la configuración
    DIVISION VARCHAR(50),
    AREA VARCHAR(50),
    COPE VARCHAR(20) NOT NULL,
    CATEGORIA VARCHAR(30) NOT NULL,
    SUBCATEGORIA VARCHAR(10) NOT NULL,
    TIPO_ORDEN VARCHAR(20) NOT NULL,
    
    -- Valores de la configuración (snapshot completo)
    TIPO_RECURSO VARCHAR(20) NOT NULL,
    HABILITADO CHAR(1) NOT NULL DEFAULT 'S',
    
    -- Metadatos
    USUARIO_REGISTRO VARCHAR(50) NOT NULL,
    FECHA_REGISTRO TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    
    PRIMARY KEY (ID),
    CONSTRAINT CHK_AUDIT_TIPO_ORDEN CHECK (TIPO_ORDEN IN ('ORDENES', 'ORDENES_SF')),
    CONSTRAINT CHK_AUDIT_TIPO_RECURSO CHECK (TIPO_RECURSO IN ('Propios', 'Terceros')),
    CONSTRAINT CHK_AUDIT_HABILITADO CHECK (HABILITADO IN ('S', 'N'))
);

CREATE INDEX IDX_AUDITORIA_ID_REF ON MATRIZ_ASIGNACION_AUDITORIA(ID_REFERENCIA);
CREATE INDEX IDX_AUDITORIA_CONFIG ON MATRIZ_ASIGNACION_AUDITORIA(DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA);
CREATE INDEX IDX_AUDITORIA_FECHA ON MATRIZ_ASIGNACION_AUDITORIA(FECHA_REGISTRO);
```

### Descripción de Campos

| Campo | Tipo | Descripción |
|-------|------|-------------|
| **ID** | INTEGER | Identificador único autogenerado de auditoría |
| **ID_REFERENCIA** | INTEGER | ID del registro anterior de la misma configuración (NULL = primer registro) |
| **DIVISION** | VARCHAR(50) | División de la configuración |
| **AREA** | VARCHAR(50) | Área de la configuración |
| **COPE** | VARCHAR(20) | Código del COPE |
| **CATEGORIA** | VARCHAR(30) | Categoría |
| **SUBCATEGORIA** | VARCHAR(10) | Subcategoría |
| **TIPO_ORDEN** | VARCHAR(20) | Tipo de orden en este snapshot |
| **TIPO_RECURSO** | VARCHAR(20) | Tipo de recurso en este snapshot |
| **HABILITADO** | CHAR(1) | Estado de habilitación en este snapshot |
| **USUARIO_REGISTRO** | VARCHAR(50) | Usuario que realizó el cambio |
| **FECHA_REGISTRO** | TIMESTAMP | Fecha y hora del cambio |

### Funcionamiento del Sistema de Snapshots

```
Ejemplo de cadena de auditoría:

ID=1, ID_REF=0     → Primer INSERT (sin referencia)
ID=2, ID_REF=1     → Primer UPDATE (referencia a ID 1)
ID=3, ID_REF=2     → Segundo UPDATE (referencia a ID 2)
ID=4, ID_REF=3     → DELETE (referencia a ID 3)
```

### Comentarios

```sql
COMMENT ON TABLE MATRIZ_ASIGNACION_AUDITORIA 
IS 'Registro de auditoría mediante snapshots completos - cada registro contiene el estado completo de la configuración';

COMMENT ON COLUMN MATRIZ_ASIGNACION_AUDITORIA.ID_REFERENCIA 
IS 'ID del registro anterior de la misma configuración (NULL o 0 para el primer registro)';
```

---

## 3. Operaciones de Base de Datos

### 3.1 Insertar Nueva Configuración

```sql
-- Insertar una selección individual
INSERT INTO MATRIZ_ASIGNACION_ELECTRONICA 
    (DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, HABILITADO, USUARIO_REGISTRO)
VALUES 
    ('División Norte', 'Red', 'COPE 1', 'Altas', 'A0', 'ORDENES', 'Propios', 'S', 'USUARIO123');

-- Insertar múltiples selecciones (al guardar la matriz completa)
INSERT INTO MATRIZ_ASIGNACION_ELECTRONICA 
    (DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, HABILITADO, USUARIO_REGISTRO)
VALUES 
    ('División Norte', 'Red', 'COPE 1', 'Altas', 'A0', 'ORDENES', 'Propios', 'S', 'USUARIO123'),
    ('División Norte', 'Red', 'COPE 1', 'Altas', 'AT', 'ORDENES_SF', 'Terceros', 'S', 'USUARIO123'),
    ('División Norte', 'Red', 'COPE 2', 'CambioDomicilio', 'D1', 'ORDENES', 'Propios', 'N', 'USUARIO123'),
    ('División Norte', 'Red', 'COPE 2', 'CambioDomicilio', 'D2', 'ORDENES_SF', 'Terceros', 'N', 'USUARIO123');
```

### 3.2 Actualizar Configuración Existente

```sql
-- Actualizar tipo de orden y recurso para una subcategoría específica
UPDATE MATRIZ_ASIGNACION_ELECTRONICA
SET TIPO_ORDEN = 'ORDENES_SF',
    TIPO_RECURSO = 'Terceros',
    USUARIO_MODIFICACION = 'USUARIO456',
    FECHA_MODIFICACION = CURRENT_TIMESTAMP
WHERE COPE = 'COPE 1' 
  AND CATEGORIA = 'Altas' 
  AND SUBCATEGORIA = 'A0';

-- Actualizar estado de habilitación del COPE
UPDATE MATRIZ_ASIGNACION_ELECTRONICA
SET HABILITADO = 'N',
    USUARIO_MODIFICACION = 'USUARIO456',
    FECHA_MODIFICACION = CURRENT_TIMESTAMP
WHERE COPE = 'COPE 1';

-- Actualizar División y Área
UPDATE MATRIZ_ASIGNACION_ELECTRONICA
SET DIVISION = 'División Sur',
    AREA = 'Comercial',
    USUARIO_MODIFICACION = 'USUARIO789',
    FECHA_MODIFICACION = CURRENT_TIMESTAMP
WHERE COPE = 'COPE 1';
```

### 3.3 Eliminar Configuración

```sql
-- Eliminar una configuración específica
DELETE FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE COPE = 'COPE 1' 
  AND CATEGORIA = 'Altas' 
  AND SUBCATEGORIA = 'A0';

-- Eliminar todas las configuraciones de un COPE
DELETE FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE COPE = 'COPE 1';
```

### 3.4 Consultas de Recuperación

```sql
-- Obtener configuración para un COPE específico
SELECT DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, HABILITADO
FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE COPE = 'COPE 1'
ORDER BY CATEGORIA, SUBCATEGORIA;

-- Obtener configuración por división y área
SELECT COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, HABILITADO
FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE DIVISION = 'División Norte' 
  AND AREA = 'Red'
ORDER BY COPE, CATEGORIA, SUBCATEGORIA;

-- Obtener todas las configuraciones
SELECT DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, HABILITADO
FROM MATRIZ_ASIGNACION_ELECTRONICA
ORDER BY DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA;

-- Obtener solo COPEs habilitados
SELECT DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO
FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE HABILITADO = 'S'
ORDER BY COPE, CATEGORIA, SUBCATEGORIA;

-- Buscar COPEs que contengan un texto específico
SELECT DISTINCT COPE, HABILITADO
FROM MATRIZ_ASIGNACION_ELECTRONICA
WHERE COPE LIKE '%1%'
ORDER BY COPE;

-- Obtener resumen de configuraciones por categoría
SELECT CATEGORIA, COUNT(*) AS TOTAL_CONFIGURACIONES,
       SUM(CASE WHEN HABILITADO = 'S' THEN 1 ELSE 0 END) AS HABILITADOS,
       SUM(CASE WHEN HABILITADO = 'N' THEN 1 ELSE 0 END) AS DESHABILITADOS
FROM MATRIZ_ASIGNACION_ELECTRONICA
GROUP BY CATEGORIA
ORDER BY CATEGORIA;
```

### 3.5 Consultas de Auditoría

```sql
-- Ver historial completo de una configuración específica
SELECT 
    A.ID,
    A.ID_REFERENCIA,
    A.TIPO_RECURSO,
    A.HABILITADO,
    A.USUARIO_REGISTRO,
    A.FECHA_REGISTRO
FROM MATRIZ_ASIGNACION_AUDITORIA A
WHERE A.DIVISION = 'División Norte'
  AND A.AREA = 'Red'
  AND A.COPE = 'COPE 1'
  AND A.CATEGORIA = 'Altas'
  AND A.SUBCATEGORIA = 'A0'
  AND A.TIPO_ORDEN = 'ORDENES'
ORDER BY A.ID;

-- Ver cambios recientes (últimas 24 horas)
SELECT 
    A.ID,
    A.ID_REFERENCIA,
    A.COPE,
    A.CATEGORIA,
    A.SUBCATEGORIA,
    A.TIPO_RECURSO,
    A.HABILITADO,
    A.USUARIO_REGISTRO,
    A.FECHA_REGISTRO
FROM MATRIZ_ASIGNACION_AUDITORIA A
WHERE A.FECHA_REGISTRO >= CURRENT_TIMESTAMP - 24 HOURS
ORDER BY A.FECHA_REGISTRO DESC;

-- Comparar un registro con su anterior para ver cambios
SELECT 
    'Actual' AS MOMENTO,
    A1.ID,
    A1.TIPO_RECURSO,
    A1.HABILITADO,
    A1.USUARIO_REGISTRO,
    A1.FECHA_REGISTRO
FROM MATRIZ_ASIGNACION_AUDITORIA A1
WHERE A1.ID = 3

UNION ALL

SELECT 
    'Anterior' AS MOMENTO,
    A2.ID,
    A2.TIPO_RECURSO,
    A2.HABILITADO,
    A2.USUARIO_REGISTRO,
    A2.FECHA_REGISTRO
FROM MATRIZ_ASIGNACION_AUDITORIA A1
INNER JOIN MATRIZ_ASIGNACION_AUDITORIA A2 ON A1.ID_REFERENCIA = A2.ID
WHERE A1.ID = 3;

-- Obtener el último estado de cada configuración
SELECT DISTINCT
    FIRST_VALUE(A.ID) OVER (PARTITION BY A.DIVISION, A.AREA, A.COPE, A.CATEGORIA, A.SUBCATEGORIA, A.TIPO_ORDEN ORDER BY A.ID DESC) AS ULTIMO_ID,
    A.DIVISION,
    A.AREA,
    A.COPE,
    A.CATEGORIA,
    A.SUBCATEGORIA,
    A.TIPO_ORDEN,
    FIRST_VALUE(A.TIPO_RECURSO) OVER (PARTITION BY A.DIVISION, A.AREA, A.COPE, A.CATEGORIA, A.SUBCATEGORIA, A.TIPO_ORDEN ORDER BY A.ID DESC) AS TIPO_RECURSO_ACTUAL,
    FIRST_VALUE(A.HABILITADO) OVER (PARTITION BY A.DIVISION, A.AREA, A.COPE, A.CATEGORIA, A.SUBCATEGORIA, A.TIPO_ORDEN ORDER BY A.ID DESC) AS HABILITADO_ACTUAL
FROM MATRIZ_ASIGNACION_AUDITORIA A;
```

---

## 4. Procedimiento Almacenado para Guardar Configuración

```sql
CREATE OR REPLACE PROCEDURE SP_GUARDAR_MATRIZ_ASIGNACION (
    IN p_division VARCHAR(50),
    IN p_area VARCHAR(50),
    IN p_cope VARCHAR(20),
    IN p_categoria VARCHAR(30),
    IN p_subcategoria VARCHAR(10),
    IN p_tipo_orden VARCHAR(20),
    IN p_tipo_recurso VARCHAR(20),
    IN p_habilitado CHAR(1),
    IN p_usuario VARCHAR(50),
    OUT p_resultado VARCHAR(100)
)
LANGUAGE SQL
BEGIN
    DECLARE v_existe INTEGER;
    DECLARE v_id INTEGER;
    DECLARE v_ultimo_audit_id INTEGER;
    
    -- Verificar si ya existe un registro
    SELECT COUNT(*), MAX(ID)
    INTO v_existe, v_id
    FROM MATRIZ_ASIGNACION_ELECTRONICA
    WHERE COPE = p_cope
      AND CATEGORIA = p_categoria
      AND SUBCATEGORIA = p_subcategoria;
    
    IF v_existe > 0 THEN
        -- Actualizar registro existente en tabla principal
        UPDATE MATRIZ_ASIGNACION_ELECTRONICA
        SET DIVISION = p_division,
            AREA = p_area,
            TIPO_ORDEN = p_tipo_orden,
            TIPO_RECURSO = p_tipo_recurso,
            HABILITADO = p_habilitado,
            USUARIO_MODIFICACION = p_usuario,
            FECHA_MODIFICACION = CURRENT_TIMESTAMP
        WHERE ID = v_id;
        
        -- Obtener el último ID de auditoría para esta configuración
        SELECT MAX(ID) INTO v_ultimo_audit_id
        FROM MATRIZ_ASIGNACION_AUDITORIA
        WHERE DIVISION = p_division
          AND AREA = p_area
          AND COPE = p_cope
          AND CATEGORIA = p_categoria
          AND SUBCATEGORIA = p_subcategoria
          AND TIPO_ORDEN = p_tipo_orden;
        
        -- Registrar snapshot completo en auditoría
        INSERT INTO MATRIZ_ASIGNACION_AUDITORIA
            (ID_REFERENCIA, DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA,
             TIPO_ORDEN, TIPO_RECURSO, HABILITADO, USUARIO_REGISTRO)
        VALUES
            (COALESCE(v_ultimo_audit_id, 0),
             p_division, p_area, p_cope, p_categoria, p_subcategoria,
             p_tipo_orden, p_tipo_recurso, p_habilitado, p_usuario);
        
        SET p_resultado = 'Registro actualizado exitosamente';
    ELSE
        -- Insertar nuevo registro en tabla principal
        INSERT INTO MATRIZ_ASIGNACION_ELECTRONICA
            (DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, 
             TIPO_ORDEN, TIPO_RECURSO, HABILITADO, USUARIO_REGISTRO)
        VALUES
            (p_division, p_area, p_cope, p_categoria, p_subcategoria,
             p_tipo_orden, p_tipo_recurso, p_habilitado, p_usuario);
        
        -- Registrar primer snapshot en auditoría (ID_REFERENCIA = 0)
        INSERT INTO MATRIZ_ASIGNACION_AUDITORIA
            (ID_REFERENCIA, DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA,
             TIPO_ORDEN, TIPO_RECURSO, HABILITADO, USUARIO_REGISTRO)
        VALUES
            (0, p_division, p_area, p_cope, p_categoria, p_subcategoria,
             p_tipo_orden, p_tipo_recurso, p_habilitado, p_usuario);
        
        SET p_resultado = 'Registro creado exitosamente';
    END IF;
    
    COMMIT;
END;
```

### Ejemplo de uso del procedimiento:

```sql
-- Declarar variable para el resultado
DECLARE v_resultado VARCHAR(100);

-- Llamar al procedimiento
CALL SP_GUARDAR_MATRIZ_ASIGNACION(
    'División Norte',    -- p_division
    'Red',               -- p_area
    'COPE 1',            -- p_cope
    'Altas',             -- p_categoria
    'A0',                -- p_subcategoria
    'ORDENES',           -- p_tipo_orden
    'Propios',           -- p_tipo_recurso
    'S',                 -- p_habilitado
    'USUARIO123',        -- p_usuario
    v_resultado          -- p_resultado (OUT)
);

-- Ver el resultado
SELECT v_resultado FROM SYSIBM.SYSDUMMY1;
```

---

## 5. Integración con la Aplicación Web

### 5.1 Estructura JSON al Guardar

Cuando el usuario presiona el botón "Guardar", el HTML genera un JSON como este:

```json
{
  "division": "División Norte",
  "area": "Red",
  "cope": "(Todos)",
  "selecciones": [
    {
      "Habilitado": "SI",
      "Division": "División Norte",
      "Area": "Red",
      "COPE": "COPE 1",
      "Categoria": "Altas",
      "Valor": "A0",
      "TipoDeRecurso": "Propios"
    },
    {
      "Habilitado": "NO",
      "Division": "División Norte",
      "Area": "Red",
      "COPE": "COPE 2",
      "Categoria": "CambioDomicilio",
      "Valor": "D2",
      "TipoDeRecurso": "Terceros"
    }
  ],
  "totalSelecciones": 2
}
```

### 5.2 Backend: Procesamiento y Guardado

El backend (Java, Node.js, Python, etc.) debe:

1. Recibir el JSON desde el frontend
2. Iterar sobre cada COPE y sus selecciones
3. Llamar al procedimiento almacenado o ejecutar INSERT/UPDATE directo
4. Retornar respuesta de éxito o error

**Pseudocódigo:**

```javascript
// Ejemplo en Node.js/Express con ibm_db
app.post('/api/matriz/guardar', async (req, res) => {
    const { division, area, selecciones } = req.body;
    const usuario = req.user.id; // Usuario autenticado
    
    try {
        for (const item of selecciones) {
            const habilitadoDB = item.Habilitado === 'SI' ? 'S' : 'N';
            const tipoOrden = item.TipoDeRecurso === 'Propios' ? 'ORDENES' : 'ORDENES_SF';
            
            // Ejecutar INSERT
            await db.query(`
                INSERT INTO MATRIZ_ASIGNACION_ELECTRONICA 
                (DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA, TIPO_ORDEN, TIPO_RECURSO, 
                 HABILITADO, USUARIO_REGISTRO)
                VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
            `, [item.Division, item.Area, item.COPE, item.Categoria, item.Valor, 
                tipoOrden, item.TipoDeRecurso, habilitadoDB, usuario]);
        }
        
        res.json({ success: true, message: 'Configuración guardada correctamente' });
    } catch (error) {
        res.status(500).json({ success: false, error: error.message });
    }
});
```

---

## 6. Ventajas del Diseño Propuesto

### ✅ Normalización
- Estructura normalizada evita redundancia de datos
- Fácil mantenimiento y actualización

### ✅ Flexibilidad
- Permite agregar nuevas categorías/subcategorías sin modificar esquema
- Soporta múltiples divisiones y áreas

### ✅ Auditoría Completa
- Tabla de auditoría rastrea todos los cambios
- Historial completo de modificaciones por usuario

### ✅ Rendimiento
- Índices optimizados para búsquedas frecuentes
- Índice único previene duplicados

### ✅ Integridad de Datos
- Constraints aseguran valores válidos
- Check constraints validan tipos de orden

### ✅ Control de Estado
- Campo HABILITADO permite activar/desactivar COPEs
- Mantiene configuraciones sin eliminarlas

### ✅ Escalabilidad
- Diseño soporta crecimiento de datos
- Fácil agregar nuevos campos si es necesario

---

## 7. Consideraciones Adicionales

### Seguridad
- Implementar control de acceso a nivel de aplicación
- Validar permisos de usuario antes de INSERT/UPDATE
- Encriptar datos sensibles si es necesario

### Respaldos
- Implementar política de respaldo regular
- Considerar archivado de registros antiguos si es necesario

### Monitoreo
- Crear vistas para reportes de uso
- Monitorear tamaño de tablas y rendimiento de índices

### Mantenimiento
- Programar mantenimiento periódico de índices (REORG, RUNSTATS)
- Archivar registros de auditoría antiguos periódicamente

---

## 8. Scripts de Mantenimiento

### Reorganizar Tablas e Índices

```sql
-- Reorganizar tabla principal
CALL SYSPROC.ADMIN_CMD('REORG TABLE MATRIZ_ASIGNACION_ELECTRONICA');

-- Actualizar estadísticas
CALL SYSPROC.ADMIN_CMD('RUNSTATS ON TABLE MATRIZ_ASIGNACION_ELECTRONICA 
    WITH DISTRIBUTION AND DETAILED INDEXES ALL');
```

### Archivar Auditoría Antigua

```sql
-- Crear tabla de archivo
CREATE TABLE MATRIZ_ASIGNACION_AUDITORIA_ARCHIVO 
    LIKE MATRIZ_ASIGNACION_AUDITORIA;

-- Mover registros antiguos (más de 1 año)
INSERT INTO MATRIZ_ASIGNACION_AUDITORIA_ARCHIVO
SELECT * FROM MATRIZ_ASIGNACION_AUDITORIA
WHERE FECHA < CURRENT_TIMESTAMP - 1 YEAR;

-- Eliminar registros archivados
DELETE FROM MATRIZ_ASIGNACION_AUDITORIA
WHERE FECHA < CURRENT_TIMESTAMP - 1 YEAR;
```

---

## 9. Resumen de Estructura Actualizada (v2.0)

### Tabla Principal: MATRIZ_ASIGNACION_ELECTRONICA

```
┌─────────────────────────────────────────────────────────────────────┐
│                  MATRIZ_ASIGNACION_ELECTRONICA                      │
├──────────────────────────┬──────────────┬─────────────────────────┤
│ Columna                  │ Tipo         │ Descripción              │
├──────────────────────────┼──────────────┼─────────────────────────┤
│ ID                       │ INTEGER      │ PK, Autoincremental     │
│ DIVISION                 │ VARCHAR(50)  │ División (Opcional)     │
│ AREA                     │ VARCHAR(50)  │ Área (Opcional)         │
│ COPE                     │ VARCHAR(20)  │ NOT NULL                │
│ CATEGORIA                │ VARCHAR(30)  │ NOT NULL                │
│ SUBCATEGORIA             │ VARCHAR(10)  │ NOT NULL                │
│ TIPO_ORDEN               │ VARCHAR(20)  │ NOT NULL, CHK           │
│ TIPO_RECURSO             │ VARCHAR(20)  │ NOT NULL, CHK           │
│ HABILITADO               │ CHAR(1)      │ NOT NULL, DEFAULT 'S'   │
│ USUARIO_REGISTRO         │ VARCHAR(50)  │ NOT NULL                │
│ FECHA_REGISTRO           │ TIMESTAMP    │ NOT NULL, DEFAULT NOW   │
│ USUARIO_MODIFICACION     │ VARCHAR(50)  │                         │
│ FECHA_MODIFICACION       │ TIMESTAMP    │                         │
└──────────────────────────┴──────────────┴─────────────────────────┘

CONSTRAINTS:
  ✓ CHK_TIPO_ORDEN: IN ('ORDENES', 'ORDENES_SF')
  ✓ CHK_TIPO_RECURSO: IN ('Propios', 'Terceros')
  ✓ CHK_HABILITADO: IN ('S', 'N')

INDICES:
  ✓ IDX_MATRIZ_COPE_CAT_SUB (UNIQUE): COPE + CATEGORIA + SUBCATEGORIA
  ✓ IDX_MATRIZ_COPE: COPE
  ✓ IDX_MATRIZ_DIVISION_AREA: DIVISION + AREA
  ✓ IDX_MATRIZ_CATEGORIA: CATEGORIA
```

### Comparación con Versión Anterior

| Aspecto | Versión 1.0 | Versión 2.0 ✨ |
|---------|-------------|----------------|
| **Orden de columnas** | ID, COPE, HABILITADO, ... | ID, DIVISION, AREA, COPE, ... |
| **Campo ACTIVO** | ✓ Presente | ✗ Eliminado |
| **Índice único** | Con filtro WHERE ACTIVO='S' | Sin filtro |
| **Complejidad** | Mayor (borrado lógico) | Menor (más simple) |
| **Consultas** | Requieren filtro ACTIVO='S' | Consultas directas |

### Beneficios de la Nueva Estructura

✅ **Simplicidad:** Eliminación del campo ACTIVO reduce complejidad  
✅ **Claridad:** Columnas organizadas por prioridad (DIVISION → AREA → COPE)  
✅ **Rendimiento:** Índice único sin filtros condicionales  
✅ **Mantenimiento:** Menos validaciones en consultas  
✅ **Flexibilidad:** HABILITADO controla estado del COPE

---

## 10. Script Completo de Creación

```sql
-- ============================================================
-- Script de Creación Completa - Matriz Asignación Electrónica
-- Versión: 2.0
-- Fecha: 18 de Noviembre 2025
-- ============================================================

-- Tabla Principal
CREATE TABLE MATRIZ_ASIGNACION_ELECTRONICA (
    ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    DIVISION VARCHAR(50),
    AREA VARCHAR(50),
    COPE VARCHAR(20) NOT NULL,
    CATEGORIA VARCHAR(30) NOT NULL,
    SUBCATEGORIA VARCHAR(10) NOT NULL,
    TIPO_ORDEN VARCHAR(20) NOT NULL,
    TIPO_RECURSO VARCHAR(20) NOT NULL,
    HABILITADO CHAR(1) NOT NULL DEFAULT 'S',
    USUARIO_REGISTRO VARCHAR(50) NOT NULL,
    FECHA_REGISTRO TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    USUARIO_MODIFICACION VARCHAR(50),
    FECHA_MODIFICACION TIMESTAMP,
    PRIMARY KEY (ID),
    CONSTRAINT CHK_TIPO_ORDEN CHECK (TIPO_ORDEN IN ('ORDENES', 'ORDENES_SF')),
    CONSTRAINT CHK_TIPO_RECURSO CHECK (TIPO_RECURSO IN ('Propios', 'Terceros')),
    CONSTRAINT CHK_HABILITADO CHECK (HABILITADO IN ('S', 'N'))
);

-- Índices
CREATE UNIQUE INDEX IDX_MATRIZ_COPE_CAT_SUB 
ON MATRIZ_ASIGNACION_ELECTRONICA(COPE, CATEGORIA, SUBCATEGORIA);

CREATE INDEX IDX_MATRIZ_COPE 
ON MATRIZ_ASIGNACION_ELECTRONICA(COPE);

CREATE INDEX IDX_MATRIZ_DIVISION_AREA 
ON MATRIZ_ASIGNACION_ELECTRONICA(DIVISION, AREA);

CREATE INDEX IDX_MATRIZ_CATEGORIA 
ON MATRIZ_ASIGNACION_ELECTRONICA(CATEGORIA);

-- Comentarios
COMMENT ON TABLE MATRIZ_ASIGNACION_ELECTRONICA 
IS 'Almacena configuración de asignación electrónica por COPE - v2.0';

COMMENT ON COLUMN MATRIZ_ASIGNACION_ELECTRONICA.HABILITADO 
IS 'Estado del COPE: S (habilitado) o N (deshabilitado)';

-- Tabla de Auditoría (Sistema de Snapshots)
CREATE TABLE MATRIZ_ASIGNACION_AUDITORIA (
    ID INTEGER NOT NULL GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
    ID_REFERENCIA INTEGER,
    DIVISION VARCHAR(50),
    AREA VARCHAR(50),
    COPE VARCHAR(20) NOT NULL,
    CATEGORIA VARCHAR(30) NOT NULL,
    SUBCATEGORIA VARCHAR(10) NOT NULL,
    TIPO_ORDEN VARCHAR(20) NOT NULL,
    TIPO_RECURSO VARCHAR(20) NOT NULL,
    HABILITADO CHAR(1) NOT NULL DEFAULT 'S',
    USUARIO_REGISTRO VARCHAR(50) NOT NULL,
    FECHA_REGISTRO TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (ID),
    CONSTRAINT CHK_AUDIT_TIPO_ORDEN CHECK (TIPO_ORDEN IN ('ORDENES', 'ORDENES_SF')),
    CONSTRAINT CHK_AUDIT_TIPO_RECURSO CHECK (TIPO_RECURSO IN ('Propios', 'Terceros')),
    CONSTRAINT CHK_AUDIT_HABILITADO CHECK (HABILITADO IN ('S', 'N'))
);

CREATE INDEX IDX_AUDITORIA_ID_REF 
ON MATRIZ_ASIGNACION_AUDITORIA(ID_REFERENCIA);

CREATE INDEX IDX_AUDITORIA_CONFIG 
ON MATRIZ_ASIGNACION_AUDITORIA(DIVISION, AREA, COPE, CATEGORIA, SUBCATEGORIA);

CREATE INDEX IDX_AUDITORIA_FECHA 
ON MATRIZ_ASIGNACION_AUDITORIA(FECHA_REGISTRO);

COMMENT ON TABLE MATRIZ_ASIGNACION_AUDITORIA 
IS 'Registro de auditoría mediante snapshots completos - cada registro contiene el estado completo de la configuración';

COMMENT ON COLUMN MATRIZ_ASIGNACION_AUDITORIA.ID_REFERENCIA 
IS 'ID del registro anterior de la misma configuración (0 o NULL para el primer registro)';
```

---

**Fin del documento - Versión 2.0**

COMMIT;
```

---

## Conclusión

Esta propuesta proporciona una base de datos robusta, escalable y auditable para el sistema de Matriz de Asignación Electrónica. El diseño permite un control completo de las configuraciones por COPE, manteniendo integridad de datos y facilidad de consulta.
