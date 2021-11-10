---
title: AutoCAD .NET API Başlangıç
permalink: /autocadnet/beginnertutorial/autocad-net-api-start/
excpert: "AutoCAd.NET API başvuru kaynakları ve gereksinimleri."
comments: true
toc: true
---

AutoCAD® için yaygın olarak kullanılan AutoLISP dışında, ObjectARX®/ObjectDBX® ve AutoCAD .NET API için Türkçe kaynak bulmak neredeyse imkansız. Bu kılavuzun amacı, AutoCAD .NET uygulama ara yüzünü yeni öğrenmeye başlayanlar için Türkçe bir kaynak oluşturmak. 

### İçerik ve Kapsam

- [x] AutoCAD .NET API'ye genel bir bakış
- [x] Microsoft Visual Studio ile AutoCAD eklentisinin oluşturulması
- [ ] AutoCAD .NET API nesne hiyerarşisi
  - [ ] Appplication, Document ve Database nesneleri
- [ ] AutoCAD komutlarının oluşturulması
- [x] AutoCAD.NET API ile  işlem yığını (transaction) kullanımı
- [ ] AutoCAD.NET API ile AutoCAD grafik nesnelerin oluşturulması
- [ ] AutoCAD nesneleri üzerinde işlemler
  - [ ] AutoCAD .NET API ile dönüşüm matrisler
  - [ ] Nesneleri kopyalama, taşıma, döndürme, ölçekleme, aynalama ve silme

- [ ] AutoCAD sözlükleri (Katmanlar, Çizgi Tipleri, Metin Stilleri vb.)
- [ ] AutoCAD'de veri saklama yöntemleri (XData ve NOD'lar)
  - [ ] ResultBuffer veri tipi
- [ ] Kullanıcı giriş yordamları
- [ ] Seçim setlerinin oluşturulması ve filtreleme
- [ ] AutoCAD. NET API ile olaylar
- [ ] AutoCAD .NET API ile WPF kullanımı
- [ ] AutoCAD .NET uygulamalarının dağıtılması
- [ ] AutoCAD Eklenti Projesi-1
- [ ] AutoCAD Eklenti Projesi-2

### Geliştirme Ortamının Hazırlanması

Bu kılavuzdaki örnekleri sınayabilmek için aşağıdaki yazılımlara sahip olmalısınız:

-  Autodesk AutoCAD 2013-2022 (Henüz kurulu değilse, Autodesk'ten 30 günlük ücretsiz deneme sürümünü indirebilirsiniz.)
-  Microsoft Visual Studio 2017 (Ücretsiz Community sürümü yeterli olacaktır.)
- Microsoft .NET Framework 4.0 (Henüz yüklenmemişse, Microsoft Visual Studio .NET Framework 4.0'ü yükleyecektir.)

### Kılavuz Kaynak Kodları

Kılavuzun kaynak kodlarına [eykaraduman/PgAutoCAD](eykaraduman/PgAutoCAD) github deposundan ulaşabilirsiniz.

### Referanslar

**AutoCAD.Net ile Programlamaya Giriş** kılavuzu hazırlanırken farklı yabancı sitelerden ve kitaplardan da faydalanılmıştır. Bu kaynaklar aşağıda listelenmiştir.

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