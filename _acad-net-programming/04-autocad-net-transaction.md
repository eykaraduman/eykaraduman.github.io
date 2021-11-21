---
title: İşlem Yığınları (Transactions)
excerpt: "İşlem yığını içsel süreçlerinde oluşacak bir hata işlem bütünlüğünün geri sarılmasını (rollback), bu süreçlerin başarıyla tamamlanması ise bu bütünlüğün onaylanmasını (commit) gerektirir."
permalink: /autocad-net-programming/autocad-net-transactions/
classes: wide
comments: true
toc: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

### İşlem Yığını Nedir?

**İşlem Yığını (Transaction)**, veri tabanı işlemlerinde işlemsel bir bütünlüğü temsil eden ve daha küçük parçalara ayrılamayan en küçük birimi olarak tanımlanabilir. Bu tanımı gereği de her işlem yığını atomsal, tutarlı, yalıtılmış ve sürekli olmalıdır. 

İşlem yığını içsel süreçlerinde oluşacak bir hata işlem bütünlüğünün geri sarılmasını (rollback), bu süreçlerin başarıyla tamamlanması ise bu bütünlüğün onaylanmasını (commit) gerektirir.

Her AutoCAD dosyası (dwg) grafik ve grafik olmayan bilgilerin ya da nesnelerin tanımlarını içeren bir veritabanı dosyası olduğundan yukarıda anılan tüm öğeleri içerir. AutoCAD .NET programlamada da işlem yığını (transaction) diğer veritabanlarında olduğu gibi veritabanı işlemlerinin doğru sıra ve yetkiyle yapılıp yapılmadığını kontrol etmek için kullanılır.

Basit bir AutoCAD .NET işlem yığınının işleyişi aşağıdaki gibidir:

1. İşlem yığının başlatılması (StartTransaction)
2. Veritabanı okuma/yazma işlemlerinin gerçekleştirilmesi (Nesne ekleme/silme gibi)
3. Eğer bir hata oluşmazsa işlem yığınının onaylanması (Commit)
4. Eğer bir hata oluşursa işlem yığınının iptal edilmesi (Abort)

AutoCAD .NET programlamada kullanılan **`TransactionManager`** ve **`Transaction`** sınıfları `Autodesk.AutoCAD.DatabaseServices` isim uzayında bulunmaktadır. İşlem yığını örneklerini vermeden önce uzun değişken bildirimlerinin önüne geçmek için aşağıdaki isim uzaylarını projemize eklemek faydalı olacaktır.

```c#
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.Geometry;
using Autodesk.AutoCAD.Runtime;
```

AutoCAD .NET API, işlem yığınını, AutoCAD `DBObject` ve `Entity` örneklerini, açma, okuma, yazma vb. amaçlarla işlemek için ana mekanizma olarak sağlamaktadır. Yeni bir nesne oluştururken bu mekanizmanın işleyişi ana hatlarıyla aşağıdaki gibidir:

- İşlem yığın yöneticisine erişim ve işlem yığınının başlatılması (StartTransaction)
- Model ve kağıt çizim uzaylarını barındıran blok tablosuna okuma amaçlı erişilmesi (BlockTable)
- Üzerinde çalışılacak blok tablo kaydına yazma amaçlı açılması (BlockTableRecord)
- Yeni nesnenin istenilen özelliklerle oluşturulması (Line, Circle vb.)
-  Oluşturulan nesnenin işlem yığınına eklenmesi (AddNewlyCreatedDBObject)
- İşlem yığınının onaylanması (Commit)
- Sırasıyla işlem yığını ve işlem yığın yöneticisinin sonlandırılması (Dispose)

### `using` Anahtar Sözcüğü Olmaksızın İşlem Yığını Kullanımı

İlk örnek, .NET Framework `using` anahtar sözcüğü kullanılmadan işlem yığını mekanizmasının nasıl çalıştığını göstermektedir. Bu durumda, `Dispose()` yordamı, işlem yığını ve işlem yığını yöneticisini sonlandırmak için açıkça çağrılmalıdır.

```csharp
[CommandMethod("TransactionWithoutUsing")]
static public void TransactionSampleWithoutUsingKeyword()
{
    // 1. Autocad aktif belgesine erişim  
    Document doc = Application.DocumentManager.MdiActiveDocument;
    // 2. Aktif belgeye ait işlem yığın yöneticisine erişim  
    Autodesk.AutoCAD.DatabaseServices.TransactionManager transactionManager =
    Application.DocumentManager.MdiActiveDocument.TransactionManager;
    // 3. İşlem yığının başlatılması  
    Transaction transaction = transactionManager.StartTransaction();
    // 4. Belge veritabanının/blok tablosunun okuma amaçlı açılması  
    BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
    // 5. Model uzayının (modelspace) yazma amaçlı açılması  
    BlockTableRecord blockTableRecord =
    (BlockTableRecord)transactionManager.
    GetObject(blockTable[BlockTableRecord.ModelSpace], OpenMode.ForWrite);
    // 6. Autocad daire nesnesinin oluşturulması  
    Circle circle = new Circle();
    circle.Center = new Point3d(0.0, 0.0, 0.0);
    circle.Radius = 10.0;
    // 7. Model uzayı blok tablo kaydına daire nesnesinin eklenmesi  
    ObjectId circleId = blockTableRecord.AppendEntity(circle);
    // 8. Daire nesnesinin işlem yığınına eklenmesi  
    transaction.AddNewlyCreatedDBObject(circle, true);
    // 9. İşlem yığınının onaylanması  
    transaction.Commit();
    // 10. Sırasıyla işlem yığını ve işlem yığın yöneticisinin sonlandırılması  
    transaction.Dispose();
    transactionManager.Dispose();
}
```

