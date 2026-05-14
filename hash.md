# 0. Важный момент: Visual Studio Code или Visual Studio?

Есть две разные программы:

## Visual Studio Code

Это лёгкий редактор кода. В нём можно писать C#, но формы мышкой делать неудобно.

## Visual Studio

Это большая среда разработки. Для Windows Forms она удобнее, потому что там есть визуальный конструктор формы.

Для этой лабораторной **лучше использовать Visual Studio**, не Visual Studio Code.

Но если тебе нужно именно **Visual Studio Code**, то мы можем сделать проект через команды и вручную написать код формы.

Ниже объясню именно через **Visual Studio Code**.

---

# 1. Что нужно установить

Тебе понадобится:

1. **Visual Studio Code**
2. **.NET SDK**
3. Расширение C# для VS Code

---

## 1.1. Проверяем, установлен ли .NET

Открой терминал.

В Windows можно нажать:

```text
Win + R
```

потом ввести:

```text
cmd
```

или открыть PowerShell.

Введи команду:

```bash
dotnet --version
```

Если видишь что-то вроде:

```text
8.0.204
```

или

```text
6.0.ххх
```

значит .NET установлен.

Если пишет, что команда не найдена, нужно установить **.NET SDK**.

---

# 2. Создаём папку для проекта

Создай папку, например:

```text
C:\Projects\FileIntegrityControl
```

Можно сделать через проводник Windows.

Или через терминал:

```bash
mkdir C:\Projects\FileIntegrityControl
```

Потом перейти в неё:

```bash
cd C:\Projects\FileIntegrityControl
```

---

# 3. Создаём Windows Forms проект

В терминале внутри папки выполни:

```bash
dotnet new winforms -n FileIntegrityControl
```

Что это значит:

```text
dotnet new winforms
```

создать новый проект Windows Forms.

```text
-n FileIntegrityControl
```

название проекта.

После этого появится папка:

```text
FileIntegrityControl
```

Перейди в неё:

```bash
cd FileIntegrityControl
```

---

# 4. Открываем проект в Visual Studio Code

В этой же папке выполни:

```bash
code .
```

Откроется Visual Studio Code с твоим проектом.

Если команда `code .` не работает, открой VS Code вручную:

```text
File → Open Folder → выбери папку FileIntegrityControl
```

---

# 5. Что ты увидишь в проекте

Примерно такие файлы:

```text
FileIntegrityControl
│
├── FileIntegrityControl.csproj
├── Program.cs
├── Form1.cs
├── Form1.Designer.cs
└── Form1.resx
```

Нас в основном интересуют:

```text
Form1.cs
Form1.Designer.cs
```

---

# 6. Проверяем, запускается ли пустой проект

В терминале VS Code выполни:

```bash
dotnet run
```

Должно открыться пустое окно Windows Forms.

Если окно открылось — отлично. Значит проект работает.

---

# 7. Что такое Form1.cs и Form1.Designer.cs

## Form1.cs

Это файл, куда мы пишем свою логику:

```text
что делать при нажатии кнопки
как считать хеш
как сохранить JSON
как проверить файлы
```

## Form1.Designer.cs

Это файл, где описан интерфейс:

```text
кнопки
текстовое поле
таблица
нижняя строка состояния
размер окна
расположение элементов
```

В Visual Studio это обычно создаётся мышкой.
В VS Code нам придётся написать это кодом.

---

# 8. Сначала делаем интерфейс

Открой файл:

```text
Form1.cs
```

Полностью замени его содержимое на этот код:

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Security.Cryptography;
using System.Text.Json;
using System.Windows.Forms;

namespace FileIntegrityControl
{
    public partial class Form1 : Form
    {
        private TextBox txtFolderPath;
        private Button btnSelectFolder;
        private Button btnCreateSnapshot;
        private Button btnCheckIntegrity;
        private DataGridView dataGridViewResults;
        private StatusStrip statusStrip;
        private ToolStripStatusLabel statusLabel;

