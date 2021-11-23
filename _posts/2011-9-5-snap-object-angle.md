---
title: Nesne Açısına Yapışma (Object Snap)
permalink: /acad/snap-object-angle/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD **SnapAng** komutu, yapışma açısının bir değer olarak ya da iki nokta seçilerek girilmesine izin verir.
Aşağıda AutoCAD.Net (C#) kaynak kodunu verdiğim ObjSnp komutu ise Polyline3d, Polyline, Line, Arc, Circle, Text, MText nesnelerinin, 
kullanıcının seçtiği noktadaki açısını (Arc, Circle ve Polyline yay parçalarında bu açı tanjantın açısıdır) hesaplayarak 
SnapAng sistem değişkenine atama becerisine sahip. 

```c#
using Autodesk.AutoCAD.Geometry;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.Internal;
using Autodesk.AutoCAD.Runtime;
using AcApp = Autodesk.AutoCAD.ApplicationServices.Application;

[CommandMethod("AbcProgramlama", "ObjectSnapAng", "ObjSnp", CommandFlags.Modal)]

public static void ObjectSnapAngle()
{
    Document doc = Application.DocumentManager.MdiActiveDocument;
    Database db = doc.Database;
    Editor ed = doc.Editor;
    PromptNestedEntityOptions opt = new PromptNestedEntityOptions("\nBir nesne seçin: ");
    opt.AllowNone = false;
    PromptNestedEntityResult res = ed.GetNestedEntity(opt);
    if (res.Status == PromptStatus.OK)
    {
        Transaction tr = db.TransactionManager.StartTransaction();
        using (tr)
        {
            // Nesneyi okuma amaçlı açalım  
            DBObject obj = tr.GetObject(res.ObjectId, OpenMode.ForRead);
            // Lightweight Polyline ise  
            if (obj.GetType() == typeof(Polyline))
            {
                Polyline lwp = obj as Polyline;
                // Nesne üzerinde seçilen noktanın hangi parçaya  
                // ait olduğunu araştıralım  
                Point2d searchPoint = lwp.GetClosestPointTo(res.PickedPoint, false).Add(lwp.StartPoint.GetAsVector()).Convert2d(lwp.GetPlane());
                // Hangi parçayı seçtiğimizi bulalım  
                int vn = lwp.NumberOfVertices;
                int i = 0;
                for (i = 0; i < vn; i++)
                {
                    if (lwp.OnSegmentAt(i, searchPoint, 0.00))
                        break;
                }
                // Parça tipine göre yapışma açısını bulalım  
                if (lwp.GetSegmentType(i) == SegmentType.Arc)
                {
                    CircularArc2d seg = lwp.GetArcSegment2dAt(i);
                    AcApp.SetSystemVariable("SNAPANG", seg.GetTangent(searchPoint).Direction.Angle);
                }
                else if (lwp.GetSegmentType(i) == SegmentType.Line)
                {
                    LineSegment2d seg = lwp.GetLineSegment2dAt(i);
                    AcApp.SetSystemVariable("SNAPANG", seg.Direction.Angle);
                }
            }
            // Polyline3d ise  
            else if (obj.GetType() == typeof(PolylineVertex3d))
            {
                PolylineVertex3d pv = (PolylineVertex3d)obj;
                Point3d startPnt = pv.Position;
                Polyline3d parent = (Polyline3d)pv.OwnerId.GetObject(OpenMode.ForRead);
                Point3d endPnt = parent.StartPoint;
                bool found = false;
                foreach (ObjectId vertexId in parent)
                {
                    PolylineVertex3d vertex = (PolylineVertex3d)vertexId.GetObject(OpenMode.ForRead);
                    if (found) { endPnt = vertex.Position; break; }
                    if (vertex.Equals(pv)) found = true;
                }
                if (found)
                {
                    AcApp.SetSystemVariable("SNAPANG", (startPnt.Convert2d(parent.GetPlane()) - endPnt.Convert2d(parent.GetPlane())).Angle);
                }
            }
            // Line ise  
            else if (obj.GetType() == typeof(Line))
            {
                Line ln = obj as Line;
                AcApp.SetSystemVariable("SNAPANG", ln.Angle);
            }
            // Arc ise  
            else if (obj.GetType() == typeof(Arc))
            {
                Arc arc = obj as Arc;
                AcApp.SetSystemVariable("SNAPANG", (arc.Center.Convert2d(arc.GetPlane()) - res.PickedPoint.Convert2d(arc.GetPlane())).Angle);
            }
            // Circle ise  
            else if (obj.GetType() == typeof(Circle))
            {
                Circle cr = obj as Circle;
                AcApp.SetSystemVariable("SNAPANG", (cr.Center.Convert2d(cr.GetPlane()) - res.PickedPoint.Convert2d(cr.GetPlane())).Angle);
            }
            // Text ise  
            else if (obj.GetType() == typeof(DBText))
            {
                DBText txt = obj as DBText;
                AcApp.SetSystemVariable("SNAPANG", txt.Rotation);
            }
            // MText ise  
            else if (obj.GetType() == typeof(MText))
            {
                MText txt = obj as MText;
                AcApp.SetSystemVariable("SNAPANG", txt.Rotation);
            }
            tr.Commit();
        }
    }
}
```

