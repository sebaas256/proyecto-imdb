Proyecto IMDb - Configuración del Entorno de Datos

Este repositorio contiene la configuración necesaria para levantar una instancia de SQL Server 2022 mediante Docker, específicamente diseñada para el análisis de la base de datos de IMDb.

---
Requisitos Previos

    Docker & Docker Compose instalados.

    Archivo de backup IMDb_Source_v2.bak (Solicitar enlace de descarga al equipo).
---

Instrucciones de Configuración
1. Preparación de Carpetas

Clona este repositorio y asegúrate de tener la siguiente estructura de carpetas:

```
proyecto_imdb/
├── backups/           # <-- Coloca aquí el archivo .bak
├── data/              # <-- Se llenará automáticamente (Ignorado por Git)
└── docker-compose.yml
```

2. Permisos (Solo para usuarios de Linux)

Si estás en Ubuntu/Linux, ejecuta los siguientes comandos para que Docker tenga permisos de escritura en la carpeta de datos:

```
sudo chown -R 10001:0 data/
sudo chmod -R 770 data/
```

(En Windows no es necesario realizar este paso).

3. Levantar el Servidor

Desde la terminal, dentro de la carpeta del proyecto, ejecuta:

```
docker-compose up -d
```

-Conexión a la Base de Datos

Utiliza Azure Data Studio o VS Code (SQL Server Extension) con los siguientes datos:

    Server: localhost,1434

    User: sa

    Password: Sebasqr456# (O la definida en el .env)

    Trust Server Certificate: True (Marcado)

Restauración de los Datos (IMDb)

Una vez conectado al servidor, abre una nueva consulta (Query) y ejecuta el siguiente script para cargar los 12 millones de registros:

```
USE [master];
GO

-- 1. Eliminar versiones previas si existen
DROP DATABASE IF EXISTS [IMDb_Source];
GO

-- 2. Restaurar Base de Datos
RESTORE DATABASE [IMDb_Source]
FROM DISK = N'/var/opt/mssql/backups/IMDb_Source_v2.bak'
WITH 
    MOVE N'IMDb_Source' TO N'/var/opt/mssql/data/IMDb_Source.mdf',
    MOVE N'IMDb_Source_log' TO N'/var/opt/mssql/data/IMDb_Source_log.ldf',
    REPLACE, 
    STATS = 5;
GO
```

Notas Importantes

    Puerto 1434: Se utiliza este puerto para evitar conflictos si ya tienes otra instancia de SQL Server en el puerto 1433.

    Persistencia: Los datos se guardan en la carpeta local ./data. Si borras el contenedor, los datos no se pierden.

    Espacio en Disco: Asegúrate de tener al menos 25 GB libres antes de iniciar la restauración.