# 🗄️ Liquibase Database Migration Project

Este proyecto demuestra la implementación de migraciones de base de datos usando **Liquibase** con **Oracle Database**, incluyendo entornos de desarrollo y producción.

## 📋 Descripción del Proyecto

Sistema de gestión de tareas y equipos que incluye las siguientes entidades:
- **Usuarios** (Users) - Gestión de usuarios del sistema
- **Equipos** (Teams) - Organización en equipos de trabajo  
- **Épicas** (Epics) - Agrupación de tareas por funcionalidad
- **Sprints** - Períodos de trabajo organizados
- **Tareas** (Tasks) - Tareas individuales del proyecto
- **Estudiantes** (Students) - Gestión de estudiantes
- **Relaciones** - Tablas de asociación many-to-many

## 🏗️ Arquitectura de Base de Datos

### Tablas Principales
```sql
USERS          - Usuarios del sistema
TEAMS          - Equipos de trabajo
EPICS          - Épicas/Funcionalidades
SPRINTS        - Períodos de trabajo
TASKS          - Tareas individuales
STUDENTS       - Información de estudiantes
```

### Tablas de Relación
```sql
USER_TEAMS        - Usuarios ↔ Equipos
TASK_ASSIGNMENTS  - Tareas ↔ Usuarios
TASK_SPRINT       - Tareas ↔ Sprints
```

## 🛠️ Tecnologías Utilizadas

- **Liquibase 4.31.1** - Herramienta de migración de base de datos
- **Oracle Database 23ai Free** - Base de datos Oracle
- **Docker** - Contenedorización de la base de datos
- **SQL** - Lenguaje de definición de datos

## 📁 Estructura del Proyecto

```
Sprint5Databases-main/
├── README.md                           # Este archivo
├── DEMO_RESULTS.md                     # Resultados de la demostración
├── liquibase-dev.properties            # Configuración de desarrollo
├── liquibase-prod.properties           # Configuración de producción
└── project/
    └── db-migrations/
        └── changelog/
            ├── changelog.xml            # Archivo principal de cambios
            └── changesets/
                ├── 001-initial-schema.sql    # Schema inicial
                ├── 002-add-users-table.sql   # Tabla de usuarios
                └── 003-create-students-table.sql # Tabla de estudiantes
```

## 🚀 Instalación y Configuración

### Prerrequisitos
- Docker instalado
- Java 11+ instalado
- Liquibase 4.31.1+ instalado

### 1. Configurar Base de Datos Oracle

#### Entorno de Desarrollo
```bash
docker run -d --name oracle-dev \
  --memory=2g --shm-size=1g \
  -p 1523:1521 \
  -e ORACLE_PASSWORD=oracle \
  gvenzl/oracle-free:slim-faststart
```

#### Entorno de Producción
```bash
# Crear usuario de producción en el contenedor
docker exec oracle-dev bash -c "sqlplus -s system/oracle@FREE <<EOF
CREATE USER prod_user IDENTIFIED BY prod_password;
GRANT CONNECT, RESOURCE, DBA TO prod_user;
EXIT;
EOF"
```

### 2. Configurar Conexiones

#### Desarrollo (`liquibase-dev.properties`)
```properties
changeLogFile=project/db-migrations/changelog/changelog.xml
url=jdbc:oracle:thin:@//localhost:1523/FREE
username=system
password=oracle
driver=oracle.jdbc.OracleDriver
```

#### Producción (`liquibase-prod.properties`)
```properties
changeLogFile=project/db-migrations/changelog/changelog.xml
url=jdbc:oracle:thin:@//localhost:1523/FREE
username=prod_user
password=prod_password
driver=oracle.jdbc.OracleDriver
```

## 📝 Uso del Proyecto

### Verificar Estado de Migraciones
```bash
# Desarrollo
liquibase --defaults-file=liquibase-dev.properties status

# Producción
liquibase --defaults-file=liquibase-prod.properties status
```

### Aplicar Migraciones
```bash
# Desarrollo
liquibase --defaults-file=liquibase-dev.properties update

# Producción
liquibase --defaults-file=liquibase-prod.properties update
```

### Verificar Estructura de Tablas
```bash
# Desarrollo
docker exec oracle-dev bash -c "echo 'describe users;' | sqlplus -s system/oracle@FREE"

# Producción  
docker exec oracle-dev bash -c "echo 'describe users;' | sqlplus -s prod_user/prod_password@FREE"
```

## 📊 Esquema de Base de Datos

