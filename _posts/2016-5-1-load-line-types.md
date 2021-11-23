---
title: Çizgi Tipi (Line Type) Yükleme
permalink: /acad/load-line-types/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

Aşağıdaki yordamları çizgi tiplerini yüklemek için kullanabilirsiniz. **LoadLineType(...)** yordamı verili çizgi tipi dosyasından 
belirli bir çizgi tipini yüklerken, **LoadLineTypes(...)** yordamı ise tüm çizgi tiplerini yüklemekte.

```c#
public static void LoadLineType(this Database db, string LineTypeName, string LineTypeFileName)
{
    try
    {
        string path = HostApplicationServices.Current.FindFile(LineTypeFileName, db, FindFileHint.Default);
        db.LoadLineTypeFile(LineTypeName, path);
    }
    catch (Autodesk.AutoCAD.Runtime.Exception ex)
    {
        if (ex.ErrorStatus == ErrorStatus.FilerError)
            AcadApplication.Editor.WriteMessage("\n\"{0}\"  çizgi tipi dosyası bulunamadı!", LineTypeFileName);
        else if (ex.ErrorStatus == ErrorStatus.DuplicateRecordName)
            AcadApplication.Editor.WriteMessage("\n\"{0}\"  çizgi tipi yüklenmiş!", LineTypeName);
        else
            AcadApplication.Editor.WriteMessage("\n{0} çizgi tipi yüklenemedi: {1}", LineTypeName, ex.Message);

    }            
}

public static void LoadLineTypes(this Database db, string LineTypeFileName)
{
    try
    {
        string path = HostApplicationServices.Current.FindFile(filename, db, FindFileHint.Default);
        db.LoadLineTypeFile("*", path);
    }
    catch (Autodesk.AutoCAD.Runtime.Exception ex)
    {
        if (ex.ErrorStatus == ErrorStatus.FilerError)
            AcadApplication.Editor.WriteMessage("\n\"{0}\"  çizgi tipi dosyası bulunamadı!", LineTypeFileName);
        else if (ex.ErrorStatus == ErrorStatus.DuplicateRecordName)
            AcadApplication.Editor.WriteMessage("\nÇizgi tipleri zaten yüklenmiş!");
        else
            AcadApplication.Editor.WriteMessage("\nHata: {0}", ex.Message);
    }
}
```

