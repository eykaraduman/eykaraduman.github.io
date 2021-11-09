---
title: "AutoCAD .NET Eklentisi Oluşturma"
permalink: /autocadnet/beginnertutorial/create-project-from-stratch/
toc: true
classes: wide
---

AutoCAD için .Net uygulaması geliştirmeye başlamadan önce aşağıda gereksinimler kurulmalıdır. AutoCAD ve Visual Studio sürümleri aşağıda verilenlere göre farklılık gösterebilir. Ancak ObjectARX SDK, AutoCAD sürümü ile uyumlu olmalıdır.

1. AutoCAD 2013
2. ObjectARX 2013 SDK (Uygulama Geliştirme Aracı. SDK'yı [buradan](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx) indirebilir.)
4. Visual Studio 2017

ObjectARX SDK, AutoCAD .NET eklentisi oluşturmakta kullanılan Visual Studio şablonlarını (AutoCAD.NET Uygulama Sihirbazını) içermekle birlikte *Uygulama Sihirbazını* kullanmaksızın AutoCAD .NET eklentisi oluşturulacaktır.

### Visual Studio Sınıf Kütüphanesinin Oluşturulması

**File → Add → New Project** seçilerek yeni bir sınıf kütüphanesi oluşturulur. Seçilen Framework ile AutoCAD sürümü uyumlu olmalıdır. Örneğin AutoCAD 2013 için .Net Framework 4'ü kullanılmalıdır. (Bkz. Şekil-1)
![Şekil-1](https://eykaraduman.github.io/assets/images/add-new-project.png "Şekil-1"){:width="800"}

<sub>Şekil-1: Yeni bir AutoCAD .Net projesi oluşturulması</sub>

### AutoCAD .NET Başvuru Dosyalarının Eklenmesi

ObjectARX SDK'nın kurulu olduğu dizinde bulunan aşağıdaki üç dosya projeye referans olarak eklenir.

- `AcCoreMgd.dll` (AutoCAD çekirdek motoru için sınıflar içerir)
- `AcDbMgd.dll` (ObjectARX AcDb ve ilişkili sınıflarını içerir) 
- `AcMgd.dll` (AutoCAD uygulama sınıflarını içerir)

Visual Studio Properties penceresini kullanarak bu üç dosyanın **Copy Local** özelliği **False** olarak ayarlanır. Böylece eklentinin derleneceği dizine kopyalanmaları engellenmiş olacaktır. (Bkz. Şekil-2)

![Şekil-2](https://eykaraduman.github.io/assets/images/copy-local-false.png "Şekil-2")
	

<sub>Şekil-2: AutoCAD .Net referans dosyalarının eklnemesi</sub>

### Eklenti Sınıfının Oluşturulması (Plugin.cs)

Sınıf kütüphanesini, bir AutoCAD.NET eklentisine dönüştürmek için,  `Autodesk.AutoCAD.Runtime` isim uzayında bulunan `IExtensionApplication` sınıfından türettiğimiz **Plugin.cs** sınıfı projeye eklenir.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.Runtime;

[assembly: ExtensionApplication(typeof(PgAutoCAD.Plugin))]

namespace PgAutoCAD
{
    public class Plugin: IExtensionApplication
    {
        public void Initialize()
        {
            Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
            ed.WriteMessage("\nPgAutoCAD yüklendi...");
        }

        public void Terminate()
        {
            
        }
    }
}
```

`[assembly: ExtensionApplication(typeof(PgAutoCAD.Plugin))]` satırı, uygulamanın giriş sınıfının **Plugin.cs** olduğunu ifade etmektedir.

`Initialize()` yordamı eklenti ilk yüklendiğinde ve `Terminate()` yordamı ise eklenti sonlandırılırken yapılacaklar için kullanılmaktadır.

### Komut Sınıfının Oluşturulması (Commands.cs)

Eklentiye ait AutoCAD komutlarını oluşturabilmek için gerekli olan **Commands.cs** sınıfı projeye eklenmelidir.

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

using Autodesk.AutoCAD.ApplicationServices;
using Autodesk.AutoCAD.EditorInput;
using Autodesk.AutoCAD.Runtime;

[assembly: CommandClass(typeof(PgAutoCAD.Commands))]

namespace PgAutoCAD
{
    public class Commands
    {
        [CommandMethod("KomutGrubu", "IlkKomutum", "IlkKomutYerel", CommandFlags.Modal)]
        public void IlkKomut() 
        {
            Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
            ed.WriteMessage("\nİlk komut oluşturuldu.");
        }
    }
}
```

Komutlar, `public void IlkKomut()` 'da olduğu gibi, `CommandMethod`etiketine bağlı birer yordam olarak tanımlanmaktadır. AutoCAD komut satırına **IlkKomutum** yazıldığında `IlkKomut()` yordamı çalışacaktır.

Artık Visual Studio 2017 ortamında, **Build &rarr; Build PgAutoCAD** sekmesini seçerek derlediğimiz sonuç dll’yi (**..\Release\PgAutoCAD.dll** ya da **..\Debug\PgAutoCAD.dll**) yüklemek için AutoCAD **Netload** komutu kullanılabilir.

### Hata Ayıklama
AutoCAD .NET projelerinde Visual Studio ile hata ayıklayabilmek için yapılması gereken birkaç basit ayar bulunmaktadır. 

Öncelikle, proje kök klasörü altında **start** adlı bir .scr (script) dosyası aşağıdaki içerikle oluşturularak projeye eklenir. Dosya özelliklerinden **Copy to Output Directory** seçeneği **Copy always** olarak değiştirilmelidir. Böylece derleme sırasında **start.scr** dosyası *Debug* klasörüne kopyalanmayacaktır.

> netload PgAutoCAD.dll

Daha sonra, Visual Studio Project menüsünden proje özellikleri seçilir. (**PgAutoCAD Properties..**) Debug sekmesinde, **Start extarnal program** ve **Command line arguments** seçeneklerini Şekil-3'deki gibi doldurup proje ayarları kaydedilir. **/nologo** anahtarı, açılışta AutoCAD logosunu gizleyerek AutoCAD’in daha hızlı açılmasını sağlayacaktır. **/b “start.scr”** ise AutoCAD açıldıktan sonra **start.scr** script dosyasını çalıştırarak eklentiyi yükleyecektir.

![Şekil-3](https://eykaraduman.github.io/assets/images/debug-properties.png "Şekil-3")
	

<sub>Şekil-3: Proje özellikleri</sub>

