---
title: "Koordinat Aktarım AutoCAD Eklentisi"
permalink: /import-coordinate/
author_profile: true
---
**Koordinat Aktarım**, metin dosyalarından koordinat bilgilerini aktaran bir AutoCAD eklentisidir. Aşağıdaki okuma şablonlarını desteklemektedir.


{% capture notice-1 %}
* X Y ve X Y
* X Y Z ve Y X Z
* ID X Y Z ve ID Y X Z,
* X Y Z Red Green Blue ve Y X Z Red Green Blue
* X Y R (Yarıçap) ve Y X R (Yarıçap)
* X Y Z R (Yarıçap) ve Y X Z R (Yarıçap)
{% endcapture %}

<div class="notice">
{{ notice-1 | markdownify }}
</div>
Aktarılan koordinat listeleri, çizime Point, Polyline, Polyline3d, Circle, Circle Blok ya da Line olarak eklenebilmekte ve destelenen şablonlara göre AutoCAD Tablo nesneleri de oluşturulabilmektedir.

{% capture notice-2 %}

**Desteklenen tablo şablonları:**

* X Y ve X Y
* X Y Z ve Y X Z
* ID X Y ve ID Y X 
* ID X Y Z ve ID Y X Z,
{% endcapture %}

<div class="notice">
{{ notice-2 | markdownify }}
</div>
Programın diğer özellikleri:

- Koordinat değerleri için istenilen ayraca göre aktarım
- RGB ile nokta yerleştirme (Sadece daire şekli için)
- Otomatik kimlik numarası ekleme (Sadece daire ve daire blok için)
- Kimlik numarası için metin hizalama seçenekleri
- Şekil ve kimlik numarası metni için katman seçimi
- Kimlik numarası metni, daire ve daire blok için ölçek seçeneği
- Kimlik numarasını önekle oluşturabilme
- Nokta aktarım ve tablo satır değerleri için hassasiyet
- Ayarların kaydedilmesi
  


[İletişim](https://eykaraduman.github.io/contact/){: .btn .btn--info}

