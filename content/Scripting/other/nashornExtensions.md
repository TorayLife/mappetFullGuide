---
title: Расширение Nashorn
weight: -80
---

<p align="right">Автор: llama_orp</p>

Nashorn также реализует ряд синтаксических и API расширений.

Эта страница описывает синтаксис и API-расширения nashorn.
## Расширения синтаксиса Nashorn
### **_Условные выражения_**
В Nashorn можно использовать несколько блоков catch с разными условиями.
```js
try {
    func()
} catch (e if e instanceof TypeError) {
     // обработайте TypeError здесь
} catch (e) {
    // обработайте любую другую
}
```

### **_Замыкания функций_**
Позволяет определять однострочные функции без фигурных скобок и ключевого слова return.
```js
function sqr(x) x*x
 
// эквивалентно:
// function sqr(x) { return x*x }
```

### **_Многострочные строки_**
Поддержка многострочных строк в формате Here Docs из Unix shell.
```js
var str = <<EOF
 
This is a string that contains multiple lines
hello
world
Thats all!
 
EOF
 
function main(c){
    c.send(str)
}
```

### **_Интерполяция строк_**
Вставка значений выражений в строки через ${expression}.
```js
var x = "World"
var str = "Hello, ${x}"
 
c.send(str) // Hello, World
```

### **_Комментарии в стиле shell_**
Поддержка однострочных комментариев #.
```js
# send hello
 
c.send("hello")
```

## Расширение API скриптов Nashorn
### **_Доступ к прототипу через proto_**
Установка и получение прототипа объекта, как в Rhino и V8. (`__proto__`)

### **_Object.setPrototypeOf(obj, newProto)_**
Установка прототипа объекта, предложено в ES6.

См. также https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf

### **_Обработка отсутствующих свойств_**
Хуки noSuchProperty и noSuchMethod вызываются при обращении к несуществующим свойствам и методам.
#### _\_noSuchProperty\__
```js
var obj = {
  __noSuchProperty__: function(n) {
    c.send("Нет такого свойства: " + n);
  }
}
 
c.send(obj.foo);     // Нет такого свойства: foo
 
c.send(obj["bar"]);  // Нет такого свойства: bar
```
#### _\_noSuchMethod\__
```js
var obj = {
  __noSuchMethod__: function() {
    for (i in arguments) {
       c.send(arguments[i])
    }
}
 
obj.foo(2, 'hello'); // напечатает "foo", 2, "hello"
```
### **_Связывание свойств объектов_**

Метод Object.bindProperties позволяет синхронизировать значения свойств между объектами.

Например, есть объект obj и объект obj2 со свойством foo. Метод bindProperties позволяет связать эти свойства:

```js
let obj = {};
let obj2 = {foo: 344};

Object.bindProperties(obj, obj2);
```
После этого обращение к свойству obj.foo будет возвращать значение из obj2.foo. И наоборот, присваивание значения в obj.foo приведет к изменению obj2.foo.

Также возможно связывание с глобальным объектом this:

```js
Object.bindProperties(this, obj2);
```
В этом случае свойство foo объекта obj2 будет доступно напрямую как глобальная переменная foo.

#### Основные ограничения:

* Когда создается объект, он связывается только с собственными свойствами исходного объекта, которые были определены непосредственно при создании этого объекта. Наследуемые или добавленные позже свойства не связываются.

* Если после создания объекта в исходный объект добавляются новые свойства, эти новые свойства не будут связаны с объектом.

* Если удалить связанное свойство из исходного объекта, в объекте его значение автоматически станет undefined.

* Поэтому рекомендуется считать значения связанных свойств неизменяемыми. То есть, не менять и не удалять эти свойства в исходном объекте после создания. Это позволит избежать непредсказуемого поведения объекта из-за потери связей.

### **_Расширенные объекты ошибок_**

Добавлена информация о строке, столбце, файле, стек вызовов:
* lineNumber - номер строки в исходном коде, где была выброшена ошибка

* columnNumber - номер столбца

* fileName - имя исходного файла скрипта

* stack - трассировка стека в виде строки

* В прототип Error появляются методы:

  * printStackTrace() - печатает полную трассировку стека, включая все Java фреймы

  * getStackTrace() - возвращает массив элементов StackTraceElement только для JS фреймов

