---
title: AutoCAD .NET API Nedir?
permalink: /autocadnet/beginnertutorial/autocad-net-api-nedir/
toc: true
classes: wide
comments: true
sidebar:
  title: "AutoCAD .NET API ile Programlamaya Giriş"
  url: /autocadnet/beginnertutorial/
  nav: autocadnet-beginner-tutorial
---

## AutoCAD .NET API Nedir?

AutoCAD .NET API, ObjectARX C++ sınıﬂarının Microsoft .Net Framework platformuna aktarılmış halidir.
AutoCAD .NET API bu C++ sınıﬂarın büyük bir çoğunluğunu içermektedir ve API ile yerleşik AutoCAD
nesneleri oluşturulabilmekte ve bu nesneler üzerinde taşıma, kopyalama, döndürme gibi işlemlerler gerçekleştirilebilmektedir. AutoCAD .NET API kullanılarak C#, F#, VB.NET ve IronPython gibi dillerle AutoCAD uygulamaları geliştirilebilir.

Nesneler bu uygulama ara yüzünün en önemli öğeleridir ve farklı isim uzayları ile çatılar altında gruplandırılmışlardır. AutoCAD .NET uygulama arayüzü bir çok farklı nesne içerir.

- Çizgi, metin, ölçülendirme gibi grafik nesneler
- Metin, ölçülendirme, tablo vb. için kullanılan stiller
- Katmanlar, gruplar ve bloklar gibi düzenleyici yapılar
- Görüş (View) ve görüş alanı (viewport) gibi görünüşle ilgili nesneler
- AutoCAD çizimi ve uygulaması

ObjectARX/ObjectDBX'de olduğu gibi AutoCAD .NET API ile yeni bir AutoCAD nesnesi oluşturmak mümkün değildir. Sadece yerleşik AutoCAD nesnelerinin davranışı değiştirilebilir. (Overrule API) 
{: .notice--warning}

Önemli AutoCAD .NET API isim uzayları:

- Autodesk.AutoCAD.Runtime
- Autodesk.AutoCAD.DatabaseServices
- Autodesk.AutoCAD.ApplicationServices
- Autodesk.AutoCAD.Geometry
- Autodesk.AutoCAD.GraphicsInterface
- Autodesk.AutoCAD.LayerManager
- Autodesk.AutoCAD.PlottingServices
- Autodesk.AutoCAD.BoundaryRepresentation 
- Autodesk.AutoCAD.Colors

## AutoCAD .NET API Bileşenleri

AutoCAD .NET API, çizim dosyası içindeki nesnelere ya da AutoCAD uygulamasına ulaşmak için tasarlanmış
sınıﬂar, yapılar, yordamlar ve olaylar içeren DLL dosyalarından oluşmaktadır. Temel AutoCAD .NET API DLL
dosyaları aşağıdaki gibidir.

- **AcCoreMgd.dll.** Komut satırına ulaşmak, baskı işlemleri vb. için kullanılır. (2013 sürümüyle birlikte yayınlanmıştır.)
- **AcDbMgd.dll.** Çizim dosyasında saklanan nesnelere ulaşmak için kullanılır.
- **AcMgd.dll.** AutoCAD uygulaması ve ara yüzü ile çalışırken kullanılır.
- **AcCui.dll.** Özelleştirme dosyalarıyla çalışılırken kullanılır.

Bir AutoCAD .NET API DLL dosyası, Microsoft Visual Studio projesine başvuru olarak eklendiğinde, başvurulan DLL'nin Copy Local özelliğinin False olarak ayarlamalıdır. Copy Local özelliği, başvurulan DLL dosyasının bir kopyasının, Microsoft Visual Studio tarafından derleme dizininde oluşturup oluşturmayacağını belirler. Başvurulan dosyalar zaten AutoCAD ile birlikte dağıtıldığından, başvurulan DLL dosyalarının kopyalarını oluşturmak, derlenen eklenti dosyası yüklendiğinde beklenmeyen sonuçlara neden olabilir.

