Python’da SQL sorguları çalıştırabilmek için SQLite3 kütüphanesini kullandım.

SORU 1: Z distribütörüne ait ürünlerinin toplam satışlarını (TL) getiren SQL sorgusu:

SELECT f.prod_id: Bu, sonuç kümesinde her bir ürünün (prod_id) dahil edilmesini sağlar.

SUM(f.tl) AS ToplamSatis: Bu, MonthlyRetailSales tablosundaki tl sütunundaki değerlerin toplamını hesaplar ve bu toplamı ToplamSatis olarak adlandırır.

GROUP BY f.prod_id: Bu, prod_id sütununa göre gruplama yapar. Bu, her bir ürün için toplam satış değerini hesaplamamızı sağlar.

WHERE : Bu kısım, RetailProduct tablosunda distributor sütunu değeri 'Z' olan satırların filtreleneceğini belirtir. Yani, sadece distribütörü 'Z' olan ürünler sorguya dahil edilecektir.

JOIN :  Bu iki tablo, prod_id sütunu üzerinden birleştiriliyor. Bu, her iki tablodaki prod_id değerlerinin eşleşmesi durumunda satırların birleştirileceği anlamına gelir.

SORU 2: 2023 yılı içerisinde aylık Bazda D ürününün satış miktarını (TL) veren SQL
sorgusu:

SELECT: strftime('%Y-%m', s.month_date) AS YilAy: month_date sütunundan yıl ve ay kısmı (YYYY-MM formatında) alınır ve sonuç kümesinde YilAy olarak adlandırılır.
s.prod_id: Her bir ürünün kimliği (prod_id) sonuç kümesine dahil edilir.
SUM(s.tl) AS ToplamSatis: MonthlyRetailSales tablosundaki tl sütunundaki değerlerin toplamı hesaplanır ve bu toplam ToplamSatis olarak adlandırılır.

JOIN: MonthlyRetailSales tablosu s takma adıyla ve RetailProduct tablosu f takma adıyla kısaltılarak kullanılır.
Bu iki tablo, prod_id sütunu üzerinden birleştirilir (JOIN). Bu, her iki tablodaki prod_id değerlerinin eşleşmesi durumunda satırların birleştirileceği anlamına gelir.

WHERE: f.production = 'D': Sadece production sütunu 'D' olan ürünler seçilir.
strftime('%Y', s.month_date) = '2023': MonthlyRetailSales tablosundaki month_date sütunundaki tarihlerden yıl kısmı '2023' olan satırlar seçilir. Bu, sadece 2023 yılına ait verileri dahil eder.

GROUP BY strftime('%Y-%m', s.month_date), s.prod_id: Yıl-ay ve ürün kimliğine göre gruplama yapılır. Bu, her bir ürün için her ayın toplam satış değerini hesaplamamızı sağlar.

ORDER BY :s.prod_id: Sonuçlar önce ürün kimliğine göre sıralanır.
strftime('%Y-%m', s.month_date): Ardından sonuçlar yıl-ay değerine göre sıralanır.

SORU 3: 2022 yılından 2023 yılının sonuna kadar Aylık Bazda C distibütörünün Pazar Payını getiren
SQL sorgusu:

SELECT : strftime('%Y-%m', s.month_date) AS YilAy: month_date sütunundan yıl ve ay kısmı (YYYY-MM formatında) alınır ve sonuç kümesinde YilAy olarak adlandırılır.

SUM(CASE WHEN p.distributor = 'Z' THEN s.tl ELSE 0 END) * 1.0 / SUM(s.tl) AS PazarPayi:
CASE WHEN p.distributor = 'Z' THEN s.tl ELSE 0 END: Eğer distributor sütunu 'Z' ise s.tl değerini alır, değilse 0 alır.
SUM(CASE WHEN p.distributor = 'Z' THEN s.tl ELSE 0 END): Distribütörü 'Z' olan ürünlerin toplam satışlarını hesaplar.
SUM(s.tl): Ayın toplam satışlarını hesaplar.
SUM(CASE WHEN p.distributor = 'Z' THEN s.tl ELSE 0 END) * 1.0 / SUM(s.tl): Distribütörü 'Z' olan ürünlerin toplam satışlarını ayın toplam satışlarına böler. * 1.0 ifadesi, sonuçların ondalıklı değer olarak dönmesini sağlar.
AS PazarPayi: Hesaplanan oran (pazar payı) PazarPayi olarak adlandırılır. 

