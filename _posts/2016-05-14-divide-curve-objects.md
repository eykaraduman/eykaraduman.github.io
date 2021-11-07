---
title: Eğrisel Nesnelerin Parçalara Bölünmesi
permalink: /acad/segmented-curve-objects/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

**Curve2Segments()** yordamı eğrisel nesneleri istenilen sayıda lineer parçaya bölerek bir LWPOLYLINE oluşturuyor. Geçerli olduğu nesneler ise ARC, CIRCLE, LWPOLYLINE, 2DPOLYLINE, LINE, SPLINE ve ELLIPSE AutoCAD nesneleri.

<p>
<img src="{{ "/assets/images/Curve2Segments.gif" | absolute_url }}"  alt="Şekil-1" style="width:50%">
<figure>
  <figcaption>Şekil-1</figcaption>
</figure>
</p>
```c#
public static void Curve2Segments()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;

    TypedValue[] tyVals = new TypedValue[9]
    { 
        new TypedValue(-4, "<OR"),
        new TypedValue((int)DxfCode.Start, "SPLINE"),
        new TypedValue((int)DxfCode.Start, "LINE"),
        new TypedValue((int)DxfCode.Start, "ARC"),
        new TypedValue((int)DxfCode.Start, "POLYLINE"),
        new TypedValue((int)DxfCode.Start, "LWPOLYLINE"),
        new TypedValue((int)DxfCode.Start, "CIRCLE"),
        new TypedValue((int)DxfCode.Start, "ELLIPSE"),
        new TypedValue(-4, "OR>")
    };
    SelectionFilter selFilter = new SelectionFilter(tyVals);
            

    PromptIntegerOptions pIntOpts = new PromptIntegerOptions("\nEnter number of segments: ");
    PromptIntegerResult pIntRes = ed.GetInteger(pIntOpts);
    if (pIntRes.Status != PromptStatus.OK)
        return;

    PromptSelectionOptions pSelOpts = new PromptSelectionOptions();
    pSelOpts.AllowDuplicates = false;
    pSelOpts.SingleOnly = false;
    PromptSelectionResult pSelRes = doc.Editor.GetSelection(pSelOpts, selFilter);
    if (pSelRes.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
        BlockTableRecord btr = (BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                                OpenMode.ForWrite, false);

        int numberOfSegments = pIntRes.Value;
        foreach (ObjectId id in pSelRes.Value.GetObjectIds())
        {
            Point2dCollection pnts = new Point2dCollection();
            Curve curve = (Curve)tr.GetObject(id, OpenMode.ForWrite);
            double curveLength = curve.GetDistanceAtParameter(curve.EndParam);
            for (int i = 0; i < numberOfSegments; i++)
            {
                double tmpDist = i * curveLength / numberOfSegments;
                Point3d tmpPnt = curve.GetPointAtDist(tmpDist);
                pnts.Add(new Point2d(tmpPnt.X, tmpPnt.Y));
            }

            Polyline pLine = new Polyline();
            pLine.SetDatabaseDefaults();
            for (int i = 0; i < pnts.Count; i++)
            {
                pLine.AddVertexAt(i, pnts[i], 0.0, 0.0, 0.0);
            }
            pLine.Layer = curve.Layer;
            pLine.Color = curve.Color;
            pLine.Closed = curve.Closed;

            btr.AppendEntity(pLine);
            tr.AddNewlyCreatedDBObject(pLine, true);
            curve.Erase();
        }

        tr.Commit();
    }
}
```

