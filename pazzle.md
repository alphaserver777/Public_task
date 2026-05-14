# 1. Создай новый проект

В Visual Studio создай:

```text
Windows Forms App
```

Название проекта, например:

```text
PuzzleGame
```

---

# 2. Открой файл `Form1.cs`

В проекте найди:

```text
Form1.cs
```

Открой его.

---

# 3. Полностью удали всё из `Form1.cs`

Нажми:

```text
Ctrl + A
```

Потом удали всё.

---

# 4. Вставь весь этот код

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace PuzzleGame
{
    public partial class Form1 : Form
    {
        private Panel panelOriginal;
        private Panel panelGame;
        private PictureBox pictureOriginal;

        private Button btnStart;
        private Button btnReset;

        private Label lblMoves;
        private Label lblStatus;

        private PictureBox[,] puzzleBoxes;
        private Bitmap[] originalPieces;

        private int gridSize = 4;
        private int pieceWidth;
        private int pieceHeight;

        private int movesCount = 0;
        private bool gameStarted = false;

        private PictureBox firstSelected = null;

        private Random random = new Random();

        public Form1()
        {
            InitializeComponent();
            CreateInterface();
        }

        private void Form1_Load(object sender, EventArgs e)
        {
        }

        private void CreateInterface()
        {
            Text = "Puzzle";
            Width = 1050;
            Height = 650;
            StartPosition = FormStartPosition.CenterScreen;
            FormBorderStyle = FormBorderStyle.FixedSingle;
            MaximizeBox = false;

            Label titleOriginal = new Label();
            titleOriginal.Text = "Исходное изображение";
            titleOriginal.Left = 20;
            titleOriginal.Top = 15;
            titleOriginal.Width = 250;
            titleOriginal.Height = 25;
            titleOriginal.Font = new Font("Arial", 11, FontStyle.Bold);

            panelOriginal = new Panel();
            panelOriginal.Left = 20;
            panelOriginal.Top = 45;
            panelOriginal.Width = 300;
            panelOriginal.Height = 300;
            panelOriginal.BorderStyle = BorderStyle.FixedSingle;

            pictureOriginal = new PictureBox();
            pictureOriginal.Left = 0;
            pictureOriginal.Top = 0;
            pictureOriginal.Width = panelOriginal.Width;
            pictureOriginal.Height = panelOriginal.Height;
            pictureOriginal.SizeMode = PictureBoxSizeMode.StretchImage;
            panelOriginal.Controls.Add(pictureOriginal);

            Label titleGame = new Label();
            titleGame.Text = "Игровое поле";
            titleGame.Left = 360;
            titleGame.Top = 15;
            titleGame.Width = 250;
            titleGame.Height = 25;
            titleGame.Font = new Font("Arial", 11, FontStyle.Bold);

            panelGame = new Panel();
            panelGame.Left = 360;
            panelGame.Top = 45;
            panelGame.Width = 560;
            panelGame.Height = 560;
            panelGame.BorderStyle = BorderStyle.FixedSingle;

            btnStart = new Button();
            btnStart.Text = "Новая игра";
            btnStart.Left = 20;
            btnStart.Top = 380;
            btnStart.Width = 140;
            btnStart.Height = 35;
            btnStart.Click += BtnStart_Click;

            btnReset = new Button();
            btnReset.Text = "Сброс";
            btnReset.Left = 180;
            btnReset.Top = 380;
            btnReset.Width = 140;
            btnReset.Height = 35;
            btnReset.Click += BtnReset_Click;

            lblMoves = new Label();
            lblMoves.Text = "Ходы: 0";
            lblMoves.Left = 20;
            lblMoves.Top = 440;
            lblMoves.Width = 300;
            lblMoves.Height = 30;
            lblMoves.Font = new Font("Arial", 12, FontStyle.Bold);

            lblStatus = new Label();
            lblStatus.Text = "Нажмите «Новая игра»";
            lblStatus.Left = 20;
            lblStatus.Top = 480;
            lblStatus.Width = 300;
            lblStatus.Height = 60;
            lblStatus.Font = new Font("Arial", 10, FontStyle.Regular);

            Controls.Add(titleOriginal);
            Controls.Add(panelOriginal);
            Controls.Add(titleGame);
            Controls.Add(panelGame);
            Controls.Add(btnStart);
            Controls.Add(btnReset);
            Controls.Add(lblMoves);
            Controls.Add(lblStatus);
        }

        private void BtnStart_Click(object sender, EventArgs e)
        {
            OpenFileDialog openDialog = new OpenFileDialog();
            openDialog.Filter = "Изображения|*.jpg;*.jpeg;*.png;*.bmp";

            if (openDialog.ShowDialog() != DialogResult.OK)
            {
                return;
            }

            Bitmap sourceImage;

            try
            {
                sourceImage = new Bitmap(openDialog.FileName);
            }
            catch
            {
                MessageBox.Show(
                    "Не удалось открыть изображение.",
                    "Ошибка",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Error
                );

                return;
            }

            StartNewGame(sourceImage);
        }

        private void StartNewGame(Bitmap sourceImage)
        {
            ClearOldGame();

            movesCount = 0;
            gameStarted = true;
            firstSelected = null;

            lblMoves.Text = "Ходы: 0";
            lblStatus.Text = "Игра началась. Выберите два фрагмента для обмена.";

            Bitmap preparedImage = PrepareImage(sourceImage, panelGame.Width, panelGame.Height);

            pictureOriginal.Image = new Bitmap(preparedImage);

            pieceWidth = preparedImage.Width / gridSize;
            pieceHeight = preparedImage.Height / gridSize;

            puzzleBoxes = new PictureBox[gridSize, gridSize];
            originalPieces = new Bitmap[gridSize * gridSize];

            CreatePuzzlePieces(preparedImage);
            ShufflePuzzle();

            sourceImage.Dispose();
        }

        private Bitmap PrepareImage(Bitmap sourceImage, int targetWidth, int targetHeight)
        {
            Bitmap result = new Bitmap(targetWidth, targetHeight);

            using (Graphics graphics = Graphics.FromImage(result))
            {
                graphics.DrawImage(sourceImage, 0, 0, targetWidth, targetHeight);
            }

            return result;
        }

        private void CreatePuzzlePieces(Bitmap image)
        {
            int index = 0;

            for (int row = 0; row < gridSize; row++)
            {
                for (int col = 0; col < gridSize; col++)
                {
                    Rectangle pieceRectangle = new Rectangle(
                        col * pieceWidth,
                        row * pieceHeight,
                        pieceWidth,
                        pieceHeight
                    );

                    Bitmap piece = image.Clone(
                        pieceRectangle,
                        image.PixelFormat
                    );

                    originalPieces[index] = new Bitmap(piece);

                    PictureBox pictureBox = new PictureBox();
                    pictureBox.Left = col * pieceWidth;
                    pictureBox.Top = row * pieceHeight;
                    pictureBox.Width = pieceWidth;
                    pictureBox.Height = pieceHeight;
                    pictureBox.BorderStyle = BorderStyle.FixedSingle;
                    pictureBox.SizeMode = PictureBoxSizeMode.StretchImage;
                    pictureBox.Image = piece;
                    pictureBox.Tag = index;
                    pictureBox.Click += PuzzlePiece_Click;

                    puzzleBoxes[row, col] = pictureBox;
                    panelGame.Controls.Add(pictureBox);

                    index++;
                }
            }
        }

        private void ShufflePuzzle()
        {
            int totalPieces = gridSize * gridSize;

            for (int i = 0; i < totalPieces * 10; i++)
            {
                int firstIndex = random.Next(totalPieces);
                int secondIndex = random.Next(totalPieces);

                int firstRow = firstIndex / gridSize;
                int firstCol = firstIndex % gridSize;

                int secondRow = secondIndex / gridSize;
                int secondCol = secondIndex % gridSize;

                SwapPieces(
                    puzzleBoxes[firstRow, firstCol],
                    puzzleBoxes[secondRow, secondCol]
                );
            }

            if (CheckWin())
            {
                ShufflePuzzle();
            }
        }

        private void PuzzlePiece_Click(object sender, EventArgs e)
        {
            if (!gameStarted)
            {
                return;
            }

            PictureBox clickedBox = sender as PictureBox;

            if (clickedBox == null)
            {
                return;
            }

            if (firstSelected == null)
            {
                firstSelected = clickedBox;
                firstSelected.BorderStyle = BorderStyle.Fixed3D;
                lblStatus.Text = "Первый фрагмент выбран. Теперь выберите второй.";
                return;
            }

            if (firstSelected == clickedBox)
            {
                firstSelected.BorderStyle = BorderStyle.FixedSingle;
                firstSelected = null;
                lblStatus.Text = "Выбор отменён.";
                return;
            }

            SwapPieces(firstSelected, clickedBox);

            firstSelected.BorderStyle = BorderStyle.FixedSingle;
            firstSelected = null;

            movesCount++;
            lblMoves.Text = "Ходы: " + movesCount;

            if (CheckWin())
            {
                gameStarted = false;
                DisablePuzzleClicks();

                lblStatus.Text = "Победа! Картинка собрана.";

                MessageBox.Show(
                    "Поздравляем! Вы собрали картинку за " + movesCount + " ходов!",
                    "Победа",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information
                );
            }
            else
            {
                lblStatus.Text = "Фрагменты поменялись местами.";
            }
        }

        private void SwapPieces(PictureBox first, PictureBox second)
        {
            Image tempImage = first.Image;
            object tempTag = first.Tag;

            first.Image = second.Image;
            first.Tag = second.Tag;

            second.Image = tempImage;
            second.Tag = tempTag;
        }

        private bool CheckWin()
        {
            if (puzzleBoxes == null)
            {
                return false;
            }

            for (int row = 0; row < gridSize; row++)
            {
                for (int col = 0; col < gridSize; col++)
                {
                    int expectedIndex = row * gridSize + col;
                    int currentIndex = Convert.ToInt32(puzzleBoxes[row, col].Tag);

                    if (currentIndex != expectedIndex)
                    {
                        return false;
                    }
                }
            }

            return true;
        }

        private void BtnReset_Click(object sender, EventArgs e)
        {
            if (puzzleBoxes == null || originalPieces == null)
            {
                MessageBox.Show(
                    "Сначала начните новую игру.",
                    "Информация",
                    MessageBoxButtons.OK,
                    MessageBoxIcon.Information
                );

                return;
            }

            int index = 0;

            for (int row = 0; row < gridSize; row++)
            {
                for (int col = 0; col < gridSize; col++)
                {
                    if (puzzleBoxes[row, col].Image != null)
                    {
                        puzzleBoxes[row, col].Image.Dispose();
                    }

                    puzzleBoxes[row, col].Image = new Bitmap(originalPieces[index]);
                    puzzleBoxes[row, col].Tag = index;
                    puzzleBoxes[row, col].BorderStyle = BorderStyle.FixedSingle;
                    puzzleBoxes[row, col].Enabled = true;

                    index++;
                }
            }

            movesCount = 0;
            firstSelected = null;
            gameStarted = true;

            lblMoves.Text = "Ходы: 0";
            lblStatus.Text = "Пазл сброшен в правильное положение.";
        }

        private void DisablePuzzleClicks()
        {
            for (int row = 0; row < gridSize; row++)
            {
                for (int col = 0; col < gridSize; col++)
                {
                    puzzleBoxes[row, col].Enabled = false;
                }
            }
        }

        private void ClearOldGame()
        {
            foreach (Control control in panelGame.Controls)
            {
                PictureBox pictureBox = control as PictureBox;

                if (pictureBox != null)
                {
                    if (pictureBox.Image != null)
                    {
                        pictureBox.Image.Dispose();
                    }

                    pictureBox.Dispose();
                }
            }

            panelGame.Controls.Clear();

            if (pictureOriginal.Image != null)
            {
                pictureOriginal.Image.Dispose();
                pictureOriginal.Image = null;
            }

            if (originalPieces != null)
            {
                for (int i = 0; i < originalPieces.Length; i++)
                {
                    if (originalPieces[i] != null)
                    {
                        originalPieces[i].Dispose();
                    }
                }
            }
        }
    }
}
```

---

# 5. Важный момент про `namespace`

Вверху кода есть строка:

```csharp
namespace PuzzleGame
```

Если твой проект называется **PuzzleGame**, ничего менять не надо.

Если проект называется, например:

```text
WindowsFormsApp1
```

то замени строку:

```csharp
namespace PuzzleGame
```

на:

```csharp
namespace WindowsFormsApp1
```

Название должно совпадать с тем, что было в твоём исходном `Form1.cs`.

---

# 6. Если снова будет ошибка `Form1_Load`

В этом коде метод уже есть:

```csharp
private void Form1_Load(object sender, EventArgs e)
{
}
```

Поэтому ошибка должна исчезнуть.

---

# 7. Как запустить

Сохрани файл:

```text
Ctrl + S
```

Потом нажми:

```text
F5
```

или:

```text
Сборка → Собрать решение
```

---

# 8. Как проверить игру

После запуска:

1. Нажми **Новая игра**.
2. Выбери любую картинку `.jpg`, `.png` или `.bmp`.
3. Картинка появится слева как образец.
4. Справа появится поле 4×4.
5. Кликни на один фрагмент.
6. Кликни на другой фрагмент.
7. Они поменяются местами.
8. Счётчик ходов увеличится.
9. Когда все фрагменты будут на местах, появится сообщение о победе.

---

# 9. Что в этом коде реализовано по заданию

Реализовано:

```text
динамическое создание PictureBox
двумерный массив PictureBox[,]
загрузка изображения через OpenFileDialog
нарезка изображения через Bitmap.Clone()
перемешивание фрагментов
обмен фрагментов в два клика
счётчик ходов
проверка победы через Tag
кнопка «Новая игра»
кнопка «Сброс»
отображение исходного изображения
```

То есть для сдачи лабораторной этого достаточно.
