---
title: "Autocad.NET Eklentisi Oluşturma"
permalink: /autocadnet/beginnertutorial/create-project-from-stratch/
classes: wide
excerpt: ""
toc: true
---

C#, F#, Visual Basic .Net vb. programlama dillerinden birini biliyorsanız AutoCAD.Net (Managed ObjectArx Wrapper) ile programlamaya giriş yapabilirsiniz. Benim tercihim .Net platformunun en popüler dillerinden biri olan C#’dan yana.

Autocad için .Net uygulaması yazmaya başlamadan önce aşağıda gereksinimleri edinip bilgisayarınıza kurmalısınız. (AutoCAD ve Visual Studio sürümleri sisteminize göre aşağıdakilersen farklılık gösterebilir. Ancak ObjectARX SDK'nın, AutoCAD sürümünüz ile uyumlu olmasına dikkat etmelisiniz.)

1. AutoCAD 2013 (Yazdığınız kodu denemek için gerekli)
2. ObjectARX 2013 SDK (Uygulama Geliştirme Aracı). (SDK'yı [buradan](https://www.autodesk.com/developer-network/platform-technologies/autocad/objectarx) indirebilirsiniz.)
4. Visual Studio 2017

ObjectARX SDK, AutoCAD.Net eklentisi oluşturabilmek için kullanabilecek olan Visual Studio şablonlarını (AutoCAD.Net  Uygulama Sihirbazı) içermektedir. Bu sihirbazı SDK içinde bulabilirsiniz. 

Biz *Uygulama Sihirbazını* kullanmaksızın, en baştan AutoCAD.Net eklentisini kendimiz oluşturacağız. 

Visual Studio 2017 ile yeni bir AutoCAD.Net projesi oluşturmak için sırasıyla aşağıdaki adımları izleyin:

- *File **→** Add **→** New Project* seçilerek açılan pencereden bir sınıf kütüphanesi oluşturun. Seçilen Framework'un AutoCAD sürümüyle uyumlu olmasına dikkat edin. Örneğin AutoCAD 2013 için .Net Framework 4'ü kullanmalısınız. (Bkz. Şekil-1)
	
	![Şekil-1](https://eykaraduman.github.io/assets/images/add-new-project.png "Şekil-1"){:width="800"}
	
	<sub>Şekil-1: Yeni bir AutoCAD.Net projesi oluşturulması</sub>
	
- ObjectARX SDK'nın kurulu olduğu dizinde bulunan aşağıdaki dosyaları projenize referans olarak ekleyin:
  - `AcCoreMgd.dll` (AutoCAD çekirdek motoru için sınıflar)
  - `AcDbMgd.dll` (ObjectArx AcDb ve ilişkili sınıflarını içerir) 
  - `AcMgd.dll` (AutoCAD uygulama sınıflarını içerir)

- Properties penceresini kullanarak bu üç dosyanın `Copy Local` özelliğini `False` olarak ayarlayın. Böylece derleme esnasında kopyalanmaları engellenecektir. 
- 
