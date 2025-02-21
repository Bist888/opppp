#include <iostream>
#include <string>
#include <vector>
#include <map>
#include <limits> //numeric_limits
#include <algorithm>

// Класс Product (Товар) 
class Product {
public:
    int id;
    std::string name;
    std::string description;

    Product(int id, const std::string& name, const std::string& description) :
        id(id), name(name), description(description) {
    }

    int getId() const { return id; }
    std::string getName() const { return name; }
    std::string getDescription() const { return description; }

    // Перегрузка оператора сравнения для использования в качестве ключа в std::map
    bool operator<(const Product& other) const {
        return id < other.id; // Используем id для сравнения
    }

    // Перегрузка оператора вывода в поток
    friend std::ostream& operator<<(std::ostream& os, const Product& product) {
        os << "ID: " << product.id << ", Name: " << product.name;
        return os;
    }
};

// Класс StockItem (Запас товара)
class StockItem {
public:
    Product product;
    double price;
    int quantity;

    StockItem(const Product& product, double price, int quantity) :
        product(product), price(price), quantity(quantity) {
    }

    double getPrice() const { return price; }
    int getQuantity() const { return quantity; }
    void setQuantity(int newQuantity) { quantity = newQuantity; }

    friend std::ostream& operator<<(std::ostream& os, const StockItem& item) {
        os << "Товар: " << item.product.name
            << ", Цена: " << item.price
            << ", Количество: " << item.quantity;
        return os;
    }
};

// Класс Shop (Магазин)
class Shop {
public:
    int id;
    std::string name;
    std::string address;
    std::map<Product, StockItem> stock;

    Shop(int id, const std::string& name, const std::string& address) :
        id(id), name(name), address(address) {
    }

    int getId() const { return id; }
    std::string getName() const { return name; }
    std::string getAddress() const { return address; }

    //Добавление товара в магазин или обновление существующего запаса
    void addStock(const Product& product, double price, int quantity) {
        stock.emplace(product, StockItem(product, price, quantity)); // Создаем StockItem прямо в emplace
    }

    //Получение запаса товара в магазине
    StockItem* getStockItem(const Product& product) {
        auto it = stock.find(product);
        if (it != stock.end()) {
            return &it->second; // Возвращаем указатель на найденный StockItem
        }
        return nullptr; // Если товар не найден, возвращаем nullptr
    }

    //Вывод информации о магазине и его товарах
    friend std::ostream& operator<<(std::ostream& os, const Shop& shop) {
        os << "Магазин ID: " << shop.id << ", Название: " << shop.name << ", Адрес: " << shop.address << std::endl;
        os << "Товары в наличии:" << std::endl;
        for (const auto& pair : shop.stock) {
            os << "  - " << pair.second << std::endl;
        }
        return os;
    }
};

// Класс Customer (Покупатель)
class Customer {
public:
    int id;
    std::string name;
    double balance;
    std::vector<std::pair<Product, int>> shoppingList;

    Customer(int id, const std::string& name, double balance) :
        id(id), name(name), balance(balance) {
    }

    int getId() const { return id; }
    std::string getName() const { return name; }
    double getBalance() const { return balance; }

    void addBalance(double amount) { balance += amount; }

    //Снятие средств с баланса
    bool removeBalance(double amount) {
        if (balance >= amount) {
            balance -= amount;
            return true;
        }
        else {
            std::cout << "Недостаточно средств на балансе." << std::endl;
            return false;
        }
    }

    //Добавление товара в список покупок
    void addToShoppingList(const Product& product, int quantity) {
        shoppingList.push_back(std::make_pair(product, quantity));
    }

    //Очистка списка покупок
    void clearShoppingList() {
        shoppingList.clear();
    }

    //Вывод информации о покупателе
    friend std::ostream& operator<<(std::ostream& os, const Customer& customer) {
        os << "Покупатель ID: " << customer.id << ", Имя: " << customer.name << ", Баланс: " << customer.balance << std::endl;
        os << "Список покупок:" << std::endl;
        for (const auto& item : customer.shoppingList) {
            os << "  - " << item.first.name << ", Количество: " << item.second << std::endl;
        }
        return os;
    }

    Customer() = default;
};

// Класс  (Система управления магазинами)
class ShoppingSystem {
public:
    std::vector<Shop> shops;
    std::vector<Customer> customers;
    std::vector<Product> products;

    //Добавление магазина
    void addShop(const Shop& shop) {
        shops.push_back(shop);
    }

    //Добавление товара
    void addProduct(const Product& product) {
        products.push_back(product);
    }

    //Добавление покупателя
    void addCustomer(const Customer& customer) {
        customers.push_back(customer);
    }

