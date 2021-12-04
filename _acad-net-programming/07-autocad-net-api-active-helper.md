---
title: "AutoCAD Çalışma Zamanı Ortam Yardımcısı (Active.cs)"
permalink: /autocad-net-programming/runtime-active-helper/
classes: wide
comments: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

AutoCAD .NET API ile eklenti geliştirirken tekrar eden kodlardan kaçınmak, kopyala ve yapıştır işlemlerini azaltmak için çeşitli sınıflar ve metotlar tasarlamak zamanla kaçınılmaz hale gelir. Bu nedenle AutoCAD .NET API'de en çok kullanılan nesnelere kolay erişmek için **Active.cs** sınıfı geliştirilmiştir. **Active.cs** sınıfı, kılavuzun ilerleyen bölümlerinde kullanılacaktır. (Bu sınıfın geliştirilmesinde, *Being a Remarkable C# .NET AutoCAD Developer, AU 2015, Scott McFarlane* esin kaynağı olmuştur.)

```c#
using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.DatabaseServices;
using Autodesk.AutoCAD.EditorInput;

namespace PgAutoCAD
{
    /// <summary>
    /// AutoCAD çalışma ortamı aktif nesnelerine kolay erişim sağlar.
    /// </summary>
    public static class Active
    {
        /// <summary>
        /// Aktif Document nesnesini döndürür.
        /// </summary>
        public static Document Document => Application.DocumentManager.MdiActiveDocument;

        /// <summary>
        ///  Aktif Editor nesnesini döndürür.
        /// </summary>
        public static Editor Editor => Document.Editor;


        /// <summary>
        ///  Aktif Database nesnesini döndürür.
        /// </summary>
        public static Database Database => Document.Database;

        /// <summary>
        /// Aktif editörü kullanarak komut satırına mesaj yazdırır.
        /// </summary>
        /// <param name="message">Yazdırılacak mesaj.</param>
        public static void WriteMessage(string message)
        {
            Editor.WriteMessage(message);
        }

        /// <summary>
        /// Aktif editörü kullanarak komut satırına
        /// String.Format biçimleyicisiyle mesaj yazdırır.
        /// </summary>
        /// <param name="message">Biçimlendiricileri içeren mesaj.</param>
        /// <param name="parameter">Biçimlendirme dizesinde değiştirilecek değişkenler.</param>
        public static void WriteMessage(string message, params object[] parameter)
        {
            Editor.WriteMessage(message, parameter);
        }
    }
}
```

Artık doküman, veritabanı ve editör nesnelerine kolayca erişebiliriz:

```c#
var doc = Active.Document;
var db = Active.Database;
var ed = Active.Editor;
```
