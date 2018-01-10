# ReactiveKit
### Signal
Сигналы принимают в качестве аргумента наблюдателя. Когда мы начинаем наблюдать за сигналом, то он выполняется(холодный сигнал).

~~~swift
let bag = DisposeBag()

let signal = Signal<Int, NoError> { observe in
    observe.next(1)
    observe.completed(with: 2)
    return bag
}
~~~
Пока за сигналом никто не наблюдает он находится в покое. Теперь создадим наблюдателя.

~~~swift
signal.observe { (event) in
    print(event)
}
~~~
После того, как мы подписались на сигнал и стали его наблюдателем, он отработает и в консоли мы увидим следующее.

> `next(1)`
> 
> `next(2)`
> 
> `completed`

В случае, если мы уверены, что при выполнении сигнала, не будет ошибки, то мы можем использовать "безопасный сигнал".

~~~swift
let safeSignal = SafeSignal<Int> { observe in
    observe.next(1)
    observe.completed(with: 2)
    return bag
}
~~~

### Наблюдатель(Observer)
Для того, чтобы подписаться на объект, нам необходимо использовать функцию, а вернее одну из трёх вариантов функции наблюдателя. 
`observe`,
`observeNext`,
`observeFailed`.

Разница в том, что функция `observe` получает в событии и элемент и ошибку, функция `observeNext` получает только элемент, а функция `observeFailed` получает только ошибку.
Так как это функция, мы можем передать замыкание нашему методу наблюдения. 

~~~swift
signal.observe { (event) in
    print(event)
}
~~~
В консоли увидим следующее:

> `next(1)`
> 
> `next(2)`
> 
> `completed`

~~~swift
signal.observeNext { (event) in
    print(event)
}
~~~

> `1`
> 
> `2`
> 
> `nil`

### Subjects (горячий сигнал)
Если сигналы срабатывали только тогда, когда мы подписывались на них, поэтому они называются "холодными сигналами", то subject'ы работают независимо от того, подписан на них кто-то или нет, поэтому их называют "горячими сигналами". 

Есть несколько видов subject'ов:
`PublishSubject`, `ReplaySubject`, `ReplayOneSubject`, `Property`.

#### PublishSubject
PublishSubject получая событие, сразу отдает его всем, кто на него подписан. Как это выглядит в коде.

~~~swift
let subject = PublishSubject<Int, NoError>()

subject.observeNext { (event) in
    print("первый -> \(event)")
}
~~~
Теперь будем передавать события

~~~swift
subject.next(3)
subject.next(1)
~~~
Что мы увидим в консоли.

> `первый -> 3`
> 
> `первый -> 1`

Теперь подпишем второго наблюдателя.

~~~swift
subject.observeNext { (event) in
    print("второй -> \(event)")
}
~~~
После подписки передадим событие

~~~swift
subject.next(4)
~~~

Что мы увидим после этого в консоли

> `первый -> 4`

> `второй -> 4`

Как мы можем увидеть, Publish Subject в отличии от сигнала, отдает события как только получает их, поэтому наблюдатель начинает получать события только с момента подписки, а не все, которые были отправлены subject'ом.

#### ReplaySubject

ReplaySubject хранит в себе все события и передает их новому наблюдателю, как это выглядит.

~~~swift
let replaySubject = ReplaySubject<Int, NoError>()

replaySubject.observeNext { (event) in
    print("первый -> \(event)")
}

replaySubject.next(1)
replaySubject.next(7)
~~~

В консоли отобразится следующее

> `первый -> 1`
> 
> `первый -> 7`
 
Теперь создадим второго наблюдателя и посмотрим, что изменится в консоли.

~~~swift
replaySubject.observeNext { (event) in
    print("второй -> \(event)")
}
~~~

> `первый -> 1`
> 
> `первый -> 7`
> 
> `второй -> 1`
> 
> `второй -> 7`
 
Мы видим, что как только второй наблюдатель подписался на subject, то он сразу же получил все события, что были до его подписки. Теперь отправим еще одно событие и посмотрим в консоль.

~~~swift
replaySubject.next(3)
~~~

> `первый -> 3`
> 
> `второй -> 3`

Тут для нас нет ничего необычного, обычная работа subject'а.

#### ReplayOneSubject

