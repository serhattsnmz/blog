---
title: "Principle of Least Privilege (PoLP) Nedir?"
---

Principle of Least Privilege (PoLP); bir kullanıcının, bir programın veya bir işlemin sadece kendi işini yapabileceği kadar minimum yetkiye sahip olması prensibidir. Siber güvenliğin en önemli kavramlarından biri olan bu prensip, özellikle pratikte kullanıcıların sahip olmaması gereken üst düzey yetkilerinin sınırlandırılmasını şart koşar.  

<center>
    <img src="{{ site.picture_path }}/polp_picture.png" width="100%" />
</center>

PoLP, yazılımsal veya sistem içinde uygulanabileceği gibi, bir şirketin her departmanında uygulanabilirliği olan bir prensiptir. Örneğin bir IT firmasında yazılım ile ilgilenen bir ekibin, sistem ile ilgilenen başka bir ekibin yetkilerine sahip olmasına gerek yoktur ve bu durum bir güvenlik zaafiyetidir. Bununla birlikte sistem ile ilgilenen ekibin de insan kaynaklarında bulunan, şirket çalışanlarının hassas verilerine ulaşımlarının olmaması gerekir.  

Bir IT ortamında Least Privilege prensibine bağlı kalmak, saldırganların kritik sistemlere ve hassas verilere erişim riskini azaltır. Eğer saldırgan sisteme sızarsa, sisteme yayılmasını kısıtlar ve sistemin güvenliğini önemli ölçüde arttırır.  

## Principle of Least Privilege'ın Önemi

Bu prensibin önemi şöyle sıralayabiliriz:  

- **Daha iyi güvenlik:** Bir sistem ne kadar güvenlik önlemleriyle donatılırsa donatılsın veya bir şirket ne kadar güvenliğe yatırım yaparsa yapsın, _insan faktörü veri güvenliğinin her zaman en zayıf halkasıdır._ Bu nedenle hassas verilere ulaşan insan sayısı ne kadar kısıtlanırsa, verilerin güvenliği o denli arttırılmış olur.  

- **Saldırı yüzeyini asgariye indirme:** Bir şirket içindeki insanların veya bir sistem üzerindeki programların/kullanıcıların yetkileri ne kadar kısıtlıysa, hassas verilere ulaşmak isteyen kötü niyetli kişilerin saldırması gereken insanlar veya hacklemesi gereken programlar/kullanıcılar o kadar az olacaktır. Bu durum da saldırganların işlerini zorlaştıracak ve korunması gereken yetkili insan/kullanıcı sayısını azaltacaktır.

- **Malware yayılımının kısıtlanması:** PoLP ile desteklenmiş sistemlere bulaşan zararlı yazılımlar, olasılık olarak daha çok sınırlı yetkilere sahip olacağından, vereceği zararlar da minimal olacaktır.

- **Daha stabil sistemler:** Saldırılarla gelen zararlar dışında, PoLP ile desteklenen sistemlerde kullanıcıların yapacağı hatalar da sınırlı olacaktır. Bu durum sistemlerin stabilliğini arttıracak ve hata oranını azaltacaktır.

- **Denetim zorluğunun azalması:** PoLP ile desteklenen sistemlerin denetimi, her kullanıcının üst düzey yetkiye sahip olduğu sistemlerin denetiminden daha kolay ve maliyetsiz olacaktır.

## PoLP Entegrasyon Adımları

- **Ayrıcalık denetimi yapın.** Varolan tüm hesapların, uygulamaların ve şahışların yetkilerini kontrol edin ve hepsinin sadece sahip olması gereken yetkilere sahip olduğuna emin olun.

- **Her kullanıcıyı minimum yetkiyle oluşturun.** Varsayılan olarak her kullanıcı yetkisinin minimum olması gerekir. Eğer gerekliyse daha sonra yetki yükseltmesi yapılabilir.

- **Geçici durumlarda geçici yetki yükseltmesi yapın.** Eğer bir kullanıcının yetkisi dışında bir işlem yapması gerekiyor fakat yetkisinin kalıcı olarak yükseltilmesine gerek yoksa bir zaman aralığında yükseltme yapın ve işin sonunda eski yetkilerine geri döndürün. 

- **Sistemleri ve ağları bölümlere ayırın.** Veriler barındıran her sistemi ve ağı, ayrıcalıklarına göre kümelendirin ve her kümeye sadece erişmesi gereken kullanıcıları atayın.

- **Yüksek erişime sahip yetkileri dağıtmakta dikkatli davranın.** Bir sistem üzerindeki superuser yetkileri gibi yüksek erişime sahip yetkileri dağıtırken gerçekten buna gerek olup olmadığını sorgulayın. Eğer geçici yetkilendirme ile sorun çözülecekse geçici yetkilendirme yapın. Yüksek erişime sahip insan sayısını asgari düzeyde tutun.

- **Önemli eylemleri izlenebilir hale getirin.** Hassas verilere erişim gibi önemli eylemleri izlenebilir hale getirmek, yetkisiz kullanıcı eylemlerini tespit etmek ve bunları sınırlandırmak için daha kolay bir yöntemdir.

- **Yetki kontrollerini düzenli hale getirin.** Düzenli kontroller, önceden bazı yetkileri olan fakat artık sahip olmasına gerek duyulmayan eski kullanıcıların bu yetkilerini tespit etmek ve kısıtlamak açısından önemlidir.

### Kaynaklar

- [https://www.beyondtrust.com/blog/entry/what-is-least-privilege](https://www.beyondtrust.com/blog/entry/what-is-least-privilege)
- [https://en.wikipedia.org/wiki/Principle_of_least_privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege)
- [https://searchsecurity.techtarget.com/definition/principle-of-least-privilege-POLP](https://searchsecurity.techtarget.com/definition/principle-of-least-privilege-POLP)