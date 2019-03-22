# Elasticsearch

Nombre     | Sección | Descripcion
---------|-------------|--------------------
WMI exporter | Para maquinas Windows | . |
prometheus-net | Add .Net | Esta es una biblioteca .NET para instrumentar sus aplicaciones y exportar métricas a [Prometheus](http://prometheus.io/). |

## Introduccion

[![Build status](https://ci.appveyor.com/api/projects/status/ljwan71as6pf2joe?svg=true)](https://ci.appveyor.com/project/martinlindhe/wmi-exporter)

`NEST` es un cliente de alto nivel de `elasticsearch` que aún se correlaciona muy de cerca con la API de elasticsearch original. Las solicitudes y respuestas se han asignado a los objetos CLR y NEST también viene con un dsl de consulta muy potente.

## Instalacion

Desde la consola del gestor de paquetes dentro de visual studio.

>PM> Install-Package NEST

O busca en la interfaz de usuario de Package Manager para NEST e instalalo desde ahí.

## Connexion

Con Elasticsearch ya está desplegado y ejecutándose, vaya a "http://<Elasticsearch_URL>:9200 en su navegador.
Se tendria que ver una respuesta similar a esto:


```powershell
{
  "status" : 200,
  "name" : "Sin-Eater",
  "version" : {
    "number" : "1.0.0",
    "build_hash" : "a46900e9c72c0a623d71b54016357d5f94c8ea32",
    "build_timestamp" : "2014-02-12T16:18:34Z",
    "build_snapshot" : false,
    "lucene_version" : "4.6"
  },
  "tagline" : "You Know, for Search"
}
```

Para conectarse a su nodo utilizando NEST, simplemente agrege:

```csharp
var node = new Uri("http://<Elasticsearch_URL>:9200");

var settings = new ConnectionSettings(
    node,
    defaultIndex: "my-application"
);

var client = new ElasticClient(settings);
...
```
Aquí creamos una nueva conexión a nuestro `node` y especificamos un `default index` que se usara cuando no especificamos explícitamente una.

Esto puede reducir enormemente los lugares en los que se debe usar una cadena mágica o una constante.

>La especificación de defaultIndex es opcional, pero NEST puede lanzar una excepción más adelante si no se especifica un índice.
De hecho, un nuevo ElasticClient() es suficiente para comunicarse con http: //<Elasticsearch_URL>:9200, pero se recomienda especificar explícitamente la configuración de la conexión.

`node` aquí hay un `Uri` pero también puede ser un `IConnectionPool` para mas informacion [IConnectionPool](https://www.elastic.co/guide/en/elasticsearch/client/net-api/1.x/nest-connecting.html)

## Ejemplos

### Indices

Imaginemos que tenemos una nueva persona [POCO](http://en.wikipedia.org/wiki/Plain_Old_CLR_Object)

```csharp
public class Person
{
    public string Id { get; set; }
    public string Firstname { get; set; }
    public string Lastname { get; set; }
}
```

Si quisieramos indexar el `POCO` en Elasticsearch. La indexación es tan simple como un llamado.

```csharp
var person = new Person
{
    Id = "1",
    Firstname = "Martijn",
    Lastname = "Laarman"
};

var index = client.Index(person);
```

Esto indexará el objeto a `/my-application/person/1` `NEST` es lo suficientemente inteligente como para inferir el índice y el nombre de tipo para el `POCO` tipo CLR también es capaz de obtener el ID de 1 a través de la convención, buscando una propiedad de ID en el objeto especificado. La propiedad que utilizará para la identificación también se puede especificar utilizando el atributo `ElasticType`.

El índice y los nombres de tipo predeterminados son configurables por tipo. [conozca mas](https://www.elastic.co/guide/en/elasticsearch/client/net-api/1.x/nest-connecting.html)

Si desea anular todos los valores predeterminados para esta llamada, debe poder hacerlo con `NEST`. Inferir a `NEST` es muy poderoso, pero si quieres pasar valores explícitos, siempre se puede hacer.

```csharp
var index = client.Index(person, i=>i
    .Index("another-index")
    .Type("another-type")
    .Id("1-should-not-be-the-id")
    .Refresh()
    .Ttl("1m")
);
```
Esto indexará el documento utilizando como contexto de la url: `/another-index/another-type/1-should-not-be-the-id?refresh=true&&ttl=1m`

### Busquedas

Ahora que hemos indexado algunos documentos podemos comenzar a buscarlos.

```csharp
var searchResults = client.Search<Person>(s=>s
    .From(0)
    .Size(10)
    .Query(q=>q
         .Term(p=>p.Firstname, "redhat")
    )
);
```

`searchResults.Documents` ahora tiene las primeras 10 personas que sabe de quién es el primer nombre `redhat`

Consulte la [sección sobre consultas](https://www.elastic.co/guide/en/elasticsearch/client/net-api/1.x/writing-queries.html) de escritura para obtener detalles sobre cómo NEST le ayuda a escribir consultas de búsqueda en elasticsearch.

De nuevo, se aplican las mismas reglas de inferencia ya que esto se indicara en `my-application/person/_search` y la misma regla que infiere puede ser anulada también.

```csharp
// uses /other-index/other-type/_search
var searchResults = client.Search<Person>(s=>s
    .Index("other-index")
    .OtherType("other-type")
);

// uses /_all/person/_search
var searchResults = client.Search<Person>(s=>s
   .AllIndices()
);

// uses /_search
var searchResults = client.Search<Person>(s=>s
    .AllIndices()
    .AllTypes()
);
```

### Sintaxis del inicializador de objetos (OIS)

Como puede ver en los ejemplos anteriores, NEST proporciona una sintaxis concisa y fluida para construir llamadas API a Elasticsearch. Sin embargo, no se preocupe si las lambdas no son lo suyo, ahora puede usar la nueva sintaxis de inicialización de objetos (OIS) introducida en 1.0.

El OIS es una alternativa a la sintaxis fluida familiar de NEST y funciona en todos los puntos finales de API. Cualquier cosa que se pueda hacer con la sintaxis fluida ahora también se puede hacer usando el OIS.

Por ejemplo, el ejemplo anterior de indexación anterior se puede volver a escribir como:

```csharp
var indexRequest = new IndexRequest<Person>(person)
{
    Index = "another-index",
    Type = "another-type",
    Id = "1-should-not-be-the-id",
    Refresh = true,
    Ttl = "1m"
};

var index = client.Index(indexRequest);
```

y para buscar:

```csharp
QueryContainer query = new TermQuery
{
    Field = "firstName",
    Value = "martijn"
};

var searchRequest = new SearchRequest
{
    From = 0,
    Size = 10,
    Query = query
};

var searchResults = Client.Search<Person>(searchRequest);
```

Para mas informacion consulte [ElasticSearch](https://www.elastic.co/guide/en/elasticsearch/client/net-api/1.x/nest-quick-start.html#nest-quick-start)
