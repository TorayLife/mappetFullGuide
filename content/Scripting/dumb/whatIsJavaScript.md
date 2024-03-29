---
title: Что такое Javascript и зачем он нужен в Mappet
weight: -110
---

{{< include file="/_includes/reconstruction.md" type="page" >}}

<p align="right">Автор: TorayLife</p>

Если ты это читаешь, то вероятно ты столкнулся с ограничениями маппета, 
которые не позволили сделать тебе ту или иную фишку. 

Однако, прежде чем читать это неправильное руководство я советую тебе ознакомиться 
с [основным функционалом]({{< ref "Mappet" >}} ). Возможно то что 
ты хочешь сделать всё же достижимо при помощи обычных стейтов и ты просто не знаешь об этом.

Mappet предоставляет API для того чтобы JS мог 
взаимодействовать с модом и майнкрафтом. JS сам по себе не имеет механизмов которые 
работают с внешней средой - все такие механизмы предоставляются средой.

{{< hint type=note >}}
**API** — программный интерфейс, то есть описание способов взаимодействия одной компьютерной программы с другими.\
[Википедия](https://ru.wikipedia.org/wiki/API)
{{< /hint >}}

JavaScript (**Не путать с Java!**) - это язык, который позволяет писать скрипты - небольшие программы,
которые можно выполнять в той среде, в которую его встроили. В нашем случае это Mappet.

При помощи JS можно реализовать механики, которые невозможно или очень сложно сделать
при помощи стандартного функционала. РПГ системы прокачки, игры с таймерами, и в целом
любая кастомная логика которую можно придумать.

## Примеры:
{{< tabs "uniqueid" >}}
{{< tab "Кастомизация персонажа" >}} {{< youtube 9BBckTVyO-0>}} {{< /tab >}}
{{< tab "Игровой режим" >}} {{< youtube 6r3ZIkzdOp0>}} {{< /tab >}}
{{< tab "Ролевой чат" >}} {{< youtube bBvrIfTjiQ0>}} {{< /tab >}}
{{< tab "Система поднимания сущностей" >}}
<video controls height="210" poster="https://media.discordapp.net/attachments/841271590326894603/966320063592411186/63036490f600bdb0b72ca6eb03994311.mp4?ex=658c1db0&amp;is=6579a8b0&amp;hm=9132ee10fb2b5c5bef55910e1eb7c7c5abc069826a4da618ffe95a6ef5536549&amp;=&amp;format=webp" width="400" src="https://media.discordapp.net/attachments/841271590326894603/966320063592411186/63036490f600bdb0b72ca6eb03994311.mp4?ex=658c1db0&amp;is=6579a8b0&amp;hm=9132ee10fb2b5c5bef55910e1eb7c7c5abc069826a4da618ffe95a6ef5536549&amp;" style="max-width: 1200px; max-height: 630px; width: 100%; height: 100%;"></video>

{{< /tab >}}
{{< /tabs >}}