---
title: "Ftp Server Kurulumu - 2 : FTP Kullanıcı Yönetimi"
---

Bu yazı dizimizde Google Cloud üzerinde `vsftpd` ile FTP server kurulumunu ve yönetimini inceleyeceğiz.

[Ftp Server Kurulumu - 1 : vsftp Kurulumu ve Yapılandırılması](/2018-09-11/ftp-server-kurulumu){:target="_self"}

[Ftp Server Kurulumu - 2 : FTP Kullanıcı Yönetimi](/2018-09-15/ftp-server-kurulumu-2){:target="_self"}

## Ortak FTP Kullanıcı Grubu Oluşturma

Oluşturulacak ortak FTP dizinine birden çok kullanıcının erişimini sağlamak ve FTP kullanıcı izinlerini tek bir yapı altında toplamak için ortak bir grup oluşlturmamız gerekmektedir. 

```bash
$ sudo groupadd <group_name>
```

Herhangi bir kullanıcıyı bu gruba eklemek için aşağıdaki komutu kullanabiliriz. 

```bash
# Kullanıcıyı bir gruba ekleme
$ sudo usermod -a -G <group_name> <user_name>

# Kullanıcının birincil grubunu değiştirme
$ sudo usermod -g <group_name> <user_name>
```

## Kullanıcılar İçin Ortak Bir Dizin Oluşturma ve İzinler

Öncelikle ortak bir dizin oluşturmamız gerekmektedir. Burada örnek olarak `/var/www/public` yolu oluşturulmuştur. Dizin yolu istenilen yol ile değiştirilebilir.

```bash
$ sudo mkdir /var/www/public
```

Daha sonrasında dizinin sahibini ve grubunu değiştiriyoruz. Grup olarak daha önce oluşturduğumuz ftp grubunu vermemiz gerekmektedir.

```bash
$ sudo chown root:<ftp_group_name> /var/www/public
```

Son olarak ftp grubu için dizinin izinlerinin ayarlanması gerekmektedir.

```bash
$ sudo chmod 775 /var/www/public
```

Son durumda dizin yetkilendirmeleri aşağıdaki gibi olmalıdır:

```bash
$ ls -la /var/www/
total 12
drwxr-xr-x  3 root root     4096 Sep 15 00:53 ./
drwxr-xr-x 15 root root     4096 Sep 15 00:53 ../
drwxrwxr-x  2 root ftpgroup 4096 Sep 15 01:35 public/
```

## Kullanıcı FTP Dizini ile Ortak FTP dizinin bağlanması

Önceki yazımızda kullanıcının ev dizininde `ftp` adlı bir dizin ve dizin içinde de `files` adlı bir dizin oluşturup, ftp üzerinden işlemlerin bu `files` dizininde yapılmasını sağlamıştık. Şimdi de bu `files` dizini ile ortak `/var/www/public` dizinini birleştirelim.

Basit olarak aşağıdaki kod bu işlemi görecektir:

```bash
$ mount --bind /var/www/public/ /home/<user_name>/ftp/files
```

## Yeni FTP Kullanıcıların Eklenmesi

Yukarıdaki ayarlamalar yapıldıktan sonra yeni ftp kullanıcıları eklenmek isteniyorsa basit olarak şu yollar takip edilebilir.

### 1 - Kullanıcı eklenmesi

```bash
# Yeni kullanıcı ekleme
$ sudo adduser <username>
```

### 2 - Kullanıcıyı FTP grubuna ekleme

```bash
# Kullanıcıyı bir gruba ekleme
$ sudo usermod -a -G <group_name> <user_name>

# Kullanıcının birincil grubunu değiştirme
$ sudo usermod -g <group_name> <user_name>
```

### 3 - Kullanıcının ev dizini içinde ftp dosya yollarının oluşturulması

```bash
# Kullanıcı için ftp dizini oluşturma ve yetkilendirme
$ sudo mkdir /home/<username>/ftp
$ sudo chown nobody:nogroup /home/<username>/ftp
$ sudo chmod a-w /home/<username>/ftp

# ftp dizini içinde files dizini oluşturma
# Not : Burda herhangi bir yetkilendirme yapmıyoruz.
# Dizin birleştirmesi sonrasında ortak dizinin yetkilerini alacaktır.
$ sudo mkdir /home/<username>/ftp/files
```

### 4- Kullanıcının FTP dizinini ortak dizin ile birleştirme

```bash
$ mount --bind /var/www/public/ /home/<user_name>/ftp/files
```

### 5 - Kullanıcının vsftpd üzerinden FTP erişimine izin verme

```bash
$ sudo echo "<username>" | sudo tee -a /etc/vsftpd.userlist
$ sudo systemctl restart vsftpd
```