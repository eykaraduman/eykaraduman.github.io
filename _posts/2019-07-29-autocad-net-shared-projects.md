---
title: Visual Studio 2017 ile AutoCAD'in Farklı Sürümleri İçin Uygulama Geliştirme
permalink: /acad/autocad-net-shared-project/
published: true
classes: wide
categories:
  - AutoCAD.NET
tags:
  - MFC
---

AutoCAD.Net için uygulama geliştirirken, uygulamamızın AutoCAD'in güncel sürümü ile birlikte geçmiş sürümlerini de desteklemesini mutlaka isteriz. Bu desteğin tek bir Visual Studio çözümü içerisinde, AutoCAD ve .Net Framework farklı sürümlerini hedefleyerek nasıl hazırlanabileceğini paylaşılmış (shared) bir projeyle göstermeye çalışacağım.

Paylaşılmış projelerin kullanılmasının nedeni, bu projede barındırılan uygulama kodu ile farklı sürümler için hazırlanan sınıf kütüphanelerinin bağımlılıklarının birbirinden ayrılmasına olanak tanımasıdır.

**PgAcadTools.AutoCAD** adından bir Paylaştırılmış Proje (Shared Project)  Şekil-1'deki gibi oluşturulabilir. Bu projenin hedeflediği .Net Framework sürümünün  bir önemi yok. Çünkü hedeflenen sürümü, sınıf kütüphanelerinin özellikler sayfasından seçerek belirleyeceğiz. (Şekil-2) AutoCAD sürümleri için seçmeniz gereken hedef .Net Framework sürümünlerine [buradan](https://eykaraduman.github.io/acad/netframework-runtime-for-autocad-net/) ulaşabilirsiniz.

<img src="{{ "/assets/images/Shared-Project-1.png" | absoluteurl }}"  alt="Şekil-1" style="width:75%">

<figure>
  <figcaption>Şekil-1</figcaption>
</figure>

<img src="{{ "/assets/images/Shared-Project-2.png" | absolute_url }}"  alt="Şekil-2" style="width:75%">

<figure>
  <figcaption>Şekil-2</figcaption>
</figure>
