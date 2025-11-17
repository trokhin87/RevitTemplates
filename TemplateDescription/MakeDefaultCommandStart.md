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

1. * LoadXmlCommand *

2. * CreateParametersCommand *

3. * ApplyDataCommand *

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

### 📌 Что дальше?

* Если всё понятно — мы создадим вторую команду: CreateParametersCommand, и я начну объяснять: *

- Что такое shared parameters

- Что такое shared parameter file

- Почему нельзя создавать параметры "на лету" без файла

- Как они привязываются к категории Rooms

- Как они появляются в Revit

