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
override func modify<T>(request: URLRequest, interactor: T) -> URLRequest where T : MagicsInteractor {
	var newRequest = request
	newRequest.setJSONBody(with: [
		"user" : "user1",
		"password" : "123456"
	])
	newRequest.setValue("token", forHTTPHeaderField: "X-Auth-Token")
	return newRequest
}
~~~

Метод 

~~~swift
//вызывается, если запрос прошел неуспешно
open func hasErrorFor<T: MagicsInteractor>(json: MagicsJSON?, response: URLResponse?, error: Error?, interactor: T) -> Error?{ return nil }
~~~

Пример реализации

~~~swift
override func hasErrorFor<T>(json: MagicsJSON?, response: URLResponse?, error: Error?, interactor: T) -> Error? where T : MagicsInteractor {
        guard let json = json else {
            return ResponseError(errorType: .noJSON)
        }
        
        let status = ResponseStatus(rawValue: json["status"].string ?? "") ?? .error
        
        print(status)
        
        let error = ResponseError(
            description: json["first_error"]?.string,
            stringKey: json["error_key"]?.string)
        
        print(error?.description)
        print(error?.stringKey)
        
		print(json)
        return error
}    
~~~