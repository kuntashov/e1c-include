# Модульность для с 1C:Executor в стиле include из PHP

**Это просто эксперимент/исследование/изучение 1С:Исполнителя**

МТП (Максимально Тупой Прототип ©️) модульности в 1С:Исполнителе по принципу `include` в PHP.

Подход: специальным скриптом-сборщиком (`src/build.sbsl`) в целевом скрипте обрабатываются места объявления `include` и вместо объявлений подставляется текст "библиотечного" скрипта, путь к которому указан в качестве аргумента `include`.

Использование модуля определяется конструкцией (пример):  `const Greetings = "include fixtures/greetings.sbsl"`. Такой формат - спонтанная попытка обойти диагностики в VS Code (чтобы не выделяло Greetings как неизвестную переменную), но хотя по факту это помогает обойти ошибку неизвестной переменной Greetings, но экспортируемые методы модуля все равно выделяются как неизвестные.  

В библиотечном скрипте экспортные методы никак специально не объявляются.

Предполагается, что в скрипте, использующем библиотечные методы, эти методы вызываются через точку от имени константы, используемой при объявлении библиотечного скрипта. Но это тоже сделано спонтанно, без какого-либо проектирования/обдумывания, буду переделывать (если разработчики 1С:Исполнителя затянут с настоящей поддержкой модульности, а у меня появится еще час-полтора это все переосмыслить и переписать).

## Пример использования:

Скрипт, который использует библиотеку `greetings.bsl`:

```sbsl
const Greetings = "include fixtures/greetings.sbsl"

method Script()
    Greetings.Hello("1C")
    Greetings.Hi("all")
;
```

Сам "библиотечный" скрипт: `greetings.bsl`:

```sbsl
method Hello(Name: String)
    Console.Write("Hello, %Name!")
;

method Hi(Name: String)
    Console.Write("Hi, %Name!")
;
```

Собираем скрипт:

```sh
$  executor -s src/builder.sbsl ./fixtures/hello.sbsl
```

В результате в папке `./build` появится результирующий `script.sbsl` такого содержания:

```sbsl
////////////////////////////////////////////////////////////////////////////////
// fixtures/greetings.sbsl
////////////////////////////////////////////////////////////////////////////////
method Greetings__Hello(Name: String)
    Console.Write("Hello, %Name!")
;

method Greetings__Hi(Name: String)
    Console.Write("Hi, %Name!")
;

////////////////////////////////////////////////////////////////////////////////
// ./fixtures/hello.sbsl
////////////////////////////////////////////////////////////////////////////////
const Greetings = "include fixtures/greetings.sbsl"

method Script()
    Greetings__Hello("1C")
    Greetings__Hi("all")
;

```

## Идеи/TODO

* [ ] Изменить формат объявления include
* [ ] Использовать иерархию каталогов модулей:
   * [ ] Имя модуля = имя файла с его исходниками 
   * [ ] Иерархия папок, имена подкаталогов - неймспейсы (?)
* [ ] Опция для запуска скрипта сразу после сборки (со сборкой во временный каталог)
