R ile Veri Temizleme Pratiği
================

R ile Veri Temizleme Pratiği başlığı altındaki bu uygulama NewslabTurkey'de yayınlanan [R ekosisteminde dağınık veriler nasıl temizlenir?](https://www.newslabturkey.org/author/sadettindemirel/) yazısı için üretilmiştir. Bu pratikte kullanılan veri setine [buradan ulaşabilirsiniz](https://github.com/sadettindemirel/veRi_temizleme/blob/master/%C3%B6rnek_veri.xls)

[**By Sadettin Demirel**](https://twitter.com/demirelsadettin)

Veri temizleme, veri haberciliği süreçlerinde gazetecilerin en çok mesai harcadığı, zahmetli adımlardan bir tanesi. Günümüzde yapılandırılmış veri kaynakları giderek artıyor olsa da analize başlamadan önce veriyi yayınlandığı formattan (bkz: TÜİK) kurtarmak, ve derli hale getirmek gerekiyor. Özellikle 7/24 haber akışı ve gazetecinin üzerindeki zaman baskısı düşünüldüğünde zaten zahmetli olan bu süreç daha da önemli hale geliyor. Bu noktada yapılan akademik araştırmalar da gazetecilerin veri haberciliği süreçlerinde en çok zamandan şikayet ettiğini doğruluyor. Fakat bu zahmetli süreci birkaç satır R koduyla kolaylaştırmak, zamandan tasarruf etmek mümkün. Peki nasıl? Bu yazıda R yazılımı ile dağınık verileri temizleme yollarını uygulamalı olarak anlatacağım. Kemerleri bağlayalım, sıkı bir yolculuk olacak :)

#### 1.Teşhis koyalım

ilk veri seti:

-   Veri setini açtığımda ilk gözüme takılan ilk satır boş ve değişkenler yerine başlık ve kısa bilgiler yer alıyor
-   Değişken isimleri düzenli değil, büyüklü küçüklü harfler ve noktalama işaretleri içeriyor.
-   Cinsiyet ve Satın Alma sütunlarına ait hücrelerde verilerde boşluklar var ve yine büyüklü küçüklü veriler görülüyor.
-   Ayrıca bazı durumlarda aynı şeyi ifade eden terimler birlikte kullanılmış (IOS / iPhone)
-   Bazı hücrelerde Kadın yerine K Erkek yerine E gibi kısaltmalar yer alıyor
-   Son olarak sayısal veri olması gereken saat metin verisi olarak yer alıyor.

#### 2.Paketleri yükleyelim

Daha önce yüklemiş olduğum için bu kısımda \# işaretini kaldırarak devam ediniz.

``` r
# install.packages("readxl")
# install.packages("tidyverse")
# install.packages("lubridate")
# install.packages("janitor")
# install.packages("readr")
```

Paketleri yükledikten sonra **library** komutu ile çağırıyoruz. Aksi takdirde kullanacağımız kodlar çalışmayabilir.

``` r
library("readxl")
library("tidyverse")
library("skimr")
library("janitor")
library("readr")
```

#### 2.Dosyayı içeri aktaralım

Veriyi doğrudan aktarabiliriz ama üst satırlar boş olduğu için R değişkenleri değer olarak algılıyor

``` r
anket_kirli <- read_excel("~/downloads/örnek_veri.xls") #anket_kirli ismine tanımladık

anket_kirli #tanımladığımız veriyi konsolda yazdırdık ve aşağıdaki veri setine ulaştık
```

    ## # A tibble: 104 x 5
    ##    `Mobil Oyun Tercihleri Anketi`           ..2    ..3      ..4    ..5     
    ##    <chr>                                    <chr>  <chr>    <chr>  <chr>   
    ##  1 Bu veri seti veri temizleme pratiği içi… <NA>   <NA>     <NA>   <NA>    
    ##  2 <NA>                                     <NA>   <NA>     <NA>   <NA>    
    ##  3 "*?CinSiyet\""                           Yaş-?/ "\"Sist… SÜRE   "SATIN_…
    ##  4 Erkek                                    35 - … IOS      2 saat Evet    
    ##  5 Erkek                                    25 - … IOS      1 saat Hayır   
    ##  6 Erkek                                    25 - … IOS      4 saat EVET    
    ##  7 Erkek                                    18 - … IOS      2 saat EVET    
    ##  8 ERKEK                                    18 - … Ios      1.5 s… Hayır   
    ##  9 Erkek                                    18 - … iphone   2 saat Hayır   
    ## 10 Erkek                                    18 - … IOS      2 saat Hayır   
    ## # … with 94 more rows

