---
classes: wide
title: AutoCAD.Net API
permalink: /autocadnet/beginnertutorial/autocad-net-api/
toc: true
---

## AutoCAD .NET API Nedir?

AutoCAD .NET API, en basit tanımıyla ObjectARX C++ sınıflarının .Net platformuna aktarılmış halidir. AutoCAD .NET API ile çizim veritabanında saklanan nesneler üzerinde çeşitli işlemler ( oluşturma, değiştirme gibi) yapılabilir. AutoCAD .NET API kullanılarak C#, F#, VB.NET ve IronPython gibi dillerle AutoCAD uygulamaları geliştirilebilir.

Nesneler bu uygulama ara yüzünün en önemli öğeleridir ve farklı isim uzayları ile çatılar altında gruplandırılmışlardır. AutoCAD .NET uygulama ara yüzü bir çok farklı nesne içerir.

- Çizgi, metin, ölçülendirme gibi grafik nesneler
- Metin, ölçülendirme, tablo vb. için kullanılan stiller
- Katmanlar, gruplar ve bloklar gibi düzenleyici yapılar
- Görüş ve görüş alanı gibi görünüşle ilgili nesneler
- AutoCAD çizimi ve uygulaması

ObjectARX/ObjectDBX'de olduğu gibi AutoCAD .NET API ile yeni bir AutoCAD nesnesi oluşturmak mümkün değildir. Sadece yerleşik AutoCAD nesnelerinin davranışı değiştirilebilir. (Overrule API) 
{: .notice--warning}

Önemli AutoCAD .NET API isim uzayları:

- Autodesk.AutoCAD.BoundaryRepresentation 
- Autodesk.AutoCAD.Colors
- Autodesk.AutoCAD.DatabaseServices
- Autodesk.AutoCAD.Geometry
- Autodesk.AutoCAD.GraphicsInterface
- Autodesk.AutoCAD.LayerManager
- Autodesk.AutoCAD.PlottingServices
- Autodesk.AutoCAD.Runtime
- Autodesk.AutoCAD.DatabaseServices
- Autodesk.AutoCAD.ApplicationServices

## AutoCAD .NET API Bileşenleri

AutoCAD .NET API, çizim dosyası içindeki nesnelere ya da AutoCAD uygulamasına ulaşmak için sınıflar, yapılar, yordamlar ve olaylar içeren DLL dosyalarından oluşmaktadır.

Sıklıkla kullanılan AutoCAD .NET API DLL dosyaları aşağıdaki gibidir.

- **AcCoreMgd.dll.** Komut satırına ulaşmak, baskı işlemleri vb. için kullanılır.
- **AcDbMgd.dll.** Çizim dosyasında saklanan nesnelere ulaşmak için kullanılır.
- **AcMgd.dll.** AutoCAD uygulaması ve ara yüzü ile çalışırken kullanılır.
- **AcCui.dll.** Özelleştirme dosyalarıyla çalışılırken kullanılır.

Bir AutoCAD .NET API DLL dosyası, Microsoft Visual Studio projesine başvuru olarak eklendiğinde, başvurulan DLL'nin Copy Local özelliğinin False olarak ayarlamalıdır. Copy Local özelliği, Microsoft Visual Studio'nun başvurulan DLL dosyasının bir kopyasını oluşturup oluşturmayacağını ve bunu proje derlenirken oluşturulan derleme dosyası (veya yürütülebilir dosya) ile aynı dizine yerleştirip yerleştirmeyeceğini belirler. Başvurulan dosyalar zaten AutoCAD ile birlikte dağıtıldığından, başvurulan DLL dosyalarının kopyalarını oluşturmak, derlenen eklenti dosyası yüklendiğinde beklenmeyen sonuçlara neden olabilir.

## AutoCAD .NET API DLL Dosyaları Nerede?

- AutoCAD programının kurulu olduğu dizinde bulunur.
- [ObjectARX SDK](https://www.autodesk.com/adn)  kurulduktan sonra, DLL dosyaları ana kurulum klasörünün altındaki **inc** dizininde bulunabilir.

ObjectARX SDK'daki DLL'ler, AutoCAD ile birlikte dağıtılan aynı dosyaların bağımlılıklarını içermezler ve  basitleştirilmiş sürümleridir. Bu yüzden ObjectARX SDK'yı indirilip kurulmalı ve ardından AutoCAD veya AutoCAD tabanlı programın kurulum dizininde bulunanlar yerine SDK ile birlikte gelen DLL dosyalarına başvurulmalıdır.
{: .notice--warning}

