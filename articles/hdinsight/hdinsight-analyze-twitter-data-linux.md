---
title: Análisis de datos de Twitter con Apache Hive en HDInsight | Microsoft Docs
description: Aprenda a usar Python para almacenar tweets que contengan palabras clave específicas y luego use Hive y Hadoop en HDInsight para transformar los datos sin procesar de TWitter en una tabla de Hive en la que se puedan realizar búsquedas.
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal

ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/27/2016
ms.author: larryfr

---
# <a name="analyze-twitter-data-using-hive-in-hdinsight"></a>Análisis de datos de Twitter con Hive en HDInsight
En este documento, obtendrá tweets mediante una API de streaming de Twitter y, luego, usará Apache Hive en un clúster de HDInsight basado en Linux para procesar los datos con formato JSON. El resultado será una lista de usuarios de Twitter que enviaron la mayoría de los tweets que contienen una palabra determinada.

> [!NOTE]
> Aunque distintas partes de este documento se pueden usar con clústeres de HDInsight basados en Windows (Python por ejemplo), muchos de los pasos se basan el uso de un clúster de HDInsight basado en Linux. Para obtener pasos específicos para un clúster basado en Windows, vea [Análisis de datos de Twitter con Hive en HDInsight](hdinsight-analyze-twitter-data.md).
> 
> 

### <a name="prerequisites"></a>Requisitos previos
Antes de empezar este tutorial, debe contar con lo siguiente:

* Un **Clúster de HDInsight de Azure basado en Linux**. Para obtener información sobre la creación de un clúster, vea [Introducción a HDInsight basado en Linux](hdinsight-hadoop-linux-tutorial-get-started.md) para conocer los pasos sobre la creación de un clúster.
* Un **cliente SSH**. Para obtener más información sobre el uso de SSH con HDInsight basado en Linux, vea los siguientes artículos:
  
  * [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux, Unix u OS X](hdinsight-hadoop-linux-use-ssh-unix.md)
  * [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Windows](hdinsight-hadoop-linux-use-ssh-windows.md)