JOIN : MonthlyRetailSales tablosu s takma adıyla ve RetailProduct tablosu p takma adıyla kısaltılarak kullanılır.
Bu iki tablo, prod_id sütunu üzerinden birleştirilir. Bu, her iki tablodaki prod_id değerlerinin eşleşmesi durumunda satırların birleştirileceği anlamına gelir.

WHERE: strftime('%Y', s.month_date) >= '2022' AND strftime('%Y', s.month_date) <= '2023': MonthlyRetailSales tablosundaki month_date sütunundaki tarihlerden yıl kısmı 2022 ve 2023 olan satırlar seçilir. Bu, sadece 2022 ve 2023 yıllarına ait verileri dahil eder.


GROUP BY: strftime('%Y-%m', s.month_date): Yıl-ay kısmına göre gruplama yapılır. Bu, her bir ay için pazar payını hesaplamamızı sağlar.

ORDER BY : strftime('%Y-%m', s.month_date): Sonuçlar yıl-ay değerine göre sıralanır.

SORU 4: 2022 ve 2023 Yılı Satış verilerini kullanarak C ürünlerinin toplam TL bazında
satışlarını 2024 yılı ilk 6 ayı için tahminleme :

Burada öncelikle 2022 ve 2023 verilerini filtreledim. Daha sonra C ürünlerinin prod_id değerlerini
RetailProduct dataframe’imden çektim. Sonra bu prod_id değerlerine sahip verileri 2022-23
satış verilerinin olduğu dataframe’den çektim ve elimde sadece C ürünlerinin 2022-2023 yıllarına ait satış verileri kalmış oldu.

Daha sonra prod_id değerlerine göre ortalama ve medyanı hesapladım, burada 275 mglik C ürünlerinin
medyan ve ortalaması birbirine yakın çıktı,bu ürünlerin satış değerlerinde bir harmoni olduğunu anlıyoruz.

4 ayrı C vardı, bunlar için ayrı dataframeler oluşturdum. Burada zaman serisi modellerinden
ARIMA modelini kullanacağım, bunun için month_date sütunumu index’e çektim. Satışların dağılımını
görmek için distplot metodunu kullandım. Burada satış miktarları çok geniş bir aralıkta dağılmamış,
nadiren de olsa yüksek miktarlarda satışın olduğu dönemler olmuş. Bu dönemlerdeki satışı etkileyen faktörler farklı veriler de ele alınarak incelenebilir.

Daha sonra verilerin durağan olup olmadığına bakmak için bir fonksiyon kullandım, burada verilerin
durağan olmadığını gördüm, durağan hale getirmek için ARIMA modelinde “d” değerini 1 vererek
ilerledim. Daha sonra en iyi arıma modeli için en optimum parametreleri seçmek için auto_arima fonksiyonu kullandım. Daha sonra 2022-23 verilerimi eğitim ve test verileri olarak ayırıp modelimi
eğittim ve tahmin yaptım. Ne kadar gerçeğe yakın tahmin yaptığını görmek için. İyi sayılabilecek bir
sonuç aldım bu sonucları grafikle de gösterdim. Burada tahminler birbirine yakın aynı zamanda
yükseliş ve düşüş trendleri birbiriyle çoğunlukla uyumluydu. Daha sonra tüm verilerimi kullanarak bir
model eğittim ve 2024 yılı ilk 6 ayı için tahmin yaptırıp yine grafikle bunu gösterdim. Burada 4 farklı
C ürünü vardı, bu sebeple tüm bu işlemleri bir fonksiyona aldım, ve diğer ürünler için de tahminleme işlemini gerçekleştirdim.
