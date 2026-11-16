# Unitree R1-7A Ayrıntılı Kurulum ve Kullanım Rehberi

[Hızlı başlangıca dön](../README.md) · Kaynak: [unitreerobotics/R1-7A `detail.md`](https://github.com/unitreerobotics/R1-7A/blob/main/detail.md) · Türkçe çeviri: [Robotlar.org](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a)

Bu belge ilk kurulumu, cihaz yapılandırmasını, kalibrasyonu, kayıt ve tekrar oynatmayı ve sorun gidermeyi anlatır. Hızlı kurulum için depoyu klonlayıp `bash setup.sh` çalıştırmak yeterlidir; belirli bir yapılandırma sorunuyla karşılaştığınızda ilgili bölüme bakın.

Donanımın adı her yerde **R1-7A** olarak geçer. Koddaki `r1arm` betik yolları, `R1ARM_*` ortam değişkenleri ve `r1_teleop` Conda ortam adı mevcut uygulamayla uyumlu kalır.

> Bu rehberdeki tüm komutlar, orijinal [unitreerobotics/R1-7A](https://github.com/unitreerobotics/R1-7A) deposunun kök dizininde çalıştırılır.

## 📥 Ortam Kurulumu

### 1. Depoyu klonlama

Bu belge sıfırdan kurulan bir makine için yazılmıştır. Ubuntu 22.04 x86_64 kullanın; önce Git, Conda ve sistem Python 3'ü kurun, ardından:

```bash
git clone --recurse-submodules https://github.com/unitreerobotics/R1-7A.git
cd R1-7A
```

Kod zaten var ama bağımlılıklar başlatılmamışsa:

```bash
git submodule update --init --recursive
```

### 2. Bağımlılıkları kurma ve derleme

Proje bağımlılıklarını kurun ve derlemeyi tamamlayın:

```bash
bash setup.sh
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate r1_teleop
export LD_LIBRARY_PATH="$CONDA_PREFIX/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
```

`setup.sh`, `r1_teleop` ortamını oluşturur ya da yeniden kullanır; sabit sürümlü Placo / Pinocchio / NumPy birleşimini, Python DDS istemcisini ve XR bağlamasını kurar; kol SDK'sını, bridge'i ve Dex1 servisini derler. **Kurulum motorlara bağlanmaz, motorları etkinleştirmez ve fiziksel cihaz kalibrasyonu üretmez.**

| Seçenek | Kullanım |
|---|---|
| `bash setup.sh --skip-apt` | Sistem bağımlılıkları zaten kuruluysa apt adımını atlar; sonraki kurulum ve derleme yine yapılır |
| `bash setup.sh --skip-xrt` | XR bağımlılıklarını atlar, önce mock simülasyonu denemek içindir; gerçek PICO teleoperasyonu için XR yine gerekir |
| `ENV_NAME=r1_teleop bash setup.sh` | Ortam adını belirler; adı değiştirirseniz tutucu başlatılırken `DEX1_CONDA_ENV` değişkenini de aynı ada ayarlayın |

Çekirdek sürümler `placo==0.9.23`, `pin==3.8.0`, `coal==3.0.2`, `numpy==2.2.6`'dır; tam birleşim için betiğe bakın. Betik varsayılan olarak Conda-forge ve Alibaba Cloud PyPI aynasını kullanır; `CONDA_FORGE` ve `PIP_INDEX` ile değiştirilebilir. Türkiye'den kurulumda yavaşlık görürseniz `PIP_INDEX=https://pypi.org/simple` ile resmi PyPI'ya geçebilirsiniz (çevirmen notu).

Üçüncü taraf bağımlılıklar submodule ile sabit sürüme kilitlenir ve üzerine [native/dependency_patches](BAGIMLILIK-YAMALARI.md) içindeki uyarlamalar uygulanır. Yalnız üst kaynak kodunu klonlayıp yamaları uygulamamak, bu deponun kullandığı temel hatla aynı değildir. Yama çakışırsa kurulum durur; çalışma ağacını kontrol edin, yerel değişiklikleri zorla ezmeyin.

## 🥽 XRoboToolkit Yapılandırması

Python bağlaması, masaüstü PC servisi ve başlık uygulaması üç ayrı parçadır:

| Parça | Nasıl hazırlanır |
|---|---|
| Python SDK / bağlama | `setup.sh` kurar, teleoperasyon süreci kullanır |
| PC Service | [XRoboToolkit-PC-Service](https://github.com/XR-Robotics/XRoboToolkit-PC-Service) kurulum talimatıyla ayrıca kurulur ve başlatılır; setup tam masaüstü servisini kurmaz |
| PICO istemcisi | [XRoboToolkit-Unity-Client](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client) talimatıyla başlığa kurulur; Unitree'nin test makinesinde `XRoboToolkit-PICO-1.1.1.apk` kullanılmıştır |

1. PC ile PICO birbirine ağ üzerinden ulaşabilmeli; misafir ağı ya da AP istemci yalıtımı olan ağlardan kaçının.
2. PC'de bir PC Service açın, PICO uygulamasına PC'nin yerel ağ IP adresini girip bağlanın.
3. Başlık uygulamasında kumanda verisi gönderimini açın, uygulamayı ön planda ve başlığı uyanık tutun, iki kumandayı hareket ettirip verinin sürekli güncellendiğini kontrol edin.
4. Gerçek kola geçmeden önce sol ve sağ kumanda girişini simülasyonda doğrulayın. İstemcide A tuşu "gönderimi duraklat" kısayolu olarak atanmışsa, bu projedeki "A ile home'a dön" işleviyle çakışmasını giderin.

Python istemcisi yerel makinedeki `127.0.0.1:60061` adresine bağlanır. `Connected to XRoboToolkit` mesajı yalnız SDK bağlantısının kurulduğunu gösterir; zaman damgası 0 ise, hiç değişmiyorsa ya da giriş eskimiş (stale) görünüyorsa normal izleme yine yapılamaz.

Aynı makinedeki tutucu DDS trafiği varsayılan olarak `lo` arayüzünü kullanır ve başlığın Wi-Fi ayarından ayrı yapılandırılır. Python bağlamasını değiştirdikten sonra Python sürecini, PC Service kitaplıklarını değiştirdikten sonra masaüstü servisini yeniden başlatın. Birden çok eski XR örnekleyiciyi ya da denetleyiciyi aynı anda çalıştırmayın.

## ⚙️ Cihaz Yapılandırması ve Kalibrasyon

### 1. Seri portları ve izinleri doğrulama

```bash
ls -l /dev/serial/by-id/
id -nG
```

Kararlı `by-id` yollarının kullanılması önerilir. USB yeniden takıldığında `/dev/ttyUSB0` gibi numaralar değişebilir; **adaptör değiştirilirse by-id içindeki seri numarası da değişir**.

Bu projedeki dört kanallı FTDI için referans kablolama aşağıdadır ve fiziksel cihazla mutlaka karşılaştırılmalıdır:

| Cihaz | Arayüz soneki | Varsayılan motor ID / kullanım |
|---|---|---|
| Sağ tutucu | `if00-port0` | ID7, DDS `right` |
| Sağ kol | `if01-port0` | ID0-6, sağ kolun yedi ekseni |
| Sol tutucu | `if02-port0` | ID7, DDS `left` |
| Sol kol | `if03-port0` | ID0-6, sol kolun yedi ekseni |

Unitree test makinesindeki adaptörün seri numarası `FTANPO49`, kodun uyumlu kaldığı eski varsayılan ise `FTBPG771`'dir. İkisi de başka makinelerin kopyalaması gereken cihaz kimlikleri değildir.

İzin yetersizse:

```bash
sudo usermod -aG dialout "$USER"
```

Oturumu kapatıp yeniden açın ya da kullanacağınız terminalde `newgrp dialout` çalıştırın; ardından Conda'yı yeniden etkinleştirip cihaz yapılandırmasını yükleyin. `getent group dialout` çıktısında kullanıcı adının görünmesi, eski terminallerde grup izninin geçerli olduğu anlamına gelmez.

### 2. Makine yapılandırmasını kaydetme

Tüm komutlar depo kökünden çalıştırılır. Aşağıdaki içeriği kök dizinde `.env` olarak kaydedin; **önce adaptör önekini ve tutucu uç nokta dosyasının yolunu değiştirin**. `.env` Git tarafından yok sayılır ve makine yapılandırmasını tutmak içindir.

```bash
# setup.sh ile derlenen depo içi bağımlılıkları kullan.
export R1ARM_SDK_ROOT="$PWD/third_party/r1_arm_sdk"
export R1ARM_BRIDGE_LIB="$PWD/build/r1arm_sdk_bridge/libr1arm_sdk_bridge.so"
export DEX1_SERVICE_ROOT="$PWD/third_party/dex1_1_service"
export UNITREE_SDK2_ROOT="$PWD/third_party/unitree_sdk2"

# Örnek, Unitree test makinesinin adaptörüdür; başka makinede ls /dev/serial/by-id/ çıktısına göre değiştirin.
export R1ARM_SERIAL_PREFIX="/dev/serial/by-id/usb-FTDI_USB__-__Serial_Converter_FTANPO49-"
export PORT_RIGHT="${R1ARM_SERIAL_PREFIX}if01-port0"
export PORT_LEFT="${R1ARM_SERIAL_PREFIX}if03-port0"
export RIGHT_DEX1_PORT="${R1ARM_SERIAL_PREFIX}if00-port0"
export LEFT_DEX1_PORT="${R1ARM_SERIAL_PREFIX}if02-port0"
export PORT="$PORT_RIGHT"

export DEX1_NETWORK=lo
export DEX1_ENDPOINTS_FILE=/absolute/path/to/your/dex1_endpoints.json
```

Yukarıdaki yollar `setup.sh` ile kurulan depo içi bağımlılıklara karşılık gelir. Seri port önekini ve tutucu uç nokta yolunu cihazınızın gerçek yapılandırmasıyla değiştirin; mevcut kalibrasyonlar ait oldukları fiziksel kolla eşleşik kalmalıdır.

**Her yeni terminalde** çalıştırın:

```bash
source "$(conda info --base)/etc/profile.d/conda.sh"
conda activate r1_teleop
source .env
source scripts/real_env.sh
```

`real_env.sh` yalnız değişkenleri ayarlar, dosyaları ve izinleri kontrol eder; motorları açmaz. Gereken seri portların `Accessible` göründüğünü ve `Calibration not readable` mesajı olmadığını görmelisiniz. Betik `.env` dosyasını kendiliğinden yüklemez, önce elle `source` etmeniz gerekir.

| Ayar | Güncel davranış |
|---|---|
| `R1ARM_SDK_ROOT` | Önce belirtilmelidir; belirtilmezse ortam betiği proje dizininin yanındaki `R1_arm_SDK_x86/r1_arm` yolunu arar, setup'ın derlediği depo içi SDK'yı değil |
| `R1ARM_BRIDGE_LIB` | Açıkça belirtilmesi önerilir; aksi halde betik önce mevcut `build/dual_sdk_bridge`, sonra standart `build/r1arm_sdk_bridge` yolunu seçer |
| `R1ARM_SERIAL_PREFIX` | Ortam betiği ve `dual/` varsayılan portları ile rol kontrolleri bu öneki ortak kullanır |
| `PORT_RIGHT` / `PORT_LEFT` | Ortam değişkenleri kontrol içindir; çift kol girişi portları varsayılan olarak önekten üretir, kablolamayı tek tek değiştirirseniz `--right-port/--left-port` verin |
| `PORT` | Tek kol girişinin varsayılan seri portu |
| `CALIB_RIGHT` / `CALIB_LEFT` | Varsayılan olarak SDK içindeki `config/joint_calib_zero_右臂.txt` / `joint_calib_zero_左臂.txt` dosyalarını gösterir (右臂 = sağ kol, 左臂 = sol kol) |
| `DEX1_SERVICE_ROOT` / `DEX1_SERVER` | Servis dizini ya da çalıştırılabilir dosya; varsayılan servis dizin içindeki `bin/dex1_1_gripper_server` |
| `UNITREE_SDK2_ROOT` | Tutucu servisinin kullandığı C++ DDS bağımlılığının yolu |
| `DEX1_NETWORK` | Aynı makinede varsayılan `lo`; servis ve Python istemcisi aynı olmalı, varsayılan domain 0 |
| `DEX1_ENDPOINTS_FILE` | Mevcut fiziksel tutucunun taraf bazlı uç nokta JSON'u; ayarlanmazsa depodaki varsayılan dosya kullanılır |

Açık terminallerde eski SDK, seri port ya da bridge değişkenleri kalmış olabilir; güncellenmiş `.env` ve `real_env.sh` dosyalarını yeniden `source` edin. Öneki değiştirirken dört portu ve `PORT` değişkenini de güncelleyin; yalnız öneki değiştirmek kabukta zaten tanımlı port değişkenlerinin üzerine yazmaz.

### 3. Kol kalibrasyonu

Kol kalibrasyon biçimi `joint dir scale offset` şeklindedir, eklem numaraları 0-6 arasıdır:

```text
calibrated_q = dir × scale × raw_motor_q + offset
```

Mevcut fiziksel kol ve enkoder durumuyla eşleşen bir kalibrasyonunuz varsa doğrudan kullanmaya devam edebilirsiniz. Depo kodu, kurulum betiği ya da seri adaptör değişimi fiziksel kalibrasyonun yerini tutmaz; yalnız USB yeniden bağlandı diye otomatik yeniden kalibrasyon yapmayın.

Varsayılan dosya konumları:

```text
$R1ARM_SDK_ROOT/config/joint_calib_zero_右臂.txt
$R1ARM_SDK_ROOT/config/joint_calib_zero_左臂.txt
```

Farklı yolları, eski kabuktaki `CALIB` takma adına güvenmeden komut satırından verin:

```bash
# Tek kol
python scripts/teleop.py --right --motion-mode --calib /absolute/path/right.txt

# Çift kol; ilk doğrulamada kameraları kapalı tutun
python scripts/teleop.py --dual --motion-mode \
  --right-calib /absolute/path/right.txt --left-calib /absolute/path/left.txt
```

Sol ve sağ kalibrasyonlar bağımsızdır; sol model varsayılan olarak J2/J6 için `mirrored`, J4 için `model` kullanır. Fiziksel kalibrasyon ile model yön dönüşümü ayrıdır; sıfır tablosunda işareti ikinci kez çevirmeyin. Home konumu ortak model koordinatında `0,1.2,0,-1.2,0,0,0`'dır, yedi eksenin mekanik sıfırı değildir.

İlk kez kullanılabilir kalibrasyon yoksa kol SDK'sının [kalibrasyon aracı açıklamasına](https://github.com/unitreerobotics/r1-7a_sdk) göre gerçek referans pozu ve her eksenin yönünü ve ölçeğini doğrulayın, `calibrate_joints` ile bu makineye özel dosyayı üretin. SDK aracı gerçek motorlara bağlanır; kolu destekleyin ve `--out` ile yeni bir dosyaya yazın. Kolu rastgele bir pozdayken sıfır olarak kaydetmeyin.

**Yönü ve ölçeği zaten doğru olan bir kalibrasyon** için fiziksel tam sıfırı yeniden yakalamak isterseniz bu projenin aracını kullanabilirsiniz. Aşağıdaki ilk komut yalnız dosyayı kontrol eder; `--capture` eklendiğinde seri port açılır ve sıfır kazançlı yoklama gönderilir. Yedi ekseni elle tam fiziksel sıfıra getirip onaylamanız gerekir:

```bash
python dual/calibrate.py --side right --bridge-lib "$R1ARM_BRIDGE_LIB"
python dual/calibrate.py --side right --bridge-lib "$R1ARM_BRIDGE_LIB" --capture
```

Sol kol için `--side right` yerine `--side left` yazın. Araç varsayılan olarak ayrı bir aday dosya ve örnek kaydı üretir; yalnız açıkça `--install` eklenirse hedef dosyayı yedekleyip değiştirir. Araç mevcut bir kalibrasyon gerektirir, ilk yön ve ölçek doğrulamasının yerini tutmaz. Eski varsayılan yolu standart derleme dizininden farklı olduğu için `--bridge-lib` değerini açıkça verin.

Açı sınır dışına çıkarsa önce mekanik pozu, kalibrasyon dosyasını, sol/sağ tarafı ve enkoder referansını kontrol edin. Program açılış kontrollerini korur; hatalı durumu geçirmek için sınırları gevşetmeyin, zorla mod almayın ve aday dosyayı otomatik benimsemeyin.

### 4. Tutucu uç noktaları ve servis

Sol ve sağ motorlar ayrı hatlarda olduğunda ikisi de ID7 olabilir; sol/sağ topic'i DDS `side` belirler, ID7'den otomatik çıkarılmaz. Gerçek motor ID'leri farklıysa çift taraflı servisi başlatırken `--right-id/--left-id` kullanın; ID15 yayın (broadcast) adresidir, yoklama için kullanılmaz.

Uç nokta JSON yapısı:

```json
{
  "source": "Bu makinede taraf bazlı ölçüm kaydı",
  "right": {"closed": -4.4258, "open": 0.8784},
  "left": {"closed": -0.4819, "open": 4.8166}
}
```

Yukarıdaki değerler yalnız biçimi gösterir, **genel geçer bir kalibrasyon değildir**. Birim motor radyanıdır; değerler sonlu olmalı ve `open > closed` sağlanmalıdır. Tutucu değiştiğinde, enkoder referansı değiştiğinde ya da güncel geri bildirim açıkça uç nokta aralığının dışındaysa yeniden kontrol edin. Belirtilen dosya yoksa ya da içeriği geçersizse hata verilir, varsayılana geri dönülmez.

Uç noktaları ölçmek gerekirse önce ilgili tutucu servisini başlatın, tüm teleoperasyon, tekrar oynatma ve diğer komut yayınlayıcılarından çıkın, ardından:

```bash
python scripts/capture_dex1_endpoints.py --network lo
```

Aracın yönergelerine göre her tarafı elle yavaşça kapalı ve açık uç noktaya getirin; fren direnci hissederseniz zorlamayın. Araç yalnız geri bildirime abone olur, `data/diagnostics/.../endpoints.json` dosyasını üretir ve otomatik kurulum yapmaz. Kontrol ettikten sonra `.env` içindeki `DEX1_ENDPOINTS_FILE` değişkenini bu dosyaya yönlendirin ve tüm terminallerde yeniden yükleyin. Uç noktaları ve kol kalibrasyonunu ayrıca yedekleyin; silinebilecek build dizinine bağlı bırakmayın.

## 🧪 Simülasyon ve Açılış Öncesi Kontroller

Gerçek kola bağlanmadan önce Conda ortamında mock simülasyonu kontrol edin:

```bash
python scripts/teleop.py --sim --right --mock-xr
python scripts/teleop.py --sim --dual --mock-xr
```

Gerçek PICO girişini kontrol etmek için `--mock-xr` bayrağını kaldırın. Donanımsız çift kol kontrol akışı kısa süreli de çalıştırılabilir:

```bash
python scripts/teleop.py --dual --mock-xr --no-cameras --duration 3
```

Bu yalnız yazılım akışını doğrular; fiziksel güç, seri port yanıtı, kalibrasyon ya da gerçek hareketin doğruluğunu kanıtlamaz. `--mock-xr`, `--motion-mode` ile birlikte kullanılamaz.

Gerçek donanıma geçmeden önce şunları doğrulayın: seri port rolleri doğru ve okunup yazılabilir, sol/sağ kalibrasyon fiziksel kollarla eşleşik, bridge yüklenebiliyor; tutucu servisi her iki tarafı da tanıyor; XR pozu ve zaman damgası sürekli güncelleniyor. Kontrol sırasında ilgili seri portu kullanan tek bir program setini çalıştırın.

## 🛠️ Teleoperasyon

### 0. Ortak komut kuralları

```bash
python scripts/teleop.py --right --motion-mode
python scripts/teleop.py --left --motion-mode
python scripts/teleop.py --dual --motion-mode
python scripts/teleop.py --left --record --motion-mode --no-cameras --output-dir data/left_real
```

`--right`, `--left`, `--dual` seçeneklerinden yalnız biri seçilmelidir. `--record` kaydı açar; eklenmezse kayıt iş parçacığı, kamera ya da çıktı dosyası hiç oluşturulmaz. Tek kol 8, çift kol 16 boyutludur; sol kol meta verisinde `control_side=left` saklanır. Tek kol gerçek donanım tekrarı kaydedilen tarafı otomatik okur ve ilgili kalibrasyonu, seri portu ve sol kol koordinat dönüşümünü kullanır; kayıtla uyuşmayan bir `--side` açıkça verilirse oynatma reddedilir.

Varsayılan olarak X başlatır/kaydeder, B duraklatır/sürdürür, A home'a döndürür. `--auto-start` yalnız `--record` ile kullanılır ve açılış başarılı olunca hemen bir episode başlatır; gerçek donanımda yine `--motion-mode` ve MOVE onayı gerekir. `--duration N` gerçek donanım modunda programın çalışma süresini sınırlar; çıkışta önce durdurur, sonra dosya yazmayı tamamlar.

Simülasyon için `--sim` ekleyin; üç kol seçimi ve kayıt anahtarı aynı kalır:

```bash
python scripts/teleop.py --sim --left --mock-xr --no-viewer --duration 3
python scripts/teleop.py --sim --dual --record --mock-xr --no-viewer \
  --auto-start --duration 1 --exit-after-duration --output-dir data/sim_dual
```

`python scripts/teleop.py --sim --help` simülasyona özel parametreleri gösterir. Simülasyon varsayılan olarak 50 Hz kontrol hızı kullanır; `--exit-after-duration`, belirtilen kayıt süresine ulaşıp episode'u kaydettikten sonra çıkar.

Aşağıdaki komutların tümü depo kökünden çalıştırılır; her terminalde önce `.env` ve `real_env.sh` yüklenmiş olmalıdır.

### 1. Terminal 1: Tutucuları başlatma

Yalnız sağ kol için:

```bash
bash scripts/start_gripper.sh
```

Çift kol için:

```bash
bash dual/start_grippers.sh
```

Birini seçin ve terminali açık tutun. İlgili taraf için `Detected motors` ve `Dex1-1 Gripper Server started.` mesajlarını görmelisiniz; çift kolda sol ve sağ için birer kez. Yalnız `Checking Dex1 port` ya da `Official Dex1` yazılması tanımanın başarılı olduğu anlamına gelmez.

Başlatıcı port kullanımını kontrol eder, çalışan servisleri kendiliğinden sonlandırmaz; yönetilen servislerden biri çıkarsa diğer tarafı da durdurur. Tek/çift kol arasında geçiş yapmadan önce eski servisi Ctrl+C ile kapatın. Cihaz yeni açıldıysa ya da yeniden bağlandıysa hazır olmasını bekleyin; yanıt gelmiyorsa güç, kablolama, ID ve portu kontrol edin, servisi tekrar tekrar paralel başlatmayın.

### 2. Terminal 2: Kol teleoperasyonunu başlatma

Sağ kol (tek kol):

```bash
python scripts/teleop.py --right --motion-mode
```

Çift kol; önce kameraya bağlı olmadan hareketi ve kumanda girişini doğrulayın:

```bash
python scripts/teleop.py --dual --motion-mode
```

Yazdırılan sol/sağ seri portları, kalibrasyonları, kazançları ve home uyarısını kontrol edin; destek, acil durdurma ve çalışma alanını doğruladıktan sonra `MOVE` yazın. **Şu an hem tek hem çift kol varsayılan olarak home'a gider**; konuma ulaştıktan sonra göreli izleme için grip'e basın. Çift kolda önce iki tutucunun geri bildirimi beklenir; önce hazır olan kol ölçülen pozunu korur, diğer taraf hazır olunca home hareketi başlar.

- Açılışta home'a gitmesini istemiyorsanız `--no-home-on-start` ekleyin; motorlar mevcut pozu korur, açılış kontrolleri yine geçerlidir.
- Tutucu yoksa `--no-enable-dex1` ekleyin ve tutucu servisini atlayın; bu seçenek kol kontrollerini gevşetmez.
- Yalnız `--record` eklendiğinde kamera istenir: tek kolda iki, çift kolda üç akış; `--no-cameras` görüntüyü kapatır. Salt teleoperasyon kamera açmaz.
- Çift kolda kontrol sürelerini görmek için `--timing-report` eklenebilir; 90 Hz hedefi, fiziksel iletişimin gerçekten 90 Hz'e ulaştığı anlamına gelmez.

### 3. Kumanda düğmeleri

| Giriş | Davranış |
|---|---|
| Sol / sağ grip | Basılı tutulunca ilgili kol göreli pozu izler; tek kolda seçilen tarafın kumandası kullanılır |
| Sol / sağ tetik | İlgili tutucunun kapanma oranını kontrol eder |
| A (sağ kumanda) | Yumuşakça home'a döner; çift kolda iki kol birlikte |
| X (sol kumanda) | Kaydı başlatır / durdurup kaydeder; yalnız --record ile etkindir |
| B (sağ kumanda) | Kayıt sırasında kaydı ve izlemeyi duraklatır / sürdürür; duraklatma yeni bir `record_segment` oluşturur |
| Ctrl+C | Çıkar ve kontrolü durdurur; çıkıştan sonra kolu desteklemeye devam edin |

Üç modda da X/B kaydı için `--record` gerekir. Bir episode'u kaydettikten sonra çıkmadan önce kaydın tamamlandığı mesajını bekleyin. Varsayılan olarak X sol kumandada, B ve A sağ kumandadadır.

Güncel XR korumaları geçerlilik, sıçrama ve tazelik kontrollerini içerir; 250 ms'den uzun süre güncelleme gelmezse durur/tutar, kararlılık dönünce yeniden çapa (anchor) alır ve kesinti sırasındaki yer değiştirmeyi sonradan telafi etmez. Eski cutoff / beta filtre parametreleri ya da poz tahmini etkin değildir.

## 📦 Veri Toplama

Kayıt girişi teleoperasyonu da içerdiği için önce çalışan denetleyiciden çıkın, tutucu servislerini açık tutun.

### 1. Kamerasız kayıt: önce hareket zincirini kontrol edin

```bash
# Sağ kol
python scripts/teleop.py --right --record --motion-mode --no-cameras \
  --output-dir data/right_real

# Çift kol
python scripts/teleop.py --dual --record --motion-mode --no-cameras --data-root data/dual_real
```

İki komuttan birini çalıştırın. X başlatır/kaydeder, B duraklatır/sürdürür, A home'a döndürür. Üç mod aynı kontrol ve kayıt uygulamasını kullanır; `--record` yoksa kayıt iş parçacığı ve kamera oluşturulmaz.

### 2. Görüntüyü açma

| Görüntü anahtarı | Varsayılan cihaz eşleşmesi | İstenen biçim |
|---|---|---|
| `color_0_head_left` | RealSense D405 | 640×480, 30 fps, pozlama 8000 µs, gain 16 |
| `color_2_wrist_left` | V4L2 / JR0001 | 640×480, 30 fps, MJPG; sol kol ve çift kolda kullanılır |
| `color_3_wrist_right` | V4L2 / JR0002 | 640×480, 30 fps, MJPG |

Varsayılan yapılandırma `teleop/local_camera_client.py` dosyasındadır; cihaz adlarını ve pozlamayı gerçek cihaza ve aydınlatmaya göre ayarlayın. `setup.sh` `pyrealsense2` paketini kurar; RealSense'in yine doğru bağlanması ve erişim izninin olması gerekir. Kameraları ayrı ayrı kontrol edin:

```bash
python scripts/verify_r1arm_right_cameras.py
python scripts/verify_r1arm_dual_cameras.py
```

İki doğrulama programını gerektikçe sırayla çalıştırın ve kayda geçmeden önce çıkın; aksi halde cihaz çakışması olur. Başarılı olunca:

```bash
# Tek kol, iki görüntü akışı; istenen tüm kameralar hazır olmalı
python scripts/teleop.py --right --record --motion-mode \
  --output-dir data/right_real

# Çift kol, üç görüntü akışı; varsayılan olarak tüm kameralar gerekir
python scripts/teleop.py --dual --record --motion-mode --data-root data/dual_real
```

Varsayılan kamera setinin tamamını değiştirmek için `--camera` tekrar tekrar verilebilir, örneğin `--camera 'key=color_3_wrist_right,backend=v4l2,match=JR0002,size=640x480,fps=30'`; söz dizimi için giriş noktasının `--help` çıktısına ve kamera uygulamasına bakın. Kamera önizlemesi için önce `scripts/view_r1arm_cameras.py` çalıştırılabilir; cihaz çakışmasını önlemek için kayıttan önce önizlemeden çıkın.

### 3. Çıktı biçimi

Dizinde sırayla `episode_0000.mcap` gibi dosyalar oluşur. Örnekler `data/right_real` / `data/dual_real` dizinlerini açıkça verir; dizin `--output-dir` ile belirtilir, `--data-root` eş anlamlı parametredir.

```text
Tek kol: seçilen tarafın J1-J7 eklemleri + ilgili tutucu = 8 boyut
Çift kol: sağ kol J1-J7 + sağ tutucu + sol kol J1-J7 + sol tutucu = 16 boyut
```

- `/r1arm/control_sample`: hedef, geri bildirim ve zamanlama bilgisi içeren yüksek frekanslı kontrol anlık görüntüsü.
- `/whole_body/frame`: gözlem ve görüntü, varsayılan hedef 30 Hz.
- Eklemler ortak model koordinatındadır; tutucu mantıksal slide konumu olarak kaydedilir ve doğrudan motor radyanı olarak gönderilemez.
- Duraklatmanın öncesi ve sonrası `record_segment` ile ayrılır; hareketler segmentler arasında doğrudan birleştirilemez.
- Kamera zaman damgası alım zamanıdır, pozlama anına eşit olduğu garanti edilmez; çift kolda seri portların paralel çalışması donanım senkronu anlamına gelmez.

Görüntü dosyası kontrolü ve önizleme:

```bash
python scripts/check_r1arm_images.py data/dual_real/episode_0000.mcap --preview
```

## 🔄 Yörünge Tekrarı

Tekrar oynatma PICO ya da canlı XR girişi gerektirmez. Gerçek donanımda tekrardan önce teleoperasyon/kayıt programından çıkın, gereken tutucu servislerini açık tutun; episode yolunu gerçek dosyanızla değiştirin.

### 1. Salt okunur kontrol

```bash
python scripts/replay/replay_r1arm_real.py --episode data/right_real/episode_0000.mcap
python dual/replay.py --episode data/dual_real/episode_0000.mcap
```

`--motion-mode` eklenmezse yalnız veri kontrol edilir; gerçek kol sürülmez ve gerçek donanım zaman çizelgesi oynatılmaz.

### 2. Gerçek donanımda tekrar

```bash
# Sağ kol
python scripts/replay/replay_r1arm_real.py \
  --episode data/right_real/episode_0000.mcap --motion-mode

# Çift kol
python dual/replay.py --episode data/dual_real/episode_0000.mcap --motion-mode
```

Varsayılan olarak yüksek frekanslı kontrol akışındaki şekillendirilmiş hedef `q_purpose` öncelikli kullanılır; eski kayıtlarda kontrol akışı yoksa gözlem akışına dönülür. `--section state` ölçülen yörüngeyi kullanır, `--speed 0.5` yavaş oynatır. Tekrar önce segmentin ilk karesine yaklaşır; duraklatma segmentleri ayrı işlenir, segmentler arasında doğrudan enterpolasyon yapılmaz.

Tek ve çift kol gerçek donanım tekrarı, varsayılan olarak ayrı bir süreçte kaydedilmiş görüntüleri eş zamanlı gösterir ve her karede eklem ve tutucu hedeflerini yazdırır. Tek kolda eklem alanı `q_motor_target_rad`, çift kolda `right.q_rad` / `left.q_rad`'dır; ikisi de hız sınırlamasından sonra gönderilen model koordinatlarıdır (rad). Tutucu alanı `gripper_slide`, kayıt uzayındaki slide hedefidir. Çift kol ayrıca tutucunun gerçekten etkin olup olmadığını gösteren `grippers_enabled` alanını yazdırır. VS Code'da `Ctrl+Shift+P` ile `Simple Browser: Show` komutunu çalıştırıp terminalde yazan `http://127.0.0.1:port` adresini yapıştırın; uzak SSH bağlantısında önce Ports panelinden bu portu yönlendirin. MCAP'te üç görüntü akışı varsa üç görünüm gösterilir, eksik görünüm doldurulmaz. `--preview-backend window` ile yerel OpenCV penceresine geçilebilir; `--no-print-actions` kare kare terminal çıktısını, `--no-show-observations` önizlemeyi kapatır. Tarayıcı önizlemesi terminalde Ctrl+C ile durdurulur; OpenCV penceresinde q / Esc de tekrarı durdurur. Yaklaşma, hız sınırı ya da geri bildirim kontrolü başarısız olursa önce veriyi ve cihazı kontrol edin, korumaları gevşetip zorla oynatmayın.

Çift kol salt önizleme (seri port ya da DDS açmaz, tutucu servisi gerekmez):

```bash
python dual/replay.py --episode data/dual_real/episode_0000.mcap --preview-only
```

Bu mod önce önizlemeyi başlatıp adresi yazdırır; VS Code Simple Browser'ı açtıktan sonra terminale dönüp `MOVE` yazdığınızda görüntü ve sol/sağ hareketler oynatılmaya başlar. Varsayılan olarak kayıt hızında oynatılır, `--speed` ile ayarlanabilir. Tarayıcı üç görünümü yan yana dizer ve pencere boyutuna göre orantılı ölçekler; `--preview-scale 0.5` görüntüyü daha da küçültür. Terminal `mode=preview` etiketini gösterir; gerçek donanım modu `mode=command` etiketini gösterir ve `approach`, `playback`, `hold` aşamalarını ayırır. `--preview-only` ile `--motion-mode` birlikte kullanılamaz; ikisi de verilmezse yalnız veri kontrol edilir. Gerçek çift kolda tutucular varsayılan olarak etkindir; önce başka bir terminalde `python -m dual.grippers --side both` çalıştırın, tutucu yoksa `--no-enable-dex1` kullanın.

### 3. Kaydı MuJoCo'da görüntüleme

```bash
python scripts/replay/replay_r1arm_sim.py --episode data/dual_real/episode_0000.mcap
```

Bu komut gerçek donanımı sürmez. `--no-viewer` robot penceresini, `--no-show-observations` kayıt görüntüsü önizlemesini kapatır.

## 🧑‍✈️ Parametreler ve Kontrol Açıklamaları

Aşağıdaki tablo gerçek donanım teleoperasyonu ve kaydının başlıca varsayılan değerleridir; tam parametreler için her girişin `--help` çıktısına bakın.

| Parametre | Varsayılan değer / davranış |
|---|---|
| Kontrol hızı `--rate` | 90 Hz hedef |
| Gözlem hızı `--frequency` | Kayıt girişinde varsayılan 30 Hz |
| `--kp` / `--kd` | 60 / 10; yedi eksen için ayrı ayrı verilebilir |
| `--max-joint-speed` | 1 rad/s |
| Çift kol `--max-joint-acceleration` | 6 rad/s² |
| Şekillendirici konum / hız kazancı | 64 / 16 |
| `--max-command-lead` | 0,15 rad; adım sınırıyla çakışırsa adım sınırı önceliklidir, toplam tork üst sınırı değildir |
| Yerçekimi ileri besleme | Varsayılan açık, `--tau-clip 8` ile eklem başına sınırlanır; motor toplam tork sınırı değildir |
| `--scale-factor` | 1.0 |
| `--home-on-start` | Varsayılan açık; ortak model home'u `0,1.2,0,-1.2,0,0,0` |
| `--reset-blend-frames` | 60; nominal en kısa süre frames/50 Hz ile hesaplanır, gerçekte yine hız ve geri bildirimle sınırlıdır |
| Çift kol `--home-recovery-margin` | 0,25 rad; yalnız sınırlı başlangıç sınır aşımından kurtulmaya izin verir |
| `--baud` | 6000000 |
| `--feedback-timeout` / `--foc-timeout` | Açılış geri bildirimi 3 s / FOC 1 s |
| `--enable-dex1` | Varsayılan açık; `--no-enable-dex1` ile açıkça kapatılır |
| `--grip-torque-limit` | -1, ek kavrama torku kilidi kapalı |

Açılış kontrolleri, model sınırları, geri bildirim arıza kontrolleri, komut adımı ve öne geçme sınırları korunmalıdır. Çift kolda her tarafın ayrı modeli, kalibrasyonu ve kontrol durumu vardır; seri port işlemleri paralel yürür. Gerçek kontrol hızı iletişime ve ana makinenin yüküne bağlıdır.

### Gerçek kontrol hızını ölçme

Önce çalışan teleoperasyon sürecinden düzgünce çıkın, gereken tutucu servislerini açık tutun. Makine ortamını yükledikten sonra:

```bash
python -u scripts/teleop.py --dual --motion-mode --rate 90 --timing-report
```

`MOVE` yazdıktan sonra home tamamlanana kadar bekleyin, ardından durağan tutma sırasında ve grip basılıyken izleme sırasında `[Timing]` çıktısını gözleyin. Bu komut kamera ya da kayıt açmaz; tek kol için `--dual` yerine `--right` ya da `--left` yazın. Kayıt yükünü karşılaştırmak için aynı komuta `--record` ekleyin; önce `--no-cameras` ile hareket toplayın, sonra gerçek kameraları açın.

| Alan | Anlamı |
|---|---|
| `target_hz` | İstenen hız, ulaşıldığı anlamına gelmez |
| `control_hz` | Kontrol döngüsü başlangıçları arasındaki aralığın tersinin ortalamasına karşılık gelen hız |
| `right_command_hz` / `left_command_hz` | Her tarafta SDK komut çağrısının tamamlanma aralığına karşılık gelen hız; bridge içindeki yeniden gönderimleri içermez, motorun iç servo döngüsü hızı değildir |
| `cycle_p95_ms` / `cycle_max_ms` | Döngü aralığının 95. yüzdeliği ve en büyük değeri; 90 Hz için hedef döngü yaklaşık 11,11 ms |
| `xr_p95_ms` | Ana iş parçacığının XR anlık görüntüsünü alma süresi |
| `feedback_p95_ms` | İki tarafın önbellekteki geri bildirimini okuma ve kontrol süresi |
| `right_prepare_p95_ms` / `left_prepare_p95_ms` | İlgili tarafta IK, yörünge şekillendirme gibi hazırlık süresi |
| `right_io_p95_ms` / `left_io_p95_ms` | İlgili tarafta gönderim aşaması süresi; yerçekimi ileri besleme hesabı ve SDK iletişimi/geri bildirim beklemesi dahil |
| `serial_barrier_p95_ms` | Ana iş parçacığının paralel çıktı aşamasının bitmesini bekleme süresi, iş parçacığı zamanlaması dahil |
| `sample_copy_p95_ms` | Çıktıdan sonra kontrol anlık görüntüsünü oluşturma ve kopyalama süresi |
| `record_publish_p95_ms` | Kontrol iş parçacığının kayıt örneğini teslim etme süresi; arka plandaki kamera/kodlama çekişmesi başka aşamalarda ya da döngü sapmasında görünebilir |
| `work_p95_ms` | Ana iş parçacığının bir turdaki işlem süresi, bir sonraki döngüyü bilinçli bekleme hariç |
| `overruns` | Zamanlayıcının saptadığı toplam süre aşımı sayısı; aşımdan sonra yetişmek için ek gönderim yapılmaz |

Her saniye son en çok 300 geçerli döngü aralığının istatistiği yazdırılır, ilk tamamlanmamış döngü sayılmaz. Saf dry run gerçek `*_command_hz` değeri vermez ve gerçek donanımın 90 Hz'e ulaştığını kanıtlamak için kullanılamaz. `[R1ArmBridge] ... retrying` görünürse kayıp geri bildirimi kontrol edin; güncel bridge en çok iki kez yeniden dener ve bu kontrol döngüsünü uzatabilir. Hızı artırmak için geri bildirim korumasını kapatmayın. `serial_barrier` her iki tarafın `io` değerinden çok yüksekse iş parçacığı zamanlamasını ve ana makine yükünü kontrol edin; her tarafın `io` değeri zaten döngü bütçesini aşıyorsa önce SDK'yı, seri portu ve geri bildirim beklemesini inceleyin.

## 🩺 Sık Karşılaşılan Sorunlar

| Belirti | Kontrol yöntemi |
|---|---|
| Kurulum bitti ama SDK / kalibrasyon bulunamıyor | `R1ARM_SDK_ROOT` değerini açıkça ayarlayıp ortam betiğini yeniden source edin; setup fiziksel sıfır noktası toplamaz |
| `Missing/inaccessible` | Gerçek by-id yolunu, seri port grup iznini ve cihaz bağlantısını kontrol edin; adaptör değiştiyse seri numarasını ve tüm port değişkenlerini güncelleyin |
| Yol `Accessible` ama motor yanıt vermiyor | Accessible yalnız yolun ve iznin var olduğunu gösterir; güç, kablo, kanal rolü, ID, adaptör ve port kullanımını kontrol edin |
| `Port is already used` | Eski denetleyiciden ya da servisten çıkın; aynı hattı birden çok sürümle paylaştırmayın |
| Tutucu `motor id=7 does not reply` | Önce hangi tarafın başarısız olduğunu bulun, tutucunun gücünü, kablosunu, ID'sini ve seri portunu kontrol edin; DDS ağ kartı seri port yoklamasının yanıt verip vermemesini belirlemez |
| Çift tutucuda bir taraf başladıktan sonra iki taraf da çıkıyor | Başlatıcı tek taraf başarısız olunca diğer tarafı temizler; ilk hatayı veren tarafa bakın, iki tarafın birden bozulduğu anlamına gelmez |
| `lo is not multicast-capable` | Yerel lo yapılandırmasındaki bilinen uyarıdır; çalışmayı motor tanıma ve sürekli DDS geri bildirimiyle değerlendirin |
| `No Dex1 feedback ...state` | Servisin çalışıp çalışmadığını, left/right topic'ini, domain'i ve DDS ağ kartının eşleşmesini kontrol edin |
| `missing joints [1,...,6]` | Liste sıfırdan başlayan indeks kullanır, ID1-6 / J2-J7'ye karşılık gelir; önce geri bildirim iletişimini, sonra kalibrasyonu kontrol edin, ID'yi körlemesine değiştirmeyin ya da yeniden kalibre etmeyin |
| J6 gibi eksenlerde açı sınır aşımı | Mevcut kalibrasyonu, gerçek referans pozu ve enkoder referansını karşılaştırın; yaklaşık bir tur fark olsa bile kontrol etmeden otomatik mod almayın |
| PC Service bağlı ama kumanda izlemiyor | Başlığın uyanık, uygulamanın ön planda, kumanda gönderiminin açık olduğunu, poz ve zaman damgasının sürekli güncellendiğini doğrulayın, sonra ağı kontrol edin |
| `CXXABI` / `GLIBCXX` / dinamik kitaplık eksik | Doğru Conda ortamını etkinleştirin, `real_env.sh` yükleyin; SDK, bridge ve Dex1 kitaplık yollarını ve mimari uyumunu kontrol edin |
| Modül eksik | base yerine r1_teleop kullandığınızı doğrulayın; modül eksikse setup.sh'nin başarıyla bitip bitmediğini kontrol edin |
| Kamera eksik olduğu için çift kol başlamıyor | Salt teleoperasyonda önce `--no-cameras` kullanın, akışları tek tek doğruladıktan sonra görüntüyü açın; kamerayı kullanan diğer programları durdurun |
| Mock başlatma ve gerçek donanım günlükleri birlikte görünüyor | Çift kol başlatılırken sol taraf geçici olarak mock kullanır, ardından sağ taraftaki XR'ı paylaşır; tek bir `Using mock XR input` satırı sonuçta gerçek giriş olmadığı anlamına gelmez |

Bir seferde tek bir koşulu değiştirin. İletişimin geri gelmesi hareketin tümüyle kabul edildiği anlamına gelmez; sırasıyla geri bildirimi, kalibrasyonu, home'u, tek taraflı izlemeyi, çift taraflı izlemeyi ve tutucu açma kapamayı doğrulayın, en son kamera ve kayıt yükünü ekleyin.

## Kol İletişim Kontrolü

Bu bölüm SDK ile gelen `test_arm_lowlevel` tanılama komutunu anlatır. Komut gerçek kolun seri portunu doğrudan açar; salt okunur bir araç değildir. Önce yukarıdaki gibi makine ortamını yükleyin, teleoperasyon, kayıt, tekrar gibi seri portu kullanan programlardan çıkın, kolu güvenle destekleyin ve acil durdurmanın erişilebilir olduğunu doğrulayın. Seri port `Accessible` görünse bile bu yalnız cihaz yolunun okunup yazılabildiğini gösterir, motorun açık olduğunu ya da geri bildirim verdiğini göstermez.

### Tek bir motoru yoklama

Örneğin sağ kolda ID 1'i kontrol etmek için:

```bash
"$ARM/bin/test_arm_lowlevel" \
  --probe-id 1 \
  --port "$PORT_RIGHT" \
  --baud 6000000 \
  --calib "$CALIB_RIGHT"
```

`--probe-id N` için ID aralığı 0-6'dır. Program önce tüm kola durdurma komutu gönderir, ardından seçilen ID'ye sürekli sıfır kazançlı FOC yoklaması gönderir; başarılı olunca Ctrl+C ile çıkın. Başarıda `Feedback ready from motor ID N` görünmeli, Bus replies artmalı, Bus timeouts 0 kalmalıdır. Komut yalnız seçilen ID'yi doğrular, yedi eksenin tamamının iletişiminin sağlam olduğunu göstermez. Yoklama sırasında motor pozu korumaz; çıkışta da tüm kol durdurulur, bu yüzden baştan sona güvenli destek gerekir.

### Bilinen arıza kodlarını temizleme

Yalnız geri bildirim normal, arıza nedeni giderilmiş ve kilitlenmiş arızanın temizlenmesi gerekiyorsa kullanın. Sağ kol örneği:

```bash
"$ARM/bin/test_arm_lowlevel" \
  --clear \
  --port "$PORT_RIGHT" \
  --baud 6000000 \
  --calib "$CALIB_RIGHT"
```

Sol kolu kontrol ederken `$PORT_LEFT` ve `$CALIB_LEFT` kullanın. `--clear` önce yoklama yapıp yedi eksenin durumunu yazdırır, tüm motorlara birkaç tur arıza temizleme komutu gönderir, sonra yeniden yoklayıp durumu yazdırır. Sıfır kazançlı FOC yoklaması gönderdiği için kol pozu korumayabilir. Kod temizlemek düşük gerilimli güç kaynağını, kopuk kabloyu ya da geri bildirim yokluğunu düzeltmez; önce kök nedeni giderin, `--clear` ya da `--reset` komutunu iletişim onarımı adımı olarak kullanmayın.

### Diğer düşük seviye komutlar

| Komut | Gerçekte yaptığı | Etkisi |
|---|---|---|
| `--stop` | Yedi eksene yaklaşık 500 ms boyunca mode 0 durdurma komutu gönderir | Motor tutması kalkar, kol sarkabilir |
| `--zero-torque` | Tüm eksenlerin geri bildirimini bekledikten sonra sürekli `kp=0`, `kd=0`, `tau=0` FOC komutu gönderir | Yedi eksen pozu etkin olarak korumaz; Ctrl+C sonrası durdurma komutu gönderilir |
| `--foc` | Yedi eksenin mevcut pozunu okur, `kp` / `kd` ile bu pozu sürekli korur | Motorlar tutma torku üretir; Ctrl+C sonrası durdurma komutu gönderilir |
| `--clear` / `--clear-fault` | Yoklar, temizleme öncesi durumu yazdırır, tüm motorlara arıza temizleme komutu gönderir, yeniden yoklar | Yalnız temizlenebilir motor arıza durumlarını temizler, süregelen arıza nedenini düzeltmez |
| `--reset` | Yoklar, durumu yazdırır, tüm motorlara sıfırlama komutu gönderir, yeniden yoklar | Motorun iç durumunu değiştirir; yalnız açıkça gerektiğinde kullanın |
| `--home` | Yedi ekseni URDF/model koordinatında tam sıfıra götürür | Gerçek hareket yapar; bu poz teleoperasyon home'u `[0, 1.2, 0, -1.2, 0, 0, 0]` ile aynı değildir |
| `--goto --q ...` | Yedi ekseni `--q` ile verilen yedi URDF/model açısına götürür, birim rad | Gerçek hareket yapar; çalıştırmadan önce hedefi, sınırları ve kolun çevresini kontrol edin |

### Sık görülen çıktılar

| Çıktı | Anlamı ve yapılacak |
|---|---|
| `No valid feedback from motor ID N` | Belirtilen ID geçerli veri döndürmedi; motor gücünü, iletişim kablosunu, USB-485 kanalını ve ID'yi kontrol edin. Kalibrasyon dosyasının yüklenmesi motor geri bildirimi alındığı anlamına gelmez |
| `Feedback incomplete: j1 ...` | `j`'den sonraki sayı sıfırdan başlayan eklem indeksidir; kol ID'si, güç ve kabloyla karşılaştırarak tek tek kontrol edin, önce kolu destekleyin, ID'yi körlemesine değiştirmeyin ya da yeniden kalibre etmeyin |
| `motor_timeout=1` | Motor geri bildirim paketindeki zaman aşımı durumudur; programın saydığı Bus timeouts (yanıt bekleme zaman aşımı) ile farklı iki alandır |
| `error=0` ama geri bildirim yok | Paket alınmadığında okunacak arıza kodu yoktur; buna bakarak motorun sağlam olduğuna karar verilemez |
| `Port is already used` | Seri port başka bir süreç tarafından kullanılıyor; o süreçten çıktıktan sonra kontrol edin, başlatma aracı onu kendiliğinden sonlandırmaz |

Tek motor tanılama programı `$ARM/bin/test_arm_lowlevel` konumundadır, doğrudan SDK'yı ve motor kontrol kitaplığını kullanır, Python bridge'e bağlı değildir. İletişim sorunlarında önce güç, bağlantı, kanal eşlemesi ve geri bildirim kontrol edilmelidir; iletişim geri gelince kalibrasyon, home, tek ve çift kol hareketi, en son tutucu, kamera ve kayıt yükü kontrol edilir. Sıradan bir güç kesintisi yeniden kalibrasyon gerektirmez; yalnız sıfır noktası eşleşmesinin değiştiği doğrulanırsa kalibrasyon yeniden toplanır.

## 🧪 Çevrimdışı Regresyon Testleri

```bash
python -m unittest discover -s tests
python -m unittest dual.tests.test_dual
```

Gerçek motora bağlanmayan regresyon testleri donanım kabulünün yerini tutmaz. Bazı çift kol testleri `R1ARM_SDK_ROOT` altındaki mevcut sol/sağ kalibrasyon dosyalarını okur; sıfırdan kurulan makinede önce yapılandırma ve dosyalar hazırlanmalıdır.

## 🗂️ Proje Yapısı

```text
assets/                         Kol URDF'si, mesh dosyaları ve MuJoCo sahneleri
robot/                          SDK bridge sarmalayıcısı, cihaz parametreleri ve Dex1 istemcisi
teleop/                         XR girişi, IK, kontrol, kamera ve MCAP
scripts/teleop.py               Ortak gerçek donanım / simülasyon teleoperasyon ve kayıt girişi
dual/                           Yeniden kullanılan kol kontrolü, kayıt, çift kol tekrarı ve kalibrasyon
native/                         C++ bridge ve bağımlılık uyarlama yamaları
third_party/                    Sabit sürümlü üçüncü taraf submodule'ler
tests/, dual/tests/             Çevrimdışı regresyon testleri
mp4/                            Demo GIF'leri ve cihaz fotoğrafları
```

Hızlı başlangıç için [README](../README.md), çift kol yönleri ve veri düzeni için [CIFT-KOL.md](CIFT-KOL.md), bağımlılık kaynakları ve yamalar için [BAGIMLILIK-YAMALARI.md](BAGIMLILIK-YAMALARI.md) dosyalarına bakın.

## 🙏 Teşekkür

[XRoboToolkit](https://github.com/XR-Robotics/XRoboToolkit-PC-Service), [XRoboToolkit Python bağlaması](https://github.com/YanjieZe/XRoboToolkit-PC-Service-Pybind), [R1-7A SDK](https://github.com/unitreerobotics/r1-7a_sdk), [Unitree SDK2](https://github.com/unitreerobotics/unitree_sdk2), [Unitree SDK2 Python](https://github.com/unitreerobotics/unitree_sdk2_python), [Dex1 servisi](https://github.com/unitreerobotics/dex1_1_service), [Placo](https://github.com/Rhoban/placo) ve [MuJoCo](https://github.com/google-deepmind/mujoco) projelerine teşekkürler.

Donanım, kontrol girişleri, veri biçimleri ve işlev açıklamalarında orijinal deponun kodu esastır.

## 📄 Lisans

Orijinal depoda şu an kök dizinde `LICENSE` dosyası yoktur; üçüncü taraf SDK, model, çalışma zamanı kitaplıkları ve bağımlılıkların lisansları kendi projelerinin beyanlarına tabidir.

---

> **Unitree R1-7A Türkiye:** tedarik, kurulum, kalibrasyon desteği ve Türkçe eğitim için [Robotlar.org R1-7A sayfası](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a) · [Kurumsal teklif](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) · 📧 info@robotlar.org
