---
title: AutoCAD .NET API Başlangıç
permalink: /autocadnet/beginnertutorial/autocad-net-api-start/
classes: wide
comments: true
toc: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-beginner-tutorial
---

AutoCAD® için AutoLISP, VBA, ObjectARX® / ObjectDBX®, .NET Framework tabanlı uygulama arayüzleri
kullanılmaktadır. Bu arayüzlerden en yaygın olanı, ilk kararlı sürümü 1996 yılında yayınlanan AutoLISP programlama dilidir. Yaygınlığını özellikle AutoCAD için tasarlanmış olmasına, derlenmeye ihtiyaç duymamasına
ve büyük bir kitle tarafından kullanılıp desteklenmesine borçludur. AutoLISP’in AutoCAD programı içinden
ulaşabilen VLISP adında bütünleşik bir geliştirme ortamı da bulunmaktadır.

ObjectARX® (AutoCAD Runtime Extension) daha çok profesyonel programlar geliştirmekte kullanılmaktadır
ve üst düzey nesne tabanlı C++ bilgisi gerektirdiğinden, uzmanlaşılması diğer AutoCAD API’leri ile karşılaştırıldığında en güç olanıdır. ObjectARX®, ObjectDBX® ile birlikte özel nesne tasarımını destekleyen tek
AutoCAD API’sidir ve yapısı nedeniyle AutoCAD çizim veritabanına ulaşmakta herhangi bir kısıt barındırmaz.
Autodesk tarafından dağıtılan ObjectARX SDK (Software Development Kit) C++ kütüphanelerinden oluşmak-
tadır.

AutoCAD .NET API ise, en basit tanımıyla, ObjectARX C++ sınıﬂarının büyük bir çoğunluğunun .NET platformuna aktarılmış/açılmış halidir. ObjectARX’te olduğu gibi özel nesne tasarımını içermese de, yerleşik AutoCAD
nesnelerinin (line, polyline, text vb.) davranışını Overrule API ile değiştirmek mümkündür.

AutoCAD .NET API, AutoLISP ile karşılaştırıldığında, ObjectARX API özelliklerinin çoğunu içermesi nedeniyle kapsamlı ve gelişmiş programlar oluşturmaya imkan tanımaktadır. Ayrıca .NET Framework ile kullanıcı
arayüzü geliştirme olanakları (WinForms ve WPF), AutoLISP ve ObjectARX’e göre kolay ve modern bir hal
almıştır.

AutoLISP, VisualLISP ile ilgili Türkçe yayınlanmış kitaplar/kaynaklar olmasına rağmen, AutoCAD .NET API ile ilgili
Türkçe bir kaynak bulunmamaktadır. Bu kılavuzun açığı kapatacağı
düşünülmüştür.

AutoCAD .NET API ile Programlamaya Giriş Kılavuzu, .NET Framework’un en popüler dili olan C# ile AutoCAD .NET
API’nin nasıl kullanılabileceğine odaklanmaktadır. 

Bu kılavuzdan tam olarak yararlanabilmek için en az orta düzeyde C# programlama dili bilgisine sahip ve Microsoft® Visual Studio Geliştirme Ortamını da temel düzeyde kullanmayı biliyor olmalısınız.

### İçerik ve Kapsam

- [x] AutoCAD .NET API'ye genel bir bakış: AutoCAD .NET API nedir?
- [x] Microsoft Visual Studio ile AutoCAD eklentisinin oluşturulması
- [ ] Blok Tabloları ve Blok Tablo Kayıtları

- [ ] AutoCAD .NET API nesne hiyerarşisi
  - [ ] Application, Document ve Database nesneleri
- [ ] AutoCAD komutlarının oluşturulması
- [ ] AutoCAD.NET API ile  işlem yığını (transaction) kullanımı
- [ ] AutoCAD.NET API ile AutoCAD grafik nesnelerin oluşturulması
- [ ] AutoCAD nesneleri üzerinde işlemler
  - [ ] AutoCAD .NET API ile dönüşüm matrisler
  - [ ] Nesneleri kopyalama, taşıma, döndürme, ölçekleme, aynalama ve silme
- [ ] AutoCAD yerleşik sözlükleri (Katmanlar, Çizgi Tipleri, Metin Stilleri vb.)
- [ ] AutoCAD'de veri saklama yöntemleri (XData ve NOD'lar)
  - [ ] ResultBuffer veri tipi
- [ ] Kullanıcı giriş yordamları
- [ ] Seçim setlerinin oluşturulması ve filtreleme
- [ ] AutoCAD. NET API ile olaylar
- [ ] AutoCAD .NET API ile WPF kullanımı
- [ ] AutoCAD .NET uygulamalarının dağıtılması
- [ ] AutoCAD Eklenti Projesi-1
- [ ] AutoCAD Eklenti Projesi-2

### Kılavuz Kaynak Kodları

Kılavuzun kaynak kodlarına [eykaraduman/PgAutoCAD](eykaraduman/PgAutoCAD) github deposundan ulaşabilirsiniz.

### Referanslar

**AutoCAD .NET ile Programlamaya Giriş** kılavuzu hazırlanırken farklı yabancı sitelerden ve kitaplardan da faydalanılmıştır. Bu kaynaklar aşağıda listelenmiştir.

#### Web Siteleri

- [AutoCAD® Developer and ObjectARX Help](https://help.autodesk.com/view/OARX/2022/ENU/)
- [Autodesk® Developer Network](https://www.autodesk.com/developer-network/overview)
- [Autodesk AutoCAD .NET API Forum](https://forums.autodesk.com/t5/net/bd-p/152)
- [AutoCAD DevBlog](https://adndevblog.typepad.com/autocad/)
- [AutoCAD University](https://www.autodesk.com/autodesk-university/au-online)
- [Through the Interface](https://www.keanw.com/)
- [Drive AutoCAD with Code](https://drive-cad-with-code.blogspot.com/)
- [theswamp.org](https://www.theswamp.org/)
- [spiderinnet1](https://spiderinnet1.typepad.com/blog/)

#### Kitaplar

- Instant Autodesk AutoCAD 2014 Customization with.NET
- VB.NET Programming for AutoCAD Customization

#### Github Kaynakları

- [https://github.com/wtertinek/Linq2Acad](https://github.com/wtertinek/Linq2Acad)
- [https://github.com/luanshixia/AutoCADCodePack](https://github.com/luanshixia/AutoCADCodePack)
- [https://github.com/ADN-DevTech/MgdDbg](https://github.com/ADN-DevTech/MgdDbg)

**Lisans:** Bu kılavuz, "https://eykaraduman.github.io/autocadnet/beginnertutorial/", Creative Commons Attribution 4.0 Uluslararası Lisansı ile lisanslanmıştır. Bu lisansın bir kopyasını görüntülemek için  https://creativecommons.org/licenses/by/4.0/ adresini ziyaret edin.
{: .notice--warning}