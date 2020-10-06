# TesteLogAspnetCore

![.NET Core](https://github.com/fformis/teste-log-aspnet-core/workflows/.NET%20Core/badge.svg?event=push)

Teste de diferentes providers de log para asp.net core

## Event Log

Para utilizar o Log de Eventos do Windows, basta adicionar a configuração:

```JSONP
{
  "Logging": {
    // Log no output do Visual Studio
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    },
    //Log no Event Viewer
    "EventLog": {
      "LogLevel": {
        "Default": "Information"
      }
    }
  },
  "AllowedHosts": "*"
}
```
Na controller devemos receber via DI a implementação de ILogger<T> e atribuir a um field.

```C#
private readonly ILogger<WeatherForecastController> _logger;
        
public WeatherForecastController(ILogger<WeatherForecastController> logger)
{
  _logger = logger;
}
```

Para escrever no log basta:

```C#
_logger.LogError("Teste Log Microsoft");
```

## Log4Net

Primeiro é necessário criar um arquivo de configuração, log4net.config, que é um arquivo xml que contém as configurações do log4net, tais quais as configurações do log4net em web.config das aplicações ASP.NET utilizando .Net Framework.

```XML
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <log4net>
    <root>
      <level value="ALL" />
      <appender-ref ref="RollingFile" />
    </root>
    <appender name="RollingFile" type="log4net.Appender.FileAppender">
      <file value="log\log.txt" />
      <layout type="log4net.Layout.PatternLayout">
        <conversionPattern value="%-5p %d{hh:mm:ss} %message%newline" />
      </layout>
    </appender>
  </log4net>
</configuration>
```

O método Main da classe Program deve ficar dessa maneira, para ler a configuração do log4net:

```C#
public static void Main(string[] args)
{
  // Adicionar o arquivo log4net.config, esse arquivo é um xml com as configurações do log.
  // Abaixo carregamos as configurações e está pronto para utilizar o log4net.
  var log4netRepository = log4net.LogManager.GetRepository(Assembly.GetEntryAssembly());
  log4net.Config.XmlConfigurator.Configure(log4netRepository, new FileInfo("log4net.config"));

  CreateHostBuilder(args).Build().Run();
}
```

Para utilizar o log4net pode ser criado um field na controller:

```C#
using log4net;
```
```C#
static readonly ILog _log4net = log4net.LogManager.GetLogger(typeof(WeatherForecastController));
```

Para escrever basta utilizar o field criado:

```C#
_log4net.Error("Teste Log Log4Net");
```

## Serilog

No arquivo Startup.cs, alterar o construtor da classe Startup para:

```C#
using Serilog;
```
```C#
public Startup(IConfiguration configuration)
{
  Configuration = configuration;
  
  // Só isso para começar gravar em arquivo
  Log.Logger = new LoggerConfiguration()
    .WriteTo.File("log/log.txt", Serilog.Events.LogEventLevel.Information)
    .CreateLogger();
}
```

Para escrever no log, basta utilizar:

```C#
using Serilog;
```
```C#
Log.Error("Teste Log Serilog");
```
