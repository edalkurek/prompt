# Görev: Bu projeye özel, tekrar kullanılabilir bir AI geliştirme asistanı yapısı kur

## Bağlam
- Kullandığım araç: **GitHub Copilot, agent modu, Claude modeli**. IDE: **[VS Code / IntelliJ — doldur]**.
- Copilot'un bu IDE ve sürümde desteklediği mekanizmaları ortamdan ve resmi GitHub/VS Code dokümantasyonundan doğrula. Olası mekanizmalar: `.github/copilot-instructions.md`, `.github/instructions/*.instructions.md` (`applyTo` ile), `.github/prompts/*.prompt.md`, `.github/agents/*.agent.md`, `AGENTS.md`, skill dosyaları. Hangisinin otomatik yüklendiğini ve hangisinin elle çağrıldığını tahmin etme. Doğrulayamıyorsan bana sor.
- Çalışma alanı: şu an açık olan proje ve erişebildiğin repository'ler.
- Amaç: Bundan sonra metin olarak vereceğim geliştirme taskları, projenin **gerçek** mimarisine, akışlarına ve standartlarına uygun şekilde ele alınsın.

## Temel ilkeler (her aşamada geçerli)
1. **Varsayma, keşfet.** Hazır bir mimari, katman listesi veya iş kuralı kabul etme. Her kuralı koddan, testten, yapılandırmadan, dokümandan veya değişiklik geçmişinden çıkar.
2. **Kanıt seviyesi belirt.** Çıkardığın her kuralı şu etiketlerden biriyle işaretle:
   - `[DOĞRULANDI]`: Birden fazla bağımsız örnekte görüldü ya da test veya config ile zorunlu tutuluyor.
   - `[GÖZLEM]`: Tek örnekte veya tek bir alanda görüldü. Genel kural değil.
   - `[AÇIK]`: Çelişkili ya da belirsiz. Talimatlara kural olarak yazılmaz, bana sorulur.
