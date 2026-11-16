# Resmi Bağımlılık Sürümleri ve Yerel Temel Hat Yamaları

[README'ye dön](../README.md) · Kaynak: [unitreerobotics/R1-7A `native/dependency_patches/README.md`](https://github.com/unitreerobotics/R1-7A/blob/main/native/dependency_patches/README.md) · Türkçe çeviri: [Robotlar.org](https://www.robotlar.org)

- `r1_arm_sdk.patch`: resmi `unitreerobotics/r1-7a_sdk` deposunun `2c89fe4f60f0c98f1bc06795cb2477cc73d0c92c` commit'ine uygulanır. Mevcut yerel hat tanılamasını, kalibrasyon araçlarını ve olay güdümlü seri port uygulamasını korur.
- `dex1_1_service.patch`: resmi `unitreerobotics/dex1_1_service` deposunun `ced3216290bb22dae356213993786beeaa7143ab` commit'ine uygulanır. Açık port seçimini, motor ID / DDS taraf ayrımını, komut başlatmayı, tanılamayı ve yerel SDK2 derleme entegrasyonunu korur.

`setup.sh`, her yamayı uygulamadan önce zaten uygulanıp uygulanmadığını kontrol eder. Çakışma olursa değişikliklerin üzerine yazmak yerine kurulum durur. Bu yamalar kaynak değişikliği içerir; yerel kalibrasyon verisi ya da derlenmiş çalışma zamanı kitaplığı içermez.

Etkin iş istasyonu hala `scripts/real_env.sh` tarafından seçilen harici SDK/kalibrasyonu ve mevcut bridge'i kullanır. Yalnız submodule Git meta verisini güncellemek bu çalışma zamanı dosyalarını değiştirmez.

- `xr_pybind.patch`: `YanjieZe/XRoboToolkit-PC-Service-Pybind` deposunun `75cb1130ac63e76d8f7e7788049be415e8be44f2` commit'ine uygulanır; atomik kare anlık görüntülerini, cihaz başına meta veriyi, düğme sayılarını ve geri çağırma zamanlamasını korur.
- `xr_pc_service.patch`: `XR-Robotics/XRoboToolkit-PC-Service` deposunun `85bac4dbc1fd5cef42c74a160d9c30aa3491f122` commit'ine uygulanır; en son kare posta kutusunu ve yönlendirme değişikliklerini korur.

Setup, yerel istemci SDK'sını ve Python bağlamasını derlemeden önce XR yamalarını uygular. Tam masaüstü PC Service'i derlemek/kurmak ayrı bir adım olarak kalır.

---

> Unitree R1-7A, Dex1-1 ve Unitree SDK için Türkçe teknik destek: [Robotlar.org](https://www.robotlar.org/tr/unitree-bilesenler/r1-7a) · [Unitree SDK rehberi](https://www.robotlar.org/unitree/sdk-rehberi)