* **Python** y [pip](https://pypi.python.org/pypi/pip)

## <a name="get-a-twitter-feed"></a>Obtención de una fuente de Twitter
Twitter permite recuperar los [datos de cada tweet](https://dev.twitter.com/docs/platform-objects/tweets) como documento de JavaScript Object Notation (JSON) a través de una API de REST. [OAuth](http://oauth.net) para autenticación en la API. También debe crear una *Aplicación de Twitter* que contenga la configuración empleada para tener acceso a la API.

### <a name="create-a-twitter-application"></a>Crear una aplicación de Twitter
1. Desde un explorador web, inicie sesión en [https://apps.twitter.com/](https://apps.twitter.com/). Haga clic en el vínculo **Sign up now** (Regístrese ahora) si no tiene una cuenta de Twitter.
2. Haga clic en **Crear nueva aplicación**.
3. Especifique los campos **Name** (Nombre), **Description** (Descripción), **Website** (Sitio web). Puede conformar una dirección URL para el campo **Website** (Sitio web). La siguiente tabla muestra algunos valores de ejemplo para utilizar:
   
   | Campo | Valor |
   |:--- |:--- |
   | Nombre |MyHDInsightApp |
   | Description |MyHDInsightApp |
   | Website |http://www.myhdinsightapp.com |
4. Active **Yes, I agree** (Acepto) y, a continuación, haga clic en **Create your Twitter application** (Crear la aplicación de Twitter).
5. Haga clic en la pestaña **Permissions** (Permisos). El permiso predeterminado es **Read only**(Solo lectura). Esto es suficiente para este tutorial.
6. Haga clic en la pestaña **Keys and Access Tokens** (Claves y tokens de acceso).
7. Haga clic en **Create my access token**(Crear mi token de acceso).
8. Haga clic en **Prueba de OAuth** en la esquina superior derecha de la página.
9. Rellene los campos **consumer key** (clave del consumidor), **Consumer secret** (Secreto del consumidor), **Access token** (Token de acceso) y **Access token secret** (Secreto del token de acceso). Los necesitará más adelante.

> [!NOTE]
> Cuando utilice el comando Curl en Windows, use comillas dobles en lugar de comillas simples para los valores de opción.
> 
> 

### <a name="download-tweets"></a>Descarga de tweets
El siguiente código Python descargará 10.000 tweets de Twitter y los guardará en un archivo denominado **tweets.txt**.

> [!NOTE]
> Los siguientes pasos se realizan en el clúster de HDInsight, puesto que Python ya está instalada.
> 
> 

1. Conéctese al clúster de HDInsight con SSH:
   
        ssh USERNAME@CLUSTERNAME-ssh.azurehdinsight.net
   
    Si usó una contraseña para proteger la cuenta de usuario SSH, se le pedirá la contraseña. Si usa una clave pública, tal vez tenga que usar el parámetro `-i` para especificar la ruta de acceso a la correspondiente clave privada. Por ejemplo: `ssh -i ~/.ssh/id_rsa USERNAME@CLUSTERNAME-ssh.azurehdinsight.net`.
   
    Para obtener más información sobre el uso de SSH con HDInsight basado en Linux, vea los siguientes artículos:
   
   * [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Linux, Unix u OS X](hdinsight-hadoop-linux-use-ssh-unix.md)
   * [Utilización de SSH con Hadoop en HDInsight basado en Linux desde Windows](hdinsight-hadoop-linux-use-ssh-windows.md)
2. De forma predeterminada, la utilidad **pip** no está instalada en el nodo principal de HDInsight. Use lo siguiente para instalar y luego actualizar esta utilidad:
   
        sudo apt-get install python-pip
        sudo pip install --upgrade pip
3. El código para descargar tweets se basa en [Tweepy](http://www.tweepy.org/) y [Progressbar](https://pypi.python.org/pypi/progressbar/2.2). Para instalarlas, use el comando siguiente:
   
        sudo apt-get install python-dev libffi-dev libssl-dev
        sudo apt-get remove python-openssl
        sudo pip install tweepy progressbar pyOpenSSL requests[security]
   
   > [!NOTE]
   > La finalidad de los fragmentos sobre cómo quitar python-openssl, instalar python-dev, libffi-dev, libssl-dev, pyOpenSSL y requests[security] es evitar una advertencia InsecurePlatform al conectarse a Twitter a través de SSL desde Python.
   > 
   > Tweepy v3.2.0 se usa para evitar [un error](https://github.com/tweepy/tweepy/issues/576) que pueda producirse al procesar tweets.
   > 
   > 
4. Use el comando siguiente para crear un nuevo archivo denominado **gettweets.py**:
   
        nano gettweets.py
5. Use lo siguiente como contenido del archivo **gettweets.py** . Reemplace la información de marcador de posición para **consumer\_secret**, **consumer\_key**, **access/\_token** y **access\_token\_secret** por la información de la aplicación de Twitter.
   
        #!/usr/bin/python
   
        from tweepy import Stream, OAuthHandler
        from tweepy.streaming import StreamListener
        from progressbar import ProgressBar, Percentage, Bar
        import json
        import sys
   
        #Twitter app information
        consumer_secret='Your consumer secret'
        consumer_key='Your consumer key'
        access_token='Your access token'
        access_token_secret='Your access token secret'
   
        #The number of tweets we want to get
        max_tweets=10000
   
        #Create the listener class that will receive and save tweets
        class listener(StreamListener):
            #On init, set the counter to zero and create a progress bar
            def __init__(self, api=None):
                self.num_tweets = 0
                self.pbar = ProgressBar(widgets=[Percentage(), Bar()], maxval=max_tweets).start()
   
            #When data is received, do this
            def on_data(self, data):
                #Append the tweet to the 'tweets.txt' file
                with open('tweets.txt', 'a') as tweet_file:
                    tweet_file.write(data)
                    #Increment the number of tweets
                    self.num_tweets += 1
                    #Check to see if we have hit max_tweets and exit if so
                    if self.num_tweets >= max_tweets:
                        self.pbar.finish()
                        sys.exit(0)
                    else:
                        #increment the progress bar
                        self.pbar.update(self.num_tweets)
                return True
   
            #Handle any errors that may occur
            def on_error(self, status):
                print status
   
        #Get the OAuth token
        auth = OAuthHandler(consumer_key, consumer_secret)
        auth.set_access_token(access_token, access_token_secret)
        #Use the listener class for stream processing
        twitterStream = Stream(auth, listener())
        #Filter for these topics
        twitterStream.filter(track=["azure","cloud","hdinsight"])
6. Use **Ctrl+X** y luego **Y** para guardar el archivo.
7. Use el siguiente comando para ejecutar el archivo y descargar tweets:
   
        python gettweets.py
   
    Debe aparecer un indicador de progreso y aumentar hasta 100% mientras los tweets se descargan y se guardan en el archivo.
   
   > [!NOTE]
   > Si la barra de progreso tarda mucho en avanzar, debería cambiar el filtro para realizar un seguimiento de los temas con más tendencia; cuando existen muchos tweets sobre el tema que está filtrando, puede obtener rápidamente los 10 000 tweets necesarios.
   > 
   > 

### <a name="upload-the-data"></a>Carga de los datos
Para cargar los datos en WASB (el sistema de archivos distribuido que emplea HDInsight), use los siguientes comandos:

    hdfs dfs -mkdir -p /tutorials/twitter/data
    hdfs dfs -put tweets.txt /tutorials/twitter/data/tweets.txt

Así se almacenan los datos en una ubicación a la que pueden tener acceso todos los nodos del clúster.

## <a name="run-the-hiveql-job"></a>Ejecución del trabajo de HiveQL
1. Use el siguiente comando para crear un archivo que contenga instrucciones de HiveQL:
   
        nano twitter.hql
   
    Use lo siguiente como contenido del archivo:
   
        set hive.exec.dynamic.partition = true;
        set hive.exec.dynamic.partition.mode = nonstrict;
        -- Drop table, if it exists
        DROP TABLE tweets_raw;
        -- Create it, pointing toward the tweets logged from Twitter
        CREATE EXTERNAL TABLE tweets_raw (
            json_response STRING
        )
        STORED AS TEXTFILE LOCATION '/tutorials/twitter/data';
        -- Drop and recreate the destination table
        DROP TABLE tweets;
        CREATE TABLE tweets
        (
            id BIGINT,
            created_at STRING,
            created_at_date STRING,
            created_at_year STRING,
            created_at_month STRING,
            created_at_day STRING,
            created_at_time STRING,
            in_reply_to_user_id_str STRING,
            text STRING,
            contributors STRING,
            retweeted STRING,
            truncated STRING,
            coordinates STRING,
            source STRING,
            retweet_count INT,
            url STRING,
            hashtags array<STRING>,
            user_mentions array<STRING>,
            first_hashtag STRING,
            first_user_mention STRING,
            screen_name STRING,
            name STRING,
            followers_count INT,
            listed_count INT,
            friends_count INT,
            lang STRING,
            user_location STRING,
            time_zone STRING,
            profile_image_url STRING,
            json_response STRING
        );
        -- Select tweets from the imported data, parse the JSON,
        -- and insert into the tweets table
        FROM tweets_raw
        INSERT OVERWRITE TABLE tweets
        SELECT
            cast(get_json_object(json_response, '$.id_str') as BIGINT),
            get_json_object(json_response, '$.created_at'),
            concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
            substr (get_json_object(json_response, '$.created_at'),27,4)),
            substr (get_json_object(json_response, '$.created_at'),27,4),
            case substr (get_json_object(json_response, '$.created_at'),5,3)
                when "Jan" then "01"
                when "Feb" then "02"
                when "Mar" then "03"
                when "Apr" then "04"
                when "May" then "05"
                when "Jun" then "06"
                when "Jul" then "07"
                when "Aug" then "08"
                when "Sep" then "09"
                when "Oct" then "10"
                when "Nov" then "11"
                when "Dec" then "12" end,
            substr (get_json_object(json_response, '$.created_at'),9,2),
            substr (get_json_object(json_response, '$.created_at'),12,8),
            get_json_object(json_response, '$.in_reply_to_user_id_str'),
            get_json_object(json_response, '$.text'),
            get_json_object(json_response, '$.contributors'),
            get_json_object(json_response, '$.retweeted'),
            get_json_object(json_response, '$.truncated'),
            get_json_object(json_response, '$.coordinates'),
            get_json_object(json_response, '$.source'),
            cast (get_json_object(json_response, '$.retweet_count') as INT),
            get_json_object(json_response, '$.entities.display_url'),
            array(
                trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
                trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
            array(
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
                trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
            trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
            trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
            get_json_object(json_response, '$.user.screen_name'),
            get_json_object(json_response, '$.user.name'),
            cast (get_json_object(json_response, '$.user.followers_count') as INT),
            cast (get_json_object(json_response, '$.user.listed_count') as INT),
            cast (get_json_object(json_response, '$.user.friends_count') as INT),
            get_json_object(json_response, '$.user.lang'),
            get_json_object(json_response, '$.user.location'),
            get_json_object(json_response, '$.user.time_zone'),
            get_json_object(json_response, '$.user.profile_image_url'),
            json_response
        WHERE (length(json_response) > 500);
2. Presione **Ctrl+X** y luego **Y** para guardar el archivo.
3. Use el siguiente comando para ejecutar el HiveQL incluido en el archivo:
   
        beeline -u 'jdbc:hive2://localhost:10001/;transportMode=http' -n admin -i twitter.hql
   
    Esto cargará el shell de Hive, ejecutará el HiveQL en el archivo **twitter.hql** y, por último, devolverá un símbolo del sistema `jdbc:hive2//localhost:10001/>`.
4. En el símbolo del sistema Beeline, use lo siguiente para comprobar que puede seleccionar datos de la tabla **tweets** creada por el HiveQL en el archivo **twitter.hql**:
   
        SELECT name, screen_name, count(1) as cc
            FROM tweets
            WHERE text like "%Azure%"
            GROUP BY name,screen_name
            ORDER BY cc DESC LIMIT 10;
   
    Esto devolverá un máximo de 10 tweets que contienen la palabra **Azure** en el texto del mensaje.

## <a name="next-steps"></a>Pasos siguientes
En este tutorial hemos visto cómo transformar un conjunto de datos JSON no estructurado en una tabla de Hive estructurada para consultar, explorar y analizar datos desde Twitter mediante HDInsight en Azure. Para obtener más información, consulte:

* [Introducción a HDInsight](hdinsight-hadoop-linux-tutorial-get-started.md)
* [Análisis de la información de retraso de vuelos con HDInsight](hdinsight-analyze-flight-delay-data-linux.md)

[curl]: http://curl.haxx.se
[curl-download]: http://curl.haxx.se/download.html

[apache-hive-tutorial]: https://cwiki.apache.org/confluence/display/Hive/Tutorial

[twitter-streaming-api]: https://dev.twitter.com/docs/streaming-apis
[twitter-statuses-filter]: https://dev.twitter.com/docs/api/1.1/post/statuses/filter



<!--HONumber=Oct16_HO2-->