        public Form1()
        {
            InitializeComponent();
            CreateInterface();
        }

        private void CreateInterface()
        {
            Text = "Контроль целостности файлов";
            Width = 1000;
            Height = 600;
            StartPosition = FormStartPosition.CenterScreen;

            txtFolderPath = new TextBox();
            txtFolderPath.Left = 10;
            txtFolderPath.Top = 10;
            txtFolderPath.Width = 760;
            txtFolderPath.ReadOnly = true;

            btnSelectFolder = new Button();
            btnSelectFolder.Left = 780;
            btnSelectFolder.Top = 8;
            btnSelectFolder.Width = 180;
            btnSelectFolder.Text = "Выбрать папку";
            btnSelectFolder.Click += BtnSelectFolder_Click;

            btnCreateSnapshot = new Button();
            btnCreateSnapshot.Left = 10;
            btnCreateSnapshot.Top = 45;
            btnCreateSnapshot.Width = 180;
            btnCreateSnapshot.Text = "Создать эталон";
            btnCreateSnapshot.Click += BtnCreateSnapshot_Click;

            btnCheckIntegrity = new Button();
            btnCheckIntegrity.Left = 200;
            btnCheckIntegrity.Top = 45;
            btnCheckIntegrity.Width = 200;
            btnCheckIntegrity.Text = "Загрузить и проверить";
            btnCheckIntegrity.Click += BtnCheckIntegrity_Click;

            dataGridViewResults = new DataGridView();
            dataGridViewResults.Left = 10;
            dataGridViewResults.Top = 85;
            dataGridViewResults.Width = 950;
            dataGridViewResults.Height = 420;
            dataGridViewResults.AllowUserToAddRows = false;
            dataGridViewResults.ReadOnly = true;
            dataGridViewResults.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;

            dataGridViewResults.Columns.Add("File", "Файл");
            dataGridViewResults.Columns.Add("Status", "Статус");
            dataGridViewResults.Columns.Add("ReferenceHash", "Хеш эталонный");
            dataGridViewResults.Columns.Add("CurrentHash", "Хеш текущий");

            statusStrip = new StatusStrip();
            statusLabel = new ToolStripStatusLabel();
            statusLabel.Text = "Готово";
            statusStrip.Items.Add(statusLabel);

            Controls.Add(txtFolderPath);
            Controls.Add(btnSelectFolder);
            Controls.Add(btnCreateSnapshot);
            Controls.Add(btnCheckIntegrity);
            Controls.Add(dataGridViewResults);
            Controls.Add(statusStrip);
        }
    }
}
```

Теперь запусти:

```bash
dotnet run
```

Должно появиться окно с:

```text
полем пути
кнопкой выбора папки
кнопкой создания эталона
кнопкой проверки
таблицей
нижней строкой состояния
```

Пока кнопки будут ругаться, потому что обработчики ещё не написаны.

---

# 9. Добавляем класс FileEntry

Внутри `Form1.cs`, но **после закрывающей фигурной скобки класса Form1**, добавь:

```csharp
public class FileEntry
{
    public string RelativePath { get; set; }
    public string Sha256 { get; set; }
}
```

В конце файла должно быть примерно так:

```csharp
    }
}

public class FileEntry
{
    public string RelativePath { get; set; }
    public string Sha256 { get; set; }
}
```

Этот класс нужен для хранения информации о файле:

```text
относительный путь
SHA-256 хеш
```

---

# 10. Добавляем перечисление FileStatus

Ниже класса `FileEntry` добавь:

```csharp
public enum FileStatus
{
    OK,
    Modified,
    Deleted,
    Added
}
```

Получится:

```csharp
public class FileEntry
{
    public string RelativePath { get; set; }
    public string Sha256 { get; set; }
}

