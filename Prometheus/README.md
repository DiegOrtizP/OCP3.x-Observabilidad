# Prometheus

Nombre     | Sección | Descripcion
---------|-------------|--------------------
WMI exporter | Para maquinas Windows | . |
prometheus-net | Add .Net | Esta es una biblioteca .NET para instrumentar sus aplicaciones y exportar métricas a [Prometheus](http://prometheus.io/). |

## Para maquinas Windows

[![Build status](https://ci.appveyor.com/api/projects/status/ljwan71as6pf2joe?svg=true)](https://ci.appveyor.com/project/martinlindhe/wmi-exporter)

Prometheus exportador para máquinas con Windows, utilizando WMI (Instrumental de administración de Windows).

### Collectores

Nombre     | Descripcion | Habilitado por default
---------|-------------|--------------------
[ad](docs/collector.ad.md) | Active Directory Domain Services |
[cpu](docs/collector.cpu.md) | CPU usage | &#10003;
[cs](docs/collector.cs.md) | "Computer System" metrics (system properties, num cpus/total memory) | &#10003;
[dns](docs/collector.dns.md) | DNS Server |
[hyperv](docs/collector.hyperv.md) | Hyper-V hosts |
[iis](docs/collector.iis.md) | IIS sites and applications |
[logical_disk](docs/collector.logical_disk.md) | Logical disks, disk I/O | &#10003;
[memory](docs/collector.memory.md) | Memory usage metrics |
[msmq](docs/collector.msmq.md) | MSMQ queues |
[mssql](docs/collector.mssql.md) | [SQL Server Performance Objects](https://docs.microsoft.com/en-us/sql/relational-databases/performance-monitor/use-sql-server-objects#SQLServerPOs) metrics  |
[netframework_clrexceptions](docs/collector.netframework_clrexceptions.md) | .NET Framework CLR Exceptions |
[netframework_clrinterop](docs/collector.netframework_clrinterop.md) | .NET Framework Interop Metrics |
[netframework_clrjit](docs/collector.netframework_clrjit.md) | .NET Framework JIT metrics |
[netframework_clrloading](docs/collector.netframework_clrloading.md) | .NET Framework CLR Loading metrics |
[netframework_clrlocksandthreads](docs/collector.netframework_clrlocksandthreads.md) | .NET Framework locks and metrics threads |
[netframework_clrmemory](docs/collector.netframework_clrmemory.md) |  .NET Framework Memory metrics |
[netframework_clrremoting](docs/collector.netframework_clrremoting.md) | .NET Framework Remoting metrics |
[netframework_clrsecurity](docs/collector.netframework_clrsecurity.md) | .NET Framework Security Check metrics |
[net](docs/collector.net.md) | Network interface I/O | &#10003;
[os](docs/collector.os.md) | OS metrics (memory, processes, users) | &#10003;
[process](docs/collector.process.md) | Per-process metrics |
[service](docs/collector.service.md) | Service state metrics | &#10003;
[system](docs/collector.system.md) | System calls | &#10003;
[tcp](docs/collector.tcp.md) | TCP connections |
[textfile](docs/collector.textfile.md) | Read prometheus metrics from a text file | &#10003;
[vmware](docs/collector.vmware.md) | Performance counters installed by the Vmware Guest agent |

Consulte la documentación vinculada a cada recopilador para obtener más información sobre las métricas informadas, los ajustes de configuración y los ejemplos de uso.

### Instalacion
La última versión se puede descargar desde el [releases page](https://github.com/martinlindhe/wmi_exporter/releases).

Cada versión proporciona un instalador .msi. El instalador configurará el WMI Exporter como un servicio de Windows, así como creará una excepción en el Firewall de Windows.

Si el instalador se ejecuta sin ningún parámetro, el exportador se ejecutará con la configuración predeterminada para los recopiladores habilitados, puertos, etc. Los siguientes parámetros están disponibles:

Nombre | Descripcion
-----|------------
`ENABLED_COLLECTORS` | Como la bandera `--collectors.enabled` , proporciona una lista separada por comas de colectores habilitados.
`LISTEN_ADDR` | La dirección IP para enlazar. El default es: to 0.0.0.0
`LISTEN_PORT` | El puerto para enlazar. Por defecto es: 9182.
`METRICS_PATH` | La ruta en la que se entregan las métricas. Por defecto es: `/metrics`.
`TEXTFILE_DIR` | Como la bandera `--collector.textfile.directory` , proporcionar un directorio para leer archivos de texto con métricas.
`EXTRA_FLAGS` | Permite pasar banderas completas de la CLI. El valor predeterminado es una cadena vacía.

Los parámetros se envían al instalador a través de `msiexec`.

### Ejemplos

```powershell
msiexec /i <path-to-msi-file> ENABLED_COLLECTORS=os,iis LISTEN_PORT=5000
```

Ejemplo de colector de servicios con una consulta personalizada.

```powershell
msiexec /i <path-to-msi-file> ENABLED_COLLECTORS=os,service --% EXTRA_FLAGS="--collector.service.services-where ""Name LIKE 'sql%'"""
```

### Uso

    go get -u github.com/golang/dep
    go get -u github.com/prometheus/promu
    go get -u github.com/martinlindhe/wmi_exporter
    cd $env:GOPATH/src/github.com/martinlindhe/wmi_exporter
    promu build -v .
    .\wmi_exporter.exe

Las métricas de prometheus serán expuestas en [localhost:9182](http://localhost:9182)

### Habilitar solo el recopilador de servicios y especificar una consulta personalizada

    .\wmi_exporter.exe --collectors.enabled "service" --collector.service.services-where "Name='wmi_exporter'"

###  Habilitar solo el colector de procesos y especificar una consulta personalizada

    .\wmi_exporter.exe --collectors.enabled "process" --collector.process.processes-where "Name LIKE 'firefox%'"

Cuando hay varios procesos con el mismo nombre, WMI representa aquellos después de la primera instancia como `process-name#index`. Por lo tanto, para obtenerlos todos, en lugar de solo el primero, la consulta debe ser una búsqueda comodín con un carácter `%`.

Tenga en cuenta que en los scripts de proceso por lotes de Windows (y cuando se utiliza el símbolo del sistema `cmd`), el carácter`% `está reservado, por lo que debe escaparse con otro`% `. Por ejemplo, la sintaxis de comodín para buscar todos los procesos de Firefox es `firefox %%`.


## Add .Net


Los objetivos de la biblioteca [.NET Standard 2.0] (https://docs.microsoft.com/en-us/dotnet/standard/net-standard) que admite los siguientes frameworks  (y más nuevos):

* .NET Framework 4.6.1
* .NET Core 2.0
* Mono 5.4

La funcionalidad específica de ASP.NET Core requiere ASP.NET Core 2.1 o más reciente.

### Mejores practicas y uso

Esta biblioteca le permite instrumentar su código con métricas personalizadas y proporciona algunas integraciones de colección de métricas incorporadas para ASP.NET y NETCore.

La documentación aquí es sólo un inicio rápido mínimo. Para obtener una guía detallada sobre el uso de Prometheus en sus soluciones, consulte  [prometheus-users foro](https://groups.google.com/forum/#!forum/prometheus-users). También se espera que se conoscan los concptos basicos que puede encontrar en la siguiente liga: [Prometheus guia de usuario](https://prometheus.io/docs/introduction/overview/).

Hay disponibles cuatro tipos de métricas: Contador, Calibrador, Resumen e Histograma. Vea la documentación en [Tipos de metricas](http://prometheus.io/docs/concepts/metric_types/) y [Mejores practicas](http://prometheus.io/docs/practices/instrumentation/#counter-vs.-gauge-vs.-summary) para aprender mas sobre cada una.

**La clase `Métricas` es el punto de entrada principal a la API de esta biblioteca.** TLa práctica más común en el código C# es tener un campo `static readonly` para cada métrica que desee exportar desde una clase determinada.

También se pueden usar patrones más complejos (por ejemplo, combinando con inyección de dependencia). La biblioteca es bastante tolerante con diferentes modelos de uso: si la API lo permite, generalmente funcionará bien y proporcionará un rendimiento satisfactorio. La biblioteca es segura para subprocesos.

### Instalación

Paquete Nuget para uso general y exportación de métricas vía HttpListener o a via Pushgateway: [prometheus-net](https://www.nuget.org/packages/prometheus-net)

>Install-Package prometheus-net

Paquete Nuget para ASP.NET Core middleware y servidor de métricas Kestrel autónomo: [prometheus-net.AspNetCore](https://www.nuget.org/packages/prometheus-net.AspNetCore)

>Install-Package prometheus-net.AspNetCore

### Contadores

Los contadores solo aumentan de valor y se restablecen a cero cuando se reinicia el proceso.

```csharp
private static readonly Counter ProcessedJobCount = Metrics
	.CreateCounter("myapp_jobs_processed_total", "Number of processed jobs.");

...

ProcessJob();
ProcessedJobCount.Inc();
```

### Calibradores

Los medidores pueden tener cualquier valor numérico y cambiar arbitrariamente.

```csharp
private static readonly Gauge JobsInQueue = Metrics
	.CreateGauge("myapp_jobs_queued", "Number of jobs waiting for processing in the queue.");

...

jobQueue.Enqueue(job);
JobsInQueue.Inc();

...

var job = jobQueue.Dequeue();
JobsInQueue.Dec();
```

### Resumen

Los resúmenes hacen un seguimiento de las tendencias de los eventos a lo largo del tiempo (10 minutos de forma predeterminada).

```csharp
private static readonly Summary RequestSizeSummary = Metrics
	.CreateSummary("myapp_request_size_bytes", "Summary of request sizes (in bytes) over last 10 minutes.");

...

RequestSizeSummary.Observe(request.Length);
```

Por defecto, solo se reportan la suma y el conteo total. También puede especificar las cantidades a medir:

```csharp
private static readonly Summary RequestSizeSummary = Metrics
	.CreateSummary("myapp_request_size_bytes", "Summary of request sizes (in bytes) over last 10 minutes.",
		new SummaryConfiguration
		{
			Objectives = new[]
			{
				new QuantileEpsilonPair(0.5, 0.05),
				new QuantileEpsilonPair(0.9, 0.05),
				new QuantileEpsilonPair(0.95, 0.01),
				new QuantileEpsilonPair(0.99, 0.01),
			}
		});
```

### Histograma

Los histogramas rastrean el tamaño y la cantidad de eventos en cubos. Esto permite el cálculo agregable de los cuantiles.

```csharp
private static readonly Histogram OrderValueHistogram = Metrics
	.CreateHistogram("myapp_order_value_usd", "Histogram of received order values (in USD).",
		new HistogramConfiguration
		{
			// We divide measurements in 10 buckets of $100 each, up to $1000.
			Buckets = Histogram.LinearBuckets(start: 100, width: 100, count: 10)
		});

...

OrderValueHistogram.Observe(order.TotalValueUsd);
```

### Duración de la operación de medición

Los temporizadores se pueden usar para informar la duración de una operación (en segundos) a un Resumen, Histograma, Calibrador o Contador. Envuelva la operación que desea medir en un bloque de uso.

```csharp
private static readonly Histogram LoginDuration = Metrics
	.CreateHistogram("myapp_login_duration_seconds", "Histogram of login call processing durations.");

...

using (LoginDuration.NewTimer())
{
    IdentityManager.AuthenticateUser(Request.Credentials);
}
```

### Seguimiento de operaciones en curso

Se puede usar `Gauge.TrackInProgress()` para rastrear cuantas operaciones concurrentes están teniendo lugar. Envuelva la operación que desea rastrear en un bloque usando.

```csharp
private static readonly Gauge DocumentImportsInProgress = Metrics
	.CreateGauge("myapp_document_imports_in_progress", "Number of import operations ongoing.");

...

using (DocumentImportsInProgress.TrackInProgress())
{
	DocumentRepository.ImportDocument(path);
}
```

### Conteo de excepciones

Se puede usar `Counter.CountExceptions()` para contar el número de excepciones que se producen al ejecutar algún código.


```csharp
private static readonly Counter FailedDocumentImports = Metrics
	.CreateCounter("myapp_document_imports_failed_total", "Number of import operations that failed.");

...

FailedDocumentImports.CountExceptions(() => DocumentRepository.ImportDocument(path));
```

También puede filtrar los tipos de excepción para observar:

```csharp
FailedDocumentImports.CountExceptions(() => DocumentRepository.ImportDocument(path), IsImportRelatedException);

bool IsImportRelatedException(Exception ex)
{
	// Do not count "access denied" exceptions - those are user error for pointing us to a forbidden file.
	if (ex is UnauthorizedAccessException)
		return false;

	return true;
}
```

### Etiquetas

Todas las métricas pueden tener etiquetas, lo que permite agrupar las series de tiempo relacionadas.

Ver las mejores practicas en la siguiente liga: [Nombrado](http://prometheus.io/docs/practices/naming/)
y [Etiquetado](http://prometheus.io/docs/practices/instrumentation/#use-labels).

Ejemplo de contador de tareas:

```csharp
private static readonly Counter RequestCountByMethod = Metrics
	.CreateCounter("myapp_requests_total", "Number of requests received, by HTTP method.",
		new CounterConfiguration
		{
			// Aquí usted especifica solo los nombres de las etiquetas.
			LabelNames = new[] { "method" }
		});

...

// Puede especificar los valores de las etiquetas más adelante, una vez que conozca los valores correctos (por ejemplo, en el código del controlador de su solicitud).
counter.WithLabels("GET").Inc();
```

¡NÓTESE BIEN! Las mejores prácticas de diseño métrico es minimizar el número de valores de etiqueta diferentes. El método de solicitud HTTP correcto, no hay muchos valores. Sin embargo, la URL sería una mala opción para el etiquetado, ya que tiene demasiados valores posibles y daría lugar a una ineficiencia significativa en el procesamiento de datos. Intente minimizar el número posible de valores de etiqueta en su modelo métrico.

### ¿Cuándo se publican las métricas?

Las métricas sin etiquetas se publican inmediatamente después de la llamada a `Metrics.CreateX()`. Las métricas que usan etiquetas se publican cuando proporciona los valores de etiqueta por primera vez.

A veces desea retrasar la publicación de una métrica hasta que haya cargado algunos datos y tenga un valor significativo que proporcionar. La API le permite suprimir la publicación del valor inicial hasta que decida que es el momento adecuado.

```csharp
private static readonly Gauge UsersLoggedIn = Metrics
	.CreateGauge("myapp_users_logged_in", "Number of active user sessions",
		new GaugeConfiguration
		{
			SuppressInitialValue = true
		});

...

// Después de establecer el valor por primera vez, la métrica se publica.
UsersLoggedIn.Set(LoadSessions().Count);
```

Tambien se puede usar `.Publish()` en una métrica para marcarla como lista para ser publicada sin modificar el valor inicial (por ejemplo, para publicar un cero).

### ASP.NET Core exporter middleware

Para proyectos creados con ASP.NET Core, se proporciona un complemento de middleware.

Si usa la plantilla de proyecto predeterminada de Visual Studio, modifique *Startup.cs* como se observa a continuación:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    // ...

    app.UseMetricServer();

    app.Run(async (context) =>
    {
        // ...
    });
}
```

Alternativamente, si usa un ciclo de inicio de proyecto personalizado, puede agregarlo directamente a la instancia de WebHostBuilder:

```csharp
WebHost.CreateDefaultBuilder()
	.Configure(app => app.UseMetricServer())
	.Build()
	.Run();
```

TLa configuración por defecto publicará métricas en la /metrics URL.

Esta funcionalidad se entrega en el `prometheus-net.AspNetCore` NuGet.

# ASP.NET Core HTTP recolexión de metricas

La biblioteca proporciona algunas métricas para las aplicaciones Core de ASP.NET:

* Número de solicitudes HTTP en curso.
* Número total de solicitudes HTTP recibidas.
* Duración de las solicitudes HTTP.

Estas métricas incluyen etiquetas para el código de estado, el método HTTP, el controlador central de ASP.NET y la acción central de ASP.NET.

Puede registrar todas las métricas usando las etiquetas y nombres predeterminados de la siguiente manera:

```csharp
// En su método Startup.cs Configure ()
app.UseHttpMetrics();
```

Si desea proporcionar una instancia de métrica personalizada o deshabilitar ciertas métricas, puede configurar las métricas HTTP de esta manera:

```csharp
app.UseHttpMetrics(options =>
{
	options.RequestCount.Enabled = false;

	options.RequestDuration.Histogram = Metrics.CreateHistogram("myapp_http_request_duration_seconds", "Some help text",
		new HistogramConfiguration
		{
			Buckets = Histogram.LinearBuckets(start: 1, width: 1, count: 64)
			Labels = new[] { "code", "method" }
		});
});
```

Las etiquetas para la métrica personalizada que proporcione *debe* ser un subconjunto de lo siguiente:

* "code" - Status Code
* "method" - HTTP method
* "controller" - ASP.NET Core Controller
* "action" - ASP.NET Core Action

### ASP.NET Core con auntenticación basica

Es posible que desee restringir el acceso a la URL de exportación de métricas. Esto se puede lograr utilizando cualquier mecanismo de autenticación de ASP.NET Core, ya que prometheus-net se integra directamente en el canal de procesamiento de solicitudes de ASP.NET Core compostable.

Para un ejemplo simple podemos ver [BasicAuthMiddleware by Johan Boström](https://www.johanbostrom.se/blog/adding-basic-auth-to-your-mvc-application-in-dotnet-core) que se puede integrar mediante la sustitución de la `app.UseMetricServer()`, con el siguiente bloque de código:

```csharp
app.Map("/metrics", metricsApp =>
{
    metricsApp.UseMiddleware<BasicAuthMiddleware>("Contoso Corporation");

    // Ya hemos especificado la URL prefix in .Map() arriba, no es necesario especificarlo de nuevo aquí.
    metricsApp.UseMetricServer("");
});
```

### Kestrel servidor stand-alone

En alguna situación, es posible que desee iniciar un servidor de métricas independiente utilizando Kestrel (por ejemplo, si su aplicación no tiene otra funcionalidad accesible mediante HTTP).

```csharp
var metricServer = new KestrelMetricServer(port: 1234);
metricServer.Start();
```

La configuración por defecto publicará métricas en la URL `/metrics`.

### Publicar el Pushgateway

Las métricas se pueden publicar en un server [Pushgateway](https://prometheus.io/docs/practices/pushing/).

```csharp
var metricServer = new MetricPusher(endpoint: "https://pushgateway.example.org:9091/metrics", job: "some_job");
metricServer.Start();
```

### Publicar via standalone HTTP handler

Como opción alternativa para los escenarios en los que Kestrel o ASP.NET Core hosting  que no son adecuados también está disponible `HttpListener` implementación del servidor de métricas,

```csharp
var metricServer = new MetricServer(port: 1234);
metricServer.Start();
```

La configuración por defecto publicará métricas en la URL `/metrics`.

`MetricServer.Start()` may throw an access denied exception on Windows if your user does not have the right to open a web server on the specified port. You can use the *netsh* command to grant yourself the required permissions:

> netsh http add urlacl url=http://+:1234/metrics user=DOMAIN\user

# Just-in-time updates

In some scenarios you may want to only collect data when it is requested by Prometheus. To easily implement this scenario prometheus-net enables you to register a callback before every collection occurs. Register your callback using `Metrics.DefaultRegistry.AddBeforeCollectCallback()`.

Note that all callbacks will be called synchronously before each collection. They should not take more than a few milliseconds in order to ensure that the scrape does not time out. Do not read data from remote systems in these callbacks.

# Suppressing default metrics

The library provides some sample metrics about the current process out of the box, simply to ensure that some output is produced in a default configuration. If these metrics are not desirable you may remove them by calling `Metrics.SuppressDefaultMetrics()` before registering any of your own metrics.

# Related projects

* [prometheus-net.DotNetRuntime](https://github.com/djluck/prometheus-net.DotNetRuntime) instruments .NET Core 2.2 apps to export metrics on .NET Core performance.
