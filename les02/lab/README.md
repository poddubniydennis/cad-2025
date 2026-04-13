# Отчет по лабораторной работе №1

**Дисциплина:** Разработка Java и C++ 
**Тема:** Разработка приложения для чтения и отображения данных из CSV-файла  
**Студент:** [Поддубный Денис Михайлович]  
**Группа:** [12002453]  

## Цель работы
Научиться создавать консольное приложение на Java с использованием Gradle для чтения данных из CSV-файла, парсинга строк в объекты и вывода их в консоль в виде таблицы.

## Задачи
1. Реализовать классы согласно заданию:
   - `Reader` / `ResourceFileReader` – чтение CSV-файла.
   - `Parser` / `CSVParser` – преобразование строк в объекты `Product`.
   - `ProductProvider` / `ConcreteProductProvider` – предоставление списка товаров.
   - `Renderer` / `ConsoleTableRenderer` – вывод таблицы в консоль.
   - `Product` – сущность "Товар" (поля: id, название, цена, количество и т.д.).
2. Настроить сборку проекта с помощью Gradle.
3. Обеспечить запуск командой `gradle run`.
4. Оформить отчёт в `README.md`.

## Выполнение работы

AppConfig.java:
package ru.bsuedu.cad.lab;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class AppConfig {
    
    @Bean
    public Reader reader() {
        return new ResourceFileReader("product.csv");
    }
    
    @Bean
    public Parser<Product> parser() {
        return new CSVParser();
    }
    
    @Bean
    public ProductProvider productProvider() {
        return new ConcreteProductProvider(reader(), parser());
    }
    
    @Bean
    public Renderer renderer() {
        return new ConsoleTableRenderer();
    }
}


ConcreteProductProvider.java:
package ru.bsuedu.cad.lab;

import java.util.List;

public class ConcreteProductProvider implements ProductProvider {
private final Reader reader;
private final Parser<Product> parser;

    public ConcreteProductProvider(Reader reader, Parser<Product> parser) {
        this.reader = reader;
        this.parser = parser;
    }

    @Override
    public List<Product> getProducts() {
        List<String> lines = reader.readAllLines();
        return parser.parse(lines);
    }
}


ConsoleTableRenderer.java:
package ru.bsuedu.cad.lab;

import java.util.List;

public class ConsoleTableRenderer implements Renderer {
    
    @Override
    public void render(List<Product> products) {
        String format = "| %-4s | %-30s | %-8s | %-15s |%n";
        
        System.out.println("+" + "-".repeat(6) + "+" + "-".repeat(32) + "+" + "-".repeat(10) + "+" + "-".repeat(17) + "+");
        System.out.printf(format, "ID", "Name", "Price", "Category");
        System.out.println("+" + "-".repeat(6) + "+" + "-".repeat(32) + "+" + "-".repeat(10) + "+" + "-".repeat(17) + "+");
        
        for (Product p : products) {
            System.out.printf(format, p.getId(), p.getName(), p.getPrice(), p.getCategory());
        }
        
        System.out.println("+" + "-".repeat(6) + "+" + "-".repeat(32) + "+" + "-".repeat(10) + "+" + "-".repeat(17) + "+");
    }
}


CSVParser. Java:
package ru.bsuedu.cad.lab;

import java.util.ArrayList;
import java.util.List;

public class CSVParser implements Parser<Product> {
    
    @Override
    public List<Product> parse(List<String> lines) {
        List<Product> products = new ArrayList<>();
        
        // Пропускаем заголовок (первую строку)
        for (int i = 1; i < lines.size(); i++) {
            String line = lines.get(i);
            // Убираем кавычки в начале и конце, если есть
            line = line.replace("'", "");
            String[] parts = line.split(",");
            
            if (parts.length >= 5) {
                // Индексы: 0 - name, 1 - description, 2 - category_id, 3 - price, 4 - stock_quantity
                String name = parts[0].trim();
                double price = Double.parseDouble(parts[3].trim());
                String category = parts[1].trim(); // используем description как категорию
                int id = i; // временный id
                
                products.add(new Product(id, name, price, category));
            }
        }
        return products;
    }
}


