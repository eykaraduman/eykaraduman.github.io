---
title: "2B (2D) Varlıkların (Entities) Oluşturulması"
permalink: /autocad-net-programming/creating-2d-entities/
classes: wide
comments: true
published: true
toc: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

[İşlem Yığınları](/autocad-net-programming/autocad-net-transactions/) ve [Dönüşümler](/autocad-net-programming/transformations/) adlı bölümlerde , ModelSpace'e nasıl çizim yapılacağına dair basit örnekler vermiştik. 2B varlıkların oluşturulmasına geçmeden önce, ilerleyen kısımlarda sıkça kullanacağımız `Point3d` ve `Vector3d` yapılarını tanımak faydalı olacaktır. Bu yapılar`Autodesk.AutoCAD.Geometry` isim uzayında yer almaktadır.

### Çok Kullanılan Geometri Yapıları

#### Point3d

Pointd3d yapısı, AutoCAD varlıkları yaratılırken yaygın olarak kullanılan basit bir nesnedir. Üç farklı kurucuya sahiptir.

```c#
public Point3d(PlanarEntity plane, Point2d point);
public Point3d(double[] xyz);
public Point3d(double x, double y, double z); // x,y ve z koordinatları ile 3B bir nokta oluşturur
```

Bir çok AutoCAD varlığının Point3d türünde özellikleri bulunmaktadır. `Line` nesnesinin başlangıç ve bitiş noktaları bunun örneğidir. Aşağıdaki kod parçası, üç birim uzunluğunda yatay bir çizgi oluşturacaktır. Çizginin görünebilmesi için daha önce gösterildiği gibi çizginin, ModelSpace ya da PaperSpace blok tablo kaydına eklenmesi gereklidir.

```c#
Point3d pntA = new Point3d(1.0, 0.0, 0.0);
Point3d pntB = new Point3d(4.0, 0.0, 0.0);
Line line = new Line(pntA, pntB);
```

#### Vector3d

Vector3d yapısı, doğrultu işlemleri için kullanılır. Kurucuları Point3d ile benzerdir.

```c#
public Vector3d(PlanarEntity plane, Vector2d vector2d);
public Vector3d(double[] xyz);
public Vector3d(double x, double y, double z);
```

Bu yapının `kXAxis` özelliği, (1, 0, 0)  vektörünü göstermektedir.

### Çizgi (Line) Oluşturulması

{% highlight csharp linenos %}
[CommandMethod("DrawLine1")]
public void DrawLine1()
{
	// Aktif doküman ve veritabanı erişim
	Document doc = Active.Document;
	Database db = Active.Database;

	// İşlem yığınının başlatılması
	using (Transaction tr = doc.TransactionManager.StartTransaction())
	{
		// Blok tablosunun okuma amaçlı ve
		// ModelSpace blok tablosu kaydının ise yazma amaçlı açılması 
		BlockTable bt = (BlockTable) db.BlockTableId.GetObject(OpenMode.ForRead);
		BlockTableRecord btr =
			(BlockTableRecord) doc.TransactionManager.GetObject(bt[BlockTableRecord.ModelSpace],
				OpenMode.ForWrite);
	
		// Çizgi nesnesinin oluşturulması
		Point3d startPnt = new Point3d(1.0, 0.0, 0.0);
		Point3d endPnt = new Point3d(4.0, 0.0, 0.0);
		Line line= new Line(startPnt, endPnt);
	
		// Çizginin blok tablo kaydına eklenmesi
		 ObjectId lineId = btr.AppendEntity(line);
		// Çizginin işlme yığınına eklenmesi
		tr.AddNewlyCreatedDBObject(line, true);
		// İşlem yığınının onaylanması
		tr.Commit();
	} // İşlem yığınının sonlandırılması
}
{% endhighlight %}

19-21 numaralı kod satırları çizgiyi oluşturulmakta ve 23-28 numaralı kod satırları ise çizgiyi çizim veritabanına eklemektedir.

`DrawLine1` yordamı, çizim veritabanına varlık eklemenin işleyişini anlamak açısından önemlidir:

- Blok tablosu ve blok tablo kaydına erişirken `GetObject` yordamı kullanılmıştır. Bu yordam `DBObject` türünde nesneler döndürür. Bu nesnelerin uygun tür dönüşümü yapılarak istenilen nesneler erişilmektedir. (Satır 13 ve 15.)
- ModelSpace blok tablo kaydına yazma amaçlı erişilmiştir. Ancak okuma amaçlı erişilmiş blok tablo kaydına her zaman `btr.UpgradeOpen()` yordamını kullanarak yazma amaçlı erişmek de mümkündür.
- Oluşturulan `Line` nesnesi sonlandırılmamıştır. Çünkü sonlandırma işini işlem yığını üstlenmiştir. (`line.Dispose();` satırını koda eklemek hatayla sonuçlanacaktır.)
- `AppendEntity` yordamı `ObjectId` türünde bir nesne döndürür. AutoCAD çiziminde her nesnenin, grafiksel olsun ya da olmasın, mutlaka benzersiz bir kimliği (ObjectId) vardir. Bu kimlikle nesnenin hangi veritabanına ait olduğu, geçerli olup olmadığı gibi bilgilere ulaşılabilmektedir. `ObjectId` için unutlmaması gereken en önemli şey, çizim kapatılıp açıldığında, nesne özelliklerinde değişiklik yapıldığında değerinin değişebileceğidir.

Diğer AutoCAD varlıklarının çizim veritabanına eklenmesinde izlenecek yol, `DrawLine1` yordamında izlenenle aynı olacaktır. Kılavuzun ilerleyen bölümlerinde kod tekrarından kaçınmak ve veritabanına varlık ekleme işlemlerini kolaylaştırmak için statik `DatabaseHelper` sınıfı tasarlanmıştır. (Bkz. [EK-2: AutoCAD Veritabanı Yardımcısı)](/autocad-net-programming/databese-helper/)

`DatabaseHelper` `AddToModelSpace` yordamının yardımı ile `DrawLine1` artık daha sade bir biçimde aşağıdaki gibi yazılabilir.

```c#
[CommandMethod("DrawLine1Yeni")]
public void DrawLine1Yeni()
{
	Point3d startPnt = new Point3d(1.0, 0.0, 0.0);
	Point3d endPnt = new Point3d(4.0, 0.0, 0.0);
	Line line = new Line(startPnt, endPnt);
	line.AddToModelSpace();
}
```

