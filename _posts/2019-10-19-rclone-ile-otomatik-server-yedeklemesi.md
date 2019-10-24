---
title: "Rclone ile Otomatik Server Yedeklemesi"
---

Bu yazımızda [Rclone](https://rclone.org/) aracını kullanarak, server üzerinde otomatik nasıl yedek alınabileceğini inceleyeceğiz.

## Rclone

Rclone, bir çok bulut hizmet sağlayıcıyı destekleyen ve komut satırı üzerinden bu servislere otomatik senkronizasyon yapabilen, açık kaynak kodlu bir programdır. Desteklediği **43 bulut hizmetinin** tam listesine, [buradan](https://rclone.org/) ulaşabilirsiniz.

<center>
    <img src="{{ site.picture_path }}/backup.jpg" />
</center>

Bu yazımızda, ***Yandex Disk*** üzerinden edindiğimiz bulut hizmetini kullanarak, ***Debian*** temelli sunucumuz üzerinde **Rclone ayarımızı** yapacak, **otomatik yedekleme scriptimizi** yazacak ve **cronjob** ile bu script dosyamızın belli aralıklarla çalışıp yedek almasını sağlayacağız.

Öncelikle aşağıdaki komutu kullanarak Rclone uygulamasını sunucumuza kuralım.

```
╰─λ curl https://rclone.org/install.sh | sudo bash
```

## Rclone Config Dosyasının Oluşturulması

Rclone uygulamasının ayarlamarını yapabilmek için basitçe konsol üzerinden aşağıdaki komutu çalıştırmamız yeterlidir:

```
╰─λ rclone config
```

Yukarıdaki komutu çalıştırdığımızda, configuration dosyasını oluşturmak için bizden bazı bilgiler isteyecektir:

- İlk kurulum yaptığımızdan, sistem üzerinde herhangi bir configuration dosyası olmadığını ve bizden ne yapmamızı istediğimizi soracaktır. `n` yanıtını vererek, yeni bir remote oluşturacağımızı söylüyoruz.

```
╰─λ rclone config
2019/10/19 17:20:24 NOTICE: Config file "/root/.config/rclone/rclone.conf" not found - using defaults
No remotes found - make a new one
n) New remote
s) Set configuration password
q) Quit config
n/s/q> n
```

- Daha sonrasında oluşturacağımız remote adresi için bir isim tanımlaması ve hangi bulut sunucu için remote ayarı yaptığımızı belirtiyoruz. Burada remote adresi ismi için `yandex-yedek`, remote ayarları için `yandex` yazarak devam ediyoruz. Başka bir bulut sunucusu için ayarlama yapacaksak, çıkan listeden ilgili remote suncuyu seçmemiz gerekiyor.

```
name> yandex-yedek

Type of storage to configure.
Enter a string value. Press Enter for the default ("").
Choose a number from below, or type in your own value
...
29 / Yandex Disk
   \ "yandex"
...

Storage> yandex
```

- Yandex bulut hizmetini seçtikten sonra, Yandex API hizmetinden manuel alabileceğimiz `clint_id` ve `client_secret` değerlerini soracaktır. Fakat biz burada bunları boş bırakıp, daha sonra Rclone ile otomatik alınmasını sağlayacağız. Bu yüzden her iki adımı da Enter ile geçelim.

```
** See help for yandex backend at: https://rclone.org/yandex/ **

Yandex Client Id
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_id>

Yandex Client Secret
Leave blank normally.
Enter a string value. Press Enter for the default ("").
client_secret>
```

- Sonrasında ileri ayarlamalar yapmak isteyip istemediğimizi soracaktır. Buna da `n` yazarak geçelim.

```
Edit advanced config? (y/n)
y) Yes
n) No
y/n> n
```

- Geldik en önemli adıma. Bu adımda Rclone uygulaması için Yandex hesabımızdan API bilgilerini almamız gerekiyor. Buradaki 1. seçenek, bilgisayar üzerinden tarayıcı açarak Yandex izinleri almamızı sağlayacaktır. Fakat server üzerinde kurulum yaparken tarayıcı açamayacağımızdan dolayı 2. seçeneği seçip, başka bir bilgisayardan bu işlemi yapamız gerekiyor.

```
Remote config
Use auto config?
 * Say Y if not sure
 * Say N if you are working on a remote or headless machine
y) Yes
n) No
y/n> n

For this to work, you will need rclone available on a machine that has a web browser available.
Execute the following on your machine (same rclone version recommended) :
        rclone authorize "yandex"
Then paste the result below:
result>
```

- Seçeniğe `n` cevabı verdiğimizde, web tarayıcısı olan bir bilgisayarda girmemiz gereken bir komut verilecektir. Herhangi bir bilgisayara Rclone uygulamasını indirerek (Window, Mac veya Linux olabilir) `rclone authorize "yandex"` kodunu çalıştırmamız, web tarayıcıdan açılan pencerede ilgili yandex hesabımızdan izin vermemiz ve komut satırında gelen kodu, server üzerindeki komut satırına yapıştırmamız gerekmektedir.

```
╰─λ rclone authorize "yandex"
If your browser doesn't open automatically go to the following link: http://127.0.0.1:53682/auth?state=02dc361939035fac2dfdbdb2ab87419c
Log in and authorize rclone for access
Waiting for code...
Got code
Paste the following into your remote machine --->
{"access_token":"xxxxxxxxxxxxxxx","token_type":"bearer","refresh_token":"xxxxxxxxxxx","expiry":"2020-10-18T18:44:31.6284049+03:00"}
<---End paste
```

- Komut sonunda çıkan `{"access_token":"xxxxxxxxxxxxxxx","token_type":"bearer","refresh_token":"xxxxxxxxxxx","expiry":"2020-10-18T18:44:31.6284049+03:00"}` token kısmını kopyalayıp, server üzerindeki konsola yapıştırıyoruz.

```
result> {"access_token":"xxxxxxxxxxxxxxx","token_type":"bearer","refresh_token":"xxxxxxxxxxx","expiry":"2020-10-18T18:44:31.6284049+03:00"}

--------------------
[yandex-yedek]
type = yandex
token = {"access_token":"xxxxxxxxxxxxxxx","token_type":"bearer","refresh_token":"xxxxxxxxxxx","expiry":"2020-10-18T18:44:31.6284049+03:00"}
--------------------
y) Yes this is OK
e) Edit this remote
d) Delete this remote

y/e/d> y

Current remotes:

Name                 Type
====                 ====
yandex               yandex

e) Edit existing remote
n) New remote
d) Delete remote
r) Rename remote
c) Copy remote
s) Set configuration password
q) Quit config
e/n/d/r/c/s/q> q
```

- Yapıştırma işleminden sonra girilen bilgileri onaylamak için `y`, en son olarak da komutları sonlandırmak ve ayar dosyalarını oluşturmak için `q` harflerini girip ayarlamaları bitiriyoruz. İstersek `rclone about yandex-yedek:` komutu ile o an bağlı olan bulut hizmetinin depolaması ile ilgili bilgi edinebiliriz:

```
╰─λ rclone about yandex-yedek:
2019/10/19 17:53:09 Automatically upgraded OAuth config.
Total:   20G
Used:    0
Free:    20G
```

Rclone ile yapılabilecek tüm işlemlerin listesine [buradan](https://rclone.org/commands/) ulaşabilirsiniz.

## Yedekleme Script Dosyasının Yazılması

Otomatik yedek alma işlemlerini yapması için sistemimizde bir script dosyası oluşturmamız gerekmektedir. Bu dosya çalıştırıldığında yapmasını istediğimiz temel işlemler şunlardır:  

- Belli bir sayıda yedeği tutarak diğer eski yedek alınacak dizinleri sil.
- Yeni bir yedek dizini oluştur.
- Yedek alınmasını istediğimiz dizini zip olarak sıkıştırıp `/backup` dizinine at.
- Rclone kullanarak yedek alınmış dosyaları, bulut sunucu ile eşitle.

Yedek almayı istediğimiz dizinin yolunu `/important_files`, yedek alacağımız dizinin yolunun da `/backup` olduğunu varsayarsak, script dosyamızın içeriği şöyle olacaktır:

```
╰─λ vi /auto-backup.sh
```

```sh
#!/bin/sh

BACKUP_PATH="/backup"
FILES_PATH="/important_files"
EXCLUDE_DIRECTORY_COUNT=2

# SON İKİ YEDEK DOSYASINI TUTUP DİĞERLERİNİ SİL
TOTAL_DIRECTORY_COUNT=`ls $BACKUP_PATH | wc -l`
if [ $TOTAL_DIRECTORY_COUNT -gt $EXCLUDE_DIRECTORY_COUNT ]
then
    INCLUDE_DIRECTORY_COUNT=$(( $TOTAL_DIRECTORY_COUNT - $EXCLUDE_DIRECTORY_COUNT ))
    ls $BACKUP_PATH | sort -nr | tail -n $INCLUDE_DIRECTORY_COUNT | xargs -i rm -rf $BACKUP_PATH/{}
fi

# YEDEK ALINACAK DOSYALARI SIKIŞTIRIP YEDEKLE
CURRENT_DATE=`date +%F__%H-%M-%S`
mkdir $BACKUP_PATH/$CURRENT_DATE
zip -s 200m -r $BACKUP_PATH/$CURRENT_DATE/backup.zip $FILES_PATH

# DOSYALARI BULUT İLE EŞİTLE
rclone sync -P $BACKUP_PATH yandex-yedek:backup

# YANDEX ÇÖP KUTUSUNU BOŞALT
rclone cleanup yandex-yedek:
```

Dosyayı kaydedip çıktıktan sonra, dosyaya çalıştırma izni tanımlıyoruz.

```
╰─λ chmod a+x /auto-backup.sh
```

Yukarıdaki script dosyası çalıştırıldığında, `/backup` dizini içinde daha önceden oluşturulmuş yedek dizinlerinden en son alınan 2 tanesi hariç kalanları silecektir. Daha sonrasında `2019-10-07__23-00-00` formatında, o anki tarih ve saate göre bir dizin oluşturup içine yedekleme yapacaktır. Zip komutu çalıştırıldığında istenilirse yukarıdaki gibi `-s` parametresiyle sıkıştırma işlemi verilen limite göre birden fazla dosyaya ayrılabilir.  

**NOT :** Eğer dosyaların boyutu çok büyükse (>500mb) Rclone için timeout süresi arttırılmalıdır.

Script dosyası çalıştırıldıktan sonra, yedek dosyaları `/backup` dizini içinde oluşturulacak ve yedeklemesi yapılacaktır. `/backup` dizininin ağaç görünümü de aşağıdaki gibi olacaktır.

```
╰─λ tree /backup
/backup
├── 2019-10-08__00-40-20
│   ├── backup.z01
│   └── backup.zip
```

Eğer yedeklerin dizin içinde parçalı zip halinde değil de, tek bir zip dosyası halinde yedeklenmesini istiyorsak, script dosyamızı aşağıdaki gibi değiştirebiliriz.

```sh
#!/bin/sh

BACKUP_PATH="/backup"
FILES_PATH="/important_files"
EXCLUDE_DIRECTORY_COUNT=2

# SON İKİ YEDEK DOSYASINI TUTUP DİĞERLERİNİ SİL
TOTAL_DIRECTORY_COUNT=`ls $BACKUP_PATH | wc -l`
if [ $TOTAL_DIRECTORY_COUNT -gt $EXCLUDE_DIRECTORY_COUNT ]
then
    INCLUDE_DIRECTORY_COUNT=$(( $TOTAL_DIRECTORY_COUNT - $EXCLUDE_DIRECTORY_COUNT ))
    ls $BACKUP_PATH | sort -nr | tail -n $INCLUDE_DIRECTORY_COUNT | xargs -i rm -rf $BACKUP_PATH/{}
fi

# YEDEK ALINACAK DOSYALARI SIKIŞTIRIP YEDEKLE
CURRENT_DATE=`date +%F__%H-%M-%S`
zip -r $BACKUP_PATH/$CURRENT_DATE.zip $FILES_PATH

# DOSYALARI BULUT İLE EŞİTLE
rclone sync -P $BACKUP_PATH yandex-yedek:backup

# YANDEX ÇÖP KUTUSUNU BOŞALT
rclone cleanup yandex-yedek:
```

Bu şekilde alınan yedek dosyalarının ağaç görünümü de aşağıdaki gibi olacaktır:

```
/backup
├── 2019-10-09__16-19-56.zip
└── 2019-10-09__16-21-20.zip
```

## Otomatik Yedekleme İçin Cron Job Ayarlaması

Linux sistemimizde, biraz önce oluşturduğumuz script dosyasının düzenli araklıklarla otomatik olarak çalışmasını ve yedek almasını istiyorsak, bu script dosyasını cron job olarak tanımlamamız gerekmektedir. Cron job tanımlamak için basit olarak aşağıdaki komut yazılabilir:

```
╰─λ crontab -e
```

Eğer script dosyamızın her gün saat 01:00'de çalışmasını istiyorsak, açılan dosyanın en altına şu satırı ekleyebiliriz:

```
0 1 * * * /auto-backup.sh
```

Farklı zaman ayarlamaları için [şu linkten](https://crontab.guru/) cron job zaman ayarlamalarının nasıl yapılacağı ile ilgili bilgi edinebilirsiniz.

### Kaynaklar

- [https://rclone.org/](https://rclone.org/)