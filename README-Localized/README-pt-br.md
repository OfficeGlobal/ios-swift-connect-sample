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
# Exemplo de Conexão com o Office 365 para iOS Usando o SDK do Microsoft Graph

O Microsoft Graph é um ponto de extremidade unificado para acessar dados, relações e ideias que vêm do Microsoft Cloud. Este exemplo mostra como realizar a conexão e a autenticação no Microsoft Graph e, em seguida, chamar APIs de mala direta e usuário por meio do [SDK do Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios).

> Observação: Experimente a página [Portal de Registro de Aplicativos do Microsoft Graph](https://graph.microsoft.io/en-us/app-registration) que simplifica o registro para que você possa executar este exemplo com mais rapidez.
 
## Pré-requisitos
* [Xcode](https://developer.apple.com/xcode/downloads/) da Apple – Atualmente, este exemplo foi testado e é compatível na versão 8.2.1 do Xcode.
* A instalação de [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) como um gerenciador de dependências.
* Uma conta de email comercial ou pessoal da Microsoft como o Office 365, ou outlook.com, hotmail.com, etc. Inscreva-se para uma [Assinatura de Desenvolvedor do Office 365](https://aka.ms/devprogramsignup), que inclui os recursos necessários para começar a criação de aplicativos do Office 365.

     > Observação: Caso já tenha uma assinatura, o link anterior direcionará você para uma página com a mensagem *Não é possível adicioná-la à sua conta atual*. Nesse caso, use uma conta de sua assinatura atual do Office 365.    
* Uma ID de cliente do aplicativo registrado no [Portal de Registro de Aplicativos do Microsoft Graph](https://graph.microsoft.io/en-us/app-registration)
* Para realizar solicitações de autenticação, é necessário fornecer um **MSAuthenticationProvider** para autenticar solicitações HTTPS com um token de portador OAuth 2.0 apropriado. Usaremos [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) para uma implementação de exemplo de MSAuthenticationProvider que pode ser usado para iniciar rapidamente o projeto. Confira a seção **Código de Interesse** a seguir para obter mais informações.

>**Observação:** O exemplo foi testado no Xcode 8.2.1. Este exemplo não é compatível com o XCode 8 e o iOS10, que usam a estrutura Swift 3.0.
       
## Executando este exemplo em Xcode

1. Clonar este repositório
2. Use o CocoaPods para importar as dependências de autenticação e o SDK do Microsoft Graph:
        
		pod 'MSGraphSDK'
		pod 'MSGraphSDK-NXOAuth2Adapter'


 Este aplicativo de exemplo já contém um podfile que colocará os pods no projeto. Basta navegar até o projeto a partir do **Terminal** e executar: 
        
        pod install
        
   Para saber mais, confira o artigo **Usando o CocoaPods** em [Recursos Adicionais](#AdditionalResources)
  
3. Abrir **Graph-iOS-Swift-Connect.xcworkspace**
4. Abra **AutheticationConstants.swift** na pasta Aplicativo. Observe que você pode adicionar o valor de **clientId** ao arquivo do processo de registro.

   ```swift
        static let clientId = "ENTER_YOUR_CLIENT_ID"
   ```    
    > Observação: Você notará que foram configurados os seguintes escopos de permissão para esse projeto: **"https://graph.microsoft.com/Mail.Send", "https://graph.microsoft.com/User.Read", "offline\_access"**. As chamadas de serviço usadas neste projeto, ao enviar um email para sua conta de email e ao recuperar algumas informações de perfil (Nome de Exibição, Endereço de Email), exigem essas permissões para que o aplicativo seja executado corretamente.


5. Execute o exemplo. Você será solicitado a conectar/autenticar uma conta de email comercial ou pessoal e, em seguida, poderá enviar um email a essa conta ou a outra conta de email selecionada.


## Código de Interesse

Todo código de autenticação pode ser visualizado no arquivo **Authentication.swift**. Usamos um exemplo de implementação do MSAuthenticationProvider estendida do [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) para oferecer suporte a logon de aplicativos nativos registrados, atualizações automáticas de tokens de acesso e funcionalidade de logout:

### Autenticar o usuário

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
Depois que o provedor de autenticação estiver definido, podemos criar e inicializar um objeto de cliente (MSGraphClient) que será usado para fazer chamadas no ponto de extremidade do serviço do Microsoft Graph (email e usuários). Em **SendViewcontroller.swift**, podemos montar uma solicitação de email e enviá-la usando o seguinte código:

### Obter foto do perfil de usuário

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
### Carregar imagem para o OneDrive

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

### Anexar a imagem a uma nova mensagem de email

```swift
            let fileAttachment = MSGraphFileAttachment()
            let data = UIImageJPEGRepresentation(unwrappedImage, 1.0)
            fileAttachment.contentType = "image/png"
            fileAttachment.oDataType = "#microsoft.graph.fileAttachment"
            fileAttachment.contentBytes = data?.base64EncodedString()
            fileAttachment.name = "me.png"
            message.attachments.append(fileAttachment)

```

### Enviar a mensagem

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

Para obter mais informações, incluindo código para chamar outros serviços, como o OneDrive, confira ao [SDK do Microsoft Graph para iOS](https://github.com/microsoftgraph/msgraph-sdk-ios)

## Perguntas e comentários

Gostaríamos de saber sua opinião sobre o projeto de conexão com o Office 365 para iOS usando o Microsoft Graph. Você pode enviar perguntas e sugestões na seção [Problemas]() deste repositório.

Faça a postagem de perguntas sobre desenvolvimento do Office 365 em geral na página do [Stack Overflow](http://stackoverflow.com/questions/tagged/Office365+API). Não deixe de marcar as perguntas ou comentários com \[Office365] e \[MicrosoftGraph].

## Colaboração
Assine o [Contributor License Agreement (Contrato de Licença de Colaborador)](https://cla.microsoft.com/) antes de enviar a solicitação pull. Para concluir o CLA (Contrato de Licença do Colaborador), você deve enviar uma solicitação através do formulário e assinar eletronicamente o CLA quando receber o email com o link para o documento. 

Este projeto adotou o [Código de Conduta de Código Aberto da Microsoft](https://opensource.microsoft.com/codeofconduct/).  Para saber mais, confira as [Perguntas frequentes sobre o Código de Conduta](https://opensource.microsoft.com/codeofconduct/faq/) ou entre em contato pelo [opencode@microsoft.com](mailto:opencode@microsoft.com) se tiver outras dúvidas ou comentários.

## Recursos adicionais

* [Centro de Desenvolvimento do Office](http://dev.office.com/)
* [Página de visão geral do Microsoft Graph](https://graph.microsoft.io)
* [Usando o CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## Direitos autorais
Copyright © 2016 Microsoft. Todos os direitos reservados.

