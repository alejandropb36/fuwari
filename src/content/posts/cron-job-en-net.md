---
title: Cron jobs en .NET
published: 2025-02-28
description: 'NuGet para crear cron jobs en .NET'
image: ''
tags: [.Net, Cron, Programación]
category: 'Programación'
draft: false
lang: ''
---

# Procesos en segundo plano

En el desarrollo de backend es común encontrarse con casos de uso donde se requiere ejecutar procesos en segundo plano, ya sea en un servicio web o en una aplicación de consola.

Dada esta necesidad y escenario común, hay librerías muy conocidas en el mundo .NET, como lo son Hangfire y Quartz.

Ya he usado estas librerías en equipos; sin embargo, quería hacer mi propia implementación de esta funcionalidad, ya que en .NET existen los `BackgroundService` y se puede implementar de una manera muy sencilla.

Esto me llevó a la implementación que les voy a presentar.

# SimpleCronWorkerService

Se logra mediante una clase abstracta, en donde las implementaciones heredarán de ella:

```csharp
using Cronos;
using Microsoft.Extensions.Hosting;
using System;
using System.Threading;
using System.Threading.Tasks;
using Timer = System.Timers.Timer;

namespace SimpleCronWorkerService
{
    public abstract class CronWorkerService : BackgroundService
    {
        private Timer _timer;
        private readonly CronExpression _cronExpression;
        private readonly TimeZoneInfo _timeZone;
        private const int DelayMaxValueMilliseconds = (int.MaxValue - (60 * 1000));

        public CronWorkerService(ICronWorkerServiceSettings settings)
        {
            _cronExpression = settings.CronExpressionIncludeSeconds
                ? CronExpression.Parse(settings.CronExpression, CronFormat.IncludeSeconds)
                : CronExpression.Parse(settings.CronExpression);

            _timeZone = settings.TimeZone ?? TimeZoneInfo.Utc;
            _timer = new Timer { AutoReset = false };
        }

        protected override async Task ExecuteAsync(CancellationToken cancellationToken)
        {
            await RestartTimer(cancellationToken);

            _timer.Elapsed += async (sender, args) =>
            {
                _timer.Stop();
                if (!cancellationToken.IsCancellationRequested)
                {
                    var doWorkTask = DoWork(cancellationToken);
                    var restartTimerTask = RestartTimer(cancellationToken);

                    await doWorkTask;
                    await restartTimerTask;
                }
            };

            await Task.CompletedTask;
        }

        private async Task RestartTimer(CancellationToken cancellationToken)
        {
            var next = _cronExpression.GetNextOccurrence(DateTimeOffset.Now, _timeZone);
            if (!next.HasValue)
            {
                return;
            }

            var delay = GetDelay(next.Value);

            while (delay.TotalMilliseconds > int.MaxValue)
            {
                await Task.Delay(DelayMaxValueMilliseconds, cancellationToken);
                delay = GetDelay(next.Value);
            }

            if (delay.TotalMilliseconds <= 0)
            {
                await RestartTimer(cancellationToken);
                return;
            }

            _timer.Interval = delay.TotalMilliseconds;
            _timer.Start();
        }

        private TimeSpan GetDelay(DateTimeOffset nextValue) => nextValue - DateTimeOffset.Now;

        protected abstract Task DoWork(CancellationToken cancellationToken);
    }
}
```

Como podemos ver, aquí tenemos una dependencia de la librería `Cronos`, la cual está desarrollada por el equipo de [Hangfire - Cronos](https://github.com/HangfireIO/Cronos). Esta librería solo nos ayuda con la interpretación de la sintaxis de cron.

La parte principal de esta clase es el método `ExecuteAsync`:

```csharp
protected override async Task ExecuteAsync(CancellationToken cancellationToken)
{
    await RestartTimer(cancellationToken);

    _timer.Elapsed += async (sender, args) =>
    {
        _timer.Stop();

        if (!cancellationToken.IsCancellationRequested)
        {
            var doWorkTask = DoWork(cancellationToken);
            var restartTimerTask = RestartTimer(cancellationToken);

            await doWorkTask;
            await restartTimerTask;
        }
    };

    await Task.CompletedTask;
}
```

Como vemos, trabaja de forma asíncrona. La ventaja de esta implementación es que estoy utilizando el `Timer` de .NET Standard, lo que hace que sea más compatible con todo el ecosistema de tecnologías de .NET.

Algo importante a mencionar de esta implementación es que la parte de `RestartTimer`, que es el método que reprograma la siguiente ejecución, tiene una solución al problema común que he visto en este tipo de implementaciones.

El problema es que el `Interval` del `Timer` nativo de .NET soporta solo la cantidad máxima de un `int`, y si tu expresión cron define una ejecución para dentro de mucho tiempo, puede hacer que el proceso falle.
Para evitar esto, se usa el siguiente código:

```csharp
var delay = GetDelay(next.Value);

while (delay.TotalMilliseconds > int.MaxValue)
{
    await Task.Delay(DelayMaxValueMilliseconds, cancellationToken);
    delay = GetDelay(next.Value);
}
```

Esto hace que la `Task` se espere en rondas hasta que el `delay` sea menor al valor máximo de un `int`.

Con esto, se garantiza que no se exceda el valor máximo de acuerdo con ciertos valores.

Todo este código se encuentra en este repositorio creado por mí:

::github{repo="alejandropb36/SimpleCronWorkerService"}

Dentro del repositorio encontrarás ejemplos de uso, configuración y la manera adecuada de usar el paquete NuGet.

El paquete NuGet se encuentra publicado tanto en [NuGet.org](https://www.nuget.org/packages/SimpleCronWorkerService) como en [GitHub](https://github.com/alejandropb36/SimpleCronWorkerService/pkgs/nuget/SimpleCronWorkerService).

