---
published: false
---

## Подробнее про придумку

Конечно, вокруг модели акторов построена нехилая математика, точнее, физика. Так что в среде агентов должны соблюдаться физические принципы распространения и последовательности сигналов. Но они почти все интуитивно понятны, и кажется, что читаешь всякое капитанство.

Попробую описать будущую модель среды и агентов. Напомню, есть агенты, у них есть входные и выходные каналы сообщений, в которые могут быть переданы собственно сообщения. 

Изначально нет никого, в какой-то момент поступает команда на создание первого агента известного типа. Этот агент (Топ) может создать конечное число других агентов, а может и не создать. Создание агента равносильно объявлению переменной c типом -- идентификатором модуля.

Создавая агентов, Топ посылать им сообщения во входные каналы и может производить запрос на значения из выходных каналов этих агентов. Но недостаточно сделать запрос на значение, надо дождаться ответа, таким образом, если после итерации есть запросы на значения, они должны быть удовлетворены. При этом очередная итерация Топа не может начаться без окончания всех вызванных ожиданий (не совсем ясно насчет дедлоков). 

***

Ситуация, когда Топ создает агентов, и не ожидает от них сообщений тоже возможна, при этом дальнейшей судьбой этих агентов Топ может не интересоваться, или интересоваться на общих основаниях, широковещательно (продумать этот момент)

Пока рассмотрим случай когда мы создаем агента для выполнения работы с результатом.

***

В свою очередь созданные агенты (Фиб, Факт) реально вступают в действие после того, как возник запрос на их данные в итерации Топа, после ее окончания. Эти агенты получают так же сообщение во входные каналы `n`. 

Фиб проверяет значение `n` и решает, что необходимо запросить значение для выходного канала у другого агента, попутно Фиб инициирует другого агента Фибi, которому передает значение входного параметра, уменьшенное на 1. Так как возник запрос, и больше нет действий, итерация завершается и переходит в состояние ожидания. Однако, если условие `n # 0` становится ложным, ожидание значения для `res` реализуется по месту, константой, и следовательно, итерация Фибi завершается, одновременно с этим отправляя сообщение Фибi-1, который тоже завершает ожидание. В итоге, в первоначальный Фиб придет значение n-го числа Фибоначчи.

Факт поступает по другому. Он не инициирует своих собратьев, но вместо этого каждый раз передает нужное значение в собственный регистр `x`, который сохраняет значение между итерациями одного агента Факт (после завершения итераций регистр всё же пропадает). Так как выходное значение, которое требуется от агента не передано в выходной канал, итерация повторяется. Здесь возникает странный момент, что считать завершающей итерацией агента? Удовлетворение всех выходных сообщений, всех ожиданий и отсутствие выполняемых действий? И опять проблема дедлоков.

Временной график

|↓шаг|агент→|Top|Fib|Fact|Fib...|
|:---:|---|---|---|---|---|
|0||NEW||||
|1||<-Fib, <-Fact||||
|2|||NEW|NEW||
|3|||<-Fib|@||
|4||||@|NEW|
|..||здесь порождаются Fib i-ые|..|..res<-|..|
|N-3|||||res<-|
|N-2|||res<-|||