3. **Referans ver, kopyalama.** Kuralları gerçek dosya yolları ve sembol adlarıyla (`path/to/file.ext` → `ClassName.method`) bağla. Kod bloğu kopyalamak yerine kaynağı göster.
4. **Erişim sınırını açıkça belirt.** Erişemediğin bir repo, servis veya doküman "yok" sayılmaz. Listele, gerekirse benden iste.
5. **Ürün koduna dokunma.** Bu görev boyunca yalnızca talimat, rehber, skill ve agent dosyalarını oluştur ya da düzenle. Commit veya push yapma.
6. **Tekrar ve çelişki yok.** Var olan AI talimatlarını (ör. `.github/copilot-instructions.md`, `.github/instructions/`, `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, README, `docs/`) önce oku. Yeni yapıyı bunlarla çelişmeyecek, aynı bilgiyi iki yerde tutmayacak şekilde kur. Gerekirse mevcut dosyayı birleştir veya sadeleştir ve bunu raporla.

## Aşamalar

### Faz 1: Keşif ve harita
- Önce genel haritayı çıkar: modüller ve repository'ler, sorumlulukları, aralarındaki bağımlılıklar, build/test/lint/çalıştırma komutları, mevcut AI talimatları.
- Sonra ilgili alanlarda derinleş. Kapsamı yönetilebilir parçalara böl; araç destekliyorsa paralel alt incelemeler (subagent) kullanabilirsin.
- **Kontrol noktası:** Bu fazın sonunda bana kısa bir özet sun: harita, erişim sınırları ve kritik `[AÇIK]` noktalar. Haritayı yanlış anlamanı gerektirecek bir belirsizlik yoksa beklemeden devam et.

### Faz 2: Gerçek geliştirmeleri uçtan uca izle
- Farklı geliştirme türlerini temsil eden **en az 3–5 tamamlanmış akış** seç (ör. yeni uç nokta, mevcut iş kuralı değişikliği, veri modeli veya migration, UI ekranı, entegrasyon, hata düzeltme; hangileri projede gerçekten varsa).
- Seçimi dosya isimlerine göre değil, kodun nasıl çalıştığına göre yap. Değişiklik geçmişi erişilebilirse ilgili commit veya PR'ları incele.
- Her tür için şunları çıkar:
  - Birlikte değişen parçalar (zorunlu)
  - Yalnızca belirli koşullarda değişen parçalar (koşulu ile birlikte)
  - Kolayca unutulan adımlar (ör. kayıt, çeviri, config, migration, test fixture, dokümantasyon)
  - O türe ait doğrulama yöntemi (hangi test veya komut)
- Güvenilir referans bulamadığın bir tür varsa benden örnek iste.

### Faz 3: Kuralları yaz
- Genel kuralları, alana özgü kuralları, koşullu gereklilikleri ve istisnaları birbirinden ayır.
- Eski, istisnai veya hatalı görünen kodu standart kabul etme. Kod, test ve doküman çelişiyorsa `[AÇIK]` olarak işaretle ve sor.
- Sadece `[DOĞRULANDI]` kurallar zorunlu dille yazılır. `[GÖZLEM]` kurallar kapsamıyla birlikte belirtilir.

### Faz 4: Araca uygun yapıyı kur
- Bağlam bölümünde doğruladığın Copilot mekanizmalarını kullan.
- Hedef yapı:
  - **Ana talimat dosyası:** Kısa olsun (hedef ~150 satırın altı). Her taskta geçerli kurallar, proje haritası, komutlar ve "hangi durumda hangi rehberi oku" yönlendirme tablosu burada yer alır.
  - **Görev rehberleri:** Her geliştirme türü için bir tane. İçerik: ne zaman kullanılır, değişen parçalar checklist'i, koşullu adımlar, sık unutulanlar, referans dosyalar, doğrulama komutları, o türe özgü soru ağacı. Dosya yolu kalıbına bağlı kurallar için `applyTo` kullanılabilir.
  - **Agent tanımı (yalnızca gerçekten gerekliyse):** Task akışını yöneten, ilgili rehberi seçen orkestratör. Gereksizse oluşturma ve nedenini yaz.
- Ortak bilgi tek bir kaynakta durur, diğer dosyalar ona referans verir. Dosya veya agent sayısını gereğinden fazla artırma.
- Yeni bir chat oturumunda asistanın hangi dosyayı otomatik okuduğu, hangisini ne zaman açacağı net olmalı.

### Faz 5: Soru mekanizması
Gelecekteki her taskta asistan şöyle davranmalı:
1. Önce talebi, ilgili kodu, rehberleri ve erişilebilir kaynakları incele.
2. Eksik bilgiyi, Faz 2'de çıkarılan gerçek akışlara göre belirle. Sabit soru listesi kullanma.
3. Sadece **iş davranışını veya kapsamı etkileyen** eksik ve çelişkili bilgileri sor. Projeden güvenle çıkarılabilen teknik kararları sorma.
4. Talepte, kodda veya önceki cevaplarda bulunan bilgiyi tekrar sorma.
5. Soruları kısa ve öncelik sırasına göre gruplanmış sor (en fazla ~5 soru/tur). Her sorunun yanında hangi kararı netleştirdiğini yaz. Varsa önerilen varsayılanı da belirt.
6. Bir eksiklik yalnızca belirli bir adımı engelliyorsa bunu söyle ve bağımsız kısımlarda ilerlemeye devam et.
7. Alınan cevapları ve kararları task boyunca koru, task sonunda raporda listele.

### Faz 6: Task yaşam döngüsü
Yapı şu akışı tanımlamalı ve rehberler her adıma bağlanmalı:
**Anla → İncele → Netleştir → Etki ve değişiklik planı → Uygula → Doğrula → Raporla**
- Her taskta yalnızca ilgili rehber ve kuralları seç. Tüm projeyi veya tüm rehberleri uygulamaya kalkma.
- Bağlantılı parçaları eksik bırakma (rehberdeki checklist ile kontrol et).
- Açık görevlerde mevcut yetki ve talimatlar doğrultusunda soru sormadan ilerle. Belirsiz iş kararlarında sor.
- Task metin olarak verilebilmeli. Harici entegrasyonlar (issue tracker vb.) varsa kullanılabilir ama yapı onlara bağımlı olmamalı.

### Faz 7: Davranış testi (ürün kodu değiştirmeden)
- Faz 2'deki gerçek geliştirme türlerinden **en az 3 senaryo** hazırla:
  - (a) Bilgisi eksik bir talep: doğru ve az sayıda soru üretmeli.
  - (b) Yeterli bilgi içeren bir talep: gereksiz soru sormamalı, doğru plan çıkarmalı.
  - (c) Farklı kapsamda bir talep: doğru rehberi seçmeli, ilgisiz olanları yüklememeli.
- Mümkünse senaryoları **taze bağlamda** (yalnızca oluşturduğun dosyaları okuyan bir subagent ile) çalıştır. Bu mümkün değilse, yeni bir chat oturumunda çalıştırmam için senaryo metinlerini ve beklenen çıktıları hazırla. Beklenen sorular, seçilen rehber ve değişiklik kapsamı ile gerçek çıktıyı karşılaştır.
- Başarısız noktaları düzelt ve tekrar dene. Sonuçları tablo halinde raporla.
- Projedeki gerçek doğrulama yöntemlerini (test, lint, typecheck, build komutları) rehberlere bağla. Talimat yazılmış olmasının davranışı teknik olarak garanti etmediğini açıkça belirt.

### Faz 8: Teslim raporu
Şu başlıklarla raporla:
1. **Oluşturulan / güncellenen dosyalar:** Her birinin yolu, görevi ve nasıl devreye girdiği (otomatik mi, ne zaman okunuyor).
2. **Kullanım kılavuzu:** Yeni bir task verirken ne yazmalıyım, iyi bir task örneği.
3. **Test sonuçları:** Senaryo, beklenen, gerçekleşen, düzeltme.
4. **Doğrulanamayan kurallar ve açık sorular:** `[GÖZLEM]` ve `[AÇIK]` listesi.
5. **Bakım düzeni:** Proje değiştikçe rehberler nasıl güncellenir. Her rehberde "son doğrulama tarihi + referans commit" alanı bulunsun. Referans verilen dosya veya sembol artık yoksa asistan bunu sorgusuz kullanmak yerine fark edip uyarmalı.

## Başla
Şimdi Faz 1 ile başla. Benden mimariyi anlatmamı veya kuralları listelememi isteme. Yalnızca incelemeyle çözemediğin ve doğru ilerlemek için gerekli olan konularda soru sor.
