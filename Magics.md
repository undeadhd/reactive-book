# Magics

Magics - библиотека для работы с API.

Библиотека очень простая в использовании и состоит из:

1. `Api`
2. `Interactor`
3. `Model`

Это те модули, которые мы будем использовать. 

Начнем с модуля `Api`.

### MagicsApi

MagicsApi обязавыет нас установить 1 значение, а именно `baseUrl`.

~~~swift
class Api: MagicsAPI {
    override var baseURL: String { return "url" }
}
~~~

Таким образом мы указываем базовый url.

`MagicsApi` предлагает еще несколько методов для опциональной реализации, а именно.

~~~swift
//Метод для редактирования запроса, установки хедеров и body
open func modify<T: MagicsInteractor>(request: URLRequest, interactor: T) -> URLRequest { return request }
~~~

Пример реализации метода:

~~~swift
func modify(request: URLRequest) -> URLRequest {
	var newRequest = request
	newRequest.setJSONBody(with: [
		"user" : "user1",
		"password" : "123456"
	])
	newRequest.setValue("token", forHTTPHeaderField: "X-Auth-Token")
	return newRequest
}
~~~

