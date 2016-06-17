---
layout: post
title:  "JSON конфигурация списка задач Hangfire и их runtime обновление"
date:   2016-06-17 23:30:29 +0300
category: dev
tags: [hangfire, scheduler]
---

Явное описание задач в коде делает их достаточно неповоротливыми и неудобными в поддержке. 
Также было бы явным излишеством описывать однотипные задачи.
Было бы удобнее хранить их минимальное описание с параметрами в отдельном файле с JSON или XML разметкой, 
который бы автоматически подгружался при старте планировщика и далее уже обновлял список при условии, если он был изменен. 
У известного планировщика Quartz, такая фича идет уже из коробки, но к сожалению, в Hangfire ее нет. 
<!-- more -->


Опишем общий интерфейс задачи.

```cs
public interface IJob
{
	void Execute(Dictionary<string, string> parameters);
}
```

Для начала зарегистрируем имеющиеся джобы, удобным нам DI-контейнером. Для примера используем Castle Windsor. 

Регистрируем все джобы на базе интерфейса IJob.

```cs
container.Register(Classes.FromThisAssembly().BasedOn<IJob>());
```


Устанавливаем из NuGet дополнительный пакет:

```
Install-Package HangFire.Windsor
```

И подключаем `JobActivator`

```cs
GlobalConfiguration.Configuration.UseActivator(new WindsorJobActivator(container.Kernel));
```


После того как наши джобы зарегистрированы, приступим непосредственно к механизму конфигурации шедулера. 
Создаем файл конфигурации config.json. И определим формат записи задач. 

```js
[
  {
    "Id": "1",
    "Name": "ExampleJob",
    "CronExpression": "0 0 * * *",
    "Parameters": {
      "Parameter1": "Text",
      "Parameter2": "123"
    }
  },…
]
```

`Name` будет выступать как в роли названия джобы, так и названия ее класса.