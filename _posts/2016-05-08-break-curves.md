---
title: Eğrisel Nesnelerin Kırılması
permalink: /acad/break-curve-objects/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

ARC, CIRCLE, LWPOLYLINE, LINE, SPLINE ve ELLIPSE nesnelerinin AutoCAD.Net API'si ile nasıl kırılabileceğini aşağıdaki <strong>BreakCurveObjects()</strong> yordamında bulabilirsiniz. Bu yordam, belirlenen noktalarda nesneleri kırmak için, <strong>Curve</strong> sınıfına ait <strong>GetSplitCurves(…)</strong> üyesini kullanıyor.

```c#
public virtual DBObjectCollection GetSplitCurves(DoubleCollection value);
public virtual DBObjectCollection GetSplitCurves(Point3dCollection points);
```

Kodla ilgili birkaç önemli noktaya dikkat etmekte fayda var;

<ul><li>Kodun kullanıcı tanımlı koordinat sistemlerinde de çalışabilmesi için seçilen noktaları WCS'ye dönüştürmeyi unutmamak gerek. Çünkü <strong>GetPoint(…)</strong> yordamı ile elde edilen tüm noktalar kullanıcı tanımlı koordinat sistemindedir.</li><li>Seçtiğiniz noktaların kırmak istediğiniz nesne üzerinde olup olmadığı doğrulanmalı. Bunun için <strong>IsPointOnCurve(…)</strong> yordamı kullanılmıştır.</li><li>GetSplitCurves(…) yordamına geçirilecek nokta listelerini eğrinin başlangıcına göre mutlaka sıralanmalı. Aksi takdirde noktaları seçim sıranıza bağlı olarak beklediğinizden farklı sonuçlarla karşılaşabilirsiniz.




<img src="{{ "/assets/images/BreakCurveObjects.gif" | absolute_url }}"  alt="Şekil-1" style="width:100%">

<figure>
  <figcaption>Şekil-1</figcaption>
</figure>




```csharp
public static void BreakCurveObjects()
{
    // Autocad veritabanına erişim
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;

    // Daire ve elips nesnelerinin seçimi için seçim filtresi oluşturulması
    TypedValue[] tyVals = new TypedValue[8]
    { 
        new TypedValue(-4, "<OR"),
        new TypedValue((int)DxfCode.Start, "SPLINE"),
        new TypedValue((int)DxfCode.Start, "LINE"),
        new TypedValue((int)DxfCode.Start, "ARC"),
        new TypedValue((int)DxfCode.Start, "LWPOLYLINE"),
        new TypedValue((int)DxfCode.Start, "CIRCLE"),
        new TypedValue((int)DxfCode.Start, "ELLIPSE"),
        new TypedValue(-4, "OR>")
    };
    SelectionFilter selFilter = new SelectionFilter(tyVals);

    // Daire ya da elips nesnesinin seçilmesi
    PromptSelectionOptions pSelOpts = new PromptSelectionOptions();
    pSelOpts.AllowDuplicates = false;
    pSelOpts.SingleOnly = true;
    PromptSelectionResult pSelRes = doc.Editor.GetSelection(pSelOpts, selFilter);

    if (pSelRes.Status == PromptStatus.OK)
    {
        Point3dCollection breakPoints = new Point3dCollection();
        DoubleCollection distOfBreakPoints = new DoubleCollection();

        // Kırma noktası seçimi için seçenekler
        PromptPointOptions pPtOpts = new PromptPointOptions("");
        pPtOpts.Message = "\nSelect break point: ";
        pPtOpts.AllowNone = true;                

        using (Transaction tr = db.TransactionManager.StartTransaction())
        {
            PromptPointResult pPtRes;
            // Seçilen eğrinin yazma amaçlı açılması
            Curve curve = (Curve)tr.GetObject(pSelRes.Value.GetObjectIds()[0], OpenMode.ForWrite);
                   
            // Kırma noktalarının seçilmesi
            do
            {
                pPtRes = doc.Editor.GetPoint(pPtOpts);
                // Seçilen koordinatın WCS'ye dönüştürülmesi         
                Point3d breakPntWCS = pPtRes.Value.TransformBy(ed.CurrentUserCoordinateSystem);
                // Seçilen noktanın eğri üzerinde olup olmadığının kontrolü
                if (IsPointOnCurve(curve, breakPntWCS))
                {
                    breakPoints.Add(curve.GetClosestPointTo(breakPntWCS, false));
                    distOfBreakPoints.Add(curve.GetDistAtPoint(breakPntWCS));
                }
                else
                {
                    ed.WriteMessage("\nSelected point is not on curve!");
                }
            }
            while (pPtRes.Status == PromptStatus.OK);

            if (breakPoints.Count > 0)
            {
                // Noktaların eğri başlangıcına göre sıralanması
                double[] rawPoints = distOfBreakPoints.ToArray();
                Array.Sort(rawPoints);
                Point3dCollection sortedBreakPoints = new Point3dCollection();
                foreach (double item in rawPoints)
                {
                    sortedBreakPoints.Add(breakPoints[distOfBreakPoints.IndexOf(item)]);
                }

                // Sıralanmış noktalara göre eğri parçalarının elde edilmesi
                DBObjectCollection objs = curve.GetSplitCurves(sortedBreakPoints);

                // Eğri parçalarının çizime eklenmesi
                if (objs != null)
                {
                    if (objs.Count > 0)
                    {
                        BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                        foreach (var obj in objs)
                        {
                            Entity ent = obj as Entity;
                            BlockTableRecord btRecord = (BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                                OpenMode.ForWrite, false);

                            btRecord.AppendEntity(ent);
                            tr.AddNewlyCreatedDBObject(ent, true);
                        }
                        // Orjinal eğrinin silinmesi
                        curve.Erase();
                        tr.Commit();
                    }
                }
            }
        }
    }
}
```

