Есть два простых способа исправить.

---

# Вариант 1 — самый простой

Открой файл:

```text
Form1.Designer.cs
```

Найди строку:

```csharp
this.Load += new System.EventHandler(this.Form1_Load);
```

И просто удали её.

Потом сохрани:

```text
Ctrl + S
```

И снова собери проект.

---

# Вариант 2 — ничего не искать, просто добавить пустой метод

Открой файл:

```text
Form1.cs
```

Внутри класса `Form1`, например сразу после конструктора:

```csharp
public Form1()
{
    InitializeComponent();
    CreateInterface();
}
```

вставь вот это:

```csharp
private void Form1_Load(object sender, EventArgs e)
{
}
```

Должно получиться так:

```csharp
public Form1()
{
    InitializeComponent();
    CreateInterface();
}

private void Form1_Load(object sender, EventArgs e)
{
}
```

---

Лучше сделай **вариант 2**, он быстрее и безопаснее: просто добавь пустой метод `Form1_Load`, сохрани и запусти проект.
