---
title: Uygulama Sözlüğü
permalink: /acad/application-dictionary/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD çizimlerinde, DBDictionary nesnesini kullanarak veri depolayabiliriz. 
Bu sözlükler, çizime özgü uygulama ve kullanıcı ayarlarını barındırmak için en etkili yollardan biri.

Eğer bu sözlüklerin nasıl oluşturulduğunu bilmiyorsanız;

- Uygulama sözlüğü nesne kimliğinin yaratılması ya da eldesini
- Uygulama anahtarlarının oluşturulması ve okunmasını
- Ayrıca yukarıdaki işlemleri deneyebileceğiniz WriteTest(...) yordamını

aşağıdıdaki **AcadCustomApplicationDictionary** sınıfında bulabilirsiniz. 

```c#
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace AbcAcadTools
{
    public class AcadCustomApplicationDictionary
    {
        /// <summary>
        /// Uygulama adı
        /// </summary>
        public string AppName { get; private set; }

        /// <summary>
        /// 
        /// </summary>
        /// <param name="appName">Uygulama adı</param>
        public AcadCustomApplicationDictionary(string appName)
        {
            AppName = appName;
        }
    
        /// <summary>
        /// Uygulama sözlüğü kimliğinin elde edilmesi.
        /// Eğer çizim içinde uygulama sözlüğü mevcut 
        /// değilse yeni bir sözlük oluşturur.
        /// </summary>
        /// <returns>Uygulama kimliği (ObjectId)</returns>
        private ObjectId ApplicationId()
        {
            ObjectId AppId = ObjectId.Null;
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
    
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary NOD = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);
    
                if (NOD.Contains(AppName))
                {
                    AppId = NOD.GetAt(AppName);
                }
                else
                {
                    NOD.UpgradeOpen();
                    DBDictionary appDict = new DBDictionary();
                    AppId = NOD.SetAt(AppName, appDict);
                    tr.AddNewlyCreatedDBObject(appDict, true);
                    tr.Commit();
                }
            }
            return AppId;
        }
    
        /// <summary>
        /// Uygulama sözlüğü için tekil değerli anahtar oluşturulması.
        /// </summary>
        /// <param name="key">Anahtar adı</param>
        /// <param name="type">Anahtar altında saklanacak tekil değerin dxf kodu</param>
        /// <param name="val">Anahtarda saklanacak değer</param>
        /// <param name="ifExistUpdate">Eğer anahtar mevcutsa değerlerin güncellenip güncellenmeyeceğinin kontrolü</param>
        public void CreateKey(string key, DxfCode type, object val, bool ifExistsUpdate)
        {
            ObjectId AppId = ApplicationId();
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary appDict = (DBDictionary)tr.GetObject(AppId, OpenMode.ForWrite);
                Xrecord dataRec;
                if (!appDict.Contains(key))
                {
                    dataRec = new Xrecord();
                    dataRec.Data = new ResultBuffer(new TypedValue((int)type, val));
                    appDict.SetAt(key, dataRec);
                    tr.AddNewlyCreatedDBObject(dataRec, true);
                    tr.Commit();
                }
                else
                {
                    if (ifExistsUpdate)
                    {
                        dataRec = (Xrecord)tr.GetObject(appDict.GetAt(key), OpenMode.ForWrite);
                        dataRec.Data = new ResultBuffer(new TypedValue((int)type, val));
                        tr.Commit();
                    }
                }
            }
        }
    
        /// <summary>
        /// Uygulama sözlüğü için anahtar oluşturulması.
        /// </summary>
        /// <param name="key">Anahtar adı</param>
        /// <param name="dataRec">Anahtar altında saklanacak değerlere ait XRecord nesnesi</param>
        /// <param name="ifExistUpdate">Eğer anahtar mevcutsa değerlerin güncellenip güncellenmeyeceğinin kontrolü</param>
        public void CreateKey(string key, Xrecord dataRec, bool ifExistsUpdate)
        {
            ObjectId AppId = ApplicationId();
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary appDict = (DBDictionary)tr.GetObject(AppId, OpenMode.ForWrite);
                if (!appDict.Contains(key))
                {
                    appDict.SetAt(key, dataRec);
                    tr.AddNewlyCreatedDBObject(dataRec, true);
                }
                else
                {
                    if (ifExistsUpdate)
                        appDict.SetAt(key, dataRec);
                }
                tr.Commit();
            }
        }
    
        /// <summary>
        /// Anahtarın okunması.
        /// </summary>
        /// <param name="key">Okunacak anahtar adı</param>
        /// <param name="index">Anahtara ait XRecord nesnesinde aranacak indeks</param>
        /// <returns>Aranan indekse ait nesne. Bulamazsa null döndürür.</returns>
        public object ReadKey(string key, int index)
        {
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary NOD = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);
                if (!NOD.Contains(AppName))
                {
                    return null;
                }
                else
                {
                    DBDictionary appDict = (DBDictionary)tr.GetObject(NOD.GetAt(AppName), OpenMode.ForRead);
                    if (!appDict.Contains(key))
                    {
                        return null;
                    }
                    else
                    {
                        Xrecord rec = (Xrecord)tr.GetObject(appDict.GetAt(key), OpenMode.ForRead);
                        TypedValue[] tv = rec.Data.AsArray();
                        if (index > tv.Length - 1)
                            return null;
                        else
                            return tv[index].Value;
                    }
                }
            }
        }
    
        /// <summary>
        /// Anahtarın okunması.
        /// </summary>
        /// <param name="key">Okunacak anahtar adı</param>
        /// <returns>Anahtara ait XRecord nesnesi. Bulamazsa null döndürür.</returns>
        public Xrecord ReadKey(string key)
        {
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary NOD = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);
                if (!NOD.Contains(AppName))
                {
                    return null;
                }
                else
                {
                    DBDictionary appDict = (DBDictionary)tr.GetObject(NOD.GetAt(AppName), OpenMode.ForRead);
                    if (!appDict.Contains(key))
                    {
                        return null;
                    }
                    else
                    {
                        return
                            (Xrecord)tr.GetObject(appDict.GetAt(key), OpenMode.ForRead);
                    }
                }
            }
        }
    
        /// <summary>
        /// Seçilen anahtarın Autocad komut satırına yazdırılması
        /// </summary>
        /// <param name="key">Anahtar adı</param>
        public void WriteKeyToEditor(string key)
        {
            Xrecord rec = ReadKey(key);
            if (rec != null)
            {
                Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
                ed.WriteMessage("\n Key name: " + key);
                TypedValue[] tvl = rec.Data.AsArray();
                foreach (TypedValue tv in tvl)
                {
                    ed.WriteMessage("\n  Type code: {0}, Value: {1}",
                        ((DxfCode)tv.TypeCode).ToString(), tv.Value);
                }
            }
        }
    
        /// <summary>
        /// Tüm anahtarların Autocad komut satırına yazdırılması
        /// </summary>
        public void WriteKeysToEditor()
        {
            Database db = Application.DocumentManager.MdiActiveDocument.Database;
            using (Transaction tr = db.TransactionManager.StartTransaction())
            {
                DBDictionary NOD = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId, OpenMode.ForRead);
                ObjectId AppId = NOD.GetAt(AppName);
                if (AppId != ObjectId.Null)
                {
                    DBDictionary appDict = (DBDictionary)tr.GetObject(AppId, OpenMode.ForRead);
                    DbDictionaryEnumerator DctEnumerator = appDict.GetEnumerator();
                    while (DctEnumerator.MoveNext())
                    {
                        WriteKeyToEditor(DctEnumerator.Current.Key);
                    }
                }
            }
        }
    
    }
    
    public static class ApplicationDictionaryTest
    {
        public static void WriteTest(string AppName)
        {
            AcadCustomApplicationDictionary dict = new AcadCustomApplicationDictionary(AppName);
    
            ResultBuffer rbUserSet = new ResultBuffer(
                new TypedValue((int)DxfCode.Text, "AbcProgramlama") // User name
                );
            Xrecord userXrec = new Xrecord();
            userXrec.Data = rbUserSet;
    
            ResultBuffer rbDwgSet = new ResultBuffer(
                new TypedValue((int)DxfCode.Real, 100.00), // Scale
                new TypedValue((int)DxfCode.Text, "cm"), // Dwg unit
                new TypedValue((int)DxfCode.Elevation, 125.00) // Reference elevation
                );
            Xrecord dwgXrec = new Xrecord();
            dwgXrec.Data = rbDwgSet;


            dict.CreateKey("UserSettings", userXrec, true);
            dict.CreateKey("DwgSettings", dwgXrec, true);
    
            rbUserSet.Dispose();
            rbDwgSet.Dispose();
        }
    
        public static void ReadTest(string AppName)
        {
            AcadCustomApplicationDictionary dict = new AcadCustomApplicationDictionary(AppName);
    
            Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    
            ed.WriteMessage("\nApplication Name: {0} ", dict.AppName);
    
            dict.WriteKeysToEditor();
        }
    }
}
```

