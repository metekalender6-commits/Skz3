# SKZ Zone Plugin (CS2 Jailbreak) — CounterStrikeSharp

## Ne yapar?
- `!cross` — Bulunduğun noktayı, SKZ başında herkesin ışınlanacağı "haç" noktası olarak kaydeder.
- `!zone` — İki köşeyi sırayla işaretleyerek güvenli bölgeyi (kutu şeklinde) kaydeder. Önce bir köşeye gidip `!zone` yaz, sonra karşı köşeye gidip tekrar `!zone` yaz.
- `!skz <saniye>` — Turu başlatır: tüm canlı Terrorist (tutsak) oyuncular cross noktasına çekilir, verdiğin süre içinde zone'a giren hayatta kalır, giremeyen süre bitince ölür.

Cross ve zone bilgileri `skz_data.json` dosyasına yazılır, bu yüzden **server kapanıp açılsa bile kaybolmaz** — bir kere kurman yeterli.

---

## ⚠️ Önemli: elindeki .dll değil, kaynak kod
Ben (Claude) bu ortamda internete çıkamıyorum ve içinde .NET derleyicisi yok, o yüzden sana hazır derlenmiş bir `.dll` dosyası **üretemedim**. Onun yerine üç dosya hazırladım ki en az işle kendi `.dll`'ini alabilesin:

- `SkzZonePlugin.cs` — asıl kod
- `SkzZone.csproj` — proje dosyası
- `.github/workflows/build.yml` — push attığında otomatik derleyen GitHub Actions dosyası

## En kolay yol: GitHub Actions ile derletmek (kurulum gerektirmez)
1. GitHub'da yeni, boş (public veya private fark etmez) bir repo aç.
2. Bu üç dosyayı repo'ya at (klasör yapısını koru: `.github/workflows/build.yml` aynı yolda kalmalı).
3. Push et.
4. Repo'da **Actions** sekmesine gir, çalışan "Build SKZ Zone Plugin" işine tıkla, altındaki **Artifacts** kısmından `SkzZone-dll` diye bir zip inecek. İçinde `SkzZone.dll` var.
5. Bu dll'i sunucunda şu klasöre at:
   `csgo/addons/counterstrikesharp/plugins/SkzZone/SkzZone.dll`
6. Server'ı restart et veya `css_plugins load SkzZone` yaz.

## Alternatif: kendi bilgisayarında derlemek
1. [.NET 10 SDK](https://dotnet.microsoft.com/download/dotnet/10.0) kur.
2. Üç dosyayı aynı klasöre koy, o klasörde terminal aç.
3. `dotnet build -c Release` çalıştır.
4. `bin/Release/net10.0/SkzZone.dll` dosyasını yukarıdaki plugin klasörüne kopyala.

> **Not:** `TargetFramework` net10.0 olarak ayarlandı, dediğin gibi. Tek dikkat edeceğin şey: CounterStrikeSharp'ın kendi native host'unun (Metamod üstüne binen kısım) hangi .NET runtime'ını yüklediği — eğer host hâlâ .NET 8 runtime'ı yüklüyorsa, net10 hedefli dll plugin klasörüne konsa bile server'da yüklenmeyebilir ("wrong framework"/"could not load runtime" gibi bir hata verir). Bu durumda sunucunda `dotnet --list-runtimes` ile hangi runtime'ların kurulu olduğuna bakabilir, ya da CounterStrikeSharp sürümünü/host'unu net10 destekleyen bir sürüme güncelleyebilirsin. Yükleme sırasında hata alırsan mesajı buraya yapıştır, ona göre bakarız.

## Yetki
Komutlar `@css/root` yetkisi ister. Kendi admin sisteminde bu flag'e sahip olduğundan emin ol (`addons/counterstrikesharp/configs/admins.json`).

## Kullanım sırası (örnek)
1. Haritada haç olacak yere git, `!cross` yaz.
2. Güvenli bölgenin bir köşesine git, `!zone` yaz. Karşı köşeye git, `!zone` yaz tekrar.
3. Round içinde istediğin an `!skz 5` yaz — 5 saniyelik SKZ turu başlar.

## Kodu nasıl doğruladım
Teleport, QAngle/Vector constructor'ları, TeamNum/CsTeam karşılaştırması, CommitSuicide, AddTimer/TimerFlags, RequiresPermissions ve CommandHelper kullanımlarının hepsini CounterStrikeSharp'ın resmi `docs.cssharp.dev` API dokümantasyonu ve GitHub'daki resmi `TestPlugin.cs` örneğiyle tek tek karşılaştırarak yazdım. Yine de sunucundaki CounterStrikeSharp sürümüne karşı gerçekten derlenmiş halini test etmedim (derleyicim yok) — GitHub Actions derlemesi başarısız olursa hata mesajını yapıştır, birlikte düzeltiriz.

## Notlar / ince ayar isteyebileceğin yerler
- Şu an sadece **Terrorist (T)** takımındaki canlı oyuncular çekiliyor ve kontrol ediliyor — jailbreak'te tutsaklar T tarafında olduğu için.
- Zone kutusunun yükseklik payı köşe noktalarına göre -40 / +90 unit ekleniyor; haritana göre `OnZoneCommand` içinde ayarlayabilirsin.
- Ölüm anında `CommitSuicide(false, true)` kullanılıyor (patlama efekti kapalı); istersen ilk parametreyi `true` yap.
