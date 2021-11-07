---
title: Polyline Üzerinde Kesişim Noktası Araştırma
permalink: /acad/polyline-intersections/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

Aşağıda örnek kodunu verdiğim <strong>FindIntersectionPoints</strong> yordamı seçilen bir polyline nesnesinin diğer nesnelerle (3dPolyline, Lwpolyline, Line, 3dFace) izdüşümsel kesişimini ve bulunan kesişim noktaların Z değerini araştırmakta.

İşleyişi ise sırasyıla aşağıdaki gibi:

<ul><li>Kaynak polyline nesnesinin seçimi </li><li>Hedef nesnelerin seçimi </li><li>İzdüşümsel kesişim noktalarının araştırılması </li><li>Bulunan kesişim noktalarının Z değerlerinin araştırılması </li><li>Bulunan noktaların kaynak polyline üzerindeki mesafelerine göre dizilmesi </li><li>Bulunan kesişim nokatalarına AutoCAD nokta nesnesi yeleştirilmesi ve bu noktaların AutoCAD editöründe listelenmesi</li></ul>

Ayrıca örnek kodda Editor sınıfının <strong>GetEntity(…)</strong>, <strong>SelectCrossingWindow(…)</strong> ve Entity sınfının <strong>IntersectWith(…)</strong> yordamlarının kulllanımını bulabilirsiniz.

```csharp
public static void FindIntersectionPoints()
 {
       Database db = Application.DocumentManager.MdiActiveDocument.Database;
       Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
       PromptEntityOptions sourceEntOpts =
         new PromptEntityOptions("\nSelect a polyline: ");
       sourceEntOpts.SetRejectMessage("\nWrong object type: Please select a polyline!");
       sourceEntOpts.AddAllowedClass(typeof(Polyline), true);
       PromptEntityResult res = ed.GetEntity(sourceEntOpts);
       ObjectId sourceId = res.ObjectId;
       if(res.Status == PromptStatus.OK)
       {
         using (Transaction tr =
           db.TransactionManager.StartTransaction())
         {
           Polyline ent =
             tr.GetObject(sourceId, OpenMode.ForRead) as Polyline;
           Extents3d ext = ent.GeometricExtents;
           Point3d pnt1 = ext.MinPoint;
           Point3d pnt2 = ext.MaxPoint;
           PromptSelectionResult targetSelResult =
             ed.SelectCrossingWindow(pnt1, pnt2,
             new SelectionFilter(new TypedValue[6]
             {
              new TypedValue(-4, "&lt;OR"),
              new TypedValue((int)DxfCode.Start, "3DFACE"),
              new TypedValue((int)DxfCode.Start, "LINE"),
              new TypedValue((int)DxfCode.Start, "POLYLINE"),
              new TypedValue((int)DxfCode.Start, "LWPOLYLINE"),
              new TypedValue(-4, "OR>")
             }));
           if (targetSelResult.Status == PromptStatus.OK
             &amp;&amp;
             targetSelResult.Value.Count > 0)
           {
             ObjectIdCollection targetIds =
               new ObjectIdCollection(targetSelResult.Value.GetObjectIds());
             if (targetIds.Contains(sourceId))
               targetIds.Remove(sourceId);
             SortedList sortedPoints =
               new SortedList();
             Polyline sourceEnt =
               tr.GetObject(sourceId, OpenMode.ForWrite) as Polyline;
             foreach (ObjectId targetId in targetIds)
             {
               Entity targetEnt =
                 tr.GetObject(targetId, OpenMode.ForRead) as Entity;
               Point3dCollection projectedIntPoints =
                 new Point3dCollection();
               targetEnt.IntersectWith(
                 sourceEnt,
                 Intersect.OnBothOperands,
                 sourceEnt.GetPlane(),
                 projectedIntPoints, IntPtr.Zero, IntPtr.Zero);
               foreach (Point3d pnt in projectedIntPoints)
               {
                 Line ln =
                   new Line(pnt, pnt + Vector3d.ZAxis);
                 Point3dCollection realIntPnts =
                   new Point3dCollection();
                 targetEnt.IntersectWith(
                   ln,
                   Intersect.ExtendArgument,
                   ln.GetPlane(),
                   realIntPnts, IntPtr.Zero, IntPtr.Zero);
                 ln.Dispose();
                 if (realIntPnts.Count > 0)
                 {
                   foreach (Point3d realPnt in realIntPnts)
                   {
                     double distance =
                       sourceEnt.GetDistAtPoint(
                       sourceEnt.GetClosestPointTo(realPnt, false));
                     if (!sortedPoints.ContainsKey(distance))
                       sortedPoints.Add(distance, realPnt);
                   }
                 }
               }
             }
             if (sortedPoints.Count > 0)
             {
               BlockTable bt =
                 (BlockTable)tr.GetObject(
                 db.BlockTableId,
                 OpenMode.ForRead,
                 false);
               BlockTableRecord btr =
                 (BlockTableRecord)tr.GetObject(
                 bt[BlockTableRecord.ModelSpace],
                 OpenMode.ForWrite,
                 false);
               int i = 1;
               foreach (KeyValuePair kvp in sortedPoints)
               {
                 DBPoint dbPnt = new DBPoint(kvp.Value);
                 btr.AppendEntity(dbPnt);
                 tr.AddNewlyCreatedDBObject(dbPnt, true);
                 ed.WriteMessage(
                   "\n{0} : {1} | {2}",
                   i,
                   kvp.Key.ToString(),
                   kvp.Value.ToString());
                 i++;
               }
               tr.Commit();
             }
           }
         }
       }
     }
```