public enum FileStatus
{
    OK,
    Modified,
    Deleted,
    Added
}
```

---

# 11. Добавляем кнопку выбора папки

Внутрь класса `Form1`, ниже метода `CreateInterface()`, добавь:

```csharp
private void BtnSelectFolder_Click(object sender, EventArgs e)
{
    using FolderBrowserDialog dialog = new FolderBrowserDialog();

    if (dialog.ShowDialog() == DialogResult.OK)
    {
        txtFolderPath.Text = dialog.SelectedPath;
        statusLabel.Text = "Папка выбрана";
    }
}
```

Что делает этот код:

```text
открывает окно выбора папки
пользователь выбирает папку
путь записывается в текстовое поле
```

---

# 12. Добавляем метод вычисления SHA-256

Внутрь класса `Form1` добавь:

```csharp
private static string ComputeSha256(string filePath)
{
    using SHA256 sha256 = SHA256.Create();
    using FileStream stream = File.OpenRead(filePath);

    byte[] hashBytes = sha256.ComputeHash(stream);

    return BitConverter
        .ToString(hashBytes)
        .Replace("-", "")
        .ToLowerInvariant();
}
```

Это главный метод лабораторной.

Он принимает путь к файлу:

```text
C:\Test\a.txt
```

и возвращает хеш:

```text
9f86d081884c7d659a2feaa0c55ad015...
```

---

# 13. Добавляем метод сканирования папки

Внутрь класса `Form1` добавь:

```csharp
private List<FileEntry> ScanFolder(string folderPath)
{
    List<FileEntry> entries = new List<FileEntry>();

    string[] files = Directory.GetFiles(
        folderPath,
        "*",
        SearchOption.AllDirectories
    );

    foreach (string file in files)
    {
        try
        {
            string relativePath = Path.GetRelativePath(folderPath, file);
            string hash = ComputeSha256(file);

            entries.Add(new FileEntry
            {
                RelativePath = relativePath,
                Sha256 = hash
            });
        }
        catch
        {
            // Если файл нельзя прочитать, пропускаем его
        }
    }

    return entries;
}
```

Что делает этот метод:

```text
берёт выбранную папку
находит все файлы внутри неё
включая вложенные папки
для каждого файла считает SHA-256
возвращает список файлов
```

---

# 14. Добавляем кнопку «Создать эталон»

Внутрь класса `Form1` добавь:

```csharp
private void BtnCreateSnapshot_Click(object sender, EventArgs e)
{
    string folderPath = txtFolderPath.Text;

    if (!Directory.Exists(folderPath))
    {
        MessageBox.Show("Сначала выберите существующую папку");
        return;
    }

    List<FileEntry> entries = ScanFolder(folderPath);

    using SaveFileDialog saveDialog = new SaveFileDialog();
    saveDialog.Filter = "JSON files (*.json)|*.json";
    saveDialog.FileName = "snapshot.json";

    if (saveDialog.ShowDialog() == DialogResult.OK)
    {
        JsonSerializerOptions options = new JsonSerializerOptions
        {
            WriteIndented = true
        };

        string json = JsonSerializer.Serialize(entries, options);

        File.WriteAllText(saveDialog.FileName, json);

        statusLabel.Text = $"Эталон создан. Файлов: {entries.Count}";
        MessageBox.Show("Эталонный снимок успешно создан");
    }
}
```

Теперь логика такая:

```text
выбрал папку
нажал "Создать эталон"
программа посчитала хеши файлов
сохранила JSON-файл
```

---

# 15. Добавляем метод добавления строки в таблицу

Внутрь класса `Form1` добавь:

```csharp
private void AddResultRow(
    string fileName,
    FileStatus status,
    string referenceHash,
    string currentHash)
{
    int rowIndex = dataGridViewResults.Rows.Add(
        fileName,
        status.ToString(),
        referenceHash,
        currentHash
    );

    DataGridViewRow row = dataGridViewResults.Rows[rowIndex];

    switch (status)
    {
        case FileStatus.OK:
            row.DefaultCellStyle.BackColor = Color.LightGreen;
            break;

        case FileStatus.Modified:
            row.DefaultCellStyle.BackColor = Color.LightYellow;
            break;

        case FileStatus.Deleted:
            row.DefaultCellStyle.BackColor = Color.LightCoral;
            break;

        case FileStatus.Added:
            row.DefaultCellStyle.BackColor = Color.LightBlue;
            break;
    }
}
```

Этот метод нужен, чтобы красиво добавлять результат проверки в таблицу.

Цвета:

```text
зелёный — файл не изменился
жёлтый — файл изменился
красный — файл удалён
синий — файл добавлен
```

---

# 16. Добавляем кнопку «Загрузить и проверить»

Внутрь класса `Form1` добавь:

```csharp
private void BtnCheckIntegrity_Click(object sender, EventArgs e)
{
    string folderPath = txtFolderPath.Text;

    if (!Directory.Exists(folderPath))
    {
        MessageBox.Show("Сначала выберите существующую папку");
        return;
    }

    using OpenFileDialog openDialog = new OpenFileDialog();
    openDialog.Filter = "JSON files (*.json)|*.json";

    if (openDialog.ShowDialog() != DialogResult.OK)
    {
        return;
    }

    string json = File.ReadAllText(openDialog.FileName);

    List<FileEntry> referenceEntries =
        JsonSerializer.Deserialize<List<FileEntry>>(json);

    if (referenceEntries == null)
    {
        MessageBox.Show("Не удалось прочитать файл эталона");
        return;
    }

    List<FileEntry> currentEntries = ScanFolder(folderPath);

    Dictionary<string, string> referenceDict =
        referenceEntries.ToDictionary(x => x.RelativePath, x => x.Sha256);

    Dictionary<string, string> currentDict =
        currentEntries.ToDictionary(x => x.RelativePath, x => x.Sha256);

    dataGridViewResults.Rows.Clear();

    int okCount = 0;
    int modifiedCount = 0;
    int deletedCount = 0;
    int addedCount = 0;

    foreach (KeyValuePair<string, string> referenceFile in referenceDict)
    {
        string relativePath = referenceFile.Key;
        string referenceHash = referenceFile.Value;

        if (!currentDict.ContainsKey(relativePath))
        {
            AddResultRow(relativePath, FileStatus.Deleted, referenceHash, "");
            deletedCount++;
        }
        else
        {
            string currentHash = currentDict[relativePath];

            if (referenceHash == currentHash)
            {
                AddResultRow(relativePath, FileStatus.OK, referenceHash, currentHash);
                okCount++;
            }
            else
            {
                AddResultRow(relativePath, FileStatus.Modified, referenceHash, currentHash);
                modifiedCount++;
            }
        }
    }

    foreach (KeyValuePair<string, string> currentFile in currentDict)
    {
        string relativePath = currentFile.Key;
        string currentHash = currentFile.Value;

        if (!referenceDict.ContainsKey(relativePath))
        {
            AddResultRow(relativePath, FileStatus.Added, "", currentHash);
            addedCount++;
        }
    }

    int total = okCount + modifiedCount + deletedCount + addedCount;

    statusLabel.Text =
        $"Всего: {total}; OK: {okCount}; Изменены: {modifiedCount}; Удалены: {deletedCount}; Добавлены: {addedCount}";

    MessageBox.Show("Проверка завершена");
}
```

---

# 17. Как должен выглядеть полный файл Form1.cs

У тебя должен получиться такой файл:

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.IO;
using System.Linq;
using System.Security.Cryptography;
using System.Text.Json;
using System.Windows.Forms;

namespace FileIntegrityControl
{
    public partial class Form1 : Form
    {
        private TextBox txtFolderPath;
        private Button btnSelectFolder;
        private Button btnCreateSnapshot;
        private Button btnCheckIntegrity;
        private DataGridView dataGridViewResults;
        private StatusStrip statusStrip;
        private ToolStripStatusLabel statusLabel;

        public Form1()
        {
            InitializeComponent();
            CreateInterface();
        }

        private void CreateInterface()
        {
            Text = "Контроль целостности файлов";
            Width = 1000;
            Height = 600;
            StartPosition = FormStartPosition.CenterScreen;

            txtFolderPath = new TextBox();
            txtFolderPath.Left = 10;
            txtFolderPath.Top = 10;
            txtFolderPath.Width = 760;
            txtFolderPath.ReadOnly = true;

            btnSelectFolder = new Button();
            btnSelectFolder.Left = 780;
            btnSelectFolder.Top = 8;
            btnSelectFolder.Width = 180;
            btnSelectFolder.Text = "Выбрать папку";
            btnSelectFolder.Click += BtnSelectFolder_Click;

            btnCreateSnapshot = new Button();
            btnCreateSnapshot.Left = 10;
            btnCreateSnapshot.Top = 45;
            btnCreateSnapshot.Width = 180;
            btnCreateSnapshot.Text = "Создать эталон";
            btnCreateSnapshot.Click += BtnCreateSnapshot_Click;

            btnCheckIntegrity = new Button();
            btnCheckIntegrity.Left = 200;
            btnCheckIntegrity.Top = 45;
            btnCheckIntegrity.Width = 200;
            btnCheckIntegrity.Text = "Загрузить и проверить";
            btnCheckIntegrity.Click += BtnCheckIntegrity_Click;

            dataGridViewResults = new DataGridView();
            dataGridViewResults.Left = 10;
            dataGridViewResults.Top = 85;
            dataGridViewResults.Width = 950;
            dataGridViewResults.Height = 420;
            dataGridViewResults.AllowUserToAddRows = false;
            dataGridViewResults.ReadOnly = true;
            dataGridViewResults.AutoSizeColumnsMode = DataGridViewAutoSizeColumnsMode.Fill;

            dataGridViewResults.Columns.Add("File", "Файл");
            dataGridViewResults.Columns.Add("Status", "Статус");
            dataGridViewResults.Columns.Add("ReferenceHash", "Хеш эталонный");
            dataGridViewResults.Columns.Add("CurrentHash", "Хеш текущий");

            statusStrip = new StatusStrip();
            statusLabel = new ToolStripStatusLabel();
            statusLabel.Text = "Готово";
            statusStrip.Items.Add(statusLabel);

            Controls.Add(txtFolderPath);
            Controls.Add(btnSelectFolder);
            Controls.Add(btnCreateSnapshot);
            Controls.Add(btnCheckIntegrity);
            Controls.Add(dataGridViewResults);
            Controls.Add(statusStrip);
        }

        private void BtnSelectFolder_Click(object sender, EventArgs e)
        {
            using FolderBrowserDialog dialog = new FolderBrowserDialog();

            if (dialog.ShowDialog() == DialogResult.OK)
            {
                txtFolderPath.Text = dialog.SelectedPath;
                statusLabel.Text = "Папка выбрана";
            }
        }

        private void BtnCreateSnapshot_Click(object sender, EventArgs e)
        {
            string folderPath = txtFolderPath.Text;

            if (!Directory.Exists(folderPath))
            {
                MessageBox.Show("Сначала выберите существующую папку");
                return;
            }

            List<FileEntry> entries = ScanFolder(folderPath);

            using SaveFileDialog saveDialog = new SaveFileDialog();
            saveDialog.Filter = "JSON files (*.json)|*.json";
            saveDialog.FileName = "snapshot.json";

            if (saveDialog.ShowDialog() == DialogResult.OK)
            {
                JsonSerializerOptions options = new JsonSerializerOptions
                {
                    WriteIndented = true
                };

                string json = JsonSerializer.Serialize(entries, options);

                File.WriteAllText(saveDialog.FileName, json);

                statusLabel.Text = $"Эталон создан. Файлов: {entries.Count}";
                MessageBox.Show("Эталонный снимок успешно создан");
            }
        }

        private void BtnCheckIntegrity_Click(object sender, EventArgs e)
        {
            string folderPath = txtFolderPath.Text;

            if (!Directory.Exists(folderPath))
            {
                MessageBox.Show("Сначала выберите существующую папку");
                return;
            }

            using OpenFileDialog openDialog = new OpenFileDialog();
            openDialog.Filter = "JSON files (*.json)|*.json";

            if (openDialog.ShowDialog() != DialogResult.OK)
            {
                return;
            }

            string json = File.ReadAllText(openDialog.FileName);

            List<FileEntry> referenceEntries =
                JsonSerializer.Deserialize<List<FileEntry>>(json);

            if (referenceEntries == null)
            {
                MessageBox.Show("Не удалось прочитать файл эталона");
                return;
            }

            List<FileEntry> currentEntries = ScanFolder(folderPath);

            Dictionary<string, string> referenceDict =
                referenceEntries.ToDictionary(x => x.RelativePath, x => x.Sha256);

            Dictionary<string, string> currentDict =
                currentEntries.ToDictionary(x => x.RelativePath, x => x.Sha256);

            dataGridViewResults.Rows.Clear();

            int okCount = 0;
            int modifiedCount = 0;
            int deletedCount = 0;
            int addedCount = 0;

            foreach (KeyValuePair<string, string> referenceFile in referenceDict)
            {
                string relativePath = referenceFile.Key;
                string referenceHash = referenceFile.Value;

                if (!currentDict.ContainsKey(relativePath))
                {
                    AddResultRow(relativePath, FileStatus.Deleted, referenceHash, "");
                    deletedCount++;
                }
                else
                {
                    string currentHash = currentDict[relativePath];

                    if (referenceHash == currentHash)
                    {
                        AddResultRow(relativePath, FileStatus.OK, referenceHash, currentHash);
                        okCount++;
                    }
                    else
                    {
                        AddResultRow(relativePath, FileStatus.Modified, referenceHash, currentHash);
                        modifiedCount++;
                    }
                }
            }

            foreach (KeyValuePair<string, string> currentFile in currentDict)
            {
                string relativePath = currentFile.Key;
                string currentHash = currentFile.Value;

                if (!referenceDict.ContainsKey(relativePath))
                {
                    AddResultRow(relativePath, FileStatus.Added, "", currentHash);
                    addedCount++;
                }
            }

            int total = okCount + modifiedCount + deletedCount + addedCount;

            statusLabel.Text =
                $"Всего: {total}; OK: {okCount}; Изменены: {modifiedCount}; Удалены: {deletedCount}; Добавлены: {addedCount}";

            MessageBox.Show("Проверка завершена");
        }

        private List<FileEntry> ScanFolder(string folderPath)
        {
            List<FileEntry> entries = new List<FileEntry>();

            string[] files = Directory.GetFiles(
                folderPath,
                "*",
                SearchOption.AllDirectories
            );

            foreach (string file in files)
            {
                try
                {
                    string relativePath = Path.GetRelativePath(folderPath, file);
                    string hash = ComputeSha256(file);

                    entries.Add(new FileEntry
                    {
                        RelativePath = relativePath,
                        Sha256 = hash
                    });
                }
                catch
                {
                    // Если файл нельзя прочитать, пропускаем его
                }
            }

            return entries;
        }

        private static string ComputeSha256(string filePath)
        {
            using SHA256 sha256 = SHA256.Create();
            using FileStream stream = File.OpenRead(filePath);

            byte[] hashBytes = sha256.ComputeHash(stream);

            return BitConverter
                .ToString(hashBytes)
                .Replace("-", "")
                .ToLowerInvariant();
        }

        private void AddResultRow(
            string fileName,
            FileStatus status,
            string referenceHash,
            string currentHash)
        {
            int rowIndex = dataGridViewResults.Rows.Add(
                fileName,
                status.ToString(),
                referenceHash,
                currentHash
            );

            DataGridViewRow row = dataGridViewResults.Rows[rowIndex];

            switch (status)
            {
                case FileStatus.OK:
                    row.DefaultCellStyle.BackColor = Color.LightGreen;
                    break;

                case FileStatus.Modified:
                    row.DefaultCellStyle.BackColor = Color.LightYellow;
                    break;

                case FileStatus.Deleted:
                    row.DefaultCellStyle.BackColor = Color.LightCoral;
                    break;

                case FileStatus.Added:
                    row.DefaultCellStyle.BackColor = Color.LightBlue;
                    break;
            }
        }
    }

    public class FileEntry
    {
        public string RelativePath { get; set; }
        public string Sha256 { get; set; }
    }

    public enum FileStatus
    {
        OK,
        Modified,
        Deleted,
        Added
    }
}
```

