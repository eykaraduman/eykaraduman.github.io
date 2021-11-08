---
title: "Autocad.NET Eklentisi Oluşturma"
permalink: /autocadnet/beginnertutorial/create-project-from-stratch/
classes: wide
---

AutoCAD için .Net uygulaması geliştirmeye başlamadan önce aşağıda gereksinimleri edinip bilgisayarınıza kurmalısınız. AutoCAD ve Visual Studio sürümleri sisteminize bağlı olarak aşağıdakilere göre farklılık gösterebilir. Ancak ObjectARX SDK'nın, AutoCAD sürümünüz ile uyumlu olmasına dikkat etmelisiniz.

1. AutoCAD 2013
2. ObjectARX 2013 SDK (Uygulama Geliştirme Aracı. SDK'yı [buradan](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx) indirebilirsiniz.)
4. Visual Studio 2017

ObjectARX SDK, AutoCAD.NET eklentisi oluşturmakta kullanılan Visual Studio şablonlarını (AutoCAD.NET Uygulama Sihirbazını) içermektedir. Ancak bu yazıda **Uygulama Sihirbazını** kullanmaksızın, en baştan AutoCAD.NET eklentisini nasıl oluşturulacağını göreceğiz.

Visual Studio 2017 ile yeni bir AutoCAD.NET projesi oluşturmak için sırasıyla aşağıdaki adımlar izlenmelidir.

- **File → Add → New Project** seçilerek açılan pencereden bir sınıf kütüphanesi oluşturun. Seçilen Framework'un AutoCAD sürümüyle uyumlu olmasına dikkat edin. Örneğin AutoCAD 2013 için .Net Framework 4'ü kullanmalısınız. (Bkz. Şekil-1)
	![Şekil-1](https://eykaraduman.github.io/assets/images/add-new-project.png "Şekil-1"){:width="800"}
	
	<sub>Şekil-1: Yeni bir AutoCAD.Net projesi oluşturulması</sub>
	
- ObjectARX SDK'nın kurulu olduğu dizinde bulunan aşağıdaki dosyaları projenize referans olarak ekleyin:
  - `AcCoreMgd.dll` (AutoCAD çekirdek motoru için sınıflar)
  - `AcDbMgd.dll` (ObjectARX AcDb ve ilişkili sınıflarını içerir) 
  - `AcMgd.dll` (AutoCAD uygulama sınıflarını içerir)

- Properties penceresini kullanarak bu üç dosyanın **Copy Local** özelliğini **False** olarak ayarlayın. Böylece eklentinizin derleneceği dizine kopyalanmaları engellenmiş olacaktır. (Bkz. Şekil-2)

  ![Şekil-2](https://eykaraduman.github.io/assets/images/copy-local-false.png "Şekil-2")
  	

  <sub>Şekil-2: AutoCAD.Net referans dosyalarının eklnemesi</sub>

- Sınıf kütüphanemizi, bir AutoCAD.NET eklentisine dönüştürmek için,  `Autodesk.AutoCAD.Runtime` isim uzayında bulunan `IExtensionApplication` sınıfından türettiğimiz  **Plugin.cs** sınıfını projemize eklemeliyiz.

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

  `[assembly: ExtensionApplication(typeof(PgAutoCAD.Plugin))]` satırı uygulamamızın giriş sınıfının **Plugin.cs** olduğunu ifade etmektedir.

  `Initialize()` yordamı eklentimiz ilk yüklendiğinde ve `Terminate()` yordamı ise eklentimiz sonlandırılırken yapılacaklar için kullanılmaktadır.

- Komutları oluşturabilmek içinse aşağıdaki **Commands.cs** sınıfını projemize eklemeliyiz.

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

  Komutlar, `public void IlkKomut()` 'da olduğu gibi, `CommandMethod`etiketine bağlı birer yordam olarak tanımlanmaktadır. AutoCAD komut satırına **IlkKomutum** yazıldığında `IlkKomut()` yormamı çalışacaktır.
  
  Artık Visual Studio 2017 ortamında **Build &rarr; Build PgAutoCAD** sekmesini seçerek derlediğimiz sonuç dll’yi (**..\Release\PgAutoCAD.dll** ya da **..\Debug\PgAutoCAD.dll**) yüklemek için AutoCAD **Netload** komutu kullanabilirsiniz.

#### Hata Ayıklama
AutoCAD.NET projelerinde Visual Studio ile hata ayıklayabilmek için yapılması gereken birkaç basit ayar var.
Aşağıdaki adımları izleyerek projelerinizde kolaylıkla hata ayıklayabilirsiniz.

- Proje kök klasörü altında **start** adlı bir .scr (script) dosyası oluşturun. **netload PgAutoCAD.dll** satırını bu dosyaya yazarak **start.scr** dosyasını projenize ekleyin. Dosya özelliklerinden **Copy to Output Directory** seçeneğini **Copy always** olarak değiştirin. Böylece derleme sırasında **start.scr** dosyası *Debug* klasörüne kopyalanacaktır.

- Project menüsünden proje özelliklerini seçin. (**PgAutoCAD Properties..**) Debug sekmesinde, **Start extarnal program** ve **Command line arguments** seçeneklerini Şekil-3'deki gibi doldurup proje ayarlarını kaydedin. **/nologo** anahtarı, açılışta AutoCAD logosunu gizleyerek AutoCAD’in daha hızlı açılmasını sağlayacaktır. **/b “start.scr”** ise AutoCAD açıldıktan sonra **start.scr** script dosyasını çalıştırarak eklentiyi yükleyecektir.

  ![Şekil-3](https://eykaraduman.github.io/assets/images/debug-properties.png "Şekil-3")
  	

  <sub>Şekil-3</sub>


