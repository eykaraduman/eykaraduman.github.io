---
title: Etiketli Blok Referans Oluşturumak
permalink: /acad/create-blok-reference/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD.Net (C#) ile etiketli bir blok tanımının nasıl oluşturulacağını bir önceki yazımda göstermeye çalışmıştım. 
Mevcut bir blok tanıma ait referansın çizime nasıl ekleneceğini merak ediyorsanız aşağıdaki kaynak kodu inceleyebililirsiniz. 

```c#
using System;
using System.Collections.Generic;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.Geometry;
using Autodesk.AutoCAD.EditorInput;
[assembly: CommandClass(typeof(BlokOlusturma.Commands))]
namespace BlokOlusturma
{
    public class Commands
    {
        [CommandMethod("AbcProgramlama", "BlokOlustur", "BlkOls", CommandFlags.Modal)]
        public void EtiketliBlokOlusturma()
        {
            // Blok ismi  
            string blockName = "ETIKETLI-NOKTA";
            // Blok konum noktası  
            Point3d insPnt = new Point3d();
            // 1. Bloğumuzun içereceği nesneleri oluşturması  
            //  Bloğumuz bir daire ve bu dairenin ortasında nokta numarasını  
            //  bize gösterecek bir etiketten oluşacak  
            // 1a. Daire nesnesinin oluşturulması  
            Circle circle = new Circle(insPnt, Vector3d.ZAxis, 0.20d);
            circle.ColorIndex = 1;
            // 1b. Etiketin oluşturulması  
            AttributeDefinition attDef = new AttributeDefinition();
            attDef.SetDatabaseDefaults();
            attDef.HorizontalMode = TextHorizontalMode.TextCenter;
            attDef.VerticalMode = TextVerticalMode.TextVerticalMid;
            attDef.TextString = "";
            attDef.Tag = "No";
            attDef.Prompt = "Nokta numarası";
            attDef.Rotation = 0.0d;
            attDef.Height = 0.15d;
            attDef.AlignmentPoint = insPnt;
            attDef.ColorIndex = 4;
            // 2. Block kaydının oluşturulması :  
            Document doc = Application.DocumentManager.MdiActiveDocument;
            Database db = doc.Database;
            Editor ed = doc.Editor;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                // Çizim blok tablosuna erişim  
                BlockTable bt = (BlockTable)tr.GetObject(db.BlockTableId, OpenMode.ForRead);
                // Blok kaydının blok tablosundaki varlığının kontrolü  
                if (!bt.Has(blockName))
                {
                    BlockTableRecord btr = new BlockTableRecord();
                    // Blok kayıt özelliklerinin ayarlanması  
                    btr.Name = blockName;
                    btr.Origin = insPnt;
                    // Blok kaydının blok tablosuna eklenmesi  
                    bt.UpgradeOpen();
                    ObjectId btrId = bt.Add(btr);
                    tr.AddNewlyCreatedDBObject(btr, true);
                    // Biraz önce oluşturduğumuz nesnelerin blok kaydına eklenmesi  
                    btr.AppendEntity(circle);
                    tr.AddNewlyCreatedDBObject(circle, true);
                    btr.AppendEntity(attDef);
                    tr.AddNewlyCreatedDBObject(attDef, true);
                    // İşlemlerin onaylanması  
                    tr.Commit();
                }
            }
        }
    }
}
```

