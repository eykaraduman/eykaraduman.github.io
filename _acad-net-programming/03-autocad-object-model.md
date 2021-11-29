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
- Katman, çizgi tipi ve ölçülendirme stilleri (sembol tabloları)
- Katmanlar, gruplar ve bloklar gibi düzenleyici ve gruplandırıcı yapılar
- Çizimin görünümüyle ilgili nesneler (View, Viewport)
- AutoCAD uygulaması ve çizimi

Nesneler, `Application` nesnesi en başta olacak şekilde hiyerarşik bir şekilde dizilmiştir. Bu hiyerarşik yapıya *Nesne Modeli* denir. Aşağıdaki gösterim, AutoCAD nesneleri arasındaki temel ilişkileri göstermektedir. AutoCAD .NET API, aşağıda gösterilenlerin dışında bir çok farklı nesne de barındırmaktadır.

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

`Autodesk.AutoCAD.ApplicationServices` ve `Autodesk.AutoCAD.ApplicationServices.Core` isim uzaylarında yer alan `Application` nesnesi, AutoCAD. NET API kök nesnesidir. Bu nesne aracılığıyla AutoCAD ana penceresine ve çizim veritabanına ulaşılabilmektedir.

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

- `LoadMainMenu`, `LoadPartialMenu`, `ReloadAllMenus`, `UnloadPartialMenu` (menü işlemleri)
- `ShowModelessDialog`, `ShowModalDialog`, `ShowModelessWindow`, `ShowModalWindow` (kullancı ara yüzüyle işlemleri)

### Document Nesnesi

`Document` nesnesi aslında bir AutoCAD çizimidir ve `DocumentCollection` nesnesinin bir parçasıdır. Çizim dosyalarını oluşturmak, açmak ve kapatmak için `DocumentExtension` ve `DocumentCollectionExtention` nesneleri kullanılır. `Document` nesnesi ile tüm grafiksel ve grafiksel olmayan AutoCAD nesnelerin içeren `Database` nesnesine erişmek mümkündür. 

`Database` ve `Document` nesneleri ile durum çubuğuna (StatusBar), belgenin açıldığı pencereye (Window), `Editor` ve `TransactionManager` nesnelerine ulaşılabilir. 

`Editor` nesnesi, kullanıcılardan string, int, double, point vb. türlerde bilgi toplamak için kullanılmaktadır. 

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

Veritabanı (Database) nesnesi, tüm grafiksel ve grafiksel olmayan AutoCAD nesnelerin içerir. Veritabanında bulunan nesnelerden bazıları varlıklar (entity), sembol tabloları (symbol tables) ve adlandırılmış sözlüklerdir (named object dictionaries). Veritabanındaki varlıklar, bir çizim içindeki grafik nesneleri temsil eder. Çizgiler, daireler, yaylar, metinler, taramalar ve çoklu çizgiler varlık örnekleridir.

#### Database Nesnesine Erişim

Mevcut dokümanın veritabanı nesnesine, `Document` nesnesinin `Database` üye özelliği ile erişilir. Aşağıdaki kod parçası bu reişimin iki faklı yolunu göstermektedir.

```c#
Database db1 = Autodesk.AutoCAD.ApplicationServices.Core.Application.DocumentManager.MdiActiveDocument.Database;
Database db2 = Autodesk.AutoCAD.DatabaseServices.HostApplicationServices.WorkingDatabase;
```

### Grafiksel ve Grafiksel Olmayan Nesneler

Varlıklar olarak da bilinen grafiksel nesneler, bir çizimi oluşturan görünür nesnelerdir (çizgiler, daireler, metinler vb.dir). Bir çizime grafiksel nesneleri eklemek için, doğru blok tablo kaydına (ModelSapace ya da PaperSpace) ulaşılmalı ve ardından da  `AppendEntity` yöntemi kullanılarak nesneler çizime eklenmelidir. 

```
Circle circle = new Circle {Center = new Point3d(0.0, 0.0, 0.0), Radius = 10.0};
ObjectId circleId = blockTableRecord.AppendEntity(circle);
```

Nesneleri değiştirmek ya da sorgulamak içinse, ModelSpace/PaperSpace blok tablosu kaydı üzerinden nesnelerin referansılarına ulaşmak gerekir. Her grafiksel nesne; kopyalama, silme, taşıma, aynalama ve döndürme gibi AutoCAD düzenleme komutu işlevlerini destekleyen ortak dönüşüm fonksiyonlarına sahiptir. AutoCAD .NET API de düzenleme işlemleri, dönüşüm matrisleri ve varlıkların`TransforBy` yordamı kullanılarak yerine getirilir.

Ayrıca varlıklar (entity) ve nesneler (objects) genişletilmiş veriler (extented data/xdata)  içerebileceği gibi başka bir varlığın özelliklerini de sahiplenebilirler. Çoğu grafiksel nesne, `ObjectId`,  `LayerId`, `LinetypeId`, `Color` ve `Hande` gibi bazı ortak özelliklere sahiptir. 

*Grafiksel olmayan nesneler*; katmanlar, çizgi tipleri, ölçülendirme stilleri, tablo stilleri gibi bir çizimin parçası olan ancak görünmeyen nesnelerdir. Bu nesneler *sembol tabloları* olarak adlandırılmıştır. Yeni bir sembol tablosu oluşturmak mümkün değildir. 

AutoCAD .NET API'de önemli nesnelerden bir diğeri ise *sözlüklerdir*. Sözlükler de grafiksel olmayan nesnelerdendir. Sözlük, herhangi bir AutoCAD nesnesini (object) ya d bir XRecord'u içerebilen kapsayıcı bir nesnedir. Sözlükler veritabanında, adlandırılmış nesne sözlüğü (NOD) ya da uzantı sözlüğü (extension dictionary) olarak saklanır. 

Adlandırılmış nesne sözlüğü (NOD), veritabanıyla ilişkili tüm diğer sözlükler için ana tablodur. (Bkz. Şekil-1) Sembol tablolarından farklı olarak, NOD altında yeni sözlükler oluşturulabilir. 

![Şekil-1](/assets/images/default-dwg-nod.png)

<figcaption>Şekil-1: Varsayılan .dwg dosyasının içerdiği adlandırılmış nesne sözlükleri</figcaption>

Sözlükler çizim varlıklarını (entities) içeremezler. Ancak çizim varlıkları, varlıkların `Handle` (değişmez kimlik) özelliği kullanılarak sözlüklerde saklamak için çeşitli yollar bulunmaktadır.

### Koleksiyon Nesneleri

AutoCAD veritabanı/çizimi, çoğu grafiksel ve grafiksel olmayan nesneyi, koleksiyonlar ya da depolama (container) nesneleri halinde gruplandırır. Bu nesneler farklı türde veriler içermesine rağmen, kullanımlarını ve öğrenilmelerini kolaylaştırmak için ortak yöntem ve özellikler içerecek şekilde tasarlanmıştır. `Count` özelliği ve `Item` işlevi bu durumun bir örneğidir. 

AutoCAD .NET API'sindeki koleksiyon üyelerine aşağıdakiler örnek olarak verilebilir: 

- Katman sembol tablosundaki (LayerTable) katman kaydı (LayerTableRecord)
- ACAD_LAYOUT sözlüğündeki Layout
- DocumentCollection'daki Document
- Bir blok referansındaki nitelikler (atttibutes)
