---
title: Veri Depolama
permalink: /acad/autocad-net-store-data/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD .NET API, AutoCAD içinde veri depolamak için önemli araçlar içerir. Bunlar **NOD**’lar, **ExtensionDictionary**’ler ve  **XData**’lar olarak adlandırılmıştır. 
NOD kayıtları, DBDictionary nesnesinden türemiş sözlükleri, ExtensionDictionary ve XData’larsa **DBObject** nesnesinin iki önemli özelliğini temsil eder. ExtensionDictionary bir **DBDictionary** nesne kimliği (ObectId), XData ise bir **ResultBuffer** nesnesidir. 
Bu araçlar ile AutoCAD nesneleri üzerinde kısmi bir özgürlük kazanırız; istediğimiz bilgileri nesnelere ekleyebilir, silebilir, güncelleyebilir hatta nesne kimliklerini (ObjectId) birbirine bağlayabiliriz. 

- Extended Entity Data -  **XData** (Genişletilmiş nesne verisi, AutoCAD varlıklarına iliştirilir) 
- Named Object Dictionary - **NOD** (Adlandırılmış nesne sözlüğü) 
- Extension Entity Dictionary – **ExtensionDictionary** (Genişletilmiş nesne sözlüğü, AutoCAD varlıklarına iliştirilir) 

XData’nın kullanımı diğerlerine göre daha kolay olmakla beraber, büyük verilerin depolanması için en uygun olanları, NOD’lar ve ExtensionDictionary’lerdir. Her AutoCAD grafik varlığı XData özelliğinde en fazla 16 Kb, 2 Gb veri kapasiteli XRecord nesneleriyle kurulabilen ExtensionDictionary’lerdeyse büyük miktarlarda veri depolayabilir. 
Bilindiği gibi, yerleşik veritabanı nesneleriyle bu nesnelere ait anahtar adlarını ikili bir düzende saklamaya yarayan haritalar, bağlı listeler (Linked List) olarak tanımlanabilecek AutoCAD sözlükleri, veri depolamada önemli bir rol oynamaktadır. Sembol tablolarında olduğu gibi sözlüklerde de kayıt anahtarları benzersiz (unique) olmak zorundadır; bir sözlük aynı adda iki kayıt anahtarı içeremez. Ayrıca sözlüklerde depolanabilecek kimliklerin (ObjectId) ait olduğu nesne türlerinde de önemli bir sınırlama vardır: sözlükler, sadece DBObject sınıfından ya da bu sınıftan türemiş nesnelere ait kimlikleri içerebilir. 

Veri depolama yöntemlerini, IPEProfile adlı bir C# sınıfı üzerinden anlatacağız. Bu sınıfın Draw() metodu sırasıyla şu işlemleri gerçekleştirecektir: 

1. ŞEKİL-1’de gördüğünüz IPE çelik profiline ait bilgileri kullanıcıdan alacak 
2. Profil çizim noktalarını hesaplayarak bir Polyline nesnesi oluşturacak ve 
3. Profil bilgilerini, “ABC_IEP_PROFILE“ anahtarını kullanarak depolayacak. 

**Draw()** metodu kod listesi: 

