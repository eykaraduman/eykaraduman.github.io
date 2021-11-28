---
title: "AutoCAD Nesne Hiyerarşisi"
permalink: /autocad-net-programming/autocad-object-hierarcy/
toc: true
classes: wide
comments: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---
Bir nesne, AutoCAD .NET API'sinin ana yapı taşıdır. AutoCAD .NET API'de birçok farklı nesne türü vardır. AutoCAD .NET API'de temsil edilen nesnelerden bazıları şunlardır: 

- Çizgiler, yaylar, metin ve ölçülendirmeler gibi grafik nesneler 
- Katman, çizgi tipi ve ölçülendirme stilleri 
- Katmanlar, gruplar ve bloklar gibi düzenleyisi yapılar
- Çizimin görünümüyle ilgili olanlar (View, Viewport)
- AutoCAD uygulaması ve çizimi

Nesneler, AutoCAD `Application` nesnesi en başta olacak şekilde hiyerarşik bir şekilde dizilmiştir. Bu hiyerarşik yapıya Nesne Modeli denir. Aşağıdaki gösterim, AutoCAD nesneleri arasındaki temel ilişkileri göstermektedir. AutoCAD .NET API burada gösterilmeyen daha birçok nesne barındırmaktadır.

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

Application nesnesi, AutoCAD. NET API kök nesnesidir. Bu nesne aracılığıyla AutoCAD ana penceresine ve çizim veritabanlarına ulaşılabilir. Application nesnesi, `Autodesk.AutoCAD.ApplicationServices` ve `Autodesk.AutoCAD.ApplicationServices.Core` isim uzaylarında yer alır.

|||
| ---------------------------- | --------------- |
| **DocumentManager**    |   Açık çizimlere ait doküman nesnelerini içerir.   |
| **DocumentWindowCollection** | Her dokümana ait doküman pencerelerini içerir. |
| **MainWindow** | AutoCAD uygulama penceresinin referansıdır. |
| **MenuBar** | AutoCAD menü çubuğu COM nesnesinin referansıdır |
| **MenuGroups** | AutoCAD menü grupları COM nesnesinin referansıdır. Menü grubu, yüklenmiş CUIx dosyalarının özelleştirme grup adlarını içerir. |
| **Preferences** | Seçenekler diyalog kutusuna erişime ve seçenekleri değiştirmeye imkan sağlar. |
| **StatusBar** | AutoCAD durum çubuğu nesnesidir. |
| **Publisher** | Çizimlerin yayınlanmasına hizmet eden nesnedir. |
| **InfoCenter** | Bilgi merkezi araç menüsü referansıdır. |
| **UserConfigurationManager** | Kayıtlı profillerle çalışmaya izin veren nesnedir. |

#### Application Nesnesine Erişim

Aşağıda verilen kod parçası Application nesnesine ulaşarak AutoCAD uygulamasının major sürüm değerini verecektir. Örneğin AutoCAD 2013 sürümü için bu değer 19'dur.

```
var majorVersion = Autodesk.AutoCAD.ApplicationServices.Application.Version.Major;
```
Application nesnesi ayrıca bazı önemli yordamlar da içermektedir:

- Menü yordamları; `LoadMainMenu`, `LoadPartialMenu`, `ReloadAllMenus`, `UnloadPartialMenu`
- .Net Framework ile oluşturulan Form ve WPF Window'ları gösteren yordamlar; `ShowModelessDialog`, `ShowModalDialog`, `ShowModelessWindow`, `ShowModalWindow`
- Sürükle-bırak için kullanılan `DragDrop` yordamı

### Document Nesnesi

#### Document Nesnesine Erişim

### Database Nesnesi

Veritabanı nesnesi, tüm grafiksel ve grafiksel olmayan AutoCAD nesnelerinin çoğunu içerir. Veritabanında bulunan nesnelerden bazıları varlıklar (entity), sembol tabloları ve adlandırılmış sözlüklerdir. Veritabanındaki varlıklar, bir çizim içindeki grafik nesneleri temsil eder. Çizgiler, daireler, yaylar, metinler, taramalar ve çoklu çizgiler varlık örnekleridir.

