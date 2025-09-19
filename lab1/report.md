# Отчет по лабораторной работе №1
## Задание
Нужно разработать консольное приложение, которое будет эмулировать вендинговый автомат, позволяющее пользователю:
1. Посмотреть список доступных товаров с их ценами и количеством.
2. Вставить монеты разных номиналов.
3. Выбрать товар и получить его, если внесённой суммы достаточно.
4. Получить сдачу (если нужно) и вернуть неиспользованные монеты при отмене операции.
5. Администраторский режим для пополнения ассортимента и сбора собранных средств.

## Решение
### Шаг №1
ОС: Linux Ubuntu  
Создаю репозиторий проекта, прописываю команду:
```
dotnet new console -n lab1
```
dotnet - инструмент для работы с C#  
new - команда, чтобы создать новый проект  
console - тип проекта, в нашем случае - консольное приложение  
-n - имя  
lab1 - имя проекта  
  
Теперь структура выглядит так:   
Program.cs - основной файл с кодом  
lab1.csproj - Project, файл с описанием структуры проекта и настройками  
lab1.sln - Solution, дерево проектов  

### Шаг №2 
Работа в Visual Studio Code.  
Листинги кода с комментариями:  
```
namespace Lab1VenMachine
```
1. Определение класса Product  
Класс будет хранить информацию о каждом товаре.  
Поля: название, цена, количество.  
```
// Класс "Товар": хранит название, цену и количество
    public class Product
    {
        public string Name; // Название товара
        public int Price; // Цена товара, тип int, ибо цены в таких автоматах небольшие
        public int Quantity; // Количество в наличии

        public Product(string name, int price, int quantity)
        {
            Name = name;
            Price = price;
            Quantity = quantity;
        }
    }
```
2. Определение класса Inventory  
Класс будет управлять списком товаров и выручкой.  
Поля: список товаров, выручка.  
Методы: показать список товаров DisplayProducts(), получить товар по номеру (принять заказ от покупателя) GetProduct(), пополнить запас товаров (в админ-режиме) Restock(), собрать выручку админом CollectMoney(), добавить деньги за товары в выручку AddToCollected().  
```
// Класс "Инвентарь": управляет товарами и выручкой
    public class Inventory
    {
        private List<Product> products; // Список товаров
        private int totalCollected; // выручка

        public Inventory()
        {
            // Создаю объекты (товары) с начальными значениями
            products = new List<Product>
            {
                new Product("Кола", 150, 7),
                new Product("Чипсы", 180, 10),
                new Product("Шоколад", 100, 8),
                new Product("Вода", 70, 15)
            };
            totalCollected = 0;
        }

        // Метод: Показать список товаров
        public void DisplayProducts()
        {
            Console.WriteLine("Доступные товары:");
            for (int i = 0; i < products.Count; i++) // Каждому товару соответствует номер (для принятия заказа)
            {
                var product = products[i];
                Console.WriteLine($"{i + 1}. {product.Name} - Цена: {product.Price} руб. - Количество: {product.Quantity}");
            }
        }

        // Метод: Получить товар по номеру (нумерация начинается с 1)
        public Product GetProduct(int productNumber)
        {
            if (productNumber < 1 || productNumber > products.Count) // Проверка
            {
                Console.WriteLine("Ошибка: неверный номер товара");
                return null;
            }
            return products[productNumber - 1];
        }

        // Метод: Пополнить запас товаров
        public void Restock()
        {
            foreach (var product in products) // Проходится по каждому товару для пополнения
            {
                Console.WriteLine($"Текущий запас {product.Name}: {product.Quantity}. Сколько добавить?");
                if (int.TryParse(Console.ReadLine(), out int add))
                {
                    product.Quantity += add;
                }
            }
            Console.WriteLine("Товары пополнены.");
        }

        // Метод: Собрать выручку (для админ-мода)
        public int CollectMoney()
        {
            int collected = totalCollected;
            totalCollected = 0;
            return collected;
        }

        // Метод: Добавляет деньги в выручку, когда покупатель берет товар
        public void AddToCollected(int amount)
        {
            totalCollected += amount;
        }
    }
```
3.  Определение класса Admin.  
Класс будет управлять админ-режимом.  
Поля: пароль для админа, товары.   
Методы: вход в админ-режим AdminMode().  
```
// Класс "Админ": управление админ-режимом
    public class Admin
    {
        private string Password = "12345"; // Пароль для админа
        private Inventory Inventory; // Товары

        public Admin(Inventory inventory)
        {
            Inventory = inventory;
        }

        // Метод: Вход в админ-режим
        public void AdminMode()
        {
            Console.WriteLine("Введите пароль:"); // админ должен ввести свой пароль
            string psswrd = Console.ReadLine();
            if (psswrd != Password)
            {
                Console.WriteLine("Неверный пароль");
                return;
            }
            bool inAdminMode = true;
            while (inAdminMode) // админ-режим будем действовать, пока сам админ из него не выйдет
            {
                Console.WriteLine("Админ-режим:");
                Console.WriteLine("1. Пополнить товары");
                Console.WriteLine("2. Собрать средства");
                Console.WriteLine("3. Выйти из Админ-режима");
                string choice = Console.ReadLine();
                if (choice == "1")
                {
                    Inventory.Restock();
                }
                else if (choice == "2")
                {
                    int collected = Inventory.CollectMoney();
                    Console.WriteLine($"Выручка: {collected} руб.");
                    Console.WriteLine("Средства собраны.");
                }
                else if (choice == "3")
                {
                    Console.WriteLine("Выход из админ-режима.");
                    inAdminMode = false;
                }
                else
                {
                    Console.WriteLine("Неверный выбор.");
                }
            }
        }
    }  
```
4. Определение класса Вендинговый автомат.
Класс будет управлять работой автомата.  
Поля: товары, админ, текущий баланс.  
Методы: пополнить баланс InsertMoney(), показать товары и текущий баланс DisplayProductsAndBalance(), купить товар BuyProduct(), отмена и возврат средств Cancel(),вход в админ-режим AdminMode().  
```
// Класс "Админ": управление админ-режимом
    public class Admin
    {
        private string Password = "12345"; // Пароль для админа
        private Inventory Inventory; // Товары

        public Admin(Inventory inventory)
        {
            Inventory = inventory;
        }

        // Метод: Вход в админ-режим
        public void AdminMode()
        {
            Console.WriteLine("Введите пароль:"); // админ должен ввести свой пароль
            string psswrd = Console.ReadLine();
            if (psswrd != Password)
            {
                Console.WriteLine("Неверный пароль");
                return;
            }
            bool inAdminMode = true;
            while (inAdminMode) // админ-режим будем действовать, пока сам админ из него не выйдет
            {
                Console.WriteLine("Админ-режим:");
                Console.WriteLine("1. Пополнить товары");
                Console.WriteLine("2. Собрать средства");
                Console.WriteLine("3. Выйти из Админ-режима");
                string choice = Console.ReadLine();
                if (choice == "1")
                {
                    Inventory.Restock();
                }
                else if (choice == "2")
                {
                    int collected = Inventory.CollectMoney();
                    Console.WriteLine($"Выручка: {collected} руб.");
                    Console.WriteLine("Средства собраны.");
                }
                else if (choice == "3")
                {
                    Console.WriteLine("Выход из админ-режима.");
                    inAdminMode = false;
                }
                else
                {
                    Console.WriteLine("Неверный выбор.");
                }
            }
        }
    }
    // Класс "Вендинговый автомат": управление работой автомата (товарами, админ-режимом и средствами покупателя)
    public class VendingMachine
    {
        private Inventory inventory; // Товары
        private Admin admin; // Админ
        private int balance;  // Текущий баланс

        public VendingMachine()
        {
            inventory = new Inventory();
            balance = 0;
            admin = new Admin(inventory);
        }

        // Метод: Пополнить баланс текущего покупателя
        public void InsertMoney(int money)
        {
            if (money > 0)
            {
                balance += money;
                Console.WriteLine($"Принято {money} руб. Баланс: {balance} руб.");
            }
            else
            {
                Console.WriteLine("Ошибка пополнения");
            }
        }

        // Метод: Показать товары и текущий баланс
        public void DisplayProductsAndBalance()
        {
            inventory.DisplayProducts();
            Console.WriteLine($"Текущий баланс: {balance} руб.");
        }

        // Метод: купить товар
        public void BuyProduct(int productNumber)
        {
            Product product = inventory.GetProduct(productNumber);
            if (product == null) return;

            if (product.Quantity == 0)
            {
                Console.WriteLine("Товара нет в наличии");
                return;
            }

            if (balance >= product.Price)
            {
                Console.WriteLine($"Покупка: {product.Name}");
                inventory.AddToCollected(product.Price);
                int change = balance - product.Price;
                balance = 0;
                product.Quantity -= 1;
            if (change > 0)
            {
                Console.WriteLine($"Сдача: {change} руб.");
            }
            }
            else
            {
                Console.WriteLine("Недостаточно средств");
            }
        }
        // Метод: Отмена и возварт средств
        public void Cancel()
        {
            if (balance > 0)
            {
                Console.WriteLine($"Возврат: {balance} руб.");
                balance = 0;
            }
            else
            {
                Console.WriteLine("Нет средств для возврата");
            }
        }

        // Метод: Вход в админ-режим
        public void AdminMode()
        {
            admin.AdminMode();
        }
    }
```
5. Определение главного класса МейнВорк.  
Основной класс.  
```
// Класс "МейнВорк": проверяет, какие действия выбирает покупатель, главный класс работы автомата
    class MainWork
    {
        static void Main()
        {
            VendingMachine machine = new VendingMachine();

            while (true)
            {
                Console.WriteLine("\nМеню вендингового автомата:");
                Console.WriteLine("1. Посмотреть товары");
                Console.WriteLine("2. Пополнить баланс");
                Console.WriteLine("3. Выбрать товар");
                Console.WriteLine("4. Отмена");
                Console.WriteLine("5. Админ-режим");
                Console.WriteLine("6. Выход");
                Console.Write("Выберите действие: ");
                string choice = Console.ReadLine();

                if (choice == "1")
                {
                    machine.DisplayProductsAndBalance();
                }
                else if (choice == "2")
                {
                    Console.WriteLine("Приложите карту к терминалу или вставьте купюру");
                    string input = Console.ReadLine();
                    int money;
                    if (int.TryParse(input, out money))
                    {
                        machine.InsertMoney(money);
                    }
                    else
                    {
                        Console.WriteLine("Ошибка пополнения счета");
                    }
                }
                else if (choice == "3")
                {
                    Console.WriteLine("Введите номер товара:");
                    string input = Console.ReadLine();
                    int number;
                    if (int.TryParse(input, out number))
                    {
                        machine.BuyProduct(number);
                    }
                    else
                    {
                        Console.WriteLine("Ошибка ввода");
                    }
                }
                else if (choice == "4")
                {
                    machine.Cancel();
                }
                else if (choice == "5")
                {
                    machine.AdminMode();
                }
                else if (choice == "6")
                {
                    Console.WriteLine("Выход из программы.");
                    break;
                }
                else
                {
                    Console.WriteLine("Ошибка");
                }
            }
        }
    }
```
## Проверка и скрины разных сценариев.
1. Основное меню:  
![1]()  
2. Выбор действия: 1 (Посмотреть товары)  
![2]()  
3. Выбор действия: 2 (Пополнить баланс)  
![3]()  
4. Выбор действия: 4 (Отмена)  
![4]()  
5. Выбор действия: 3 (Выбрать товар (без пополнения))  
![5]()  
6. Выбор действия: 3 (Выбрать товар (после пополнения))  
![6]()  
7. Выбор действия: 6 (Выход)  
![7]()  
8. Проверка на ошибки в предыдущих пунктах  
![8]()  
![9]()  
![10]()  
![11]()  
9. Выбор действия: 5 (Админ-режим)  
![12]()  
![13]()  
![14]()  
![15]()  
![16]()  

## Заключение
Всё работает согласно предоставленному ТЗ. Спасибо за внимание!

