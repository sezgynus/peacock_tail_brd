# Peacock Tail Controller (PTC)

[English](README.md) | [Türkçe](README-tr.md)

**Peacock Tail Controller, 3D yazıcılarda kullanılan çoklu malzeme sistemleri için geliştirilmekte olan açık kaynak bir kontrol kartıdır.** Kart; RP2040-Zero denetleyici, iki adet sökülüp takılabilir TMC2209 SilentStepStick uyumlu step motor sürücü yuvası, 5 V buck regülatör ve bir MMU mekanizmasının ihtiyaç duyduğu aktüatör/sensör bağlantılarını tek PCB üzerinde toplar.

Güncel KiCad tasarımı **revizyon 0.1** olarak işaretlenmiştir ve şema tarihi **2026-05-06**'dır.

> Bu depo aktif geliştirme aşamasındaki bir donanım projesidir. Dokümantasyon mevcut şema ve PCB tasarımını açıklar; depoda bulunmayan firmware desteği veya doğrulanmamış donanım özellikleri çalışıyormuş gibi gösterilmez.

## Projeye Genel Bakış

PTC, iki motorlu bir çoklu malzeme mekanizmasının elektroniklerini merkezileştirmek üzere tasarlanmıştır:

- filament taşıma için **gear step motoru**;
- malzeme yolunu seçmek için **selector step motoru**;
- **servo** çıkışı;
- **encoder** girişi;
- **selector/endstop** girişi;
- **extra/toolhead sensör** girişi;
- ayrı bir **LED** bağlantısı.

Kartın denetleyicisi **RP2040-Zero** modülüdür. İki TMC2209 sürücü modülü için gerekli step/direction/enable, UART ve diagnostic sinyalleri kart üzerinde yönlendirilmiştir.

```text
                         Peacock Tail Controller
                 ┌──────────────────────────────────┐
                 │                                  │
                 │          RP2040-ZERO             │
                 │                 │                │
                 │       ┌─────────┴─────────┐      │
                 │       │                   │      │
                 │   Gear TMC2209      Selector    │
                 │                     TMC2209      │
                 │       │                   │      │
                 └───────┼───────────────────┼──────┘
                         │                   │
                    Gear motor         Selector motor

                 Servo  Encoder  Endstop  Extra  LED
                   │       │        │       │     │
                   └───────┴────────┴───────┴─────┘

VIN girişi ──► motor beslemesi
     │
     └──► LM2596S-ADJ buck ──► +5 V ──► kart çevre birimleri
```

## Hızlı Bakış

| Alan | Güncel tasarım |
|---|---|
| Proje | Peacock Tail Controller / PEACOCK TAIL BRD |
| Donanım revizyonu | 0.1 |
| Şema tarihi | 2026-05-06 |
| MCU modülü | RP2040-Zero |
| Step motor sürücüleri | 2 × TMC2209 SilentStepStick uyumlu modül |
| Step motor görevleri | Gear + Selector |
| Motor konnektörleri | 2 × 4 pin |
| Güç girişi | VIN + GND vidalı terminal |
| Kart üzeri dönüştürücü | LM2596S-ADJ ayarlanabilir buck katı |
| Yerel regüle hat | +5 V |
| Lojik hat | +3,3 V |
| Servo konnektörü | 3 pin |
| Encoder konnektörü | 3 pin |
| Endstop konnektörü | 3 pin |
| Extra/toolhead konnektörü | 3 pin |
| LED konnektörü | 3 pin |
| TMC kontrolü | STEP/DIR/EN + UART/DIAG ilişkili yönlendirme |
| Mekanik montaj | 3 × M3 montaj deliği |
| CAD | KiCad |
| 3D kart modeli | `Board.step` |
| Üretim paketi | `production/PEACOCK_TAIL_BRD_v0.1.zip` |

## Donanım Mimarisi

Şemada tasarım beş ana işlevsel blok halinde görülür:

```text
┌─────────────┐
│     MCU     │  RP2040-Zero
└──────┬──────┘
       │
       ├──────────────► Gear step motor sürücüsü
       │
       ├──────────────► Selector step motor sürücüsü
       │
       ├──────────────► Servo
       ├──────────────► Encoder
       ├──────────────► Endstop
       ├──────────────► Extra / toolhead girişi
       └──────────────► LED çıkışı

VIN
 │
 ├────────────────────► TMC2209 motor beslemesi
 │
 └──► 5 V buck katı ──► +5 V çevre birimleri / denetleyici besleme yolu
```