1.  Satırdaki değişken isimlerini değer olarak algıladı. Bu durumu düzeltmek için read\_excel( ) fonksiyonunun “skip” argümanını kullanarak R’a gereksiz satırları atlamasını belirteceğiz.

``` r
anket_skip <- read_excel("~/downloads/örnek_veri.xls", skip = 4) #anket_skip ismine tanımladık

anket_skip #tanımladığımız veriyi konsolda yazdırdık ve aşağıdaki veri setine ulaştık
```

    ## # A tibble: 101 x 5
    ##    `*?CinSiyet"` `Yaş-?/` `"Sistem"` SÜRE     `SATIN_alma"`
    ##    <chr>         <chr>    <chr>      <chr>    <chr>        
    ##  1 Erkek         35 - 50  IOS        2 saat   Evet         
    ##  2 Erkek         25 - 35  IOS        1 saat   Hayır        
    ##  3 Erkek         25 - 35  IOS        4 saat   EVET         
    ##  4 Erkek         18 - 25  IOS        2 saat   EVET         
    ##  5 ERKEK         18 - 25  Ios        1.5 saat Hayır        
    ##  6 Erkek         18 - 25  iphone     2 saat   Hayır        
    ##  7 Erkek         18 - 25  IOS        2 saat   Hayır        
    ##  8 K             18 - 25  IOS        1 saat   Evet         
    ##  9 K             18 - 25  ıOS        1 saat   Hayır        
    ## 10 KADIN         18 - 25  IOS        3 saat   Hayır        
    ## # … with 91 more rows

Değişken isimleri acayip dağınık bir şekilde işlenmiş. Bunun üstesinden gelmek için veriyi R yazılımına aktarırken “col\_names” argümanını kullanarak kendi değişken isimlerini değiştire biliriz. Ayrıca değerlerden önce yer alan boşlukları da *trim\_ws* yani trim whitespace argümanıyla ortadan kaldırabiliriz Nasıl?

