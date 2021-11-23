---
title: Sembol Tabloları
permalink: /acad/autocad-net-symbol-tables/
published: true
classes: wide
categories:
  - AutoCAD.Net
---
AutoCAD çizim dosyası, görünür (grafik) ve görünmez (grafik olmayan) nesnelere ait tablo ve kayıtları içeren bir veritabanı dosyasıdır ve aşağıdaki yerleşik (built-in) sembol tablo kayıtlarını mutlaka içerir:

- **“0”** adlı bir katman,
- **“Standart”** adlı metin, ölçülendirme ve tablo stilleri
- **CAD_GROUP, ACAD_MLINESTYLE** vb. sözlük kayıtlarını içeren Adlandırılmış Nesne Sözlüğü (NOD: Named Object Dictionary)
- <strong>*MODEL_SPACE, *PAPER_SPACE</strong> ve <strong>*paper_space0</strong> kayıtlarını içeren blok tablosu
- **CONTINUOS**, **BY_LAYER** ve **BY_BLOCK** kayıtlarını içeren çizgi tipi tablosu
- **ACAD** kaydını içeren kayıtlı uygulamalar tablosu.

AutoCAD, bu nesnelerin sembol tablo kayıtlarını oluştururken, önce her nesnenin ait olduğu sembol tablosunu açar, yeni nesneyi 
tabloya kaydeder ve daha sonra sırasıyla tabloyu ve kayıt için açtığı çizim veritabanını kapatır.

AutoCAD çizim dosyası birçok sembol tablosu içerir. Ancak bu sembol tablolarına yeni bir tane eklemek mümkün değildir; 
sadece yeni kayıtlar eklenebilir ya da var olan kayıtlar silinebilir.

Her AutoCAD.NET programcısı, yazdığı uygulamaya özgü birtakım verileri (ayarlar gibi) çizim veritabanında saklama ihtiyacını hisseder. 
Bu ihtiyacı ise AutoCAD.NET API’sinin NOD nesnesi karşılar. Sembol tabloları ve NOD arasındaki en temel fark, sembol tablolarının sadece kendi türlerine ait kayıtları saklayabilmesi (örneğin Metin Stil Tablosu sadece Metin Stil Kayıtlarını içerebilir), NOD’unsa DBObject nesnesinden türemiş her nesneyi barındırabilmesidir.

AutoCAD sembol tablolarının bazılarının adlarını verip işlevlerini özetleyerek devam edelim.

| Tablo Adı | Tablo İçin Açıklama |
|---|---|
| BlockTable  | Çizim veritabanındaki blok tanımlarını temsil eden kayıtların (BlockTableRecords) tablosu. Model Uzayı (model space) ve Kağıt Uzayı (paper space) adlı iki önemli kayıt içerir. Tüm görünür (grafik) nesneler bu tabloya aittir. |
| DimStyleTable | Çizim veritabanındaki ölçülendirme stillerini temsil eden kayıtların (DimStyleTableRecords) tablosu. |
| LayerTable | Katman kayıtlarının (LayerTableRecords) saklandığı tablo. |
| LinetypeTable | Çizim veritabanı çizgi tipi kayıtlarının (LinetypeTableRecords) tablosu. |
| RegAppTable | Kaydettirilmiş uygulama adlarını içeren kayıtların (RegAppTableRecords) tablosu. Bu tablodaki her kayıt, çizim veritabanı nesneleriyle ilişkilendirilmiş Genişletilmiş Nesne Verisini (Extended Entity Data) belirlemek için kullanılan uygulama kimliklerini (ID) temsil eder. |
| TextStyleTable | Çizim veritabanındaki ölçülendirme stillerini temsil eden kayıtların (TextStyleTableRecords) tablosu. |
| UCSTable | Çizim veritabanında saklanan kullanıcı tanımlı koordinat sistemlerini (UCSTableRecords) barındıran tablo. |
| ViewportTable | AutoCAD sistem değişkeni TILEMODE = 1 iken  viewportdüzenleme kayıtlarını (ViewPortTableRecords) temsil eden tablo. (AutoCAD ve DXF dosyaları için VPORT tablosu.) |
| ViewTable | Çizim veritabanı ile birlikte saklanan görünüş (view) kayıtlarını (ViewTableRecords) temsil eden tablo.( AutoCAD ve DXF dosyaları için VIEW tablosu.) |

