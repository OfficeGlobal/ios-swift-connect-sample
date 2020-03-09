---
page_type: sample
products:
- office-365
- ms-graph
languages:
- swift
extensions:
  contentType: samples
  technologies:
  - Microsoft Graph
  - Microsoft identity platform
  services:
  - Office 365 
  - Microsoft identity platform
  - Users
  platforms:
  - iOS
  createdDate: 5/12/2016 8:23:10 AM
---
# Пример приложения для iOS, подключающегося к Office 365 и использующего пакет SDK Microsoft Graph

Microsoft Graph — единая конечная точка для доступа к данным, отношениям и аналитике из Microsoft Cloud. В этом примере показано, как подключиться к ней и пройти проверку подлинности, а затем вызывать почтовые и пользовательские API через[пакет SDK Microsoft Graph для iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Примечание. Воспользуйтесь упрощенной регистрацией на [портале регистрации приложений Microsoft Graph](https://graph.microsoft.io/en-us/app-registration), чтобы ускорить запуск этого примера.
 
## Необходимые компоненты
* [Xcode](https://developer.apple.com/xcode/downloads/) от Apple. Этот пример в настоящее время проверяется и поддерживается в Xcode версии 8.2.1.
* Установка [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) в качестве диспетчера зависимостей.
* Рабочая или личная учетная запись Майкрософт, например Office 365, outlook.com или hotmail.com. Вы можете [подписаться на план Office 365 для разработчиков](https://aka.ms/devprogramsignup), который включает ресурсы, необходимые для создания приложений Office 365.

     > Примечание. Если у вас уже есть подписка, при выборе приведенной выше ссылки откроется страница с сообщением *К сожалению, вы не можете добавить это к своей учетной записи*. В этом случае используйте учетную запись, связанную с текущей подпиской на Office 365.    
* Идентификатор клиента из приложения, зарегистрированного на [портале регистрации приложений Microsoft Graph](https://graph.microsoft.io/en-us/app-registration)
* Чтобы отправлять запросы, необходимо указать протокол **MSAuthenticationProvider**, который способен проверять подлинность HTTPS-запросов с помощью соответствующего маркера носителя OAuth 2.0. Для реализации протокола MSAuthenticationProvider и быстрого запуска проекта мы будем использовать [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter). Дополнительные сведения см. в разделе **Полезный код** ниже.

>**Примечание.** Пример прошел проверку в Xcode 8.2.1. Он поддерживает XCode 8 и операционную систему iOS 10, которая использует платформу Swift 3.0.
       
## Запуск этого примера в Xcode

1. Клонируйте этот репозиторий.
2. Используйте CocoaPods, чтобы импортировать пакет SDK Microsoft Graph и зависимости проверки подлинности:
        
		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 Этот пример приложения уже содержит podfile, который добавит компоненты pod в проект. Просто перейдите к проекту из раздела **Терминал** и выполните следующую команду: 
        
        pod install
        
   Для получения дополнительных сведений выберите ссылку **Использование CocoaPods** в разделе [Дополнительные ресурсы](#AdditionalResources).
  
3. Откройте **Graph-iOS-Swift-Connect.xcworkspace**
4. Откройте **AutheticationConstants.swift** в папке "Приложение". Вы увидите, что в этот файл можно добавить **идентификатор клиента**, скопированный в ходе регистрации.

   ```swift
        static let clientId = "ENTER_YOUR_CLIENT_ID"
   ```    
    > Примечание. Вы увидите, что для этого проекта настроены следующие разрешения: **"https://graph.microsoft.com/Mail.Send", "https://graph.microsoft.com/User.Read", "offline\_access"**. Эти разрешения необходимы для правильной работы приложения, в частности отправки сообщения в учетную запись почты и получения сведений профиля (отображаемое имя, адрес электронной почты).


5. Запустите приложение. Вам будет предложено подключить рабочую или личную учетную запись почты и войти в нее, после чего вы сможете отправить сообщение в эту или другую учетную запись.


## Полезный код

Весь код для проверки подлинности можно найти в файле **Authentication.swift**. Мы используем протокол MSAuthenticationProvider из файла [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) для поддержки входа в зарегистрированных собственных приложениях, автоматического обновления токенов доступа и выхода:

### Аутентификация пользователя

```swift
        // Set client ID
        NXOAuth2AuthenticationProvider.setClientId(clientId, scopes: scopes)
        
        // Try silent log in. This will attempt to sign in if there is a previous successful
        // sign in user information.
        if NXOAuth2AuthenticationProvider.sharedAuth().loginSilent() == true {
            completion(nil)
        }
        // Otherwise, present log in controller.
        else {
            NXOAuth2AuthenticationProvider.sharedAuth()
                .login(with: nil) { (error: Error?) in
                    
                    if let nserror = error {
                        completion(MSGraphError.nsErrorType(error: nserror as NSError))
                    }
                    else {
                        completion(nil)
                    }
            }
        }
    ...
    
    func disconnect() {
        NXOAuth2AuthenticationProvider.sharedAuth().logout()
    }

```
Если поставщик проверки подлинности настроен, мы сможем создать и инициализировать объект клиента (MSGraphClient), который будет использоваться для вызова службы Microsoft Graph (почта и пользователи). Мы можем собрать почтовый запрос в **SendViewcontroller.swift** и отправить его с помощью следующего кода:

### Получение аватара пользователя

```swift
        self.graphClient.me().photoValue().download {
            (url: URL?, response: URLResponse?, error: Error?) in
            
                guard let picUrl = url else {
                    return
                }
            
                let picData = NSData(contentsOf: picUrl)
                let picImage = UIImage(data: picData! as Data)
            
                if let validPic = picImage {
                    completion(.success(validPic))
                }
            }

```
### Отправка изображения в OneDrive

```swift
        let data = UIImageJPEGRepresentation(unwrappedImage, 1.0)
        self.graphClient
            .me()
            .drive()
            .root()
            .children()
            .driveItem("me.png")
            .contentRequest()
            .upload(from: data, completion: {
                (driveItem: MSGraphDriveItem?, error: Error?) in
                if let nsError = error {
                    return
                } else {
                    webUrl = (driveItem?.webUrl)!
                }
            })

```

### Добавление вложенного изображения к новому сообщению

```swift
            let fileAttachment = MSGraphFileAttachment()
            let data = UIImageJPEGRepresentation(unwrappedImage, 1.0)
            fileAttachment.contentType = "image/png"
            fileAttachment.oDataType = "#microsoft.graph.fileAttachment"
            fileAttachment.contentBytes = data?.base64EncodedString()
            fileAttachment.name = "me.png"
            message.attachments.append(fileAttachment)

```

### Отправка сообщения

```swift
    let requestBuilder = graphClient.me().sendMail(with: message, saveToSentItems: false)
    let mailRequest = requestBuilder?.request()
            
        mailRequest?.execute(completion: {
            (response: [AnyHashable: Any]?, error: Error?) in
            if let nsError = error {
                print(NSLocalizedString("ERROR", comment: ""), nsError.localizedDescription)
                DispatchQueue.main.async(execute: {
                    self.statusTextView.text = NSLocalizedString("SEND_FAILURE", comment: "")
                })
                    
            }

...            

```

Дополнительные сведения, в том числе код для вызова других служб, например OneDrive, см. в статье [Пакет SDK Microsoft Graph для iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

## Вопросы и комментарии

Мы будем рады получить от вас отзывы о проекте приложения iOS, подключающегося к Office 365 и использующего Microsoft Graph. Отправляйте нам свои вопросы и предложения в раздел этого репозитория, посвященный [проблемам]().

Общие вопросы о разработке решений для Office 365 следует публиковать на сайте [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Обязательно помечайте свои вопросы и комментарии тегами \[Office365] и \[MicrosoftGraph].

## Участие
Прежде чем отправить запрос на включение внесенных изменений, необходимо подписать [лицензионное соглашение с участником](https://cla.microsoft.com/). Чтобы заполнить лицензионное соглашение с участником (CLA), вам потребуется отправить запрос с помощью формы, а затем после получения электронного сообщения со ссылкой на этот документ подписать CLA с помощью электронной подписи. 

Этот проект соответствует [Правилам поведения разработчиков открытого кода Майкрософт](https://opensource.microsoft.com/codeofconduct/). Дополнительные сведения см. в разделе [вопросов и ответов о правилах поведения](https://opensource.microsoft.com/codeofconduct/faq/). Если у вас возникли вопросы или замечания, напишите нам по адресу [opencode@microsoft.com](mailto:opencode@microsoft.com).

## Дополнительные ресурсы

* [Центр разработки для Office](http://dev.office.com/)
* [Страница с общими сведениями о Microsoft Graph](https://graph.microsoft.io)
* [Использование CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Авторские права
(c) Корпорация Майкрософт (Microsoft Corporation), 2016. Все права защищены.