PCB, RP2040 ve TMC2209 entegrelerini doğrudan karta yerleştirmek yerine modüler denetleyici ve step motor sürücü kartları için taşıyıcı/ara bağlantı kartı görevi görür.

## RP2040-Zero Denetleyici

Güncel şemada **U1: RP2040-ZERO** kullanılmaktadır.

Denetleyiciye bağlı başlıca sinyaller:

- `G-EN`, `G-STEP`, `G-DIR`;
- `S-EN`, `S-STEP`, `S-DIR`;
- `G-UART`, `S-UART`;
- `G-DIAG`, `S-DIAG`;
- `G-INDEX`, `S-INDEX`;
- `SERVO`;
- `ENCODER`;
- `ENDSTOP`;
- `EXTRA`;
- `LED`.

Depoda bu sinyalleri belirli bir Klipper veya Marlin çalışma yapılandırmasına bağlayan firmware/config dosyası bulunmadığından yazılım pin eşlemeleri ve firmware davranışı burada varsayılmamıştır.

## Step Motor Sürücü Kanalları

Kartta iki TMC2209 SilentStepStick uyumlu sürücü modülü yuvası bulunur.

### Gear Sürücü — U3

Gear kanalı şu sinyalleri kullanır:

- `G-STEP`;
- `G-DIR`;
- `G-EN`;
- `G-UART`;
- `G-DIAG`;
- `G-INDEX`.

Motor fazları dört pinli **J7** konnektörüne yönlendirilmiştir.

### Selector Sürücü — U4

Selector kanalı şu sinyalleri kullanır:

- `S-STEP`;
- `S-DIR`;
- `S-EN`;
- `S-UART`;
- `S-DIAG`;
- `S-INDEX`.

Motor fazları dört pinli **J8** konnektörüne yönlendirilmiştir.

Her sürücü yuvasında VIN hattında yerel bulk kapasitör ve kontrol sinyalleri çevresinde gerekli direnç/pull-up elemanları bulunur.

## TMC2209 UART Yönlendirmesi

PCB üzerinde iki TMC2209 modülüyle ilişkili dört solder jumper bulunur:

- **JP7 / JP8** — gear sürücüsünün UART ilişkili yönlendirmesi;
- **JP9 / JP10** — selector sürücüsünün UART ilişkili yönlendirmesi.

Güncel PCB/netlist'e göre:

- JP7 ve JP9 normalde açık solder jumper olarak tanımlıdır;
- JP8 ve JP10 köprülenmiş/kapalı solder jumper olarak tanımlıdır.

Bu jumper'lar TMC modülünün UART/kontrol yönlendirmesinin bir parçasıdır. Varsayılan PCB durumu değiştirilmeden önce kullanılacak firmware ve TMC2209 modül varyantına göre bağlantı doğrulanmalıdır.

## Güç Mimarisi

### Ana Güç Girişi

**J6**, iki pinli ana güç konnektörüdür:

| J6 pini | Hat |
|---|---|
| 1 | GND |
| 2 | VIN |

VIN hem step motor sürücülerinin motor besleme alanına hem de kart üzerindeki DC/DC katına gider.

Depoda monte edilmiş kart için doğrulanmış bir çalışma giriş gerilimi aralığı belirtilmemiştir. Bu nedenle güvenli besleme aralığı yalnızca konnektör adından tahmin edilmemeli; kullanılan gerçek komponentler ve hedef montaj dikkate alınarak belirlenmelidir.

### 5 V Buck Dönüştürücü

Yerel 5 V besleme **U2: LM2596S-ADJ** etrafında kurulmuştur.

Güç katında:

- **L1:** 47 µH;
- **D1:** SS34 Schottky diyot;
- **RV1:** 10 kΩ ayar potansiyometresi;
- **R5:** 1 kΩ feedback direnci;
- giriş/çıkış filtreleme ve decoupling kapasitörleri

bulunur.

Bu ayarlanabilir bir regülatör topolojisidir. 5 V çevre birimleri bağlanmadan önce monte edilmiş kartın çıkış gerilimi ayarlanmalı ve ölçülmelidir.

### Besleme Hatları

Tasarımda üç temel besleme hattı vardır:

- `VIN` — giriş/motor beslemesi;
- `+5V` — buck regülatörden gelen çevre birimi/denetleyici hattı;
- `+3V3` — lojik/sensör hattı.

