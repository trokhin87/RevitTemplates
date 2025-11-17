# 📘 ШАГ 1 — Что такое ExternalApplication в Revit

## У тебя есть такой класс:
```C#
public class Application : ExternalApplication
{
    public override void OnStartup()
    {
        var panel = Application.CreatePanel("Commands", "testTask");

        panel.AddPushButton<StartupCommand>("Execute")
            .SetImage(...)
            .SetLargeImage(...)
            .SetToolTip("Запустить XML-парсер комнат");
    }
}
```
---
> Разберём это по-человечески.

## 🧠 Revit загружает add-in двумя способами
### 1) ExternalApplication — запускается при старте Revit

* Это как “инициализация плагина”.

Revit вызывает:

- OnStartup() — когда Revit запускается

- OnShutdown() — когда Revit закрывается

* То есть этот класс создаёт вкладки, кнопки, панели — всё, что относится к Ribbon UI.

### 2) ExternalCommand — запускается по нажатию кнопки

Это твой:
```C#
public class StartupCommand : ExternalCommand
{
    public override void Execute()
    {
        var viewModel = new testTaskViewModel();
        var view = new testTaskView(viewModel);
        view.ShowDialog();
    }
}
```

* ExternalCommand — это “действие по клику”.

## 🧩 Итак:

ExternalApplication → создаёт кнопки

ExternalCommand → выполняется, когда кнопку нажали
