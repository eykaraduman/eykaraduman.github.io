---
classes: wide
title: Trapez Kanallarda Su Yüksekliğinin Python ile Çözümü
excerpt: "Trapez kanallarda Manning eşitliği için su derinliğinin SciPy `root` fonksiyonu kullanılarak çözümü."
permalink: /hidrolik/trapezoidal-section-hydraulics/
published: true
categories:
  - Python
  - Hidrolik
tags:
  - SciPy
  - Trapez Kesit
---

Trapez kanallarda Manning eşitliği, $A$ kesit alanı, $R$ hidrolik yarıçap, $n$ manning katsayısı $S_o$ kanal taban eğimi ve $Q$ debi olmak üzere aşağıdaki gibidir. 

$$
Q = \dfrac{1}{n} \cdot A \cdot R^{2/3} \cdot S_{o}^{1/2}
$$

Python SciPy kütüphanesi ait `root` fonksiyonu manning eşitliğini çözmek için kullanılmıştır. Su yüksekliğini bulunabilmesi için başlangıç tahmini olarak ise kritik derinlik seçilmiştir. 

$$
\dfrac{Q^{2} \cdot T} {g \cdot A^{3}} =1
$$

<script src="https://gist.github.com/eykaraduman/0bdbf426cf7065b691f366aaac783436.js"></script>
