FONKSIYON ATM_ParaCekme_Sistemi()
BASLANGIC

    // 1. Kart Girişi ve Okuma Aşaması
    EKRANA_YAZ("Lütfen banka/kredi kartınızı takınız.")
    KART_OKUYUCU_AKTIF()
    KART_OKUNDU = KART_OKUYUCUDAN_VERI_AL(KartVerisi)

    EGER KART_OKUNDU == YANLIS VEYA KartVerisi GECERSIZ ISE
        EKRANA_YAZ("Kart okunamadı veya geçersiz. Lütfen tekrar deneyin.")
        KART_CIKAR()
        BITIR
    SON

    // 2. PIN Doğrulama Aşaması
    PIN_DENEME_HAKKI = 3
    PIN_DOGRULANDI = YANLIS

    TEKRARLA PIN_DENEME_HAKKI > 0 VE PIN_DOGRULANDI == YANLIS
        EKRANA_YAZ("Lütfen PIN kodunuzu giriniz.")
        KULLANICIDAN_GIRIS_AL(GirilenPIN)

        EGER BANKA_SISTEMINE_SOR(KartVerisi, GirilenPIN) == DOGRU ISE
            PIN_DOGRULANDI = DOGRU
        DIGER
            PIN_DENEME_HAKKI = PIN_DENEME_HAKKI - 1
            EKRANA_YAZ("Yanlış PIN. Kalan hakkınız: " + PIN_DENEME_HAKKI)
        SON
    SON_TEKRARLA

    // PIN Hata Kontrolü ve Bloke Etme
    EGER PIN_DOGRULANDI == YANLIS ISE
        EKRANA_YAZ("3 kez yanlış PIN girildi. Kartınız bloke edilmiştir.")
        BANKA_SISTEMINI_BILGILENDIR_KARTI_BLOKE_ET(KartVerisi)
        KART_CIKAR()
        BITIR
    SON

    // 3. Hesap ve İşlem Seçimi Aşaması
    HESAP_BAKIYESI = BANKA_SISTEMINDEN_BAKIYE_AL(KartVerisi)

    EKRANA_YAZ("Lütfen işleminizi seçiniz: (1: Para Çekme, 2: Bakiye, 3: İptal)")
    KULLANICIDAN_GIRIS_AL(Secim)

    EGER Secim == 1 ISE // Para Çekme İşlemi
        
        // 4. Miktar Girişi Aşaması
        MIKTAR_GECERLI = YANLIS
        
        TEKRARLA MIKTAR_GECERLI == YANLIS
            EKRANA_YAZ("Lütfen çekmek istediğiniz miktarı giriniz:")
            KULLANICIDAN_GIRIS_AL(CekilecekMiktar)

            // 5. Miktar Kontrolü
            EGER CekilecekMiktar > 0 VE CekilecekMiktar ATM_VERILEBILIR_KATLARDA_MI() ISE
                MIKTAR_GECERLI = DOGRU
            DIGER
                EKRANA_YAZ("Geçersiz miktar. Lütfen 10, 20 gibi banknot katlarında giriniz.")
            SON
        SON_TEKRARLA

        // 6. Bakiye Yeterlilik Kontrolü
        ISLEM_UCRETI = BANKA_SISTEMINDEN_UCRET_AL(KartVerisi) // Varsa komisyon
        GEREKLI_TOPLAM = CekilecekMiktar + ISLEM_UCRETI
        
        EGER HESAP_BAKIYESI < GEREKLI_TOPLAM ISE
            EKRANA_YAZ("Hesabınızdaki bakiye ( " + HESAP_BAKIYESI + " ) yetersizdir.")
            KART_CIKAR()
            BITIR
        SON

        // 7. ATM Nakit Yeterlilik Kontrolü
        EGER ATM_KASASINDA_YETERLI_NAKIT_VAR_MI(CekilecekMiktar) == YANLIS ISE
            EKRANA_YAZ("Üzgünüz, ATM'de istenen miktarda para bulunmamaktadır.")
            KART_CIKAR()
            BITIR
        SON

        // 8. Hesap ve Banka İşlemlerini Gerçekleştirme
        YENI_BAKIYE = HESAP_BAKIYESI - GEREKLI_TOPLAM
        
        // BANKA SİSTEMİ İLE İLETİŞİM
        EGER BANKA_SISTEMINE_TRANSAKSIYON_BILGISI_GONDER(KartVerisi, CekilecekMiktar, ISLEM_UCRETI) == BASARILI ISE
            // 9. Para Verme Aşaması
            EKRANA_YAZ("İşleminiz gerçekleştiriliyor. Lütfen paranızı alınız.")
            ATM_KASASINDAN_NAKIT_VER(CekilecekMiktar)
            
            // 10. İşlem Fişi ve Kart Çıkarma
            EKRANA_YAZ("Fiş ister misiniz? (E/H)")
            KULLANICIDAN_GIRIS_AL(FisIstegi)

            EGER FisIstegi == 'E' VEYA FisIstegi == 'e' ISE
                ISLEM_FISI_BAS(CekilecekMiktar, ISLEM_UCRETI, YENI_BAKIYE)
            SON

            EKRANA_YAZ("İşleminiz tamamlandı. Lütfen kartınızı alınız.")
            KART_CIKAR()

        DIGER
            EKRANA_YAZ("Banka sisteminde bir hata oluştu. İşleminiz gerçekleştirilemedi.")
            KART_CIKAR()
        SON

    DIGER EGER Secim == 2 ISE // Bakiye Sorgulama (Farklı bir işlem)
        EKRANA_YAZ("Hesabınızdaki güncel bakiye: " + HESAP_BAKIYESI)
        KART_CIKAR()
    
    DIGER // Geçersiz seçim veya iptal
        EKRANA_YAZ("İşlem iptal edildi veya geçersiz seçim yapıldı.")
        KART_CIKAR()
    SON

BITIR
