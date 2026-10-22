# prompt

AI asistanlarına proje yapıları kurdurmak için kullanılan promptlar.

| Prompt | Ne kurar |
|---|---|
| [`ai-asistan-kurulum-prompt.md`](ai-asistan-kurulum-prompt.md) | Projeye özel AI geliştirme asistanı yapısı |
| [`ai-review-talimat-reposu-kurulum-prompt.md`](ai-review-talimat-reposu-kurulum-prompt.md) | YKB AI Review eklentisinin okuduğu ortak talimat reposu (repolara özel inceleme kuralları) |

## Proje asistanı kurulumu
1. Hedef projeyi IDE'de aç.
2. Copilot Chat'i **Agent** modunda, model olarak Claude seçili şekilde aç (Plan/Ask modları dosya oluşturamaz).
3. [`ai-asistan-kurulum-prompt.md`](ai-asistan-kurulum-prompt.md) içeriğini kopyala, `[VS Code / IntelliJ — doldur]` alanını doldur ve gönder.
4. Faz 1 sonunda gelen haritayı ve soruları kontrol et.

## AI Review talimat reposu kurulumu
1. Bitbucket'ta talimat reposunu oluştur (ör. `SDLC/ai-review-rules`) ve yerelde klonla. Eklentiyi kullanacak herkesin bu repoda okuma yetkisi olmalı.
2. Kurallarını çıkarmak istediğin repoları ve talimat reposunu aynı çalışma alanında aç.
3. Copilot Chat'i **Agent** modunda, model olarak Claude seçili şekilde aç.
4. [`ai-review-talimat-reposu-kurulum-prompt.md`](ai-review-talimat-reposu-kurulum-prompt.md) içeriğini kopyala, `— doldur` alanlarını doldur ve gönder.
5. Faz 1 sonundaki repo tablosunu kontrol et. Teslimde oluşturulan dalı inceleyip talimat reposuna PR olarak gönder.
6. Eklenti ayarlarında talimat reposunu tanımla ve "Test Et / Yenile" ile doğrula.
