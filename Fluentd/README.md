# WMI exporter

[![Build status](https://ci.appveyor.com/api/projects/status/ljwan71as6pf2joe?svg=true)](https://ci.appveyor.com/project/martinlindhe/wmi-exporter)

Prometheus exportador para máquinas con Windows, utilizando WMI (Instrumental de administración de Windows).

## Collectores

Name     | Description | Enabled by default
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

## Instalacion
La última versión se puede descargar desde el [releases page](https://github.com/martinlindhe/wmi_exporter/releases).

Cada versión proporciona un instalador .msi. El instalador configurará el WMI Exporter como un servicio de Windows, así como creará una excepción en el Firewall de Windows.

Si el instalador se ejecuta sin ningún parámetro, el exportador se ejecutará con la configuración predeterminada para los recopiladores habilitados, puertos, etc. Los siguientes parámetros están disponibles:

Name | Description
-----|------------
`ENABLED_COLLECTORS` | Como la bandera `--collectors.enabled` , proporciona una lista separada por comas de colectores habilitados.
`LISTEN_ADDR` | La dirección IP para enlazar. El default es: to 0.0.0.0
`LISTEN_PORT` | El puerto para enlazar. Por defecto es: 9182.
`METRICS_PATH` | La ruta en la que se entregan las métricas. Por defecto es: `/metrics`.
`TEXTFILE_DIR` | Como la bandera `--collector.textfile.directory` , proporcionar un directorio para leer archivos de texto con métricas.
`EXTRA_FLAGS` | Permite pasar banderas completas de la CLI. El valor predeterminado es una cadena vacía.

Los parámetros se envían al instalador a través de `msiexec`.

## Ejemplos

```powershell
msiexec /i <path-to-msi-file> ENABLED_COLLECTORS=os,iis LISTEN_PORT=5000
```

Ejemplo de colector de servicios con una consulta personalizada.

```powershell
msiexec /i <path-to-msi-file> ENABLED_COLLECTORS=os,service --% EXTRA_FLAGS="--collector.service.services-where ""Name LIKE 'sql%'"""
```

## Uso

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