```c#
public void Draw(DataStoreType StoreType)  
 {  
      if (!GetData()) return;  
      #region Noktaların hesaplanması  
      Point2d Pnt1 = new Point2d(LeftLowerPnt.X, LeftLowerPnt.Y) + b * Vector2d.XAxis;  
      Point2d Pnt2 = Pnt1 + t * Vector2d.YAxis;  
      Point2d Pnt3 = Pnt2 - 0.5 * (b - s1) * Vector2d.XAxis;  
      Point2d Pnt4 = Pnt3 - r * (Vector2d.XAxis - Vector2d.YAxis);  
      Point2d Pnt5 = Pnt4 + d * Vector2d.YAxis;  
      Point2d Pnt6 = Pnt5 + r * (Vector2d.XAxis + Vector2d.YAxis);  
      Point2d Pnt7 = Pnt6 + 0.5 * (b - s1) * Vector2d.XAxis;  
      Point2d Pnt8 = Pnt7 + t * Vector2d.YAxis;  
      Point2d Pnt9 = Pnt8 - b * Vector2d.XAxis;  
      Point2d Pnt10 = Pnt9 - t * Vector2d.YAxis;  
      Point2d Pnt11 = Pnt10 + 0.5 * (b - s1) * Vector2d.XAxis;  
      Point2d Pnt12 = Pnt11 + r * (Vector2d.XAxis - Vector2d.YAxis);  
      Point2d Pnt13 = Pnt12 - d * Vector2d.YAxis;  
      Point2d Pnt14 = Pnt13 - r * (Vector2d.XAxis + Vector2d.YAxis);  
      Point2d Pnt15 = Pnt14 - 0.5 * (b - s1) * Vector2d.XAxis;  
      #endregion  
      double bulge = Math.Tan(0.5 * Math.PI / 4.0);  
      #region Polyline'ın oluşturulması  
      Polyline pline = new Polyline();  
      pline.SetDatabaseDefaults();  
      pline.Closed = true;  
      pline.AddVertexAt(0, new Point2d(LeftLowerPnt.X, LeftLowerPnt.Y), 0.0, 0.0, 0.0);  
      pline.AddVertexAt(1, Pnt1, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(2, Pnt2, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(3, Pnt3, -bulge, 0.0, 0.0);  
      pline.AddVertexAt(4, Pnt4, 0.00, 0.0, 0.0);  
      pline.AddVertexAt(5, Pnt5, -bulge, 0.0, 0.0);  
      pline.AddVertexAt(6, Pnt6, 0.00, 0.0, 0.0);  
      pline.AddVertexAt(7, Pnt7, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(8, Pnt8, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(9, Pnt9, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(10, Pnt10, 0.0, 0.0, 0.0);  
      pline.AddVertexAt(11, Pnt11, -bulge, 0.0, 0.0);  
      pline.AddVertexAt(12, Pnt12, 0.00, 0.0, 0.0);  
      pline.AddVertexAt(13, Pnt13, -bulge, 0.0, 0.0);  
      pline.AddVertexAt(14, Pnt14, 0.00, 0.0, 0.0);  
      pline.AddVertexAt(15, Pnt15, 0.0, 0.0, 0.0);  
      #endregion  
      ObjectId plineId = ObjectId.Null;  
      Database db = Application.DocumentManager.MdiActiveDocument.Database;  
      Autodesk.AutoCAD.DatabaseServices.TransactionManager transactionManager =  
           db.TransactionManager;  
      using (Transaction transaction = transactionManager.StartTransaction())  
      {  
           BlockTable blkTable = (BlockTable)transactionManager.GetObject(db.BlockTableId,  
                OpenMode.ForRead, false);  
           BlockTableRecord blkTableRec =  
                (BlockTableRecord)transactionManager.GetObject(blkTable[BlockTableRecord.ModelSpace],  
                OpenMode.ForWrite, false);  
           plineId = blkTableRec.AppendEntity(pline);  
           transactionManager.AddNewlyCreatedDBObject(pline, true);  
           transaction.Commit();  
      }  
      if (plineId != ObjectId.Null)  
      {  
           switch (StoreType)  
           {  
                case DataStoreType.NamedObjDict:  
                     AddNodRecord(AppName, plineId);  
                     break;  
                case DataStoreType.ExtObjDict:  
                     AddExtEntDict(AppName, plineId);  
                     break;  
                case DataStoreType.ExtData:  
                     AddXData(AppName, plineId);  
                     break;  
                default:  
                     break;  
           }  
      }  
      else  
      {  
           Application.ShowAlertDialog("Kayıt eklenemedi.");  
      }  
 } 
```

**IPEProfile** sınıf yapısını gösteren diyagram için lütfen Şekil-2’ye bakınız.

![Şekil-1](/assets/images/VeriDepolama1.jpg)

Şekil-1

![Şekil-2](/assets/images/VeriDepolama2.jpg)

Şekil-2