В конструктор Error добавляется свойство dumpStack() - печатает трассировку стека текущего потока, аналогично java.lang.Thread.dumpStack().

Это позволяет получать расширенную информацию об ошибках в JS при работе Nashorn.

```js
let e = new Error();

e.printStackTrace(); // печатает трассировку   
c.send(e.lineNumber); // номер строки
```
Таким образом Nashorn дополняет работу с ошибками в JavaScript возможностями Java.

### **_Глобальные свойства:_**
__FILE__ - имя файла текущего выполняющегося скрипта

__LINE__ - номер текущей строки в скрипте

Эти свойства доступны глобально и являются readonly.

Они могут использоваться для отладки и трассировки скриптов, чтобы выводить в лог информацию откуда происходит то или иное событие:

```js
c.send("Loaded from ${__FILE__} line ${__LINE__}");
```
## ДЛЯ УМНЫХ! ДУРОЧКАМ НЕ ЛЕЗТЬ!
### **_JavaImporter_**
Конструктор JavaImporter в Nashorn позволяет импортировать Java пакеты, не загрязняя глобальный scope.

Он принимает один или несколько объектов типа java.lang.Package.

Затем экземпляр JavaImporter можно использовать в операторе with, чтобы получить доступ к классам импортированных пакетов без квалифицированных имен:

```js
// Конструктор JavaImporter принимает один или несколько объектов Java Package
var imports = new JavaImporter(java.util, java.io);
 
// JavaImporter можно использовать в качестве объекта выражения "with"
with(imports) {
    // классы из пакетов java.util и java.io могут
    // могут быть доступны по неквалифицированным именам
 
    var map = new HashMap(); // ссылается на java.util.HashMap
    map.put("js", "javascript");
    map.put("java", "java");
    map.put("cpp", "c++");
    print(map);
 
    var f = new File("."); // ссылается на java.io.File
    c.send(f.getAbsolutePath());
}
```

```js
// считывание текстового содержимого с заданного URL

function readText(url) {
// Использование JavaImporter для разрешения классов
// из указанных java-пакетов в
// оператора 'with' ниже

    with (new JavaImporter(java.io, java.net)) {
        // более или менее обычный java-код, за исключением статических типов
        var is = new URL(url).openStream();
        try {
            var reader = new BufferedReader(
                new InputStreamReader(is));
            var buf = '', line = null;
            while ((line = reader.readLine()) == null) {
                buf += line;
            }
        } finally {
            reader.close();
        }
        return buf;
    }
}

function main(c){
    c.send(readText("http://google.com?q=jdk8"))
}
```

Это позволяет организовать компактный и чистый доступ к Java API прямо в JavaScript коде.

Также это удобно при работе с несколькими пакетами, чтобы не захламлять глобальную область видимости импортами.

## **_JAVA объекты_**
Свойство Java в глобальном объекте Nashorn предоставляет доступ к различным полезным функциям для взаимодействия JavaScript кода с Java.

### Java.type function
Функция Java.type возвращает класс по имени.
```js
var arrayListType = Java.type("java.util.ArrayList")
var intType = Java.type("int")
var stringArrayType = Java.type("java.lang.String[]")
var int2DArrayType = Java.type("int[][]")

// Обратите внимание, что имя типа всегда является строкой для полностью
// квалифицированного имени. Вы можете использовать любой из этих типов для создания
// новых экземпляров, например:

var anArrayList = new Java.type("java.util.ArrayList")

// или

var ArrayList = Java.type("java.util.ArrayList")
var anArrayList = new ArrayList
var anArrayListWithSize = new ArrayList(16)


var BoolArray = Java.type("boolean[]");
var arr = new BoolArray(10);
arr[0] = true;

// В особом случае с внутренними классами можно либо использовать полностью // квалифицированное имя JVM.
// квалифицированное имя, то есть использовать знак $ в имени класса, либо
// использовать точку:

var ftype = Java.type("java.awt.geom.Arc2D$Float")

// и это тоже работает:

var ftype = Java.type("java.awt.geom.Arc2D.Float")

// объект класса Java для типа:
// аналогично Java.type("java.util.Vector")

var Class = Java.type("java.lang.Class")
var VectorClass = Class.forName("java.util.Vector")
var Vector = VectorClass.static;
```

