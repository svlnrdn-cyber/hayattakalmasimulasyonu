<pre> 
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
#include &lt;time.h&gt;
#include &lt;ctype.h&gt;

// --- Yeni ve Mevcut Durum Değişkenleri ---
int saglik = 100;      // Oyuncunun canı (0-100)
int enerji = 100;      // Oyuncunun enerjisi (0-100)
int aclik = 100;       // YENİ: Açlık Durumu (0-100). 0'a yaklaşınca tehlike.
int yemek_sayisi = 3;  // Yemek stoğu
int yara_bandi_sayisi = 0; // YENİ: Yara bandı stoğu
int sığınak_var = 0;   // Sığınak durumu (0: Yok, 1: Var)
int avlanma_sayaci = 0; // YENİ: Her 3 avlanmada saldırı için sayaç
int simülasyon_basladi = 0; // YENİ: Sığınak seçimi zorunluluğu için kontrol

// --- Fonksiyon Prototipleri ---
void temizle_sinirla();
void envanter_goster();
void dinlen();
void avlan();
void sığınak_ara();
void tehlike_dalga_simulasyonu();
void şifreli_ilerleme();
void yemek_ye(); // YENİ: Yemek yeme fonksiyonu
void yara_bandi_kullan(); // YENİ: Yara bandı kullanma fonksiyonu

// ====================================================================
// --- ANA FONKSİYON ---
// ====================================================================
int main() {
    srand(time(NULL));

    char komut;
    int simülasyon_devam = 1;
    
    printf("--- KARAKTER TABANLI HAYATTA KALMA SİMÜLATÖRÜ ---\n");
    
    // YENİ ZORUNLULUK: Sığınak seçimi zorunluluğu
    while (sığınak_var == 0) {
        printf("\n!!! HAYATTA KALMAK İÇİN ÖNCE S (Sığınak Ara) KOMUTU İLE GÜVENLİ BİR YER BULMALISINIZ. !!!\n");
        printf("Sığınak arayın (S) veya Çıkış (X): ");
        scanf(" %c", &komut);
        char kucuk_komut = tolower(komut);

        if (kucuk_komut == 's') {
            sığınak_ara();
        } else if (kucuk_komut == 'x') {
            printf("Simülasyon sonlandırılıyor...\n");
            return 0;
        } else {
            printf("Geçersiz komut. Sadece S veya X kullanabilirsiniz.\n");
        }
    }
    
    // Sığınak bulunduktan sonra ana simülasyon başlar
    simülasyon_basladi = 1;
    printf("\n--- Sığınak Başarıyla Kuruldu. Hayatta Kalma Başlıyor. ---\n");
    printf("Komutlar: A, S, R, E, F, P, Y (Yemek Ye), B (Yara Bandı Kullan), X (Çıkış)\n");

    // Simülasyonun ana döngüsü (DO-WHILE zorunluluğu)
    do {
        temizle_sinirla();
        
        if (saglik <= 0 || enerji <= 0) {
            printf("\n!!! HAYATTA KALAMADINIZ. OYUN BİTTİ. !!!\n");
            simülasyon_devam = 0;
            break;
        }

        // YENİ KURAL: Açlık kontrolü (20'nin altında kritik durum)
        if (aclik <= 20) {
            printf("\n!!! KRİTİK UYARI: AÇLIK SEVİYENİZ ÇOK DÜŞÜK (%d)! YEMEK YEMEZSENİZ ÖLECEKSİNİZ. !!!\n", aclik);
        }
        
        // YENİ KURAL: Açlık 20 altındayken Y harici komut basma
        if (aclik <= 20 && komut != 'y' && komut != 'Y' && komut != 'X' && komut != 'x') {
             printf("\n!!! AÇLIK KRİTİK DÜZEYDE. YEMEK YEMEK DIŞINDA BİR ŞEY YAPAMADINIZ VE HAYATINIZI KAYBETTİNİZ. !!!\n");
             simülasyon_devam = 0;
             break;
        }
        
        envanter_goster();
        printf("------------------------------------------\n");
        printf("Eyleminizi girin (X: Çıkış): ");
        
        if (scanf(" %c", &komut) != 1) { komut = ' '; }

        char kucuk_komut = tolower(komut); 
        
        // Komut işleme sistemi (SWITCH-CASE zorunluluğu)
        switch (kucuk_komut) {
            case 'a': avlan(); break; 
            case 's': sığınak_ara(); break; 
            case 'r': dinlen(); break; 
            case 'e': printf("Durumunuz yukarıda gösterilmiştir.\n"); break; 
            case 'f': tehlike_dalga_simulasyonu(); break; 
            case 'p': şifreli_ilerleme(); break;
            case 'y': yemek_ye(); break; // YENİ Komut
            case 'b': yara_bandi_kullan(); break; // YENİ Komut
            case 'x': printf("Simülasyon sonlandırılıyor...\n"); simülasyon_devam = 0; break; 
            default: printf("!!! Geçersiz komut. Lütfen geçerli bir harf girin. !!!\n");
        }

    } while (simülasyon_devam);

    return 0;
}