#### Database Nesnesine Erişim

Mevcut dokümanın veritabanı nesnesine, doküman nesnesinin veritabanı üye özelliği ile erişlir.

```c#
Application.DocumentManager.MdiActiveDocument.Database
```

#### 

### Grafik ve Grafik Olmayan Nesneler

Varlıklar olarak da bilinen grafik nesneler, bir çizimi oluşturan görünür nesnelerdir (çizgiler, daireler, metinler vb.). Bir çizime grafik nesneleri eklemek için, doğru blok tablo kaydına ulaşılmalı ve ardından da  `AppendEntity` yöntemi kullanılarak çizime yeni nesneler eklenmelidir. 

// ekleme örneği

Nesneleri değiştirmek ya da sorgulamak içinse, ModelSpace/PaperSpace blok tablosu kaydı üzerinden nesnelerin referansına ulaşmak gerekir. Her grafik nesne, Kopyala, Sil, Taşı, Aynala, Döndür gibi AutoCAD düzenleme komutu işlevlerini gerçekleştiren yöntemlere sahiptir. 

// nesne üzerinde işlem örneği

Bu nesneler genişletilmiş veriler (xdata)  içerebilir ve başka bir varlığın özelliklerini de sahiplenebilir. Çoğu grafik nesnesi, `ObjectId`,  `LayerId`, `LinetypeId`, `Color` ve `Hande` gibi bazı ortak özelliklere sahiptir. 

*Grafiksel olmayan nesneler*, katmanlar, çizgi tipleri, ölçülendirme stilleri, tablo stilleri gibi bir çizimin parçası olan ancak görünmeyen nesnelerdir. Bu nesneler sembol tabloları olarak adlandırılmıştır ve bir veritabanına yeni sembol tabloları eklenemez.

Yeni bir sembol tablosu kaydı oluşturmak için, ilgilenilen tablosunun `Add` yordamı kullanılır.

// sembol tablo add



Adlandırılmış nesne sözlüğüne (NOD) bir sözlük eklemek için `SetAt` yordamını kullanmak gerekir.

// nod setat örnek

Sözlük, herhangi bir AutoCAD nesnesini veya bir XRecord'u içerebilen kapsayıcı bir nesnedir. Sözlükler ya veritabanında adlandırılmış nesne sözlüğü altında ya da bir tablo kaydının/grafiksel varlığın uzantı sözlüğü (extension dictionary) olarak saklanır. Adlandırılmış nesne sözlüğü, bir veritabanıyla ilişkili tüm sözlükler için ana tablodur. Sembol tablolarından farklı olarak, yeni sözlükler oluşturulabilir ve bu sözlük adlandırılmış nesne sözlüğüne eklenebilir. Sözlükler çizim varlıklarını içeremez. Ancak çizim varlıkları sözlüklerde `Handle`'ları  (değişmez kimlikleri) aracılığıyla saklanabilir.

### Koleksiyon Nesneleri

AutoCAD, çoğu grafiksel ve grafiksel olmayan nesneyi koleksiyonlar veya depolama (container) nesneleri halinde gruplandırır. Bu nesneler farklı türde veriler içermesine rağmen, kullanımlarını ve öğrenmelerini kolaylaştırmak için ortak yöntem ve özellikleri içerir. `Count` özelliği ve `Item` işlevi bunların bir örneğidir. 

// item, count örneği

AutoCAD .NET API'sindeki koleksiyon üyelerine aşağadakiler örnek olarak verilebilir: 

- Katmanlar sembol tablosundaki katman tablosu kaydı A
- CAD_LAYOUT sözlüğündeki Layout
- DocumentCollection'daki Document
- Bir blok referansındaki nitelikler (atttibutes)