b (profil başlık genişliği), h (profil yüksekliği), t (profil başlık et kalınlığı), s (profil gövde et kalınlığı) ve r değerlerinin, 
sözlüklerde ya da XData’larda depolayacağımız kayıtları temsil ettiğini unutmayarak devam edelim. 

```c#
private void AddXData(string RegAppName, ObjectId ObjId)  
 {  
      Document doc =Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      using (Transaction tr = doc.TransactionManager.StartTransaction())  
      {  
           DBObject obj = tr.GetObject(ObjId, OpenMode.ForWrite);  
           RegAppTable regAppTable =  
                (RegAppTable)tr.GetObject(db.RegAppTableId,  
                OpenMode.ForRead, false);  
           if (!regAppTable.Has(RegAppName))  
           {  
                regAppTable.UpgradeOpen();  
                RegAppTableRecord regAppTableRec = new RegAppTableRecord();  
                regAppTableRec.Name = RegAppName;  
                regAppTable.Add(regAppTableRec);  
                tr.AddNewlyCreatedDBObject(regAppTableRec, true);  
           }  
           ResultBuffer rb = new ResultBuffer(  
                 new TypedValue((int)DxfCode.ExtendedDataRegAppName,  
                 RegAppName),  
                 new TypedValue((int)DxfCode.ExtendedDataReal, b),  
                 new TypedValue((int)DxfCode.ExtendedDataReal, h),  
                 new TypedValue((int)DxfCode.ExtendedDataReal, t),  
                 new TypedValue((int)DxfCode.ExtendedDataReal, s),  
                 new TypedValue((int)DxfCode.ExtendedDataReal, r)  
                 );  
           obj.XData = rb;  
           rb.Dispose();  
           tr.Commit();  
      }  
 } 
```

Uygulama kayıt adını ve veritabanı model uzayına eklediğimiz Polyline’ın kimliğini parametre olarak kabul eden **AddXData()** metodunu 
dikkatle incelersek, iki önemli işlevi yerine getirdiğini görürüz. 
Bunlardan ilki uygulama kayıt adını, kayıtlı uygulamalar tablosunda (RegAppTable) araması (RegAppTable.Has() metodu) ve bulamaması 
durumunda eklemesi (RegAppTable.Add(…) metodu). İkincisi ise profil verisinden oluşturduğu kayıt bilgilerini nesneye iliştirmesi 
(DBObject.XData özelliği). XData’da sakladığımız bilgileri silmek istediğimizde kullanacağımız metot **RemoveXData()** olacaktır: 

```c#
public static void RemoveXData(string RegAppName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt = new PromptEntityOptions("\nIEP profil seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj = tr.GetObject(res.ObjectId, OpenMode.ForWrite);  
                if (obj.GetXDataForApplication(RegAppName) != null)  
                {  
                     ResultBuffer rb = new ResultBuffer(  
                          new TypedValue((int)DxfCode.ExtendedDataRegAppName,  
                          RegAppName));  
                     obj.XData = rb;  
                     rb.Dispose();  
                     tr.Commit();  
                }  
                else  
                {  
                     ed.WriteMessage("\nNesne {0} uygulamasına ait XDATA içermiyor.", RegAppName);  
                }  
           }  
      }  
 }
```

Seçilen bir profilin uygulama adıyla kayıtlı XData’sını ise **GetXData()** metodu ile AutoCAD komut satırına kolayca yazdırabiliriz:

```c#
public static void GetXData(string RegAppName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt = new PromptEntityOptions("\nIEP profil seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj = tr.GetObject(res.ObjectId, OpenMode.ForRead);;  
                ResultBuffer rb = obj.XData;  
                if (rb == null)  
                {  
                     ed.WriteMessage("\nNesne XDATA içermiyor.");  
                }  
                else  
                {  
                     int i = 1;  
                     foreach (TypedValue tv in rb)  
                     {  
                          ed.WriteMessage("\nVeri No {0} ->> Tip: {1}, Değer: {2}",  
                               i++, ((DxfCode)tv.TypeCode).ToString(), tv.Value);  
                     }  
                     rb.Dispose();  
                }  
           }  
      }  
 } 
```

NOD’da veri saklama: 

