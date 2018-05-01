---
title: 浅谈iOS签名机制
date: 2017-01-23 23:33:40
tags: [iOS技巧]
---

> “Users appreciate code signing.” –[《Apple Developer Library:About Code Signing》](https://developer.apple.com/library/content/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)

*P.S. 苹果已经无耻地在最新的文档里删掉了这行描述，想来苹果估计也发现了code signing并没有被users appreciate，对于普通用户而言，根本不需要关注code signing，而对于开发者而言，code signing简直就是噩梦~！*

下面我粗略谈一谈对于code signing的一些浅显的认识，其中很多部分是源于[《Apple Developer Library:About Code Signing》](https://developer.apple.com/library/content/documentation/Security/Conceptual/CodeSigningGuide/Introduction/Introduction.html)

#### 补充知识

因为Code Signing机制涉及到加密技术，下面简略说明一下常用的加密技术，如果已经非常了解相关知识，请略过这部分内容。

**对称加密和非对称加密：**顾名思义，对称加密就是用来加密和解密的密钥是同一个，平常在项目里常用到的AES就是典型的对称加密。非对称加密指的就是加密和解密的密钥是不用的，有一把公钥，一把私钥，如果使用公钥加密，就得用私钥进行解密，而使用私钥加密就得用公钥进行解密，如常用的RSA就是非对称加密。

优缺点：对称加密使用相同的密钥进行加解密，所以效率非常高，但是由于解密一方也必须知道密钥，那么密钥势必就得通过某种途径进行传输，这就造成了密钥不安全的情况。非对称加密正好和对称加密相反，它的特点就是，加解密效率较低，但是安全性高。所以通常的做法是将两者结合使用，即使用对称加密的方式对正文进行加密，使用非对称加密的方式对密钥进行加密。

**数字签名:** 数字签名技术类似现实生活中的印章和签名，主要的作用是保证内容没有被修改，以及签名方无法进行抵赖。数字签名技术简单来讲就是发送方采用特定的算法产生正文的摘要，然后使用发送方的私钥进行加密，接收方收到数字签名后，使用发送方的公钥进行解密，然后比对正文摘要，如果相同，表示该数字签名是合法的，否则就是有问题的。

**数字证书: **数字证书是由权威机构－－CA证书授权（Certificate Authority）中心发行的，能提供在Internet上进行身份验证的一种权威性电子文档，人们可以在互联网交往中用它来证明自己的身份和识别对方的身份。

#### Code Signing的好处

书归正题，下面我们开始讲讲Code Signing的好处。

* 保证代码不会被篡改

* 明确代码的来源，Code Signing可以描述一个App是由谁进行签名的：Apple或者特定的开发者，或者其他一些组织和机构。

* 决定代码是否可以继续访问特定的数据，因为iOS的隐私管理机制，App在第一次访问如相册，通讯录等数据时会询问用户是否允许，一旦允许后后续再对这些数据进行访问的时候就不再进行询问，但是如果App的版本进行了更新，对于系统而言，一个新的版本其实就是一个新的App，系统就是通过Code Signing来判断这个版本是否可以继续访问这些隐私数据。

#### Code Signing不能保证的

* 无法保证经过签名的代码是没有安全问题的。

* 无法保证App是否会在运行过程中加载不安全，没经过签名的代码。

* 无法提供数字产权保护(DRM)和copy protection technology

#### iOS Code Signing工作原理

iOS的签名设置简直就是一项不能更加坑爹的工作了，当你潇潇洒洒写完一大段代码后，因为签名设置的问题被Xcode各种报错的时候，心中必然会有一万只草泥马奔过。
下面大致来聊一聊iOS整个签名过程。

#### 组成

* CertificateSigningRequest.certSigningRequest，这个文件是在Certificate Assistant中生成的，本质上这个文件就是一个申请书，里面描述了申请人的信息，申请人的公钥，还有摘要文件和公钥的加密算法。一般情况下开发者是不需要关心这个文件的（除非你还是你们公司ADP的管理人员）可以使用`openssl asn1parse -i -in CertificateSigningRequest.certSigningRequest`查看CSR文件。

* Certificate文件，使用上面的CSR能够在Apple Developer Member Center中生成对应的certificate文件，可以使用`openssl x509 -inform der -in ios_development.cer -noout -text`对生成的.cer文件进行查看。双击这个.cer文件之后就会在Keychain中添加一个certificate文件，如果这时候你有刚才生成CSR的私钥，那么你的certificate就可以使用了，否则的话就需要导入对应的私钥。

* mobileprovision文件，严格来说mobileprovision文件已经不属于签名的范畴了，它其实是一个描述文件，用来描述app所使用的某些服务是被苹果认可的，比如APN推送服务以及限制app的装机规模。如果没有这个玩意，每个app开发者就可以自己签app然后放到自己的网站上供用户下载，谁还提交到AppStore，审批时间又长，还可能各种被拒。使用`security cms -D -i`命令可以查看一个mobileprovision文件的内容，这个文件主
要描述了下面这些内容：

* 被描述的App的信息，AppIDName， App Bundle ID等。

* 使用哪些Certificate，所以如果你有一个certificate,有一个和certificate匹配的私钥，也有一个mobileprovision文件，但是你的mobileprovision文件和certificate是不匹配的，那么还是无法真机调试。
功能授权列表，用来描述这个App能够使用哪些功能，如APN，IAP等。
这个描述文件允许的Device UDID列表，只有在这个列表中的设备，才可以安装这个App。至于企业签名的描述文件，则没有ProvisionedDevices这个字段，取而代之的是一个叫ProvisionsAllDevices的字段，这个字段值为true。当然了，这个文件本身是被苹果做过签名的，所以什么自己在ProvisionedDevices下面加一个Device的UDID事情就不要多想了。

* Certificate的公钥，关于这点，貌似并没有在命令的输出中体现出来，所以不要问我是怎么知道的，因为我是猜的！既然Xcode使用私钥对App进行签名，那么就一定要有一个机制将对应的公钥放到App中，这样App在运行之前才可以用公钥来对数字签名进行解密，从而认证App的合法性。而每个App编译完成后在它的content下一定有一个embedded.mobileprovision文件。而上面提到过certificate文件是有我的公钥信息的，而mobileprovision文件是一定要通过certificate文件才能生成的，我实在想不到有什么理由苹果不将公钥放到mobileprovision文件中。

* 过期时间等其他信息。

* _CodeSignature文件夹，在每个经过签名的iOS app文件下都有一个叫_CodeSignature的文件夹，这个文件夹下有一个叫CodeResources的文本文件。其实这个文件本质上就是一个plist，其中描述的就是每个资源文件的签名信息。至于程序本身的二进制文件，签名信息应该是直接写入到这个二进制文件中的。这个文件的存在就保证了app在被签名之后，其中的任何资源发生了修改，在进行签名验证的时候都会出现问题，而导致验证不过。可以使用codesign --verify Sample.app来对app的签名进行验证，如果这个命令没有输出任何内容，那么签名验证就是通过的。

* entitlement文件，当你在程序中打开如APN，Associated Domains，App Groups等功能时，Xcode会帮你自动生成一个entitlement文件，而这个文件也将会作为codesign的参数传入，如果你的entitlement文件和mobileprovision文件中描述的不一致，也会造成app安装后的校验不通过，导致app无法被正常安装。

#### 过程

1. 在程序完成编译后，Xcode会使用codesign，通过certificate文件下的私钥对app进行签名。可以使用codesign -vv -d Example.app查看app的签名信息。
2. 校验BundleID等信息
3. 使用embedded.mobileprovision下的公钥来进行程序是否被篡改的校验。
4. 使用embedded.mobileprovision和entitlement文件校验功能授权列表。

*上面的过程中的顺序不一定是真正校验的顺序，上面列举的校验步骤也不一定是完整的校验步骤，一切都是我个人的猜测。*
