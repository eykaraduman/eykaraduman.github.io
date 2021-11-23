---
title: AutoCAD Nesnelerinin Geometrik Sınırları
permalink: /acad/object-geometric-extends/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
Bir Autocad nesnesinin sınırlarını (minimum ve maksimum noktalarını) bulmakta **Autodesk.AutoCAD.DatabaseServices** isim uzayında bulunan **Extents3d** yapısını kullanabilirsiniz. Çünkü her AutoCAD nesnesi **GeometricExtents** adlı bir özellik içerir ve bu özellik 
Extents3d türündedir. 

**FindExtentsFromObjectIdCollection** ise seçili autocad nesnelerinin geometrik sınırını, bu nesnelerin kimlikleri üzerinde ilerleyerek belirleyen basit bir yordam.

```c#
public static Extents3d FindExtentsFromObjectIdCollection(Transaction tr, ObjectIdCollection ids)
{
    Extents3d tmpExt = new Extents3d();
    foreach (ObjectId id in ids)
    {
        Entity ent = tr.GetObject(id, OpenMode.ForRead, false, true) as Entity;
        if (ent.Bounds.HasValue)
        {
            Extents3d ext = ent.GeometricExtents;
            tmpExt.AddPoint(ext.MinPoint);
            tmpExt.AddPoint(ext.MaxPoint);
        }               
    }
    return tmpExt;
}
```