```c#
private static bool IsPointOnCurve(Curve crv, Point3d pt)
{
    try
    {
        Point3d p = crv.GetClosestPointTo(pt, false);
        return (p - pt).Length <= Tolerance.Global.EqualPoint;
    }
    catch { }

    return false;
}
```

<strong>BreakObjectByObject()</strong> yordamı ise eğrisel nesnelerin kesişim noktalarını esas alarak kırma işlemi gerçekleştiriyor.





<img src="{{ "/assets/images/BreakObjectsByObject.gif" | absolute_url }}"  alt="Şekil-2" style="width:100%">

<figure>
  <figcaption>Şekil-2</figcaption>
</figure>





```csharp
public static void BreakObjectByObject()
{
    Document doc = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    Editor ed = Autodesk.AutoCAD.ApplicationServices.Application.DocumentManager.MdiActiveDocument.Editor;

    PromptEntityOptions pEntOpt = new PromptEntityOptions("");
    pEntOpt.AllowNone = false;
    pEntOpt.Message = "\nSelect a object to break: ";
    pEntOpt.SetRejectMessage("\nSelect Arc, Circle, Ellipse, Line, Polyline ve Spline!");
    pEntOpt.AddAllowedClass(typeof(Arc), true);
    pEntOpt.AddAllowedClass(typeof(Circle), true);
    pEntOpt.AddAllowedClass(typeof(Ellipse), true);
    pEntOpt.AddAllowedClass(typeof(Line), true);
    pEntOpt.AddAllowedClass(typeof(Polyline), true);
    pEntOpt.AddAllowedClass(typeof(Spline), true);

    PromptEntityResult pEntResObjectToBreak = ed.GetEntity(pEntOpt);
    if (pEntResObjectToBreak.Status != PromptStatus.OK)
        return;

    pEntOpt.Message = "\nSelect breaking object: ";
    PromptEntityResult pEntResBreakinObject = ed.GetEntity(pEntOpt);
    if (pEntResBreakinObject.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Curve objectToBreak = (Curve)tr.GetObject(pEntResObjectToBreak.ObjectId, OpenMode.ForWrite);
        Curve breakinObject = (Curve)tr.GetObject(pEntResBreakinObject.ObjectId, OpenMode.ForRead);

        Point3dCollection intersectPts = new Point3dCollection();
        objectToBreak.IntersectWith(breakinObject, Intersect.OnBothOperands, intersectPts, IntPtr.Zero, IntPtr.Zero);

        if (intersectPts.Count != 0)
        {
            DoubleCollection distOfBreakPoints = new DoubleCollection();
            foreach (Point3d pt in intersectPts)
            {
                distOfBreakPoints.Add(objectToBreak.GetDistAtPoint(pt));
            }

            double[] rawPoints = distOfBreakPoints.ToArray();
            Array.Sort(rawPoints);
            Point3dCollection sortedBreakPoints = new Point3dCollection();
            foreach (double item in rawPoints)
            {
                sortedBreakPoints.Add(intersectPts[distOfBreakPoints.IndexOf(item)]);
            }

            DBObjectCollection objs = objectToBreak.GetSplitCurves(sortedBreakPoints);

            if (objs != null)
            {
                if (objs.Count > 0)
                {
                    BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                    foreach (var obj in objs)
                    {
                        Entity ent = obj as Entity;
                        BlockTableRecord btRecord = (BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                            OpenMode.ForWrite, false);

                        btRecord.AppendEntity(ent);
                        tr.AddNewlyCreatedDBObject(ent, true);
                    }

                    objectToBreak.Erase();
                    tr.Commit();
                }
            }
        }
    }
}
```

