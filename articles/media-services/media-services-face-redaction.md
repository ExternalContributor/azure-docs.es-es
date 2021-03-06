---
title: Censura de rostros con Análisis multimedia de Azure | Microsoft Docs
description: Este tema muestra cómo censurar caras con Análisis multimedia de Azure.
services: media-services
documentationcenter: ''
author: juliako
manager: erikre
editor: ''

ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: dotnet
ms.topic: article
ms.date: 09/12/2016
ms.author: juliako;

---
# Censura de rostros con Análisis multimedia de Azure
## Información general
**Redactor multimedia de Azure** es un procesador multimedia (MP) de [Análisis multimedia de Azure](media-services-analytics-overview.md) que ofrece censura de rostros escalable en la nube. La censura de rostros le permite modificar un vídeo con el fin de difuminar las caras de personas seleccionadas. Puede usar el servicio de censura de rostros en escenarios de seguridad pública y de noticias en los medios de comunicación. Unos minutos de material de archivo que contenga varias caras puede tardar horas en censurarse manualmente, pero con este servicio, el proceso de censura de caras requiere solamente unos pocos pasos sencillos. Para más información, vea [este blog](https://azure.microsoft.com/blog/azure-media-redactor/).

En este tema se proporcionan detalles sobre **Redactor multimedia de Azure** y se muestra cómo se usa con el SDK de Media Services para .NET.

El procesador de multimedia **Redactor multimedia de Azure** está actualmente en versión preliminar.

## Modos de censura de rostros
La censura facial funciona detectando caras en cada fotograma de vídeo y realizando un seguimiento del objeto de cara tanto hacia delante como hacia atrás en el tiempo, para que la imagen de la misma persona pueda difuminarse también desde otros ángulos. El proceso de censura automatizada es muy complejo y no siempre genera los resultados deseados al 100 %, por este motivo, Análisis multimedia de Azure le ofrece un par de métodos para modificar el resultado final.

Además de un modo totalmente automático, hay un flujo de trabajo de dos pasos que permite la selección y deselección de caras encontradas a través de una lista de identificadores. Igualmente, para realizar ajustes arbitrarios por fotograma, el procesador multimedia utiliza un archivo de metadatos en formato JSON. Este flujo de trabajo se divide en los modos **Analyze** (Analizar) y **Redact** (Censurar). Puede combinar los dos modos en un único paso que ejecuta las tareas de un trabajo; este modo se denomina **Combinado**.

### Modo combinado
Se generará un mp4 censurado automáticamente sin necesidad de intervención manual.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4 |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'combined'}} |
| Recurso de salida |foo\_redacted.mp4 |Vídeo con difuminado aplicado |

#### Ejemplo de entrada:
[Vea este vídeo](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fed99001d-72ee-4f91-9fc0-cd530d0adbbc%2FDancing.mp4)

#### Ejemplo de salida:
[Vea este vídeo](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fc6608001-e5da-429b-9ec8-d69d8f3bfc79%2Fdance_redacted.mp4)

### Modo Analyze (Análisis)
El paso **analizar** del flujo de trabajo de dos pasos toma una entrada de vídeo y genera un archivo JSON de ubicaciones de rostros e imágenes jpg de cada rostro detectado.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4 |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Recurso de salida |foo\_annotations.JSON |Datos de anotación de ubicaciones de rostros en formato JSON. El usuario puede editarlo para modificar los rectángulos de selección para el difuminado. Vea el ejemplo a continuación. |
| Recurso de salida |foo\_thumb%06d.jpg [foo\_thumb000001.jpg, foo\_thumb000002.jpg] |Un jpg recortado de cada rostro detectado, donde el número indica el identificador (labelId) de la cara |

