---
title: Uso de Azure Redis Cache | Microsoft Docs
description: "Obtener más información acerca de cómo mejorar el rendimiento de sus aplicaciones de Azure con Caché en Redis de Azure"
services: redis-cache,app-service
documentationcenter: 
author: steved0x
manager: douge
editor: 
ms.assetid: c502f74c-44de-4087-8303-1b1f43da12d5
ms.service: cache
ms.workload: tbd
ms.tgt_pltfrm: cache-redis
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 08/25/2016
ms.author: sdanie
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 209d4f610f0d5199d9018c506acef3b7328478ef


---
# <a name="how-to-use-azure-redis-cache"></a>Uso de Caché en Redis de Azure
> [!div class="op_single_selector"]
> * [.NET](cache-dotnet-how-to-use-azure-redis-cache.md)
> * [ASP.NET](cache-web-app-howto.md)
> * [Node.js](cache-nodejs-get-started.md)
> * [Java](cache-java-get-started.md)
> * [Python](cache-python-get-started.md)
> 
> 

En esta guía se muestra cómo empezar a usar **Caché en Redis de Azure**. Caché en Redis de Microsoft Azure se basa en la conocida Caché en Redis de código fuente abierto. Le proporciona acceso a una caché en Redis segura y dedicada, administrada por Microsoft. Una caché creada usando Caché en Redis de Azure es accesible desde cualquier aplicación dentro de Microsoft Azure.

Caché en Redis de Microsoft Azure está disponible en los siguientes niveles:

* **Básico** – Nodo único. Varios tamaños de hasta 53 GB.
* **Estándar**: principal/réplica de dos nodos. Varios tamaños de hasta 53 GB. Contrato de nivel de servicio del 99,9 %.
* **Premium** : principal/réplica de dos nodos con hasta 10 particiones. Varios tamaños, desde 6 GB a 530 GB (póngase en contacto con nosotros para obtener más información). Todas las características del nivel Estándar y algunas otras características son compatibles con los [clústeres de Redis](cache-how-to-premium-clustering.md), la [persistencia de Redis](cache-how-to-premium-persistence.md) y [Azure Virtual Network](cache-how-to-premium-vnet.md). Contrato de nivel de servicio del 99,9 %.

Estos niveles difieren en las características y el precio. Para información sobre los precios, consulte los [precios de Redis Cache][precios de Redis Cache].

En esta guía se explica cómo usar el cliente [StackExchange.Redis][StackExchange.Redis] con código C\#. Entre los escenarios tratados, se incluye la **creación y configuración de una memoria caché**, la **configuración de clientes de caché** y la **adición y eliminación de objetos de la memoria caché**. Para más información acerca del uso de Azure Redis Cache, consulte la sección [Pasos siguientes][Pasos siguientes]. Para obtener un tutorial paso a paso de creación de una aplicación web ASP.NET MVC con Caché en Redis, consulte [Creación de una aplicación web con Caché en Redis](cache-web-app-howto.md).

<a name="getting-started-cache-service"></a>

## <a name="get-started-with-azure-redis-cache"></a>Introducción a Caché en Redis de Azure
Ponerse en marcha con Caché en Redis de Azure es fácil. En primer lugar, tiene que aprovisionar y configurar una caché. A continuación, debe configurar los clientes de caché para que puedan obtener acceso a la caché. Una vez que los clientes de caché estén configurados, ya puede empezar a trabajar con ellos.

* [Creación de una caché][Creación de una caché]
* [Configuración de los clientes de caché][Configuración de los clientes de caché]

<a name="create-cache"></a>

## <a name="create-a-cache"></a>Creación de una caché
[!INCLUDE [redis-cache-create](../../includes/redis-cache-create.md)]

### <a name="to-access-your-cache-after-its-created"></a>Para acceder a la memoria caché una vez creada
[!INCLUDE [redis-cache-create](../../includes/redis-cache-browse.md)]

Para más información acerca de cómo configurar la memoria caché, consulte [Configuración de Caché en Redis de Azure](cache-configure.md).

<a name="NuGet"></a>

## <a name="configure-the-cache-clients"></a>Configuración de los clientes de caché
[!INCLUDE [redis-cache-configure](../../includes/redis-cache-configure-stackexchange-redis-nuget.md)]

Cuando el proyecto de cliente ya está configurado para el almacenamiento en caché, puede usar las técnicas descritas en las siguientes secciones para trabajar con la caché.

<a name="working-with-caches"></a>

## <a name="working-with-caches"></a>Trabajo con cachés
En esta sección se describe cómo realizar tareas comunes con el servicio de caché.