    //Поиск магазина с самой низкой ценой на товар
    Shop* findShopWithLowestPrice(const Product& product) {
        Shop* lowestPriceShop = nullptr;
        double lowestPrice = std::numeric_limits<double>::max(); //Устанавливаем начальную цену как максимальное возможное значение double

        for (Shop& shop : shops) {
            StockItem* stockItem = shop.getStockItem(product);
            if (stockItem != nullptr && stockItem->getPrice() < lowestPrice) {
                lowestPrice = stockItem->getPrice();
                lowestPriceShop = &shop;
            }
        }

        return lowestPriceShop;
    }

    //Совершение покупки
    void purchase(Customer& customer, Shop& shop) {
        double totalPrice = 0.0;
        bool canPurchase = true;

        std::cout << "Начинаем покупку в магазине: " << shop.name << std::endl;

        //Проверяем наличие товаров и расчитываем общую стоимость
        for (const auto& item : customer.shoppingList) {
            Product product = item.first;
            int quantity = item.second;

            StockItem* stockItem = shop.getStockItem(product);
            if (stockItem == nullptr) {
                std::cout << "Товара " << product.name << " нет в наличии в магазине " << shop.name << std::endl;
                canPurchase = false;
                break;
            }

            if (stockItem->getQuantity() < quantity) {
                std::cout << "Недостаточно товара " << product.name << " в магазине " << shop.name << std::endl;
                canPurchase = false;
                break;
            }

            totalPrice += stockItem->getPrice() * quantity;
        }

        //Если все товары есть в наличии и у покупателя достаточно средств, совершаем покупку
        if (canPurchase) {
            if (customer.getBalance() >= totalPrice) {
                //Снимаем деньги с баланса покупателя
                customer.removeBalance(totalPrice);

                //Обновляем запасы магазина
                for (const auto& item : customer.shoppingList) {
                    Product product = item.first;
                    int quantity = item.second;
                    StockItem* stockItem = shop.getStockItem(product);
                    stockItem->setQuantity(stockItem->getQuantity() - quantity);
                }

                std::cout << "Покупка прошла успешно." << std::endl;
                std::cout << "Списано с баланса: " << totalPrice << std::endl;
                std::cout << "Остаток на балансе: " << customer.getBalance() << std::endl;

                customer.clearShoppingList(); // Очищаем список покупок после успешной покупки
            }
            else {
                std::cout << "Недостаточно средств для совершения покупки." << std::endl;
            }
        }
        else {
            std::cout << "Невозможно совершить покупку из-за отсутствия товаров или недостаточного количества." << std::endl;
        }
    }
};

int main() {
    setlocale(LC_ALL, "RU");
    // Создание товаров
    Product product1(1, "Хлеб", "Белый хлеб");
    Product product2(2, "Молоко", "Молоко 3.2%");
    Product product3(3, "Яблоки", "Красные яблоки");

    // Создание магазинов
    Shop shop1(101, "Магазин у дома", "ул. Ленина, 1");
    Shop shop2(102, "Супермаркет", "пр. Мира, 10");

    // Добавление товаров в магазины
    shop1.addStock(product1, 35.0, 50);
    shop1.addStock(product2, 70.0, 30);
    shop2.addStock(product1, 30.0, 100);
    shop2.addStock(product2, 65.0, 80);
    shop2.addStock(product3, 100.0, 40);

    // Создание покупателей
    Customer customer1(201, "Иван Иванов", 500.0);
    Customer customer2(202, "Петр Петров", 1000.0);

    // Создание системы управления магазинами
    ShoppingSystem system;

    // Добавление магазинов, товаров и покупателей в систему
    system.addShop(shop1);
    system.addShop(shop2);
    system.addProduct(product1);
    system.addProduct(product2);
    system.addProduct(product3);
    system.addCustomer(customer1);
    system.addCustomer(customer2);

    // Поиск магазина с самой низкой ценой на хлеб
    Shop* lowestPriceShop = system.findShopWithLowestPrice(product1);
    if (lowestPriceShop != nullptr) {
        std::cout << "Самая низкая цена на " << product1.name << " в магазине: " << lowestPriceShop->name << std::endl;
    }
    else {
        std::cout << "Товар " << product1.name << " не найден ни в одном магазине." << std::endl;
    }

    // Покупатель добавляет товары в список покупок
    customer1.addToShoppingList(product1, 2);
    customer1.addToShoppingList(product2, 1);

    // Покупатель совершает покупку в магазине
    system.purchase(customer1, shop1);

    // Вывод информации о покупателе после покупки
    std::cout << customer1 << std::endl;

    // Вывод информации о магазине после покупки
    std::cout << shop1 << std::endl;

    return 0;
}
