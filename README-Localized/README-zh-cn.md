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
# 适用于 iOS 的 Office 365 连接示例（使用 Microsoft Graph SDK）

Microsoft Graph 是访问来自 Microsoft 云的数据、关系和见解的统一终结点。此示例介绍如何连接并对其进行身份验证，然后通过[适用于 iOS 的 Microsoft Graph SDK](https://github.com/microsoftgraph/msgraph-sdk-ios) 调用邮件和用户 API。

> 注意：尝试 [Microsoft Graph 应用注册门户](https://graph.microsoft.io/en-us/app-registration)页，该页简化了注册，因此你可以更快地运行该示例。
 
## 先决条件
* 来自 Apple 的 [Xcode](https://developer.apple.com/xcode/downloads/) \- 当前在 Xcode 的8.2.1 版本中对此示例进行了测试和支持。
* 安装 [CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html) 成为依存关系管理器。
* Microsoft 工作或个人电子邮件帐户，例如 Office 365 或 outlook.com、hotmail.com 等。你可以注册 [Office 365 开发人员订阅](https://aka.ms/devprogramsignup)，其中包含你开始构建 Office 365 应用所需的资源。

     > 注意：如果您已经订阅，之前的链接会将您转至包含以下信息的页面：*抱歉，您无法将其添加到当前帐户*。在这种情况下，请使用当前 Office 365 订阅中的帐户。    
* [Microsoft Graph 应用注册门户](https://graph.microsoft.io/en-us/app-registration)已注册应用的客户端 ID
* 若要生成请求，必须提供 **MSAuthenticationProvider**（它能够使用适当的 OAuth 2.0 持有者令牌对 HTTPS 请求进行身份验证）。我们将使用 [msgraph-sdk-ios-nxoauth2-adapter](https://github.com/microsoftgraph/msgraph-sdk-ios-nxoauth2-adapter) 作为 MSAuthenticationProvider 的示例实现，它可用于快速启动你的项目。有关详细信息，请参阅下面的“**相关代码**”部分。

>**注意：**已对 Xcode 8.2.1 测试此示例。此示例支持使用 Swift 3.0 框架的 Xcode 8 和 iOS10。
       
## 在 Xcode 中运行此示例

1. 克隆该存储库
2. 使用 CocoaPods 导入 Microsoft Graph SDK 和身份验证依赖项：
        
		pod 'MSGraphSDK'
		pod "MSGraphSDK-NXOAuth2Adapter"


 该示例应用已包含可将 pod 导入项目中的 Podfile。只需从**终端**中导航到该项目并运行： 
        
        pod install
        
   更多详细信息，请参阅[其他资源](#AdditionalResources)中的**使用 CocoaPods**
  
3. 打开 **Graph-iOS-Swift-Connect.xcworkspace**
4. 打开应用程序文件夹下的 **AutheticationConstants.swift**。你将看到可将注册过程中的 **clientId** 添加到此文件中。

   ```swift
        static let clientId = "ENTER_YOUR_CLIENT_ID"
   ```    
    > 注意：你会注意到，已为此项目配置以下权限范围：**"https://graph.microsoft.com/Mail.Send", "https://graph.microsoft.com/User.Read", "offline\_access"**。该项目中所使用的服务调用，向你的邮件帐户发送邮件并检索一些个人资料信息（显示名称、电子邮件地址）需要这些应用的权限以正常运行。


5. 运行示例。系统将要求你连接至工作或个人邮件帐户或对其进行身份验证，然后你可以向该帐户或其他所选电子邮件帐户发送邮件。


## 相关代码

可在 **Authentication.swift** 文件中查看所有身份验证代码。我们使用从 [NXOAuth2Client](https://github.com/nxtbgthng/OAuth2Client) 扩展的 MSAuthenticationProvider 示例实现来提供对已注册的本机应用的登录支持、访问令牌的自动刷新和注销功能：

### 对用户进行身份验证

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
设置身份验证提供程序后，可创建和初始化客户端对象 (MSGraphClient)，该对象将用于对 Microsoft Graph 服务终结点（邮件和用户）进行调用。在 **SendViewcontroller.swift** 中，可以使用以下代码汇编邮件请求并发送：

### 获取用户个人资料图片

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
### 将图片上传到 OneDrive

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

### 将图片附加到新电子邮件

```swift
            let fileAttachment = MSGraphFileAttachment()
            let data = UIImageJPEGRepresentation(unwrappedImage, 1.0)
            fileAttachment.contentType = "image/png"
            fileAttachment.oDataType = "#microsoft.graph.fileAttachment"
            fileAttachment.contentBytes = data?.base64EncodedString()
            fileAttachment.name = "me.png"
            message.attachments.append(fileAttachment)

```

### 发送邮件

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

有关详细信息（包括用于调用 OneDrive 等其他服务的代码），请参阅[适用于 iOS 的 Microsoft Graph SDK](https://github.com/microsoftgraph/msgraph-sdk-ios)

## 问题和意见

我们乐意倾听您有关 Office 365 iOS Microsoft Graph Connect 项目的反馈。您可以在该存储库中的[问题]()部分将问题和建议发送给我们。

与 Office 365 开发相关的问题一般应发布在[堆栈溢出](http://stackoverflow.com/questions/tagged/Office365+API)中。确保您的问题或意见使用了 \[Office365] 和 \[MicrosoftGraph] 标记。

## 贡献
您需要在提交拉取请求之前签署[参与者许可协议](https://cla.microsoft.com/)。要完成参与者许可协议 (CLA)，你需要通过表格提交请求，并在收到包含文件链接的电子邮件时在 CLA 上提交电子签名。 

此项目已采用 [Microsoft 开放源代码行为准则](https://opensource.microsoft.com/codeofconduct/)。有关详细信息，请参阅[行为准则常见问题解答](https://opensource.microsoft.com/codeofconduct/faq/)。如有其他任何问题或意见，也可联系 [opencode@microsoft.com](mailto:opencode@microsoft.com)。

## 其他资源

* [Office 开发人员中心](http://dev.office.com/)
* [Microsoft Graph 概述页](https://graph.microsoft.io)
* [使用 CocoaPods](https://guides.cocoapods.org/using/using-cocoapods.html)

## 版权信息
版权所有 (c) 2016 Microsoft。保留所有权利。

