# Bond Framework

## Содержание
[TableView](#TableView)

## Введение
`Bond` это binding framework, написанный на `Swift`. С помощью этого фреймворка мы можем связывать на UI-элементы с реактивными частями кода.

## Bind'ing
Для того, чтобы установить связь между 2-мя объектами, нам понадобиться всего 1 строчка кода.
Например мы хотим, чтобы вводя текст в `textField` он сразу же отображался в нашем `label`, стандартный пример, посмотрим как это выглядит в `bond`.

~~~swift
textField.reactive.text.bind(to: label.reactive.text)
~~~

Такой binding можно проводить с любыми элементами и любыми свойствами этих элементов, но не все элементы имеют расширения, поэтому `bond` позволяет удобно и легко нам их создавать с помощью **`Reactive Extension`**, об этом чуть позже.

теперь мы хотим создать кнопку и выполнять какую-то функцию, после ее нажатия.

~~~swift
button.reactive.tap.observeNext {
    // наша функция
}
~~~

С тем как делать простые привязки, вопросов возникнуть не должно, поэтому перейдем к интересному.

## TableView

`TableView` как и `CollectionView` можно привязывать к массивам, изменяющимся массивам и многому другому. Начнем с простого, привязка к `ObservableArray`.

Создадим таблицу и поместим в нее наш массив данных.

~~~swift
let array = ObservableArray(["Первый", "Второй"])

array.bind(to: tableView) { item, index, tableView in
	let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: index)
    cell.textLabel?.text = item[index.row]
    return cell
}
~~~

Вот так просто мы привязали массив к таблице. Теперь, мы хотим, чтобы массив мог изменяться по нажатию на кнопку (добавлять элемент). Для этого создадим кнопку и переделаем наш код, чтобы он выглядел следжующим образом.

~~~swift
let mutableArray = MutableObservableArray(["первый", "второй", "третий"])

mutableArray.bind(to: tableView) { item, index, tableView in
	let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: index)
	cell.textLabel?.text = item[index.row]
	return cell
}

button.reactive.tap.observeNext {
	mutableArray.append("элемент №\(mutableArray.count + 1)")
}
~~~

Теперь при нажатии на кнопку, в таблицу будет добавляться новый элемент, перезагрузка таблицы происходит автоматически, после изменения массива.

Следующим шагом, попробуем создать таблицу с `header'ом` и `footer'ом`.

Для этого нам нужно создать структуру, которая наследуется от `TableViewBond` и реализовать один метод, остальные методы являются по умолчанию.

~~~swift
typealias SectionMetadata = (header: String, footer: String)

struct CustomTableBond: TableViewBond {
    
    //обязательный метод
    func cellForRow(at indexPath: IndexPath, tableView: UITableView, dataSource: Observable2DArray<SectionMetadata, String>) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath)
        cell.textLabel?.text = dataSource[indexPath]
        return cell
    }
    
    // optional
    func titleForHeader(in section: Int, dataSource: Observable2DArray<SectionMetadata, String>) -> String? {
        return dataSource[section].metadata.header
    }
    
    // optional    
//    func titleForFooter(in section: Int, dataSource: Observable2DArray<SectionMetadata, String>) -> String? {
//        return dataSource[section].metadata.footer
//    }
}
~~~

Создадим массив 2D. 

~~~swift
let cities = Observable2DArraySection<SectionMetadata, String>(
            metadata: (header: "Cities", footer: "That's it"),
            items: ["Paris", "Berlin"]
        )
~~~

Поместим этот массив в `MutableObservableArray`.

~~~swift
let array = MutableObservable2DArray([cities])
~~~

И осталось привязать массив `array` к таблице.

~~~swift
array.bind(to: tableView, using: CustomTableBond())
~~~

Обратите внимание, что используется другой метод привязки, с указанием еще одного параметра.

Кнопка будет добавлять в секцию новое значение.

~~~swift
addItemButton.reactive.tap.observeNext {
    array.appendItem("Moscow", toSection: 0)
}
~~~

Для того, чтобы создать новую секцию, мы должны создать еще один массив.

~~~swift
addItemButton.reactive.tap.observeNext {
	let countries = Observable2DArraySection<SectionMetadata, String>(metadata: ("Countries", "No more..."), items: ["France", "Croatia"])
	array.appendSection(countries)
}
~~~

Работать с таблицами очень легко с помощью bond. Работа с `CollectionView` никак не отличается от работы с `TableView`.

## Reactive Extension

В случае, если нам необходимо создать свое рекативное свойство, мы можем использовать `ReactiveExtension`. Для этого нам нужно создать следующего вида код.

Например мы хотим реактивно отключать возможность скролить у пользователя, или же наоборот давать ему такую возможность, по умолчанию такой возможности нет в `bond`, но стоит написать пару строчек кода и она появится.

~~~swift
extension ReactiveExtensions where Base: UIScrollView {
    var isScrollEnabled: Bond<Bool> {
        return bond { $0.isScrollEnabled = $1 }
    }
}
~~~

Теперь мы можем использовать это в коде, уже знакомым нам `binding'ом`.

~~~swift
*boolVariable*.bind(to: scrollView.reactive.isScrollEnabled)
~~~

Или же нам хочется, чтобы наш `webView` автоматически грузил нам страницы, как только ссылка поменялась, для этого напишем следующее расширение.

~~~swift
extension ReactiveExtensions where Base: UIWebView {
    var loadRequest: Bond<URL> {
        return bond { $0.loadRequest(URLRequest(url: $1))}
    }
}
~~~
И после этого мы также сможем использовать это при обычном `binding'е`.