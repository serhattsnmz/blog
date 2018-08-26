---
title: "GPG - 1 : GPG (GnuPG) Nedir?"
---

Bu yazı dizimizde GPG’nin ne olduğundan, GPG ile şifreleme ve imzalamanın nasıl yapıldığından ve git ile çalışırken commitlerimizi nasıl GPG ile imzalayacağımızdan bahsedeceğiz.

[GPG - 1 : GPG (GnuPG) Nedir?](/2018-02-20/gpg-nedir-git-commitleri-gpg-ile-nasil-imzalanir){:target="_self"}

[GPG - 2 : Git Commit'leri GPG İle Nasıl İmzalanır?](/2018-02-21/gpg-2-git-commitleri-gpg-ile-nasil-imzalanir){:target="_self"}

## GPG Nedir?

GPG ( GnuPG — GNU Privacy Guard); GPL lisansına sahip, açık kaynak kodlu bir OpenPGP şifreleme standartıdır. OpenPGP ise, izni olmayan kişilerin görmesini istemediğiniz her türlü verinin bütünlüğünü ve güvenliğini sağlayan bir protokoldür. (Protokolün detaylarına [buradan](https://tools.ietf.org/html/rfc4880) ulaşabilirsiniz.) Basit olarak herhangi bir datanın şifrelenmesini ve çözülmesini kapsar. OpenPGP standartını oluşturan şifreleme ve anahtarlama yöntemi, 1991 yılında Philip Zimmermann tarafından geliştirilen PGP *(Pretty Good Privacy)* programına dayanır.

GPG şifrelemesinden bahsetmeden önce, klasik şifreleme ve bu şifrelemenin dezavantajından bahsetmekte fayda var.

Klasik şifreleme yönteminde, veriler şifrelenirken bir parola belirlenir ve bu parola şifrelenirken kullanılmasının yanında, şifrelenmiş datanın çözülmesinde de kullanılması gerekir. Bu yöntemle şifrelediğiniz datayı gönderdiğiniz başka bir kullanıcının bu dataya erişimini sağlamasını istiyorsanız, parolayı da bu kullanıcıya ulaştırmanız gerekmektedir. Fakat bu yöntem, eğer aranızdaki iletişimi dinleyen biri varsa (Man-in-the-middle attacks gibi) güvenilmezdir. Çünkü aradaki iletişimi çözen kimse, şifrelenmiş dataya ulaşabildiği gibi, parolaya da ulaşacak ve dataya çözebilecektir. Böyle olunca da datanın şifrelenmesi hiçbir anlam ifade etmeyecektir.

Yukarıda bahsettiğimiz sorunun aslında çok basit bir çözüm yolu var: İki tane anahtar oluşturmak. GPG tam bu anda devreye girmektedir. Bu iki tane anahtardan biri datayı şifrelemek için kullanılırken (public key), diğer anahtar, public key ile şifrelenmiş datanın çözülmesinde kullanılmaktadır (private key). Bu anahtarlardan public key, adından da anlaşılacağı üzere, herkese açık olarak yayınlanabilir, fakat private key’in size özel kalması ve kimseyle paylaşmamanız gerekmektedir. Herhangi biri size bir data göndermek istediği zaman (dosya, mail, yazı vs.) açıktan paylaştığınız public key ile bunu datayı şifreleyebilir ve siz de private key ile bu datayı çözebilirsiniz. Private key sadece sizde olduğu için, datanın sizden başkası tarafından açılmadığından emin olabilirsiniz.

<center>
    <img src="{{ site.picture_path }}/gpg-01.png" />
</center>

## Şifreleme ve İmzalama Arasındaki Farklar

Şifreleme ve imzalama birbirine karıştırılan önemli iki kavramdır. Farkları ise şunlardır :

- Şifreleme yaptığınızda, **başka birinin public key**’ini kullanırsınız ve datayı yolladığınız kişi bu datayı çözmek için **kendi private key**’ini kullanır.

- İmzalama yaptığınızda, **kendi private key**’iniz ile **kendi verinizi** imzalarsınız ve datayı yolladığınız kişi **sizin public key**’inizi kullanarak, datanın gerçekten sizden gelip gelmediğini çözümler.

Bu ayrımlar ışığında, eğer bir kişiye bir data gönderiyor ve sadece bu kişinin bu datayı aldığından emin olmak istiyorsanız ***şifreleme***, kamuya bir data paylaşıyor ve bu datayı alanların, datanın kaynağının siz olduğundan emin olmasını istiyorsanız ***imzalama*** kullanmalısınız.

## GPG Kurulumu

GPG ile ilgili tüm kütüphane GUI arayüz programlarına [buradan](https://www.gnupg.org/software/) ulaşabilirsiniz. Bunlar dışında birçok açık kaynak mail istemcisi ( *Evolution, KMail* vs.) GPG kullanarak mail alıp gönderme hizmetini vermektedir.

Birçok linux dağıtımı, içinde GPG ile birlikte gelmektedir. Sisteminizde GPG olup olmadığını kontrol etmek için şu komutu çalıştırabilirsiniz:

```
╰─λ gpg --version
```

Eğer GPG sisteminizde kurulu değilse [buradan](https://gnupg.org/download/) Linux, Windows ve MacOS için uygun sürümünü indirebilirsiniz.

## Varolan Key Kontrolü

Sisteminizde daha önceden yaratılmış bir GPG key çifti olup olmadığını kontrol etmek için aşağıdaki kodu konsol üzerinden çalıştırabilirsiniz. Eğer key çifti varsa listenelecektir.

```
╰─λ gpg --list-keys
```

## Anahtar Yaratmak ve Kullanmak

```
╰─λ gpg --gen-key

gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1

RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096

Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0

Key does not expire at all
Is this correct? (y/N) y

You need a user ID to identify your key; the software constructs the user ID
from the Real Name, Comment and Email Address in this form:
    "Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>"

Real name: Serhat Sönmez
Email address: mail@mail.com
Comment: Software Developer

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

You need a Passphrase to protect your secret key.
Enter passphrase:

gpg: gpg-agent is not available in this session
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
...
```

GPG key oluşturmak için komut satırına `gpg --gen-key` komutunu giriyoruz. Komutu çalıştırdığınızda size bazı sorular soracaktır:

- İlk sorulan soru, key yaratılırken hangi tür algoritma kullanılcağıdır. Algoritmaların detaylarına [buradan](https://tools.ietf.org/html/rfc4880#section-9.1) ulaşabilirsiniz. Default algoritma seçilip geçilebilir.

- Daha sonra key uzunluğu sorulacak. Daha güvenli olaması açısından en yüksek boyut seçilebilir.

- Üçüncü olarak, keylerin son kullanım tarihini oluşturmak istiyorsanız bunu seçebilirsiniz veya 0 seçerek son kullanım tarihi tanımlanmasını atlayabilirsiniz. Sonrasında bunu tekrar onaylamanızı isteyebilir.

- Sonraki adımda kişisel bilgilerinizi tanımlamanızı isteyecektir. Bu bilgiler birleştirilip bir User ID oluşturacak (örn: `Heinrich Heine (Der Dichter) <heinrichh@duesseldorf.de>`) ve public key içerisine eklenecektir. Bu bilgiler, public key’inizi kullanmak isteyen kişilere sizi tanıtmada yardımcı olacaktır. Bilgileri girdikten sonra onaylamanızı isteyecektir, "O" girerek onaylayabilirsiniz. User ID bilgisinin public key içinde gösterimi :

```
...
0000520 311 343 271  \0 021 001  \0 001
0000528 264   3   S   e   r   h   a   t
0000536       S 303 266   n   m   e   z
0000544       (   S   o   f   t   w   a
0000552   r   e       D   e   v   e   l
0000560   o   p   e   r   )       <   m
0000568   a   i   l   @   m   a   i   l
0000576   .   c   o   m   > 211 002   8
0000584 004 023 001 002  \0   " 005 002
...
```

- Kişisel bilgilerinizi düzgün ve eksiksiz girdikten sonra, sizden bir parola girmeniz ve doğrulamanız isteyecektir. Bu parola, kendi bilgisayarınızda depolanacak private key’in güvenliği için gereklidir. Private key kullanılırken sizden yine bu parolayı girmenizi isteyecektir. Güçlü bir parola seçmeniz güvenliğiniz açısından iyi olacaktır.

Bu bilgileri girdiyseniz, bundan sonra key’iniz oluşturulmaya başlanacaktır. Oluşturulan key tamamen random byte karakterlerinden oluşacağı ve o anda üretileceği için, key oluşturulana kadar sizden random hareket etmenizi isteyecektir (Mause hareketi, keybord ile random karakter girme veya diski kullanan herhangi bir işlem yapma).

```
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
```

Tüm bu işlemler başarıyla geçildiğinde private ve public key’leriniz `/home/<user>/.gnupg` altında oluşturulmuş olacaktır.

## GPG User ID Değiştirme

Oluşturduğunuz GPG dosyalarındaki User ID kısımlarını daha sonra değiştirebilirsiniz. Bunun için öncelikle sistemdeki key’leri listeleyip değiştirmek istediğiniz key’in ID’sini almanız gerekir.

```
╰─λ gpg --list-keys --keyid-format long
/home/serhat/.gnupg/pubring.gpg
-------------------------------
pub   4096R/705A56F8F4EE61F6 2018-02-20 [expires: 2018-04-21]
uid   Serhat Sönmez (Software Developer) <mail@mail.com>
sub   4096R/18FF5E291D61C1EB 2018-02-20 [expires: 2018-04-21]
```

Daha sonrasında `gpg --edit-key 705A56F8F4EE61F6` komutunu yazıp, `adduid` komutuyla bilgileri tekrardan girebilirsiniz.

```
╰─λ gpg --edit-key 705A56F8F4EE61F6

gpg (GnuPG) 1.4.20; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

gpg> adduid

Real Name: Yeni İsim
Email address: yenisim@github.com
Comment: GitHub key
Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit?
```

## Public Key Dosyası Oluşturmak

`gpg --list-keys --keyid-format long` komutuyla, daha önce oluşturulmuş key’ler görülebilir. Buradaki pub kısmı yanında bulunan id kısmını alıp, public key oluşturabiliriz.

```
╰─λ gpg --list-keys --keyid-format long
/home/serhat/.gnupg/pubring.gpg
-------------------------------
pub   4096R/99C0A0DD37141AF5 2018-02-20
uid   Serhat Sönmez (Software Developer) <mail@mail.com>
sub   4096R/53ECA83A41BF4A75 2018-02-20
```

`gpg --armor --export 99C0aODD37141AF5` komutuyla public key oluşturulabilir. Public key uzunluğu, oluştururken seçtiğiniz ayarlara göre değişkenlik gösterebilir. Komutun çıktısındaki `--BEGIN--` ve `--END--` blokları arasındaki kısmı kopyalayıp public key olarak kullanabilirsiniz.

```
╰─λ gpg --armor --export 99C0A0DD37141AF5
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v1
mQINBFqMG74BEACjlhbQ5bw165td72bhs57BQKxUmABII6Xge3Ib3I8QEobRkblQ
ahfxLUpRrNHS6CcELCw+huB4/cLcL9tCIshqpEfhlNzQ8rppSLAiE+EDi7CbZnp2
Vrm1XV+liWnJvu352lz2sdVkSw3PpEhtCj+ssIiVnpjkayPM17H1mrGwUpZWoGuj
s47hc79/J18kxM7gG8GW+T0IsPZwxwctbaXZdn/N4Jkch7xFndGSD44+Lcy2obW6
...
-----END PGP PUBLIC KEY BLOCK-----
```

Eğer oluşturduğunuz public key’i bir dosyada tutmak istiyorsanız `gpg -a -o <fileName> --export <KeyID>` komutunu kullanabilirsiniz. Bu komut ile, bulunduğunuz dizinde `<fileName>` kısmına yazdığınız isimle, public key dosyası oluşacaktır.

```
╰─λ gpg -a -o myPublicKey.txt --export 99C0A0DD37141AF5
```

## GPG İle Data Şifreleme ve Şifreli Dosyayı Çözme

GPG kullanılarak çok basit bir şekilde dosyaları şifreleyebilir ve çözebilirsiniz. Herhangi bir dosyayı şifrelemek için yapmanız gereken tek şey `gpg -r <KeyID> -e <dosyaAdi>` komutunu çalıştırmaktır. Buradaki -r parametresi, hangi kullanıcı ile imzalamak istiyorsak onu seçtiğimiz yerdir. Sistemimizde bulunan keyleri listeleyebilir ve bunların ID’lerinden birini şifreleme için kullanabiliriz.

```
╰─λ ls
deneme.txt

╰─λ cat deneme.txt
Serhat Sönmez

╰─λ gpg -r 705A56F8F4EE61F6 -e deneme.txt

╰─λ ls
deneme.txt  deneme.txt.gpg
```

Oluşturulan şifrelenmiş dosyayı çözmek de aynı derece basittir. `gpg -o <outputFilePath> -d <cryptedFile>` kodunu kullanarak, şifrelenmiş dosya açılabilir. Dosya çıkarılırken size, daha önce GPG key'leri oluştururken girdiğiniz parola sorulacaktır. `-o` parametresini girmeyerek, içeriğin sadece konsol üzerinden okunmasını sağlayabilirsiniz.

```
╰─λ ls
deneme.txt.gpg

╰─λ gpg -o cikti.txt -d deneme.txt.gpg

╰─λ ls
cikti.txt  deneme.txt.gpg

╰─λ cat cikti.txt
Serhat Sönmez
```

## GPG ile Data İmzalama ve İmza Kontrolü

Daha önce de bahsettiğimiz gibi, herhangi bir verinin sizden geldiğini göstermek istiyorsanız, o veriyi GPG ile imzalayabilirsiniz. Basit bir şekilde `gpg -s <file>` komutuyla imzalama yapabilirsiniz. Bu komut sonucunda `.gpg` uzantılı, imzalanmış bir dosya oluşacaktır. Bu dosya oluşturulurken sıkıştırma kullanıldığından okunabilir değildir. Okunabilir şekilde dosya imzalamak için `gpg --clearsign <file>` komutunu kullanabilirsiniz.

```
╰─λ ls
cikti.txt

╰─λ gpg --clearsign cikti.txt

╰─λ ls
cikti.txt cikti.txt.asc

╰─λ cat cikti.txt.asc
-----BEGIN PGP SIGNED MESSAGE-----
Hash: SHA1

Serhat Sönmez
-----BEGIN PGP SIGNATURE-----
Version: GnuPG v1

iQIcBAEBAgAGBQJajFYMAAoJEHBaVvj07mH2npwQAJRK1Nux8vnslz8T/4e9CYoM
kXZ5zJza91dx+xHYZO0heCZz0H+I61d0vPbQTjApwpDY4B875PyXDYnaXoCimLC2
QQ4HrcB5fTV8p8Sq3i/VydJP8HBSe7fxc7gF22sgBIzvJWmekY3TrMrzFGwSLMAy
aDi1UXCIZREpZk6sNVmxycdYUXomVHt4nLo0S8c2TyKpFhHjPw5+cpTwQy8oiqrb
41d2KVmPUkYjR7dR5FpiEwOvR19l9rFwYp1bE6KlSIEhQZhk7ADvWtBIUivjbmlF
8EaCleertI3Ig2yby6dTmZWrp/OliLweAcuQZL1EYsO5C2DFNrWHgCcg1VK/aotO
WPFjs9Q39sVycMssup1RQPVXJBML4ZM/o5B5Ds5dQsRe1fupePvQfdRssFwXfBRd
0+mrXCd0ALWoWoFsCWGsx5aqW/njcD0NJs49elbwYTUI8GG9AN7C0Jj8WqT6+VO1
b132BdA5Kz9NbYDHX/7Swi+b4qn6XiqGSWpEY1l0K5HXrs2CWr2zeLfo+gDnY350
JuHzMywGPeAnlaLCL7Ee9AhpDWm3Oo6rWHB2E5kv1bwYz+5RU6d7kjjQeHiUAvW3
CcVqx7q2p4N5g28avTqj4YnlMoK6Cu2szXNxNFszgnAXB3SZ7vOAG4Udbl9SKJV7
+SCOrlApMF9rVp8kaFos
=DogW
-----END PGP SIGNATURE-----
```

İmzalanmış dosyanın kontrolünü yapmak için `gpg --verify <file>` komutu çalıştırılabilir.

```
╰─λ gpg --verify cikti.txt.gpg

gpg: Signature made Tue 20 Feb 2018 08:05:06 PM +03 using RSA key ID F4EE61F6
gpg: Good signature from "Serhat Sönmez (Software Developer) <mail@mail.com>"
```

## GPG Key Silme

Var olan GPG anahtarlarını silmek için öncelikle secret keyi silmemiz gerekir. Daha sonrasında public keyi silerek tüm anahtarları silmiş oluruz.

```
╰─λ gpg --delete-secret-key "Kullanıcı Adı"
╰─λ gpg --delete-key "Kullanıcı Adı"
```

### Kaynaklar

- [https://www.openpgp.org/](https://www.openpgp.org/)
- [https://www.gnupg.org/](https://www.gnupg.org/)
- [https://www.gnupg.org/documentation/howtos.html](https://www.gnupg.org/documentation/howtos.html)
- [http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/](http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/)