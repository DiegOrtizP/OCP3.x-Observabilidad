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

**The `Metrics` class is the main entry point to the API of this library.** The most common practice in C# code is to have a `static readonly` field for each metric that you wish to export from a given class.

More complex patterns may also be used (e.g. combining with dependency injection). The library is quite tolerant of different usage models - if the API allows it, it will generally work fine and provide satisfactory performance. The library is thread-safe.

# Installation

Nuget package for general use and metrics export via HttpListener or to Pushgateway: [prometheus-net](https://www.nuget.org/packages/prometheus-net)

>Install-Package prometheus-net

Nuget package for ASP.NET Core middleware and stand-alone Kestrel metrics server: [prometheus-net.AspNetCore](https://www.nuget.org/packages/prometheus-net.AspNetCore)

>Install-Package prometheus-net.AspNetCore

# Counters

Counters only increase in value and reset to zero when the process restarts.

```csharp
private static readonly Counter ProcessedJobCount = Metrics
	.CreateCounter("myapp_jobs_processed_total", "Number of processed jobs.");

...

ProcessJob();
ProcessedJobCount.Inc();
```

# Gauges

Gauges can have any numeric value and change arbitrarily.

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

# Summary

Summaries track the trends in events over time (10 minutes by default).

```csharp
private static readonly Summary RequestSizeSummary = Metrics
	.CreateSummary("myapp_request_size_bytes", "Summary of request sizes (in bytes) over last 10 minutes.");

...

RequestSizeSummary.Observe(request.Length);
```

By default, only the sum and total count are reported. You may also specify quantiles to measure:

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

# Histogram

Histograms track the size and number of events in buckets. This allows for aggregatable calculation of quantiles.

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

# Measuring operation duration

Timers can be used to report the duration of an operation (in seconds) to a Summary, Histogram, Gauge or Counter. Wrap the operation you want to measure in a using block.

```csharp
private static readonly Histogram LoginDuration = Metrics
	.CreateHistogram("myapp_login_duration_seconds", "Histogram of login call processing durations.");

...

using (LoginDuration.NewTimer())
{
    IdentityManager.AuthenticateUser(Request.Credentials);
}
```

# Tracking in-progress operations

You can use `Gauge.TrackInProgress()` to track how many concurrent operations are taking place. Wrap the operation you want to track in a using block.

```csharp
private static readonly Gauge DocumentImportsInProgress = Metrics
	.CreateGauge("myapp_document_imports_in_progress", "Number of import operations ongoing.");

...

using (DocumentImportsInProgress.TrackInProgress())
{
	DocumentRepository.ImportDocument(path);
}
```

# Counting exceptions

You can use `Counter.CountExceptions()` to count the number of exceptions that occur while executing some code.


```csharp
private static readonly Counter FailedDocumentImports = Metrics
	.CreateCounter("myapp_document_imports_failed_total", "Number of import operations that failed.");

...

FailedDocumentImports.CountExceptions(() => DocumentRepository.ImportDocument(path));
```

You can also filter the exception types to observe:

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

# Labels

All metrics can have labels, allowing grouping of related time series.

See the best practices on [naming](http://prometheus.io/docs/practices/naming/)
and [labels](http://prometheus.io/docs/practices/instrumentation/#use-labels).

Taking a counter as an example:

```csharp
private static readonly Counter RequestCountByMethod = Metrics
	.CreateCounter("myapp_requests_total", "Number of requests received, by HTTP method.",
		new CounterConfiguration
		{
			// Here you specify only the names of the labels.
			LabelNames = new[] { "method" }
		});

...

// You can specify the values for the labels later, once you know the right values (e.g in your request handler code).
counter.WithLabels("GET").Inc();
```

NB! Best practices of metric design is to minimize the number of different label values. HTTP request method is good - there are not many values. However, URL would be a bad choice for labeling - it has too many possible values and would lead to significant data processing inefficiency. Try to minimize the possible number of label values in your metric model.

# When are metrics published?

Metrics without labels are published immediately after the `Metrics.CreateX()` call. Metrics that use labels are published when you provide the label values for the first time.

Sometimes you want to delay publishing a metric until you have loaded some data and have a meaningful value to supply for it. The API allows you to suppress publishing of the initial value until you decide the time is right.

```csharp
private static readonly Gauge UsersLoggedIn = Metrics
	.CreateGauge("myapp_users_logged_in", "Number of active user sessions",
		new GaugeConfiguration
		{
			SuppressInitialValue = true
		});

...

// After setting the value for the first time, the metric becomes published.
UsersLoggedIn.Set(LoadSessions().Count);
```

You can also use `.Publish()` on a metric to mark it as ready to be published without modifying the initial value (e.g. to publish a zero).

# ASP.NET Core exporter middleware

For projects built with ASP.NET Core, a middleware plugin is provided.

If you use the default Visual Studio project template, modify *Startup.cs* as follows:

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

Alternatively, if you use a custom project startup cycle, you can add this directly to the WebHostBuilder instance:

```csharp
WebHost.CreateDefaultBuilder()
	.Configure(app => app.UseMetricServer())
	.Build()
	.Run();
```

The default configuration will publish metrics on the /metrics URL.

This functionality is delivered in the `prometheus-net.AspNetCore` NuGet package.

# ASP.NET Core HTTP request metrics

The library provides some metrics for ASP.NET Core applications:

* Number of HTTP requests in progress.
* Total number of received HTTP requests.
* Duration of HTTP requests.

These metrics include labels for status code, HTTP method, ASP.NET Core Controller and ASP.NET Core Action.

You can register all of the metrics using the default labels and names as follows:

```csharp
// In your Startup.cs Configure() method
app.UseHttpMetrics();
```

If you wish to provide a custom metric instance or disable certain metrics you can configure the HTTP metrics like this:

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

The labels for the custom metric you provide *must* be a subset of the following:

* "code" - Status Code
* "method" - HTTP method
* "controller" - ASP.NET Core Controller
* "action" - ASP.NET Core Action

# ASP.NET Core with basic authentication

You may wish to restrict access to the metrics export URL. This can be accomplished using any ASP.NET Core authentication mechanism, as prometheus-net integrates directly into the composable ASP.NET Core request processing pipeline.

For a simple example we can take [BasicAuthMiddleware by Johan Boström](https://www.johanbostrom.se/blog/adding-basic-auth-to-your-mvc-application-in-dotnet-core) which can be integrated by replacing the `app.UseMetricServer()` line with the following code block:

```csharp
app.Map("/metrics", metricsApp =>
{
    metricsApp.UseMiddleware<BasicAuthMiddleware>("Contoso Corporation");

    // We already specified URL prefix in .Map() above, no need to specify it again here.
    metricsApp.UseMetricServer("");
});
```

# Kestrel stand-alone server

In some situation, you may wish to start a stand-alone metric server using Kestrel (e.g. if your app has no other HTTP-accessible functionality).

```csharp
var metricServer = new KestrelMetricServer(port: 1234);
metricServer.Start();
```

The default configuration will publish metrics on the `/metrics` URL.

# Publishing to Pushgateway

Metrics can be posted to a [Pushgateway](https://prometheus.io/docs/practices/pushing/) server.

```csharp
var metricServer = new MetricPusher(endpoint: "https://pushgateway.example.org:9091/metrics", job: "some_job");
metricServer.Start();
```

# Publishing via standalone HTTP handler

As a fallback option for scenarios where Kestrel or ASP.NET Core hosting is unsuitable, an `HttpListener` based metrics server implementation is also available.

```csharp
var metricServer = new MetricServer(port: 1234);
metricServer.Start();
```

The default configuration will publish metrics on the `/metrics` URL.

`MetricServer.Start()` may throw an access denied exception on Windows if your user does not have the right to open a web server on the specified port. You can use the *netsh* command to grant yourself the required permissions:

> netsh http add urlacl url=http://+:1234/metrics user=DOMAIN\user

# Just-in-time updates

In some scenarios you may want to only collect data when it is requested by Prometheus. To easily implement this scenario prometheus-net enables you to register a callback before every collection occurs. Register your callback using `Metrics.DefaultRegistry.AddBeforeCollectCallback()`.

Note that all callbacks will be called synchronously before each collection. They should not take more than a few milliseconds in order to ensure that the scrape does not time out. Do not read data from remote systems in these callbacks.

# Suppressing default metrics

The library provides some sample metrics about the current process out of the box, simply to ensure that some output is produced in a default configuration. If these metrics are not desirable you may remove them by calling `Metrics.SuppressDefaultMetrics()` before registering any of your own metrics.

# Related projects

* [prometheus-net.DotNetRuntime](https://github.com/djluck/prometheus-net.DotNetRuntime) instruments .NET Core 2.2 apps to export metrics on .NET Core performance.
