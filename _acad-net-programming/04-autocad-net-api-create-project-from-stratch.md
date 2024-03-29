---
title: "AutoCAD .NET Eklentisi Oluşturma"
excerpt: "Eklentinin Uygulama Sihirbazı kullanılmaksızın oluşturulması."
permalink: /autocad-net-programming/create-project-from-stratch/
toc: true
classes: wide
comments: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---

AutoCAD için .Net uygulaması geliştirmeye başlamadan önce aşağıdaki gereksinimler kurulmalıdır. AutoCAD ve Visual Studio sürümleri aşağıda verilenlere göre farklılık gösterebilir. Ancak kullanılacak ObjectARX SDK, AutoCAD sürümü ile mutlaka uyumlu olmalıdır.

1. AutoCAD 2013 (Deneme sürümü [buradan](https://www.autodesk.com/products/autocad/free-trial) indirilebilir.)
2. [ObjectARX 2013 SDK](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx) (Uygulama Geliştirme Aracı)
4. [Visual Studio 2017](https://visualstudio.microsoft.com/tr/vs/community/) (Ücretsiz Community sürümü yeterli olacaktır.)

ObjectARX SDK, AutoCAD .NET eklentisi oluşturmakta kullanılan Visual Studio şablonlarını (AutoCAD.NET
Uygulama Sihirbazını) içermekle birlikte, bu bölümde *Uygulama Sihirbazını* kullanmaksızın AutoCAD .NET
eklentisi oluşturulacaktır.

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
	

<sub>Şekil-2: AutoCAD .Net referans dosyalarının eklenmesi</sub>

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

Eklentiye ait AutoCAD komutlarını oluşturabilmek için gerekli olan **Commands.cs** sınıfı projeye eklenir.

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

`[assembly: CommandClass(typeof(PgAutoCAD.Commands))]` satırı, uyguylamanın komut sınıfının **Commands.cs**
olduğunu ifade etmektedir ve başka bir bir sınıf içinde oluşturulan komutlar çalışmayacaktır. Eğer başka sınıﬂar
içinde de komut oluşturulmak isteniyorsa bu satır silinmelidir.

Komutlar, `public void IlkKomut()` 'da olduğu gibi, `CommandMethod` özniteliğine bağlı birer yordam olarak tanımlanmaktadır. AutoCAD komut satırına **IlkKomutum** yazıldığında `IlkKomut()` yordamı çalışacaktır.

### Eklentinin Derlenmesi

PgAutoCAD eklentisi, Visual Studio 2017 ortamında, **Build &rarr; Build PgAutoCAD** sekmesi seçilerek derlenir. Sonuç dll, "**..\Release**" ya da " **..\Debug**" dizinlerinde oluşturulur.

### Eklentinin Yüklenmesi

AutoCAD .NET eklentilerinin yüklenmesi AutoCAD **Netload** komutu yardımı ile yapılmaktadır. Komut satırına *netload* yazılarak "**..\Release\PgAutoCAD.dll**" ya da "**..\Debug\PgAutoCAD.dll**" sonuç dll dosyalarından biri seçilir ve eklenti yüklenir. Artık *Commands.cs* sınıfı içinde tanımlanan komutlar kullanılabilir.

### Hata Ayıklama
AutoCAD .NET projelerinde Visual Studio ile hata ayıklayabilmek için yapılması gereken birkaç basit ayar bulunmaktadır. 

Öncelikle, proje kök klasörü altında **start** adlı bir .scr (script) dosyası aşağıdaki içerikle oluşturularak projeye eklenir. Dosya özelliklerinden **Copy to Output Directory** seçeneği **Copy always** olarak değiştirilmelidir. Böylece derleme sırasında **start.scr** dosyası *Debug* klasörüne kopyalanmayacaktır.

> netload PgAutoCAD.dll

Daha sonra, Visual Studio Project menüsünden proje özellikleri seçilir. (**PgAutoCAD Properties..**) Debug sekmesinde, **Start extarnal program** ve **Command line arguments** seçenekleri Şekil-3'deki gibi doldurulup proje ayarları kaydedilir. **/nologo** anahtarı, açılışta AutoCAD logosunu gizleyerek AutoCAD’in daha hızlı açılmasını sağlayacaktır. **/b “start.scr”** ise AutoCAD açıldıktan sonra **start.scr** script dosyasını çalıştırarak eklentiyi yükleyecektir.

![Şekil-3](https://eykaraduman.github.io/assets/images/debug-properties.png "Şekil-3")
	

<sub>Şekil-3: Proje özellikleri</sub>

