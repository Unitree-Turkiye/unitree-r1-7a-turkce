<div align="center">

# Unitree R1-7A Türkçe: 7 Eksenli Robot Kol Teleoperasyon Rehberi

**PICO / XRoboToolkit ile tek kol ve çift kol teleoperasyon · Çok kameralı kayıt · MCAP kayıt ve tekrar oynatma**

**Unitree Robotics Türkiye Resmi Distribütörü · [Robotlar.org](https://www.robotlar.org)**

👉 **R1-7A ürün sayfası ve teklif: [robotlar.org/tr/unitree-bilesenler/r1-7a](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a)**

[Hızlı Kurulum](#-ortam-kurulumu) · [Teleoperasyon](#-teleoperasyon) · [Veri Toplama](#-veri-toplama) · [Ayrıntılı Rehber](docs/AYRINTILI-REHBER.md) · [Unitree Ürünleri](#-unitree-ürün-ailesi-türkiye)

</div>

> Bu depo, Unitree Robotics'in resmi [unitreerobotics/R1-7A](https://github.com/unitreerobotics/R1-7A) projesinin belgelerinin Türkçe çevirisidir. Kaynak kod, URDF/MuJoCo modelleri ve görseller orijinal depoda kalır; burada yalnız Türkçe anlatım bulunur. Tüm haklar Unitree Robotics'e aittir. Kodu çalıştırmak için her zaman orijinal depoyu klonlayın.
>
> Çeviri ve Türkçe teknik destek: Unitree Türkiye resmi distribütörü [Robotlar.org](https://www.robotlar.org).

| Tek kol teleoperasyon | Çift kol teleoperasyon |
|:---:|:---:|
| ![Unitree R1-7A tek kol teleoperasyon demosu](https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/single_arm_teleop.gif) | ![Unitree R1-7A çift kol teleoperasyon demosu](https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/dual_arm_teleop.gif) |

## ✳️ Genel Bakış

R1-7A teleoperasyon çatısı, iki PICO kumandasıyla 7 serbestlik dereceli (7-DoF) **Unitree R1-7A robot kollarını** ve **Unitree Dex1-1 tutucuları** yönetir. Tek kol ve çift kol çalışmayı, kamera ve eklem verisi toplamayı destekler.

- 🎮 **Teleoperasyon:** kumandanın göreli pozunu izler; her kol ve tutucu ayrı ayrı kontrol edilir.
- 📦 **Veri toplama:** kontrol hedefleri, ölçülen eklem durumları ve birden çok kamera akışı MCAP dosyasına kaydedilir.
- 🔄 **Yörünge tekrarı:** kayıtlar gerçek donanımda tekrar oynatılır ya da MuJoCo içinde incelenir.

`Donanım Hazırlığı` → `Ortam Kurulumu` → `Cihaz Kalibrasyonu` → `Teleoperasyon ve Kayıt` → `Yörünge Tekrarı`

### R1-7A nedir?

[Unitree R1-7A](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a), metal ve yüksek dayanımlı mühendislik plastiği gövdeli, yaklaşık 5,5 kg ağırlığında 7 eksenli biyonik bir koldur. 2 kg yük taşır, becerikli el ile 650 mm erişime ulaşır; kuvvet geri bildirimi ve çarpışma algılaması vardır. Tak-çalıştır kurulur, açık kaynak programlanır ve uç efektörü değiştirilebilir. Bu yüzden taklit öğrenme (imitation learning), VLA veri toplama ve üniversite laboratuvarları için uygun bir platformdur.

## 🔧 Donanım Hazırlığı

<p align="center">
  <img src="https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/r1_7a_with_dex1.jpg" alt="Unitree R1-7A robot kollar, Dex1-1 tutucular ve kameralar" width="800">
</p>

<p align="center">
  <img src="https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/hardware_wiring.png" alt="R1-7A donanım kablolama şeması: PC, USB-RS485 modülü, kol güç modülü ve tutucu bağlantıları (etiketler Çince)" width="800">
</p>

| Cihaz | Açıklama |
|---|---|
| [R1-7A 7 eksenli robot kol](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a) | Tek kol çalışma için bir, çift kol için iki adet; her kolun kendi seri portu ve kalibrasyonu olur |
| [Dex1-1 tutucu](https://www.robotlar.org/tr/robot-elleri/unitree-dex1-1) | Sol ve sağ bağımsız kontrol; takılı değilse devre dışı bırakılabilir |
| PICO başlık ve iki kumanda | XRoboToolkit istemcisini çalıştırır |
| Linux PC | Ubuntu 22.04, x86_64, Git ve Conda kurulu |
| [4 kanallı USB-RS485 modülü](https://support.unitree.com/home/en/Motor_SDK_Dev_Guide/4-channel_485_to_USB_module) | PC, kollar ve tutucuları ayrı kanallardan bağlar |
| Kameralar (isteğe bağlı) | Görüntü toplamak için RealSense ya da V4L2 / UVC kameralar |

> ⚠️ Gerçek donanımı çalıştırmadan önce kol tabanlarını sabitleyin, kolları destekleyin, acil durdurma düğmesini erişilebilir ve home (başlangıç) yolunu boş tutun. Sıfır kazançlı yoklama (zero-gain probing) kolları yerinde tutmaz. Çift kol çarpışma planlaması sağlanmaz.

## 📥 Ortam Kurulumu

### 1. Depoyu klonlayın

Kod orijinal Unitree deposundadır:

```bash
git clone --recurse-submodules https://github.com/unitreerobotics/R1-7A.git
cd R1-7A
```

### 2. Bağımlılıkları kurun

Tüm proje bağımlılıklarını tek komutla kurun ve gerekli bileşenleri derleyin:

```bash
bash setup.sh
```

### 3. PICO yapılandırması

1. PC'ye [XRoboToolkit PC Service](https://github.com/XR-Robotics/XRoboToolkit-PC-Service) kurun ve başlatın.
2. PICO başlığa [XRoboToolkit istemcisini](https://github.com/XR-Robotics/XRoboToolkit-Unity-Client) kurun ve PC'nin yerel ağ IP adresine bağlanın.
3. Uygulamayı ön planda tutun, kumanda verisi gönderimini açın ve kumandaları hareket ettirerek verinin sürekli güncellendiğini doğrulayın.

Kurulum seçenekleri, bağımlılık sürümleri ve sorun giderme için [ayrıntılı kurulum rehberine](docs/AYRINTILI-REHBER.md#-ortam-kurulumu) bakın.

## ⚙️ Cihaz Yapılandırması ve Kalibrasyon

İlk çalıştırmadan önce seri portları, iki kolun kalibrasyon dosyalarını ve tutucu uç noktalarını (endpoint) yapılandırın. Makineye özgü ayarları `.env` dosyasına kaydedin. Tam örnek için [yapılandırma ve kalibrasyon rehberine](docs/AYRINTILI-REHBER.md#️-cihaz-yapılandırması-ve-kalibrasyon) bakın.

Dört kanallı FTDI adaptörü için referans kablolama:

| Sağ Tutucu | Sağ Kol | Sol Tutucu | Sol Kol |
|:---:|:---:|:---:|:---:|
| if00 | if01 | if02 | if03 |

Adaptör seri numarasını ve fiziksel kablolamayı doğrulayın, her kol için doğru kalibrasyon dosyasını kullanın. Mevcut kalibrasyon dosyaları, fiziksel cihazlarla ve güncel enkoder referanslarıyla eşleşiyorsa yeniden kullanılabilir. Kurulum betiği sıfır konumlarını otomatik yakalamaz.

**Her terminalde ortamı depo kökünden yükleyin:**

```bash
conda activate r1_teleop
source .env
source scripts/real_env.sh
```

`r1_teleop` Conda ortamının adıdır. Komutlardaki `r1arm` adları mevcut kod yollarıdır.

## 🛠️ Teleoperasyon

Tek giriş noktası `scripts/teleop.py` kullanılır ve `--right`, `--left` ya da `--dual` seçeneklerinden yalnız biri seçilir. **Kaydı açmak için `--record` ekleyin. Bu bayrak yoksa kaydedici, kamera ya da kayıt dosyası oluşturulmaz.**

### 1. Terminal 1: Seçilen tutucu servislerini başlatın

Ortamı yükleyin, bir komut seçin ve bu terminali açık tutun:

```bash
# Sağ kol
python -m dual.grippers --side right

# Sol kol
python -m dual.grippers --side left

# İki kol
python -m dual.grippers --side both
```

Gereken her tarafın `Detected motors` ve `Gripper Server started` mesajlarını verdiğini doğrulayın.

### 2. Terminal 2: Teleoperasyonu başlatın

Aynı ortamı yükleyin ve bir komut seçin:

```bash
# Sağ kol
python scripts/teleop.py --right --motion-mode

# Sol kol
python scripts/teleop.py --left --motion-mode

# İki kol
python scripts/teleop.py --dual --motion-mode
```

`MOVE` yazmadan önce portları, kalibrasyon dosyalarını ve açılış mesajlarını kontrol edin. Seçilen kollar varsayılan olarak yumuşakça home konumuna gider. Home tamamlandıktan sonra izlemeyi başlatmak için ilgili grip düğmesini basılı tutun. Sol kol modu, sağ kol donanımı gerektirmeden sol kalibrasyonu, koordinat dönüşümünü ve tutucu konusunu (topic) kullanır.

Tutucu takılı değilse tutucu servislerini atlayın ve `--no-enable-dex1` ekleyin.

### 3. Kumanda düğmeleri

| Giriş | Davranış |
|---|---|
| Sol / sağ grip | Basılı tutulduğunda ilgili kumandanın göreli pozunu izler |
| Sol / sağ tetik | İlgili tutucunun kapanma oranını kontrol eder |
| A (sağ kumanda) | Seçilen kolları home konumuna döndürür |
| X (sol kumanda) | `--record` ile: kaydı başlatır / durdurup kaydeder |
| B (sağ kumanda) | Etkin kayıt sırasında kaydı ve izlemeyi duraklatır / sürdürür |
| Ctrl+C | Durdurur ve çıkar; çıkıştan sonra kolları desteklemeye devam edin |

Varsayılan kayıt kontrolleri tek kol modunda da iki kumandayı kullanır: X sol kumandada, A ve B sağ kumandadadır.

## 📦 Veri Toplama

Çalışan teleoperasyon programından çıkın, seçilen tutucu servislerini açık bırakın ve aynı giriş noktasına `--record` ekleyin:

```bash
# Sağ kol: varsayılan olarak kafa ve sağ bilek kameraları
python scripts/teleop.py --right --record --motion-mode --output-dir data/right_real

# Sol kol: varsayılan olarak kafa ve sol bilek kameraları
python scripts/teleop.py --left --record --motion-mode --output-dir data/left_real

# İki kol: varsayılan olarak kafa ve iki bilek kamerası
python scripts/teleop.py --dual --record --motion-mode --output-dir data/dual_real
```

Başlatmak / kaydetmek için X, duraklatmak / sürdürmek için B tuşuna basın. MCAP vektörleri tek kol modunda 8, çift kol modunda 16 değer içerir. Dosyalar `episode_0000.mcap` biçiminde adlandırılır. Gerçek donanımda hedef hızlar kontrol için 90 Hz, görüntü gözlemi için 30 Hz'dir.

Kamera yapılandırması için [görüntü toplama rehberine](docs/AYRINTILI-REHBER.md#2-görüntüyü-açma) bakın. Görüntüsüz kayıt için `--no-cameras` ekleyin:

```bash
python scripts/teleop.py --dual --record --motion-mode --no-cameras --output-dir data/dual_real
```

Toplanan bölümler (episode) LeRobot gibi veri seti biçimlerine dönüştürülerek ACT, Diffusion Policy ve VLA eğitiminde kullanılabilir. Türkçe iş akışı için: [Unitree LeRobot rehberi](https://www.robotlar.org/tr/rehber/unitree-lerobot-rehberi).

## 🔄 Yörünge Tekrarı

Önce teleoperasyon ve kayıt programlarından çıkın. Salt okunur kontrol motorları sürmez:

```bash
python dual/replay.py --episode data/dual_real/episode_0000.mcap
```

Veriyi ve cihaz yapılandırmasını kontrol ettikten sonra tutucu servislerini açık tutun ve gerçek donanımda tekrar oynatın:

```bash
python dual/replay.py --episode data/dual_real/episode_0000.mcap --motion-mode
```

Tekrar oynatma PICO gerektirmez. Tek kol tekrarı, yavaş oynatma ve görüntü önizlemesi için [ayrıntılı tekrar rehberine](docs/AYRINTILI-REHBER.md#-yörünge-tekrarı) bakın.

## 🧪 Simülasyon

Gerçek kollara bağlanmadan, sahte (mock) girişle çift kol simülasyonu çalıştırın:

```bash
python scripts/teleop.py --sim --dual --mock-xr
```

Gerçek PICO girişini denemek için `--mock-xr` bayrağını kaldırın. Simülasyon `--right`, `--left` ve `--record` seçeneklerini de destekler; simülasyon seçenekleri için `python scripts/teleop.py --sim --help` kullanın.

![Unitree R1-7A çift kol MuJoCo simülasyonu](https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/simulation.gif)

## 🧑‍✈️ Temel Parametreler

| Parametre | Varsayılan |
|---|---|
| Hedef kontrol hızı | 90 Hz |
| Motor kp / kd | 60 / 10 |
| Azami eklem hızı | 1 rad/s |
| Kumanda öteleme ölçeği | 1.0 |
| Home | `0,1.2,0,-1.2,0,0,0` rad |

Tam parametre listesi için her giriş noktasında `--help` kullanın. Cihaz yapılandırması, kalibrasyon, veri biçimleri ve sorun giderme [ayrıntılı rehberde](docs/AYRINTILI-REHBER.md) anlatılır.

## 🧠 Eğitim ve 🚀 Dağıtım

![Unitree R1-7A eğitim ve dağıtım demosu](https://raw.githubusercontent.com/unitreerobotics/R1-7A/main/mp4/training_deployment.gif)

Orijinal depoda bu bölümler henüz boştur; Unitree güncelledikçe çeviri de güncellenecektir.

## 📚 Belgeler

| Dosya | İçerik |
|---|---|
| [docs/AYRINTILI-REHBER.md](docs/AYRINTILI-REHBER.md) | Kurulum, XRoboToolkit, kalibrasyon, kayıt, tekrar, parametreler, sorun giderme, düşük seviye kol komutları (`detail.md` çevirisi) |
| [docs/CIFT-KOL.md](docs/CIFT-KOL.md) | Çift kol temel hattı: sol ve sağ model eşlemesi, 16 boyutlu MCAP, havlu katlama örneği (`dual/README.md` çevirisi) |
| [docs/BAGIMLILIK-YAMALARI.md](docs/BAGIMLILIK-YAMALARI.md) | Resmi bağımlılık sürümleri ve yerel yamalar (`native/dependency_patches/README.md` çevirisi) |

---

## 🤖 Unitree Ürün Ailesi (Türkiye)

Robotlar.org, Unitree Robotics'in Türkiye resmi distribütörüdür. Aşağıdaki ürünlerin Türkçe tanıtımı, teknik özellikleri ve kurumsal teklif formu sitede yer alır. Fiyatlar konfigürasyona göre değiştiği için [teklif üzerinden](https://www.robotlar.org/tr/fiyat/unitree-kurumsal-teklif-al) paylaşılır.

### Robot kollar ve robot elleri

| Ürün | Ne işe yarar |
|---|---|
| [Unitree R1-7A](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a) | 7 eksenli biyonik kol, 2 kg yük, 650 mm erişim, teleoperasyon ve veri toplama |
| [Unitree Z1](https://www.robotlar.org/tr/unitree-bilesenler/z1) | Mobil robot üstünde çalışan 6 eksenli kol, 740 mm erişim, yaklaşık 0,1 mm tekrarlanabilirlik |
| [Unitree D1-T](https://www.robotlar.org/tr/unitree-bilesenler/d1-t) | Çift ya da dört kollu teleoperasyon kiti, 6 eksen ve tutucu, 550 mm erişim |
| [Unitree Dex1-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex1-1) | R1-7A ile kullanılan iki çeneli tutucu: 90 mm strok, 5-120 N ayarlı kavrama, RS-485 |
| [Unitree Dex3-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex3-1) | G1 için 7 serbestlik dereceli el, opsiyonel 33 dokunsal sensör |
| [Unitree Dex5-1](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-1) | 20 serbestlik dereceli (16 aktif) el, Dex5-1P sürümünde 94 dokunsal sensör |
| [Unitree Dex5-S](https://www.robotlar.org/tr/robot-elleri/unitree-dex5-s) | 22 aktif serbestlik, 620 g, çift T1s Ethernet, 1000 Hz |
| [Tüm robot elleri →](https://www.robotlar.org/tr/robot-elleri) | |

### İnsansı robotlar

| Ürün | Ne işe yarar |
|---|---|
| [Unitree G1](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1) | 132 cm, 35 kg, 23-43 DoF insansı robot |
| [Unitree G1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-d) | Uçtan uca insansı robot veri toplama ve eğitim platformu, Jetson Orin |
| [Unitree G1 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-g1-plus) | 2 DoF boyun, yüksek tork, dokunsal kafa ve 6 mikrofonlu G1 |
| [Unitree H2](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2) | 182 cm, 70 kg, 31 DoF tam boy insansı robot |
| [Unitree H2-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-d) | 7 DoF kollu, tekerlekli çift kollu insansı robot |
| [Unitree H2 Plus](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-plus) | NVIDIA Isaac GR00T referans insansı robotu, 75 DoF, Jetson Thor |
| [Unitree H2-X](https://www.robotlar.org/tr/insansi-robotlar/unitree-h2-x) | Hizmet ve koruma görevleri, 4 m platformdan iniş |
| [Unitree R1](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1) | 123 cm, 29 kg, 26 DoF insansı robot (AIR, R1, EDU sürümleri) |
| [Unitree R1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1-d) | R1-A5 ya da **R1-A7 (7-DOF)** kollu çift kollu insansı platform |
| [Tüm insansı robotlar →](https://www.robotlar.org/tr/insansi-robotlar) | |

### Dört ayaklı robotlar (robot köpekler)

| Ürün | Ne işe yarar |
|---|---|
| [Unitree Go2](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2) | 4D LiDAR L2 ve GPT entegrasyonlu robot köpek (Air, Pro, X, EDU) |
| [Unitree Go2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-go2-w) | Tekerlekli bacaklı robot köpek, 15 km menzil |
| [Unitree B2](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2) | Endüstriyel robot köpek, IP67, devriye ve saha denetimi |
| [Unitree B2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-b2-w) | Hibrit tekerlekli endüstriyel köpek, 30 km menzil, IP67 |
| [Unitree A2](https://www.robotlar.org/tr/kopek-robotlar/unitree-a2) | 100 kg statik yük, 45° tırmanma, saha robotu |
| [Unitree A2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-a2-w) | Tekerlekli A2, yaklaşık 35 km menzil, çift batarya |
| [Unitree AS2](https://www.robotlar.org/tr/kopek-robotlar/unitree-as2) | Kompakt endüstriyel robot köpek, devriye ve saha denetimi |
| [Unitree AS2-W](https://www.robotlar.org/tr/kopek-robotlar/unitree-as2-w) | Tekerlekli AS2, 30+ km menzil, 150 TOPS'a kadar açık platform |
| [Tüm dört ayaklı robotlar →](https://www.robotlar.org/tr/kopek-robotlar) | |

### Bileşenler ve sensörler

[4D LiDAR L1](https://www.robotlar.org/tr/unitree-bilesenler/4d-lidar-l1) · [4D LiDAR L2](https://www.robotlar.org/tr/unitree-bilesenler/4d-lidar-l2) · [GO-M8010-6 motor](https://www.robotlar.org/tr/unitree-bilesenler/go-m8010-6) · [Tüm Unitree bileşenleri](https://www.robotlar.org/tr/unitree-bilesenler) · [Yedek parça](https://www.robotlar.org/tr/unitree-yedek-parca)

## 📖 Türkçe Rehberler

- [Unitree Dex1-1 robotik tutucu rehberi](https://www.robotlar.org/tr/rehber/unitree-dex1-1-robotik-tutucu)
- [Unitree Z1 ve Z1 Pro robot kol rehberi](https://www.robotlar.org/tr/rehber/unitree-z1-ve-z1-pro-robotik-kol)
- [Unitree LeRobot: veri toplama ve taklit öğrenme](https://www.robotlar.org/tr/rehber/unitree-lerobot-rehberi)
- [Unitree G1 XR teleoperasyon](https://www.robotlar.org/unitree/g1-teleoperasyon)
- [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
- [Unitree eğitim çözümleri (üniversite ve laboratuvar)](https://www.robotlar.org/tr/unitree-egitim-cozumleri)
- [Unitree yetkili servis Türkiye](https://www.robotlar.org/tr/rehber/unitree-yetkili-servis-turkiye)
- [Tüm açık kaynak Unitree projeleri (Türkçe)](https://www.robotlar.org/tr/acik-kaynak)

## 🔗 Unitree Türkiye GitHub Ağı

**Modeller:** [Go2](https://github.com/Unitree-Turkiye/unitree-go2-turkce) · [Go2-W](https://github.com/Unitree-Turkiye/unitree-go2-w-turkce) · [B2](https://github.com/Unitree-Turkiye/unitree-b2-turkce) · [A2](https://github.com/Unitree-Turkiye/unitree-a2-turkce) · [G1](https://github.com/Unitree-Turkiye/unitree-g1-turkce) · [H2](https://github.com/Unitree-Turkiye/unitree-h2)
**Geliştirici:** [SDK2](https://github.com/Unitree-Turkiye/unitree-sdk2-turkce) · [XR Teleoperasyon](https://github.com/Unitree-Turkiye/unitree-teleoperasyon-turkce) · [LeRobot](https://github.com/Unitree-Turkiye/unitree-lerobot-turkce) · [Becerikli El](https://github.com/Unitree-Turkiye/unitree-hunerli-el-turkce) · [MuJoCo](https://github.com/Unitree-Turkiye/unitree-mujoco-turkce) · [ROS2](https://github.com/Unitree-Turkiye/unitree-ros2-turkce) · [VLA](https://github.com/Unitree-Turkiye/unitree-vla-turkce) · [🏠 Unitree Türkiye](https://github.com/Unitree-Turkiye)

## 🙏 Teşekkür

[XRoboToolkit](https://github.com/XR-Robotics/XRoboToolkit-PC-Service), [Placo](https://github.com/Rhoban/placo), [MuJoCo](https://github.com/google-deepmind/mujoco) ve ilgili Unitree SDK projelerine teşekkürler (orijinal depodan).

## 📄 Lisans

Orijinal depoda şu an kök dizinde `LICENSE` dosyası yoktur; üçüncü taraf bağımlılıklar kendi lisanslarını korur. Bu depo yalnız belge çevirisi içerir ve kaynak kod ya da görsel dosyası barındırmaz; görseller orijinal depodan gösterilir. Unitree Robotics talep ederse çeviri kaldırılır.

---

> ℹ️ **Unitree Türkiye:** Unitree Robotics'in Türkiye resmi satış ve teknik distribütörü. R1-7A ve Dex1-1 tedariki, kurulum, eğitim ve Türkçe geliştirici desteği: **[robotlar.org](https://www.robotlar.org/tr/rehber/unitree-turkiye-distributoru)** · 📧 info@robotlar.org

© 2026 **[www.robotlar.org](https://www.robotlar.org)**
