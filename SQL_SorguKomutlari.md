# SQL_SorguKomutlari

- pgadmin ile bir tane "KitapKiralama" database oluşturalım.
- ilgili database sağ tıklayıp "query tool" seçip komutlara başlayabiliriz.

---

```sql
--- Öğretmen Kitap Kiralama İşlemleri -----
-------------------------------------------
create table tblkitap(
  id bigserial primary key,
  ad varchar(200),
  sayfasayisi int,
  yayinevi varchar(200),
  cevirmen varchar(100),
  basimsayisi int,
  basimyili int,
  aciklama varchar(200),

  createat bigint default extract(epoch from now())*1000,
  updateat bigint default extract(epoch from now())*100,
  state int default 1
)

-- yazar oluşturalım

create table tblyazar(
  id bigserial primary key,
  ad varchar(100),
  biyagrafi varchar(500),
  resim varchar(255),
  artalanresmi varchar(255),
  createat bigint default extract(epoch from now())*100,
  updateat bigint default extract(epoch from now())*1000
)
-- Dikkat -> iki tablo arasında ilişki var ise bu tablolarda verilerin tutarlı
  -- olmasını sağlamak zorundayız.
create table tblkitapyazar(
  id bigserial primary key,
  yazarid bigint,
  kitapid bigint
)
insert into tblkitap(ad) values ('Java SE'), ('Java Spring Boot')
select * from tblkitap --kitap eklenmiş mi kontrol edelim

insert into tblyazar(ad) values ('Mert Hoca'),('Ayşe Hoca')
select * from tblyazar
  
insert into tblkitapyazar(yazarid,kitapid) values(1,1),(2,1),(1,2)
select * from tblkitapyazar

--- Bu durumda tblkitapyazar kısmında Mert Hoca'ı silersek diğer alanlarda 
-- bu kısım kalacak o yüzden bu tür işlemler doğru değil şimdi tüm table' silip hepsini
-- yeniden oluşturalım.
drop table tblkitap
drop table tblyazar
drop table tblkitapyazar
-- tblkitapyazar'ı aşağıdaki şekilde oluşturalım.
create table tblkitapyazar(
  id bigserial primary key,
  yazarid bigint references tblyazar(id), --foreignkey olarak işaretlenir.
  kitapid bigint references tblkitap(id),
)
-- tlbkitap ve tblyazar aynı şekilde oluşturabilriz.
-- tekrardan insert edelim.
insert into tblkitapyazar(yazarid,kitapid) values(1,1),(2,1),(1,2)

--foreignkey olan tablolarda kayıtlar tutarlılık nedeniyle silinemez.
-- foreignkey olan tablolarda yine ekleme yaparken ekleme yapılamaz mesela 
--insert into tblkitapyazar(yazarid,kitapid) values(15,1) buna izin vermez

-- yine tblkitapyazar silinmeden diğer tblkitap ve tblyazar silemezsiniz.
-- şimdi tblkonu diye bir table oluşturlaım ve bunu tblkitap ile ilişkilendirelim.
-- bir konuda bir sürü kitap olabilir . roman konusu altında birçok kitao vardır.
-- ilişki one to many olacak.

--tblkitao yeniden oluşturalım önce hepsini silmek lazım.
drop table tblktapyazar
drop table tblkitap
drop table tblyazar

-- tblkonu oluşturalım
  create table tblkonu(
  id bigserial primary key,
  ad varchar(100)
  )
-- tblkitap oluşturlaım 
create table tblkitap(
  id bigserial primary key,
  ad varchar(200),
  sayfasayisi int,
  yayinevi varchar(200),
  cevirmen varchar(100),
  basimsayisi int,
  basimyili int,
  aciklama varchar(200),
  konuid bigint references tblkonu(id), -- bunu ekledik
  createat bigint default extract(epoch from now())*1000,
  updateat bigint default extract(epoch from now())*100,
  state int default 1
)

create table tblyazar(
  id bigserial primary key,
  ad varchar(100),
  biyagrafi varchar(500),
  resim varchar(255),
  artalanresmi varchar(255),
  createat bigint default extract(epoch from now())*100,
  updateat bigint default extract(epoch from now())*1000
)

create table tblkitapyazar(
  id bigserial primary key,
  yazarid bigint references tblyazar(id), --foreignkey olarak işaretlenir.
  kitapid bigint references tblkitap(id),
)
-- hepsini sırayla sildikten sonra tekrar oluşturalım.
----------------
-- Şimdi örnekleri geliştirelim
create table tblbrans(
  id bigserial primary key,
  ad varchar(100)
)
  create table tblokul(
  id bigserial primary key,
  ad varchar(100),
  adres varchar(300),
  iletisim varchar(100),
  okulmuduru varchar(100)
  )
  
create table tblogretmen(
  id bigserial primary key,
  ad varchar(100),
  soyad varchar(100),
  telefon varchar(20),
  bransid bigint references tblbrans(id),   -- Matematik, mat., MATEMATİK, matematik bu şekilde olmaması için bir table daha açalım yukarıda
  okulid bigint references tblokul(id),
  tecrube int
)
--- öğretmenin adresi birden fazla oalcağı için bir table açmamız lazım
create table tbladres(
  id bigserial primary key,
  ad varchar(100),
  il varchar(100),
  ilce varchar(200),
  mahalle varchar(100),
  tanim varchar(400),
  telefon varchar(20),
  ogretmenid bigserial references tblogretmen(id)
)

-------
--- Kiralama örneği yapalım
create table tblkiralama(
  id bigserial primary key,
  tarih bigint, -- long time demek
  sure int,
  ucret numeric(5,2), --- 00,000.00₺
  odemedurum int, -- 0: ödenmedi, 1: ödendi, 2: iade
  iadetarihi bigint,
  gecikmenedeni varchar(200),
  farkucreti numeric(5,2),
  toplamucret numeric(5,2),
  durum int,
  kitapid bigint references tblkitap(id),
  ogretmenid bigint references tblogretmen(id)
)

create table tblodeme(
  id bigserial primary key,
  kiralamaid bigint references tblkiralama(id),
  odemeturu varchar(100),
  tarih bigint,
  tutar numeric(5,2)
)

-- pgadmin de table sekmesine gleip sağ tıklayıp refresh derseniz oluşturduğumuz table'lar gözükcektir.

---------------------------------------------------------------------------
-- Şimdi CRUD konusunu işleyelim.
-- CRUD
-- Create
-- Read
-- Update
-- Delete
------------------
--- INSERT INTO -> veritabanı tablolarına kayıt eklemek için kullanılır.
-- insert into [TABLE_NAME]([TABLE_COLUMNS]) values ([VALUES])

-- Yukarıda oluşturduğumuz table'lara ekleme yapalım 
  -- id bigserial olduğu için onu otomatik atayacaktır onu eklememize gerek yok
insert into tblkonu(ad) values ('Roman') -- Burada konu tablosua bir adet kayıt eklenir.

-- Birden fazla kayıt eklemek için
insert into tblkonu(ad) values ('Hikaye'), ('Bilisim'), ('Öykü'), ('Biyografi')

-- Dikkat!!!
  -- Eğer tablo kolonlarını hangi sıra ile yazdıysak values alanına o sıra ile değeleri eklemek zorundayız.

-- Bir tabloyu görüntülemek için SELECT kullanılır. 
select * from tblkonu 


--- UPTADE -> veritabanındaki kayıtları güncellemek için kullanılır.
update tblkonu set ad = 'Polisiye' where id =6 -- id'si 6 olan Roman'ı Polisiye olarak değiştiriyoruz.
-- Bunu yapma
update tblkonu set ad = 'Polisiye' -- -> konu tablosundaki tüm ad alanlarını polisiye ile değiştirir.

-- DELETE -> veritabanındaki satılar silmek için kullanılır, yani kayıt silmekte kullanırız.
delete from tblkonu -- bunu yapma içindeki her şeyi siler !!!!
delete from tblkonu where id=4 -- id si 4 olan satırı siler.
-- delete from tblkonu yaptığımızda tüm satır silinir ancak sequence tarafında silinme olmaz. son id 6 olsn tüm satırları
-- silip tekrar kayıt eklediğimizde id 7 den başlar. 

-----------------------------------------
-----------------------------------------
-- DB de sorgulama yapma işlemleri 
------------ SELECT -------------
-- Select önüne geldiği bir değer, tablo, view, function gibi varlığı
--  tablo olarak gösterme eğilimdedir. Yani select mümkün olduğu kadar tablo olarak sonuç döner.
select 'Nasılsınız?' 
select 51+7
select now() 
-------------
-- AS -> takma ad diyebiliriz
select 'Mert' as Ad -- column name "AD" olarak gösterir.
select 5+7 as toplama -- -> column name "toplama" olur

--------------------------------------
-- tablolara select atılması
-- select -> seç, * -> her şeyi ( tüm kolonlar), from -> -den, [TABLE_NAME]
select * from tblyazar -- -> tüm kolonları seçer

-- Belirli column name alma
select ad, resim from tblyazar -- ad ve resim kolonları gelir.

-- ad yerine yazar_adi yazdırarak çağıralaım
select ad as yazar_adi, resim from tblyazar --ad değişmiyor sadece yazar_adi olarak gösteriyor.

-- ilerleyen konularda join girecek o yüzden select atarken doğrusu ilgili table ismini vererek select atmaktır.
select tblyazar.ad, tblyazar.resim from tblyazar 
-- daha profesyonel olmak için kısaltma işimizi görür çünkü uzun table isimlerini kısaltmamız gerekebilir.
select y.ad, y.resim from tblyazar as y 

--- link : https://www.mockaroo.com/ 
-- Yukarıdaki link random data oluşturmak için harika bir site. pratik yapmak için birebir.

-- Siteden fake olarak tblyazar için dummy data oluşturun. 1000 satır yapabilirsiniz sorun değil. 
-- pgadmin sağ tıklayıp "query tool" -> open dedikten sonra indirdiğiniz sql dosyasını seçin 1000 kayıt ekleyin.
-- şimdi sorgu işlemini yapaibliiriz.

---- Sorgu İşlemleri -----
select * from tblyazar -- tüm dataları getirir.
select * from tblyazar where id=7 -- tek kayıt döner. id'si 7 olan kayıt.
select * from tblyazar where id<9 -- id'si 9 dan küçük olan kayıtlar gelir.
select * from tblyazar where id<=9 
select * from tblyazar where id>5 and id<12 -- id'si 5'ten büyük 12'den küçük kayıtları getirir
select * from tblyazar where id<9 or id>998 
select * from tblyazar where id id=5 or id=50 or id=500
select * from tblyazar where id=500000 -- var olmayan sorgular boş tablo döner.

---------
-- Sorgulama işlemlerinde sıralama yapmak
-- ORDER BY ile yapılır
-- Syntax: select * from [TABLE_NAME] ORDER BY [KOLON_ADI] [ASC(default) or DESC]

--ASC : default sıralama yönü -> a....z, 0...9
--DESC -> z...a, 9...0
select * from tblkonu order by ad
select * from tblkonu order by ad desc
select * from tblkonu order by id asc
select * from tblkonu order by id desc

--------
-- Tablolarımızda belli miktalarda data çekmek
-- LIMIT --> postgresql için özel ad, normalde TOP kullanılır (Oracle,MsSQL)
select * from tblyazar order by ad -- a'den z^ye kadar sırala demek
select * from tblyazar order by ad limit 5 -- a'dan z'ye sırala ilk 5'ini getir demek

----------------
-- IN ([array VALUES])
select * from tblyazar where id>50 and id<300 -- id'si 50 ile 300 arasındakileri getir. bu tamam güzel
-- peki spesifik şeyleri istersek
-- normalde
select * from tblyazar where id=4 or id=453 or id=998 or id=435 -- bu çok hantal bir sorgu
-- olması gereken:
select * from tblyazar where id in(4,453,998,435) -- olması gerkeen budur
```