// ====================================================================
// --- YARDIMCI VE KOMUT FONKSİYONLARI ---
// ====================================================================

/**
 * Açlık, Sağlık ve Enerjiyi 0 ile 100 arasında sınırlar.
 */
void temizle_sinirla() {
    if (saglik < 0) saglik = 0;
    if (saglik > 100) saglik = 100;

    if (enerji < 0) enerji = 0;
    if (enerji > 100) enerji = 100;

    if (aclik < 0) aclik = 0;
    if (aclik > 100) aclik = 100;
}

/**
 * Mevcut oyuncu durumunu ekrana yazdırır.
 */
void envanter_goster() {
    printf("\n--- Mevcut Durum ---\n");
    printf("Sağlık: %d/100 | Enerji: %d/100 | Açlık: %d/100\n", saglik, enerji, aclik);
    printf("Yemek Stoğu: %d | Yara Bandı: %d | Sığınak: %s\n", yemek_sayisi, yara_bandi_sayisi, sığınak_var ? "KURULU" : "YOK");
}

/**
 * YENİ KOMUT: Yemek yeme ve açlığı artırma.
 */
void yemek_ye() {
    if (yemek_sayisi > 0) {
        int aclik_kazanci = 30;
        aclik += aclik_kazanci;
        yemek_sayisi--;
        printf("\nBir yemek yediniz. Açlık seviyeniz %d arttı. Kalan yemek: %d\n", aclik_kazanci, yemek_sayisi);
    } else {
        printf("\nYemeğiniz yok. Yemek yiyemezsiniz.\n");
    }
}

/**
 * YENİ KOMUT: Yara bandı kullanma ve sağlığı %100 yapma.
 */
void yara_bandi_kullan() {
    if (yara_bandi_sayisi > 0 && saglik < 100) {
        saglik = 100; // Sağlığı %100'e geri çıkar
        yara_bandi_sayisi--;
        printf("\nYara bandı kullandınız. Sağlığınız anında 100'e yükseldi! Kalan yara bandı: %d\n", yara_bandi_sayisi);
    } else if (saglik == 100) {
        printf("\nSağlığınız zaten %d. Yara bandı kullanmaya gerek yok.\n", saglik);
    } else {
        printf("\nYara bandınız yok.\n");
    }
}

/**
 * Genel hareket sonrası durum güncellemeleri
 * YENİ KURAL: Her hareket açlığı ve enerjiyi düşürür.
 */
void hareket_yapildi() {
    if (simülasyon_basladi) {
        enerji -= 10;
        aclik -= 15;
    }
}

/**
 * A Komutu: Avlan.
 */
