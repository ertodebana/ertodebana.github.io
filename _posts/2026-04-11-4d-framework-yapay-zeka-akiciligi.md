---
title: "4D Framework: Yapay Zeka Akıcılığı İçin Bir Pusula"
date: 2026-04-10 10:00:00 +0300
lang: tr
locale: tr-TR
page_id: 4d-framework-ai-fluency
permalink: /posts/4d-framework-yapay-zeka-akiciligi/
categories: [AI, Education]
tags: [ai-fluency, 4d-framework, ai-education, ai-literacy, prompting, claude, delegation, discernment, diligence]
description: "Rick Dakan ve Joseph Feller'ın geliştirdiği 4D Framework'ü ve yapay zeka akıcılığının neden sadece prompt yazmaktan fazlası olduğunu anlatıyorum."
header:
  overlay_image: "https://images.unsplash.com/photo-1500964757637-c85e8a162699?ixlib=rb-4.1.0&auto=format&fit=crop&w=1200&q=80"
  overlay_filter: 0.4
  caption: "Fotoğraf: [Simon / Unsplash](https://unsplash.com/photos/twukN12EN7c)"
---

## Herkes "AI Kullanıyor" — Kaçı Gerçekten Kullanıyor?

Şöyle bir şey var: git'i *bilen* insanlarla, git'i *kullanan* insanlar arasında derin bir mesafe var. İlk grup `git add . && git commit -m "düzeltme"` yazıp iter. İkinci grup ne zaman rebase, ne zaman merge yapacağını biliyor; değişim grafiğini kafasında taşıyor; commit mesajını belge gibi yazıyor. Her iki grup da "git biliyor" der ama operasyon bambaşka.

Yapay zeka kullanımı da tam bu noktada benzer bir ayrışma yaşıyor. Büyük çoğunluk bir şeyler soruyor, bir şeyler alıyor, bir şeyler yapıştırıyor. Ama bunun ötesinde — modeli ne zaman devreye alacağını, nasıl yönlendireceğini, çıktısını nasıl değerlendireceğini ve bu süreçten kimin hesap vereceğini bilen insan sayısı çok daha az.

İşte tam bu fark için bir çerçeve var: **4D Framework**. Ringling College of Art and Design'dan Prof. Rick Dakan ve University College Cork'tan Prof. Joseph Feller tarafından geliştirilen bu model, yapay zeka akıcılığını dört temel yeterlilikle açıklıyor.

## Dört D, Bir Pusula

Framework'ün dört bileşeni şunlar: **Delegation** (Delegasyon), **Description** (Tanımlama), **Discernment** (Sorgulama) ve **Diligence** (Özen). Dört soyut isim ard arda sıralandığında "yine bir framework" hissi uyansa da içerikleri oldukça işlevsel.

## Delegation: Kimi Ne Yaptırıyorsunuz?

İlk D, işi *kime* verdiğinizle ilgili. Yapay zekaya ne yaptırmak mantıklı, ne yaptırmak mantıksız? Hedefiniz ne, AI'ın gerçek kapasitesi nerede bitiyor?

Delegasyonu en iyi şöyle düşünebilirsiniz: bir yazılım mühendisine "bu fonksiyonu refactor et" demek ile "commit mesajını düzelt, deploy et" demek arasındaki fark. Her ikisi de "bir şey yap" ama birincisinde bağımsız yargıya, ikincisinde mekanik yürütmeye ihtiyaç var. Yapay zeka da aynı ayrıma ihtiyaç duyuyor.

Delegasyon, "AI ne yapabilir?" sorusundan çok "bu iş için AI *doğru araç mı?*" sorusunu sormayı gerektiriyor. Ve bu soruya dürüstçe cevap vermeyi.

Güvenilir olgusal zemin, dikkatli insan yargısı veya doğrulayamayacağınız alan bilgisi gerektiren bir işi modele devretmek — bu bir delegasyon hatasıdır, AI hatası değil. Araç kapasitesini yanlış iletmedi; siz görevi yanlış tanımladınız.

## Description: AI'a Ne Söylediğiniz Önemli

İkinci D, AI ile iletişim kurma biçimiyle ilgili. Çıktıyı nasıl tanımlıyorsunuz, süreci nasıl yönlendiriyorsunuz, istediğiniz davranışı nasıl ifade ediyorsunuz?

"Prompt engineering" terimini duymuşsunuzdur. Bu terimi duyduğumda aklıma 2015'te "cloud engineer" unvanının ne hızlı popülerleştiği geliyor. O zamanlar kimisi bir S3 bucket açıp "bulut mühendisiyim" diyordu. Şimdi de kimisi ChatGPT'ye "sen deneyimli bir uzman asistanısın ve..." yazıp "prompt engineer" oluyor. Kavram gerçek; ama bu unvana giden çıta hiç bu kadar düşük olmamıştı.

Gerçek Description yetkinliği daha derinde. Temel şu: ne istediğinizi, *gerçekten* ne istediğinize kadar düşünmeden modele iletebilmeniz mümkün değil. GIGO — *Garbage In, Garbage Out* — 1957'den bu yana geçerli bir bilişim ilkesi. Büyük dil modelleri bu ilkeyi ortadan kaldırmadı; sadece çöp çıktıyı daha akıcı, daha emin ve daha ikna edici bir biçimde üretir hale getirdi.