---

# 18. Запускаем программу

В терминале VS Code:

```bash
dotnet run
```

Откроется окно приложения.

---

# 19. Как проверить, что всё работает

Создай тестовую папку, например:

```text
C:\TestIntegrity
```

Внутри создай три файла:

```text
a.txt
b.txt
c.txt
```

Например:

```text
a.txt → Привет
b.txt → Тест
c.txt → Файл без изменений
```

---

## Шаг 1. Создаём эталон

В программе:

1. Нажми **Выбрать папку**.
2. Выбери `C:\TestIntegrity`.
3. Нажми **Создать эталон**.
4. Сохрани файл как:

```text
snapshot.json
```

Теперь программа запомнила исходное состояние папки.

---

## Шаг 2. Меняем файлы

Теперь вручную сделай изменения:

1. Открой `a.txt` и измени текст.
2. Удали `b.txt`.
3. `c.txt` не трогай.
4. Создай новый файл `d.txt`.

Теперь папка изменилась.

---

## Шаг 3. Проверяем

В программе:

1. Нажми **Загрузить и проверить**.
2. Выбери `snapshot.json`.

В таблице должно быть:

```text
a.txt — Modified
b.txt — Deleted
c.txt — OK
d.txt — Added
```

---

# 20. Самое главное, что нужно понять