```c#
private void AddNodRecord(string NodRecName, ObjectId ObjId)  
 {  
      Database db =  
           Application.DocumentManager.MdiActiveDocument.Database;  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {  
           DBDictionary AppDict = null;  
           DBDictionary NODict =  
                (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId,  
                OpenMode.ForRead);  
           if (!NODict.Contains(NodRecName))  
           {  
                NODict.UpgradeOpen();  
                AppDict = new DBDictionary();  
                NODict.SetAt(NodRecName, AppDict);  
                tr.AddNewlyCreatedDBObject(AppDict, true);  
           }  
           AppDict =  
                (DBDictionary)tr.GetObject(NODict.GetAt(NodRecName),  
                OpenMode.ForWrite);  
           DBObject plineObj = tr.GetObject(ObjId, OpenMode.ForRead);  
           string recName = plineObj.Handle.Value.ToString();  
           Xrecord rec = new Xrecord();  
           ResultBuffer rb = new ResultBuffer(  
                new TypedValue((int)DxfCode.Real, b),  
                new TypedValue((int)DxfCode.Real, h),  
                new TypedValue((int)DxfCode.Real, t),  
                new TypedValue((int)DxfCode.Real, s),  
                new TypedValue((int)DxfCode.Real, r)  
                );  
           rec.Data = rb;  
           AppDict.SetAt(recName, rec);  
           tr.AddNewlyCreatedDBObject(rec, true);  
           tr.Commit();  
           rb.Dispose();  
      }  
 } 
```

Kod listesini yukarda bulabileceğiniz **AddNodRecord()** metodu, öncelikle AutoCAD NOD’una ulaşarak uygulama sözlüğünün (AppDict) 
NOD’a kayıtlı olup olmadığını (DBDictionary.Contains() metodu) kontrol edecek, kayıtlı değilse yeni sözlüğü oluşturup NOD’a 
ekleyecek ve Polyline’ın kimliğinden benzersiz bir kayıt anahtarı (recName) yaratarak profil bilgilerini NOD’a (AppDict) kaydedecektir. 
NOD’u ve NOD kaydını silen metotlar ise sırasıyla şöyle: 

```c#
public static void RemoveNod(string NodName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {  
           DBDictionary NODict =  
                (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId,  
                OpenMode.ForWrite);  
           if (NODict.Contains(NodName))  
           {  
                NODict.Remove(NodName);  
                tr.Commit();  
                ed.WriteMessage("\"{0}\" sözlüğü başarıyla silindi.", NodName);  
           }  
      }  
 }  
 public static void RemoveNodRecord(string NodRecName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt =  
           new PromptEntityOptions("\nNOD'dan IEP profili seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj =  
                     tr.GetObject(res.ObjectId, OpenMode.ForRead);  
                DBDictionary NODict =  
                     (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId,  
                     OpenMode.ForRead);  
                if (!NODict.Contains(NodRecName))  
                {  
                     ed.WriteMessage("\nNOD bulunamadı.");  
                }  
                else  
                {  
                     DBDictionary AppDict =  
                          (DBDictionary)tr.GetObject(NODict.GetAt(NodRecName),  
                          OpenMode.ForWrite);  
                     NODict.Dispose();  
                     string kayitAd = obj.Handle.Value.ToString();  
                     if (!AppDict.Contains(kayitAd))  
                     {  
                          ed.WriteMessage("\nSeçilen nesne bir profil bilgisi içermiyor.");  
                     }  
                     else  
                     {  
                          AppDict.Remove(kayitAd);  
                          tr.Commit();  
                          ed.WriteMessage("\nProfil bilgisi NOD'dan başarıyla temizlendi.");  
                     }  
                }  
           }  
      }  
 }  
```

