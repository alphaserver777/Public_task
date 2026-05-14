У тебя ошибки потому, что проект создан, скорее всего, как **.NET Framework / старый C# 7.3**, а я дал код под более новый C# / .NET 6.

Исправляем просто: оставляем твой проект, но вставляем код, совместимый со старым C#.

---

# 1. Сначала установи `System.Text.Json`

В Visual Studio:

```text
Проект → Управление пакетами NuGet
```

Потом:

```text
Обзор → System.Text.Json → Установить
```

Или через консоль диспетчера пакетов:

```powershell
Install-Package System.Text.Json
```

Это исправит ошибки:

```text
JsonSerializer не существует
JsonSerializerOptions не найден
System.Text.Json не существует
```

---

# 2. Теперь полностью замени `Form1.cs`

Открой:

```text
Form1.cs
```

Выдели всё:

```text
Ctrl + A
```

Удали и вставь **весь этот код**:

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
            using (FolderBrowserDialog dialog = new FolderBrowserDialog())
            {
                if (dialog.ShowDialog() == DialogResult.OK)
                {
                    txtFolderPath.Text = dialog.SelectedPath;
                    statusLabel.Text = "Папка выбрана";
                }
            }
        }

        private void BtnCreateSnapshot_Click(object sender, EventArgs e)
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

            using (SaveFileDialog saveDialog = new SaveFileDialog())
            {
                saveDialog.Filter = "JSON files (*.json)|*.json";
                saveDialog.FileName = "snapshot.json";

                if (saveDialog.ShowDialog() == DialogResult.OK)
                {
                    JsonSerializerOptions options = new JsonSerializerOptions();
                    options.WriteIndented = true;

                    string json = JsonSerializer.Serialize(entries, options);

                    File.WriteAllText(saveDialog.FileName, json);

                    statusLabel.Text = "Эталон создан. Файлов: " + entries.Count;

                    MessageBox.Show(
                        "Эталонный снимок успешно создан.",
                        "Готово",
                        MessageBoxButtons.OK,
                        MessageBoxIcon.Information
                    );
                }
            }
        }

        private void BtnCheckIntegrity_Click(object sender, EventArgs e)
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

            using (OpenFileDialog openDialog = new OpenFileDialog())
            {
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
                    "Всего: " + total +
                    "; OK: " + okCount +
                    "; Изменены: " + modifiedCount +
                    "; Удалены: " + deletedCount +
                    "; Добавлены: " + addedCount;

                MessageBox.Show(
                    "Проверка завершена.",
                    "Готово",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information
                );
            }
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
                    string relativePath = GetRelativePath(folderPath, file);
                    string hash = ComputeSha256(file);

                    FileEntry entry = new FileEntry();
                    entry.RelativePath = relativePath;
                    entry.Sha256 = hash;

                    entries.Add(entry);
                }
                catch
                {
                    // Если файл нельзя прочитать, он пропускается.
                }
            }

            return entries;
        }

        private static string GetRelativePath(string basePath, string fullPath)
        {
            if (!basePath.EndsWith(Path.DirectorySeparatorChar.ToString()))
            {
                basePath += Path.DirectorySeparatorChar;
            }

            Uri baseUri = new Uri(basePath);
            Uri fullUri = new Uri(fullPath);

            string relativePath = Uri.UnescapeDataString(
                baseUri.MakeRelativeUri(fullUri).ToString()
            );

            relativePath = relativePath.Replace('/', Path.DirectorySeparatorChar);

            return relativePath;
        }

        private static string ComputeSha256(string filePath)
        {
            using (SHA256 sha256 = SHA256.Create())
            {
                using (FileStream stream = File.OpenRead(filePath))
                {
                    byte[] hashBytes = sha256.ComputeHash(stream);

                    string hash = BitConverter
                        .ToString(hashBytes)
                        .Replace("-", "")
                        .ToLowerInvariant();

                    return hash;
                }
            }
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

        public FileEntry()
        {
            RelativePath = "";
            Sha256 = "";
        }
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

# 3. Почему этот код должен исправить ошибки

Я убрал из кода новые возможности C#:

```text
using FolderBrowserDialog dialog = ...
object? sender
List<FileEntry>?
Path.GetRelativePath
$"строки"
```

И заменил их на старый синтаксис, который работает в проектах с C# 7.3.

---

# 4. После вставки

Нажми:

```text
Ctrl + S
```

Потом:

```text
Сборка → Перестроить решение
```

или:

```text
Build → Rebuild Solution
```

---

# 5. Если останется только ошибка `System.Text.Json`

Тогда проблема только в пакете.

Сделай так:

```text
Проект → Управление пакетами NuGet → Обзор → System.Text.Json → Установить
```

После установки снова:

```text
Сборка → Перестроить решение
```

У тебя проект сейчас старого типа, поэтому главный фикс — **поставить System.Text.Json через NuGet и вставить этот совместимый код**.