### Tabla USERS
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| USER_ID | NUMBER | PRIMARY KEY, IDENTITY |
| NAME | VARCHAR2(255) | NOT NULL |
| EMAIL | VARCHAR2(255) | UNIQUE, NOT NULL |
| PASSWORD | VARCHAR2(255) | NOT NULL |
| ROLE | VARCHAR2(20) | CHECK (Manager, Developer, Tester) |
| WORK_MODE | VARCHAR2(20) | CHECK (On Site, Hybrid, Remote) |
| TELEGRAM_CHAT_ID | NUMBER | UNIQUE |
| CREATED_AT | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| IS_ACTIVE | NUMBER(1,0) | DEFAULT 1, NOT NULL |

### Tabla TASKS
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| TASK_ID | NUMBER | PRIMARY KEY, IDENTITY |
| TITLE | VARCHAR2(255) | NOT NULL |
| DESCRIPTION | CLOB | |
| EPIC_ID | NUMBER | FK → EPICS(EPIC_ID) |
| PRIORITY | VARCHAR2(20) | CHECK (Low, Medium, High) |
| STATUS | VARCHAR2(20) | CHECK (Backlog, ToDo, InProgress, Done, Blocked) |
| TYPE | VARCHAR2(20) | CHECK (Ticket, Bug, Feature, Improvement) |
| ESTIMATED_DEADLINE | DATE | |
| REAL_DEADLINE | DATE | |
| USER_POINTS | NUMBER(3,0) | CHECK > 0 |
| CREATED_AT | TIMESTAMP | DEFAULT CURRENT_TIMESTAMP |
| ESTIMATED_HOURS | NUMBER | |
| REAL_HOURS | NUMBER | |

### Tabla STUDENTS
| Campo | Tipo | Restricciones |
|-------|------|---------------|
| ID | NUMBER | PRIMARY KEY, IDENTITY |
| FIRST_NAME | VARCHAR2(255) | NOT NULL |
| LAST_NAME | VARCHAR2(255) | NOT NULL |
| EMAIL | VARCHAR2(255) | NOT NULL |
| ENROLLMENT_DATE | DATE | NOT NULL |

## 🔄 Versionado de Cambios

### Changesets Implementados

1. **001-initial-schema** - Esquema inicial con tablas principales
   - USERS, TEAMS, EPICS, SPRINTS, TASKS
   - Relaciones USER_TEAMS, TASK_ASSIGNMENTS, TASK_SPRINT

2. **002-add-users-table** - Extensiones de tabla de usuarios

3. **003-create-students-table** - Tabla de estudiantes

## 🧪 Testing y Validación

### Verificar Migraciones Aplicadas
```sql
SELECT id, author, filename, dateexecuted 
FROM databasechangelog 
ORDER BY dateexecuted;
```

### Verificar Tablas Creadas
```sql
SELECT table_name 
FROM user_tables 
WHERE table_name IN ('USERS', 'TEAMS', 'EPICS', 'SPRINTS', 'TASKS', 'STUDENTS') 
ORDER BY table_name;
```

## 📷 Capturas de Pantalla

Para obtener las capturas de pantalla necesarias, ejecutar:

1. **Estructura de Tablas en Desarrollo**
   ```bash
   docker exec oracle-dev bash -c "echo 'describe students;' | sqlplus -s system/oracle@FREE"
   ```

2. **Aplicación Exitosa en Desarrollo**
   ```bash
   liquibase --defaults-file=liquibase-dev.properties update
   ```

3. **Aplicación Exitosa en Producción**
   ```bash
   liquibase --defaults-file=liquibase-prod.properties update
   ```

## 🎯 Resultados Esperados

✅ **Desarrollo**: 3 changesets aplicados exitosamente  
✅ **Producción**: 3 changesets aplicados exitosamente  
✅ **Esquema**: 8 tablas creadas con relaciones  
✅ **Tracking**: Registro completo en DATABASECHANGELOG  

## 👥 Integrantes del Equipo

| Nombre | Matrícula |
|--------|-----------|
| Adair Virgilio Figueroa Medina | A00572826 |
| Diego Alejandro Michel Castro | A01641907 |
| José Oswaldo Sobrevilla Vázquez | A01412742 |
| Omar Arias Zepeda | A00830966 |

## 📞 Información de Contacto

- **Proyecto**: Sprint 5 - Bases de Datos
- **Tecnología**: Liquibase + Oracle Database
- **Estado**: ✅ Completado y Funcional

## 📜 Licencia

Este proyecto es parte de un ejercicio académico para el curso de Bases de Datos.

---

**Nota**: Para más detalles sobre la ejecución y resultados, consultar `DEMO_RESULTS.md`.