Örneklere geçmeden önce çizim veritabanında saklı NOD ve sembol tablo kayıt anahtarlarına nasıl erişilebileceğine bir bakalım. 
NOD kayıtları ile başlıyoruz:

```c#
 public static void NODKayitListele()  
 {  
 // Çizim veritabanına erişim  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
 // Komut editörüne erişim  
      Editor ed = doc.Editor;  
      ed.WriteMessage("\nAdlandırılmış Nesne Sözlüğü (NOD) Kayıtları:");  
      ed.WriteMessage("\n--------------------------------------------");  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {  
           // Veritabanı sözlüğünün (NOD'un) okuma amaçlı açılması  
           DBDictionary NOD = (DBDictionary)tr.GetObject(db.NamedObjectsDictionaryId,  
                OpenMode.ForRead);  
           // Sözlük içerisinde dolaşmak için  
           // veritabanı sözlük numaralandırıcısının eldesi  
           DbDictionaryEnumerator eNOD = NOD.GetEnumerator();  
           while (eNOD.MoveNext())  
           {  
                // Sözlük kayıt anahtarlarının AutoCAD  
                // komut satırına yazdırılması  
                ed.WriteMessage("\n{0}", eNOD.Current.Key);  
           }  
           // Numaralandırıcının sonlandırılması  
           eNOD.Dispose();  
      }  
 }  
 [CommandMethod("NODListe")]  
 static public void NODKayitlar()  
 {  
      NODKayitListele();  
 }
```

![Şekil-1](/assets/images/SembolTablo1.jpg)

Şimdiyse en önemli sembol tablolarından biri olan blok tablosu kayıtlarını araştıralım:

```c#
 public static void BlokTabloKayitListele()  
 {  
      Document doc = Application.DocumentManager.MdiActiveDocument;  
      Database db = doc.Database;  
      Editor ed = doc.Editor;  
      string BlockTableKayitListe = "Blok Tablosu Kayıt Listesi :";  
      using (Transaction tr = db.TransactionManager.StartTransaction())  
      {  
           // Block tablosunun okuma amaçlı açılması  
           BlockTable blockTable = (BlockTable)tr.GetObject(db.BlockTableId,  
                OpenMode.ForRead);  
           // Blok tablosu içerisinde dolaşmak için  
           // sembol tablo numaralandırıcısının eldesi  
           SymbolTableEnumerator eBlockTable = blockTable.GetEnumerator();  
           while (eBlockTable.MoveNext())  
           {  
                // Sembol tablo kaydının eldesi  
                SymbolTableRecord blockTableRec =  
                     (SymbolTableRecord)tr.GetObject(eBlockTable.Current,OpenMode.ForRead);  
                BlockTableKayitListe += "\n" + blockTableRec.Name;  
           }  
           // Numaralandırıcının sonlandırılması  
           eBlockTable.Dispose();  
           Application.ShowAlertDialog(BlockTableKayitListe);  
      }  
 }  
 [CommandMethod("BlkTblListe")]  
 static public void BlokTabloKayitlar()  
 {  
      BlokTabloKayitListele();  
 } 
```

**BlkTblListe** AutoCAD komutunun sonucuysa aşağıdaki mesaj diyaloğu olacaktır:

![Şekil-2](/assets/images/SembolTablo2.jpg)

Uygun sembol tablo kimliklerini kullanarak diğer tabloların içerdikleri kayıtlara da kolayca ulaşabiliriz. 
Bunun için **BlokTabloKayitListele()** fonksiyonundaki, 

