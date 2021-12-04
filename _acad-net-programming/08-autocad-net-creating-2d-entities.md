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
		BlockTable bt = (BlockTable)db.BlockTableId.GetObject(OpenMode.ForRead);
		BlockTableRecord btr =
			(BlockTableRecord) doc.TransactionManager.GetObject(bt[BlockTableRecord.ModelSpace],
				OpenMode.ForWrite);
	
		// Çizgi nesnesinin oluşturulması
		Point3d startPnt = new Point3d(1.0, 0.0, 0.0);
		Point3d endPnt = new Point3d(4.0, 0.0, 0.0);
		Line line= new Line(startPnt, endPnt);
	
		// Çizginin blok tablo kaydına eklenmesi
		btr.AppendEntity(line);
		// Çizginin işlme yığınına eklenmesi
		tr.AddNewlyCreatedDBObject(line, true);
		// İşlem yığınının onaylanması
		tr.Commit();
	} // İşlem yığınının sonlandırılması
}
{% endhighlight %}

16-19 numaralı kod satırları çizgiyi oluşturulmakta ve 21-26 numaralı kod satırları ise çizgiyi çizim veritabanına eklemektedir.

*DrawLine1* yordamında aşağıdaki noktaları gözden kaçırmamak çizim veritabanına varlık eklemenin mekanizmasını anlamak açısından önemlidir:

- Blok tablosu ve blok tablo kaydına erişirken `GetObject` yordamı kullanılmıştır.
- ModelSpace blok tablo kaydına yazma amaçlı  erişilmiştir. Ancak okuma amaçlı erişilmiş blok tablo kaydına her zaman `btr.UpgradeOpen()` yordamını kullanarak yazma amaçlı erişmek de mümkündür.
- Oluşturulan `Line` nesnesi sonlandırılmamıştır. Çünkü sonlandırma işini işlem yığını üstlenmiştir. (`line.Dispose();` satırını koda eklemek hatayla sonuçlanacaktır.)

Diğer AutoCAD varlıklarının çizim veritabanına eklenmesinde izlenecek yol, `DrawLine1` yordamında izlenenle aynı olacaktır. Bu nedenle, kod tekrarından kaçınmak için varlıkları parametre olarak kabul eden bir yordamın hazırlanması bize hem zaman kazandıracak hem de kolaylık sağlayacaktır. Statik `DatabaseHelper` sınıfı ModelSpace ve PaperSpace blok tablo kayıtlarına varlıkların eklenmesini kolaylaştırmak için tasarlanmıştır.

.**DatabaseHelper.cs** :

```c#
using System.Collections.Generic;
using System.Linq;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace PgAutoCAD
{
    /// <summary>
    /// AutoCAD uzayları numaralandırması.
    /// </summary>
    public enum AcadSpace
    {
        Current, Model, Paper
    }

    /// <summary>
    /// Veritabanı işlemleri için yardımcı.
    /// </summary>
    public static class DatabaseHelper
    {
        /// <summary>
        /// Model uzayı kimliğini verir.
        /// </summary>
        /// <param name="db">Veritabanı.</param>
        /// <returns></returns>
        public static ObjectId ModelSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockModelSpaceId(db);
        }

        /// <summary>
        /// Kağıt uzayı kimliğini verir.
        /// </summary>
        /// <param name="db">Veritabanı</param>
        /// <returns></returns>
        public static ObjectId PaperSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockPaperSpaceId(db);
        }

        /// <summary>
        /// Mevcut uzayın kağıt uzayı olup olmadığını kontrol eder.
        /// </summary>
        /// <param name="db">Veritabanı.</param>
        /// <returns></returns>
        public static bool IsPaperSpace(this Database db)
        {
            if (db.TileMode)
                return false;

            Editor ed = Active.Editor;
            if (db.PaperSpaceVportId == ed.CurrentViewportObjectId)
                return true;

            return false;
        }

        /// <summary>
        /// Mevcut uzayın model uzayı olup olmadığını kontrol eder.
        /// </summary>
        /// <param name="db">Veritabanı</param>
        /// <returns></returns>
        public static bool IsModelSpace(this Database db)
        {
            return !IsPaperSpace(db);
        }

        /// <summary>
        /// Seçilen uzaya varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="space">Varlığın ekleneceği uzay.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimliği.</returns>
        internal static ObjectId AddToSpace(this Entity entity, AcadSpace space, Database db = null)
        {
            db = db ?? HostApplicationServices.WorkingDatabase;

            ObjectId spaceId;
            switch (space)
            {
                case AcadSpace.Current:
                    spaceId = db.CurrentSpaceId;
                    break;
                case AcadSpace.Model:
                    spaceId = db.ModelSpaceId();
                    break;
                case AcadSpace.Paper:
                    spaceId = db.PaperSpaceId();
                    break;
                default:
                    spaceId = db.CurrentSpaceId;
                    break;
            }

            using (var trans = db.TransactionManager.StartTransaction())
            {
                var currentSpace = (BlockTableRecord)trans.GetObject(spaceId, OpenMode.ForWrite, false);
                var id = currentSpace.AppendEntity(entity);
                trans.AddNewlyCreatedDBObject(entity, true);
                trans.Commit();
                return id;
            }
        }

        /// <summary>
        /// Seçilen uzaya varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="space">Varlıkların ekleneceği uzay.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesnelerin kimlikleri.</returns>
        internal static ObjectId[] AddToSpace(this IEnumerable<Entity> entities, AcadSpace space, Database db = null)
        {
            db = db ?? HostApplicationServices.WorkingDatabase;

            ObjectId spaceId = ObjectId.Null;
            switch (space)
            {
                case AcadSpace.Current:
                    spaceId = db.CurrentSpaceId;
                    break;
                case AcadSpace.Model:
                    spaceId = db.ModelSpaceId();
                    break;
                case AcadSpace.Paper:
                    spaceId = db.PaperSpaceId();
                    break;
                default:
                    spaceId = db.CurrentSpaceId;
                    break;
            }
            using (var trans = db.TransactionManager.StartTransaction())
            {
                var currentSpace = (BlockTableRecord)trans.GetObject(spaceId, OpenMode.ForWrite, false);
                var ids = entities
                    .ToArray()
                    .Select(entity =>
                    {
                        var id = currentSpace.AppendEntity(entity);
                        trans.AddNewlyCreatedDBObject(entity, true);
                        return id;
                    })
                    .ToArray();

                trans.Commit();
                return ids;
            }
        }

        /// <summary>
        /// Açık olan uzaya varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimliği.</returns>
        public static ObjectId AddToCurrentSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Current, db);
        }

        /// <summary>
        /// Açık olan uzaya varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToCurrentSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Current, db);
        }

        /// <summary>
        /// Model uzayına varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId AddToModelSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Model, db);
        }

        /// <summary>
        /// Model uzayına varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToModelSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Model, db);
        }

        /// <summary>
        /// Kağıt uzayına varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId AddToPaperSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Paper, db);
        }

        /// <summary>
        /// Kağıt uzayına varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToPaperSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Paper, db);
        }
    }
}
```

`DrawLine1` yordamı artık daha sade bir biçimde aşağıdaki gibi yazılabilir.

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

