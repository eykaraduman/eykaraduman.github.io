---
title: "AutoCAD .NET API Nesne Modeli"
permalink: /autocad-net-programming/autocad-object-model/
toc: true
classes: wide
comments: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---
AutoCAD .NET API'de birçok farklı nesne türü barındırmaktadır. Bu nesnelerin bazıları şunlardır: 

- Çizgiler, yaylar, metinler ve ölçülendirmeler gibi grafiksel nesneler 
- Katman, çizgi tipi ve ölçülendirme stilleri 
- Katmanlar, gruplar ve bloklar gibi düzenleyici yapılar
- Çizimin görünümüyle ilgili nesneler (View, Viewport)
- AutoCAD uygulaması ve çizimi

Nesneler, AutoCAD `Application` nesnesi en başta olacak şekilde hiyerarşik bir şekilde dizilmiştir. Bu hiyerarşik yapıya *Nesne Modeli* denir. Aşağıdaki gösterim, AutoCAD nesneleri arasındaki temel ilişkileri göstermektedir. AutoCAD .NET API, aşağıda gösterilenlerin dışında nesneler de barındırmaktadır.

<div class="mermaid">
graph TD
A(Application) --> B[DocumentManager]
B --> C[Document]
C --> D[Database]
D --> E[Tables]
E --> F[BlockTable]
F --> J[BlockTableRecord]
J --> J1((1. Nesne<br>Text))
J --> J2((2. Nesne<br>Polyline))
J --> J3((n. Nesne<br>...))
E --> G[LayerTable]
G --> G1[LayerTableRecord]
E --> H[TextStyleTable]
H --> H1[TextStyleTableRecord]
E -.-> I[...]
E -.-> I1[...]
E -.-> I2[...]
</div>

### Application Nesnesi

Application nesnesi, AutoCAD. NET API kök nesnesidir. Bu nesne aracılığıyla AutoCAD ana penceresine ve çizim veritabanına ulaşılabilir. Application nesnesi, `Autodesk.AutoCAD.ApplicationServices` ve `Autodesk.AutoCAD.ApplicationServices.Core` isim uzaylarında yer alır.

|Application Nesnesinin İçerdikleri||
| ---------------------------- | --------------- |
| **DocumentManager**    |   Açık çizimlere ait doküman nesnelerini içerir.   |
| **DocumentWindowCollection** | Her dokümana ait doküman pencerelerini içerir. |
| **MainWindow** | AutoCAD uygulama penceresinin referansıdır. |
| **MenuBar** | AutoCAD MenuBar COM nesnesinin referansıdır |
| **MenuGroups** | AutoCAD MenuGroup COM nesnesinin referansıdır. |
| **Preferences** | AutoCAD seçeneklerine erişmek ve değiştirmek için kullanılır. |
| **StatusBar** | AutoCAD StatusBar (durum çubuğu) nesnesidir. |
| **Publisher** | Çizimlerin yayınlanmasına hizmet eden nesnedir. |
| **InfoCenter** | Bilgi merkezi araç menüsü referansıdır. |
| **UserConfigurationManager** | Kullanıcı profilleriyle çalışmaya izin veren nesnedir. |

#### Application Nesnesine Erişim

Aşağıdaki kod parçası `Application` nesnesine ulaşarak AutoCAD uygulamasının major sürüm değerini verecektir. Örneğin AutoCAD 2013 sürümü için bu değer 19'dur.

```
var majorVersion = Autodesk.AutoCAD.ApplicationServices.Application.Version.Major;
```
Application nesnesi ayrıca bazı önemli yordamlar da içermektedir:

- `LoadMainMenu`, `LoadPartialMenu`, `ReloadAllMenus`, `UnloadPartialMenu` gibi menü işlemleri ile ilgili olanlar
- `ShowModelessDialog`, `ShowModalDialog`, `ShowModelessWindow`, `ShowModalWindow` gibi kullancı ara yüzüyle ilgili olanlar
- Sürükle-bırak için kullanılan `DragDrop` yordamı

### Document Nesnesi

 `Document` nesnesi aslında bir AutoCAD çizimidir ve `DocumentCollection` nesnesinin bir parçasıdır. Çizim dosyalarını oluşturmak, açmak ve kapatmak için `DocumentExtension` ve `DocumentCollectionExtention` nesneleri kullanılır. `Document` nesnesi ile tüm grafiksel ve grafiksel olmayan AutoCAD nesnelerinin çoğunu içeren `Database` nesnesine erişilebilmektedir. 

`Database` ve `Document` nesneleri ile durum çubuğuna, belgenin açıldığı pencereye, `Editor` ve `TransactionManager` nesnelerine ulaşılabilir. 

`Editor` nesnesi, kullanıcılardan bilgi toplamak için kullanılmaktadır. 

İşlem yığını yöneticisi (TransactionManager nesnesi) ise tek bir işlem (transaction) altında birden çok veritabanı nesnesine erişmek için kullanılır.

<div class="mermaid">
graph TD
A(Application) ---B[DocumentManager]
B --- C[Document]
C --- D1[Database]
C --- D2[Editor]
C --- D3[GraphicsManager]
C --- D4[Statusbar]
C --- D5[TransactionManager]
C --- D6[UserData]
C --- D7[Window]
</div>
#### Document Nesnesine Erişim

