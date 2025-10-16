FONKSİYON Ders_Kayit_Islemi(Ogrenci_ID)
    // 1. Başlangıç ve Durum Kontrolleri
    OGRENCI = Ogrenci_Bilgilerini_Getir(Ogrenci_ID)

    EĞER OGRENCI.KayıtDurumu != "Aktif" VEYA OGRENCI.BorcDurumu == "Borçlu" İSE
        EKRANA_YAZ("Hata: Kayıt aktif değil veya borç durumu kayıt yapmaya izin vermiyor.")
        GERİ_DÖN HATA
    SON_EĞER

    SECILEN_DERSLER_LISTESI = Ogrencinin_Mevcut_Derslerini_Getir(Ogrenci_ID)
    MAKS_KREDI_LIMITI = OGRENCI.Bölüm.MaksimumKrediLimitini_Getir()
    GÜNCEL_KREDI_TOPLAMI = Hesapla_Kredi_Toplami(SECILEN_DERSLER_LISTESI)

    // 2. Ders Seçim Döngüsü
    DÖNGÜ: Kullanıcı Yeni Ders Seçimi Yapmak İstediği SÜRECE
        EKRANA_YAZ("Alınabilecek dersleri listele veya ders kodu girin.")
        YENI_DERS_KODU = Kullanicidan_Ders_Kodu_Al()

        EĞER YENI_DERS_KODU == "BITIR" İSE
            DÖNGÜYÜ_SONLANDIR
        SON_EĞER

        YENI_DERS = Ders_Bilgilerini_Getir(YENI_DERS_KODU)

        EĞER YENI_DERS == BOŞ İSE
            EKRANA_YAZ("Hata: Geçersiz ders kodu.")
            DEVAM_ET
        SON_EĞER

        // 3. Ders Kontrolleri
        KONTROL_SONUCU = Kontrol_Et_Ders_Ekleme(OGRENCI, YENI_DERS, SECILEN_DERSLER_LISTESI, MAKS_KREDI_LIMITI, GÜNCEL_KREDI_TOPLAMI)

        EĞER KONTROL_SONUCU == "Başarılı" İSE
            // Ders başarılı bir şekilde eklenebilir
            Kontenjan_Azalt(YENI_DERS)
            SECILEN_DERSLER_LISTESI.Ekle(YENI_DERS)
            GÜNCEL_KREDI_TOPLAMI = GÜNCEL_KREDI_TOPLAMI + YENI_DERS.Kredi
            EKRANA_YAZ(YENI_DERS.Adı + " dersi seçildi.")
        AKSİ_HALDE
            EKRANA_YAZ("Hata: Ders eklenemedi. Sebep: " + KONTROL_SONUCU)
        SON_EĞER
    SON_DÖNGÜ

    // 4. Kaydı Taslak Olarak Kaydetme
    Veritabanina_Taslak_Kaydet(Ogrenci_ID, SECILEN_DERSLER_LISTESI)

    // 5. Danışman Onayı Aşaması
    EKRANA_YAZ("Ders seçimi tamamlandı. Danışman onayına gönderiliyor.")
    Danisman_Onay_Talebi_Gonder(Ogrenci_ID)

    // Kayıt, Danışman Onayı sonrasında kesinleşir
    EKRANA_YAZ("Kayıt, danışman onayı bekliyor. Onay sonrası kesinleşecektir.")

GERİ_DÖN BAŞARILI
BİTİR

// Ders Ekleme Kısıtlamalarını Kontrol Eden Alt Fonksiyon
FONKSİYON Kontrol_Et_Ders_Ekleme(Ogrenci, Yeni_Ders, Mevcut_Dersler, Maks_Kredi, Guncel_Kredi)
    // a. Önkoşul Kontrolü
    EĞER Yeni_Ders.Önkoşul_Var İSE
        EĞER NOT Ogrencinin_Transkriptinde_Onkosul_Basarili(Ogrenci, Yeni_Ders) İSE
            GERİ_DÖN "Önkoşul Başarısızlığı"
        SON_EĞER
    SON_EĞER

    // b. Kontenjan Kontrolü
    EĞER Yeni_Ders.Kontenjan <= 0 İSE
        GERİ_DÖN "Kontenjan Dolu"
    SON_EĞER

    // c. Çakışma Kontrolü
    HER Mevcut_Ders İÇİN YAP
        EĞER Ders_Saatleri_Cakışıyor(Yeni_Ders, Mevcut_Ders) İSE
            GERİ_DÖN "Ders Saati Çakışması"
        SON_EĞER
    SON_HER

    // d. Ders Tekrarı Kontrolü (Başarılı ders tekrar seçilemez, başarısız ders önceliklidir)
    EĞER Ogrencinin_Transkriptinde_Ders_Basarili(Ogrenci, Yeni_Ders) İSE
        GERİ_DÖN "Daha Önce Başarılı Olundu"
    SON_EĞER

    // e. Kredi Limiti Kontrolü
    EĞER Guncel_Kredi + Yeni_Ders.Kredi > Maks_Kredi İSE
        GERİ_DÖN "Kredi Limiti Aşıldı"
    SON_EĞER

    GERİ_DÖN "Başarılı"
BİTİR
