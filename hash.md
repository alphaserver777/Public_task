Задание требует Windows Forms-приложение на C#, которое выбирает папку, считает SHA-256 для файлов, сохраняет эталон в JSON и потом проверяет статусы `OK`, `Modified`, `Deleted`, `Added`. 

---

# 1. Создай проект

Открой терминал в VS Code или PowerShell и выполни:

```bash
dotnet new winforms -n FileIntegrityControl
```

Потом перейди в папку проекта:

```bash
cd FileIntegrityControl
```

Открой проект в VS Code:

```bash
code .
```

---

# 2. Открой файл `Form1.cs`

В VS Code слева найди файл:

```text
Form1.cs
```

Открой его.

---

# 3. Полностью удали всё из `Form1.cs`

Прямо выдели весь текст в файле:

```text
Ctrl + A
```

и удали.

---

# 4. Вставь туда ВЕСЬ этот код

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

        private void BtnSelectFolder_Click(object? sender, EventArgs e)
        {
            using FolderBrowserDialog dialog = new FolderBrowserDialog();

            if (dialog.ShowDialog() == DialogResult.OK)
            {
                txtFolderPath.Text = dialog.SelectedPath;
                statusLabel.Text = "Папка выбрана";
            }
        }

        private void BtnCreateSnapshot_Click(object? sender, EventArgs e)
        {
            string folderPath = txtFolderPath.Text;

            if (!Directory.Exists(folderPath))
            {
                MessageBox.Show(
                    "Сначала выберите существующую папку.",
                    "Ошибка",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning
                );

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

                MessageBox.Show(
                    "Эталонный снимок успешно создан.",
                    "Готово",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information
                );
            }
        }

        private void BtnCheckIntegrity_Click(object? sender, EventArgs e)
        {
            string folderPath = txtFolderPath.Text;

            if (!Directory.Exists(folderPath))
            {
                MessageBox.Show(
                    "Сначала выберите существующую папку.",
                    "Ошибка",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Warning
                );

                return;
            }

            using OpenFileDialog openDialog = new OpenFileDialog();
            openDialog.Filter = "JSON files (*.json)|*.json";

            if (openDialog.ShowDialog() != DialogResult.OK)
            {
                return;
            }

            string json = File.ReadAllText(openDialog.FileName);

            List<FileEntry>? referenceEntries =
                JsonSerializer.Deserialize<List<FileEntry>>(json);

            if (referenceEntries == null)
            {
                MessageBox.Show(
                    "Не удалось прочитать файл эталона.",
                    "Ошибка",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );

                return;
            }

            List<FileEntry> currentEntries = ScanFolder(folderPath);

            Dictionary<string, string> referenceDict =
                referenceEntries.ToDictionary(
                    item => item.RelativePath,
                    item => item.Sha256
                );

            Dictionary<string, string> currentDict =
                currentEntries.ToDictionary(
                    item => item.RelativePath,
                    item => item.Sha256
                );

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
                    AddResultRow(
                        relativePath,
                        FileStatus.Deleted,
                        referenceHash,
                        ""
                    );

                    deletedCount++;
                }
                else
                {
                    string currentHash = currentDict[relativePath];

                    if (referenceHash == currentHash)
                    {
                        AddResultRow(
                            relativePath,
                            FileStatus.OK,
                            referenceHash,
                            currentHash
                        );

                        okCount++;
                    }
                    else
                    {
                        AddResultRow(
                            relativePath,
                            FileStatus.Modified,
                            referenceHash,
                            currentHash
                        );

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
                    AddResultRow(
                        relativePath,
                        FileStatus.Added,
                        "",
                        currentHash
                    );

                    addedCount++;
                }
            }

            int total = okCount + modifiedCount + deletedCount + addedCount;

            statusLabel.Text =
                $"Всего: {total}; OK: {okCount}; Изменены: {modifiedCount}; Удалены: {deletedCount}; Добавлены: {addedCount}";

            MessageBox.Show(
                "Проверка завершена.",
                "Готово",
                MessageBoxButtons.OK,
                MessageBoxIcon.Information
            );
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

                    FileEntry entry = new FileEntry
                    {
                        RelativePath = relativePath,
                        Sha256 = hash
                    };

                    entries.Add(entry);
                }
                catch
                {
                    // Если файл нельзя прочитать, он пропускается.
                    // Это нужно, чтобы программа не падала из-за одного проблемного файла.
                }
            }

            return entries;
        }

        private static string ComputeSha256(string filePath)
        {
            using SHA256 sha256 = SHA256.Create();
            using FileStream stream = File.OpenRead(filePath);

            byte[] hashBytes = sha256.ComputeHash(stream);

            string hash = BitConverter
                .ToString(hashBytes)
                .Replace("-", "")
                .ToLowerInvariant();

            return hash;
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
        public string RelativePath { get; set; } = "";
        public string Sha256 { get; set; } = "";
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

# 5. Сохрани файл

Нажми:

```text
Ctrl + S
```

---

# 6. Запусти проект

В терминале внутри папки проекта выполни:

```bash
dotnet run
```

Должно открыться окно:

```text
Контроль целостности файлов
```

---

# 7. Как проверить работу

Создай папку, например:

```text
C:\TestIntegrity
```

Внутри создай файлы:

```text
a.txt
b.txt
c.txt
```

Потом в программе:

1. Нажми **Выбрать папку**.
2. Выбери `C:\TestIntegrity`.
3. Нажми **Создать эталон**.
4. Сохрани файл как `snapshot.json`.

Теперь измени папку:

1. В `a.txt` измени текст.
2. `b.txt` удали.
3. `c.txt` не трогай.
4. Создай новый файл `d.txt`.

Потом в программе:

1. Нажми **Загрузить и проверить**.
2. Выбери `snapshot.json`.

В таблице должно быть примерно так:

```text
a.txt — Modified
b.txt — Deleted
c.txt — OK
d.txt — Added
```

---

# 8. Если при запуске будет ошибка

Проверь, что ты находишься именно внутри папки проекта:

```bash
cd FileIntegrityControl
```

И там должен быть файл:

```text
FileIntegrityControl.csproj
```

После этого снова:

```bash
dotnet run
```
