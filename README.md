# NAROODO-
THE BRELIANT GAME IN THE WORLD
## Review

Your `CarDodgerGame` code is solid and should run correctly as a WinForms game. The only issue in the snippet you pasted is that the entire file appears duplicated.

## Fix

Use a single copy of the file and remove the duplicated duplicate block at the end.

## Clean version

```csharp
using System;
using System.Collections.Generic;
using System.Drawing;
using System.Windows.Forms;

namespace CarDodgerGame
{
    internal static class Program
    {
        [STAThread]
        private static void Main()
        {
            Application.EnableVisualStyles();
            Application.SetCompatibleTextRenderingDefault(false);
            Application.Run(new GameForm());
        }
    }

    public sealed class GameForm : Form
    {
        private readonly Timer gameTimer = new Timer();
        private readonly Random random = new Random();
        private readonly HashSet<Keys> pressedKeys = new HashSet<Keys>();

        private Rectangle playerCar;
        private readonly List<Rectangle> enemyCars = new List<Rectangle>();
        private readonly List<int> laneCenters = new List<int>();

        private int roadLeft;
        private int roadWidth;
        private int score;
        private int highScore;
        private int speed;
        private int spawnCounter;
        private int roadLineOffset;
        private bool isGameOver;

        private const int PlayerWidth = 46;
        private const int PlayerHeight = 82;
        private const int EnemyWidth = 46;
        private const int EnemyHeight = 82;
        private const int FrameRate = 16;

        public GameForm()
        {
            Text = "C# Car Dodger Game";
            ClientSize = new Size(520, 720);
            StartPosition = FormStartPosition.CenterScreen;
            FormBorderStyle = FormBorderStyle.FixedSingle;
            MaximizeBox = false;
            DoubleBuffered = true;
            BackColor = Color.FromArgb(22, 24, 28);
            KeyPreview = true;

            KeyDown += OnKeyDown;
            KeyUp += OnKeyUp;
            Paint += OnPaint;

            gameTimer.Interval = FrameRate;
            gameTimer.Tick += OnGameTick;

            ResetGame();
            gameTimer.Start();
        }

        private void ResetGame()
        {
            roadWidth = 330;
            roadLeft = (ClientSize.Width - roadWidth) / 2;

            laneCenters.Clear();
            laneCenters.Add(roadLeft + roadWidth / 6);
            laneCenters.Add(roadLeft + roadWidth / 2);
            laneCenters.Add(roadLeft + roadWidth * 5 / 6);

            playerCar = new Rectangle(
                laneCenters[1] - PlayerWidth / 2,
                ClientSize.Height - PlayerHeight - 36,
                PlayerWidth,
                PlayerHeight
            );

            enemyCars.Clear();
            score = 0;
            speed = 7;
            spawnCounter = 0;
            roadLineOffset = 0;
            isGameOver = false;
            pressedKeys.Clear();
        }

        private void OnGameTick(object? sender, EventArgs e)
        {
            if (isGameOver)
            {
                Invalidate();
                return;
            }

            MovePlayer();
            MoveRoad();
            SpawnEnemies();
            MoveEnemies();
            CheckCollision();
            IncreaseDifficulty();

            Invalidate();
        }

        private void MovePlayer()
        {
            int moveSpeed = 8;

            if (pressedKeys.Contains(Keys.Left) || pressedKeys.Contains(Keys.A))
                playerCar.X -= moveSpeed;

            if (pressedKeys.Contains(Keys.Right) || pressedKeys.Contains(Keys.D))
                playerCar.X += moveSpeed;

            if (pressedKeys.Contains(Keys.Up) || pressedKeys.Contains(Keys.W))
                playerCar.Y -= moveSpeed;

            if (pressedKeys.Contains(Keys.Down) || pressedKeys.Contains(Keys.S))
                playerCar.Y += moveSpeed;

            int minX = roadLeft + 10;
            int maxX = roadLeft + roadWidth - playerCar.Width - 10;
            int minY = 20;
            int maxY = ClientSize.Height - playerCar.Height - 20;

            playerCar.X = Math.Max(minX, Math.Min(maxX, playerCar.X));
            playerCar.Y = Math.Max(minY, Math.Min(maxY, playerCar.Y));
        }

        private void MoveRoad()
        {
            roadLineOffset += speed;
            if (roadLineOffset >= 50)
                roadLineOffset = 0;
        }

        private void SpawnEnemies()
        {
            spawnCounter++;
            int spawnRate = Math.Max(22, 56 - score / 12);

            if (spawnCounter < spawnRate)
                return;

            spawnCounter = 0;

            int lane = random.Next(laneCenters.Count);
            int x = laneCenters[lane] - EnemyWidth / 2;
            int y = -EnemyHeight - random.Next(20, 120);

            Rectangle newEnemy = new Rectangle(x, y, EnemyWidth, EnemyHeight);

            foreach (Rectangle enemy in enemyCars)
            {
                if (Math.Abs(enemy.X - newEnemy.X) < 4 && Math.Abs(enemy.Y - newEnemy.Y) < 140)
                    return;
            }

            enemyCars.Add(newEnemy);
        }

        private void MoveEnemies()
        {
            for (int i = enemyCars.Count - 1; i >= 0; i--)
            {
                Rectangle enemy = enemyCars[i];
                enemy.Y += speed;
                enemyCars[i] = enemy;

                if (enemy.Top > ClientSize.Height)
                {
                    enemyCars.RemoveAt(i);
                    score += 5;
                    highScore = Math.Max(highScore, score);
                }
            }
        }

        private void CheckCollision()
        {
            Rectangle playerHitBox = Rectangle.Inflate(playerCar, -7, -7);

            foreach (Rectangle enemy in enemyCars)
            {
                Rectangle enemyHitBox = Rectangle.Inflate(enemy, -7, -7);
                if (playerHitBox.IntersectsWith(enemyHitBox))
                {
                    isGameOver = true;
                    highScore = Math.Max(highScore, score);
                    return;
                }
            }
        }

        private void IncreaseDifficulty()
        {
            speed = 7 + score / 80;
            if (speed > 18)
                speed = 18;
        }

        private void OnKeyDown(object? sender, KeyEventArgs e)
        {
            pressedKeys.Add(e.KeyCode);

            if (isGameOver && e.KeyCode == Keys.Enter)
                ResetGame();

            if (e.KeyCode == Keys.Escape)
                Close();
        }

        private void OnKeyUp(object? sender, KeyEventArgs e)
        {
            pressedKeys.Remove(e.KeyCode);
        }

        private void OnPaint(object? sender, PaintEventArgs e)
        {
            Graphics g = e.Graphics;
            g.SmoothingMode = System.Drawing.Drawing2D.SmoothingMode.AntiAlias;

            DrawBackground(g);
            DrawRoad(g);
            DrawHud(g);
            DrawCar(g, playerCar, Color.FromArgb(42, 141, 255), Color.White);

            foreach (Rectangle enemy in enemyCars)
                DrawCar(g, enemy, Color.FromArgb(230, 58, 58), Color.Gold);

            if (isGameOver)
                DrawGameOver(g);
        }

        private void DrawBackground(Graphics g)
        {
            using Brush grass = new SolidBrush(Color.FromArgb(28, 84, 44));
            g.FillRectangle(grass, ClientRectangle);

            using Pen sideMark = new Pen(Color.FromArgb(45, 120, 64), 4);
            for (int y = 0; y < ClientSize.Height; y += 28)
            {
                g.DrawLine(sideMark, roadLeft - 34, y, roadLeft - 16, y + 16);
                g.DrawLine(sideMark, roadLeft + roadWidth + 16, y + 16, roadLeft + roadWidth + 34, y);
            }
        }

        private void DrawRoad(Graphics g)
        {
            using Brush road = new SolidBrush(Color.FromArgb(49, 52, 57));
            using Pen border = new Pen(Color.White, 5);
            using Pen yellow = new Pen(Color.Gold, 4);
            using Pen lane = new Pen(Color.White, 4);

            g.FillRectangle(road, roadLeft, 0, roadWidth, ClientSize.Height);
            g.DrawLine(border, roadLeft, 0, roadLeft, ClientSize.Height);
            g.DrawLine(border, roadLeft + roadWidth, 0, roadLeft + roadWidth, ClientSize.Height);

            int laneOne = roadLeft + roadWidth / 3;
            int laneTwo = roadLeft + roadWidth * 2 / 3;

            for (int y = -50 + roadLineOffset; y < ClientSize.Height; y += 70)
            {
                g.DrawLine(lane, laneOne, y, laneOne, y + 38);
                g.DrawLine(lane, laneTwo, y, laneTwo, y + 38);
            }

            g.DrawLine(yellow, roadLeft + roadWidth / 2, 0, roadLeft + roadWidth / 2, ClientSize.Height);
        }

        private void DrawHud(Graphics g)
        {
            using Font titleFont = new Font("Segoe UI", 14, FontStyle.Bold);
            using Font smallFont = new Font("Segoe UI", 10, FontStyle.Regular);
            using Brush textBrush = new SolidBrush(Color.White);
            using Brush panelBrush = new SolidBrush(Color.FromArgb(160, 0, 0, 0));

            Rectangle panel = new Rectangle(12, 12, 170, 88);
            g.FillRoundedRectangle(panelBrush, panel, 12);

            g.DrawString($"Score: {score}", titleFont, textBrush, 24, 22);
            g.DrawString($"High Score: {highScore}", smallFont, textBrush, 26, 54);
            g.DrawString("Move: WASD / Arrows", smallFont, textBrush, 26, 76);
        }

        private void DrawCar(Graphics g, Rectangle car, Color bodyColor, Color windowColor)
        {
            using Brush body = new SolidBrush(bodyColor);
            using Brush dark = new SolidBrush(Color.FromArgb(25, 25, 25));
            using Brush window = new SolidBrush(windowColor);
            using Pen outline = new Pen(Color.FromArgb(15, 15, 15), 2);

            Rectangle bodyRect = new Rectangle(car.X + 4, car.Y + 4, car.Width - 8, car.Height - 8);
            g.FillRoundedRectangle(body, bodyRect, 12);
            g.DrawRoundedRectangle(outline, bodyRect, 12);

            Rectangle frontWindow = new Rectangle(car.X + 13, car.Y + 12, car.Width - 26, 18);
            Rectangle rearWindow = new Rectangle(car.X + 13, car.Y + car.Height - 32, car.Width - 26, 18);
            g.FillRoundedRectangle(window, frontWindow, 6);
            g.FillRoundedRectangle(window, rearWindow, 6);

            g.FillRectangle(dark, car.X, car.Y + 15, 8, 18);
            g.FillRectangle(dark, car.Right - 8, car.Y + 15, 8, 18);
            g.FillRectangle(dark, car.X, car.Bottom - 34, 8, 18);
            g.FillRectangle(dark, car.Right - 8, car.Bottom - 34, 8, 18);
        }

        private void DrawGameOver(Graphics g)
        {
            using Brush overlay = new SolidBrush(Color.FromArgb(195, 0, 0, 0));
            using Brush text = new SolidBrush(Color.White);
            using Brush accent = new SolidBrush(Color.Gold);
            using Font big = new Font("Segoe UI", 34, FontStyle.Bold);
            using Font medium = new Font("Segoe UI", 16, FontStyle.Bold);
            using Font small = new Font("Segoe UI", 12, FontStyle.Regular);

            g.FillRectangle(overlay, ClientRectangle);

            string gameOver = "GAME OVER";
            string scoreText = $"Final Score: {score}";
            string restart = "Press ENTER to restart";
            string quit = "Press ESC to quit";

            DrawCenteredText(g, gameOver, big, accent, ClientSize.Height / 2 - 95);
            DrawCenteredText(g, scoreText, medium, text, ClientSize.Height / 2 - 24);
            DrawCenteredText(g, restart, small, text, ClientSize.Height / 2 + 24);
            DrawCenteredText(g, quit, small, text, ClientSize.Height / 2 + 52);
        }

        private void DrawCenteredText(Graphics g, string value, Font font, Brush brush, int y)
        {
            SizeF size = g.MeasureString(value, font);
            float x = (ClientSize.Width - size.Width) / 2f;
            g.DrawString(value, font, brush, x, y);
        }
    }

    public static class GraphicsExtensions
    {
        public static void FillRoundedRectangle(this Graphics graphics, Brush brush, Rectangle bounds, int radius)
        {
            using var path = CreateRoundedRectanglePath(bounds, radius);
            graphics.FillPath(brush, path);
        }

        public static void DrawRoundedRectangle(this Graphics graphics, Pen pen, Rectangle bounds, int radius)
        {
            using var path = CreateRoundedRectanglePath(bounds, radius);
            graphics.DrawPath(pen, path);
        }

        private static System.Drawing.Drawing2D.GraphicsPath CreateRoundedRectanglePath(Rectangle bounds, int radius)
        {
            int diameter = radius * 2;
            var path = new System.Drawing.Drawing2D.GraphicsPath();

            path.AddArc(bounds.Left, bounds.Top, diameter, diameter, 180, 90);
            path.AddArc(bounds.Right - diameter, bounds.Top, diameter, diameter, 270, 90);
            path.AddArc(bounds.Right - diameter, bounds.Bottom - diameter, diameter, diameter, 0, 90);
            path.AddArc(bounds.Left, bounds.Bottom - diameter, diameter, diameter, 90, 90);
            path.CloseFigure();

            return path;
        }
    }
}
```

If you want, I can also help refactor this into separate files like `Program.cs`, `GameForm.cs`, and `GraphicsExtensions.cs`.