### `using` Anahtar Sözcüğü ile İşlem Yığını Kullanımı

İkinci örnek ise  `using` anahtar sözcüğü kullanılarak işlem yığını mekanizmasının nasıl çalıştığını göstermektedir. `Dispose()` yordamını, işlem yığını ve işlem yığını yöneticisini sonlandırmak için çağırmaya gerek yoktur.  Çünkü `using` anahtar sözcüğü bu görevi üstlenmektedir.

```csharp
[CommandMethod("TransactionWithUsing")]
public void TransactionSampleWithUsingKeyword()
{
    // 1. Autocad aktif belgesine erişim  
    Document doc = Application.DocumentManager.MdiActiveDocument;
    // 2. İşlem yığının başlatılması  
    Transaction transaction =
    Application.DocumentManager.MdiActiveDocument.TransactionManager.StartTransaction();
    using (transaction)
    {
        // 3. Belge veritabanının/blok tablosunun okuma amaçlı açılması 
        BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
        // 4. Model uzayının (modelspace) yazma amaçlı açılması  
        BlockTableRecord blockTableRecord =
        (BlockTableRecord)Application.DocumentManager.MdiActiveDocument.
        TransactionManager.GetObject(blockTable[BlockTableRecord.ModelSpace],
        OpenMode.ForWrite);
        // 5. Autocad daire nesnesinin oluşturulması  
        Circle circle = new Circle();
        circle.Center = new Point3d(0.0, 0.0, 0.0);
        circle.Radius = 10.0;
        // 8. Model uzayı blok tablo kaydına daire nesnesinin eklenmesi  
        ObjectId circleId = blockTableRecord.AppendEntity(circle);
        // 9. Daire nesnesinin işlem yığınına eklenmesi  
        transaction.AddNewlyCreatedDBObject(circle, true);
        // 10. İşlem yığınının onaylanması  
        transaction.Commit();
    }
}
```

Yukardaki örneklerden de kolayca anlaşılacağı üzere, okuma ya da yazma amaçlı tüm AutoCAD veritabanı işlemlerinde, işlem yığınlarının (transactions) kullanılmasının zorunlu olduğudur. Veritabanı okuma amaçlı erişimlerinin yazma amaçlı erişimlerden farkı ise işlem yığınının onaylanması gerekliliğinin bulunmamasıdır.

### İşlem Yığını Kullanımında Dikkat Edilmesi Gerekenler

AutoCAD .NET içerisinde IDisposable nesnelerle (`DbObject`, `Transaction` gibi) çalışırken dikkatli olmak gereklidir. Çünkü nesneleri sonlandırma işleminin, .NET Framework gereksiz nesne toplayıcısına (garbage collector) hangi durumlarda bırakılmaması gerektiğini bilmek önemlidir. AutoCAD nesne türlerine göre aşağıdaki önerilere uyulması gerekmektedir.

- İlk olarak, veritabanında kalıcı olmayan ya da olmayacak, `DBObject` nesnesinden türememiş geçici nesneler kodla sonlandırılabileceği gibi, bu nesneleri  .NET Framework  gereksiz nesne toplayıcısının sonlandırmasına da izin verilebilir.
- İkinci olarak, `DBObject` nesnesinden türemiş geçici (vertitabanına eklenmesi düşünülmeyen) nesneler .NET Framework gereksiz nesne toplayıcısına bırakılmaksızın mutlaka kodla sonlandırılmalıdır.
- Üçüncü ve son olaraksa, veritabanında kalıcı olan ve işlem yığını tarafından yönetilen (`AddNewlyCreatedDBObject()` yada `GetObject()` aracılığıyla sahiplenilmiş) nesneler, işlem yığını yöneticisi sonlandırıldığında sonlandırılacağı için ayrıca kodla sonlandırılmalarına gerek yoktur.

### İşlem Yığını Performansının Araştırılması

İşlem yığını performansının gözlemenin en güzel yollarından biri, çok sayıda nesneyi farklı işlem yığınları ve tek bir işlem yığını içinde oluşturup geçen süreleri karşılaştırmaktır. 

Aşağıdaki kod örneğinde, `CreateLine(...)` yordamı ile her bir işlem yığına eklenen bir adet çizgi oluşturmaktadır. `CreateLines1()` yordamıyla ise `CreateLines(...)` yordamı binlerce kez çağrılmaktadır. 

