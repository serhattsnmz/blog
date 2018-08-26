---
title: "GPG - 2 : Git Commit'leri GPG İle Nasıl İmzalanır?"
---

Bu yazı dizimizde GPG’nin ne olduğundan, GPG ile şifreleme ve imzalamanın nasıl yapıldığından ve git ile çalışırken commitlerimizi nasıl GPG ile imzalayacağımızdan bahsedeceğiz.

[GPG - 1 : GPG (GnuPG) Nedir?](/2018-02-20/gpg-nedir-git-commitleri-gpg-ile-nasil-imzalanir){:target="_self"}

[GPG - 2 : Git Commit'leri GPG İle Nasıl İmzalanır?](/2018-02-21/gpg-2-git-commitleri-gpg-ile-nasil-imzalanir){:target="_self"}

## Git Kullanırken Neden GPG İmzalama Yapmalıyım?
GPG ile imzalanan commit’ler, sizin kimliğinizi doğrulamaktadır. GPG key’ler kişiye özel olduğu için, gelen commit’lerin gerçekten o şahıstan geldiğini doğrulamış olursunuz.

Peki imzalama hangi aşamada gerçekleşir? GPG imzalaması, *commit ekleme* aşamasında gerçekleştirilir ve *lokalde*dir. İmzalanmış commit’ler uzak server üzerine yüklenirken, daha önce uzak server üzerine yüklediğiniz GPG public key ile doğrulama gerçekleşir ve commit *verified* olarak işaretlenir.

## GitHub üzerine GPG Key Ekleme

- Github sayfasından `Settings > SSH ve GPG Keys` sekmesine gelerek `New GPG Key` butonuna tıklayınız.

- Açılan pencereden, daha önce oluşturmuş olduğunuz public key’inizi kopyalayıp yapıştırınız.

GPG key’iniz eklenecektir. Eğer GPG key oluştururken girmiş olduğunuz mail adresi Github hesabınızda tanımlı değilse, `unverified` uyarısıyla karşılacaksınız. Bunu aşmak için, ilgili mail adresini GitHub üzerine tanımlayabilir veya tanımlı mail adresinizle yeni bir GPG key oluşturup sisteme yükleyebilirsiniz.

## GitLab üzerine GPG Key Ekleme

- Gitlab sayfasından `Settings > GPG Keys` sekmesine geliniz.

- Açılan pencerede direk olarak public key’inizi yapıştırınız.

GPG key’iniz eklenecektir. Github’ta olduğu gibi, burada da mail adresiniz hesabınıza tanımlı değilse `unverified` uyarısı verecektir.

## Git Üzerine Yollanılan Commitleri İmzalama

Öncelikle Git üzerine, oluşturdumuz GPG private key’ini tanıtmamız gerekiyor. Bunun için `gpg --list-secret-keys --keyid-format long` komutuyla sistemimiz üzerinde olan keylerin listesine ulaşıp, ilgili key’in ID’sini alıyoruz. Daha sonra `git config --global user.signingkey <ID>` komutuyla, ilgili GPG key’ini git üzerinde tanıtmış oluyoruz.

```
╰─λ gpg --list-secret-keys --keyid-format LONG
/home/serhat/.gnupg/secring.gpg
-------------------------------
sec   4096R/705A56F8F4EE61F6 2018-02-20 [expires: 2018-04-21]
uid   Serhat Sönmez (Software Developer) <mail@mail.com>
ssb   4096R/18FF5E291D61C1EB 2018-02-20

╰─λ git config --global user.signingkey 705A56F8F4EE61F6
```

Git üzerinde GPG tanımlaması yapıldıktan sonra, atacağımız her commit’i bu key ile imzalamamız gerekmektedir. Bunun için `-S` parametresini kullanacağız. Komutu çalıştırdığımda bizden, private key çözümlemesi için daha önce girdiğimiz parolayı isteyecektir. Parolayı girdikten sonra commit’imiz yapılmış olacaktır.

```
╰─λ git commit -S -m <commit mesaji>
╰─λ git push
```

Eğer her commit’inizin otomatik olarak GPG ile gönderilmesini istiyorsanız, global veya local ayarlardan `gpgsign` özelliğini aktif hale getirebilirsiniz.

```
# Global düzenleme
╰─λ git config --global commit.gpgsign true

# Lokal düzenleme
╰─λ git config commit.gpgsign true

# Eski haline getirme
╰─λ git config --global commit.gpgsign false
╰─λ git config commit.gpgsign false
```

### Kaynaklar

- [https://www.openpgp.org/](https://www.openpgp.org/)
- [https://www.gnupg.org/](https://www.gnupg.org/)
- [https://www.gnupg.org/documentation/howtos.html](https://www.gnupg.org/documentation/howtos.html)
- [http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/](http://irtfweb.ifa.hawaii.edu/~lockhart/gpg/)