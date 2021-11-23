---
title: XData Uygulama Adlarının Listelenmesi 
permalink: /acad/list-xdata-app-names/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
Autodesk.AutoCAD.DatabaseServices.DBObject sınıfına ait yerleşik nesnelere, farklı uygulama adlarıyla veri (XData) eklenebilmektedir. 
Aşağıdaki örnek kod, bu uygulama adlarının AutoCad komut satırında nasıl listelenebileceğini göstermekte. 

```c#
[CommandMethod("GetXDataAppNames")]  
 public void GetXDataAppNames()  
 {  
      // Aktif döküman, veritabanı ve editöre erişim  
      Document doc = Application.DocumentManager.MdiActiveDocument;       
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      // Eklenmiş veriyi (XData) içeren nesnenin seçimi  
      PromptEntityOptions opt = new PromptEntityOptions("\nBir nesne seçin: ");  
      PromptEntityResult res = ed.GetEntity(opt);  
      if (res.Status == PromptStatus.OK)  
      {  
           using (Transaction tr = doc.TransactionManager.StartTransaction())  
           {  
                // Nesnenin okuma amaçlı açılması  
                DBObject obj = tr.GetObject(res.ObjectId, OpenMode.ForRead);  
                // Eklenmiş verinin eldesi  
                ResultBuffer rb = obj.XData;  
                if (rb != null)  
                {  
                     TypedValue[] tvArray = rb.AsArray();  
                     rb.Dispose();  
                     // Uygulama adlarının komut satırına yazdırılması  
                     foreach (TypedValue tv in tvArray)  
                     {  
                          if ((DxfCode)tv.TypeCode == DxfCode.ExtendedDataRegAppName)  
                          {  
                               ed.WriteMessage("\n{0}", tv.Value);  
                          }  
                     }  
                }  
           }  
      }        
 } 
```

