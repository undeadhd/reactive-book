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
...

### MagicsInteractor

Интеракторы тесно взаимодействуют с api, дальше мы узнаем как именно. Интерактор позволяет определить `relativeUrl`.
 
~~~swift
var relativeURL: String = "/registration"
~~~

А также ряд методов, для осуществления запроса и его обработки.

#### Метод modify

Позволяет указать тело и хедеры запроса.

~~~swift
//устанавливаем хедеры + тело запроса
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

#### Методы process

~~~swift
// Вызывается, если запрос выполнился успешно и есть ответ ввиде JSON'а
func process(json: MagicsJSON, response: URLResponse?, api: MagicsAPI)
~~~

~~~swift
func process(json: MagicsJSON, response: URLResponse?, api: MagicsAPI) {
	//действия с json 
	print(json["result"])
}
~~~

Другой метод вызывается в случае неуспешного выполнения запроса.

~~~swift
func process(error: Error, response: URLResponse?)
~~~


~~~swift
func process(error: Error, response: URLResponse?) {
	print("Error -> \(error)")
}
~~~

#### Метод completedWith

~~~swift
func completedWith(json: MagicsJSON?, response: URLResponse?)
~~~

Метод выполняется в случае успешного выполнения запроса.

#### Метод hasErrorFor

~~~swift
func hasErrorFor(json: MagicsJSON?, response: URLResponse?, error: Error?) -> Error?
~~~

Выполнится в случае неуспешного выполнения запроса, также возвращает ошибку.

#### Пример интерактора

~~~swift
class RegistrationInterractor: MagicsInteractor {
    
    var relativeURL: String = "/registration"
    
    var method: MagicsMethod { return .post }
    
    let emailField: String
    let passwordField: String
    var registrationSuccessed: Bool = false
    
    init(email: String, password: String) {
        self.emailField = email
        self.passwordField = password
    }
   	 
   	 //модифицируем запрос
    func modify(request: URLRequest) -> URLRequest {
        var newRequest = request
        
        let params = [
            "password" : passwordField,
            "email" : emailField,
        ]
        
        newRequest.httpBody = formData(params: params).data(using: .utf8)
        
        return newRequest
    }
    
    //конвертируем данные в form data
    func formData<T>(params: [String : T]) -> String {
        var data = [String]()
        for(key, value) in params {
            data.append(key + "=\(value)")
        }
        let formData = data.map { String($0) }.joined(separator: "&")
        return formData
    }
    
    //в случае успешного выполнения запроса выполнится
    func process(json: MagicsJSON, response: URLResponse?, api: MagicsAPI) {
        
        print("result -> \(json["result"]) message -> \(json["message"])")
        
        if json["result"].string! == "success" {
            
            registrationSuccessed = true
            
            let keychain = KeychainSwift()
            keychain.set(json["token"].string, forKey: "token")
            
        } else {
            registrationSuccessed = false
        }
        print("\(json)")
    }
}
~~~

Совмещаем с MagicsApi

~~~swift
class RegistrationApi: MagicsApi {
	override var baseUrl: String { return "наша_ссылка" }
~~~

Пример выполнения запроса.

~~~swift
let api = RegistrationApi()
let interactor = RegistrationInteractor(email: a, password: b)

api.interact(interactor, completion: { (interac, error) in {
	print("запрос завершился")
}
~~~

Мы создаем экземпляр нашего `MagicsApi`, создаем экземпляр  `Interactor`, а дальше вызываем метод `ineract` (бывает 2 видом, с и без completion-функции), после этого MAgics все сделает за нас.