```c#
public void CreateLine(Point3d startPoint, Point3d endPoint)
{
	Document doc = Application.DocumentManager.MdiActiveDocument;

	using (Transaction transaction = doc.TransactionManager.StartTransaction())
	{
		BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
		BlockTableRecord blockTableRecord =
			(BlockTableRecord)Application.DocumentManager.MdiActiveDocument.
				TransactionManager.GetObject(blockTable[BlockTableRecord.ModelSpace], OpenMode.ForWrite);

		Line ln = new Line(startPoint, endPoint);
		ln.SetDatabaseDefaults();
		blockTableRecord.AppendEntity(ln);
		transaction.AddNewlyCreatedDBObject(ln, true);
		transaction.Commit();
	}
}

[CommandMethod("CreatLines1")]
public void CreateLines1()
{
	Document doc = Application.DocumentManager.MdiActiveDocument;

	DateTime starTime = DateTime.Now;
	for (int i = 0; i < 5000; i++)
	{
		CreateLine(new Point3d(0.0, i * 100, 0.0), new Point3d(100.0, i * 100, 0.0));
	}

	TimeSpan elapsedTime = DateTime.Now.Subtract(starTime);
	doc.Editor.WriteMessage($"\nCreateLines1() yordamının çalışmasında geçen zaman: " +
							$"{elapsedTime.TotalMilliseconds}");
}
```

İkinci kod örneği ise, `CreateLines2()`  yordamıyla tek bir işlem yığına eklenen binlerce çizgi oluşturmaktadır.

```c#
[CommandMethod("CreatLines2")]
public void CreateLines2()
{
	Document doc = Application.DocumentManager.MdiActiveDocument;

	DateTime starTime = DateTime.Now;
	using (Transaction transaction = doc.TransactionManager.StartTransaction())
	{
		BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
		BlockTableRecord blockTableRecord =
			(BlockTableRecord)Application.DocumentManager.MdiActiveDocument.
				TransactionManager.GetObject(blockTable[BlockTableRecord.ModelSpace],  OpenMode.ForWrite);

		for (int i = 0; i < 5000; i++)
		{
			Line ln = new Line(new Point3d(0.0, i * 100, 0.0), new Point3d(100.0, i * 100, 0.0));
			ln.SetDatabaseDefaults();
			blockTableRecord.AppendEntity(ln);
			transaction.AddNewlyCreatedDBObject(ln, true);
		}
		transaction.Commit();
	}

	TimeSpan elapsedTime = DateTime.Now.Subtract(starTime);
	doc.Editor.WriteMessage($"\nCreateLines2() yordamının çalışmasında geçen zaman: " +
							$"{elapsedTime.TotalMilliseconds}");
}
```

`CreateLines1()` yordamının çalışmasında geçen zaman 1227.9627 mili saniye, `CreateLines2()` yordamının çalışmasında geçen zaman ise 81.032 mili saniyedir. İki yordam arasında ciddi bir süre farkı bulunmaktadır. Bunun nedeni ilk yordamda, her bir çizginin farklı işlem yığınlarına eklenerek işlem yığınlarının ayrı ayrı onaylanmasıdır. 

İlk yordam, kodun okunabilirliği, kullanılabilirliği ve perfomansı açısından aşağıdaki biçimde sadeleştirilebilir.

```c#
public void CreateLine(Document doc, Transaction transaction, Point3d startPoint, Point3d endPoint)
{
	BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
	BlockTableRecord blockTableRecord =
		(BlockTableRecord)doc.TransactionManager.GetObject(blockTable[BlockTableRecord.ModelSpace], OpenMode.ForWrite);

	Line ln = new Line(startPoint, endPoint);
	ln.SetDatabaseDefaults();
	blockTableRecord.AppendEntity(ln);
	transaction.AddNewlyCreatedDBObject(ln, true);
}

[CommandMethod("CreatLines3")]
public void CreateLines3()
{
	Document doc = Application.DocumentManager.MdiActiveDocument;
	using (Transaction transaction = doc.TransactionManager.StartTransaction())
	{
		for (int i = 0; i < 5000; i++)
		{
			CreateLine(doc, transaction, 
				new Point3d(0.0, i * 100, 0.0), new Point3d(100.0, i * 100, 0.0));
		}
		transaction.Commit();
	}
}
```

Çizgi oluşturmakta kullanılan tüm yordamlar değerlendirildiğinde, nesnelerin tek bir işlem yığına ekleyip işlem yığınını onaylamak AutoCAD performansı açısından ciddi bir fark yaratmaktadır. 

### İç İçe İşlem Yığınları

İşlem yığınları iç içe geçebilir. Tasarlanan yordam tarafından tapılan tüm değişiklikleri geri almak için en dış, değişiklerin sadece bir kısmını geri almak için iç işlem yığınları kullanılmaktadır. İç içe işlem yığınları oluşturdukları sıranın tersine onaylanmalı ya da iptal edilmelidir.



