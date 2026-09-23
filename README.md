# Yüzey EMG Sinyal Koşullandırma Kartı

KiCad ile tasarlanmış, yüzey elektromiyografi (sEMG) sinyalini yükseltip filtreleyen iki katmanlı analog ön uç prototipi. Girişteki diferansiyel elektrot sinyali, kas etkinliğini temsil eden tek uçlu bir zarf sinyaline dönüştürülmek üzere işlenir. Çıkış, ESP32 gibi bir mikrodenetleyicinin ADC girişine bağlanması hedeflenerek tasarlanmıştır.

> **Proje durumu:** Bu bir araştırma ve eğitim prototipidir. Rapor, LTspice simülasyonlarını, KiCad PCB yerleşimini ve sınırlı fiziksel testleri içerir. Sürekli çalışma, elektriksel güvenlik ve klinik performans doğrulanmış değildir. Elektrotları bir kişiye bağlamadan önce hasta bağlantısı, izolasyon, kaçak akım ve test ekipmanı güvenliği ayrıca mühendislik incelemesinden geçmelidir.

## İçindekiler

- [Sistem özeti](#sistem-özeti)
- [Devre blokları](#devre-blokları)
- [KiCad projesini açma](#kicad-projesini-açma)
- [PCB tasarımı ve üretim öncesi kontroller](#pcb-tasarımı-ve-üretim-öncesi-kontroller)
- [Besleme ve arayüz](#besleme-ve-arayüz)
- [Doğrulama ve mevcut sınırlamalar](#doğrulama-ve-mevcut-sınırlamalar)
- [Katkı ve issue açma](#katkı-ve-issue-açma)

## Sistem özeti

```mermaid
flowchart LR
    A["Diferansiyel elektrot girişi"] --> B["INA121"]
    B --> C["19,4 Hz HPF"]
    C --> D["50 Hz Twin-T notch"]
    D --> E["482 Hz LPF"]
    E --> F["Ek kazanç"]
    F --> G["Doğrultucu ve zarf"]
    G --> H["ADC çıkışı"]
```

PCB, bu analog sinyal zincirini taşır. Raporda çıkışın ESP32 GPIO34 ADC girişine verilmesi hedeflenir; **ESP32'nin kart üzerinde bulunduğu veya mikrodenetleyici yazılımının bu depoda yer aldığı varsayılmamalıdır.**

## Devre blokları

| Aşama | İşlev | Raporda verilen tasarım değeri |
| --- | --- | --- |
| INA121 enstrümantasyon yükselteci | Elektrotlar arasındaki küçük diferansiyel sinyali yükseltir, ortak mod gürültüsünü bastırır | `R_G = 5,6 kΩ`, ideal kazanç yaklaşık `9,93` |
| Sallen-Key yüksek geçiren filtre | Hareket artefaktları ve DC kaymasını azaltır | `82 kΩ`, `100 nF`; karakteristik frekans yaklaşık `19,41 Hz` |
| Aktif Twin-T notch | Şebeke kaynaklı 50 Hz bileşenini azaltır | Merkez yaklaşık `50,05 Hz`; ayarlanabilir geri besleme |
| Sallen-Key alçak geçiren filtre | Yüksek frekanslı gürültüyü bastırır | `33 kΩ`, `10 nF`; karakteristik frekans yaklaşık `482,28 Hz` |
| Evirmez ek kazanç katı | Filtrelenmiş sinyali ADC için ölçekler | `91 kΩ` ve `10 kΩ`; ideal kazanç `10,1` |
| Hassas yarım dalga doğrultucu | Küçük genlikli sinyali diyot eşik kaybını azaltarak doğrultur | Raporda `1N4148` diyotları kullanılır |
| Zarf dedektörü | Doğrultulmuş dalgalardan kas aktivitesi zarfını elde eder | `33 kΩ × 1 µF = 33 ms` zaman sabiti |
| Çıkış koruması | ADC girişindeki aşırı gerilimi sınırlamayı amaçlar | Raporda nominal `3,3 V` Zener diyot |

Tablodaki frekanslar ve kazançlar rapordaki **hesaplanan/benzetimde hedeflenen** değerlerdir; üretilecek her PCB'nin ölçülmüş performansı olarak değerlendirilmemelidir. Filtre topolojisi, eleman toleransı, yükselteç davranışı ve notch ayarı gerçek cevabı değiştirir. Kazanç katlarının ideal çarpımı yaklaşık `100,3` olur; raporda çeşitli yerlerde yaklaşık `101` ve `91` değerleri geçer. Bu fark gerçek kart üzerinde ölçülmelidir.

## KiCad projesini açma

1. [KiCad'i](https://www.kicad.org/download/) kurun. Proje dosyalarının kaydedildiği sürümü biliyorsanız aynı veya uyumlu bir sürüm kullanın.
2. Depoyu tüm dosyalarıyla indirin veya klonlayın. KiCad proje dosyası (`*.kicad_pro`) varsa onu açın; şema (`*.kicad_sch`) ve PCB (`*.kicad_pcb`) dosyalarını proje yöneticisinden inceleyin.
3. Şema editöründe **ERC (Electrical Rules Checker)** çalıştırın. Uyarıları ve bilinçli istisnaları gözden geçirin.
4. PCB editöründe katmanları, footprint eşleşmelerini, kart dış çizgisini ve bakır bölgelerini inceleyin. Bakır bölgelerini yeniden doldurup **DRC (Design Rules Checker)** çalıştırın.
5. 3D görünümü mekanik yerleşimi incelemek için kullanın. Üretim düşünülüyorsa güncel şema/PCB'den BOM, Gerber ve delik dosyalarını yeniden oluşturun; çıktıları ayrıca Gerber görüntüleyicide kontrol edin.

**Dosyalar hakkında:** Bu README hazırlanırken erişilebilir ek yalnızca proje raporunun PDF kopyasıydı. Gerçek `*.kicad_pro`, `*.kicad_sch` ve `*.kicad_pcb` dosyaları incelenemediğinden dosya adları, PCB ölçüleri, footprint listesi, bağlantı konnektörlerinin pin sırası, ERC/DRC sonucu ve üretim dosyalarının varlığı burada doğrulanmıyor. Bu dosyalar depoya eklendiğinde bu bölüm gerçek dosya yapısına göre güncellenmelidir.

## PCB tasarımı ve üretim öncesi kontroller

Rapordaki PCB görüntüsü iki katmanlı bir yerleşim, alt tarafta GND bakır bölgesi, SMD pasifler, soketli entegreler ve harici bağlantılar gösterir. Analog girişlerin kısa ve gürültü kaynaklarından uzak tutulması tasarımın amacıdır.

Üretim siparişinden önce en az şu noktaları şema ve PCB üzerinde birlikte doğrulayın:

- Elektrot girişleri, pil bağlantıları ve ADC çıkışının **pin sırası, polaritesi ve ortak GND** bağlantısı.
- INA121 ve operasyonel yükselteçlerin pin dizilimleri, besleme gerilimleri ve kullanılan footprintlerin gerçek parçalarla eşleşmesi.
- Twin-T ağındaki direnç/kondansatör değerleri ve trimpotun bağlantı yönü.
- Toprak düzleminin gerçekten bağlı ve yeterince sürekli olması; dar dönüş yolları ve hassas giriş çevresindeki yerleşim.
- Entegrelerin besleme pinlerine yakın yerel bypass kondansatörleri, giriş koruması, güç açılış geçicileri ve ADC çıkışındaki akım sınırlaması.
- Şema ile PCB senkronizasyonu, ERC/DRC bulguları ve üreticinin iz genişliği/boşluk/delik kuralları.

**Önemli tasarım incelemesi:** Raporda GND düzleminin ayrık bypass kondansatörlerinin yerini alacağı varsayılmış ve giriş koruması gelecekteki geliştirme olarak bırakılmış. GND düzlemi, entegre besleme pinlerindeki yerel bypass işlevini tek başına garanti etmez. Benzer şekilde nominal 3,3 V Zener diyotun ADC girişini her arızada tam 3,3 V'ta tutacağı varsayılmamalıdır; akım sınırlaması, tolerans ve arıza halleri ayrıca hesaplanıp ölçülmelidir.

## Besleme ve arayüz

Rapor, analog katlar için seri bağlanmış iki 9 V pilden elde edilen yaklaşık `+9 V / GND / -9 V` beslemeyi tarif eder. Orta noktanın analog GND olarak bağlanması ve polaritenin kart üzerindeki etiketlerle doğrulanması gerekir. Beslemeye geçmeden önce kartın gerçek şemasındaki pin dizilimini esas alın.

Hedeflenen sinyal çıkışı tek uçlu kas aktivitesi zarfıdır. ESP32 ADC'ye bağlamadan önce çıkışın **bütün çalışma ve açılış/kapanış durumlarında** seçilen ESP32 girişinin izin verilen aralıkta kaldığı doğrulanmalıdır. Raporun hedefi 0–3,3 V'tur; nominal Zener değeri tek başına bu doğrulamayı sağlamaz. Elektrot girişlerine bağlı bir kişiye, şebeke topraklı osiloskop veya USB bağlantılı bilgisayar üzerinden istenmeyen akım yolu oluşturabilecek test düzenleri kurulmamalıdır.

## Doğrulama ve mevcut sınırlamalar

- Raporda LTspice AC ve geçici rejim benzetimleri ile 50 Hz çentiği, filtre sınırları ve 33 ms zarf tepkisi incelenmiştir.
- Rapor, iki fiziksel prototip kartın üretildiğini ve ilk tezgâh testlerinde beklenen dalga biçimlerinin gözlendiğini bildirir; tam kalibrasyon veya uzun süreli güvenilirlik testi bildirmez.
- Ön uçtaki **INA121 iki testte hasar görmüştür**. Rapor, açılış geçicileri ve probla temas sırasında ESD/gerilim zorlanmasını olası nedenler arasında sayar. Bu nedenle yeni kartta giriş koruması, güç sıralaması ve bypass ağı gözden geçirilmeden ölçümlerin tekrarlanması uygun değildir.
- Klinik tanı, hasta izleme veya rehabilitasyon cihazı olarak kullanım doğrulanmamıştır. Raporda tarif edilen ortotik/ESP32 entegrasyonları gelecekteki uygulama hedefleridir.

### Tekrar üretilebilir test kaydı için öneri

Bir ölçüm veya değişiklik paylaşıyorsanız kart revizyonunu, kullanılan bileşenleri, besleme değerlerini, test düzeneğini, osiloskop prob bağlantısını, giriş sinyalinin genlik/frekansını ve ölçülen çıkış dalga biçimini kaydedin. İnsan bağlantısı içermeyen güvenli tezgâh testlerini tercih edin. Bu README herhangi bir kişinin üzerinde deney yapılması için uygulama talimatı değildir.

## Katkı ve issue açma

GitHub'da hata veya geliştirme önerisi için önce mevcut **Issues** kayıtlarını arayın, ardından **New issue** açın. Kart revizyonunu, ilgili KiCad şema/PCB konumunu, beklenen ve görülen davranışı, ERC/DRC çıktısını veya ölçüm koşullarını ekleyin. Kişisel sağlık verileri ve kişi üzerinde yapılan ölçümlerin kimlik bilgilerini paylaşmayın. Değişiklik önerisi için ayrı bir dalda çalışıp şema ile PCB'yi birlikte güncelleyin ve bir Pull Request'te doğrulama adımlarını yazın.

## Referanslar

- Talha Uğur, *Design and Implementation of a High-Precision Signal Conditioning Circuit for Electromyography (EMG) Applications*, proje raporu, 24 Mayıs 2026. Bu README'deki devre değerleri ve prototip durumu bu rapordan alınmıştır.
- [KiCad başlangıç ve ERC/DRC kılavuzu](https://docs.kicad.org/10.0/en/getting_started_in_kicad/getting_started_in_kicad.pdf)
- [FDA: Tıbbi cihazlarda elektromanyetik ve elektriksel güvenlik araştırmaları](https://www.fda.gov/medical-devices/medical-device-regulatory-science-research-programs-conducted-osel/electromagnetic-and-electrical-safety-program-research-electromagnetic-and-electrical-safety-medical)
