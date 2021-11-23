---
title: Bir Çizimden Diğerine UCS Kayıtlarının Kopyalanması
permalink: /acad/import-ucs/
published: true
classes: wide
categories:
  - AutoCAD.Net
---



Bir AutoCAD çizimindeki nesneleri diğerine kopyalamak için Database sınıfının **WblockCloneObjects(...)** yordamını kullanabilirsiniz.

```c#
public void WblockCloneObjects(ObjectIdCollection identifiers, ObjectId id, IdMapping mapping, DuplicateRecordCloning cloning, bool deferTranslation);
```

Aşağıdaki **UcsImport** yordamı, kaynak çizimdeki ucs tablo kayıtlarını hedef çizime kopyalıyor. Metin stili, katmanlar gibi diğer kayıt türlerinin kopyalanmasında da benzer bir yordamı kullanabilirsiniz. Örneğin katmanları kopyalayabilmek için *sourceDb.UcsTableId* yerine *sourceDb.TextStyleTableId*, *sourceDb.UcsTableId* yerine *sourceDb.TextStyleTableId* ve *UcsTableRecord* yerine ise *TextStyleTableRecord* koymanız yeterli.

```c#
public static void UcsImport(string sourceDbFilePath)
{
    // 1. Aktif veri tabanı isminin elde edilmesi
    Database destinationDb = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument.Database;
    // 2. Kaynak veri tabanının oluşturulması
    using (Database sourceDb = new Database(false, true))
    {
        try
        {
            // 3. Kaynak dosyasındaki bilgilerinin okunması
            sourceDb.ReadDwgFile(sourceDbFilePath, System.IO.FileShare.Read, true, null);

            using (Transaction tr = sourceDb.TransactionManager.StartTransaction())
            {
                // 4. Ucs tablosunun elde edilmesi
                DBDictionary ucsTable =
                    (DBDictionary)tr.GetObject(sourceDb.UcsTableId, OpenMode.ForRead, false);
    
                ObjectIdCollection ucsIds = new ObjectIdCollection();
    
                // 5. Ucs tablo kayıt kimliklerinin elde edilmesi
                foreach (DBDictionaryEntry tblEnt in ucsTable)
                {
                    UcsTableRecord tblRec =
                        (UcsTableRecord)tr.GetObject(tblEnt.Value, OpenMode.ForRead, false);
                    if (!tblRec.IsErased)
                        ucsIds.Add(tblRec.Id);
                }
                // 6. Ucs tablo kayıtlarının kopyalanması
                IdMapping mapping = new IdMapping();
                sourceDb.WblockCloneObjects(ucsIds, destinationDb.UcsTableId, mapping,
                    DuplicateRecordCloning.Replace, false);
            }
        }
        catch (System.Exception exp)
        {
            System.Windows.Forms.MessageBox.Show(exp.ToString());
        }
    }
}
```