```c#
BlockTable blockTable =  
        (BlockTable)tr.GetObject(db.BlockTableId,OpenMode.ForRead); 
```

satırı yerine aşağıdakilerden birini koymamız yeterli olacaktır: 

```
DimStyleTable symTable =  
      (DimStyleTable)tr.GetObject(db.DimStyleTableId, OpenMode.ForRead);  
 LayerTable symTable =  
      (LayerTable)tr.GetObject(db.LayerTableId, OpenMode.ForRead);  
 LinetypeTable symTable =  
      (LinetypeTable)tr.GetObject(db.LinetypeTableId, OpenMode.ForRead);  
 RegAppTable symTable =  
      (RegAppTable)tr.GetObject(db.RegAppTableId, OpenMode.ForRead);  
 TextStyleTable symTable =  
      (TextStyleTable)tr.GetObject(db.TextStyleTableId, OpenMode.ForRead);  
 UcsTable symTable =  
      (UcsTable)tr.GetObject(db.UcsTableId, OpenMode.ForRead);  
 ViewportTable symTable =  
      (ViewportTable)tr.GetObject(db.ViewportTableId, OpenMode.ForRead);  
 ViewTable symTable =  
      (ViewTable)tr.GetObject(db.ViewTableId, OpenMode.ForRead);
```

**Transaction** nesnesinin, sembol tablo kimliğini ve tabloya erişim türünü parametre olarak kabul eden **GetObject(…)** metodunu farklı tablolar için tekrarlamaktan başka bir şey yapmadık.
Artık sembol tablo kayıtlarının nasıl ekleneceğini gösteren örneklere geçebiliriz. Ama önce, bu tablolarla çalışmamızı kolaylaştıracak birkaç yardımcı fonksiyon yazmak yerinde olacaktır.
Bu fonksiyonlardan ilki, üzerinde çalışacağımız tablonun veritabanı nesne kimliğini (ObjectId), tablo kayıt türüyle elde etmemizi sağlayacak **SembolTabloNesneKimlikAl(…)** statik fonksiyonu:

```c#
public static ObjectId SembolTabloNesneKimlikAl(Database db, Type SinifTip)  
 {  
      if (SinifTip == typeof(BlockTableRecord))  
           return db.BlockTableId;  
      if (SinifTip == typeof(DimStyleTableRecord))  
           return db.DimStyleTableId;  
      if (SinifTip == typeof(LayerTableRecord))  
           return db.LayerTableId;  
      if (SinifTip == typeof(LinetypeTableRecord))  
           return db.LinetypeTableId;  
      if (SinifTip == typeof(TextStyleTableRecord))  
           return db.TextStyleTableId;  
      if (SinifTip == typeof(RegAppTableRecord))  
           return db.RegAppTableId;  
      if (SinifTip == typeof(UcsTableRecord))  
           return db.UcsTableId;  
      if (SinifTip == typeof(ViewTableRecord))  
           return db.ViewTableId;  
      if (SinifTip == typeof(ViewportTableRecord))  
           return db.ViewportTableId;  
      return ObjectId.Null;  
 } 
```

İkincisi, tablo kaydının veritabanı nesne kimliğini, kayıt türü ve adıyla elde etmemizi sağlayacak **SembolTabloKayitNesneKimlikAl(…)** 
fonksiyonu: 

```c#
public static ObjectId SembolTabloKayitNesneKimlikAl(Database db, Type SinifTip,  
       string SymAd)  
 {  
      ObjectId symbolTableId = SembolTabloNesneKimlikAl(db, SinifTip);  
      ObjectId recId = ObjectId.Null;  
      using (Transaction transaction = db.TransactionManager.StartTransaction())  
      {  
           SymbolTable table = (SymbolTable)transaction.GetObject(symbolTableId,  
                OpenMode.ForRead);  
           if (table.Has(SymAd))  
           {  
                recId = table[SymAd];  
           }  
      }  
      return recId;  
 } 
```

