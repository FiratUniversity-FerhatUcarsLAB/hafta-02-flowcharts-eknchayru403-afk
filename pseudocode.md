// Gerekli Veri Yapıları:
// SEPET: Ürünleri ve miktarlarını tutan bir sözlük (dictionary) veya hash map.
//        Anahtar: Ürün ID'si (veya Adı), Değer: Miktar.
// ÜRÜN_VERİSİ: Tüm ürünlerin detaylarını (ID, Ad, Fiyat, Stok) tutan bir veritabanı/liste.

PROSEDÜR Sepet_Başlat()
    SEPET = Yeni Boş Sözlük (veya Hash Map)
SON PROSEDÜR

PROSEDÜR Ürün_Ekle(Ürün_ID, Miktar)
    // 1. Miktarın geçerli olup olmadığını kontrol et
    EĞER Miktar <= 0 İSE
        Hata_Mesajı("Miktar sıfırdan büyük olmalıdır.")
        GERİ DÖN
    SON EĞER

    // 2. Ürünün stok durumunu kontrol et (ÜRÜN_VERİSİ'nden alınır)
    ÜRÜN = ÜRÜN_VERİSİ'nden Ürün_ID'ye ait bilgiyi getir
    
    EĞER ÜRÜN bulunamazsa İSE
        Hata_Mesajı("Ürün ID'si geçersiz.")
        GERİ DÖN
    SON EĞER

    MEVCUT_SEPET_MİKTARI = SEPET'te Ürün_ID'ye ait miktar (Eğer yoksa 0)
    YENİ_TOPLAM_MİKTAR = MEVCUT_SEPET_MİKTARI + Miktar

    EĞER YENİ_TOPLAM_MİKTAR > ÜRÜN.Stok İSE
        Hata_Mesajı("Yeterli stok yok. Sepetinizdeki ürün miktarı stok limitini aşıyor.")
        GERİ DÖN
    SON EĞER

    // 3. Sepete ekleme veya miktarı güncelleme
    SEPET[Ürün_ID] = YENİ_TOPLAM_MİKTAR
    Başarı_Mesajı(Ürün.Ad + " ürünü sepete eklendi/güncellendi. Yeni Miktar: " + YENİ_TOPLAM_MİKTAR)

SON PROSEDÜR

PROSEDÜR Ürün_Çıkar(Ürün_ID, Miktar)
    // 1. Ürünün sepette olup olmadığını kontrol et
    EĞER Ürün_ID SEPET'te Yoksa İSE
        Hata_Mesajı("Ürün sepetinizde bulunmuyor.")
        GERİ DÖN
    SON EĞER

    // 2. Çıkarılacak miktarı kontrol et
    MEVCUT_MİKTAR = SEPET[Ürün_ID]
    
    EĞER Miktar >= MEVCUT_MİKTAR İSE
        // Sepetten tamamen kaldır
        SEPET'ten Ürün_ID'yi Sil
        Başarı_Mesajı(Ürün_ID'ye ait ürün sepetten tamamen kaldırıldı.)
    BAŞKA DURUMDA
        // Miktarı azalt
        SEPET[Ürün_ID] = MEVCUT_MİKTAR - Miktar
        Başarı_Mesajı(Ürün_ID'ye ait ürün miktarı azaltıldı. Yeni Miktar: " + SEPET[Ürün_ID])
    SON EĞER

SON PROSEDÜR

FONKSİYON Sepet_Toplam_Tutar_Hesapla()
    TOPLAM_TUTAR = 0
    
    HER Ürün_ID ve Miktar İçin SEPET'te DÖNGÜ YAP
        ÜRÜN = ÜRÜN_VERİSİ'nden Ürün_ID'ye ait bilgiyi getir
        
        EĞER ÜRÜN bulunursa İSE
            ÜRÜN_FİYATI = ÜRÜN.Fiyat
            KALEM_TUTARI = Miktar * ÜRÜN_FİYATI
            TOPLAM_TUTAR = TOPLAM_TUTAR + KALEM_TUTARI
        // NOT: İndirimler, vergi vb. bu aşamada eklenebilir.
        SON EĞER
    SON DÖNGÜ

    GERİ DÖN TOPLAM_TUTAR

SON FONKSİYON

PROSEDÜR Sepeti_Görüntüle()
    EĞER SEPET boşsa İSE
        Ekrana Yazdır("Sepetinizde ürün bulunmamaktadır.")
        GERİ DÖN
    SON EĞER
    
    Ekrana Yazdır("--- ALIŞVERİŞ SEPETİ ---")
    
    HER Ürün_ID ve Miktar İçin SEPET'te DÖNGÜ YAP
        ÜRÜN = ÜRÜN_VERİSİ'nden Ürün_ID'ye ait bilgiyi getir
        EĞER ÜRÜN bulunursa İSE
            Ekrana Yazdır(Ürün.Ad + " | Miktar: " + Miktar + " | Birim Fiyat: " + Ürün.Fiyat + " | Ara Toplam: " + (Miktar * Ürün.Fiyat))
        BAŞKA DURUMDA
            Ekrana Yazdır("Bilinmeyen Ürün ID: " + Ürün_ID + " | Miktar: " + Miktar)
        SON EĞER
    SON DÖNGÜ
    
    TOPLAM = Sepet_Toplam_Tutar_Hesapla()
    Ekrana Yazdır("-----------------------------")
    Ekrana Yazdır("Genel Toplam Tutar: " + TOPLAM)
SON PROSEDÜR

// Örnek Kullanım Akışı:
// Sepet_Başlat()
// Ürün_Ekle(Ürün_ID: 101, Miktar: 2) // Ürün A, Fiyat: 50 TL
// Ürün_Ekle(Ürün_ID: 205, Miktar: 1) // Ürün B, Fiyat: 120 TL
// Ürün_Ekle(Ürün_ID: 101, Miktar: 1) // Ürün A miktarı 3'e yükseldi
// Ürün_Çıkar(Ürün_ID: 205, Miktar: 1) // Ürün B sepetten kalktı
// Sepeti_Görüntüle() // Görüntü: Ürün A (Miktar: 3, Toplam: 150 TL)
// Sepet_Toplam_Tutar_Hesapla() // Sonuç: 150