- Devamını aşağıda

```sql
-- bu kısımda like ile devam edelim
-- LIKE '[ARANAN_IFADE]'
select * from tblyazar where ad = 'Rab Flux'
select * from tblyazar where ad like 'Rab Flux'
-- bir alan içinde aranılan değerin belli kurallara uyarak içeren değerleri getirmek için özel hafr kullanırız.
-- % -> joker karakter. 0...n herhangi bir değerin yerini alır.
select * from tblyazar where ad like 'Rab%' ---> Rab ile başlayan her hangibir uzunlukluktaki kelimeleri getirir.

select * from tblyazar where ad like '%rab%' --> xxxrabxxxxx, rabxxxx, xxxxxrab, rab ---> içinde rab keçen her şeyi bulur. Case Sensetive'dir Dikkat!!!!

select * from tblyazar where ad like '%ra' --> xxxra veya ra olarak gelir.


---- [_] -> alt çizgi tek karakter yerine geçer.
select * from tblyazar where ad like '____a%' --> ilk dört harfi herhangi bir değer olabilir ancak 5. harrfi a olan tüm kayıtlar.

-- Büyük küçük harf duyarlılığı olmadna arama yapmak için
-- ilike 
select * from tblyazar where ad ilike 'k____%' 
-------
--- between [DEĞER_1] and [DEĞER_2]
select * from tblyazar where id>5 and id<20 --> (5,20)
select * from tblyazar where id between 5 and 20 -->> [5,20] aslında bunu and küçük eşit büyük eşit ile de yapabiliriz ancak bu daha profesyoneldir.

--------------------
-- değerleri null yapalım önce
update tblyazar set resim=null where id in(3,34,44,35,78,98,123,543,768) 
-- bazen kayıtlarımız içinde null olan kayıtlar bulunur bu kayıtları göstermek isyeyebilir 
-- ya da onları sorgu dışında bırakmak isteyebilirz. bunun için 
-- NULL, NOT NULL, IS NULL, IS NOT NULL -->> GİBİ KAVRAMLARI KULLANIRIZ.
select  * from tblyazar where resim is null --->> null olanları getirir.
select * from tblyazar where resim is not null -->> null olmayanları getirir.
-- hem null olan kayıtlar gelsin, hem de bu alanlar null olarak gelmesin.
-- boş olan kısma bir değer atıyoruz örnek
select id, ad, biyagrafi, coalesce(resim,'https://') from tblyazar where id>2 and id<36 --- 2 ile 36 arasındaki id'lerde resim kısmı null olanlar "https://" olarak gelir.
-- buna coalesce denir : postgresql :::: mssql ->isNull, SQL -> ifNull olarak tanımlanır.


-- BİR SONRAKİ KISIMDA SAYISAL İŞLEMLERE BAKACAĞIZ.
```

