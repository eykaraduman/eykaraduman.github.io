---
title: AutoCAD.Net ile Programlamaya Giriş 
permalink: /acad/autocad-net-start/
published: true
classes: wide
categories:
  - AutoCAD.Net
---

C#, F#, Visul Basic.Net vb. programlama dillerinden birini biliyorsanız AutoCAD.Net (Managed ObjectArx Wrapper) ile programlamaya giriş yapabilirsiniz. Benim tercihim .Net platformunun en popüler dillerinden biri olan C#’dan yana.
 Autocad için .Net uygulaması yazmaya başlamadan önce aşağıda gereksinimleri edinip bilgisayarınıza kurmalısınız.

- AutoCAD 2014 (Yazdığınız kodu denemek için gerekli)
- ObjectARX 2014 SDK (Uygulama Geliştirme Aracı)
- AutoCAD 2014 Dotnet Wizards (Visual Studio şablonları)
- Visual Studio 2010 ya da 2012

Visual Studio 2012 ile yeni bir AutoCAD.Net projesi oluşturmak için sırasıyla aşağıdaki adımları izleyin:

1. **File->New->Project…** sekmesini seçin.
2. Proje tipini (AutoCAD 2014 CSharp plug in), ismini ve projenin kaydedileceği yeri sırayla belirleyin. (**Şekil-1**)
3. Sihirbaz sizin için AutoCAD’in ve ObjectARX SDK’nın kurulu olduğu klasörleri bulacaktır. Bulamazsa kendiniz seçin. (**Şekil-2**)

![Şekil-1](/assets/images/AutoCADNetGiris-1.png)

Şekil-1

![Şekil-2](/assets/images/AutoCADNetGiris-2.png)

Şekil-2

**AutoCAD Managed C# Uygulama Sihirbazı** bizim yerimize proje hazırlıklarını yapacak ve dört örnek AutoCAD komutu için 
gerekli kodu oluşturacaktır. Visual Studio proje referanslarına göz atarsanız sihirbazın, AutoCAD.NET’e özgü **AcCoreMgd.dll** 
(AutoCAD çekirdek motoru için sınıflar), **AcDbMgd.dll** (ObjectArx AcDb ve ilişkili sınıflarını içerir) ve **AcMgd.dll** 
(AutoCAD uygulama sınıflarını içerir) dosyalarını referans listesine eklediğini görebilirsiniz.

```csharp
 using Autodesk.AutoCAD.Runtime;  
 using Autodesk.AutoCAD.ApplicationServices;  
 using Autodesk.AutoCAD.DatabaseServices;  
 using Autodesk.AutoCAD.Geometry;  
 using Autodesk.AutoCAD.EditorInput;  
 // This line is not mandatory, but improves loading performances  
 [assembly: CommandClass(typeof(AutoCAD_CSharp_Plugin.MyCommands))]  
 namespace AutoCAD_CSharp_Plugin  
 {  
   // This class is instantiated by AutoCAD for each document when  
   // a command is called by the user the first time in the context  
   // of a given document. In other words, non static data in this class  
   // is implicitly per-document!  
   public class MyCommands  
   {  
     // The CommandMethod attribute can be applied to any public member   
     // function of any public class.  
     // The function should take no arguments and return nothing.  
     // If the method is an intance member then the enclosing class is   
     // intantiated for each document. If the member is a static member then  
     // the enclosing class is NOT intantiated.  
     //  
     // NOTE: CommandMethod has overloads where you can provide helpid and  
     // context menu.  
     // Modal Command with localized name  
     [CommandMethod("MyGroup", "MyCommand", "MyCommandLocal", CommandFlags.Modal)]  
     public void MyCommand() // This method can have any name  
     {  
       // Put your command code here  
     }  
     // Modal Command with pickfirst selection  
     [CommandMethod("MyGroup", "MyPickFirst", "MyPickFirstLocal", 
     CommandFlags.Modal | CommandFlags.UsePickSet)]  
     public void MyPickFirst() // This method can have any name  
     {  
       PromptSelectionResult result = 
       Application.DocumentManager.MdiActiveDocument.Editor.GetSelection();  
       if (result.Status == PromptStatus.OK)  
       {  
         // There are selected entities  
         // Put your command using pickfirst set code here  
       }  
       else  
       {  
         // There are no selected entities  
         // Put your command code here  
       }  
     }  
     // Application Session Command with localized name  
     [CommandMethod("MyGroup", "MySessionCmd", "MySessionCmdLocal", 
     CommandFlags.Modal | CommandFlags.Session)]  
     public void MySessionCmd() // This method can have any name  
     {  
       // Put your command code here  
     }  
     // LispFunction is similar to CommandMethod but it creates a lisp   
     // callable function. Many return types are supported not just string  
     // or integer.  
     [LispFunction("MyLispFunction", "MyLispFunctionLocal")]  
     public int MyLispFunction(ResultBuffer args) // This method can have any name  
     {  
       // Put your command code here  
       // Return a value to the AutoCAD Lisp Interpreter  
       return 1;  
     }  
   }  
 }  
```

Sihirbazın oluşturduğu yukarıdaki kodu inceleyecek olursanız, **CommandMethodAttribute** için gerekli **Autodesk.AutoCAD.Runtime** isim 
uzayınının projenize eklendiğini ve uygulamanız için **MyCommands** adlı bir sınıf oluşturduğunu görebilirsiniz. 
**MyCommands** sınıfı içerisinde, komut metodları (CommandMethods) olarak bildirilen geridönüş fonksiyonları tanımlanabilir. 
Bu geridönüş fonksiyonlarının erişilebilirlik anahtarı mutlaka public olmalıdır. 
Ayrıca bu fonksiyonların MyCommands sınıfı içinde tanımlanması gibi bir zorunluluk da yoktur, 
**public** erişime sahip her sınıf içerisinde tanımlanabilmektedir.

Artık Visual Studio 2012 ortamında **“Build->Build AutoCAD CSharp Plugin”** sekmesini seçerek derlediğimiz sonuç 
dll’yi (**“..\Release\ AutoCAD CSharp Plugin.dll”** yada **“..\Debug\ AutoCAD CSharp Plugin.dll”**) yüklemek için 
AutoCAD Netload komutu kullanabilirsiniz.

