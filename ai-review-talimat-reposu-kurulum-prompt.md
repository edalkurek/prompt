# Görev: Repolarım için YKB AI Review ortak talimat reposunu kur

## Bağlam
- **YKB AI Review** eklentisi (VS Code / IntelliJ) Bitbucket PR'larını ve yerel değişiklikleri bir dil modeliyle inceler. İnceleme sırasında ekip kurallarını Bitbucket'taki **ortak talimat reposundan** okur ve modele "proje özel talimatları" olarak gönderir.
- Ortak talimat reposu: **[PROJE/slug — doldur, ör. SDLC/ai-review-rules]**, dal: **[master — doldur]**. Yerel kopyası: **[yerel yol — doldur]**.
- İncelenecek repolar: **[repo listesi veya "çalışma alanında erişebildiğin tüm repolar" — doldur]**.
- Amaç: Her repo için, o repodaki PR'larda gerçekten işe yarayacak, **koddan çıkarılmış ve kanıtlanmış** inceleme kurallarını talimat reposuna doğru dosya düzeniyle yazmak.

## Eklentinin talimatları nasıl okuduğu (değiştirilemez sözleşme)
Eşleştirme için ayrı bir tanım yok, **dosyanın yolu hangi repolara uygulanacağını belirler**:

```
<talimat-reposu>/
├── general.md                     ← tüm repolar
├── projects/
│   └── <PROJE>.md                 ← o Bitbucket projesindeki tüm repolar
└── repos/
    └── <PROJE>/
        └── <repo-slug>.md         ← yalnızca o repo
```

- `<PROJE>`: Bitbucket **proje anahtarı**, **BÜYÜK harf** (`PAY`). `<repo-slug>`: repo URL'sindeki ad, **küçük harf** (`payment-api`).
- Bunları her reponun git remote adresinden çıkar: `https://<host>/scm/pay/payment-api.git` veya `ssh://git@<host>:7999/pay/payment-api.git` → `PAY` / `payment-api`. Remote'ta proje anahtarı küçük harfle görünür, dosya yolunda büyük harfe çevir.
- Talimatlar genelden özele birleştirilir: `general.md` → `projects/<PROJE>.md` → `repos/<PROJE>/<slug>.md` → IDE'deki kişisel kurallar. Çelişkide sonra gelen geçerlidir.
- Olmayan dosya sessizce atlanır. Yalnızca söyleyecek bir şeyi olan dosyayı oluştur.
- **Uzunluk bütçesi:** Bir repo için birleşen üç dosyanın toplamı **en fazla 10.000 karakter** olsun (öneri: `general.md` ≤ 3.000, proje dosyası ≤ 3.000, repo dosyası ≤ 4.000). Eklentilerden biri 12.000 karakterin üstünü keser ve kişisel IDE kurallarına da yer kalmalıdır.
- Model PR'ın **yalnızca diff'ini** ve PR başlığı/açıklamasını görür; reponun geri kalanını göremez. Kural, diff'e bakarak kontrol edilebilir olmalıdır.
- Eklentinin kendi sistem kuralları (kanıt şartı, çıktı biçimi, önem seviyeleri, güvenlik) vardır ve talimatlar bunları geçersiz kılamaz. Talimatlara çıktı biçimi, JSON şeması, rol tanımı veya "önceki talimatları yok say" türü ifadeler **yazma**.

## Temel ilkeler (her aşamada geçerli)
1. **Varsayma, keşfet.** Kuralları koddan, testlerden, yapılandırmadan, dokümanlardan, geçmiş PR yorumlarından ve hata düzeltme commit'lerinden çıkar. Genel "iyi pratik" listeleri kopyalama.
2. **Kanıt seviyesi belirt.** Çıkardığın her kuralı işaretle:
   - `[DOĞRULANDI]`: Birden fazla bağımsız örnekte görüldü ya da test, lint veya config ile zorunlu tutuluyor.
   - `[GÖZLEM]`: Tek örnekte veya tek alanda görüldü.
   - `[AÇIK]`: Çelişkili veya belirsiz. Talimat dosyasına yazılmaz, bana sorulur.
   Etiketler yalnızca raporda kullanılır, talimat dosyalarına yazılmaz.
