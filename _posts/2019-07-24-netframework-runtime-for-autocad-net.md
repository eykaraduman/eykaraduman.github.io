---
title: AutoCAD.Net ve .Net Framework Runtime Uyumluluğu
permalink: /acad/netframework-runtime-for-autocad-net/
published: true
categories:
  - AutoCAD.Net
---

AutoCAD, ilk olarak 2005 yılında ObjectARX kütüphanelerinin .NET sürümlerini yayınlamıştır. AutoCAD.Net ile yazılan uygulamaların, kullanıcıların bilgisayarlarında çalışabilmesi için bu bilgisayarların barındırdıkları AutoCAD sürümüne göre derlenmesi gerekmektedir. Ayrıca AutoCAD sürümüne göre uygulamalarınızı derlemekte kullanmanız gereken Visual Studio sürümleri de değişmektedir. 

Aşağıdaki tabloda AutoCAD sürümlerini için hangi .NET Framework Runtime ve Visual Studio sürümlerini kullanmanız gerektiğini bulabilirsiniz.

| AutoCAD Harici Sürüm | AutoCAD İç Sürüm | .NET Framework Runtime | Visual Studio Sürüm |
| -------------------- | ---------------- | ---------------------- | ------------------- |
| 2005                 | 16.1             | 1.0                    | 2002                |
| 2006                 | 16.2             | 1.1                    | 2003                |
| 2007                 | 17.0             | 2.0                    | 2005/2008/2010/2012 |
| 2008                 | 17.1             | 2.0                    | 2005/2008/2010/2012 |
| 2009                 | 17.2             | 3.0                    | 2008/2010/2012      |
| 2010                 | 18.0             | 3.5                    | 2008/2010/2012      |
| 2011                 | 18.1             | 3.5                    | 2008/2010/2012      |
| 2012                 | 18.2             | 4.0                    | 2010/2012/2013      |
| 2013                 | 19.0             | 4.0                    | 2010/2012/2013      |
| 2014                 | 19.1             | 4.0                    | 2010/2012/2013      |
| 2015                 | 20.0             | 4.5                    | 2012/2013           |
| 2016                 | 20.1             | 4.5                    | 2012/2013/2015      |
| 2017                 | 21.0             | 4.6                    | 2015                |
| 2018                 | 22.0             | 4.6                    | 2015                |
| 2019                 | 23.0             | 4.7                    | 2017                |
| 2020                 | 23.1             | 4.7                    | 2017                |

Harici sürümü, AutoCAD'in ticari olarak pazarlanmasında kullanılan sürüm adıdır. Çoğunlukla sürüm hakkında yapılan tartışmalar ve hata bildirimlerinde kullanılır. İç sürüm adı ise AutoCAD'in windows kayıt defterindeki adıdır ve daha çok uygulama geliştiricileri tarafından sürüm faklılıklarından doğan uyum sorunlarını çözmekte kullanılır. İç sürüm adının ilk iki hanesi AutoCAD uygulamaları açısından ikili (binary) uyumluluğu gösterir. Örneğin AutoCAD 17.0 için geliştirdiğiniz bir uygulama AutoCAD 17.1  ve AutoCAD 17.2 için de geçerlidir.