- SAYISAL İŞLEMLER

```sql
--- sayısal işlemler
-- adı "a" ile başlayan kaç yazar vardır ?
select * from tblyazar where ad ilike 'a%' -->> a ile başlayan tüm adları getirir.
select count(*) from tblyazar where ad ilike 'a%' --> a ile başlayan tüm adların sayısını verir. 

--- NOT Dikkat!!! null olan alanlar count ile sayılamaz bu nedenle kayıt sayısına bakarken null olanları dikkate
-- alıp almadığınıza iyi bakınız.
select count(resim) from tblyazar --990 tane var
select count(*) from tblyazar -->> 1002 tane çıkar 

-----
-- MAX, MIN belli bir alandaki en yüksek-düşük sayısal değerleri bulmak.
select max(ucret) from tblkiralama
select min(ucret) from tblkiralama
-- AVG, ortalama
select avg(ucret) from tblkiralama
-- SUM, total
select sum(ucret) from tblkiralama
---
select concat(ad, ' - ', soyad) as adsoyad from tblogretmen -- javadaki gibi concat ile yapabiliriz.
-- LENGHT
select lenght(ad), ad, id from tblkonu order by lenght(ad)

--TRIM
select *, lenght(ad) from tblkonu
select *, length(trim(ad)) from tblkonu

-- UPPER, LOWEr
select ad from tblkonu
select UPPER(ad) from tblkonu
select Lower(ad) from tblkonu

-- REVERSE
select reserver(ad) from tblkonu
```