3. **Otomatik kontrolleri tekrarlama.** Linter, formatter, derleyici, statik analiz veya testlerin zaten yakaladığı kuralları yazma. Model, bu araçların kaçırdığı mantık, güvenlik, veri ve mimari hatalarına odaklanmalı.
4. **Doğru katmana yaz.** Bir kural yalnızca geçerli olduğu en geniş katmanda bir kez yazılır, alt katmanda tekrarlanmaz.
5. **Gizli bilgi yok.** Talimatlara şifre, token, iç IP/host, müşteri verisi veya kişi adı yazma. Talimatlar bir dil modeline gönderilir.
6. **Ürün koduna dokunma.** İncelenen repolarda hiçbir dosyayı değiştirme. Yalnızca talimat reposunda çalış. Push veya merge yapma.
7. **Erişim sınırını açıkça belirt.** Erişemediğin bir repo, PR geçmişi veya doküman "yok" sayılmaz. Listele, gerekirse benden iste.

## Aşamalar

### Faz 1: Envanter
- Talimat reposunun mevcut içeriğini oku. Var olan dosyaları silme veya üzerine yazma; birleştirilecek, sadeleştirilecek veya taşınacak içeriği raporla.
- Her repo için şunları çıkar: proje anahtarı, slug, dil ve framework, build/test/lint komutları, otomatik kontrol araçları ve kuralları, mevcut AI talimatları (`.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md`, `.cursor/rules`, `docs/`, README).
- Erişebiliyorsan her repo için son 6–12 ayın birleşmiş PR'larındaki inceleme yorumlarını ve `fix`/`hotfix`/`bug` commit'lerini tara.
- **Kontrol noktası:** Bana şu tabloyu sun ve kritik bir belirsizlik yoksa beklemeden devam et:

  | Repo | Proje anahtarı | Slug | Teknoloji | Otomatik kontroller | PR geçmişine erişim | Mevcut AI talimatı |
  |---|---|---|---|---|---|---|