Main. Java:
package ru.bsuedu.cad.lab;

import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class Main {
    public static void main(String[] args) {
       ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        
        ProductProvider productProvider = context.getBean(ProductProvider.class);
        Renderer renderer = context.getBean(Renderer.class);
        
        renderer.render(productProvider.getProducts());
    }
}


Parser. Java:
package ru.bsuedu.cad.lab;

import java.util.List;

public interface Parser<T> {
    List<T> parse(List<String> lines);
}
Product.java:
package ru.bsuedu.cad.lab;

public class Product {
    private int id;
    private String name;
    private double price;
    private String category;

    public Product(int id, String name, double price, String category) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.category = category;
    }

    public int getId() { return id; }
    public String getName() { return name; }
    public double getPrice() { return price; }
    public String getCategory() { return category; }

    public void setId(int id) { this.id = id; }
    public void setName(String name) { this.name = name; }
    public void setPrice(double price) { this.price = price; }
    public void setCategory(String category) { this.category = category; }
}

ProductProvider.java:
package ru.bsuedu.cad.lab;

import java.util.List;

public interface ProductProvider {
    List<Product> getProducts();
}


Reader.java:
package ru.bsuedu.cad.lab;

import java.util.List;

public interface Reader {
    List<String> readAllLines();
}


Renderer.java:
package ru.bsuedu.cad.lab;

import java.util.List;

public interface Renderer {
    void render(List<Product> products);
}


ResourceFileReader.java:
package ru.bsuedu.cad.lab;

import java.io.BufferedReader;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.nio.charset.StandardCharsets;
import java.util.List;
import java.util.stream.Collectors;

public class ResourceFileReader implements Reader {
    private final String filePath;

    public ResourceFileReader(String filePath) {
        this.filePath = filePath;
    }

    @Override
    public List<String> readAllLines() {
        try (InputStream is = getClass().getClassLoader().getResourceAsStream(filePath);
             BufferedReader reader = new BufferedReader(new InputStreamReader(is, StandardCharsets.UTF_8))) {
            
            return reader.lines().collect(Collectors.toList());
        } catch (Exception e) {
            throw new RuntimeException("Ошибка чтения файла: " + filePath, e);
        }
    }
}


### Описание классов

**Product** – класс, описывающий товар. Содержит поля:
- `id` (int)
- `name` (String)
- `price` (double)
- `quantity` (int)

**Reader** – интерфейс с методом `readAllLines()`, возвращающим список строк.  
**ResourceFileReader** – реализация, читающая CSV-файл из ресурсов.

**Parser** – интерфейс с методом `parse(List<String> lines)`.  
**CSVParser** – реализация, преобразующая каждую строку (кроме заголовка) в объект `Product`.

**ProductProvider** – интерфейс с методом `getProducts()`.  
**ConcreteProductProvider** – реализация, вызывающая Reader и Parser.

**Renderer** – интерфейс с методом `render(List<Product> products)`.  
**ConsoleTableRenderer** – реализация, выводящая данные в виде таблицы с использованием `System.out.printf`.
<img width="1706" height="924" alt="lab1" src="https://github.com/user-attachments/assets/4e96ef99-90a4-441c-a0a9-1dcf2e73d941" />


### Сборка и запуск

