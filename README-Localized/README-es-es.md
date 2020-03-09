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
# Muestra de la conexión de Office 365 con iOS usando el SDK de Microsoft Graph

Microsoft Graph es un punto de conexión unificado para tener acceso a los datos, relaciones e información procedente de la nube de Microsoft. En esta muestra le enseña cómo conectarse, autenticar y llamar a las API de usuario y correo a través del [SDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Nota: Consulte la página del [Portal de registro de la aplicación de Microsoft Graph](https://graph.microsoft.io/en-us/app-registration) que simplifica el registro para poder conseguir que esta muestra se ejecute más rápidamente.
 
## Requisitos previos
* [Xcode](https://developer.apple.com/xcode/downloads/) de Apple: esta muestra es compatible y se ha probado en la versión 8.2.1 de Xcode.
* La instalación de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) como administrador de dependencias.
* Una cuenta de correo electrónico personal o profesional de Microsoft como Office 365, outlook.com, hotmail.com, etc. Puede registrarse para obtener [una suscripción a Office 365 Developer](https://aka.ms/devprogramsignup) que incluye los recursos que necesita para comenzar a crear aplicaciones de Office 365.

     > Nota: Si ya dispone de una suscripción, el vínculo anterior le dirigirá a una página con el mensaje *No se puede agregar a su cuenta actual*. En ese caso, utilice la cuenta de su suscripción actual a Office 365.    
* La Id. de cliente de la aplicación registrada en el [Portal de registro de la aplicación de Microsoft Graph](https://graph.microsoft.io/en-us/app-registration)
* Para realizar solicitudes, se debe proporcionar un **MSAuthenticationProvider** capaz de autenticar solicitudes de HTTPS con un token de portador OAuth 2.0 adecuado. Usaremos [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) para una implementación de muestra de MSAuthenticationProvider con la cual puede usarse para poner en marcha el proyecto. Consulte la sección **código de interés** para más información.

>**Nota:** Esta muestra se probó en la versión 8.2.1 de Xcode. Esta muestra es compatible con Xcode 8 y iOS10, ya que usa el marco Swift 3.0.
       
## Ejecutar esta muestra en Xcode

1. Clone este repositorio.
2. Use CocoaPods para importar el SDK de Microsoft Graph y las dependencias de autenticación:
        
		pod de 'MSGraphSDK'
		pod de 'MSGraphSDK-NXOAuth2Adapter'


 Esta aplicación de muestra contiene un podfile que recibirá los pods en el proyecto. Simplemente vaya al proyecto desde la**terminal** y ejecute: 
        
        instalación de pod
        
   Para más información, consulte **usar CocoaPods** en [recursos adicionales](#AdditionalResources)
  
3. Abra **Graph-iOS-Swift-Connect.xcworkspace**
4. Abra **AutheticationConstants.swift en la carpeta de la aplicación. Verá que el clientID del proceso de registro puede ser agregado a este archivo.

   ```swift
        static let clientId = "ENTER_YOUR_CLIENT_ID"
   ```    
    > Nota: Observará que se han configurado los siguientes ámbitos de permiso para este proyecto: **"https://graph.microsoft.com/Mail.Send", "https://graph.microsoft.com/User.Read", "offline\_access"** Las llamadas al servicio usadas en este proyecto, el envío de un correo a su cuenta de correo y la recuperación de la información de perfil (nombre para mostrar, dirección de correo electrónico) requieren estos permisos para ejecutar la aplicación correctamente.


5. Ejecute el ejemplo. Deberá conectarse a una cuenta de correo personal o profesional, o autenticarlas, y, después, puede enviar un correo a esa cuenta, o a otra cuenta de correo electrónico seleccionada.


## Código de interés

Todos los códigos de autenticación se pueden ver en el archivo **Authentication.swift[](https://github.com/nxtbgthng/OAuth2Client). Implementamos la muestra de MSAuthenticationProvider procedente de [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) para proporcionar compatibilidad de inicio de sesión a aplicaciones nativas registradas, actualización automática de tokens de acceso y funcionalidad de cierre de sesión:

### Autentificar el usuario

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
Una vez se defina el proveedor de autenticación, podemos crear e inicializar el objeto de cliente (MSGraphClient) que se usará para realizar llamadas en el punto de conexión del servicio de Microsoft Graph (correo y usuarios). En **SendViewcontroller.swift** podemos armar una solicitud de correo y enviarla usando el siguiente código:

### Obtener la imagen de perfil de usuario

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
### Cargar la imagen a OneDrive

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

### Adjuntar una imagen a un nuevo mensaje de correo electrónico

```swift
            let fileAttachment = MSGraphFileAttachment()
            let data = UIImageJPEGRepresentation(unwrappedImage, 1.0)
            fileAttachment.contentType = "image/png"
            fileAttachment.oDataType = "#microsoft.graph.fileAttachment"
            fileAttachment.contentBytes = data?.base64EncodedString()
            fileAttachment.name = "me.png"
            message.attachments.append(fileAttachment)

```

### Enviar mensaje

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

Para más información, incluyendo el código para llamar a otros servicios, como OneDrive, vea el [GDK de Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)

## Preguntas y comentarios

Nos encantaría recibir sus comentarios sobre el proyecto Connect de Office 365 para iOS con Microsoft Graph. Puede enviarnos sus preguntas y sugerencias a través de la sección [Problemas]() de este repositorio.

Las preguntas generales sobre desarrollo en Office 365 deben publicarse en [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Asegúrese de que sus preguntas o comentarios se etiquetan con \[Office365] y \[MicrosoftGraph].

## Colaboradores
Deberá firmar un [Contrato de licencia de colaborador](https://cla.microsoft.com/) antes de enviar la solicitud de incorporación de cambios. Para completar el Contrato de licencia de colaborador (CLA), deberá enviar una solicitud mediante un formulario y, después, firmar electrónicamente el CLA cuando reciba el correo electrónico que contiene el vínculo al documento. 

Este proyecto ha adoptado el [código de conducta de código abierto de Microsoft](https://opensource.microsoft.com/codeofconduct/). Para obtener más información, vea [Preguntas frecuentes sobre el código de conducta](https://opensource.microsoft.com/codeofconduct/faq/) o póngase en contacto con [opencode@microsoft.com](mailto:opencode@microsoft.com) si tiene otras preguntas o comentarios.

## Recursos adicionales

* [Centro para desarrolladores de Office](http://dev.office.com/)
* [Página de información general de Microsoft Graph](https://graph.microsoft.io)
* [Usar CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Copyright
Copyright (c) 2016 Microsoft. Todos los derechos reservados.

