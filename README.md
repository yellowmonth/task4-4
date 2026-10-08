# Практическая работа №4: Циклические конструкции в C# (while, do-while, for) и разработка консольной RPG

**Выполнили:** Бандурин Дмитрий , Башкатова Ксения 

---




```csharp
using System;


class Program
{
    static void Main()
    {
        // НАСТРОЙКИ СЕТТИНГА
        Console.Title = "Neon RetroWave: Уличный гонщик";
        Console.OutputEncoding = System.Text.Encoding.UTF8;

        // ПАРАМЕТРЫ ГЕРОЯ
        string heroName = "Уличный гонщик";
        int maxHp = 120;              // Запас прочности авто
        int hp = maxHp;
        int nitroMax = 5;             // Баллоны закиси азота
        int nitro = nitroMax;
        int repairKits = 3;           // Ремкомплекты (восстановление ресурса)

        //  ПАРАМЕТРЫ ПРОТИВНИКОВ (3 ВОЛНЫ) 
        string[] enemyNames = { "Патрульный байк", "Броневик синдиката", "Тюремный автобус" };
        int[] enemyMaxHp = { 60, 100, 160 };
        int[] enemyMinDmg = { 6, 10, 14 };
        int[] enemyMaxDmg = { 12, 18, 26 };

        Random rnd = new Random();

        // ВСТУПЛЕНИЕ 
        Console.ForegroundColor = ConsoleColor.Magenta;
        Console.WriteLine("||        NEON RETROWAVE :: УЛИЧНЫЙ ГОНЩИК                  ||");
        Console.WriteLine("||        Ночная трасса. Синдикат объявил охоту.            ||");
        Console.ResetColor();
        Console.WriteLine();
        Console.WriteLine("Нажмите ENTER, чтобы завести двигатель...");
        Console.ReadLine();

        bool gameOver = false;
        bool playerWon = false;

        // ГЛАВНЫЙ ИГРОВОЙ ЦИКЛ (while) — волны через for 
        for (int wave = 0; wave < enemyNames.Length && !gameOver; wave++)
        {
            // Параметры текущего противника
            string enemyName = enemyNames[wave];
            int enemyHp = enemyMaxHp[wave];
            int enemyHpMax = enemyMaxHp[wave];
            int eMin = enemyMinDmg[wave];
            int eMax = enemyMaxDmg[wave];
            int waveNumber = wave + 1;

            // Уведомление о новой волне
            Console.WriteLine();
            Console.ForegroundColor = ConsoleColor.Cyan;
            Console.WriteLine("══════════════════════════════════════════════════════════");
            Console.WriteLine($"   ВОЛНА {waveNumber} / {enemyNames.Length}  ::  {enemyName.ToUpper()}");
            Console.WriteLine("══════════════════════════════════════════════════════════");
            Console.ResetColor();
            Console.WriteLine("Нажмите ENTER для начала боя...");
            Console.ReadLine();

            int turn = 1;
            bool defenseStance = false;   // Активна ли защитная стойка
            int waveDamageDealt = 0;
            int waveTurns = 0;

            // ПОШАГОВЫЙ ЦИКЛ БОЯ (while) 
            while (hp > 0 && enemyHp > 0)
            {
                Console.WriteLine();
                Console.ForegroundColor = ConsoleColor.DarkYellow;
                Console.WriteLine($"--- ХОД {turn} ---");
                Console.ResetColor();

                // Визуализация шкал (for) 
                DrawBar("Гонщик", hp, maxHp, ConsoleColor.Green);
                DrawBar("Закись N2O", nitro, nitroMax, ConsoleColor.Blue);
                DrawBar(enemyName, enemyHp, enemyHpMax, ConsoleColor.Red);
                Console.WriteLine($"Ремкомплектов: {repairKits}");
                if (defenseStance)
                {
                    Console.ForegroundColor = ConsoleColor.Cyan;
                    Console.WriteLine("[АКТИВНО] Защитная стойка: получаемый урон снижен вдвое");
                    Console.ResetColor();
                }

                // Меню выбора действия (do-while + switch)
                int action;
                bool isValid;
                do
                {
                    Console.WriteLine();
                    Console.WriteLine("Выберите действие:");
                    Console.WriteLine("  1 — Базовая атака (стабильный урон)");
                    Console.WriteLine("  2 — Специальный приём (усиленный урон, тратит закись)");
                    Console.WriteLine("  3 — Защитная стойка (урон вдвое меньше)");
                    Console.WriteLine("  4 — Ремонт (восстановление прочности)");
                    Console.Write("Ваш выбор: ");

                    isValid = int.TryParse(Console.ReadLine(), out action) && action >= 1 && action <= 4;

                    if (!isValid)
                    {
                        Console.ForegroundColor = ConsoleColor.Red;
                        Console.WriteLine("Ошибка: введите число от 1 до 4!\n");
                        Console.ResetColor();
                    }
                } while (!isValid);

                // Сбрасываем защиту на начало хода игрока (действует только на ход противника)
                defenseStance = false;

                int damageToEnemy = 0;
                string actionLog = "";

                // Обработка выбора игрока (switch) 
                switch (action)
                {
                    case 1:
                        // Базовая атака — стабильный урон
                        damageToEnemy = rnd.Next(10, 16);
                        actionLog = $"Гонщик таранит противника! Урон: {damageToEnemy}";
                        break;

                    case 2:
                        // Специальный приём — усиленный урон за закись
                        if (nitro > 0)
                        {
                            nitro--;
                            damageToEnemy = rnd.Next(22, 34);
                            actionLog = $"N2O-РЫВОК! Пробивающий удар. Урон: {damageToEnemy} (осталось баллонов: {nitro})";
                        }
                        else
                        {
                            // Закиси нет — переходим к базовой атаке, не тратя ход впустую
                            Console.ForegroundColor = ConsoleColor.Red;
                            Console.WriteLine("Закись азота закончилась! Выполняется базовая атака.");
                            Console.ResetColor();
                            damageToEnemy = rnd.Next(8, 13);
                            actionLog = $"Вынужденная атака. Урон: {damageToEnemy}";
                        }
                        break;

                    case 3:
                        // Защитная стойка
                        defenseStance = true;
                        actionLog = "Гонщик уходит в защитную стойку (получаемый урон снижен вдвое).";
                        break;

                    case 4:
                        // Ремонт — восстановление прочности
                        if (repairKits > 0)
                        {
                            repairKits--;
                            int repair = rnd.Next(18, 28);
                            hp += repair;
                            if (hp > maxHp) hp = maxHp;
                            actionLog = $"Ремонт корпуса: +{repair} прочности (осталось ремкомплектов: {repairKits})";
                        }
                        else
                        {
                            Console.ForegroundColor = ConsoleColor.Red;
                            Console.WriteLine("Ремкомплектов не осталось! Ход потрачен на попытку ремонта.");
                            Console.ResetColor();
                            actionLog = "Попытка ремонта провалилась — запчастей нет.";
                        }
                        break;
                }

                //  Применяем урон противнику 
                if (damageToEnemy > 0)
                {
                    enemyHp -= damageToEnemy;
                    waveDamageDealt += damageToEnemy;
                    if (enemyHp < 0) enemyHp = 0;
                }

                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine(actionLog);
                Console.ResetColor();

                // Проверка победы над противником 
                if (enemyHp <= 0)
                {
                    Console.ForegroundColor = ConsoleColor.Green;
                    Console.WriteLine();
                    Console.WriteLine($"☠ {enemyName} УНИЧТОЖЕН!");
                    Console.ResetColor();
                    break;
                }

                // ---- Ход противника ----
                int enemyDamage = rnd.Next(eMin, eMax + 1);

                if (defenseStance)
                {
                    enemyDamage = enemyDamage / 2;
                    if (enemyDamage < 1) enemyDamage = 1;
                }

                hp -= enemyDamage;
                if (hp < 0) hp = 0;

                Console.ForegroundColor = ConsoleColor.Red;
                Console.WriteLine($"{enemyName} атакует! Урон: {enemyDamage}");
                Console.ResetColor();

                // Небольшой шанс крит-удара у сильных противников (для разнообразия)
                if (waveNumber >= 2 && rnd.Next(0, 100) < 15)
                {
                    int crit = rnd.Next(4, 10);
                    hp -= crit;
                    if (hp < 0) hp = 0;
                    Console.ForegroundColor = ConsoleColor.DarkRed;
                    Console.WriteLine($"!!! КРИТИЧЕСКИЙ УДАР {enemyName}: дополнительный урон {crit} !!!");
                    Console.ResetColor();
                }

                // Проверка поражения 
                if (hp <= 0)
                {
                    Console.ForegroundColor = ConsoleColor.Red;
                    Console.WriteLine();
                    Console.WriteLine("╔══════════════════════════════════════════════════════════╗");
                    Console.WriteLine("║              АВТОМОБИЛЬ УНИЧТОЖЕН. GAME OVER.            ║");
                    Console.WriteLine("╚══════════════════════════════════════════════════════════╝");
                    Console.ResetColor();
                    gameOver = true;
                    break;
                }

                waveTurns++;
                turn++;
            }

            // Итоги волны 
            if (!gameOver && enemyHp <= 0)
            {
                Console.ForegroundColor = ConsoleColor.Green;
                Console.WriteLine();
                Console.WriteLine($"Волна {waveNumber} пройдена! Ходов: {turn - 1}, нанесено урона: {waveDamageDealt}");
                Console.ResetColor();

                // Бонус за прохождение волны — восстановление закиси
                if (nitro < nitroMax)
                {
                    nitro++;
                    Console.ForegroundColor = ConsoleColor.Blue;
                    Console.WriteLine("Бонус за победу: +1 баллон закиси азота.");
                    Console.ResetColor();
                }

                // Небольшой ремонт между волнами
                int bonusRepair = 10;
                hp += bonusRepair;
                if (hp > maxHp) hp = maxHp;
                Console.WriteLine($"Экстренный ремонт между волнами: +{bonusRepair} прочности.");

                Console.WriteLine("Нажмите ENTER, чтобы продолжить...");
                Console.ReadLine();
            }
        }

        // ФИНАЛ 
        Console.WriteLine();
        if (!gameOver)
        {
            playerWon = true;
            Console.ForegroundColor = ConsoleColor.Magenta;
            Console.WriteLine("||                 ПОБЕДА! ТРАССА СВОБОДНА!               ||");
            Console.WriteLine("||      Уличный гонщик пересёк финишную черту и скрылся.    ||");
            Console.ResetColor();
            Console.WriteLine($"Итоговое состояние: {hp}/{maxHp} прочности, закиси: {nitro}/{nitroMax}, ремкомплектов: {repairKits}");
        }
        else
        {
            Console.ForegroundColor = ConsoleColor.Red;
            Console.WriteLine("Синдикат захватил трассу. Поражение.");
            Console.ResetColor();
        }

        Console.WriteLine();
        Console.WriteLine("Нажмите ENTER для выхода...");
        Console.ReadLine();
    }

    //  ВСПОМОГАТЕЛЬНЫЙ МЕТОД: ШКАЛА (for) 
    static void DrawBar(string label, int current, int max, ConsoleColor color)
    {
        if (max <= 0) max = 1;
        if (current < 0) current = 0;
        if (current > max) current = max;

        int totalWidth = 20;
        int filled = (int)Math.Round((double)current / max * totalWidth);
        if (filled < 0) filled = 0;
        if (filled > totalWidth) filled = totalWidth;

        Console.Write($"{label,-14}: [");
        Console.ForegroundColor = color;
        for (int i = 0; i < filled; i++) Console.Write("#");
        Console.ResetColor();
        for (int i = 0; i < totalWidth - filled; i++) Console.Write("-");
        Console.WriteLine($"] {current}/{max}");
    }
}
```