- Sroularda sorgu yapalım

```sql
-- Hangi kitap kaç defa kiralanmıştır?
select kitapid, count(*) from tblkiralama group by kitapid
-- hangi kitap ne kadar kazandırdı ?
select kitapid, sum(ucret) from tblkiralama group by kitapid
-- Tek seferde en çok kazan sağlayan satışlar
select kitapid, max(ucret) from tblkiralama group by kitapid

-- Gruplama işlemlerinde kısıtlama yapmak
select * from tblogretmen
insert into tblogretmen(ad,soyad) values
('demet','aktan'), ('busra','pekin'),('canan','güleç'), ('betül','ayaz') -- biraz ogretmen ekledik sorgu işlemleri için

select * from tblkitap -- kitapta yok biraz kitap ekleyelim
insert into tblkitap(ad,sayfasayisi,yayinevi,basimsayisi,basimyili) values
('Hızlı ve Yavaş Düşünme',650,'Varlık Yayınları',3,2002),
('Taşın altındaki el',344'Varlık Yayınları',3,2019),
('Siya KUĞU',511,'Varlık Yayınları',6,2011),
('Rezonans Kanunu',206,'Koridor Yayıncılık',2,2019),
('Hızlı ve Yavaş Düşünme',224,'Varlık Yayınları',3,2002),

-- daha sonra vermiş olduğum siteden tblkiralama için kayıt ekleyebiliriz 
-- örnek 1 tane
insert into tblkiralama(ucret,kitapid,ogrementid) values (336.4,10,1)
-------------
-- HAVING - group by da kısıtlama yapmak için kullanılır.
  -- where kullanamıyz çünkü kiralama_Adedi diye orjinal diye column yok.
select ogrementid, count(*) as kiralama_adedi from tblkiralama group by ogretmenid
having count(*)>10 
order by kiralama_adedi desc

-- ödeme tutarı 300₺ nin üzerinde olan kiralamarın öğretmenlere göre adedini bulunuz.
select ogretmenid, count(*) as kiralama_adedi from tblkiralama where ucret > 300 group by ogretmenid having count(*) >=7

-- Hangi yayın evinin kaç kitabı var?
select yayinevi, count(*) from tblkitap group by yayinevi
--- DISTINCT
-- Bir tabloda mükerrer olan kayıtların ayıklanarak tekilleştirilmesi gerekebilir.
-- Mesela, şuan kaç farklı yayınevi ile çalışıyoruz bilgisine ulaşmak için kullanılabilir.
select distinct(yayinevi) from tblkitap

-----------------------------------------------------
-- Tabloların birleştirilmesi
------ JOIN TABLE ------
--- bir A kitabını kiralayan öğretmenlerin listesi? TABLO
select * from tblkiralama where kitapid=5 -- öğretmenleri burdan bulabilirim.
select * from tblogretmen where id in(2,3)

-- bunu farklı yapalım
select distinct(ogretmenid) from tblkiralama where kitapid=6
-- Dikkat!! IN içerisine sadece bir değer alabilir, yani bir tabloda kısıtlama yaparak tek bir alanı liste olarak IN içerisinde verebiliriz.
-- NOT!!! genellikle iç içe sorguları kullanmayız. maliyeti çok olur. örnek olması açısında veriyorum
select * from tblogretmen where id in(select distinct(ogretmenid) from tblkiralama where kitapid=6) -- bu bize doğru sonuç verir.

-- JOIN
-- Bir biri ile ilişkili olan tabloların JOIN yaparak birleştirebilirsiniz. Bu işlemde doğru sorgular ve kısıtlarla yüksek başarımlı sonuçlar elde edebiliriz.
-- sorgu yapılacak ilk tablo ve alanlar belirlenir. [select * from tblXXX]
-- join işlemi tnaımlanır. [INNER - LEFT - RIGHT - NATURAL - CROSS] [(OUTER) - JOIN]
-- bağlantı kurulacak tablo yazılır [tblYYYY] (optional - AS kullanılabilir)
-- iki tablo arasındaki bağıntının tarifi yapılır. [ ON tblXXX.id = tblYYY.xid]
-- filtreleme işlemleri ya da gruplama işlemleri buradan itibaren yazılabilir.
select * from tblkiralama
left join tblogretmen
on tblkiralama.ogretmenid=tblogretmen.id -- ogretmen tablosunu kiralama tablosuna eklemiş oldu

-----
-- bazen tablo adları ve sorgular uzun olabilir ve gerekler alanlar var olabilir. bunları düzenleme için kısıtlamak gereklidir.
select tblkiralama.id, tblkiralama.ogretmentid, tblkiralama.kitapid,tblkiralama.ucret,
tblogretmen.id, tblogretmen.ad,tblogretmen.soyad
from tblkiralama
letf join tblogretmen
on tblkiralama.ogretmenid=tblogretmen.id
-- tabloların adları bazen çok uzuuuun olabilir. böyle durumlarda kolonları ayı ayrı belirlemek zor olacaktır. Bu neden AS kullanmak mantıklı olur.
select ki.id,ki.ogrementid, ki.kitapid, ki.ucret, og.ad, og.soyad
from tblkiralama as ki left join tbl ogretmen as og
on ki.ogrementid = og.id
-------------------------

-- 2 den fazla tabloyu join ile bağlamak.
select ki.id,ki.ogrementid, ki.kitapid, ki.ucret, og.ad, og.soyad,ktp.ad,ktp,yayinevi
from tblkiralama as ki 
left join tbl ogretmen as og on ki.ogrementid = og.id
left join tblkitap as ktp on ktp.id = ki.kitapid

--- 
-- left join right join farkı join kısmının solundaki table'a ekleme olursa left, sağına ekleme olursa right join kullanılır.
```

