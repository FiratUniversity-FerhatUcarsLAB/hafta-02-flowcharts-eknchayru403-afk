ALGORITMA AkıllıEvGüvenlikSistemi

DEĞİŞKENLER:
  Durum = "DevreDışı" // Olası değerler: "Devrede", "DevreDışı", "Alarm"
  HareketAlgılandı = YANLIŞ
  KapıPencereAçık = YANLIŞ
  DumanAlgılandı = YANLIŞ
  GeçerliŞifreGirildi = YANLIŞ
  Zamanlayıcı = 0 // Alarm gecikmesi için

BAŞLAT:
  Durum = "DevreDışı"
  SürekliDöngüdeÇalış()

FONKSİYON SürekliDöngüdeÇalış:
  DÖNGÜ SÜRESİNCE DOĞRU
    SensörleriOku()
    SistemiKontrolEt()
    Bekle(1 Saniye) // Verimlilik için bekleme

FONKSİYON SensörleriOku:
  // Fiziksel sensörlerden verileri simüle et/oku
  HareketAlgılandı = HareketSensöründenOku()
  KapıPencereAçık = KapıPencereSensörlerindenOku()
  DumanAlgılandı = DumanSensöründenOku()

FONKSİYON SistemiKontrolEt:
  EĞER Durum EŞİTTİR "Devrede" İSE
    EĞER DumanAlgılandı EŞİTTİR DOĞRU İSE
      AlarmıÇalıştır("Yangın")
      Durum = "Alarm"
    YOKSA EĞER HareketAlgılandı EŞİTTİR DOĞRU VEYA KapıPencereAçık EŞİTTİR DOĞRU İSE
      EĞER Zamanlayıcı EŞİTTİR 0 İSE
        ZamanlayıcıyıBaşlat(30) // 30 saniye gecikme
        KullanıcıyaBildir("Giriş Algılandı. Alarm devreye girmek üzere.")
      YOKSA EĞER Zamanlayıcı BÜYÜKTÜR 0 İSE
        Zamanlayıcı = Zamanlayıcı - 1
        EĞER ŞifreGirişiKontrolEt() EŞİTTİR DOĞRU İSE
          AlarmıİptalEt()
          Durum = "DevreDışı" // Şifre girildi, sistemi devre dışı bırak
          Zamanlayıcı = 0
        YOKSA EĞER Zamanlayıcı EŞİTTİR 0 İSE
          AlarmıÇalıştır("İzinsiz Giriş")
          Durum = "Alarm"
        SON EĞER
      SON EĞER
    YOKSA // Hiçbir şey algılanmadı ve gecikme yoksa
      Zamanlayıcı = 0
    SON EĞER

  YOKSA EĞER Durum EŞİTTİR "Alarm" İSE
    // Alarm zaten çalışıyor, ek eylem gerekebilir
    EĞER ŞifreGirişiKontrolEt() EŞİTTİR DOĞRU İSE
      AlarmıİptalEt()
      Durum = "DevreDışı"
    YOKSA EĞER AcilDurumDüğmesiBasıldı() EŞİTTİR DOĞRU İSE
      AlarmıİptalEt()
      Durum = "DevreDışı" // Veya sadece alarmı sustur

  YOKSA EĞER Durum EŞİTTİR "DevreDışı" İSE
    // Sistem devre dışı, sadece izle
    EĞER UzaktanKomutAlındı("SistemiDevreyeAl") EŞİTTİR DOĞRU İSE
      Durum = "Devrede"
      KullanıcıyaBildir("Sistem Devrede.")
    YOKSA EĞER DumanAlgılandı EŞİTTİR DOĞRU İSE
      AlarmıÇalıştır("Yangın") // Devre dışı olsa bile yangın alarmı çalışmalı
      Durum = "Alarm"
    SON EĞER
  SON EĞER

// YARDIMCI FONKSİYONLAR (detayları atlanmıştır)

FONKSİYON AlarmıÇalıştır(Tip):
  SirenleriAç()
  PolisiAra(Tip) // veya İtfaiyeyiAra(Tip)
  KullanıcıyaBildir("!!! " + Tip + " ALARMI !!!")

FONKSİYON AlarmıİptalEt:
  SirenleriKapat()
  PolisiAramayıİptalEt()
  KullanıcıyaBildir("Alarm İptal Edildi.")

FONKSİYON ŞifreGirişiKontrolEt:
  // Şifre paneli veya uygulama üzerinden geçerli şifre girilip girilmediğini kontrol eder
  DÖNDÜR GeçerliŞifreGirildi

FONKSİYON KullanıcıyaBildir(Mesaj):
  // Mobil uygulama bildirimi veya e-posta gönderir
  EKRANA_YAZ(Mesaj)

FONKSİYON HareketSensöründenOku:
  // Sensör okuma işlemini simüle eder
  DÖNDÜR YANLIŞ // Örnek olarak

FONKSİYON KapıPencereSensörlerindenOku:
  // Sensör okuma işlemini simüle eder
  DÖNDÜR YANLIŞ // Örnek olarak

FONKSİYON DumanSensöründenOku:
  // Sensör okuma işlemini simüle eder
  DÖNDÜR YANLIŞ // Örnek olarak

FONKSİYON UzaktanKomutAlındı(Komut):
  // Uzaktan komut gelip gelmediğini kontrol eder
  DÖNDÜR YANLIŞ // Örnek olarak

FONKSİYON AcilDurumDüğmesiBasıldı:
  // Fiziksel veya sanal panik düğmesinin basılıp basılmadığını kontrol eder
  DÖNDÜR YANLIŞ // Örnek olarak

FONKSİYON ZamanlayıcıyıBaşlat(Saniye):
  // Gecikme zamanlayıcısını ayarlar
  Zamanlayıcı = Saniye

SON ALGORİTMA