```bash
# Переход в директорию лабораторной работы
cd les02/lab

# Запуск приложения через Gradle
gradle run
<img width="818" height="349" alt="test_lab1" src="https://github.com/user-attachments/assets/41de4959-6ed3-42e7-83ef-b70a33298d97" />

+------+--------------------------------+----------+-----------------+
| ID   | Name                           | Price    | Category        |
+------+--------------------------------+----------+-----------------+
| 1    | 1                              | 1.0      | Сухой корм для собак |
| 2    | 2                              | 2.0      | Игрушка для кошек "Мышка" |
| 3    | 3                              | 3.0      | Лакомство для попугаев |
| 4    | 4                              | 4.0      | Когтеточка для кошек |
| 5    | 5                              | 5.0      | Гель для чистки ушей собак |
| 6    | 6                              | 6.0      | Аквариум 50 литров |
| 7    | 7                              | 7.0      | Наполнитель для кошачьего туалета |
| 8    | 8                              | 5.0      | Шампунь для собак с алоэ |
| 9    | 9                              | 8.0      | Клетка для хомяков |
| 10   | 10                             | 9.0      | Поводок для собак 3м |
+------+--------------------------------+----------+-----------------+
Контрольные вопросы:
1. Spring. Определение, назначение, особенности
•	Определение: Легковесный фреймворк для Java, реализующий инверсию управления (IoC).
•	Назначение: Упрощает разработку корпоративных приложений: управление бинами, внедрение зависимостей, транзакции, безопасность, веб-модули, доступ к данным.
•	Особенности: Модульность, не требует сервера приложений, конфигурация (XML, аннотации, JavaConfig), интеграция с другими фреймворками.
2. Проблемы ручной сборки приложений
•	Управление зависимостями (версии, транзитивные).
•	Повторяющиеся действия (компиляция, копирование).
•	Настройка classpath вручную.
•	Трудности с автоматизацией тестов и развёртыванием.
•	Отсутствие единого подхода в команде.
3. Системы автоматической сборки (кратко о каждой)
•	Maven: Декларативная (POM), управление зависимостями из центральных репозиториев, стандартная структура.
•	Gradle: Декларативная + гибкость (DSL на Groovy/Kotlin), инкрементальная сборка, кэширование.
•	Ant: Процедурная (XML), требует ручного управления зависимостями (обычно с Ivy).
•	Make: Классическая (для C/C++), для Java используется редко.
•	Bazel: От Google, для больших монорепозиториев, поддерживает несколько языков.
4. Типовая структура Java проекта
text
project/
├── src/
│   ├── main/
│   │   ├── java/       – исходные коды
│   │   └── resources/  – ресурсы
│   └── test/
│       ├── java/       – тесты
│       └── resources/  – ресурсы для тестов
├── build.gradle / pom.xml
└── README.md
5. Типы зависимостей в Gradle
•	implementation – основная, не транслируется в API.
•	api – входит в публичный API модуля.
•	compileOnly – только для компиляции (не в runtime).
•	runtimeOnly – только для выполнения.
•	testImplementation – для тестов.
•	testRuntimeOnly – для выполнения тестов.
•	annotationProcessor – для процессоров аннотаций.
6. Принцип инверсии управления (IoC). Определение и применение
•	Определение: Контроль над созданием и связыванием объектов передаётся от кода внешнему контейнеру.
•	Применение: Снижение связанности, упрощение тестирования, централизованное управление жизненным циклом объектов.
7. Отличие IoC от внедрения зависимостей (DI)
•	IoC – широкий принцип: фреймворк управляет потоком и созданием объектов.
•	DI – способ реализации IoC: зависимости передаются извне (конструктор, сеттер, поле).
IoC может быть без DI (например, Service Locator), но DI – предпочтительный способ.
8. Принципы инверсии управления (кратко)
•	Dependency Inversion (DIP): Зависимости от абстракций, а не от конкретных реализаций.
•	Inversion of Control: Управление передаётся контейнеру.
•	Hollywood Principle: «Не звоните нам, мы вам позвоним» – контейнер вызывает объекты.
•	Single Responsibility: Объекты занимаются бизнес-логикой, а не созданием зависимостей.
•	Open/Closed: Легко добавлять новые реализации без изменения существующего кода.
9. Сцепление (Coupling) и связность (Cohesion)
•	Coupling – степень зависимости между модулями. Низкое сцепление – хорошо.
•	Cohesion – степень сосредоточенности элементов модуля на одной задаче. Высокая связность – хорошо.
IoC/DI помогают снизить сцепление и повысить связность.
10. Какой принцип внедрения зависимости желательно использовать и почему
•	Constructor injection (внедрение через конструктор) – предпочтительный способ.
•	Почему: поля можно сделать final; все зависимости явно видны; упрощается тестирование; исключаются циклические зависимости; объект полностью сконструирован после создания.

