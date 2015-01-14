---
layout: post
title: "AFNetworking源码解析(三)"
date: 2014-10-30 15:02:47 +0800
comments: true
categories: 
---
原文地址：<http://www.cocoachina.com/ios/20140916/9632.html>

安全相关的AFSecurityPolicy模块，AFSecurityPolicy用于验证HTTPS请求的证书，先来看看HTTPS的原理和证书相关的几个问题。

HTTPS
HTTPS连接建立过程大致是，客户端和服务端建立一个连接，服务端返回一个证书，客户端里存有各个受信任的证书机构根证书，用这些根证书对服务端 返回的证书进行验证，经验证如果证书是可信任的，就生成一个pre-master  secret，用这个证书的公钥加密后发送给服务端，服务端用私钥解密后得到pre-master secret，再根据某种算法生成master  secret，客户端也同样根据这种算法从pre-master secret生成master secret，随后双方的通信都用这个master  secret对传输数据进行加密解密。

这里说下一开始我比较费解的两个问题：

####1.证书是怎样验证的？怎样保证中间人不能伪造证书？

首先要知道非对称加密算法的特点，非对称加密有一对公钥私钥，用公钥加密的数据只能通过对应的私钥解密，用私钥加密的数据只能通过对应的公钥解密。

我们来看最简单的情况：一个证书颁发机构(CA)，颁发了一个证书A，服务器用这个证书建立https连接。客户端在信任列表里有这个CA机构的根证书。

首先CA机构颁发的证书A里包含有证书内容F，以及证书加密内容F1，加密内容F1就是用这个证书机构的私钥对内容F加密的结果。（这中间还有一次hash算法，略过。）

建立https连接时，服务端返回证书A给客户端，客户端的系统里的CA机构根证书有这个CA机构的公钥，用这个公钥对证书A的加密内容F1解密得 到F2，跟证书A里内容F对比，若相等就通过验证。整个流程大致是：F->CA私钥加密->F1->客户端CA公钥解密->F。 因为中间人不会有CA机构的私钥，客户端无法通过CA公钥解密，所以伪造的证书肯定无法通过验证。

####2.什么是SSL Pinning？

可以理解为证书绑定，是指客户端直接保存服务端的证书，建立https连接时直接对比服务端返回的和客户端保存的两个证书是否一样，一样就表明证书 是真的，不再去系统的信任证书机构里寻找验证。这适用于非浏览器应用，因为浏览器跟很多未知服务端打交道，无法把每个服务端的证书都保存到本地，但CS架 构的像手机APP事先已经知道要进行通信的服务端，可以直接在客户端保存这个服务端的证书用于校验。

为什么直接对比就能保证证书没问题？如果中间人从客户端取出证书，再伪装成服务端跟其他客户端通信，它发送给客户端的这个证书不就能通过验证吗？确 实可以通过验证，但后续的流程走不下去，因为下一步客户端会用证书里的公钥加密，中间人没有这个证书的私钥就解不出内容，也就截获不到数据，这个证书的私 钥只有真正的服务端有，中间人伪造证书主要伪造的是公钥。

为什么要用SSL  Pinning？正常的验证方式不够吗？如果服务端的证书是从受信任的的CA机构颁发的，验证是没问题的，但CA机构颁发证书比较昂贵，小企业或个人用户 可能会选择自己颁发证书，这样就无法通过系统受信任的CA机构列表验证这个证书的真伪了，所以需要SSL Pinning这样的方式去验证。

AFSecurityPolicy
NSURLConnection已经封装了https连接的建立、数据的加密解密功能，我们直接使用NSURLConnection是可以访问 https网站的，但NSURLConnection并没有验证证书是否合法，无法避免中间人攻击。要做到真正安全通讯，需要我们手动去验证服务端返回的 证书，AFSecurityPolicy封装了证书验证的过程，让用户可以轻易使用，除了去系统信任CA机构列表验证，还支持SSL  Pinning方式的验证。使用方法：

	//把服务端证书(需要转换成cer格式)放到APP项目资源里，AFSecurityPolicy会自动寻找根目录下所有cer文件
	AFSecurityPolicy *securityPolicy = [AFSecurityPolicy policyWithPinningMode:AFSSLPinningModePublicKey];
	securityPolicy.allowInvalidCertificates = YES;
	[AFHTTPRequestOperationManager manager].securityPolicy = securityPolicy;
	[manager GET:@"https://example.com/" parameters:nil success:^(AFHTTPRequestOperation *operation, id 	responseObject) {
	} failure:^(AFHTTPRequestOperation *operation, NSError *error) {
	}];
	
AFSecurityPolicy分三种验证模式：

AFSSLPinningModeNone

这个模式表示不做SSL pinning，只跟浏览器一样在系统的信任机构列表里验证服务端返回的证书。若证书是信任机构签发的就会通过，若是自己服务器生成的证书，这里是不会通过的。

AFSSLPinningModeCertificate

这个模式表示用证书绑定方式验证证书，需要客户端保存有服务端的证书拷贝，这里验证分两步，第一步验证证书的域名/有效期等信息，第二步是对比服务端返回的证书跟客户端返回的是否一致。

这里还没弄明白第一步的验证是怎么进行的，代码上跟去系统信任机构列表里验证一样调用了SecTrustEvaluate，只是这里的列表换成了客户端保存的那些证书列表。若要验证这个，是否应该把服务端证书的颁发机构根证书也放到客户端里？

AFSSLPinningModePublicKey

这个模式同样是用证书绑定方式验证，客户端要有服务端的证书拷贝，只是验证时只验证证书里的公钥，不验证证书的有效期等信息。只要公钥是正确的，就能保证通信不会被窃听，因为中间人没有私钥，无法解开通过公钥加密的数据。

整个AFSecurityPolicy就是实现这这几种验证方式，剩下的就是实现细节了，详见源码。

####3.源码注释

AFSecurityPolicy