# SQL

- pdadmin ile bağlandığımız postgre sql'e sağ tıklayıp "new database" diyoruz.
- oluşturduğumuz database ismine sağ tıkladığımızda query tool seçeneceğini seçiyoruz.
- Karşımıza bir alan gelecek ve sql komutlarını burada yazabiliriz.

---

### SQL Komutları

```other
--- Structued Query Langugeage ---
--- SQL tüm Reational Database'lerde ortak dildir.
--- Tek tük değişiklerle karşımıza çıkar
--- Burada bilmeniz gereken iki kavram vardır.
--- DML - DDL ->>> araştırmanızı tavsiye ederim.

--- CREATE
--- Herhangi bir veritabanı nesnesini oluşturmak için kullanılır.
--- CREATE [TYPE] [NAME] (
---  [COLUMN_NAME] [DATA TYPE] [PROP],
---  [COLUMN_NAME] [DATA TYPE] [PROP] 
---)
--- serial(bigserial) -> Bu otomatik sayı üretmek için sequence oluşturur 
--- ve yeni bir kayıt eklenmeden önce çalışarak sıradaki sayıyı iletir.

-- iki cizgi yorum satırını belirtir.
-- maouse ile seçtiğimiz alanı start dersek ilgili alan çalışır.
```

- Örnek

```sql
create table tbl_musteri(
 id serial primary key,
  ad varchar(100),
  telefon varchar(20),
  kullaniciadi varchar(64),
  sifre varchar(64),
  createat bigint, -- oluşturulma zamanı
  updateat bigint, -- güncellenme zamanı
  state int -- 0-> pasif, 1-> aktif, 2-> silinmiş
)
-- Bir tabloya sorgu çekmek ve kayıtları göstermek için
-- SELECT komutu
select * from tbl_musteri

-- INSERT INTO komutu
insert into tbl_musteri(ad,telefon) --ilgili columlara ekleme 
values('Mert Hoca','0 555 444 2222')
-- yukarıdaki iki satırı seçip "play" tuşuna basarsak ekleme işlemi yapılır.
insert into tbl_musteri(ad,telefon)
values('Mert Ertugral','2131231'),
('Ayşe','32423423') -- bu şekilde iki kayıt birden ekleyebiliriz.

-- DELETE komutu
delete from tbl_musteri where id=2 --silme işlemini yapar.

-- Yukarıda table oluştururken oluşturulma zamanı ve güncellen zamanını verdik  ama ekleme yapmadık
-- table oluştururken default bir değer atamamız lazım .aşağaıdaki gibi

  -- Biz en başta table oluşturduk yeniden oluşturmak için önceki table silmemiz lazım
  -- DROP TABLE 
  drop table tbl_musteri
  
create table tbl_musteri(
  id serial primary key,
  ad varchar(100),
  telefon varchar(20),
  kullaniciadi varchar(64) not null,
  sifre varchar(64) not null, --oluşturma yaparken boş kalmaması için 
  createat bigint default extract (epoch from now())*1000
  updateat bigint default extract(epoch from now())*1000 -- sistemdeki zamanı alır.
  updateat bigint, -- güncellenme zamanı
  state int -- 0-> pasif, 1-> aktif, 2-> silinmiş
)

-- null olmama durumu da table' a ekledik -> not null o zaman yeni eklemeyi yapalım
-- önce mevcut table silelim
drop table tbl_musteri

-- yeni table ekleyelim 
create table tbl_musteri(
  id serial primary key,
  ad varchar(100),
  telefon varchar(20),
  kullaniciadi varchar(64) not null,
  sifre varchar(64) not null, --oluşturma yaparken boş kalmaması için 
  createat bigint default extract (epoch from now())*1000
  updateat bigint default extract(epoch from now())*1000 -- sistemdeki zamanı alır.
  updateat bigint, -- güncellenme zamanı
  state int -- 0-> pasif, 1-> aktif, 2-> silinmiş
)

insert into tbl_musteri(ad,telefon,kullaniciadi,sifre)
values('Mert Ertugral','2131231','superadmin','12334'),
('Ayşe','32423423','tester','1231')
```

