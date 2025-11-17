# 🎯 ШАГ 2 — Подготовим место для трёх команд в Ribbon

> Сейчас у тебя в Ribbon создаётся одна кнопка:

[Commands]
   └── Execute


* Мы сделаем так: *
```
[Commands]
   ├── Load XML  
   ├── Create Shared Params  
   └── Apply XML Data
```

 ##### Но сначала нужно объяснить как Revit понимает, какую команду запускать.

># 🧠 Как Revit запускает команду?

Кнопка в Ribbon работает только если ты укажешь класс ExternalCommand.

* Пример: *
```C#
panel.AddPushButton<StartupCommand>("Execute")
```

<u >Тут Revit видит: если нажмут кнопку — создай объект StartupCommand и вызови метод Execute(). </u>

### Значит, чтобы сделать три кнопки, нам нужно три класса:

1. LoadXmlCommand

2. CreateParametersCommand

3. ApplyDataCommand

Каждый должен наследовать ExternalCommand.

## 📘 ШАГ 3 — Создаём первую команду: LoadXmlCommand

Мы перенесём твоё окно (testTaskView) сюда.
```C#
📄 testTask/Commands/LoadXmlCommand.cs
using Nice3point.Revit.Toolkit.External;
using testTask.ViewModels;
using testTask.Views;

namespace testTask.Commands
{
    [UsedImplicitly]
    [Transaction(TransactionMode.Manual)]
    public class LoadXmlCommand : ExternalCommand
    {
        public override void Execute()
        {
            var vm = new testTaskViewModel();
            var view = new testTaskView(vm);
            view.ShowDialog();
        }
    }
}
```

👉 Это почти копия твоего StartupCommand — просто новое имя, под отдельную кнопку.

## 📘 ШАГ 4 — Регистрируем кнопку в Ribbon

Теперь перепишем твой Application.cs.

Тебе станет полностью понятно, как Revit строит интерфейс.

```C# 📄 Application.cs (обновлённый)
using Nice3point.Revit.Toolkit.External;
using testTask.Commands;

namespace testTask
{
    [UsedImplicitly]
    public class Application : ExternalApplication
    {
        public override void OnStartup()
        {
            var panel = Application.CreatePanel("Commands", "testTask");

            // 1) Кнопка "Load XML"
            panel.AddPushButton<LoadXmlCommand>("Load XML")
                .SetImage("/testTask;component/Resources/Icons/RibbonIcon16.png")
                .SetLargeImage("/testTask;component/Resources/Icons/RibbonIcon32.png")
                .SetToolTip("Открыть XML файл и показать данные");

            // 2) Кнопка "Create Shared Parameters"
            panel.AddPushButton<CreateParametersCommand>("Create Params")
                .SetImage("/testTask;component/Resources/Icons/RibbonIcon16.png")
                .SetLargeImage("/testTask;component/Resources/Icons/RibbonIcon32.png")
                .SetToolTip("Создать shared параметры из XML");

            // 3) Кнопка "Apply Data to Rooms"
            panel.AddPushButton<ApplyDataCommand>("Apply Data")
                .SetImage("/testTask;component/Resources/Icons/RibbonIcon16.png")
                .SetLargeImage("/testTask;component/Resources/Icons/RibbonIcon32.png")
                .SetToolTip("Заполнить параметры помещений из XML");
        }
    }
}
```
> 🎓 Пояснение, что тут происходит (простым языком)


▶ panel.AddPushButton<LoadXmlCommand>("Load XML")

— создаёт кнопку
— пишет на ней текст
— привязывает её к классу LoadXmlCommand
— значит Revit знает, что запускать

▶ .SetImage(...)

— иконка маленькая

▶ .SetLargeImage(...)

— иконка большая

▶ .SetToolTip("...")

— всплывающая подсказка при наведении

## 🎓 ШАГ 5 — Что значат атрибуты над классами Revit команд?

Ты видишь такие штуки:

- [UsedImplicitly]
- [Transaction(TransactionMode.Manual)]
``` public class LoadXmlCommand : ExternalCommand```


#### Давай разберём каждую.

### 🟦 1. [UsedImplicitly] — атрибут Nice3point / JetBrains
📌 Простыми словами:

**Он говорит Visual Studio/Resharper:**
> “Не удаляйте этот класс как ‘неиспользуемый’, Revit будет вызывать его извне.”

*Зачем?*

>Потому что Revit не вызывает команду через C# код.

>Он вызывает её через .addin файл, который создаёт Nice3point toolkit.

>То есть в коде нет ссылок вида:

```var cmd = new LoadXmlCommand();```


>И VS может решить: “Раз этот класс нигде не используется — можно удалить”.

#### А атрибут говорит:

👉 "Нет, этот класс используется динамически Revit, не трогай!"

### 🟦 2. [Transaction(TransactionMode.Manual)] — атрибут Revit API

Это очень важный атрибут.

**В Revit любое изменение модели (создание элементов, запись параметров, удаление чего-либо) должно происходить внутри транзакции.**

📌 Что такое Transaction?

>Это как BEGIN … COMMIT в базе данных.

>Ты открываешь транзакцию

>Делаешь изменения

>Закрываешь (commit)

>Revit обновляет модель

**Без транзакции Revit не разрешит менять модель.**

📌 Что делает атрибут TransactionMode.Manual?
Он говорит Revit:

👉 “Эта команда сама будет открывать транзакции вручную.”

То есть код:
```C#
using (Transaction t = new Transaction(doc))
{
    t.Start("Название");
    ...
    t.Commit();
}
```

#### Это твой контроль.

#### Если бы стояло:

```TransactionMode.Automatic```

#### Revit сам бы открывал и закрывал транзакцию за тебя.

#### Но это плохо, потому что:

#### Транзакция открывается даже если ты просто показываешь окно

#### Нельзя делать вложенные транзакции

#### Возникают ошибки при сложной логике

####Поэтому почти всегда стоит Manual.