``` r
anket <- read_excel("~/downloads/örnek_veri.xls", skip = 5, col_names =c("cinsiyet","yaş_grubu","sistem","süre","satın_alma"), trim_ws = TRUE)

anket
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem süre     satın_alma
    ##    <chr>    <chr>     <chr>  <chr>    <chr>     
    ##  1 Erkek    35 - 50   IOS    2 saat   Evet      
    ##  2 Erkek    25 - 35   IOS    1 saat   Hayır     
    ##  3 Erkek    25 - 35   IOS    4 saat   EVET      
    ##  4 Erkek    18 - 25   IOS    2 saat   EVET      
    ##  5 ERKEK    18 - 25   Ios    1.5 saat Hayır     
    ##  6 Erkek    18 - 25   iphone 2 saat   Hayır     
    ##  7 Erkek    18 - 25   IOS    2 saat   Hayır     
    ##  8 K        18 - 25   IOS    1 saat   Evet      
    ##  9 K        18 - 25   ıOS    1 saat   Hayır     
    ## 10 KADIN    18 - 25   IOS    3 saat   Hayır     
    ## # … with 91 more rows

Fakat her veri 5 değişkenden (sütundan) ibaret değil. 50 değişkenden oluşan bir veri setiyle çalıştığımızı düşündüğümüzde 50 ayrı değişken ismini manuel olarak değiştirmek kabus gibi bir şey olurdu. Janitor paketini işte tam da bu yüzden bu pratiğe dahil ettik. Paketin clean\_names() fonksiyonu bizi gereksiz ifadelerden kurtaracak, ayrıca Türkçe karakterler var ise onları İngilizce olarak değiştirecek

``` r
anket_janitor <- read_excel("~/downloads/örnek_veri.xls", skip = 4)
anket_janitor <- clean_names(anket_janitor)
anket_janitor
```

    ## # A tibble: 101 x 5
    ##    cin_siyet yas     sistem sure     satin_alma
    ##    <chr>     <chr>   <chr>  <chr>    <chr>     
    ##  1 Erkek     35 - 50 IOS    2 saat   Evet      
    ##  2 Erkek     25 - 35 IOS    1 saat   Hayır     
    ##  3 Erkek     25 - 35 IOS    4 saat   EVET      
    ##  4 Erkek     18 - 25 IOS    2 saat   EVET      
    ##  5 ERKEK     18 - 25 Ios    1.5 saat Hayır     
    ##  6 Erkek     18 - 25 iphone 2 saat   Hayır     
    ##  7 Erkek     18 - 25 IOS    2 saat   Hayır     
    ##  8 K         18 - 25 IOS    1 saat   Evet      
    ##  9 K         18 - 25 ıOS    1 saat   Hayır     
    ## 10 KADIN     18 - 25 IOS    3 saat   Hayır     
    ## # … with 91 more rows

Değişken isimlerini temizlediğimize göre şimdi daha derinlere inme zamanı. R'ın değişkenleri doğru tanımlaması bizim için çok önemli. O halde R'a veri setinin yapısını soralım. Bu noktada str() veya glimpse() gibi komutlar kullanabiliriz.

``` r
str(anket)
```

    ## Classes 'tbl_df', 'tbl' and 'data.frame':    101 obs. of  5 variables:
    ##  $ cinsiyet  : chr  "Erkek" "Erkek" "Erkek" "Erkek" ...
    ##  $ yaş_grubu : chr  "35 - 50" "25 - 35" "25 - 35" "18 - 25" ...
    ##  $ sistem    : chr  "IOS" "IOS" "IOS" "IOS" ...
    ##  $ süre      : chr  "2 saat" "1 saat" "4 saat" "2 saat" ...
    ##  $ satın_alma: chr  "Evet" "Hayır" "EVET" "EVET" ...

``` r
glimpse(anket)
```

    ## Observations: 101
    ## Variables: 5
    ## $ cinsiyet   <chr> "Erkek", "Erkek", "Erkek", "Erkek", "ERKEK", "Erkek",…
    ## $ yaş_grubu  <chr> "35 - 50", "25 - 35", "25 - 35", "18 - 25", "18 - 25"…
    ## $ sistem     <chr> "IOS", "IOS", "IOS", "IOS", "Ios", "iphone", "IOS", "…
    ## $ süre       <chr> "2 saat", "1 saat", "4 saat", "2 saat", "1.5 saat", "…
    ## $ satın_alma <chr> "Evet", "Hayır", "EVET", "EVET", "Hayır", "Hayır", "H…

Her iki komut sonucunda da **cinsiyet** değişkeni *chr* yani metin verisi. Yaş grubu da aynı şekilde. Fakat **süre** değişkeni sayısal bir formatta değil, yani sayı verisi değil. Fakat daha büyük sorunlarımız var: Büyüklü küçüklü *EVET, Evet, Hayır, HAYIR* değerleri, kısaltmalar *E, K*, ve son olarak *IOS, iPhone, Apple ios* gibi ayni şeyi ifade eden farklı terimler. Excel'de bunları teşhis etmiş olsak da aynı şekilde görmek için R'da **count()** komutunu kullanabiliriz

``` r
anket %>% count(cinsiyet)
```

    ## # A tibble: 6 x 2
    ##   cinsiyet     n
    ##   <chr>    <int>
    ## 1 E           12
    ## 2 Erkek       59
    ## 3 ERKEK       14
    ## 4 K            6
    ## 5 KADIN        5
    ## 6 Kadın        5

``` r
anket %>% count(satın_alma)
```

    ## # A tibble: 4 x 2
    ##   satın_alma     n
    ##   <chr>      <int>
    ## 1 Evet          38
    ## 2 EVET           2
    ## 3 HAYIR          3
    ## 4 Hayır         58

``` r
anket %>% count(sistem)
```

    ## # A tibble: 10 x 2
    ##    sistem        n
    ##    <chr>     <int>
    ##  1 Android      52
    ##  2 ANDROID      16
    ##  3 Apple        15
    ##  4 Apple IOS     6
    ##  5 Huawei        1
    ##  6 Ios           1
    ##  7 IOS           7
    ##  8 iphone        1
    ##  9 ıOS           1
    ## 10 Sony          1

Bu noktada ilk olarak yaş verisini *parse\_number* komutu ile metin verisinden sayı verisine dönüştüreceğiz. Sonrasında **anket** veri setindeki ifadeleri küçük harfli haline getireceğiz. Böylelikle kısaltmalar dışında temizleme işlemlerini tamamlamış olacağız. Bunu **stringr** paketinin **str\_to\_lower** komutu ve dplyr paketinin **mutate** fiiliyle gerçekleştireceğiz. Bu dpyr ve mutate nedir? diye merak edenleri daha önce newslabturkey'de yayınlanmış olan **veri manipülasyonu** yazımı öneriyorum.

``` r
anket %>% mutate(süre = parse_number(süre),#metin verisinden sayıs verisine
                  cinsiyet = str_to_lower(cinsiyet),#değerleri küçük harflere dönüştürme
                  sistem = str_to_lower(sistem), 
                  satın_alma = str_to_lower(satın_alma))
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem  süre satın_alma
    ##    <chr>    <chr>     <chr>  <dbl> <chr>     
    ##  1 erkek    35 - 50   ios      2   evet      
    ##  2 erkek    25 - 35   ios      1   hayır     
    ##  3 erkek    25 - 35   ios      4   evet      
    ##  4 erkek    18 - 25   ios      2   evet      
    ##  5 erkek    18 - 25   ios      1.5 hayır     
    ##  6 erkek    18 - 25   iphone   2   hayır     
    ##  7 erkek    18 - 25   ios      2   hayır     
    ##  8 k        18 - 25   ios      1   evet      
    ##  9 k        18 - 25   ıos      1   hayır     
    ## 10 kadin    18 - 25   ios      3   hayır     
    ## # … with 91 more rows