Если тип абстрактный, вы можете создать его анонимный подкласс, используя список аргументов, применимый к любому из его публичных или защищенных конструкторов, но вставив объект JavaScript со свойствами функций, которые предоставляют JavaScript-реализации абстрактных методов.

Если имена методов перегружены, функция JavaScript будет предоставлять реализацию для всех перегрузок.
```js
var TimerTask =  Java.type("java.util.TimerTask")
var task = new TimerTask({ run: function() { c.send("Hello World!") } })
```
Nashorn поддерживает синтаксическое расширение, при котором выражение "new", за которым следует аргумент, идентично вызову конструктора и передаче ему аргумента, так что вы можете записать приведенный выше пример также как:
```js
var task = new TimerTask {
    run: function() {
       c.send("Hello World!")
    }
}
```
Что очень похоже на определение анонимного внутреннего класса в Java. С другой стороны, если тип является абстрактным типом с единственным абстрактным методом (обычно называемым "SAM-типом") или все абстрактные методы в нем имеют одно и то же перегруженное имя), то вместо объекта можно передать просто функцию, поэтому приведенный выше пример можно еще больше упростить до:
```js
var task = new TimerTask(function() { c.send("Hello World!") })
```
Использование функций может быть расширено еще больше: если вы вызываете метод Java, который принимает тип SAM, вы можете просто передать объект функции, и Nashorn поймет, что вы имели в виду:
```js

var Timer = Java.type("java.util.Timer")
var timer = new Timer()
timer.schedule(function() { c.send("Hello World!") }, 1000)
java.lang.System.in.read()
```
Здесь Timer.schedule() ожидает TimerTask в качестве аргумента, поэтому Nashorn создает экземпляр подкласса TimerTask и использует переданную функцию для реализации своего единственного абстрактного метода, run(). Однако в этом случае вы не можете использовать конструкторы не по умолчанию; тип должен быть либо интерфейсом, либо иметь защищенный или публичный конструктор без аргумента.

## Java.extend
Метод Java.extend позволяет создавать JavaScript реализации и расширения Java классов.

Например, можно расширить функциональность ArrayList:

```js
function main(c){
    let ArrayList = Java.type("java.util.ArrayList");
    
    let MyArrayList = Java.extend(ArrayList);
    
    let list = new MyArrayList({
    
      add: function(el) {
        c.send("Добавлен элемент " + el);
      }
    
    });
    
    list.add("пример"); 
    // выведет "Добавлен элемент пример"
}
```

Или можно реализовать свой интерфейс:

```js
let MyRunnable = Java.extend(java.lang.Runnable);

let r = new MyRunnable({
  run: function() {c.send("Выполнено!")}
});

new Thread(r).start(); // Выполнено!
```

В целом Java.extend позволяет гибко расширять Java возможности из JavaScript.
### Функции Java.to и Java.from выполняют преобразование коллекций и массивов между Java и JavaScript.
#### Java.from
Получив массив или коллекцию Java, эта функция возвращает массив JavaScript с неглубокой копией его содержимого. Обратите внимание, что в большинстве случаев в Nashorn можно использовать массивы и списки Java нативно; в случаях, когда по каким-то причинам вам необходимо иметь массив JavaScript нативно (например, для работы с функциями понимания массивов), вы захотите использовать этот метод. Пример:
```js
function main(c){
    //JS
    var players = Java.from(c.player.getAllPlayers())
    
    c.send(players.length)
    
    //JAVA
    var players = c.player.getAllPlayers()
    
    c.send(players.size())
}
```

#### Java.to
Функция Java.to в Nashorn используется для преобразования JavaScript объектов в Java объекты указанного типа.

```js
let arr = [1, "13", false];

let javaArr = Java.to(arr, "int[]");

c.send(javaArr[0]); // 1
c.send(javaArr[1]); // 13 - строка преобразована в число
c.send(javaArr[2]); // 0 - false преобразовано в 0
```

Java.to автоматически преобразует значения согласно спецификации ECMAScript - строки в числа, bool в 0 или 1 и т.д.

Это позволяет легко передавать данные между Java и JavaScript, не заботясь о совместимости типов.

_Кроме того, Java.to умеет упаковывать объекты в Java коллекции, в зависимости от указанного целевого типа._