## AutoCAD .NET API DLL Dosyaları Nerede?

- AutoCAD programının kurulu olduğu dizinde bulunur.
- [ObjectARX SDK](https://www.autodesk.com/adn)  kurulduktan sonra, DLL dosyaları ana kurulum klasörünün altındaki **inc** dizininde bulunabilir.

ObjectARX SDK'daki DLL'ler, AutoCAD ile birlikte dağıtılan aynı dosyaların bağımlılıklarını içermezler ve  basitleştirilmiş sürümleridir. Bu yüzden AutoCAD veya AutoCAD tabanlı programın kurulum dizininde bulunanlar yerine SDK ile birlikte gelen DLL dosyalarına başvurulmalıdır.
{: .notice--warning}

## AutoCAD .NET API ve .Net Framework Runtime Uyumluluğu

Autodesk, ilk olarak 2005 yılında ObjectARX kütüphanelerinin .NET sürümlerini yayınlamıştır. AutoCAD
.NET ile yazılan uygulamaların, kullanıcıların bilgisayarlarında çalışabilmesi için bu bilgisayarların barındırdıkları AutoCAD sürümüne göre derlenmesi gerekmektedir. Ayrıca AutoCAD sürümüne göre uygulamaları
derlemekte kullanması gereken Visual Studio sürümleri de değişmektedir.

Aşağıdaki tabloda AutoCAD sürümlerini için hangi .NET Framework Runtime ve Microsoft Visual Studio sürümlerinin kullanması gerektiğini göstermektedir.

| AutoCAD Harici Sürüm | AutoCAD İç Sürüm | .NET Framework Runtime | Microsoft Visual Studio Sürümü |
| :------------------- | :--------------- | :--------------------- | :----------------------------- |
| 2005                 | 16.1             | 1.0                    | 2002                           |
| 2006                 | 16.2             | 1.1                    | 2003                           |
| 2007                 | 17.0             | 2.0                    | 2005/2008/2010/2012            |
| 2008                 | 17.1             | 2.0                    | 2005/2008/2010/2012            |
| 2009                 | 17.2             | 3.0                    | 2008/2010/2012                 |
| 2010                 | 18.0             | 3.5                    | 2008/2010/2012                 |
| 2011                 | 18.1             | 3.5                    | 2008/2010/2012                 |
| 2012                 | 18.2             | 4.0                    | 2010/2012/2013                 |
| 2013                 | 19.0             | 4.0                    | 2010/2012/2013                 |
| 2014                 | 19.1             | 4.0                    | 2010/2012/2013                 |
| 2015                 | 20.0             | 4.5                    | 2012/2013                      |
| 2016                 | 20.1             | 4.5                    | 2012/2013/2015                 |
| 2017                 | 21.0             | 4.6                    | 2015                           |
| 2018                 | 22.0             | 4.6                    | 2015                           |
| 2019                 | 23.0             | 4.7                    | 2017                           |
| 2020                 | 23.1             | 4.7                    | 2017                           |
| 2021                 | 24.0             | 4.7                    | 2019                           |
| 2022                 | 24.1             | 4.7                    | 2019                           |

Harici sürümü, AutoCAD’in ticari olarak pazarlanmasında kullanılan sürüm adıdır. Çoğunlukla sürüm hakkında yapılan tartışmalar ve hata bildirimlerinde kullanılır. İç sürüm adı ise AutoCAD’in windows kayıt defterindeki adıdır ve daha çok uygulama geliştiricileri tarafından sürüm faklılıklarından doğan uyum sorunlarını çözmekte kullanılır. İç sürüm adının ilk iki hanesi AutoCAD uygulamaları açısından ikili (binary) uyumluluğu gösterir. Örneğin AutoCAD 17.0 için geliştirdiğiniz bir uygulama AutoCAD 17.1 ve AutoCAD 17.2 için de geçerlidir.