``` r
anket_küçük <- anket %>% mutate(süre = parse_number(süre),
                  cinsiyet = str_to_lower(cinsiyet), 
                  sistem = str_to_lower(sistem), 
                  satın_alma = str_to_lower(satın_alma))
anket_küçük
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem  süre satın_alma
    ##    <chr>    <chr>     <chr>  <dbl> <chr>     
    ##  1 erkek    35 - 50   ios      2   evet      
    ##  2 erkek    25 - 35   ios      1   hayır     
    ##  3 erkek    25 - 35   ios      4   evet      
    ##  4 erkek    18 - 25   ios      2   evet      
    ##  5 erkek    18 - 25   ios      1.5 hayır     
    ##  6 erkek    18 - 25   iphone   2   hayır     
    ##  7 erkek    18 - 25   ios      2   hayır     
    ##  8 k        18 - 25   ios      1   evet      
    ##  9 k        18 - 25   ıos      1   hayır     
    ## 10 kadin    18 - 25   ios      3   hayır     
    ## # … with 91 more rows

#### 3.Değerleri yeniden kodlayalım

Tabloda görüldüğü üzere tüm veri değerlerini küçük harfli hale getirdik. Geriye kısaltmalar kaldı. Aşağıda **count** komutuyla yaptığımız hesaplamaya göre kısaltmalar sadece **cinsiyet** ve **sistem** değişkenlerinde yer alıyor. O halde Excel'deki bul ve değiştir (find and replace) komutune benzer bir şekilde malum kısaltmaları veyahut benzer terimleri yeniden kodlayalım.

``` r
anket_küçük %>% count(cinsiyet)
```

    ## # A tibble: 5 x 2
    ##   cinsiyet     n
    ##   <chr>    <int>
    ## 1 e           12
    ## 2 erkek       73
    ## 3 k            6
    ## 4 kadin        5
    ## 5 kadın        5

``` r
anket_küçük %>% count(sistem)
```

    ## # A tibble: 8 x 2
    ##   sistem        n
    ##   <chr>     <int>
    ## 1 android      68
    ## 2 apple        15
    ## 3 apple ios     6
    ## 4 huawei        1
    ## 5 ios           8
    ## 6 iphone        1
    ## 7 ıos           1
    ## 8 sony          1

Kodlama işlemini **mutate** ve **recode** fiileriyle gerçekleştireceğiz. Öncelikle en son tanımladığımız **anket\_küçük** veri setini zincir operatörü ile **mutate** fiiline bağlıyoruz. Sonrasında yeniden kodlayacağımız değişken (**cinisyet**) ismini **recode( )** komutuna eşitliyoruz. Bundan sonraki adım çok önemli! **recode** komutu içerisine önce temizlenecek değişkenin ismi (*cinsiyet*), sonra tırnak içinde yeniden kodlanacak değeri(**e** ve **k**) istenilen ifadeye (**erkek** veya **kadın**) yine tırnak içinde tanımlıyoruz. Bu kadar!

Basitçe açıklamak gerekirse R'a istediğimiz ifadeleri bulmasını ve yeniden kodlamasını(**recode**) etmesini belirtiyoruz.

``` r
anket_küçük %>% 
  mutate(cinsiyet = recode(cinsiyet, "e" = "erkek","k"="kadın"))
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem  süre satın_alma
    ##    <chr>    <chr>     <chr>  <dbl> <chr>     
    ##  1 erkek    35 - 50   ios      2   evet      
    ##  2 erkek    25 - 35   ios      1   hayır     
    ##  3 erkek    25 - 35   ios      4   evet      
    ##  4 erkek    18 - 25   ios      2   evet      
    ##  5 erkek    18 - 25   ios      1.5 hayır     
    ##  6 erkek    18 - 25   iphone   2   hayır     
    ##  7 erkek    18 - 25   ios      2   hayır     
    ##  8 kadın    18 - 25   ios      1   evet      
    ##  9 kadın    18 - 25   ıos      1   hayır     
    ## 10 kadin    18 - 25   ios      3   hayır     
    ## # … with 91 more rows

