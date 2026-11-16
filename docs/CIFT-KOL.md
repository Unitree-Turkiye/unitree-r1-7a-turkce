# Unitree R1-7A Çift Kol Temel Hattı

[README'ye dön](../README.md) · Kaynak: [unitreerobotics/R1-7A `dual/README.md`](https://github.com/unitreerobotics/R1-7A/blob/main/dual/README.md) · Türkçe çeviri: [Robotlar.org](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a)

Tam işletim akışı, iki ayrı komut setinin bakımını önlemek için [ana README](../README.md) içinde toplanmıştır.

```bash
# Terminal 1: sol ve sağ ID7 tutucu servisleri
bash dual/start_grippers.sh

# Terminal 2: üç görüntü akışı, 90 Hz kontrol / 30 Hz gözlem kaydı
python scripts/teleop.py --dual --record --motion-mode --data-root data/dual_real

# Salt okunur kontrol; doğruladıktan sonra --motion-mode ekleyin; görüntü varsayılan olarak ayrı süreçte gösterilir
python dual/replay.py --episode data/dual_real/episode_0000.mcap

# Gerçek donanımı sürmeden görüntüyü ve yörüngeyi oynat
python scripts/replay/replay_r1arm_sim.py --episode data/dual_real/episode_0000.mcap
```

Yukarıdaki komutlardan önce `r1_teleop` ortamını etkinleştirip `scripts/real_env.sh` dosyasını source edin. X başlatır/kaydeder, B duraklatır/sürdürür, A iki kolu home'a döndürür; sol ve sağ grip/tetik ilgili tarafı kontrol eder.

## Sol ve sağ model eşlemesi

Aynı R1Arm URDF'si kullanılır, ancak sol ve sağ model, kinematik veri, şekillendirme durumu ve kalibrasyon bağımsızdır. Kalibrasyon hattından ortak model yönüne:

- Sağ: `[1, 1, 1, -1, 1, 1, 1]`
- Sol: `[1, -1, 1, 1, 1, -1, 1]`

Solda varsayılan olarak J2/J6 mirrored, J4 model kullanılır; sıfır noktası dosyasında işareti ikinci kez çevirmeyin. Ortak model home'u `[0,1.2,0,-1.2,0,0,0]`'dır. Çift kol çarpışma planlaması yoktur. Normal açılış konum kontrolünün toleransı 0,10 rad'dır; home etkinse en çok 0,25 rad'lık sınırlı başlangıç sınır aşımından kurtulmaya izin verilir, otomatik mod alma ya da hatalı kalibrasyonu düzeltme yapılmaz.

## Kayıt ve tekrar

MCAP 16 boyutludur: sağ yedi eksen + sağ tutucu + sol yedi eksen + sol tutucu. Tutucunun mantıksal konumu slide birimidir, tutucu motorunun radyanı değildir. İki seri port kalıcı iş parçacıklarında paralel yürür, bitince eşleştirilmiş kontrol anlık görüntüsü yayınlanır; XR her turda tek bir anlık görüntüyü paylaşır. Kayıt arka planda kodlanıp diske yazılır; kontrol kuyruğu sınırlıdır, taşma ya da disk hatası açıkça başarısız olur; tutucu geri bildirimi geçersizse kayıt durur. Görüntüler kafa D405, sol bilek JR0001 ve sağ bilek JR0002'dir, 640×480 / 30 fps. Tekrarda yüksek frekanslı kontrol akışı önceliklidir; duraklatma segmentleri ilk kareye ayrı ayrı yaklaşır; eş zamanlı görüntü yalnız kaydedilen içeriği gösterir.

### Örnek: beş adımlı havlu katlama

Beş adımlı havlu katlama, iki kolun kontrolünün aynı süreç içinde tutulmasını gerektirir. Havluyu ve iki kolu başlangıca aldıktan sonra orijinal kaydı 8, 12, 20 ve 26. saniyelerde duraklatarak oynatın. Her segment bitince tutma komutu sürekli gönderilir ve geri bildirim kontrol edilir; operatör 120 saniye içinde `MOVE` yazarsa devam edilir, başka bir giriş ya da Ctrl-C güvenli biçimde durdurur. Beş parçayı ayrı ayrı başlatmayın: her süreç çıkışında motorlar durur ve kollar sarktığı için bir sonraki segmentin başlangıcı uyuşmayabilir.

```bash
python dual/replay.py --episode data/dual_real/episode_0002.mcap \
  --motion-mode --speed 0.5 --dex1-net lo --settle-to-recorded-state \
  --pause-at 8,12,20,26 --no-show-observations --no-print-actions
```

Başlamadan önce iki kolun pozunun tekrar kurtarma aralığında olduğunu, tutucu servislerinin ve dört kameranın normal çalıştığını doğrulayın, fiziksel acil durdurmayı erişilebilir tutun. Duraklamadayken kolu hareket ettirmeyin; arayüzden çıkmak yalnız görüntüyü kapatır, tekrarı durdurmak için tekrar terminalinde Ctrl+C'ye basın.

`grippers.py` tek ve çift tutucu için ortak başlatma uygulamasıdır; `scripts/start_gripper.sh` varsayılan olarak yalnız sağ tarafı, `dual/start_grippers.sh` varsayılan olarak iki tarafı açar. Yinelenen başlatma kontrolü takılabilen fuser'ı artık çağırmaz ve çalışan servisi kendiliğinden sonlandırmaz.

SDK ve tutucu servisi resmi kaynaklı submodule'lerdir; yerel cihaz uyarlamaları `native/dependency_patches/` içinde saklanır ve kurulumda setup tarafından uygulanır, ayrıntılar için [BAGIMLILIK-YAMALARI.md](BAGIMLILIK-YAMALARI.md).

---

> Çift kollu insansı platform arıyorsanız: aynı R1-A7 kolları kullanan [Unitree R1-D](https://www.robotlar.org/tr/insansi-robotlar/unitree-r1-d) · Teleoperasyon kiti: [Unitree D1-T](https://www.robotlar.org/tr/unitree-bilesenler/d1-t) · [Robotlar.org](https://www.robotlar.org)