Kimliği AppDic sözlüğüne kaydedilmiş bir nesne grafik ekrandan silindiğinde, tahmin edebileceğiniz gibi, bu nesneye ait kayıt AppDict 
sözlüğünden silinmeyecektir. AutoCAD.NET API’sinin, grafik olarak bir karşılığı kalmayan bu tür kayıtlardan korunmak için NOD ile 
ilişkilendirilmiş nesneler silindiğinde sözlükleri uyaran reaktörleri vardır. 
API, uygulama sözlüğüne kayıt eklemek için DBDictionary.SetAt() metodu çağrıldığında, sözlüğü nesnenin sahibi olarak kabul edip 
nesneye persistent reactor olarak tutturur. Bu ayrıntıyı da vurguladıktan sonra NOD kaydının nasıl elde edileceğini gösteren 
**GetNodRecord()** metodunu listeleyerek ExtensionDictionary kullanımına geçelim: 

```c#
public static void GetNodRecord(string NodRecName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt = new PromptEntityOptions("\nIEP profil seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj =  
                     tr.GetObject(res.ObjectId, OpenMode.ForRead);  
                DBDictionary NODict =  
                     (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId,  
                     OpenMode.ForRead);  
                if (!NODict.Contains(NodRecName))  
                {  
                     ed.WriteMessage("\nSözlük kaydı bulunamadı.");  
                }  
                else  
                {  
                     DBDictionary AppDict =  
                          (DBDictionary)tr.GetObject(NODict.GetAt(NodRecName),  
                          OpenMode.ForRead);  
                     string recName = obj.Handle.Value.ToString();  
                     if (!AppDict.Contains(recName))  
                     {  
                          ed.WriteMessage("\nSeçilen nesne profil bilgisi içermiyor.");  
                     }  
                     else  
                     {  
                          ObjectId recId = AppDict.GetAt(recName);  
                          Xrecord rec = (Xrecord)tr.GetObject(recId, OpenMode.ForRead);  
                          ResultBuffer rb = rec.Data;  
                          if (rb == null)  
                          {  
                               ed.WriteMessage("\nProfil bilgileri bulunamadı.");  
                          }  
                          else  
                          {  
                               int i = 1;  
                               foreach (TypedValue tv in rb)  
                               {  
                                    ed.WriteMessage("\nNo {0} ->> Tip: {1}, Değer: {2}",  
                                         i++, ((DxfCode)tv.TypeCode).ToString(), tv.Value);  
                               }  
                               rb.Dispose();  
                          }  
                     }  
                }  
           }  
      }  
 }  
```

ExtensionDictionary’de veri saklama: 

```c#
private void AddExtEntDict(string ExtDictName, ObjectId ObjId)  
 {  
      Database db = Application.DocumentManager.MdiActiveDocument.Database;  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {  
           DBDictionary ExtDict;  
           DBObject obj = tr.GetObject(ObjId, OpenMode.ForWrite);  
           if(obj.ExtensionDictionary == ObjectId.Null)  
                obj.CreateExtensionDictionary();  
           ObjectId extDictId = obj.ExtensionDictionary;  
           ExtDict = (DBDictionary)tr.GetObject(extDictId, OpenMode.ForWrite);  
           Xrecord rec = new Xrecord();  
           ResultBuffer rb = new ResultBuffer(  
                new TypedValue((int)DxfCode.Real, b),  
                new TypedValue((int)DxfCode.Real, h),  
                new TypedValue((int)DxfCode.Real, t),  
                new TypedValue((int)DxfCode.Real, s),  
                new TypedValue((int)DxfCode.Real, r)  
                );  
           rec.Data = rb;  
           ExtDict.SetAt(ExtDictName, rec);  
           tr.AddNewlyCreatedDBObject(rec, true);  
           tr.Commit();  
           rb.Dispose();  
      }  
 }  
```

**AddExtEntDict()** metodu içerisindeki en önemli metot DBObject.CreateExtensionDictionary()’dir. 
Bu metot, seçtiğimiz nesneye ait herhangi bir sözlük yoksa, yeni bir tane oluşturarak bu sözlüğü nesnenin (örneğimizde obj) 
ExtensionDictionary’si olarak ayarlar. Bu ayardan sonra DBDictionary.SetAt() metodunu kullanarak istediğimiz kaydı sözlüğe 
ekleyebiliriz. ExtensionDictionary sözlük kayıtlarını silmek de oldukça kolaydır. Öncelikle, nesneye ait ExtensionDictionary’nin, 
silmek istediğimiz sözlük kaydını (ExtDictName) içerip içermeğini DBDictionary.Contains() metodu ile kontrol eder, içeriyorsa 
DBDictionary.Remove() metodunu kullanarak sözlüğü gönül rahatlığıyla silebiliriz. 

