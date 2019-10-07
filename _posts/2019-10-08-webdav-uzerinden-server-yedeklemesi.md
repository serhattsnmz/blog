---
title: "WebDAV Üzerinden Server Yedeklemesi"
---

Bu yazımızda, WebDAV üzerinden basit bir Linux server yedeklemesinin nasıl yapılacağını inceleyeceğiz.  

## WebDAV Nedir? Hangi Amaçlarla Kullanılır?

**Web Distributed Authoring and Versioning** (Web Dağıtımlı Yayın ve Sürümleme - WebDAV), World Wide Web sunucularında depolanmış belge ve dosyaları düzenleme ve yönetmede kullanıcılar arasında iş birliğini kolaylaştıran bir **Hiper Metin Aktarım Protokolü (HTTP) uzantısıdır**. WebDAV, İnternet Mühendisliği Görev Gücü (IETF) tarafından RFC 4918 ile tanımlanmıştır. 80 ve 443 bağlantı noktalarında çalışmaktadır. 

WebDAV ile **Google Drive, OneDrive, OwnCloud** gibi bulut ortamına bağlanabilir ve silme, yazma, kopyalama, taşıma gibi bir çok işlemleri yapabilir ve bunun için hiçbir uygulama yüklemeden internet tarayıcınızı kullanarak yapabilirsiniz. Hatta WebDAV adresinizi başka bir ağdaki bilgisayarınıza ağ sürücüsü olarak bağlayarak dosyalarınıza erişim sağlayabilirsiniz. WebDAV üzerindeki haberleşme HTTP üzerinden sağlanabileceği gibi, SSL sertifikasına sahip bir server üzerinde HTTPS kullanarak daha güvenli hale getirilebilir.  

<center>
    <img src="{{ site.picture_path }}/webdav.png" />
</center>

Günümüzde popüler olan bazı bulut ortamların WebDAV linkleri şöyledir:  

| Bulut Servis  | URL                                                       |
| ------------- | --------------------------------------------------------- |
| Box	        | https://dav.box.com/dav                                   |
| Yandex.Disk	| https://webdav.yandex.com                                 |
| freenetcloud	| https://webmail.freenet.de/webdav                         |
| HiDrive	    | https://webdav.hidrive.strato.com                         |
| IDrive	    | https://dav.idrivesync.com                                |
| Mail.Ru	    | https://webdav.cloud.mail.ru                              |
| Nextcloud	    | https://{host}/{path}/remote.php/dav/files/{username}     |
| ownCloud	    | https://{host}/{path}/remote.php/webdav                   |
| pCloud	    | https://webdav.pcloud.com                                 |
| Mailbox.org	| https://dav.mailbox.org/servlet/webdav.infostore/         |

## WebDAV Hesabını Lokal Disk Olarak Bağlama 

Bu yazımızda Yandex Disk servisinden edindiğimiz WebDAV hizmetini Debian temelli sunucumuzda otomatik yedekleme için kullanacağız. Öncelikle `davfs2` adlı aracı sistemimize kuracağız. davfs2, WebDAV paylaşımlarına yerel disklermiş gibi bağlanmak için kullanılan, open-source GPL lisansına sahip bir Linux aracıdır. Aracın resmi web sitesine [buradan](http://savannah.nongnu.org/projects/davfs2) ulaşabilirsiniz.  

- Paket yöneticisi ile `davfs2` adlı aracı kuruyoruz.

```
╰─λ apt install davfs2
```

- Daha sonrasında `/backup` adlı bir dizin oluşturup, Yandex Disk hesabımızın kullanıcı maili ve parolasıyla lokal diskimizi bağlıyoruz.

```
╰─λ mkdir /backup
╰─λ mount.davfs -o username=<yandex mail> https://webdav.yandex.ru /backup
```

Son komutu çalıştıdığımızda bizden hesabımızın parolasını isteyecektir. Parolayı girdikten sonra ilgili dizin ile Yandex Disk hesabımızın bağlantısı kurulmuş olacak. Artık bu dizin içinde yapılan her işlem, otomatik olarak bağlı olduğu WebDAV servisi ile senkronize olacaktır. 

## Yedekleme Script Dosyasının Yazılması

Otomatik yedek alma işlemlerini yapması için sistemimizde bir script dosyası oluşturmamız gerekmektedir. Bu dosya çalıştırıldığında yapmasını istediğimiz temel işlemler şunlardır:  

- Belli bir sayıda yedeği tutarak diğer eski yedek alınacak dizinleri sil
- Yeni bir yedek dizini oluştur
- Yedek alınmasını istediğimiz dizini zip olarak sıkıştırıp `/backup` dizinine at

Burada istenilirse dosyalar direk olarak `cp <dosya yolu> <kopyalama yolu>` komutu kullanılarak, sıkıştırılmadan da yedeklenebilir. Fakat yedek alınacak dosyaların boyutu büyükse hız açısından sıkıştırıp yedeklemek daha uygundur.  

Yedek almayı istediğimiz dizinin yolunu `/important_files`, WebDAV servisini bağladığımız dizinin de `/backup` olduğunu varsayarsak, script dosyamızın içeriği şöyle olacaktır:

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
INCLUDE_DIRECTORY_COUNT=$(( $TOTAL_DIRECTORY_COUNT - $EXCLUDE_DIRECTORY_COUNT ))
ls $BACKUP_PATH | sort -nr | tail -n $INCLUDE_DIRECTORY_COUNT | xargs -i rm -rf $BACKUP_PATH/{}

# YEDEK ALINACAK DOSYALARI SIKIŞTIRIP YEDEKLE
CURRENT_DATE=`date +%F__%H-%M-%S`
mkdir $BACKUP_PATH/$CURRENT_DATE
zip -s 200m -r $BACKUP_PATH/$CURRENT_DATE/backup.zip $FILES_PATH
```

Dosyayı kaydedip çıktıktan sonra, dosyaya çalıştırma izni tanımlıyoruz.

```
╰─λ chmod a+x /auto-backup.sh
```

Yukarıdaki script dosyası çalıştırıldığında, `/backup` dizini içinde daha önceden oluşturulmuş yedek dizinlerinden en son alınan 2 tanesi hariç kalanları silecektir. Daha sonrasında `2019-10-07__23-00-00` formatında, o anki tarih ve saate göre bir dizin oluşturup içine yedekleme yapacaktır. Zip komutu çalıştırıldığında istenilirse yukarıdaki gibi `-s` parametresiyle sıkıştırma işlemi verilen limite göre birden fazla dosyaya ayrılabilir.  

Script dosyası çalıştırıldıktan sonra, alınan yedek dosyalarının ağaç görünümü şöyle olacaktır:

```
╰─λ tree /backup
/backup
├── 2019-10-08__00-40-20
│   ├── backup.z01
│   └── backup.zip
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

Farklı zaman ayarlamaları için [şu linkten](https://crontab.guru/) cron job zaman ayarlamarının nasıl yapılacağı ile ilgili bilgi edinebilirsiniz.

### Kaynaklar

- [https://en.wikipedia.org/wiki/WebDAV](https://en.wikipedia.org/wiki/WebDAV)
- [https://community.cryptomator.org/t/webdav-urls-of-common-cloud-storage-services/75](https://community.cryptomator.org/t/webdav-urls-of-common-cloud-storage-services/75)