* [Conexión a la memoria caché][Conexión a la memoria caché]
* [Agregación y recuperación de objetos de la memoria caché][Agregación y recuperación de objetos de la memoria caché]
* [Trabajar con objetos .NET en la memoria caché](#work-with-net-objects-in-the-cache)

<a name="connect-to-cache"></a>

## <a name="connect-to-the-cache"></a>Conexión a la memoria caché
Para trabajar con una caché mediante programación, necesita una referencia a la misma. Agregue lo siguiente a la parte superior de cualquier archivo del que desea usar el cliente StackExchange.Redis para acceder a una Caché en Redis de Azure.

    using StackExchange.Redis;

> [!NOTE]
> El cliente StackExchange.Redis requiere .NET Framework 4 o superior.
> 
> 

La clase `ConnectionMultiplexer` administra la conexión con Caché en Redis de Azure. Esta clase está diseñada para compartirse y reusarse a través de su aplicación cliente y no necesita crearse basándose en operación. 

Para conectarse a una Caché en Redis de Azure y que se devuelva una instancia de `ConnectionMultiplexer`, llame al método estático `Connect` y pase el extremo y la clave de caché como en el siguiente ejemplo. Use la clave generada desde el Portal de Azure como parámetro de contraseña.

    ConnectionMultiplexer connection = ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");

> [!IMPORTANT]
> Advertencia: nunca almacene credenciales en el código fuente. Para que este ejemplo resulte sencillo, se muestra en código fuente. Consulte [How Application Strings and Connection Strings Work][How Application Strings and Connection Strings Work] (Cómo funcionan las cadenas de aplicación y las cadenas de conexión) para información sobre cómo almacenar credenciales.
> 
> 

Si no desea usar SSL, establezca `ssl=false` u omita el parámetro `ssl`.

> [!NOTE]
> El puerto no SSL está deshabilitado de forma predeterminada para las cachés nuevas. Para obtener instrucciones acerca de cómo habilitar el puerto no SSL, consulte [Puertos de acceso](cache-configure.md#access-ports).
> 
> 

Un enfoque para compartir una instancia `ConnectionMultiplexer` en su aplicación es tener una propiedad estática que devuelva una instancia conectada, similar al ejemplo siguiente. Esto proporciona una manera segura para subprocesos para inicializar una sola instancia `ConnectionMultiplexer` conectada. En estos ejemplos `abortConnect` está establecida en falso, lo que significa que la llamada se realizará correctamente incluso si no se establece ninguna conexión a la Caché en Redis de Azure. Una característica clave de `ConnectionMultiplexer` es que restaurará automáticamente la conectividad a la memoria caché una vez que el problema de red u otras causas se hayan resuelto.

    private static Lazy<ConnectionMultiplexer> lazyConnection = new Lazy<ConnectionMultiplexer>(() =>
    {
        return ConnectionMultiplexer.Connect("contoso5.redis.cache.windows.net,abortConnect=false,ssl=true,password=...");
    });

    public static ConnectionMultiplexer Connection
    {
        get
        {
            return lazyConnection.Value;
        }
    }

Para más información sobre las opciones de configuración de conexión avanzadas, consulte [StackExchange.Redis configuration model][StackExchange.Redis configuration model] (Modelo de configuración de StackExchange.Redis).

[!INCLUDE [redis-cache-create](../../includes/redis-cache-access-keys.md)]

Una vez establecida la conexión, devuelva una referencia a la base de datos de caché en Redis llamando al método `ConnectionMultiplexer.GetDatabase` . El objeto devuelto desde el método `GetDatabase` es un objeto de paso a través ligero y no necesita almacenarse.

    // Connection refers to a property that returns a ConnectionMultiplexer
    // as shown in the previous example.
    IDatabase cache = Connection.GetDatabase();

    // Perform cache operations using the cache object...
    // Simple put of integral data types into the cache
    cache.StringSet("key1", "value");
    cache.StringSet("key2", 25);

    // Simple get of data types from the cache
    string key1 = cache.StringGet("key1");
    int key2 = (int)cache.StringGet("key2");

Ahora que sabe cómo conectarse a una instancia de Caché en Redis de Azure y devolver una referencia a la base de datos de caché, echemos un vistazo a cómo se trabaja con la memoria caché.

<a name="add-object"></a>

## <a name="add-and-retrieve-objects-from-the-cache"></a>Agregación y recuperación de objetos de la memoria caché
Los elementos se pueden almacenar en una memoria caché y recuperarse de esta usando los métodos `StringSet` y `StringGet`.

    // If key1 exists, it is overwritten.
    cache.StringSet("key1", "value1");

    string value = cache.StringGet("key1");

Redis almacena la mayoría de los datos como cadenas Redis, pero estas cadenas pueden contener muchos tipos de datos, como por ejemplo datos binarios serializados, que se pueden usar cuando se almacenan objetos .NET en caché.

Cuando llame a `StringGet`, si el objeto existe, se devuelve y, si no existe, se devuelve `null`. En este caso, puede recuperar el valor desde el origen de datos que desee y almacenarlo en la memoria caché para su uso posterior. Esto se conoce como patrón cache-aside.

    string value = cache.StringGet("key1");
    if (value == null)
    {
        // The item keyed by "key1" is not in the cache. Obtain
        // it from the desired data source and add it to the cache.
        value = GetValueFromDataSource();

        cache.StringSet("key1", value);
    }

Para especificar la expiración de un elemento en la memoria caché, use el parámetro `TimeSpan` de `StringSet`.

    cache.StringSet("key1", "value1", TimeSpan.FromMinutes(90));

## <a name="work-with-net-objects-in-the-cache"></a>Trabajar con objetos .NET en la memoria caché
Caché en Redis de Azure puede almacenar en caché objetos .NET así como tipos de datos primitivos, pero antes de poder almacenar un objeto .NET en caché, se debe serializar. Esta es la responsabilidad del desarrollador de la aplicación, que tiene total flexibilidad a la hora de elegir el serializador.

Una manera sencilla para serializar objetos es usar los métodos de serialización `JsonConvert` en [Newtonsoft.Json.NET](https://www.nuget.org/packages/Newtonsoft.Json/8.0.1-beta1) y serializar a y desde JSON. En el ejemplo siguiente se muestra un get y set con una instancia de objeto `Employee` .

    class Employee
    {
        public int Id { get; set; }
        public string Name { get; set; }

        public Employee(int EmployeeId, string Name)
        {
            this.Id = EmployeeId;
            this.Name = Name;
        }
    }

    // Store to cache
    cache.StringSet("e25", JsonConvert.SerializeObject(new Employee(25, "Clayton Gragg")));

    // Retrieve from cache
    Employee e25 = JsonConvert.DeserializeObject<Employee>(cache.StringGet("e25"));

<a name="next-steps"></a>

## <a name="next-steps"></a>Pasos siguientes
Ahora que está familiarizado con los aspectos básicos, siga estos vínculos para obtener más información sobre Caché en Redis de Azure.

* Consulte los proveedores de ASP.NET para Caché en Redis de Azure.
  * [Proveedor de estado de sesión de Redis de Azure](cache-aspnet-session-state-provider.md)
  * [Proveedor de caché de resultados de ASP.NET de caché en Redis de Azure](cache-aspnet-output-cache-provider.md)
* [Habilite los diagnósticos de cache](cache-how-to-monitor.md#enable-cache-diagnostics) para que pueda [supervisar](cache-how-to-monitor.md) el estado de la memoria caché. Puede ver las métricas en el Portal de Azure y también [descargarlas y revisarlas](https://github.com/rustd/RedisSamples/tree/master/CustomMonitoring) mediante el uso de las herramientas que prefiera.
* Consulte la [documentación del cliente de caché StackExchange.Redis][documentación del cliente de caché StackExchange.Redis].
  * Se puede obtener acceso a Caché en Redis de Azure desde numerosos clientes Redis e idiomas de desarrollo. Para más información, consulte [http://redis.io/clients][http://redis.io/clients].
* Caché en Redis de Azure puede utilizarse también con herramientas como Redsmin y Redis Desktop Manager y servicios de terceros.
  * Para más información sobre Redsmin, consulte [How to retrieve an Azure Redis connection string and use it with Redsmin][How to retrieve an Azure Redis connection string and use it with Redsmin] (Cómo recuperar una cadena de conexión de Azure Redis y usarla con Redsmin).
  * Acceda e inspeccione los datos en caché en Redis de Azure con una GUI mediante [RedisDesktopManager](https://github.com/uglide/RedisDesktopManager).
* Consulte la documentación de [Redis][Redis] y lea sobre los [tipos de datos de Redis][tipos de datos de Redis] y [una introducción de quince minutos sobre los tipos de datos de Redis][una introducción de quince minutos sobre los tipos de datos de Redis].

<!-- INTRA-TOPIC LINKS -->
[Next Steps]: #next-steps
[Introducción a Azure Redis Cache (vídeo)]: #video
[¿Qué es Azure Redis Cache?]: #what-is
[Creación de una memoria caché de Azure]: #create-cache
[¿Qué tipo de almacenamiento en caché es el adecuado para mí?]: #choosing-cache
[Preparación del proyecto de Visual Studio para usar el almacenamiento en caché de Azure]: #prepare-vs
[Configuración de la aplicación para usar el almacenamiento en caché]: #configure-app
[Introducción a Caché en Redis de Azure]: #getting-started-cache-service
[Creación de una caché]: #create-cache
[Configuración de la memoria caché]: #enable-caching
[Configuración de los clientes de caché]: #NuGet
[Trabajo con cachés]: #working-with-caches
[Conexión a la memoria caché]: #connect-to-cache
[Agregación y recuperación de objetos de la memoria caché]: #add-object
[Especificación de la expiración de un objeto en la memoria caché]: #specify-expiration
[Almacenamiento del estado de la sesión de ASP.NET en la memoria caché]: #store-session


<!-- IMAGES -->


[StackExchangeNuget]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-stackexchange-redis.png

[NuGetMenu]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-nuget-menu.png

[CacheProperties]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-properties.png

[ManageKeys]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-manage-keys.png

[SessionStateNuGet]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-session-state-provider.png

[BrowseCaches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-browse-caches.png

[Caches]: ./media/cache-dotnet-how-to-use-azure-redis-cache/redis-cache-caches.png







<!-- LINKS -->
[http://redis.io/clients]: http://redis.io/clients
[Desarrollo en otros lenguajes para Azure Redis Cache]: http://msdn.microsoft.com/library/azure/dn690470.aspx
[How to retrieve an Azure Redis connection string and use it with Redsmin]: https://redsmin.uservoice.com/knowledgebase/articles/485711-how-to-connect-redsmin-to-azure-redis-cache
[Proveedor de estado de sesión de Redis de Azure]: http://go.microsoft.com/fwlink/?LinkId=398249
[Configuración de un cliente de caché mediante programación]: http://msdn.microsoft.com/library/windowsazure/gg618003.aspx
[Proveedor de estado de sesión para la caché de Azure]: http://go.microsoft.com/fwlink/?LinkId=320835
[Caché de Azure AppFabric: almacenamiento en caché del estado de sesión]: http://www.microsoft.com/showcase/details.aspx?uuid=87c833e9-97a9-42b2-8bb1-7601f9b5ca20
[Proveedor de la caché de resultados para la caché de Azure]: http://go.microsoft.com/fwlink/?LinkId=320837
[Azure Shared Caching]: http://msdn.microsoft.com/library/windowsazure/gg278356.aspx
[Blog del equipo]: http://blogs.msdn.com/b/windowsazure/
[Almacenamiento en caché de Azure]: http://www.microsoft.com/showcase/Search.aspx?phrase=azure+caching
[Configuración de tamaños de la máquina virtual]: http://go.microsoft.com/fwlink/?LinkId=164387
[Consideraciones sobre el planeamiento de la capacidad de almacenamiento en la caché de Azure]: http://go.microsoft.com/fwlink/?LinkId=320167
[Almacenamiento en caché de Azure]: http://go.microsoft.com/fwlink/?LinkId=252658
[Establecimiento de la capacidad de almacenamiento en caché de una página ASP.NET mediante declaración]: http://msdn.microsoft.com/library/zd1ysf1y.aspx
[Establecimiento de la capacidad de almacenamiento en caché de una página mediante programación]: http://msdn.microsoft.com/library/z852zf6b.aspx
[Configuración de una memoria caché en Azure Redis Cache]: http://msdn.microsoft.com/library/azure/dn793612.aspx

[StackExchange.Redis configuration model]: http://github.com/StackExchange/StackExchange.Redis/blob/master/Docs/Configuration.md

[Trabajar con objetos .NET en la memoria caché]: http://msdn.microsoft.com/library/dn690521.aspx#Objects


[NuGet Package Manager Installation]: http://go.microsoft.com/fwlink/?LinkId=240311
[precios de Redis Cache]: http://www.windowsazure.com/pricing/details/cache/
[Portal de Azure]: https://portal.azure.com/

[Información general de Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=320830
[Caché en Redis de Azure]: http://go.microsoft.com/fwlink/?LinkId=398247

[Migración a Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=317347
[Ejemplos de Azure Redis Cache]: http://go.microsoft.com/fwlink/?LinkId=320840
[Uso de grupos de recursos para administrar los recursos de Azure]: ../azure-resource-manager/resource-group-overview.md

[StackExchange.Redis]: http://github.com/StackExchange/StackExchange.Redis
[documentación del cliente de caché StackExchange.Redis]: http://github.com/StackExchange/StackExchange.Redis#documentation

[Redis]: http://redis.io/documentation
[Tipos de datos de Redis]: http://redis.io/topics/data-types
[una introducción de quince minutos sobre los tipos de datos de Redis]: http://redis.io/topics/data-types-intro

[How Application Strings and Connection Strings Work]: http://azure.microsoft.com/blog/2013/07/17/windows-azure-web-sites-how-application-strings-and-connection-strings-work/





<!--HONumber=Nov16_HO2-->


