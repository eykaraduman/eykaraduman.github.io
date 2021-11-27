---
title: "Koordinat Aktarım AutoCAD Eklentisi"
permalink: /import-coordinate/
author_profile: true
---
![Şekil-1](/assets/images/imp-xyz.png)

**Koordinat Aktarım**, metin dosyalarından koordinat bilgilerini aktaran basit bir AutoCAD eklentisidir. Aşağıdaki okuma şablonlarını desteklemektedir ve AutoCAD'in 2013-2022 arası sürümleri için derlenmiştir. 

Kurulum dosyası indirme bağlantısını aşağıda bulabilirsiniz.

{% capture notice-1 %}
**Desteklenen okuma şablonları:**

* X Y ve Y X
* X Y Z ve Y X Z
* ID X Y Z ve ID Y X Z,
* X Y Z Red Green Blue ve Y X Z Red Green Blue
* X Y R (Yarıçap) ve Y X R (Yarıçap)
* X Y Z R (Yarıçap) ve Y X Z R (Yarıçap)
{% endcapture %}

<div class="notice">
{{ notice-1 | markdownify }}
</div>
Aktarılan koordinat listeleri, çizime Point, Polyline, Polyline3d, Circle, Circle Block ya da Line olarak eklenebilmekte ve destelenen şablonlara göre AutoCAD Tablo nesneleri de oluşturulabilmektedir.

{% capture notice-2 %}

**Desteklenen tablo şablonları:**

* X Y ve Y X
* X Y Z ve Y X Z
* ID X Y ve ID Y X 
* ID X Y Z ve ID Y X Z,
{% endcapture %}

<div class="notice">
{{ notice-2 | markdownify }}
</div>
**Programın Özellikleri**

- Dosyadan koordinat bilgisi aktarma ve çizdirme
- Nokta, Daire ve Blok çizim nesnelerinden koordinat listesi oluşturma ve dışarı aktarma
- Koordinat bilgilerinden tablo oluşturma
- Dosyadan aktarımda koordinat değerleri için ayraç belirleme
- RGB ile nokta yerleştirme (Sadece daire şekli için)
- Otomatik kimlik numarası ekleme (Sadece daire ve daire blok için)
- Kimlik numarası için metin hizalama seçenekleri
- Çizim şekil ve kimlik numarası metni için katman seçimi
- Kimlik numarası metni, daire ve daire blok için ölçek seçeneği
- Kimlik numarasını önekle oluşturabilme
- Nokta aktarım ve tablo satır değerleri için ondalık hassasiyet

**Dosyadan Koordinat Bilgisi Aktarım**

- Komut satırına *ImpXYZ* yazın.
- Listenize uygun bir dosya okuma şablonu belirleyin.
- Koordinat dosyasını seçerek içeri aktarın.
- Daha sonra *Koordinat Yerleştir* tuşuna basarak noktaları seçmiş olduğunuz ayarlara göre çizdirin.
- *Tablo Oluştur* tuşunu basarak aktardığınız koordinatlardan tablo oluşturun.

**Dışarıya Koordinat Bilgisi Aktarma**

- *Nesne Seç* tuşunu kullanarak Nokta, Daire ve Blok çizim nesneleri koordinat listesini oluşturun.
- *Dosyaya Aktar* tuşuna basarak belirlediğiniz yazma şablonuna göre koordinatları dosyaya yazdırın.

**İndirme**

[Kurulum Dosyasını İndir](https://eykaraduman.github.io/contact/){: .btn .btn--info}

