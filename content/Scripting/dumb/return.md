---
title: Оператор return
weight: -50
---

### Оператор `return` в JavaScript используется для возврата значения из функции.

**Где можно использовать return:**

- В любой обычной функции:

```js
function sum(a, b) {
  return a + b;  
}

let result = sum(1, 2); // 3
```

- В методах объекта:

```js
let user = {
  getName() {
    return this.name; 
  }
}
```

**Что происходит при вызове `return`?**

- Функция сразу же прекращает выполнение
- Передает управление обратно вызвавшему коду
- Возвращает указанное после `return` значение

Таким образом `return` используется для возврата результатов работы функции.

# **_ВАЖНО:_**
{{< hint type=note >}}
- Вы также можете не указывать никакого значения после return. В этом случае из функции будет возвращено значение undefined.
{{< /hint >}}

Это бывает полезно, когда нужно просто досрочно выйти из функции, не возвращая конкретного результата.

Кроме того, можно использовать return без значения в комбинации с оператором if:
```js
function main(c){
    if(c.player.hp == 5) {
       c.send("Ваше здоровье ровно 5")
       return;
    }

    if(c.player.hp > 10){
        c.send("Ваше здоровье больше 10 хп");
    }
}
```