## Çevre Birimi Konnektörleri

### J1 — Servo

| Pin | Sinyal |
|---|---|
| 1 | SERVO |
| 2 | GND |
| 3 | +5V |

### J2 — Encoder

| Pin | Sinyal |
|---|---|
| 1 | ENCODER |
| 2 | GND |
| 3 | +3V3 |

### J3 — Extra / Toolhead

| Pin | Sinyal |
|---|---|
| 1 | EXTRA |
| 2 | GND |
| 3 | +3V3 |

### J4 — Endstop

| Pin | Sinyal |
|---|---|
| 1 | ENDSTOP |
| 2 | GND |
| 3 | +3V3 |

### J5 — LED

| Pin | Sinyal |
|---|---|
| 1 | LED |
| 2 | GND |
| 3 | +5V |

### J7 / J8 — Step Motorlar

J7 gear sürücünün, J8 selector sürücünün motor çıkışlarına bağlıdır. Her konnektör TMC2209'un dört motor faz çıkışını taşır.

Bobin sırası bağlanan step motor ve kablo demetine göre değişebileceğinden motor beslemesi verilmeden önce faz çiftleri doğrulanmalıdır.

## Sensör ve Diagnostic Yolları

Kart yalnızca temel STEP/DIR kontrolünü sunmaz.

İki TMC kanalı için ayrı:

- UART;
- DIAG;
- INDEX

hatları bulunmaktadır.

Bu sayede TMC2209'un diagnostic/kontrol özellikleri donanım seviyesinde erişilebilir durumdadır. Bu sinyallerin sensorless homing, stall algılama, durum izleme veya başka bir işlev için kullanılması firmware seviyesinde verilecek bir karardır ve yalnızca bu depo tarafından tanımlanmaz.

Ayrı `ENDSTOP` ve `EXTRA` girişleri, TMC diagnostic hatlarından bağımsız geleneksel harici sensör bağlantılarına da imkân verir.

## LED Çıkışı

J5 şu hatları sağlar:

```text
LED sinyali
GND
+5 V
```

Şema elektriksel konnektörü ve MCU sinyalini tanımlar; ancak kullanılacak LED protokolünü belirleyen firmware bu depoda bulunmaz. Bu nedenle firmware tanımlanana kadar çıkışın NeoPixel, PWM veya başka belirli bir protokol kullandığı varsayılmamalıdır.

## Mekanik Tasarım

PCB üzerinde **üç M3 montaj deliği** bulunur:

- H1;
- H2;
- H3.

Depoda kartın tamamına ait dışa aktarılmış:

```text
Board.step
```

3D modeli ve şu klasör altında komponent STEP modelleri bulunur:

```text
peacock_tail_brd.pretty/packages3d/
```

Bu dosyalar KiCad projesini açmadan kasa tasarımı ve mekanik açıklık kontrolleri yapılabilmesini sağlar.

## KiCad Tasarım Dosyaları

Güncel proje modern KiCad dosyalarını kullanır:

- `peacock_tail_brd.kicad_sch` — şema;
- `peacock_tail_brd.kicad_pcb` — PCB yerleşimi;
- `peacock_tail_brd.kicad_pro` — proje ayarları.

Şema içindeki proje bilgileri:

```text
Oluşturan KiCad sürümü : 10.0
Proje başlığı           : PEACOCK TAIL BRD
Revizyon                : 0.1
Tarih                   : 2026-05-06
```

Eski `.sch`, `.pro`, `.lib` ve `.dcm` dosyaları da depoda tutulmaktadır. Bu README'de güncel donanım tanımı için `.kicad_*` dosyaları esas alınmıştır.

## Özel Kütüphaneler ve 3D Modeller

`peacock_tail_brd.pretty/` proje özelinde kullanılan footprint'leri içerir. Bunların arasında:

- TMC2209 SilentStepStick modül footprint'i;
- proje geçmişinden kalan RP2040/Seeeduino tipi modül footprint varlıkları;
- özel pin-header ve terminal-block footprint'leri

bulunur.

`packages3d/` klasöründe TMC2209 modülleri, denetleyici modülü, JST konnektörler, jumper'lar ve vidalı terminaller için modeller bulunur.

Bazı kütüphane dosyalarının adları tasarımın önceki iterasyonlarını yansıtmaktadır. Bu README'deki komponent tanımları tarihsel footprint/model dosya adlarına değil, **güncel şemada kullanılan komponentlere** göre yapılmıştır.