ReplayOneSubject очень похож на ReplaySubject, за одним исключением, он передает новому подписчику только последнее значение, изменим наш предыдущий код и посмотрим, что получится.

~~~swift
let replayOneSubject = ReplayOneSubject<Int, NoError>()

replayOneSubject.observeNext { (event) in
    print("первый -> \(event)")
}

replayOneSubject.next(1)
replayOneSubject.next(7)

replayOneSubject.observeNext { (event) in
    print("второй -> \(event)")
}

replayOneSubject.next(3)
~~~

Посмотрим в консоль.
> `первый -> 1`
> 
> `первый -> 7`
> 
> `второй -> 7`
> 
> `первый -> 3`
> 
> `второй -> 3`

То есть второй подписчик не получил всех событий, которые были отправлены до его подписки, а получил только последнее событие.

#### Property

Property выполняет те же функции, что и ReplayOneSubject, с одним исключением. Для Property можно задать стартовое событие при его инициализации, которое он точно отправит подписчику. При новой подписке он отправит последнее событие, если с момента его инициализации новых событий не было, то отправит то, что было при инициализации.

~~~swift
let property = Property(2)

property.observeNext { (event) in
    print("первый -> \(event)")
}

property.next(4)
property.next(2)
property.next(1)

property.observeNext { (event) in
    print("второй -> \(event)")
}

property.next(4)
~~~

> `первый -> 2`
> 
> `первый -> 4`
> 
> `первый -> 1`
> 
> `второй -> 1`
> 
> `первый -> 4`
> 
> `второй -> 4`

### Операторы

ReactiveKit предоставляет большое количество полезных операторов, в этой главе мы рассмотрим их.

#### take
Оператор `take` позволяет нам взять определенное количество событий из последовательности, посмотрим на код.

~~~swift
let array = PublishSubject<String, NoError>()

array.take(first: 2).observeNext { (e) in
    print(e)
}

array.next("P")
array.next("A")
array.next("E")
~~~

> `P`
> 
> `A`

есть 3 вариации этого оператора `take` `first/lats/until`

#### filter

Оператор позволяет отбрасывать ненужные нам события и обрабатывать только те, что прошли проверку и вернули `true`.

~~~swift
let array = PublishSubject<String, NoError>()

array.filter {$0.hasPrefix("P")}.observeNext { e in
    print(e)
}

array.next("P")
array.next("A")
array.next("E")
~~~

> `P`

#### throttle

Оператор `throttle ` позволяет установить задержку, если в течении `n` времени не пришло событий, то берет последнее.

~~~swift
array.throttle(seconds: 2.0).observeNext { (e) in
    print(e)
}

array.next("P")
array.next("A")
array.next("E")
~~~

> `E`

#### debug
Очень полезный метод для отладки - `debug`. Использовать его очень просто, возьмем пример с `take` и добавим туда `debug`.

~~~swift
let array = PublishSubject<String, NoError>()

array
	.debug("Оператор Debug")
	.take(first: 2).observeNext { (e) in
    print(e)
}

array.next("P")
array.next("A")
array.next("E")
~~~

> `[Оператор Debug] started`
> 
> `[Оператор Debug] next(P)`
> 
> `P`
> 
> `[Оператор Debug] next(A)`
> 
> `A`
> 
> `[Оператор Debug] disposed`

#### merge

Оператор позволяет соединить 2 последовательности

~~~swift
let seq = Signal<Int, NoError>.sequence([1, 3, 2, 5])
let seq2 = Signal<Int, NoError>.sequence([2, 1, 4, 7])

let mergeSeq = seq.merge(with: seq2)

mergeSeq.observeNext { (e) in
    print(e)
}
~~~

> `1`
> 
> `3`
> 
> `2`
> 
> `5`
> 
> `2`
> 
> `1`
> 
> `4`
> 
> `7`

#### zip

Чтобы понять что представляет из себя оператора `zip` проще посмотреть на результат его работы.

~~~swift
let seq = Signal<Int, NoError>.sequence([1, 3, 2, 5])
let seq2 = Signal<Int, NoError>.sequence([2, 1, 4, 7])

let ziped = seq.zip(with: seq2)

ziped.observeNext { (e) in
    print(e)
}
~~~

> `(1, 2)`
> 
> `(3, 1)`
> 
> `(2, 4)`
> 
> `(5, 7)`