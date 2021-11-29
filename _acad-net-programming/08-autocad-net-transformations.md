---
title: Dönüşümler
excerpt: "AutoCAD.Net API ile dönüşüm (ölçekleme, döndürme, yer değiştirme)."
permalink: /autocad-net-programming/transformations/
classes: wide
toc: true
published: true
sidebar:
  title: "AutoCAD .NET API ile Programlama"
  nav: autocadnet-programming-tutorial
---
3 boyutlu uzayda noktaları/nesneleri taşımak, ölçeklemek ve döndürmek için 4x4 boyutunda dönüşüm matrisi kullanılır. Bu matrisler CAD sistemlerinde önemli bir yer tutar. Dönüşüm matrisinin ilk 3 kolonu ölçekleme ve döndürme, 4. kolonu ise yer değiştirme/taşıma ile ilgilidir. Aşağıda temel dönüşüm matrislerininin nasıl oluşturulabileceği gösterilmiştir.

### Birim dönüşüm matrisi

$$
\begin{bmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0\\ 0 & 0 & 1 & 0\\ 0 & 0 & 0 & 1 \end{bmatrix}
$$

### Döndürme matrisleri

#### x ekseni etrafında döndürme

$$
\begin{bmatrix}
 1 & 0 &  0 & 0\\
 0 &  cos\theta & −sin\theta & 0\\ 
 0 &  sin\theta &  cos\theta & 0\\ 
 0 & 0&   0 & 1
\end{bmatrix}
$$

#### y ekseni etrafında döndürme

$$
\begin{bmatrix}
 cos\theta & 0 &  sin\theta & 0\\
 0 &  1 & 0 & 0\\
 −sin\theta &  0 &  cos\theta & 0\\
 0 & 0 &  0 & 1
\end{bmatrix}
$$

#### z ekseni etrafında döndürme

$$
\begin{bmatrix}
 cos\theta &  −sin\theta &  0& 0\\
 sin\theta &  cos\theta &  0& 0\\
 0& 0&  1& 0\\
 0&  0&  0& 1
\end{bmatrix}
$$

### Ölçekleme matrisi

$$
\begin{bmatrix}
 S_{x}&  0&  0& 0\\
 0&  S_{y}&  0& 0\\
 0& 0&  S_{z}& 0\\
 0&  0&  0& 1
\end{bmatrix}
$$

### Yer değiştirme matrisi

$$
\begin{bmatrix} 
1 & 0 & 0 & t_{x}\\ 
0 & 1 & 0 & t_{y}\\ 
0 & 0 & 1 & t_{z}\\ 
0 & 0 & 0 & 1\end{bmatrix}
$$

### AutoCAD.Net API ile dönüşüm matrisleri

Dönüşüm matrisi, AutoCAD.Net API'ye ait `Autodesk.AutoCAD.Geometry`  isim uzayında bulunanan <code>Matrix3d</code> nesnesi ile oluşturulur.

```csharp
Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
ed.WriteMessage(Matrix3d.Identity.ToString());
```
kod parçacığı birim matris olan `((1,0,0,0),(0,1,0,0),(0,0,1,0),(0,0,0,1))` değerini komut satırına yazdıracaktır.

<code>(0, 0, 0)</code> noktasından <code>(3.0, 3.0, 2.0)</code> noktasına herhangi bir varlığı taşımak için <code>Matrix3d.Displacement</code> matrisini kullanabilir.

```csharp
ed.WriteMessage(Matrix3d.Displacement(new Vector3d(3.0, 3.0, 2.0)).ToString());
```

> Sonuç: `((1,0,0,3),(0,1,0,3),(0,0,1,2),(0,0,0,1))`

Bir AutoCAD nesnesini <code>(1.0, 1.0, 1.0)</code> noktasını merkez alıp <code>Z</code> ekseni etrafında <code>45°</code> döndürmek içinse kullanması gereken matris  <code>Matrix3d.Rotation</code>'dır. 

```csharp
ed.WriteMessage(Matrix3d.Rotation(Math.PI / 4, Vector3d.ZAxis, new Point3d(1.0, 1.0, 1.0)).ToString());
```

> Sonuç: `((0.707106781186547,-0.707106781186548,0,1),(0.707106781186548,0.707106781186547,0,-0.414213562373095),(0,0,1,0),(0,0,0,1))`

<code>(0.0, 0.0, 0.0)</code> noktasını merkez alıp <code>5</code> katsayısı ile ölçekleme yapmak içinse <code>Matrix3d.Scaling</code> matrisi aşağıdaki gibi kullanılabilir.

```csharp
ed.WriteMessage(Matrix3d.Scaling(5.0, Point3d.Origin).ToString());
```

> Sonuç: `((5,0,0,0),(0,5,0,0),(0,0,5,0),(0,0,0,1)`

Ayrıca <code>Matrix3d</code> sınıfı, <code>PreMultiplyBy</code>, <code>PostMultiplyBy</code> fonksiyonlarıyla birleştirilmiş dönüşüm matrisleri oluşturmaya imkan sağladığı gibi izdüşüm (<code>Projection</code>) ve aynalama (<code>Mirroring</code>) için dönüşüm matrisleri de içerir. 

#### <code>Matrix3d.Displacement</code> dönüşüm matrisinin kullanımı

AutoCAD nesnelerinin bir noktadan diğerine nasıl taşınacağını göstermek için genişlik ve yükseklik değerlerini parametre olarak kabul eden <code>CreateRectangle</code> yordamını oluşturalım öncelikle.

```csharp
public static ObjectId CreateRectangle(double width, double height)
{
    Database db = HostApplicationServices.WorkingDatabase;
    ObjectId id = ObjectId.Null;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;

    PromptPointOptions pPtOpts = 
        new PromptPointOptions("\nBaşlangıç noktasını seçin: ");
    PromptPointResult pPtRes = ed.GetPoint(pPtOpts);
    if (pPtRes.Status != PromptStatus.OK)
        return id;
    Point2d startPt = new Point2d(pPtRes.Value.X, pPtRes.Value.Y);

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        Polyline pl = new Polyline();
        pl.SetDatabaseDefaults();
        pl.AddVertexAt(0, startPt, 0.0, 0.0, 0.0);
        pl.AddVertexAt(1, startPt.Add(new Vector2d(width, 0.0)), 0.0, 0.0, 0.0);
        pl.AddVertexAt(2, startPt.Add(new Vector2d(width, height)), 0.0, 0.0, 0.0);
        pl.AddVertexAt(3, startPt.Add(new Vector2d(0.0, height)), 0.0, 0.0, 0.0);
        pl.Closed = true;
        pl.ColorIndex = 1;

        BlockTableRecord btRecord =
            (BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

        id = btRecord.AppendEntity(pl);
        tr.AddNewlyCreatedDBObject(pl, true);
        pl.TransformBy(ucs2wcs);
        tr.Commit();

        ed.WriteMessage(pl.StartPoint.ToString());
    }
    return id;
}
```

Yukardaki metotta dikkate edilmesi gereken iki önemli nokta var; ilki, <code>Editor</code> sınıfı <code>GetPoint</code> metodu ile elde edilen noktanın etkin kullanıcı tanımlı koordinat sisteminde (UCS'de) olması, ikincisi ise <code>Polyline</code>'ın, veritabanına eklendikten sonra UCS'den WCS'ye <code>TransformBy</code> metoduyla dönüşümünün yapılmış olmasıdır. Bu dönüşümün nedeni <code>Editor.GetPoint()</code> metoduyla elde edilen noktanın UCS'de tanımlı olması ve  AutoCAD nesnelerinin geometrik bilgilerinin WCS'de saklanmasıdır.

Aslında AutoCAD .Net API ile geliştirilen her uygulamayı UCS'de denemek önemlidir. Çünkü AutoCAD grafik nesnelerine erişirken, onları veritabanına eklerken ve komut satırından nokta bilgisi alırken, API'nin kullandığı koordinat sistemleri arasındaki dönüşümleri doğru yapmak gerekir. Ayrıca UCS'de doğru çalışan bir uygulama, WCS'nin UCS'nin özel bir hali olması nedeniyle, WCS'de de sorunsuz çalışacaktır. Kullanıcı koordinat sistemine <code>Editor</code> sınıfının <code>CurrentUserCoordinateSystem</code> özelliğiyle ulaşılabilir.

UCS'den WCS'ye ve WCS'den UCS'ye dönüşüm için aşağıdaki matrisler kullanılır.

```csharp
Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;
Matrix3d wcs2ucs = ed.CurrentUserCoordinateSystem.Inverse();
```

<code>MoveEntity</code> metodu, seçilen herhangi bir noktadan başlayarak, <code>5.0</code> birim genişlik ve yükseklikte çizilen bir dikdörtgenin nasıl taşınabileceğinin bir örneğini sergilemektedir.

```csharp
public static void MoveEntity()
{
    Database db = HostApplicationServices.WorkingDatabase;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;

    ObjectId id = CreateRectangle(5.0, 5.0);

    PromptPointOptions pPtOpts =
        new PromptPointOptions("\nNesnenin taşınacağı noktayı seçin: ");
    PromptPointResult pPtRes = ed.GetPoint(pPtOpts);
    if (pPtRes.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        if (id != ObjectId.Null)
        {
            Polyline ent = (Polyline)tr.GetObject(id, OpenMode.ForWrite);
            Vector3d displacementVector = pPtRes.Value.TransformBy(ucs2wcs) - ent.StartPoint;

            Matrix3d displacementMatrix = Matrix3d.Displacement(displacementVector);
            Entity displacedEnt = ent.GetTransformedCopy(displacementMatrix);

            displacedEnt.ColorIndex = 2;

            BlockTableRecord btRecord =
                (BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

            btRecord.AppendEntity(displacedEnt);
            tr.AddNewlyCreatedDBObject(displacedEnt, true);

            tr.Commit();
        }
    }
}
```

<code>MoveEntity</code> metodu, yerdeğiştirmenin daha rahat takip edilebilmesi için oluşturulan dikdörtgen yerine, onun dönüştürülmüş bir kopyasını taşımakta ve bu işlem için <code>Entity</code> sınıfının <code>GetTransformedCopy</code> yordamını kullanmaktadır. Ayrıca dikdörtgenin başlangıç noktası dünya koordinat sisteminde (WCS'de) olduğundan, yer değiştrime vektörü, seçilen taşıma noktası UCS'den WCS'ye dönüştürülerek kurulmuştur.



![Şekil-1](https://eykaraduman.github.io/assets/images/move-entity-ucs.png "Şekil-1"){:width="400"}

<sub>Şekil-1: `MoveEntity` metodunun etkin UCS'de çalıştırılmasıyla elde edilen sonuç.</sub>


#### <code>Matrix3d.Scaling</code> dönüşüm matrisinin kullanımı

AutoCAD nesnelerini ölçekleyebilmek için,

<ul><li>Öncelikle, ölçekleme değeri ve noktası kullanılarak <code>Matrix3d.Scaling</code> dönüşüm matrisi kurulmaldır.</li><li>Daha sonra ölçeklenecek nesneye, ölçekleme matrisini parametre olarak kabul eden <code>Entity</code> sınıfının <code>GetTransformedCopy</code> ya da <code>TransformBy</code> metodu uygulanmalıdır. </li></ul>

XY planında çizdirilen bir dikdörtgeni 3 katına çıkararak kopyalayan <code>ScaleEntity</code> metodu ise aşağıdaki gibi olacaktır

```csharp
public static void ScaleEntity()
{
    Database db = HostApplicationServices.WorkingDatabase;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;

    ObjectId id = CreateRectangle(5.0, 5.0);

    PromptPointOptions pPtOpts =
        new PromptPointOptions("\nÖlçekleme noktasını seçin: ");
    PromptPointResult pPtRes = ed.GetPoint(pPtOpts);
    if (pPtRes.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        if (id != ObjectId.Null)
        {
            Polyline ent = (Polyline)tr.GetObject(id, OpenMode.ForWrite);
     
            Matrix3d scaleMatrix = Matrix3d.Scaling(3.0, pPtRes.Value.TransformBy(ucs2wcs));
            Entity scaledEnt = ent.GetTransformedCopy(scaleMatrix);

            scaledEnt.ColorIndex = 2;

            BlockTableRecord btRecord =
                (BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

            btRecord.AppendEntity(scaledEnt);
            tr.AddNewlyCreatedDBObject(scaledEnt, true);

            tr.Commit();
        }
    }
}
```

![Şekil-2](https://eykaraduman.github.io/assets/images/scale-entity-ucs.png "Şekil-2"){:width="400"}

<sub>Şekil-2: <code>ScaleEntity</code> metodunun etkin UCS'de çalıştırılmasıyla elde edilen sonuç.</sub>

#### <code>Matrix3d.Rotation</code> dönüşüm matrisinin kullanımı

<ul><li>Öncelikle, nesnenin etrafında döndürüleceği eksen ve nokta kullanılarak <code>Matrix3d.Rotation</code> dönüşüm matrisi kurulmalıdır.</li><li>Daha sonra döndürülecek nesneye, döndürme matrisini parametre olarak kabul eden <code>Entity</code> sınıfının <code>GetTransformedCopy</code> ya da <code>TransformBy</code> metodu uygulanmalıdır.</li></ul>

XY planında çizdirilen bir dikdörtgeni Z ekseni etrafında 45° döndüren <code>RotateEntity</code> metodu ise aşağıdaki gibi olacaktır.

```csharp
public static void RotateEntity()
{
    Database db = HostApplicationServices.WorkingDatabase;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;

    ObjectId id = CreateRectangle(5.0, 5.0);

    PromptPointOptions pPtOpts =
        new PromptPointOptions("\nDöndürme noktasını seçin: ");
    PromptPointResult pPtRes = ed.GetPoint(pPtOpts);
    if (pPtRes.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        if (id != ObjectId.Null)
        {
            Polyline ent = (Polyline)tr.GetObject(id, OpenMode.ForWrite);

            Matrix3d rotationMatrix = Matrix3d.Rotation(Math.PI/4.0, Vector3d.ZAxis, pPtRes.Value.TransformBy(ucs2wcs));
            Entity rotatedEnt = ent.GetTransformedCopy(rotationMatrix);

            rotatedEnt.ColorIndex = 2;

            BlockTableRecord btRecord =
                (BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

            btRecord.AppendEntity(rotatedEnt);
            tr.AddNewlyCreatedDBObject(rotatedEnt, true);

            tr.Commit();
        }
    }
}
```

![Şekil-3](https://eykaraduman.github.io/assets/images/rotate-entity-wcs.png "Şekil-3"){:width="400"}

<sub>Şekil-3: <code>RotateEntity</code> metodunun WCS'de çalıştırılmasıyla elde edilen sonuç.</sub>

#### Birleştirilmiş dönüşüm matrisinin uygulanması

Yukarıda gerçekleştirilen taşıma, ölçekleme ve döndürme işlemlerini <code>Matrix3d</code> sınıfının <code>PreMultiplyBy</code> metodu ile bir kerede yapılabilir. Bunun bir örneğini aşağıdaki <code>MoveScaleRotateEntity</code> metoduyla gösterilmiştir.

```csharp
public static void MoveScaleRotateEntity()
{
    Database db = HostApplicationServices.WorkingDatabase;
    Editor ed = Application.DocumentManager.MdiActiveDocument.Editor;
    Matrix3d ucs2wcs = ed.CurrentUserCoordinateSystem;

    ObjectId id = CreateRectangle(5.0, 5.0);

    PromptPointOptions pPtOpts =
        new PromptPointOptions("\nBir nokta seçin: ");
    PromptPointResult pPtRes = ed.GetPoint(pPtOpts);
    if (pPtRes.Status != PromptStatus.OK)
        return;

    using (Transaction tr = db.TransactionManager.StartTransaction())
    {
        if (id != ObjectId.Null)
        {
            Polyline ent = (Polyline)tr.GetObject(id, OpenMode.ForWrite);
            Vector3d displacementVector = pPtRes.Value.TransformBy(ucs2wcs) - ent.StartPoint;

            Matrix3d displacementMatrix = Matrix3d.Displacement(displacementVector);
            Matrix3d scaleMatrix = Matrix3d.Scaling(3.0, pPtRes.Value.TransformBy(ucs2wcs));
            Matrix3d rotationMatrix = Matrix3d.Rotation(Math.PI / 4.0, Vector3d.ZAxis, 
                pPtRes.Value.TransformBy(ucs2wcs));

            Entity modEnt = 
                ent.GetTransformedCopy(displacementMatrix.PreMultiplyBy(scaleMatrix).PreMultiplyBy(rotationMatrix));

            modEnt.ColorIndex = 2;

            BlockTableRecord btRecord =
                (BlockTableRecord)tr.GetObject(db.CurrentSpaceId, OpenMode.ForWrite);

            btRecord.AppendEntity(modEnt);
            tr.AddNewlyCreatedDBObject(modEnt, true);

            tr.Commit();
        }
    }
}
```

