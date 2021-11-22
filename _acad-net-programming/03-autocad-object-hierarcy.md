---
title: "AutoCAD Nesne Hiyerarşisi"
permalink: /autocad-net-programming/autocad-object-hierarcy/
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

Nesneler, AutoCAD `Application` nesnesi en başta olacak şekilde hiyerarşik bir şekilde dizilmiştir. Bu hiyerarşik yapıya Nesne Modeli denir. Aşağıdaki gösterim, AutoCAD nesneleri arasındaki temel ilişkileri göstermektedir. AutoCAD .NET API'de burada gösterimeyen daha birçok nesne bulunmaktadır.

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