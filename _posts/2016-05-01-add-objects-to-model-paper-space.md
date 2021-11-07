---
title: Model ve Kağıt Uzayına Nesne Ekleme
permalink: /acad/add-objects-to-model-paper-space/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

 <strong>AcadDatabase</strong> sınıfının aşağıdaki yordamlarını, AutoCAD nesnelerini, model ve kağıt uzayına eklemek için kullanabilirsiniz. Bu yordamların bazıları tek bir nesneyi, diğerleri ise nesne listelerini (DBObjectCollection) parametre olarak kabul ediyor. 

```cpp
public static ObjectId AddToCurrentSpace(this Entity ent)
public static ObjectId AddToCurrentSpace(this Entity ent, Database db)
public static ObjectIdCollection AddToCurrentSpace(this Database db, DBObjectCollection objects)
public static ObjectId AddToModelSpace(this Entity ent)
public static ObjectId AddToModelSpace(this Entity ent, Database db)
public static ObjectIdCollection AddToModelSpace(this Database db, DBObjectCollection objects)
public static ObjectId AddToModelSpace(this Entity ent, Database db, Transaction tr)
public static ObjectId AddToPaperSpace(this Entity ent)
public static ObjectId AddToPaperSpace(this Entity ent, Database db)
public static ObjectId AddToPaperSpace(this Entity ent, Database db, Transaction tr)
public static ObjectIdCollection AddToPaperSpace(this Database db, DBObjectCollection objects)
```

```cpp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Autodesk.AutoCAD.Geometry;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.Colors;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.Internal;
using Autodesk.AutoCAD.Runtime;
using AcApp = Autodesk.AutoCAD.ApplicationServices.Application;

namespace PgAutoCAD
{
    public static class AcadDatabase
    {
        public static ObjectId ModelSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockModelSpaceId(db);
        }

        public static ObjectId PaperSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockPaperSpaceId(db);
        }

        public static bool IsPaperSpace(this Database db)
        {
            if (db.TileMode)
                return false;

            Editor ed = AcadApplication.Editor;
            if (db.PaperSpaceVportId == ed.CurrentViewportObjectId)
                return true;

            return false;
        }

        public static ObjectId AddToCurrentSpace(this Entity ent)
        {
            return ent.AddToCurrentSpace(AcadApplication.Database);
        }

        public static ObjectId AddToCurrentSpace(this Entity ent, Database db)
        {
            ObjectId id;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                id = ((BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite, false)).AppendEntity(ent);
                tr.AddNewlyCreatedDBObject(ent, true);
                tr.Commit();
            }
            return id;
        }

        public static ObjectIdCollection AddToCurrentSpace(this Database db, DBObjectCollection objects)
        {
            ObjectIdCollection ids = new ObjectIdCollection();
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                foreach (Entity item in objects)
                {
                    ObjectId id = ((BlockTableRecord)tr.GetObject(db.CurrentSpaceId,
                    OpenMode.ForWrite, false)).AppendEntity(item);
                    tr.AddNewlyCreatedDBObject(item, true);
                    ids.Add(id);
                }
                tr.Commit();
            }
            return ids;
        }

        public static ObjectId AddToModelSpace(this Entity ent)
        {
            return ent.AddToModelSpace(AcadApplication.Database);
        }

        public static ObjectId AddToModelSpace(this Entity ent, Database db)
        {
            ObjectId id;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                id = ((BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                    OpenMode.ForWrite, false)).AppendEntity(ent);
                tr.AddNewlyCreatedDBObject(ent, true);
                tr.Commit();
            }
            return id;
        }

        public static ObjectIdCollection AddToModelSpace(this Database db, DBObjectCollection objects)
        {
            ObjectIdCollection ids = new ObjectIdCollection() ;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                foreach (Entity item in objects)
                {
                    ObjectId id = ((BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                    OpenMode.ForWrite, false)).AppendEntity(item);
                    tr.AddNewlyCreatedDBObject(item, true);
                    ids.Add(id);
                }
                tr.Commit();
            }
            return ids;
        }

        public static ObjectId AddToModelSpace(this Entity ent, Database db, Transaction tr)
        {
            ObjectId id;
            BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
            id = ((BlockTableRecord)tr.GetObject(bt[BlockTableRecord.ModelSpace],
                OpenMode.ForWrite, false)).AppendEntity(ent);
            tr.AddNewlyCreatedDBObject(ent, true);
            return id;
        }

        public static ObjectId AddToPaperSpace(this Entity ent)
        {
            return ent.AddToPaperSpace(AcadApplication.Database);
        }

        public static ObjectId AddToPaperSpace(this Entity ent, Database db)
        {
            ObjectId id;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                id = ((BlockTableRecord)tr.GetObject(LayoutManager.Current.GetLayoutId(LayoutManager.Current.CurrentLayout),
                    OpenMode.ForWrite, false)).AppendEntity(ent);
                tr.AddNewlyCreatedDBObject(ent, true);
                tr.Commit();
            }
            return id;
        }

        public static ObjectId AddToPaperSpace(this Entity ent, Database db, Transaction tr)
        {
            ObjectId id;
            BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
            id = ((BlockTableRecord)tr.GetObject(bt[BlockTableRecord.PaperSpace],
                OpenMode.ForWrite, false)).AppendEntity(ent);
            tr.AddNewlyCreatedDBObject(ent, true);
            return id;
        }

        public static ObjectIdCollection AddToPaperSpace(this Database db, DBObjectCollection objects)
        {
            ObjectIdCollection ids = new ObjectIdCollection();
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead, false);
                foreach (Entity item in objects)
                {
                    ObjectId id = ((BlockTableRecord)tr.GetObject(bt[BlockTableRecord.PaperSpace],
                    OpenMode.ForWrite, false)).AppendEntity(item);
                    tr.AddNewlyCreatedDBObject(item, true);
                    ids.Add(id);
                }
                tr.Commit();
            }
            return ids;
        }
    }
}
```

