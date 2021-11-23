---
title: Nesne Olayları
permalink: /acad/object-events/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD nesnelerinin eklenmesini, silinmesini ve bu nesnelerde yapılan değişiklikleri dinlemekte kullanılan olaylar aşağıda gibi: 

```c#
public event Autodesk.AutoCAD.DatabaseServices.ObjectEventHandler ObjectAppended  
public event Autodesk.AutoCAD.DatabaseServices.ObjectErasedEventHandler ObjectErased  
public event Autodesk.AutoCAD.DatabaseServices.ObjectEventHandler ObjectModified  
public event Autodesk.AutoCAD.DatabaseServices.ObjectEventHandler ObjectOpenedForModify  
public event Autodesk.AutoCAD.DatabaseServices.ObjectEventHandler ObjectReappended  
public event Autodesk.AutoCAD.DatabaseServices.ObjectEventHandler ObjectUnappended  
```

Nesne olaylarını dinleyebilmek için yapmamız gereken tek şey bu olayları AutoCAD veritabanıyla ilişkilendirmek ve her ilişkilendirdiğimiz olay için bir geri dönüş fonksiyonu yazmak. Geri dönüş fonksiyonlarıyla da yakaladığımız nesneler üzerinde istediğimiz işlemleri yapabiliriz. Bu olayların nasıl işlediğini anlamak için ObjectAppended (nesne eklendi) ve ObjectErased (nesne silindi) olaylarını dinleyen aşağıdaki örnek kodu inceleyebilirsiniz.

```c#
using System;
using Autodesk.AutoCAD.Runtime;
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

[assembly: ExtensionApplication(typeof(Autodesk.AutoCAD.ObjectEvents.MyPlugin))]

namespace Autodesk.AutoCAD.ObjectEvents
{
    public class MyPlugin : IExtensionApplication
    {
        void IExtensionApplication.Initialize()
        {
            Document doc = Application.DocumentManager.MdiActiveDocument;
            Database db = doc.Database;
            // Olayların veritabanıyla ilişkilendirilmesi  
            db.ObjectAppended += new ObjectEventHandler(NesneEklendi);
            db.ObjectErased += new ObjectErasedEventHandler(NesneSilindi);
        }
        void IExtensionApplication.Terminate()
        {
            try
            {
                Document doc = Application.DocumentManager.MdiActiveDocument;
                Database db = doc.Database;
                // Olayların veritabanıyla ilişkilerinin sonlandırılması  
                db.ObjectAppended -= new ObjectEventHandler(NesneEklendi);
                db.ObjectErased -= new ObjectErasedEventHandler(NesneSilindi);
            }
            catch (System.Exception)
            {
            }
        }
        // Database.ObjectAppended için geri dönüş fonksiyonu  
        private void NesneEklendi(
        object sender, ObjectEventArgs e)
        {
            Autodesk.AutoCAD.ApplicationServices.Application.ShowAlertDialog("Değiştirilen Nesne :\n" + e.DBObject.GetType().ToString());
        }
        //Database.ObjectErased olayı için geridönüş fonksiyonu  
        private void NesneSilindi(
        object sender, ObjectErasedEventArgs e)
        {
            ObjectId id = e.DBObject.ObjectId;
            if (e.Erased)
            {
                Autodesk.AutoCAD.ApplicationServices.Application.ShowAlertDialog("Silinen Nesne :\n" + e.DBObject.GetType().ToString());
            }
        }
    }
}
```

