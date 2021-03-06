---
title: "Realización de una consulta en el Índice de Azure Search mediante el SDK para .NET | Microsoft Docs"
description: "Cree una consulta de búsqueda en Búsqueda de Azure y use los parámetros de búsqueda para filtrar y ordenar los resultados de búsqueda."
services: search
manager: jhubbard
documentationcenter: 
author: brjohnstmsft
ms.assetid: 12c3efba-ea99-4187-9d2d-f63b5ec7040d
ms.service: search
ms.devlang: dotnet
ms.workload: search
ms.topic: get-started-article
ms.tgt_pltfrm: na
ms.date: 08/29/2016
ms.author: brjohnst
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: f85c3a0d3bb9fb61802ba3ce070ead2e650a29cc


---
# <a name="query-your-azure-search-index-using-the-net-sdk"></a>Consultas del índice de Búsqueda de Azure con el SDK de .NET
> [!div class="op_single_selector"]
> * [Información general](search-query-overview.md)
> * [Portal](search-explorer.md)
> * [.NET](search-query-dotnet.md)
> * [REST](search-query-rest-api.md)
> 
> 

En este artículo se muestra cómo realizar una consulta en un índice con el [SDK de .NET de Búsqueda de Azure](https://msdn.microsoft.com/library/azure/dn951165.aspx).

Antes de comenzar este tutorial, debe haber [creado ya un índice de Azure Search](search-what-is-an-index.md) y [haberlo rellenado con datos](search-what-is-data-import.md).

Tenga en cuenta que todo el código de ejemplo de este artículo está escrito en C#. El código fuente completo se puede encontrar [en GitHub](http://aka.ms/search-dotnet-howto).

## <a name="i-identify-your-azure-search-services-query-apikey"></a>I. Identificación de la clave de API de consulta del servicio de Búsqueda de Azure
Ahora que ha creado un índice de Búsqueda de Azure, está casi listo para emitir consultas mediante el SDK de .NET. En primer lugar, debe obtener una de las claves de API de consulta que se generaron para aprovisionar el servicio de búsqueda. El SDK para .NET enviará esta clave de API en cada solicitud al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

1. Para buscar las claves de API del servicio debe iniciar sesión en el [Portal de Azure](https://portal.azure.com/)
2. Vaya a la hoja de servicio de Búsqueda de Azure
3. Haga clic en el icono "Claves"

El servicio tendrá *claves de administración* y *claves de consulta*.

* Sus *claves de administración* principal y secundaria permiten conceder derechos completos para todas las operaciones, incluida la capacidad para administrar el servicio, crear y eliminar índices, indexadores y orígenes de datos. Existen dos claves, de forma que puede usar la clave secundaria si decide volver a generar la clave principal y viceversa.
* Las *claves de consulta* conceden acceso de solo lectura a índices y documentos y normalmente se distribuyen entre las aplicaciones cliente que emiten solicitudes de búsqueda.

Para consultar un índice, puede utilizar una de las claves de consulta. Las claves de administración también se pueden utilizar para las consultas, pero debe utilizar una clave de consulta en el código de aplicación ya que esto se adapta mejor al [principio de privilegios mínimos](https://en.wikipedia.org/wiki/Principle_of_least_privilege).

## <a name="ii-create-an-instance-of-the-searchindexclient-class"></a>II. Creación de una instancia de la clase SearchIndexClient
Para emitir consultas con el SDK de .NET de Búsqueda de Azure, necesitará crear una instancia de la clase `SearchIndexClient` . Esta clase tiene varios constructores. El que desea tiene el nombre del servicio de búsqueda, el nombre del índice y un objeto `SearchCredentials` como parámetros. `SearchCredentials` incluye su clave de API.

El código siguiente crea un nuevo `SearchIndexClient` para el índice "hoteles" (creado en [Creación de un índice de Azure Search mediante el SDK de .NET](search-create-index-dotnet.md)) con valores para el nombre de servicio de búsqueda y la clave de API que se almacenan en el archivo de configuración de la aplicación (`app.config` o `web.config`):

```csharp
string searchServiceName = ConfigurationManager.AppSettings["SearchServiceName"];
string queryApiKey = ConfigurationManager.AppSettings["SearchServiceQueryApiKey"];

SearchIndexClient indexClient = new SearchIndexClient(searchServiceName, "hotels", new SearchCredentials(queryApiKey));
```

`SearchIndexClient` tiene una propiedad `Documents`. Esta propiedad proporciona todos los métodos que necesita para consultar los índices de Búsqueda de Azure.

## <a name="iii-query-your-index"></a>III. Consulta del índice
Buscar con el SDK de .NET es tan sencillo como llamar al método `Documents.Search` en su `SearchIndexClient`. Este método usa unos cuantos parámetros, incluido el texto de búsqueda, además de un objeto `SearchParameters` que se puede usar para refinar la consulta.

#### <a name="types-of-queries"></a>Tipos de consultas
Los dos [tipos de consultas](search-query-overview.md#types-of-queries) principales que usará son `search` y `filter`. Una consulta `search` realiza búsquedas de uno o más términos en todos los campos *habilitados para búsqueda* del índice. Una consulta `filter` evalúa una expresión booleana en todos los campos *filtrables* de un índice.

Tanto los filtros como las búsquedas se realizan con el método `Documents.Search` . Se puede pasar una consulta de búsqueda en el parámetro `searchText`, mientras que una expresión de filtro se puede pasar en la propiedad `Filter` de la clase `SearchParameters`. Para filtrar sin buscar, solo tiene que pasar `"*"` al parámetro `searchText`. Para buscar sin filtrar, simplemente deje la propiedad `Filter` sin establecer, o no pase ninguna instancia de `SearchParameters`.

#### <a name="example-queries"></a>Consultas de ejemplo
El código de ejemplo siguiente muestra algunas formas de consultar el índice "hoteles" definido en [Creación de un índice de Búsqueda de Azure con el SDK de .NET](search-create-index-dotnet.md#DefineIndex). Tenga en cuenta que los documentos que se devuelven con los resultados de búsqueda son instancias de la clase `Hotel` , que se definió en [Importación de datos en Búsqueda de Azure con el SDK de .NET](search-import-data-dotnet.md#HotelClass). El código de ejemplo usa un método `WriteDocuments` para enviar los resultados de búsqueda a la consola. Este método se describe en la sección siguiente.

```csharp
SearchParameters parameters;
DocumentSearchResult<Hotel> results;

Console.WriteLine("Search the entire index for the term 'budget' and return only the hotelName field:\n");

parameters =
    new SearchParameters()
    {
        Select = new[] { "hotelName" }
    };

results = indexClient.Documents.Search<Hotel>("budget", parameters);

WriteDocuments(results);

Console.Write("Apply a filter to the index to find hotels cheaper than $150 per night, ");
Console.WriteLine("and return the hotelId and description:\n");

parameters =
    new SearchParameters()
    {
        Filter = "baseRate lt 150",
        Select = new[] { "hotelId", "description" }
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);

Console.Write("Search the entire index, order by a specific field (lastRenovationDate) ");
Console.Write("in descending order, take the top two results, and show only hotelName and ");
Console.WriteLine("lastRenovationDate:\n");

parameters =
    new SearchParameters()
    {
        OrderBy = new[] { "lastRenovationDate desc" },
        Select = new[] { "hotelName", "lastRenovationDate" },
        Top = 2
    };

results = indexClient.Documents.Search<Hotel>("*", parameters);

WriteDocuments(results);

Console.WriteLine("Search the entire index for the term 'motel':\n");

parameters = new SearchParameters();
results = indexClient.Documents.Search<Hotel>("motel", parameters);

WriteDocuments(results);
```

## <a name="iv-handle-search-results"></a>IV. Control de los resultados de la búsqueda
El método `Documents.Search` devuelve un objeto `DocumentSearchResult` que contiene los resultados de la consulta. En el ejemplo de la sección anterior, se usa un método llamado `WriteDocuments` para enviar los resultados de búsqueda a la consola:

```csharp
private static void WriteDocuments(DocumentSearchResult<Hotel> searchResults)
{
    foreach (SearchResult<Hotel> result in searchResults.Results)
    {
        Console.WriteLine(result.Document);
    }

    Console.WriteLine();
}
```

Este es el aspecto de los resultados de las consultas de la sección anterior, suponiendo que el índice "hoteles" contenga los datos de ejemplo de [Importación de datos en Búsqueda de Azure con el SDK de .NET](search-import-data-dotnet.md):

```
Search the entire index for the term 'budget' and return only the hotelName field:

Name: Roach Motel

Apply a filter to the index to find hotels cheaper than $150 per night, and return the hotelId and description:

ID: 2   Description: Cheapest hotel in town
ID: 3   Description: Close to town hall and the river

Search the entire index, order by a specific field (lastRenovationDate) in descending order, take the top two results, and show only hotelName and lastRenovationDate:

Name: Fancy Stay        Last renovated on: 6/27/2010 12:00:00 AM +00:00
Name: Roach Motel       Last renovated on: 4/28/1982 12:00:00 AM +00:00

Search the entire index for the term 'motel':

ID: 2   Base rate: 79.99        Description: Cheapest hotel in town     Description (French): Hôtel le moins cher en ville      Name: Roach Motel       Category: Budget        Tags: [motel, budget]   Parking included: yes   Smoking allowed: yes    Last renovated on: 4/28/1982 12:00:00 AM +00:00 Rating: 1/5     Location: Latitude 49.678581, longitude -122.131577

```

El código de ejemplo anterior envía los resultados de búsqueda a la consola. Del mismo modo, necesitará mostrar los resultados de búsqueda en su propia aplicación. Consulte [este ejemplo en GitHub](https://github.com/Azure-Samples/search-dotnet-getting-started/tree/master/DotNetSample) para ver cómo representar los resultados de búsqueda en una aplicación web basada en ASP.NET MVC.




<!--HONumber=Nov16_HO2-->


