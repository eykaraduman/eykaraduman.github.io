---
title: "Blok Tabloları ve Blok Tablo Kayıtları"
permalink: /autocadnet/beginnertutorial/block-tables/
excerpt: ""
toc: true
---

AutoCAD çizim dosyası, görünür (grafik) ve görünmez (grafik olmayan) nesnelere ait tablo ve kayıtları içeren bir veritabanı dosyasıdır.Çizim dosyasında bir BlockTable (blok tablosu) ve birçok BlockTableRecords (blok tablo kayıtları) bulunur. ModelSpace, PaperSpace ve Layout'lar gerçekte birer blok tablo kaydıdır. Model uzayında  çizim yapmak istendiğinde ModelSpace blok tablo kaydı (BTR), kağıt uzayında çizim yapmak istendiğinde ise PaperSpace blok kaydı kullanılır.

Yeni oluşturulan bir AutoCAD .dwg dosyası, varsayılan olarak bir adet ModelSpace ve iki adet de PaperSpace blok tablo kaydı içerir. Bunları AutoCAD komut satırına yazdıran ShowBlockTableRecords() yordamı verilmiştir.

//ShowBlockTableRecords() yordamı

// ShowBlockTableRecords() komutu

// ShowBlockTableRecords() komutu ekran görüntüsü