#### Ejemplo de salida:
    {
      "version": 1,
      "timescale": 50,
      "offset": 0,
      "framerate": 25.0,
      "width": 1280,
      "height": 720,
      "fragments": [
        {
          "start": 0,
          "duration": 2,
          "interval": 2,
          "events": [
            [  
              {
                "id": 1,
                "x": 0.306415737,
                "y": 0.03199235,
                "width": 0.15357475,
                "height": 0.322126418
              },
              {
                "id": 2,
                "x": 0.5625317,
                "y": 0.0868245438,
                "width": 0.149155334,
                "height": 0.355517566
              }
            ]
          ]
        },

... truncado

### Modo Redact (Censurar)
El segundo paso del flujo de trabajo tiene un mayor número de entradas que tienen que combinarse en un solo recurso.

Esto incluye una lista de identificadores para difuminar, el vídeo original y las anotaciones JSON. Este modo usa las anotaciones para aplicar difuminado en el vídeo de entrada.

El resultado de la fase de análisis no incluye el vídeo original. El vídeo tiene que estar cargado en el recurso de entrada para la tarea de modo Redact (Censurar) y seleccionado como el archivo principal.

| Fase | Nombre de archivo | Notas |
| --- | --- | --- |
| Recurso de entrada |foo.bar |Vídeo en formato WMV, MOV o MP4. El mismo vídeo que en el paso 1. |
| Recurso de entrada |foo\_annotations.JSON |archivo de metadatos de anotaciones de la fase uno, con modificaciones opcionales. |
| Recurso de entrada |foo\_IDList.txt (opcional) |Nueva lista opcional separada por líneas de identificadores de rostro para censurar. Si se deja en blanco, se difuminarán todas las caras. |
| Configuración de entrada |Configuración predeterminada de trabajo |{'version':'1.0', 'options': {'mode':'analyze'}} |
| Recurso de salida |foo\_redacted.mp4 |Vídeo con difuminado aplicado en base a las anotaciones |

#### Ejemplo de salida
Este es el resultado de una lista de identificadores con un identificador seleccionado.

[Vea este vídeo](http://ampdemo.azureedge.net/?url=http%3A%2F%2Freferencestream-samplestream.streaming.mediaservices.windows.net%2Fad6e24a2-4f9c-46ee-9fa7-bf05e20d19ac%2Fdance_redacted1.mp4)

## Descripciones de atributos
El procesador multimedia de censura proporciona detección de ubicación y seguimiento de rostros de alta precisión que puede detectar hasta 64 caras humanas en un fotograma de vídeo. Las caras de frente ofrecen los mejores resultados, mientras que las que se encuentran de lado y las caras pequeñas (inferiores o iguales a 24x24 píxeles) podrían no ser tan precisas.

Los rostros que se han detectado y de los que se ha realizado seguimiento se devuelven con coordenadas que indican su ubicación, así como un número de identificación de rostro que indica el seguimiento de esa persona. Los números de identificación de cara son propensos a restablecerse en circunstancias en las que la cara de frente se pierde o se superpone en el fotograma, lo que provoca que a algunas personas se les asigne varios identificadores.

Para ver explicaciones detalladas de los atributos, consulte [Detección de caras y emociones con Análisis multimedia de Azure](media-services-face-and-emotion-detection.md).

## Código de ejemplo
El programa siguiente muestra cómo:

1. Crear un recurso y cargar un archivo multimedia en dicho recurso.
2. Crear un trabajo con una tarea de censura de rostros basándose en un archivo de configuración que contiene el siguiente valor predeterminado de JSON.
   
        {'version':'1.0', 'options': {'mode':'combined'}}
3. Descargar los archivos JSON de salida.
   
        using System;
        using System.Configuration;
        using System.IO;
        using System.Linq;
        using Microsoft.WindowsAzure.MediaServices.Client;
        using System.Threading;
        using System.Threading.Tasks;
   
        namespace FaceRedaction
        {
            class Program
            {
                // Read values from the App.config file.
                private static readonly string _mediaServicesAccountName =
                    ConfigurationManager.AppSettings["MediaServicesAccountName"];
                private static readonly string _mediaServicesAccountKey =
                    ConfigurationManager.AppSettings["MediaServicesAccountKey"];
   
                // Field for service context.
                private static CloudMediaContext _context = null;
                private static MediaServicesCredentials _cachedCredentials = null;
   
                static void Main(string[] args)
                {
   
                    // Create and cache the Media Services credentials in a static class variable.
                    _cachedCredentials = new MediaServicesCredentials(
                                    _mediaServicesAccountName,
                                    _mediaServicesAccountKey);
                    // Used the cached credentials to create CloudMediaContext.
                    _context = new CloudMediaContext(_cachedCredentials);
   
                    // Run the FaceRedaction job.
                    var asset = RunFaceRedactionJob(@"C:\supportFiles\FaceRedaction\SomeFootage.mp4",
                                                @"C:\supportFiles\FaceRedaction\config.json");
   
                    // Download the job output asset.
                    DownloadAsset(asset, @"C:\supportFiles\FaceRedaction\Output");
                }
   
                static IAsset RunFaceRedactionJob(string inputMediaFilePath, string configurationFile)
                {
                    // Create an asset and upload the input media file to storage.
                    IAsset asset = CreateAssetAndUploadSingleFile(inputMediaFilePath,
                        "My Face Redaction Input Asset",
                        AssetCreationOptions.None);
   
                    // Declare a new job.
                    IJob job = _context.Jobs.Create("My Face Redaction Job");
   
                    // Get a reference to Azure Media Redactor.
                    string MediaProcessorName = "Azure Media Redactor";
   
                    var processor = GetLatestMediaProcessorByName(MediaProcessorName);
   
                    // Read configuration from the specified file.
                    string configuration = File.ReadAllText(configurationFile);
   
                    // Create a task with the encoding details, using a string preset.
                    ITask task = job.Tasks.AddNew("My Face Redaction Task",
                        processor,
                        configuration,
                        TaskOptions.None);
   
                    // Specify the input asset.
                    task.InputAssets.Add(asset);
   
                    // Add an output asset to contain the results of the job.
                    task.OutputAssets.AddNew("My Face Redaction Output Asset", AssetCreationOptions.None);
   
                    // Use the following event handler to check job progress.  
                    job.StateChanged += new EventHandler<JobStateChangedEventArgs>(StateChanged);
   
                    // Launch the job.
                    job.Submit();
   
                    // Check job execution and wait for job to finish.
                    Task progressJobTask = job.GetExecutionProgressTask(CancellationToken.None);
   
                    progressJobTask.Wait();
   
                    // If job state is Error, the event handling
                    // method for job progress should log errors.  Here we check
                    // for error state and exit if needed.
                    if (job.State == JobState.Error)
                    {
                        ErrorDetail error = job.Tasks.First().ErrorDetails.First();
                        Console.WriteLine(string.Format("Error: {0}. {1}",
                                                        error.Code,
                                                        error.Message));
                        return null;
                    }
   
                    return job.OutputMediaAssets[0];
                }
   
                static IAsset CreateAssetAndUploadSingleFile(string filePath, string assetName, AssetCreationOptions options)
                {
                    IAsset asset = _context.Assets.Create(assetName, options);
   
                    var assetFile = asset.AssetFiles.Create(Path.GetFileName(filePath));
                    assetFile.Upload(filePath);
   
                    return asset;
                }
   
                static void DownloadAsset(IAsset asset, string outputDirectory)
                {
                    foreach (IAssetFile file in asset.AssetFiles)
                    {
                        file.Download(Path.Combine(outputDirectory, file.Name));
                    }
                }
   
                static IMediaProcessor GetLatestMediaProcessorByName(string mediaProcessorName)
                {
                    var processor = _context.MediaProcessors
                        .Where(p => p.Name == mediaProcessorName)
                        .ToList()
                        .OrderBy(p => new Version(p.Version))
                        .LastOrDefault();
   
                    if (processor == null)
                        throw new ArgumentException(string.Format("Unknown media processor",
                                                                   mediaProcessorName));
   
                    return processor;
                }
   
                static private void StateChanged(object sender, JobStateChangedEventArgs e)
                {
                    Console.WriteLine("Job state changed event:");
                    Console.WriteLine("  Previous state: " + e.PreviousState);
                    Console.WriteLine("  Current state: " + e.CurrentState);
   
                    switch (e.CurrentState)
                    {
                        case JobState.Finished:
                            Console.WriteLine();
                            Console.WriteLine("Job is finished.");
                            Console.WriteLine();
                            break;
                        case JobState.Canceling:
                        case JobState.Queued:
                        case JobState.Scheduled:
                        case JobState.Processing:
                            Console.WriteLine("Please wait...\n");
                            break;
                        case JobState.Canceled:
                        case JobState.Error:
                            // Cast sender as a job.
                            IJob job = (IJob)sender;
                            // Display or log error details as needed.
                            // LogJobStop(job.Id);
                            break;
                        default:
                            break;
                    }
                }
   
            }
        }

## Paso siguiente
Consulte las rutas de aprendizaje de Servicios multimedia.

[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## Envío de comentarios
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

## Vínculos relacionados
[Azure Media Services Analytics Overview (Información general sobre análisis de Servicios multimedia de Azure)](media-services-analytics-overview.md)

[Demostraciones de Análisis multimedia de Azure](http://azuremedialabs.azurewebsites.net/demos/Analytics.html)

<!---HONumber=AcomDC_0914_2016-->