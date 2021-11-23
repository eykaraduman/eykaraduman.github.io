---
title: Blok Tanımları İçin Wipeout Oluşturma
permalink: /acad/wipeout-for-block-references/
published: true
classes: wide
categories:
  - AutoCAD.Net
---



Blok tanımlarını wipeout ile maskelemek için aşağıdaki yordamı kullanabilirsiniz. Yordam sırasıyla,

- Blok tanımı içindeki nesnelere ulaşarak bloğun geometrik sınırına (Extents3d) ait minimum ve maksimum noktaları buluyor. 
- Wipeout nesnesini bu noktalardan oluşturarak AutoCAD veritabanına ekliyor. 
- Wipeout’un çizim sıralama tablosundaki (DrawOrderTable) yerini diğer blok nesnelerinin altında kalacak şekilde yeniden belirliyor. 
- Blok tanımındaki bu değişikliğin blok referanslarına yansıması için bu referansları güncelliyor.

```c#
public static void CreateWipeoutForBlockDefination(string BlockDefName)
{
    Database db = Application.DocumentManager.MdiActiveDocument.Database;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        BlockTable bt = tr.GetObject(db.BlockTableId, OpenMode.ForRead, false, true) as BlockTable;
        if (bt.Has(BlockDefName))
        {
            BlockTableRecord btr = 
                tr.GetObject(bt[BlockDefName], OpenMode.ForWrite, false, true) as BlockTableRecord;
    
            Extents3d blockExtents = new Extents3d();
            ObjectIdCollection blockIds = new ObjectIdCollection();
            foreach (ObjectId id in btr)
            {
                Entity ent = tr.GetObject(id, OpenMode.ForRead, false, true) as Entity;
                if (ent.Bounds.HasValue)
                {
                    Extents3d entExtents = ent.GeometricExtents;
                    blockExtents.AddPoint(entExtents.MinPoint);
                    blockExtents.AddPoint(entExtents.MaxPoint);
                }
    
                blockIds.Add(id);
            } 
    
            Point3d BL = blockExtents.MinPoint;
            Point3d UR = blockExtents.MaxPoint;
    
            double w = UR.X - BL.X;
            double h = UR.Y - BL.Y;
    
            Point3d BR = new Point3d(BL.X + w, BL.Y, 0.0);
            Point3d UL = new Point3d(BL.X, BL.Y + h, 0.0);
    
            Point2dCollection pts = new Point2dCollection(5);
            pts.Add(new Point2d(UL.X, UL.Y));
            pts.Add(new Point2d(UR.X, UR.Y));
            pts.Add(new Point2d(BR.X, BR.Y));
            pts.Add(new Point2d(BL.X, BL.Y));
            pts.Add(new Point2d(UL.X, UL.Y));
    
            Wipeout wo = new Wipeout();
            wo.SetDatabaseDefaults();
            wo.SetFrom(pts, Vector3d.ZAxis);
    
            ObjectId idWo = btr.AppendEntity(wo);
            tr.AddNewlyCreatedDBObject(wo, true);
    
            DrawOrderTable dot = 
                (DrawOrderTable)tr.GetObject(btr.DrawOrderTableId, OpenMode.ForWrite);
            dot.MoveAbove(blockIds, idWo);
    
            TypedValue[] tyVals = new TypedValue[6]{ 
                new TypedValue(-4, "<AND"),
                new TypedValue((int)DxfCode.Start, "INSERT"),
                new TypedValue(-4, "<OR"),
                new TypedValue((int)DxfCode.BlockName, BlockDefName),
                new TypedValue(-4, "OR>"),
                new TypedValue(-4, "AND>")
            };
    
            PromptSelectionResult res = ed.SelectAll(new SelectionFilter(tyVals));
            if (res.Status == PromptStatus.OK)
            {
                if (res.Value != null)
                {
                    foreach (ObjectId id in res.Value.GetObjectIds())
                    {
                        BlockReference br = 
                            tr.GetObject(id, OpenMode.ForWrite, false, true) as BlockReference;
                        br.RecordGraphicsModified(true);
                    }
                }
            }
        }                
        tr.Commit();
    }
}
```

