---
title:AutoCAD.Net ile Transaction Kullanımı
permalink: /acad/autocad-net-transactions/
published: true
categories:
  - AutoCAD.Net
---

**Transaction**, veri tabanı işlemlerinde işlemsel bir bütünlüğü temsil eden ve daha küçük parçalara ayrılamayan en küçük işlem yığını (transaction) olarak tanımlanabilir. Bu tanımı gereği de her işlem yığını atomsal, tutarlı, yalıtılmış ve sürekli olmalıdır. 

İşlem yığını içsel süreçlerinde oluşacak bir hata işlem bütünlüğünün geri sarılmasını (rollback), bu süreçlerin başarıyla tamamlanması ise bu bütünlüğün onaylanmasını (commit) gerektirir.

Her AutoCAD dosyası (dwg) grafik ve grafik olmayan bilgilerin ya da nesnelerin tanımlarını içeren bir veritabanı dosyasıdır. 

Bu nedenle yukarda anılan tüm öğeleri içerir. AutoCAD.Net programlamada da işlem yığını (transaction) diğer veritabanlarında olduğu gibi veritabanı işlemlerinin doğru sıra ve yetkiyle yapılıp yapılmadığını kontrol etmek için kullanılır.

Basit bir AutoCAD.Net işlem yığınının işleyişi aşağıdaki gibidir:

1. İşlem yığının başlatılması (StartTransaction)
2. Veritabanı okuma/yazma işlemlerinin gerçekleştirilmesi (Nesne ekleme/silme gibi)
3. Eğer bir hata oluşmazsa işlem yığınının onaylanması (Commit)
4. Eğer bir hata oluşursa işlem yığınının iptal edilmesi (Abort)

AutoCAD .Net programlamada kullanılan **TransactionManager** ve **Transaction** sınıfları ise `Autodesk.AutoCAD.DatabaseServices` isim uzayında bulunmaktadır. İşlem yığını örneklerini vermeden önce uzun değişken bildirimlerinin önüne geçmek için aşağıdaki isim uzaylarını projemize eklemek faydalı olacaktır.

**Örnek-1**

```csharp
[CommandMethod("TrSmp1")]
static public void TransactionSampleWithoutUsingKeyword()
{
    // 1. Autocad aktif belgesine erişim  
    Document doc = Application.DocumentManager.MdiActiveDocument;
    // 2. Aktif belgeye ait işlem yığın yöneticisine erişim  
    Autodesk.AutoCAD.DatabaseServices.TransactionManager transactionManager =
    Application.DocumentManager.MdiActiveDocument.TransactionManager;
    // 3. İşlem yığının başlatılması  
    Transaction transaction = transactionManager.StartTransaction();
    // 4. Belge veritabanının okuma amaçlı açılması  
    BlockTable blockTable = (BlockTable)doc.Database.BlockTableId.GetObject(OpenMode.ForRead);
    // 5. Model uzayının (modelspace) yazma amaçlı açılması  
    BlockTableRecord blockTableRecord =
    (BlockTableRecord)transactionManager.
    GetObject(blockTable[BlockTableRecord.ModelSpace], OpenMode.ForWrite);
    // 6. Autocad daire nesnesinin oluşturulması  
    Circle circle = new Circle();
    circle.Center = new Point3d(0.0, 0.0, 0.0);
    circle.Radius = 10.0;
    // 7. Model uzayı blok tablosuna daire nesnesinin eklenmesi  
    ObjectId circleId = blockTableRecord.AppendEntity(circle);
    // 8. Daire nesnesinin işlem yığınına eklenmesi  
    transaction.AddNewlyCreatedDBObject(circle, true);
    // 9. İşlem yığınının onaylanması  
    transaction.Commit();
    // 10. Sırasıyla işlem yığınını ve işlem yığın yöneticisini sonlandırılması  
    transaction.Dispose();
    transactionManager.Dispose();
}
```

**Örnek-2**

```csharp
static public void TransactionSampleWithUsingKeyword()
{
    // 1. Autocad aktif belgesine erişim  
    Document doc = Application.DocumentManager.MdiActiveDocument;
    // 2. İşlem yığının başlatılması  
    Transaction transaction =
    Application.DocumentManager.MdiActiveDocument.TransactionManager.StartTransaction();
    using (transaction)
    {
        // 3. Belge veritabanının okuma amaçlı açılması  
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
        // 8. Model uzayı blok tablosuna daire nesnesinin eklenmesi  
        ObjectId circleId = blockTableRecord.AppendEntity(circle);
        // 9. Daire nesnesinin işlem yığınına eklenmesi  
        transaction.AddNewlyCreatedDBObject(circle, true);
        // 10. İşlem yığınının onaylanması  
        transaction.Commit();
    }
}
```

Yukardaki örneklerden de kolayca anlaşılacağı üzere, okuma yada yazma amaçlı tüm Autocad veritabanı işlemlerinde, işlem yığınlarının (transactions) kullanılmasının zorunlu olduğudur. Veritabanı okuma amaçlı erişimlerinin yazma amaçlı erişimlerden farkı ise işlem yığınının onaylanması gerekliliğinin olmamasıdır.

İşlem yığınını sonlandırma [`Dispose()`] fonksiyonunun kullanımı ile ilgili birkaç önemli noktaya değinerek bu konuyla ilgili söyleyeceklerimi bitirmek istiyorum. 

AutoCAD.Net içerisinde IDisposable nesnelerle (`DbObject`, `Transaction` gibi) çalışırken dikkatli olmak gerekir. Çünkü nesneleri sonlandırma işleminin, .Net gereksiz nesne toplayıcısına (garbage collector) hangi durumlarda bırakılmaması gerektiğini bilmek önemlidir. AutoCAD nesne türlerine göre aşağıdaki önerilere uyulması gerekmektedir.

- İlk olarak, veritabanında kalıcı olmayan ya da olmayacak, `DBObject` nesnesinden türememiş geçici nesneler kodla sonlandırılabileceği gibi, bu nesneleri .Net gereksiz nesne toplayıcısının sonlandırmasına da izin verilebilir.
- İkinci olarak, `DBObject` nesnesinden türemiş geçici (vertitabanına eklenmesi düşünülmeyen) nesneler .Net gereksiz nesne toplayıcısına bırakılmaksızın mutlaka kodla sonlandırılmalıdır.
- Üçüncü ve son olaraksa, veritabanında kalıcı olan ve işlem yığını tarafından yönetilen (`AddNewlyCreatedDBObject()` yada `GetObject()` aracılığıyla sahiplenilmiş) nesneler, işlem yığını yöneticisi sonlandırıldığında sonlandırılacağı için ayrıca kodla sonlandırılmalarına gerek yoktur.
