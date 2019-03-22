# Fluentd

Nombre     | Sección | Descripcion
---------|-------------|--------------------
Instalacion | Para maquinas Windows | . |
Pruebas | Pruebas de la centralizacion de logs | . |

## Para maquinas Windows

En este artículo, explicamos cómo comenzar a recopilar datos de las máquinas con Windows (esta configuración se ha probado en una máquina con Windows 8 de 64 bits).

A partir de la v10, `Fluentd` no es compatible con Windows. Sin embargo, hay ocasiones en las que debe recopilar flujos de datos de las máquinas con Windows. Por ejemplo:

    1. Archivos de registro de seguimiento en Windows: recopile y analice los datos de registro de una aplicación de Windows.
    2. Recopilación de registros de eventos de Windows: recopile registros de eventos de sus servidores de Windows para el análisis del sistema, verificación de cumplimiento, etc.

Si no está familiarizado con Fluentd, primero conozca más sobre Fluentd.

[![fluentd](https://docs.fluentd.org/articles/architecture)

## Instalacion

### Prerequisitos

1. [![nxlog](http://nxlog.org/), una herramienta de administración de registro de código abierto que se ejecuta en Windows.
2. Un contenedor linux (Fluentd en Openshift en este caso)

### Configurar un contendor  con rsyslogd y Fluentd

1. Consiga un servidor Linux. En este ejemplo, asumimos que es Ubuntu.
2. Asegúrese de que tiene puertos abiertos para TCP. *5140 default*
  `En el siguiente ejemplo, asumimos que el puerto 5140 está abierto.`

### Configurar nxlog en Windows

Abra el instalador de nxlog descargado y siga las instrucciones de forma predeterminada, debe instalarse en:

>C:\Archivos de programa(x86)\nxlog`

Cree un archivo de configuración nxlog de la siguiente manera y guárdelo como `nxlog.conf`

```xml
#define ROOT C:\Program Files\nxlog
define ROOT C:\Program Files(x86)\nxlog

Moduledir %ROOT%\modules
CacheDir %ROOT%\data
Pidfile %ROOT%\data\nxlog.pid
SpoolDir %ROOT%\data
LogFile %ROOT%\data\nxlog.log

<Input in>
  Module im_file
  File 'C:\Users\SomeUser\Desktop\nxlog_test.log' #Pon el archivo a leer aquí..
  SavePos TRUE
  InputType LineBased
</Input>

<Output out>
  Module om_tcp
  Host LINUX_MACHINE_RUNNING_FLUENTD
  Port 5140
</Output>

<Route r>
  Path in => out
</Route>
```

Esta configuración enviará cada línea del archivo de registro (consulte el parámetro Archivo dentro de `<Input in&gt…</Input>`) como un mensaje de registro del sistema hacia una instancia remota de Fluentd/Treasure Agent.

### Pruebas

1. Vaya al directorio de nxlog (en Powershell o símbolo del sistema) y ejecute el siguiente comando:

```powershell
\nxlog.exe -f -c  <path to nxlog.conf>
```
La opción `-f` ejecuta nxlog en primer plano (esto es para probar). Para producción, se requiere convertirlo en un Servicio de Windows.

2. Una vez que se esté ejecutando nxlog, agregue una nueva línea "redhat openshift" en el archivo de cola como este:

```powershell
echo Windows is awesome >> 'C:\Users\SomeUser\Desktop\nxlog_test.log'
```

3. Ahora, ve al contenedor y ejecuta lo siguiente:

```sh
$ sudo tail -f /var/log/td-agent/td-agent.log
 ...
 ...
 ...
 2014-12-20 02:19:36 +0000 windowslog: {"message":"redhat openshift \r"}
 ```

 4. Usted envió con éxito datos desde una máquina Windows a una instancia remota de Fluentd que se ejecuta en contenedor.

### Análisis de registros JSON

Si está enviando registros JSON en Windows a Fluentd, Fluentd puede analizarlos a medida que ingresan.
Para hacerlo, simplemente cambie la configuración de Fluentd de la siguiente manera.
Note el cambio de `format none` a `format json`. (Consulte [este](https://docs.fluentd.org/v0.12/articles/parser-plugin-overview) artículo para obtener más detalles sobre los complementos del analizador)

```config
<source>
  @type tcp
  format json
  port 5140
  tag windowslog
</source>    
<match windowslog>
  @type stdout
</match>
```

Luego, si agrega una nueva línea al archivo en su máquina con Windows de esta manera:

```powershell
echo {"name":"RedHat", "age":26} >> 'C:\Users\SomeUser\Desktop\nxlog_test.log'
```
En la máquina Linux que ejecuta Fluentd, verá la siguiente línea:

```sh
2019-04-02 08:08:08 +0000 windowslog: {"name":"RedHat","age":26}
```

### Siguientes updatePeriodSeconds

NOTA: Este ejemplo mostró que podemos recopilar datos de una máquina con Windows y enviarlos a una instancia remota de Fluentd.
Sin embargo, los datos no son tan eficientes porque cada línea de datos se coloca en el campo "mensaje" como texto no estructurado. Para fines de producción, probablemente querrá escribir un complemento / extender el complemento de syslog para poder analizar el campo "mensaje" en el evento.