İyi bir Description şunları yapar: çıktıyı konu bazında değil somut bir hedef olarak tanımlar; kısıtları açıkça belirtir; modele "ne yapmak istiyorsun" yerine "ne üretmeni istiyorum" diye bakar.

## Discernment: Güvenme, Doğrula (Ama Daha Dikkatli)

Üçüncü D, AI çıktısını eleştirel gözle değerlendirmeyle ilgili. Kalite, doğruluk, uygunluk — bunlar çıktıda var mı?

LLM'lerin "hallucination" denen özelliğini biliyorsunuzdur: hiç olmayan kaynaklar, hiç yazılmamış makaleler, hiç var olmamış kütüphane metodları. Model bunu son derece güvenli bir üslupla üretiyor. Bir anlamda, sınav kağıdını baştan sona uydurmuş ama sizi en çok ikna eden meslektaş gibi.

Discernment burada devreye giriyor. Goodhart Yasası'nı hatırlayın: *"Bir ölçüt hedefe dönüştüğünde, iyi bir ölçüt olmaktan çıkar."* AI çıktısını "içeriği var, uzun görünüyor, düzgün biçimlendirilmiş" diye onaylamak tam bu tuzağa düşmek. Yüzeysel tutarlılığı optimize ettiniz, doğruluğu kontrol etmediniz.

İyi bir Discernment şöyle sorular sorar: Bu doğru mu? Bu duruma uygun mu? Nerede eksik kalıyor? Nasıl daha iyi hale gelir? Bu sorular kuşkuculuk için değil — AI ile iyi çalışmayı sıradan kullanımdan ayıran editoryal süreç için sorulur.

## Diligence: Kim Hesap Verecek?

Dördüncü D, belki en az konuşulan ama en önemli olan: sorumluluk. AI'ı sorumlu ve etik kullanmak; şeffaflık, hesap verebilirlik ve hangi sistemi hangi koşullarda kullandığınıza dair bilinçli seçimler.

"Bunu AI yaptı" savunması, tıpkı "bunu compiler üretmişti" savunması kadar geçerli. Yani: geçerli değil. Kodu commit eden sizsiniz, analizi sunan sizsiniz. AI bir araç; sorumluluk, aracı kullanan insanda kalıyor. Sorumluluğun araçta kaldığı yorumunu kabul eden dil tipi statik değil, ama siz dinamik tipli olmak zorunda değilsiniz.

Diligence aynı zamanda şeffaflığı kapsıyor: ne zaman AI kullandığınızı açıklamak, bağlamı paylaşmak, süreci görünür kılmak. Bir ekipte çalışıyorsanız bu temel iletişim; yüksek riskli bağlamlarda bu güvenin temel koşulu.

Ve son olarak: araç seçimi de bu D'nin parçası. Her model aynı değil, her kullanım aynı riski taşımıyor. Tüm AI araçlarını birbirinin yerine geçebilirmiş gibi kullanmak, tüm veritabanlarını birbirinin yerine kullanmak gibi bir şey — teorik olarak mümkün, pratikte sorunlu.

## Dördü Bir Arada

4D Framework'ün güzel yanı şu: bu dört yetkinlik birbirini besliyor. İyi Delegasyon doğru işi tanımlamanıza yardım ediyor. İyi Description modeli doğru yönlendiriyor. İyi Discernment çıktının kalitesini koruyor. İyi Diligence tüm bu sürecin hesap verebilir kalmasını sağlıyor.

Dördünü birlikte düşünmek, "AI kullanıyor muyum?" sorusundan "AI ile ne kadar iyi çalışıyorum?" sorusuna geçmek demek. Bu, bir üretkenlik meselesi olduğu kadar bir profesyonel olgunluk meselesi.

## Yapay Zeka Akıcılığı: Yeni Bir Temel Yetkinlik

"Yapay zeka okuryazarlığı" kavramı son birkaç yılda sıkça kullanılır oldu. Çoğu zaman "ChatGPT'yi açıp kullanabilmek" anlamına indirgeniyor. 4D Framework bu kavramı genişletiyor: sadece araçları çalıştırmak değil, araçlarla *akıllıca* çalışmak.

Ve bu dört refleks — Delegasyon, Tanımlama, Sorgulama, Özen — temelde yeni şeyler değil. İyi bir mühendiste, iyi bir bilim insanında, iyi bir ekip oyuncusunda zaten aranan şeyler bunlar. AI, bu refleksleri daha görünür, daha acil ve daha kasıtlı geliştirilmesi gereken bir hale getiriyor.

Framework Ringling College ve University College Cork'taki araştırma işbirliğinden çıktı — ama içerdiği sorular evrensel. Ve bu soruları ne kadar erken kendinize sormaya başlarsanız, "AI kullanıcısı" olmaktan "AI ile akıllıca çalışan biri" olmaya o kadar erken geçersiniz.

Bu fark küçük görünüyor. Değil.