void avlan() {
    hareket_yapildi(); // Enerji ve Açlık düşüşü

    printf("\nAvlanmaya çıktınız. Enerji ve Açlık harcadınız.\n");

    // YENİ KURAL: Her 3 avlanmada bir saldırı
    avlanma_sayaci++;
    if (avlanma_sayaci % 3 == 0) {
        int hasar = (rand() % 20) + 10; // 10-30 arası hasar
        saglik -= hasar;
        printf("!!! **HAYVAN SALDIRISI** !!! Saldırıya uğradınız. Sağlık -%d.\n", hasar);
        avlanma_sayaci = 0; // Sayacı sıfırla
    }

    int sans = rand() % 100; 

    if (sans > 40) { // %60 başarılı avlanma şansı
        yemek_sayisi += 2;
        printf("Başarılı bir av oldu. 2 yemek kazandınız.\n");

        // YENİ KURAL: Yara bandı bulma durumu
        if (rand() % 100 < 15) { // %15 yara bandı bulma şansı
            yara_bandi_sayisi++;
            printf("** Şanslısınız! Bir YARA BANDI buldunuz! **\n");
        }
    } else {
        printf("Avlanma başarısız oldu. Hiçbir şey bulamadınız.\n");
    }
}

/**
 * R Komutu: Dinlen.
 */
void dinlen() {
    hareket_yapildi(); // Enerji ve Açlık düşüşü

    int enerji_kazanci = 25;
    enerji += enerji_kazanci;
    
    printf("\nDinleniyorsunuz. Enerjiniz %d arttı.\n", enerji_kazanci);

    if (aclik < 50) {
        printf("Ancak aç olduğunuz için dinlenmeniz verimli olmadı.\n");
    } else if (saglik < 100) {
        saglik += 5; // Hafif sağlık yenilenmesi
        printf("Sağlığınız hafifçe yenilendi (+5 Sağlık).\n");
    }
}

/**
 * S Komutu: Sığınak arama (İlk başta zorunlu).
 */
void sığınak_ara() {
    // Sadece simülasyon başladıktan sonra maliyet uygula
    if (simülasyon_basladi) {
        hareket_yapildi(); 
    }
    
    if (sığınak_var == 1) {
        printf("\nZaten güvenli bir sığınağınız var. Enerjiniz boşa harcanmadı.\n");
        return;
    }
    
    printf("\nGüvenli bir sığınak arıyorsunuz...\n");

    if (rand() % 100 > 50) { 
        sığınak_var = 1;
        printf("TEBRİKLER! **SIĞINAK BULDUNUZ** ve kurdunuz.\n");
    } else {
        printf("Bu bölge sığınak için uygun değilmiş. Başarısız oldunuz.\n");
    }
}

/**
 * F Komutu: Tehlike dalgası.
 */
void tehlike_dalga_simulasyonu() {
    hareket_yapildi(); // Enerji ve Açlık düşüşü
    
    printf("\n*** TEHLİKE DALGASI BAŞLADI! 3 aşamalı saldırı. ***\n");
    
    for (int i = 1; i <= 3; i++) {
        if (sığınak_var == 1) { 
            enerji -= 5;
            printf("Sığınak sizi koruyor. Çok az enerji kaybettiniz.\n");
        } else {
            saglik -= 15;
            enerji -= 10;
            printf("**Hasar Aldınız!** Sığınak yokluğunda ağır darbe aldınız. Sağlık -15, Enerji -10.\n");
        }
        
        if (saglik <= 0) {
            printf("Tehlike sırasında can kaybından bayıldınız...\n");
            break;
        }
    }
    printf("*** TEHLİKE DALGASI SONA ERDİ ***\n");
}

/**
 * P Komutu: Şifreli ilerleme.
 */
void şifreli_ilerleme() {
    hareket_yapildi(); // Enerji ve Açlık düşüşü

    char girilen_karakter;
    const char GEREKEN_KARAKTER = 'C';
    int engel_asildi = 0;

    printf("\n** Şifreli Kapı Önündesiniz. **\n");
    
    do {
        printf("Şifre karakterini girin: ");
        scanf(" %c", &girilen_karakter);

        if (tolower(girilen_karakter) == tolower(GEREKEN_KARAKTER)) {
            engel_asildi = 1;
            printf("Şifre doğru! Kapı açıldı ve 15 Enerji kazandınız.\n");
            enerji += 15;
        } else {
            saglik -= 5;
            printf("Yanlış deneme. Elektrik şoku aldınız ve 5 sağlık kaybettiniz.\n");
            
            if (saglik <= 0) {
                printf("Çok fazla deneme yaptınız ve bilincinizi kaybettiniz.\n");
                break;
            }
        }
    } while (!engel_asildi);
}
</pre>
