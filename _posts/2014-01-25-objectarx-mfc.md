---
title: ObjectArx ile MFC Kullanımı
permalink: /acad/objectarx-mfc/
published: true
classes: wide
categories:
  - ObjectARX
tags:
  - MFC
---

MFC hem Windows hem de ObjectARX uygulamaları için arayüzler hazırlayabileceğiniz zengin içerikli bir kütüphanedir. Bu yazı da ObjectARX ile MFC kullanırken dikkat etmeniz gerekenler ve ObjectARX’in yerleşik MFC desteğiyle ilgili.

MFC kütüphaneleri uygulamalara statik ya da dinamik olarak bağlanabilmektedir. 

Ancak AutoCAD kütüphanelerinin MFC ile dinamik bağlı olması nedeniyle, ObjectARX de sadece dinamik bağlı MFC DLL uygulamalarını desteklemektedir. Bu yüzden MFC’yi ObjectARX ile birlikte kullanmak istiyorsak ObjectArx Visual Studio projemizi oluştururken <strong>Şekil-1</strong>‘de gösterildiği gibi “Use MFC in a Shared DLL” seçeneğini seçmeli ve MFC ile dinamik bağlantı kurmalıyız.

<img src="{{ "/assets/images/UsingMfc-1.png" | absolute_url }}"  alt="Şekil-1" style="width:100%">
<figure>
  <figcaption>Şekil-1</figcaption>
</figure>

MFC kütüphaneleri ile çalışırken karşımıza çıkan en önemli kavramlardan biri kaynakların (Resources) kullanımıdır. Kaynaklar kullanacağımız arayüz nesnelerini (diyalog kutuları, liste kutuları vb.) içerir. Her dinamik bağlı kütüphane (dll) sadece kendisinin kullanacağı ya da başka dll’ler ile paylaşacağı bir kaynak paketine sahiptir. Dolayısıyla AutoCAD ve ObjectARX uygulamalarının kaynakları birbirlerinden farklıdır. Bu nedenle ObjectARX uygulamasının hangi kaynağı kullanılacağına dair karışıklık uygulama kaynakları arasında geçiş yapılarak giderilmelidir. Örneğin ObjectARX uygulamasına ait kaynağına geçilerek uygulama kullanılmalı ve bu kaynağa ihtiyaç kalmadığında ise AutoCAD kaynağına geri dönülmelidir.

ObjectARX SDK’sı kaynaklar arası geçişi, kaynak yönetimini kolaylaştırmak için iki basit C++ sınıfı sağlar. Bunlar <strong>CAcExtensionModule</strong> ve <strong>CAcModuleResourceOverride</strong> sınıflarıdır.

<strong>CAcExtensionModule</strong>, “MFC Extension DLL”sini uyandırıp sonlandırmakta kullanılan AFX_EXTENSION_MODULE yapısını sağlar ve ObjectARX modülü ile varsayılan AutoCAD kaynak sağlayıcılarını izler.

İki kaynak arasındaki geçişi sağlayabilmek için, <strong>DllMain(…)</strong> işlev içinde <strong>CAcExtensionModule</strong> sınıfının bir örneğini aşağıdaki gibi oluşturmanız yeterlidir.

```c++
 AC_IMPLEMENT_EXTENSION_MODULE(ObjectArxMfcDLL)  
 extern "C"  
 BOOL WINAPI DllMain (HINSTANCE hInstance, DWORD dwReason, LPVOID lpReserved) {  
      //- Remove this if you use lpReserved  
      UNREFERENCED_PARAMETER(lpReserved) ;  
      if ( dwReason == DLL_PROCESS_ATTACH ) {  
     _hdllInstance =hInstance ;  
           ObjectArxMfcDLL.AttachInstance (hInstance) ;  
           InitAcUiDLL () ;  
      } else if ( dwReason == DLL_PROCESS_DETACH ) {  
           ObjectArxMfcDLL.DetachInstance () ;  
      }  
      return (TRUE) ;  
 }  
```

İkinci sınıf olan <strong>CAcModuleResourceOverride</strong> sınıfı ise AutoCAD ile ObjectARX modülü kaynakları arasındaki geçişi sağlar. Uygulamanıza ait diyalog kutularını kullanmadan önce bu sınıfın bildirimini aşağıdaki gibi yaparsanız diyalog kutunuz başarıyla yaratılır ve beklenmedik hatalarla karşılaşmazsınız. 

```cpp
 CAcModuleResourceOverride resOverride;  
 // Artık diyaloğunuzu çağırabilirsiniz  
 CTestDialog diaTest;  
 diaTest.DoModal();
```

ObjectArx, kendi arayüzlerimizi oluşturabilmemiz için AutoCAD içinde kullanılan arayüzlere benzer bazı MFC kullanıcı ara yüzü sınıfları (UI) içerir. Bunlar <strong>AdUi</strong> ve <strong>AcUi</strong> ön ekleriyle başlayan sınıflardır. AdUi, AutoCAD ve diğer Autodesk ürünlerinde kullanılmak üzere geliştirilmiştir ve MFC UI’yi genişletir. <strong>AcUi</strong> ise AutoCAD’e özgü görünüş ve davranışı sağlayan AdUi tabanlı bir kütüphanedir. <strong>AdUi</strong> AutoCAD’den bağımsız çalışan uygulamalarda kullanılabilir olmasına rağmen lisans anlaşması gereği AutoCAD ile etkileşimli olmayan kullanımı yasaktır. <strong>AdUi</strong> ile <strong>AcUi</strong> sınıflarını kullanabilmek için <strong>adui.h</strong> başlık dosyasını ilgili C++ kaynak dosyamıza ve <strong>adui18.lib,  acui18.lib</strong> (AutoCAD 2012 için) kütüphanelerini ise projelerimize eklemeliyiz. Ayrıca <strong>DllMain(…)</strong> metodu içinde, <strong>InitAcUiDLL()</strong> ve <strong>InitAdUiDLL()</strong> metodlarını kullanarak AutoCAD’in bu kütüphaneleri çağırmasını sağlayabiliriz. 