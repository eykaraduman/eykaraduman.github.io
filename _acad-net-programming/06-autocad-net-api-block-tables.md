---
title: "Blok Tabloları ve Blok Tablo Kayıtları"
permalink: /autocad-net-programming/autocad-netblock-tables/
classes: wide
comments: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

AutoCAD çizim dosyası, görünür (graﬁk) ve görünmez (graﬁk olmayan) nesnelere ait tablo ve kayıtları içeren bir veritabanı dosyasıdır. Çizim dosyasında bir `BlockTable` (blok tablosu) ve birçok `BlockTableRecords`
(blok tablo kayıtları) bulunur. ModelSpace, PaperSpace ve Layout’lar gerçekte birer blok tablo kaydıdır. Model
uzayında çizim yapmak için ModelSpace, kağıt uzayında çizim yapmak için ise PaperSpace blok kaydı kullanılır.

Yeni oluşturulan bir AutoCAD .dwg dosyası, varsayılan olarak bir adet `ModelSpace` ve iki adet de `PaperSpace`
blok tablo kaydı içerir. Bunları AutoCAD komut satırına yazdıran `ShowBlockTableRecords` yordamı aşağıda verilmiştir.

```c#
[CommandMethod("ShowBlockTableRecords")]
public void ShowBlockTableRecords()
{
	// 1. Autocad aktif belgesine erişim  
	Document doc = Autodesk.AutoCAD.ApplicationServices.Core.Application.DocumentManager.MdiActiveDocument;
	// 2. İşlem yığının başlatılması  
	using (Transaction transaction = doc.TransactionManager.StartTransaction())
	{
		// 3. Belge veritabanının okuma amaçlı açılması  
		BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
		// 4. Blok tablo kayıtlarını okuma amaçlı açılması  
		foreach (var blockTableRecordId in blockTable)
		{
			BlockTableRecord blockTableRecord = 
				(BlockTableRecord)transaction.GetObject(blockTableRecordId, OpenMode.ForRead);
			// 5. Blok tablo kayıt adının komut satırını yazdırılması
			doc.Editor.WriteMessage($"\n{blockTableRecord.Name}");
		}
	}
}
```

*ShowBlockTableRecords* AutoCAD komutu, *Model_Space, *Paper_Space ve *Paper_Space0 blok tablo
kayıtlarını komut satırına sırasıyla yazacaktır.

Aşağıda verilen **`NumberOfInsertBtrsInModelSpace`** yordamı, AutoCAD `ModelSpace` blok tablo kayıtlarına nasıl ulaşılacağının bir örneğini göstermektedir.

```c#
[CommandMethod("NumberOfInsertBtrsInModelSpace")]
public static void NumberOfInsertBtrsInModelSpace()
{
	// 1. Autocad aktif belgesine erişim  
	Document doc = Application.DocumentManager.MdiActiveDocument;
	// 2. Autocad aktif veritabanına erişim  
	Database db = doc.Database;
	// 3. İşlem yığının başlatılması
	using (Transaction tr = db.TransactionManager.StartTransaction())
	{
		// 4. Blok tablosunun okuma amaçlı açılması
		BlockTable blkTable = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead);
		// 5. ModelSpace blok tablosu kaydına okuma amaçlı erişim
		BlockTableRecord btRecord =
			(BlockTableRecord)tr.GetObject(blkTable[BlockTableRecord.ModelSpace], OpenMode.ForRead);

		int objectCount = 0;
		// 6. Tek tek ModelSpace kayıtlarına erişim
		foreach (ObjectId objectId in btRecord)
		{
			if(objectId.ObjectClass.DxfName == "INSERT")
				objectCount++;
		}
		doc.Editor.WriteMessage($"\nÇizimdeki blok referans sayısı: {objectCount}");
		tr.Commit();
	}
}
```

*NumberOfInsertBtrsInModelSpace* komutu, [Blocks and Tables (Metric)](https://download.autodesk.com/us/samplefiles/acad/blocks_and_tables_-_metric.dwg) çizim dosyasında çalıştırıldığında, blok referans nesne sayısını komut satırına 161 olarak yazdıracaktır. 

<figure style="width: 550px">
  <img src="{{ '/assets/images/count-of-block-reference.png' | relative_url }}" alt="count of block references">
  <figcaption>ModelSapace blok tablo kaydı içindeki blok referans nesnelerinin sayılması.</figcaption>
</figure>