## Üretim Dosyaları

Depoda:

```text
production/PEACOCK_TAIL_BRD_v0.1.zip
```

dosyasının yanı sıra tarihli yedek arşivler ve `production/netlist.ipc` bulunur.

Üretim arşivinin bulunması revizyon 0.1 için fabrikasyon çıktılarının oluşturulduğunu gösterir. Ancak tek başına bu dosyanın varlığı kartın elektriksel doğrulamadan, montaj doğrulamasından veya uzun süreli yazıcı testinden geçtiği anlamına gelmez.

## 3D Yazıcı Firmware Entegrasyonu

Donanım MMU tipi hareket/kontrol yapısına göre düzenlenmiştir ve harici firmware'in ihtiyaç duyacağı sinyalleri dışarı çıkarır.

Projenin önceki açıklamasında Klipper ve Marlin hedef entegrasyonlar olarak belirtilse de bu depoda şu anda yalnızca **PCB tasarım varlıkları** bulunmaktadır. Şunlar mevcut değildir:

- Klipper MCU yapılandırması;
- Klipper `printer.cfg` örneği;
- Happy Hare yapılandırması;
- Marlin kart tanımı;
- RP2040 firmware kaynak kodu;
- test edilmiş yazılım pin haritası.

Bu nedenle README, belirli bir firmware yapılandırmasıyla doğrudan uyumluluk iddiasında bulunmadan donanım arayüzünü belgeler.

## İlk Çalıştırma Kontrol Listesi

Yeni monte edilmiş karta enerji vermeden önce:

1. PCB üzerinde lehim köprüsü ve montaj hatası kontrolü yapın;
2. VIN, +5 V, +3,3 V ve GND arasında kısa devre olmadığını doğrulayın;
3. mümkünse hassas çevre birimleri bağlı değilken regülatör katını enerjilendirin;
4. 5 V yükleri bağlamadan önce LM2596 çıkışını ayarlayıp ölçün;
5. denetleyicinin 3,3 V hattını doğrulayın;
6. TMC2209 modüllerini takmadan önce yönlerini kontrol edin;
7. J7/J8 üzerindeki step motor bobin çiftlerini doğrulayın;
8. kullanılacak solder-jumper yapılandırmasını doğrulayın;
9. sensör/servo/LED bağlantılarından önce gerekli gerilim ve sinyal tiplerini kontrol edin;
10. motor testlerine düşük ve güvenli sürücü akım ayarlarıyla başlayın.

## Mevcut Proje Durumu

Depo projeyi **geliştirme aşamasında** olarak tanımlar ve mevcut tasarım varlıklarına göre bu durum hâlâ uygundur.

Depoda doğrulanabilen mevcut içerik:

- revizyon 0.1 şema ve PCB;
- RP2040-Zero denetleyici tasarımı;
- iki TMC2209 sürücü kanalı;
- 5 V buck güç katı;
- çevre birimi ve sensör konnektörleri;
- özel footprint'ler ve 3D modeller;
- tam kart STEP çıktısı;
- revizyon 0.1 üretim paketi.

Depoda bulunmayanlar da önemlidir: commit edilmiş firmware, doğrulama raporu, montaj kılavuzu, BOM, elektriksel test kaydı veya belgelenmiş üretim kalifikasyonu bulunmamaktadır.

## Depo Yapısı

```text
peacock_tail_brd/
├── peacock_tail_brd.kicad_sch
├── peacock_tail_brd.kicad_pcb
├── peacock_tail_brd.kicad_pro
├── Board.step
├── A4_ISO5457-1999_ISO7200-2004-OpenEYE.kicad_wks
├── fabrication-toolkit-options.json
├── fp-lib-table
├── sym-lib-table
├── peacock_tail_brd.pretty/
│   ├── *.kicad_mod
│   └── packages3d/
│       └── *.step / *.STEP
├── production/
│   ├── PEACOCK_TAIL_BRD_v0.1.zip
│   ├── netlist.ipc
│   └── backups/
├── eski KiCad dosyaları
└── README.md / README-tr.md
```

## Lisans

Depoda şu anda proje lisansı tanımlanmamıştır. Bir lisans eklenene kadar geçerli telif hakkı mevzuatının verdiği izinlerin ötesinde kullanım hakkı bulunduğu varsayılmamalıdır.