Программа работает так:

```text
1. Первый раз сканирует папку.
2. Сохраняет список файлов и их хеши в JSON.
3. Потом сканирует папку ещё раз.
4. Сравнивает старые хеши с новыми.
5. Показывает, что изменилось.
```

То есть JSON-файл — это **эталонный снимок**.

Он отвечает на вопрос:

```text
Как папка выглядела раньше?
```

А повторное сканирование отвечает на вопрос:

```text
Как папка выглядит сейчас?
```

Сравнение даёт результат:

```text
OK / Modified / Deleted / Added
```

---

# 21. Возможные ошибки

## Ошибка 1. `dotnet` не найден

Значит не установлен .NET SDK.

---

## Ошибка 2. `The template "winforms" was not found`

Значит установлен не тот .NET SDK или проект создаётся не на Windows.

Windows Forms нормально работает только на Windows.

---

## Ошибка 3. Кнопки не работают

Проверь, что в коде есть строки:

```csharp
btnSelectFolder.Click += BtnSelectFolder_Click;
btnCreateSnapshot.Click += BtnCreateSnapshot_Click;
btnCheckIntegrity.Click += BtnCheckIntegrity_Click;
```

Они связывают кнопки с методами.

---

## Ошибка 4. Красное подчёркивание `JsonSerializer`

Проверь, что сверху есть:

```csharp
using System.Text.Json;
```

---

## Ошибка 5. Красное подчёркивание `SHA256`

Проверь, что сверху есть:

```csharp
using System.Security.Cryptography;
```

---

## Ошибка 6. Красное подчёркивание `DataGridView`

Проверь, что сверху есть:

```csharp
using System.Windows.Forms;
```