Üçüncüsü, adını bildiğimiz bir tablo kaydının sembol tablosunda var olup olmadığını sorgulamakta kullanacağımız 
**SembolTabloKaydiMevcutMu(…)** fonksiyonu:

```c#
 public static bool SembolTabloKaydiMevcutMu(Database db, Type SinifTip, string SymAd)  
 {  
      bool mevcut = false;  
      ObjectId symbolTableId = SembolTabloNesneKimlikAl(db, SinifTip);  
      using (Transaction transaction = db.TransactionManager.StartTransaction())  
      {  
           mevcut = ((SymbolTable)transaction.GetObject(symbolTableId,  
                OpenMode.ForRead)).Has(SymAd);  
      }  
      return mevcut;  
 } 
```

Dördüncüsü ve sonuncusu ise herhangi bir çizgi tipini, verili çizgi tipi dosyasında arayan ve bulduğunda çizgi tipi nesne kimliğini 
döndüren **CizgiTipiKimlikAdAlYadaYükle(…)** fonksiyonu (katman kaydı oluştururken kullanacağız):

```c#
public static ObjectId CizgiTipiKimlikAlYadaYukle(string CizgiTipDosya, string CizgiTipAd)  
{  
   Document doc = Application.DocumentManager.MdiActiveDocument;  
   Database db = doc.Database;  
   ObjectId id = SembolTabloKayitNesneKimlikAl(db, typeof(LinetypeTableRecord),  
     CizgiTipAd);  
   if (!id.IsNull)  
     return id;  
   try  
   {  
     db.LoadLineTypeFile(CizgiTipAd, CizgiTipDosya);  
     id = SembolTabloKayitNesneKimlikAl(db, typeof(LinetypeTableRecord), CizgiTipAd);  
     if (!id.IsNull)  
       return id;  
     else  
       return db.ContinuousLinetype;  
   }  
   catch  
   {  
     return db.ContinuousLinetype;  
   }  
}  
```

Bu fonksiyonların yardımıyla artık ilk sembol tablo kaydımızı yaratabiliriz. Bir katman (layer) kaydı oluşturarak başlayalım. 
Aşağıda kodunu bulacağınız **KatmanOlustur(…)** fonksiyonu, içerdiği bilgi satırlarından da kolayca takip edebileceğiniz gibi; adı, 
çizgi tipi ve rengi verili bir katmanı çizim veritabanına ekleyecektir. 

```c#
public static ObjectId KatmanOlustur(Database db, string KatmanAdi,  
       string KatmanCizgiTipiAdi, short KatmanRengi)  
 {  
      ObjectId id = ObjectId.Null;  
      using (Transaction trans = db.TransactionManager.StartTransaction())  
      {  
           // Eklemek istediğimiz katmanının varlığını sorgulamak için  
           // katman tablosunu okuma amaçlı açalım.  
           LayerTable tbl = (LayerTable)trans.GetObject(db.LayerTableId,  
                OpenMode.ForRead);  
           // Eklemek isteğimiz katman mevcutsa  
           // kayıt kimliğini fonksiyon geri dönüş değerine atayalım.  
           if (tbl.Has(KatmanAdi))  
                id = tbl[KatmanAdi];  
           else  
           {  
                // Eklemek istediğimiz katmanın varolmadığına artık eminiz.  
                // Yeni katman tablo kaydını oluşturalım.  
                LayerTableRecord rec = new LayerTableRecord();  
                rec.Name = KatmanAdi;  
                rec.Color = Color.FromColorIndex(ColorMethod.ByAci, KatmanRengi);  
                ObjectId lineId = CizgiTipiKimlikAlYadaYukle(KatmanCizgiTipiAdi,  
                     "acadiso.lin");  
                if (lineId != ObjectId.Null) rec.LinetypeObjectId = lineId;  
                // Daha önce okuma amaçlı açtığımız katman tablosunu  
                // yeni kaydı eklemek için yazma amaçlı açalım.  
                tbl.UpgradeOpen();  
                //Katman tablosuna kaydı ekleyelim.  
                id = tbl.Add(rec);  
                // İşlem yığınının kaydı sahiplenmesini sağlayalım.  
                trans.AddNewlyCreatedDBObject(rec, true);  
                // İşlem yığınını onaylayalım.  
                trans.Commit();  
           }  
      }  
      return id;  
 }  
 [CommandMethod("KtmEk")]  
 static public void KatmanEkle()  
 {  
      KatmanOlustur(Application.DocumentManager.MdiActiveDocument.Database,  
           "AbcProgramlama", "HIDDEN", 1);  
 }  
```