```c#
 public static void RemoveExtEntDict(string ExtDictName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt = new PromptEntityOptions("\nIEP profil seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj = tr.GetObject(res.ObjectId, OpenMode.ForWrite);  
                if (obj.ExtensionDictionary == ObjectId.Null)  
                {  
                     ed.WriteMessage("\nSözlük kaydı içermiyor.");  
                }  
                else  
                {  
                     DBDictionary ExtDict =  
                          (DBDictionary)tr.GetObject(obj.ExtensionDictionary,  
                          OpenMode.ForWrite);  
                     if (!ExtDict.Contains(ExtDictName))  
                     {  
                          ed.WriteMessage("\nSözlük kaydı bulunamadı.");  
                     }  
                     else  
                     {  
                          ExtDict.Remove(ExtDictName);  
                          tr.Commit();  
                     }  
                }  
           }  
      }  
 } 
```

**GetExtEntDict()** metodunu kullanarak nesneye ait ExtensionDictionary kaydını görebilirsiniz: 

```c#
public static void GetExtEntDict(string ExtDictName)  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      PromptEntityOptions opt = new PromptEntityOptions("\nIEP profil seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                DBObject obj =  
                     tr.GetObject(res.ObjectId, OpenMode.ForRead);  
                if (obj.ExtensionDictionary == ObjectId.Null)  
                {  
                     ed.WriteMessage("\nSözlük içermiyor.");  
                }  
                else  
                {  
                     DBDictionary ExtDict =  
                          (DBDictionary)tr.GetObject(obj.ExtensionDictionary,  
                          OpenMode.ForRead);  
                     if (!ExtDict.Contains(ExtDictName))  
                     {  
                          ed.WriteMessage("\nSözlük kaydı bulunamadı.");  
                     }  
                     else  
                     {  
                          ObjectId recId = ExtDict.GetAt(ExtDictName);  
                          Xrecord rec = (Xrecord)tr.GetObject(recId, OpenMode.ForRead);  
                          ResultBuffer rb = rec.Data;  
                          if (rb == null)  
                          {  
                               ed.WriteMessage("\nProfil bilgileri bulunamadı.");  
                          }  
                          else  
                          {  
                               int i = 1;  
                               foreach (TypedValue tv in rb)  
                               {  
                                    ed.WriteMessage("\nKayıt No {0} ->> Tip: {1}, Değer: {2}",  
                                         i++, ((DxfCode)tv.TypeCode).ToString(), tv.Value);  
                               }  
                               rb.Dispose();  
                          }  
                     }  
                }  
           }  
      }  
 } 
```

**Yukarıdaki Örnekler için AutoCAD Komutları:**<br>
XData için : XDataEkle, XDataYaz, XDataSil<br>ExtensionDictionary için : ExtDictEkle, ExtDictYaz, ExtDictSil<br>
NOD içinse : NodKayitEkle, NodKayitYaz, NodKayitSil, NodSil 

**Uygulama Sözlüğü Yardımcı C# Sınıfı:**<br>
Aşağıdan indirebileceğiniz Visual.Studio.Net 2010 projesine ExtDictHelper adlı bir C# sınıfını ekledim. 
Bu sınıfı her türlü AutoCAD grafik nesnesi ExtensionDictionary’sine sözlük kaydı iliştirmekte kullanabilirsiniz. 
Bu sınıfın AutoCAD komutları olan AddExtDct, WriteExtDct ve RemoveExtDct. AddExtDct komutuyla aynı nesne için birden fazla sözlük oluşturabilirsiniz. 
Sözlüklerde depolanabilecek veri türleri ise Angle, ObjectId, Int32, Text, Point ve Real. 

### İndir
[Veri Depolama Visual Studio 2010 Projesi](http://eykaraduman.github.io/assets/downloads/AbcExtensionDictionary.zip)