- KODLAMA MANTIĞI

```sql
do $$
declare -- optional - Eğer değişken tanımı yapılacak ise kullanılır
-- buraya tanımlamak istediğimiz değişkenleri yazıyoruz
begin
------ kodlama bloğu (çalıştırılacak kodlamalar burada olacak)
end; $$

--- Basit döngü ve ekrana çıktı vermek

do $$
begin
  for counter in 1..6 loop -- 1 den başlasın 6 ya kadar gitsin
     raise notice 'döngüyü çıktılamak: %', counter;
  end loop;
end; $$


-- başka bir döngü kullanalım
select * from tblokul  -- buradaki table bakalım
do $$
declare
  dongu integer;
  dongu2 integer :=40;
begin
  for sayac in 2..dongu2 loop
  insert insto tblokul(ad,adres) values(concat('Okul-',sayac), concat('Adres-',sayac));
  end loop;
end; $$

----
-- while
do $$
declare 
  degisken integer -- değişken tanımalama [DATA NAME] [DATA TYPe]
  degisken2 integer := 5; -- DEğer atama (DATA NAME) [DATA TYPE] := [VALUE]
begin
  degisken :=10;
  while degisken<20 loop
    raise notice 'Döngü devam....: %', degisken;
    degisken := degisken + 1;
  end loop;
end; $$


---- 
-- if
do $$
declare
  sayi1 integer :=5;
  sayi2 integer :=25;
begin
  if sayi1 > sayi2 then
    raise notice 'sayi 1 büyüktür';
  elseif sayi2>sayi1 then
    raise notice 'sayi 2 büyüktür';
  else
    raise notice 'iki sayı eşitti';
  end if
end;$$

--- 
-- PROCEDURE - verilen görevi yapar (void)
-- FUNCTION - verilen görevi yapar, sonuç döner. (return)
-- Normal koşullarda uygulamalarda sorgular ve DML işlemleri temel kodlamalarla yapılabilir.
-- ancak bazen daha kompleks kodlamalar yapmak zorunda kalabiliriz. Bu nedenle kodlamaların 
-- back-end tarafından çağırması gerekdir. Bu işlemleri yapabilmek için yazdığımmız kod bloklarını tekrar tekrar
-- tüketilmesi gerekecektir. Bunu yapabilmek için -Procedure ya da Function kullanmak zorundayız

- Procedure nasıl tnaımlanır?
create procedure ekranayaziyaz()
language plpgsql
as
$$
begin
  raise notice 'Selaaaam :)';
end;$$

-- Bir procedure yapısını nasıl çağrırırız ya da çalıştırırız?
call ekranayaziyaz();


--- Function
create function bulIdsiBuyukOlanTutar(findId integer)
returns double -- burada geriye dönülecek olan değer türünü tanımlıyoruz
language plpgsql
as
$$
declare
    sonuc double;
begin
  -- kod gövdesi
  return sonuc;
end;
$$


--- 
-- örnek
create function oranBul()
returns integer
language plpgsql
as
$$
declare
  enCokSatilanKitapSayisi integer;
  enAzSatilanKitapSayisi integer;
  oran integer;
begin
  enCokSatilanKitapSayisi := (select count(kitapid) as kiralamaAdedi from tblkiralama group by kitapid 
  order by count(kitapid) desc limit 1) as integer;
  enAzSatilanKitapSayisi :=  (select count(kitapid) as kiralamaAdedi from tblkiralama group by kitapid 
  order by count(kitapid) limit 1) as integer;
  oran = enAzSatilanKitapSayisi * 100 / enCokSatilanKitapSayisi;
  -- Burada bir değişkene değer atama yöntemlerinde farklı kullanımlar mevcuttur.
  -- Bu neden aşağıdaki gibi bir kullanım örpenğini görebilirsiniz.
  select count(kitapid) into enCokSatilanKitapSayisi from tblkiralama group by kitapid order by count(kitapid) desc limit 1;
  raise notice 'oran: %', oran;
  return oran;
end;
$$

-- function çağırmak, çalıştırmak
select oranBul();
```