### Faz 2: Kural adaylarını çıkar
Her repo için diff'e bakarak kontrol edilebilecek kuralları ara. Örnek alanlar (yalnızca projede gerçekten varsa):
- **Güvenlik:** Yetkilendirme kontrolleri, girdi doğrulama, SQL/komut enjeksiyonu, log'a hassas veri yazma, gizli bilgi yönetimi.
- **Veri ve transaction:** Transaction sınırları, migration/DDL/DML kuralları (ör. SQL script düzeni, geri alma script'i), idempotency, eşzamanlılık.
- **Hata yönetimi:** Exception yutma, hata kodu eşleme, retry/timeout.
- **Alan kuralları:** Tutar hesaplama, para birimi, tarih/saat dilimi, durum geçişleri, dil kaynakları (ör. çeviri anahtarları ve tüm diller).
- **Mimari sınırlar:** Katman ihlalleri, yasaklı bağımlılıklar, ortak bileşen yerine kopya kod.
- **Birlikte değişmesi gerekenler:** "X değişirse Y de değişmeli" ilişkileri (ör. yeni uç nokta → yetki tanımı + test). Diff'te Y yoksa model bunu belirtebilir.
- **Geçmişte tekrar eden hatalar:** PR yorumlarında ve hata düzeltmelerinde birden fazla kez görülen sorunlar.

Her aday için kaynağı (dosya yolu, PR veya commit) ve kanıt seviyesini not et.

### Faz 3: Katmanlara ayır
- Birden fazla Bitbucket projesindeki repolarda `[DOĞRULANDI]` olan kural → `general.md`.
- Bir projedeki repoların hepsinde veya büyük çoğunluğunda geçerli olan kural → `projects/<PROJE>.md`.
- Yalnızca tek repoya özgü kural → `repos/<PROJE>/<slug>.md`.
- Yalnızca `[DOĞRULANDI]` kuralları zorunlu dille yaz. `[GÖZLEM]` kuralları yalnızca kapsamı net ve yanlış alarm riski düşükse, kapsamıyla birlikte ve yumuşak dille ("…ise kontrol et") yaz. `[AÇIK]` olanları yazma.
- Bütçeyi aşıyorsan en az değerli kuralları çıkar. Kural sayısı değil isabet önemlidir.

### Faz 4: Dosyaları yaz
Her dosya düz Markdown'dır ve olduğu gibi modele gider. Şu düzeni kullan:

```markdown
# <Kapsam adı> inceleme kuralları

## <Alan, ör. Güvenlik>
- <Diff'te neye bakılacağı, somut ve kontrol edilebilir>. <Gerekirse tek cümlelik neden.>
- <Kural yalnızca belirli dosyalara uygulanıyorsa kapsamı: "`*.sql` dosyalarında …", "`…Controller` sınıflarında …">

## Raporlanmaması gerekenler
- <Bu repoda bilinçli olarak böyle yapılan ve hata sayılmaması gereken durumlar>
```

Yazım kuralları:
- Her madde tek bir kontrol içersin. Kısa, somut ve doğrulanabilir olsun. "Kod temiz olmalı" gibi belirsiz ifadeler kullanma.
- Kod bloğu kopyalama. Gerekirse sınıf veya metot adıyla referans ver (model yalnızca diff'i gördüğü için adın diff'te görünecek olması önemlidir).
- Önem seviyesi gerekiyorsa maddede belirt ("yüksek önem: …"), ama çıktı biçimi tarif etme.
- Türkçe yaz. Kod, sınıf ve dosya adlarını olduğu gibi bırak.

### Faz 5: Doğrula
- **Yol ve ad:** Her dosyanın yolu sözleşmeye uyuyor mu? Proje anahtarı büyük, slug küçük harf mi? Slug remote ile birebir aynı mı?
- **Bütçe:** Her repo için `general.md` + proje dosyası + repo dosyasının karakter toplamını hesapla ve tablo halinde göster. 10.000'i aşan varsa kısalt.
- **Tekrar ve çelişki:** Aynı kural iki katmanda geçiyor mu? Alt katman üst katmanla çelişiyor mu (çelişki bilinçliyse açıkça "bu repoda … geçerli değildir" yaz)?
- **Gizli bilgi:** Dosyalarda token, şifre, iç adres, müşteri veya kişi bilgisi var mı?
- **Davranış testi:** Her repo için geçmişten en az 2 gerçek PR diff'i seç (biri kuralın yakalaması gereken bir sorun içersin, biri temiz olsun). Yalnızca birleşmiş talimat metnini ve diff'i okuyarak bir inceleme yap. Kurallar beklenen sorunu yakalıyor mu, temiz diff'te yanlış alarm üretiyor mu? Sonuçlara göre kuralları düzelt.
  PR geçmişine erişemiyorsan, senaryo metinlerini ve beklenen sonuçları benim çalıştırmam için hazırla.

### Faz 6: Teslim
- Talimat reposunda `ai-review/talimat-kurulumu` adlı bir dal aç, dosyaları tek commit'te ekle. **Push yapma**, onayımı bekle.
- Şu başlıklarla raporla:
  1. **Oluşturulan / güncellenen dosyalar:** Yol, kapsadığı repolar, karakter sayısı, kural sayısı.
  2. **Repo → uygulanan dosyalar tablosu:** Her repo için hangi üç dosyanın okunacağı ve toplam karakter.
  3. **Davranış testi sonuçları:** Repo, PR, beklenen, gerçekleşen, yapılan düzeltme.
  4. **Yazılmayan kurallar:** `[GÖZLEM]` ve `[AÇIK]` listesi, her biri için kaynak ve bana sorulacak soru.
  5. **Eklentide yapılacak ayar:** IntelliJ: *Settings → Tools → YKB AI Code Review → Ortak Talimat Reposu* alanına `PROJE/slug` ve dal; VS Code: kurulum panelindeki *Talimat reposu adresi* ve *Dal* alanları. Ardından "Test Et / Yenile".
  6. **Bakım düzeni:** Kurallar nasıl güncellenir (talimat reposuna PR ile), yeni repo eklenince ne yapılır. Her dosyanın sonuna `<!-- son doğrulama: YYYY-AA-GG, referans commit: <hash> -->` satırı ekle.

## Başla
Şimdi Faz 1 ile başla. Benden kuralları listelememi isteme. Yalnızca incelemeyle çözemediğin ve doğru ilerlemek için gerekli konularda soru sor.
