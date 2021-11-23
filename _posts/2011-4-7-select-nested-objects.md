---
title: İçiçe AutoCAD Nesnelerinin Seçilmesi 
permalink: /acad/select-nested-objects/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
Editor sınıfının sağladığı **GetNestedEntity(…)** metodu iç içe AutoCAD nesnelerini seçmekte kullanılır. 
Seçilen nesne bilgilerini AutoCAD komut satırında listeleyen **NstSel** komutunu çalıştırarak bu seçim işleminin nasıl 
gerçekleştiğini görebilirsiniz.

```c#
[CommandMethod("NstSel")]
public static void SelectingNestedEntity()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    Editor ed = doc.Editor;
    PromptNestedEntityOptions opt =
    new PromptNestedEntityOptions("\nSelect a nested entity: ");
    opt.AllowNone = false;
    PromptNestedEntityResult res =
    ed.GetNestedEntity(opt);
    if (res.Status == PromptStatus.OK)
    {
        if (!res.ObjectId.IsNull)
        {
            Transaction tr =
            Application.DocumentManager.MdiActiveDocument.
            TransactionManager.StartTransaction();
            using (tr)
            {
                Entity ent =
                (Entity)tr.GetObject(res.ObjectId,
                OpenMode.ForRead);
                ed.WriteMessage("\n");
                ent.List();
                ent.Dispose();
                tr.Commit();
            }
        }
    }
}
```

