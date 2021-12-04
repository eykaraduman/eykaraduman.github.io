---
title: "EK-2: AutoCAD Veritabanı Yardımcısı (DatabaseHelper.cs)"
permalink: /autocad-net-programming/databese-helper/
classes: wide
comments: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

`DatabaseHelper` statik sınıfı, AutoCAD .NET API veritabanı işlemlerini kolaylaştıran yardımcı bir sınıftır. Varlıkları model ve kağıt uzaylarına ekleyen metotlar barındırmaktadır.

![ClassDiagram](/assets/images/cd-database-helper.png "ClassDiagram")

**DatabaseHelper.cs**:

```c#
using System.Collections.Generic;
using System.Linq;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace PgAutoCAD
{
    /// <summary>
    /// AutoCAD uzayları numaralandırması.
    /// </summary>
    public enum AcadSpace
    {
        Current, Model, Paper
    }

    /// <summary>
    /// Veritabanı işlemleri için yardımcı.
    /// </summary>
    public static class DatabaseHelper
    {
        /// <summary>
        /// Model uzayı kimliğini verir.
        /// </summary>
        /// <param name="db">Veritabanı.</param>
        /// <returns></returns>
        public static ObjectId ModelSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockModelSpaceId(db);
        }

        /// <summary>
        /// Kağıt uzayı kimliğini verir.
        /// </summary>
        /// <param name="db">Veritabanı</param>
        /// <returns></returns>
        public static ObjectId PaperSpaceId(this Database db)
        {
            return SymbolUtilityServices.GetBlockPaperSpaceId(db);
        }

        /// <summary>
        /// Mevcut uzayın kağıt uzayı olup olmadığını kontrol eder.
        /// </summary>
        /// <param name="db">Veritabanı.</param>
        /// <returns></returns>
        public static bool IsPaperSpace(this Database db)
        {
            if (db.TileMode)
                return false;

            Editor ed = Active.Editor;
            if (db.PaperSpaceVportId == ed.CurrentViewportObjectId)
                return true;

            return false;
        }

        /// <summary>
        /// Mevcut uzayın model uzayı olup olmadığını kontrol eder.
        /// </summary>
        /// <param name="db">Veritabanı</param>
        /// <returns></returns>
        public static bool IsModelSpace(this Database db)
        {
            return !IsPaperSpace(db);
        }

        /// <summary>
        /// Seçilen uzaya varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="space">Varlığın ekleneceği uzay.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimliği.</returns>
        internal static ObjectId AddToSpace(this Entity entity, AcadSpace space, Database db = null)
        {
            db = db ?? HostApplicationServices.WorkingDatabase;

            ObjectId spaceId;
            switch (space)
            {
                case AcadSpace.Current:
                    spaceId = db.CurrentSpaceId;
                    break;
                case AcadSpace.Model:
                    spaceId = db.ModelSpaceId();
                    break;
                case AcadSpace.Paper:
                    spaceId = db.PaperSpaceId();
                    break;
                default:
                    spaceId = db.CurrentSpaceId;
                    break;
            }

            using (var trans = db.TransactionManager.StartTransaction())
            {
                var currentSpace = (BlockTableRecord)trans.GetObject(spaceId, OpenMode.ForWrite, false);
                var id = currentSpace.AppendEntity(entity);
                trans.AddNewlyCreatedDBObject(entity, true);
                trans.Commit();
                return id;
            }
        }

        /// <summary>
        /// Seçilen uzaya varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="space">Varlıkların ekleneceği uzay.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesnelerin kimlikleri.</returns>
        internal static ObjectId[] AddToSpace(this IEnumerable<Entity> entities, AcadSpace space, Database db = null)
        {
            db = db ?? HostApplicationServices.WorkingDatabase;

            ObjectId spaceId = ObjectId.Null;
            switch (space)
            {
                case AcadSpace.Current:
                    spaceId = db.CurrentSpaceId;
                    break;
                case AcadSpace.Model:
                    spaceId = db.ModelSpaceId();
                    break;
                case AcadSpace.Paper:
                    spaceId = db.PaperSpaceId();
                    break;
                default:
                    spaceId = db.CurrentSpaceId;
                    break;
            }
            using (var trans = db.TransactionManager.StartTransaction())
            {
                var currentSpace = (BlockTableRecord)trans.GetObject(spaceId, OpenMode.ForWrite, false);
                var ids = entities
                    .ToArray()
                    .Select(entity =>
                    {
                        var id = currentSpace.AppendEntity(entity);
                        trans.AddNewlyCreatedDBObject(entity, true);
                        return id;
                    })
                    .ToArray();

                trans.Commit();
                return ids;
            }
        }


        /// <summary>
        /// Açık olan uzaya varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimliği.</returns>
        public static ObjectId AddToCurrentSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Current, db);
        }

        /// <summary>
        /// Açık olan uzaya varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToCurrentSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Current, db);
        }


        /// <summary>
        /// Model uzayına varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId AddToModelSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Model, db);
        }

        /// <summary>
        /// Model uzayına varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToModelSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Model, db);
        }

        /// <summary>
        /// Kağıt uzayına varlık ekler.
        /// </summary>
        /// <param name="entity">Eklenecek varlık.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId AddToPaperSpace(this Entity entity, Database db = null)
        {
            return AddToSpace(entity, AcadSpace.Paper, db);
        }

        /// <summary>
        /// Kağıt uzayına varlıklar ekler.
        /// </summary>
        /// <param name="entities">Eklenecek varlıklar.</param>
        /// <param name="db">Veritabanı.</param>
        /// <returns>Nesne kimlikleri.</returns>
        public static ObjectId[] AddToPaperSpace(this IEnumerable<Entity> entities, Database db = null)
        {
            return AddToSpace(entities, AcadSpace.Paper, db);
        }
    }
}
```