**KtmEk** AutoCAD komutunu çalıştırdığınızda, “AbcProgramlama” katmanının çizim veritabanı katman listesine eklendiğini göreceksiniz. 
Katman oluşturmak için izlediğimiz yolu diğer tablo kayıtlarını oluşturmakta da izleyebiliriz. Bir metin stil kaydının (TextStyleTableRecord) çizim veritabanına nasıl ekleneceğini gösteren örnek fonksiyon aşağıda: 

```c#
public static ObjectId MetinBicemOlustur(Database db, string MetinDosyaAdi,  
       string MetinStilAd, double MetinYks, double GenislikFaktor)  
 {  
      ObjectId id = ObjectId.Null;  
      using (Transaction trans = db.TransactionManager.StartTransaction())  
      {  
           // Eklemek istediğimiz metin biçeminin varlığını  
           // sorgulamak için metin biçem tablosunu okuma amaçlı açalım.  
           TextStyleTable tbl = (TextStyleTable)trans.GetObject(db.TextStyleTableId,  
                OpenMode.ForRead);  
           // Eklemek isteğimiz metin biçemi mevcutsa  
           // kayıt kimliğini fonksiyon geri dönüş değerine atayalım.  
           if (tbl.Has(MetinStilAd))  
                id = tbl[MetinStilAd];  
           else  
           {  
                // Eklemek istediğimiz metin biçeminin varolmadığına artık eminiz.  
                // Yeni metin biçemi tablo kaydını oluşturalım.  
                TextStyleTableRecord rec = new TextStyleTableRecord();  
                rec.Name = MetinStilAd;  
                rec.FileName = MetinDosyaAdi;  
                rec.BigFontFileName = "";  
                rec.ObliquingAngle = 0.0;  
                rec.TextSize = MetinYks;  
                rec.XScale = GenislikFaktor;  
                // Daha önce okuma amaçlı açtığımız metin biçem tablosunu  
                // kaydı eklemek için yazma amaçlı açalım.  
                tbl.UpgradeOpen();  
                //Metin biçem tablosuna kaydı ekleyelim.  
                id = tbl.Add(rec);  
                // İşlem yığınının kaydı sahiplenmesini sağlayalım.  
                trans.AddNewlyCreatedDBObject(rec, true);  
                // İşlem yığınını onaylayalım.  
                trans.Commit();  
           }  
      }  
      return id;  
 }  
 [CommandMethod("TxtStyEk")]  
 static public void MetinBicemEkle()  
 {  
      MetinBicemOlustur(Application.DocumentManager.MdiActiveDocument.Database,  
           "Txt.shx", "Metin", 0.00, 1.00);  
 }  
```

