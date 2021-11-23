---
title: Genişletilmiş Sözlük Kullanarak Nesneleri Birbirine Bağlama
permalink: /acad/link-objects/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

Metin (DBText) nesnesi içeriğinin Daire (Circle) nesnesi alanıyla dinamik olarak nasıl bağlanabileceğini aşağıda bir örnekle göstermeye çalıştım. Metin nesnesini daire nesnesine bağlamak için sırasıyla,
Daire nesnesinin genişletilmiş sözlüğüne (ExtensionDictionary) metin nesnesinin kimliğini (ObjectId) ekledim.

- Daire nesnesi değiştiğinde oluşan olayı (DbCircle_Modified_Event) tanımlayıp daire nesnesi ile ilişkilendirdim (DBObject.Modified).
- LinkObjects komutuyla daire nesnesine metin nesnesine bağlayıp nasıl çalıştığını kendiniz deneyebilirsiniz.

```c#
private void LinkObjectIds(Database db, Transaction tr, string ExtDictName, ObjectId firstId, ObjectId SecondId)  
 {  
      DBObject obj = tr.GetObject(firstId, OpenMode.ForWrite);  
      obj.CreateExtensionDictionary();  
      DBDictionary ExtDict = (DBDictionary)tr.GetObject(obj.ExtensionDictionary, OpenMode.ForWrite);  
      using (ResultBuffer rb = new ResultBuffer(new TypedValue((int)DxfCode.SoftPointerId, SecondId)))  
      {  
           Xrecord rec = new Xrecord();  
           rec.Data = rb;  
           ExtDict.SetAt(ExtDictName, rec);  
           tr.AddNewlyCreatedDBObject(rec, true);  
      }  
 }  
public void DbCircle_Modified_Event(object sender, EventArgs e)  
 {  
      string ExtDictName = "ABC_LINK_OBJECTS";  
      DBObject obj = sender as DBObject;  
      Database db = Application.DocumentManager.MdiActiveDocument.Database;  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {          
           DBDictionary ExtDict = (DBDictionary)tr.GetObject(obj.ExtensionDictionary, OpenMode.ForRead);  
           if (ExtDict.Contains(ExtDictName))  
           {  
                Xrecord rec = (Xrecord)tr.GetObject(ExtDict.GetAt(ExtDictName), OpenMode.ForRead);  
                using (ResultBuffer rb = rec.Data)  
                {  
                     if (rb != null)  
                     {  
                          Circle crc = obj as Circle;  
                          ObjectId txtId = (ObjectId)rb.AsArray()[0].Value;  
                          DBText txt = (DBText)tr.GetObject(txtId, OpenMode.ForWrite);  
                          txt.TextString = crc.Area.ToString("0.00");  
                     }              
                }  
           }  
           tr.Commit();  
      }        
 }  
[CommandMethod("AbcGroup", "LinkObjects", "LinkObjectsLocal", CommandFlags.Modal)]  
 public void LinkObjects()  
 {  
      string DictName = "ABC_LINK_OBJECTS";  
      Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;  
      Database db = Application.DocumentManager.MdiActiveDocument.Database;  
      PromptEntityOptions peo = new PromptEntityOptions("\nSelect circle:");  
      peo.SetRejectMessage("\nSelect circle!");  
      peo.AddAllowedClass(typeof(Circle), true);        
      peo.AllowNone = false;  
      PromptEntityResult per = ed.GetEntity(peo);  
      if (per.Status == PromptStatus.OK)  
      {  
           ObjectId crcId = per.ObjectId;  
           peo = new PromptEntityOptions("\nSelect text:");  
           peo.SetRejectMessage("\nSelect text!");  
           peo.AddAllowedClass(typeof(DBText), true);          
           peo.AllowNone = false;  
           per = ed.GetEntity(peo);  
           if (per.Status == PromptStatus.OK)  
           {  
                ObjectId txtId = per.ObjectId;            
                using (Transaction tr = db.TransactionManager.StartTransaction())  
                {  
                     LinkObjectIds(db, tr, DictName, crcId, txtId);  
                     tr.Commit();  
                }  
                using (Transaction tr = db.TransactionManager.StartTransaction())  
                {  
                     DBObject dbObjCrc = tr.GetObject(crcId, OpenMode.ForNotify);              
                     dbObjCrc.Modified += DbCircle_Modified_Event;  
                     tr.Commit();  
                }  
           }  
      }  
 }
```