```
Document doc = Autodesk.AutoCAD.ApplicationServices.Core.Application.DocumentManager.MdiActiveDocument;
```

AutoCAD etkin doküman nesnesine yukarıdaki kod parçasıyla ulaşılabilir.

### Database Nesnesi

Veritabanı nesnesi, tüm grafiksel ve grafiksel olmayan AutoCAD nesnelerinin çoğunu içerir. Veritabanında bulunan nesnelerden bazıları varlıklar (entity), sembol tabloları (symbol tables) ve adlandırılmış sözlüklerdir (named object dictionary). Veritabanındaki varlıklar, bir çizim içindeki grafik nesneleri temsil eder. Çizgiler, daireler, yaylar, metinler, taramalar ve çoklu çizgiler varlık örnekleridir.

#### Database Nesnesine Erişim

Mevcut dokümanın veritabanı nesnesine, doküman nesnesinin veritabanı üye özelliği ile erişilir.

```c#
Database db1 = Autodesk.AutoCAD.ApplicationServices.Core.Application.DocumentManager.MdiActiveDocument.Database;
Database db2 = Autodesk.AutoCAD.DatabaseServices.HostApplicationServices.WorkingDatabase;
```

### Grafiksel ve Grafiksel Olmayan Nesneler

Varlıklar olarak da bilinen grafiksel nesneler, bir çizimi oluşturan görünür nesnelerdir (çizgiler, daireler, metinler vb.dir). Bir çizime grafiksel nesneleri eklemek için, doğru blok tablo kaydına ulaşılmalı ve ardından da  `AppendEntity` yöntemi kullanılarak çizime yeni nesneler eklenmelidir. 

```
Circle circle = new Circle {Center = new Point3d(0.0, 0.0, 0.0), Radius = 10.0};
ObjectId circleId = blockTableRecord.AppendEntity(circle);
```

Nesneleri değiştirmek ya da sorgulamak içinse, ModelSpace/PaperSpace blok tablosu kaydı üzerinden nesnelerin referansına ulaşmak gerekir. Her grafiksel nesne, Kopyala, Sil, Taşı, Aynala, Döndür gibi AutoCAD düzenleme komutu işlevlerini gerçekleştiren yöntemlere sahiptir. AutoCAD .NET API terminolojisinde bu yöntemlerin genel adı dönüşümlerdir . Bu yöntemlerin uygulanmasında dönüşüm matrisleri ve varlıkların`TransforBy` yordamı kullanılır.

Ayrıca varlıklar ve nesneler genişletilmiş veriler (extented data/xdata)  içerebileceği gibi başka bir varlığın özelliklerini de sahiplenebilirler. Çoğu grafiksel nesne, `ObjectId`,  `LayerId`, `LinetypeId`, `Color` ve `Hande` gibi bazı ortak özelliklere sahiptir. 

*Grafiksel olmayan nesneler*, katmanlar, çizgi tipleri, ölçülendirme stilleri, tablo stilleri gibi bir çizimin parçası olan ancak görünmeyen nesnelerdir. Bu nesneler sembol tabloları olarak adlandırılmıştır. Yeni bir sembol tablosu oluşturmak mümkün değildir.

Yeni bir sembol tablosu kaydı oluşturmak için ilgilenilen tablo türünün `Add` yordamı kullanılır.

Diğer bir grafiksel olmayan nesnelerden biri de sözlüklerdir. Sözlük, herhangi bir AutoCAD nesnesini veya bir XRecord'u içerebilen kapsayıcı bir nesnedir. Sözlükler ya adlandırılmış nesne sözlüğü ya da bir sembol tablo kaydının/grafiksel varlığın uzantı sözlüğü (extension dictionary) olarak saklanır. Adlandırılmış nesne sözlüğüne (NOD) bir sözlük eklemek için `SetAt` yordamını kullanmak gerekir.

Adlandırılmış nesne sözlüğü (NOD), veritabanıyla ilişkili tüm diğer sözlükler için ana tablodur. Sembol tablolarından farklı olarak, yeni sözlükler oluşturulabilir. Sözlükler çizim varlıklarını içeremezler. Ancak çizim varlıkları sözlüklerde `Handle`'ları  (değişmez kimlikleri) aracılığıyla saklanabilmektedir.

![Şekil-1](/assets/images/default-dwg-nod.png)

<figcaption>Varsayılan .dwg dosyasının içerdiği adlandırılmış nesne sözlükleri</figcaption>

### Koleksiyon Nesneleri

AutoCAD veritabanı/çizimi, çoğu grafiksel ve grafiksel olmayan nesneyi koleksiyonlar veya depolama (container) nesneleri halinde gruplandırır. Bu nesneler farklı türde veriler içermesine rağmen, kullanımlarını ve öğrenmelerini kolaylaştırmak için ortak yöntem ve özellikler içerir. `Count` özelliği ve `Item` işlevi bunların bir örneğidir. 

AutoCAD .NET API'sindeki koleksiyon üyelerine aşağıdakiler örnek olarak verilebilir: 

- Katman sembol tablosundaki (LayerTable) katman kaydı (LayerTableRecord)
- ACAD_LAYOUT sözlüğündeki Layout
- DocumentCollection'daki Document
- Bir blok referansındaki nitelikler (atttibutes)