Son olarak, AutoCAD içerisindeki tüm grafik nesneleri barındıran blok tablosuna yeni kayıtların nasıl ekleneceğinden bahsedeceğim. 
Yeni bir AutoCAD çizimi blok tablosu, daha önce de gördüğümüz gibi, *MODEL_SPACE, *PAPER_SPACE , *paper_space0 başlangıç kayıtlarını 
içerir. Çizime yeni bir layout eklediğimizde ise AutoCAD, *paper_space1 adlı yeni bir kaydı blok tablosuna ekleyecektir. 
Yukarda tanımladığımız **BlkTblListe** AutoCAD komutunu kullanarak bu durumu gözlemleyebilirsiniz. 
AutoCAD nesneleri, model uzayında yaratılabileceği gibi kağıt uzaylarından (layout’lar) herhangi birinde de yaratılabilir. 
Model uzayında çizim yapmak istediğimizde model uzayı, kağıt uzayında çizim yapmak istediğimizde ise kağıt uzayı blok tablo kaydını kullanacağız. AutoCAD blok nesnelerinin blok tablosuna eklenmesi ise diğer nesnelerinkinden biraz farklıdır; blok tablosunun blok tablo kaydı aracılığıyla yapılır. Bu farkı örnekleyen fonksiyonumuzun kodu ve AutoCAD komutu aşağıda: 

```c#
public static void YeniBlokTabloOlustur(string blkName)  
 {  
      // Çizim veritabanının eldesi  
      Database db = Application.DocumentManager.MdiActiveDocument.Database;  
      // İşlem yığınının başlatılması  
      using (Transaction transaction = db.TransactionManager.StartTransaction())  
      {  
           // Blok tablosunun yazma amaçlı eldesi  
           BlockTable blkTable =  
                (BlockTable)transaction.GetObject(db.BlockTableId, OpenMode.ForWrite);  
           // Blok tablosu eklemek istediğimiz blok adını içeriyor mu?  
           // (Var olan bir bloğun aynı adla tekrar oluşurulmaya çalışılması bir  
           // hataya neden olacağından kontrol edilmelidir.)  
           if (!blkTable.Has(blkName))  
           {  
                // Bloğa eklenecek çizginin oluşturulması  
                Line cizgi =  
                     new Line(new Point3d(0.0, 0.0, 0.0), new Point3d(0.0, 1.0, 0.0));  
                // Blok tablo kaydı örenğinin oluşturulması  
                BlockTableRecord blkRec = new BlockTableRecord();  
                // Blok isminin kayda atanması  
                blkRec.Name = blkName;  
                // Blok kaydına daha önce tanımlanan çizgi nesnesini eklenmesi  
                blkRec.AppendEntity(cizgi);  
                // Blok kaydının blok tablosuna eklenmesi  
                blkTable.Add(blkRec);  
                // Blok kaydının işlem yığını tarafından sahiplenilmesi  
                transaction.AddNewlyCreatedDBObject(blkRec, true);  
                // İşlem yığınının onaylanması  
                transaction.Commit();  
           }  
      }  
 }  
 [CommandMethod("BlkTblEkle")]  
 static public void BlokTabloEkle()  
 {  
      YeniBlokTabloOlustur("*CizgiBlok");  
 }  
```

Model ve kağıt uzaylarında çizim yapmak için izleyeceğimiz yolda yukarıdakine benzeyecektir:

- Önce çizim veritabanına, ardından da işlem yığın yöneticisine erişir
- İşlem yığınını başlatır
- Blok tablosunu okuma, nesneyi eklemek istediğiniz uzayı ise yazma amaçlı açar
- İstediğiniz nesneleri oluşturup uzaya ekler
- İşlem yığını yöneticisinin nesneleri sahiplenmesini sağlar
- Ve işlem yığınını onaylarız

Yukarda andığımız işleri yapan fonksiyon ise aşağıdaki gibi olacaktır:

![Şekil-3](/assets/images/SembolTablo3.jpg)

Tüm anlattıklarımı toparlayan ve kolayca sınayıp kontrol edebileceğiniz bir Windows formunu Visual Studio.Net projesinde bulabilirsiniz. 
AutoCAD komutu **SmbTblDia**.

### İndir
[Sembol Tabloları Visual Studio 2010 Projesi](http://eykaraduman.github.io/assets/downloads/AbcSymbolTables.zip)
