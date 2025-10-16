// Veri Yapıları:
// Hasta: Kimlik, Ad, Soyad, Telefon vb.
// Doktor: Kimlik, Ad, Soyad, Branş, Çalışma Saatleri vb.
// Randevu: RandevuID, HastaID, DoktorID, Tarih, Saat, Durum (Örn: "Onaylandı", "İptal Edildi")
// ÇalışmaPlanı: DoktorID, Tarih, BaşlangıçSaati, BitişSaati, RandevuAralığı (Örn: 15 dakika)

FONKSİYON RandevuTarihSaatleriniAl(DoktorID, Tarih)
    // Belirli bir doktorun belirli bir tarihteki uygun randevu saatlerini döndürür.

    ÇalışmaPlanı = ÇalışmaPlanlarınıBUL(DoktorID, Tarih)

    EĞER ÇalışmaPlanı BOŞ İSE
        DÖN BOŞ_LİSTE // O gün doktorun çalışması yok
    SON

    Başlangıç = ÇalışmaPlanı.BaşlangıçSaati
    Bitiş = ÇalışmaPlanı.BitişSaati
    Aralık = ÇalışmaPlanı.RandevuAralığı
    
    TümRandevular = RandevularıBUL(DoktorID, Tarih)
    UygunSaatler = BOŞ_LİSTE

    MevcutSaat = Başlangıç

    İKENGESEL MevcutSaat < Bitiş YAP
        RandevuSaati = Tarih + MevcutSaat

        // Bu saatte onaylanmış bir randevu var mı kontrol et
        EĞER TümRandevular İÇİNDE (Randevu.Saat = MevcutSaat VE Randevu.Durum = "Onaylandı") YOK İSE
            UygunSaatler LİSTESİNE Ekle(MevcutSaat)
        SON

        // Sonraki randevu saatine geç
        MevcutSaat = MevcutSaat + Aralık
    SON

    DÖN UygunSaatler

SON FONKSİYON

FONKSİYON RandevuOLUŞTUR(HastaID, DoktorID, Tarih, Saat)
    // Yeni bir randevu oluşturur ve uygunluğunu kontrol eder.

    // 1. Hasta veya Doktor var mı kontrol et (Bu kısım atlandı)
    // 2. Randevu