Yaptığımız işlemi yeni bir isme tanımlamalıyız. Bunu **&lt;-** sembolü ile aşağıda görüldüğü gibi yapabiliriz. Bunun nedeni bir sonraki aşamada **derli\_anket** veri setini kullanarak sistem verisini yeniden kodlayacağız.

``` r
derli_anket <-anket_küçük %>% 
  mutate(cinsiyet = recode(cinsiyet, "e" = "erkek","k"="kadın"))
```

``` r
derli_anket %>% 
  mutate(sistem = 
           recode(sistem, "iphone"="ios", "apple" = "ios",
                  "apple ios"="ios","ıos" ="ios",
                  "huawei"="android","sony"="android"))
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem  süre satın_alma
    ##    <chr>    <chr>     <chr>  <dbl> <chr>     
    ##  1 erkek    35 - 50   ios      2   evet      
    ##  2 erkek    25 - 35   ios      1   hayır     
    ##  3 erkek    25 - 35   ios      4   evet      
    ##  4 erkek    18 - 25   ios      2   evet      
    ##  5 erkek    18 - 25   ios      1.5 hayır     
    ##  6 erkek    18 - 25   ios      2   hayır     
    ##  7 erkek    18 - 25   ios      2   hayır     
    ##  8 kadın    18 - 25   ios      1   evet      
    ##  9 kadın    18 - 25   ios      1   hayır     
    ## 10 kadin    18 - 25   ios      3   hayır     
    ## # … with 91 more rows

Son olarak yaptığımız işlemi **derli\_anket** verisine tanımlayarak anket verimizi temizleme işlemini tamamlamış oluyoruz.

``` r
derli_anket <- derli_anket %>% 
  mutate(sistem = 
           recode(sistem, "iphone"="ios", "apple" = "ios",
                  "apple ios"="ios","ıos" ="ios",
                  "huawei"="android","sony"="android"))

#iki ayrı mutate() komutunu %>% zincir operatörü ile birbirne bağlayarak da aynı işlemi yapabilirdik.

derli_anket
```

    ## # A tibble: 101 x 5
    ##    cinsiyet yaş_grubu sistem  süre satın_alma
    ##    <chr>    <chr>     <chr>  <dbl> <chr>     
    ##  1 erkek    35 - 50   ios      2   evet      
    ##  2 erkek    25 - 35   ios      1   hayır     
    ##  3 erkek    25 - 35   ios      4   evet      
    ##  4 erkek    18 - 25   ios      2   evet      
    ##  5 erkek    18 - 25   ios      1.5 hayır     
    ##  6 erkek    18 - 25   ios      2   hayır     
    ##  7 erkek    18 - 25   ios      2   hayır     
    ##  8 kadın    18 - 25   ios      1   evet      
    ##  9 kadın    18 - 25   ios      1   hayır     
    ## 10 kadin    18 - 25   ios      3   hayır     
    ## # … with 91 more rows

#### 4.Veri setini dışarı aktarma

Temizlediğimiz veriyi dışarıya csv olarak aktarabiliriz. Bunu yapmak için öncelikle **readr** paketini çağıryoruz. Sonrasında **write\_csv()** komutu içerisine önce veri seti ismi ("derli\_anket"), sonrasında kaydedilecek klasörü dosyanın uzantısı ise belirtiyoruz.

``` r
library(readr)
write_csv(derli_anket, "~/desktop/derlianket.csv")
```
