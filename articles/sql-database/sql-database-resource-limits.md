---
title: Límites de recursos de Base de datos SQL
description: En esta página se describen algunos límites de recursos comunes para Base de datos SQL de Azure.
services: sql-database
documentationcenter: na
author: CarlRabeler
manager: jhubbard
editor: monicar

ms.service: sql-database
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-management
ms.date: 07/19/2016
ms.author: carlrab

---
# Límites de recursos de Base de datos SQL
## Información general
La Base de datos SQL de Azure administra los recursos disponibles para una base de datos mediante dos mecanismos diferentes: **Regulación de recursos** y **Aplicación de límites**. Este tema explica estas dos áreas principales de la administración de recursos.

## Regulador de recursos
Uno de los objetivos de diseño de los niveles de servicio Basic, Standard y Premium es que la Base de datos SQL de Azure se comporte como si la base de datos se ejecutase en su propio equipo, completamente aislada de otras bases de datos. La regulación de recursos emula este comportamiento. Si el uso de recursos agregados alcanza el máximo de CPU disponible, memoria, E/S de registro y los recursos de E/S de datos asignados a la base de datos, el regulador de recursos pondrá en cola las consultas en ejecución y asignará recursos a las consultas en cola a medida que se vayan liberando.

Como en un equipo dedicado, el uso de todos los recursos disponibles, provocará una ejecución más larga de las consultas en ejecución, lo que a su vez puede provocar la finalización del tiempo de espera de los comandos en el cliente. Las aplicaciones con lógica de reintento agresivo y aplicaciones que ejecutan consultas en la base de datos con una frecuencia alta, pueden encontrar mensajes de error al intentar ejecutar nuevas consultas cuando se ha alcanzado el límite de solicitudes simultáneas.

### Recomendaciones:
Supervisar el uso de recursos, así como los tiempos de respuesta promedio de las consultas cuando se aproxime al uso máximo de una base de datos. Cuando encuentre latencias de consulta más elevadas generalmente tienen tres opciones:

1. Reducir la cantidad de solicitudes entrantes a la base de datos para evitar que el tiempo de espera finalice y las solicitudes se apilen.
2. Asignar un nivel de rendimiento superior a la base de datos.
3. Optimizar las consultas para reducir el uso de recursos de cada consulta. Para obtener más información, consulte la sección de sugerencias/optimizació de consulta en el artículo de Guía de rendimiento de Base de datos SQL de Azure.

## Cumplimiento de límites
Al denegar nuevas solicitudes cuando se alcanzan los límites, se aplican recursos otros recursos diferentes de la CPU, memoria, registro de E/S y datos de E/S. Los clientes recibirán un [mensaje de error](sql-database-develop-error-messages.md) dependiendo del límite que alcance.

Por ejemplo, el número de conexiones a una base de datos SQL, así como el número de solicitudes simultáneas que se pueden procesar está restringido. La Base de datos SQL permite que el número de conexiones a la base de datos sea mayor que el número de solicitudes simultáneas para admitir la agrupación de conexiones. Mientras que la cantidad de conexiones que están disponibles fácilmente puede controlarse mediante la aplicación, la cantidad de solicitudes paralelas suele ser más difícil de calcular y controlar. Los errores se producen especialmente durante las cargas máximas cuando la aplicación envía demasiadas solicitudes o la base de datos alcanza su límite de recursos y empieza a amontonar los subprocesos de trabajo debido a consultas de ejecución más largas.

## Niveles de servicio y niveles de rendimiento
Existen niveles de servicio y rendimiento para los grupos elásticos y los grupos de bases de datos independientes.

### Bases de datos independientes
Para una base de datos independiente, los límites de una base de datos se definen mediante el nivel de servicio de base de datos el nivel de rendimiento. En la tabla siguiente se describen las características de las bases de datos de nivel Básico, Estándar y Premium en distintos niveles de rendimiento.

[!INCLUDE [Tabla de niveles de servicio de SQL Database](../../includes/sql-database-service-tiers-table.md)]

### Grupos elásticos
Los [grupos elásticos](sql-database-elastic-pool.md) comparten recursos entre bases de datos del grupo. En la tabla siguiente se describen las características de los grupos de bases de datos elásticas de nivel Básico, Estándar y Premium.

[!INCLUDE [Tabla de niveles de servicio de Base de datos SQL para bases de datos elásticas](../../includes/sql-database-service-tiers-table-elastic-db-pools.md)]

Para obtener una definición expandida de cada recurso enumerado en las tablas anteriores, consulte las descripciones en [Límites y capacidades de nivel de servicio](sql-database-performance-guidance.md#service-tier-capabilities-and-limits). Para obtener una descripción general de los niveles de servicio, consulte [Niveles de servicio y niveles de rendimiento de Base de datos SQL de Azure](sql-database-service-tiers.md).

## Otros límites de Base de datos SQL
| Ámbito | Límite | Description |
| --- | --- | --- |
| Bases de datos que usan exportación automatizada por suscripción |10 |La exportación automatizada le permite crear una programación personalizada para realizar copias de seguridad de las bases de datos SQL. Para obtener más información, consulte [Bases de datos SQL: compatibilidad con exportaciones automatizadas de Base de datos SQL](http://weblogs.asp.net/scottgu/windows-azure-july-updates-sql-database-traffic-manager-autoscale-virtual-machines). |
| Base de datos por servidor |Hasta 5000 |Se permiten hasta 5000 bases de datos por servidor en servidores V12. |
| DTU por servidor |45000 |Hay disponibles 45000 DTU por servidor en servidores V12 para el aprovisionamiento de las bases de datos, los grupos elásticos y los almacenes de datos. |

## Recursos
[Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../azure-subscription-service-limits.md)

[Niveles de servicio y niveles de rendimiento de la Base de datos SQL de Azure](sql-database-service-tiers.md)

[Mensajes de error para los programas de cliente de base de datos SQL](sql-database-develop-error-messages.md)

<!---HONumber=AcomDC_0914_2016-->