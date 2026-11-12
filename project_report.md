# Ülke Kredi Risk Sınıflandırması: Kapsamlı Proje Raporu

## 1. Giriş ve Proje Amacı
Bu çalışma, Dünya Bankası WDI (World Development Indicators) ve IMF World Economic Outlook (WEO) veri setleri kullanılarak 2010–2023 yılları arasındaki ülke-yıl düzeyindeki makroekonomik göstergelerin derlenmesi, açık ve şeffaf kurallarla temsili makroekonomik risk sınıflarının oluşturulması ve KNN (K-Nearest Neighbors) sınıflandırıcısı ile modellenmesi amacıyla gerçekleştirilmiştir. Projenin temel amacı, veri sızıntısından arındırılmış, akademik standartlara uygun, şeffaf ve gerçekçi bir temel (baseline) risk sınıflandırma modeli geliştirmektir.

---

## 2. Keşifsel Veri Analizi (EDA) ve Veri Hazırlığı
* **Veri Kapsamı:** Çok yıllı ülke panel verisi (2010–2023) üzerinde çalışılmış, eksik veriler Dünya Bankası ve IMF WEO kaynaklarıyla zenginleştirilmiştir.
* **Dağılım Analizi:** Veri setindeki değişkenlerin genel olarak non-normal (normal dağılmayan) ve outlier (aşırı değer) ağırlıklı olduğu tespit edilmiştir.
- **Hedef Değişken (Target) Mühendisliği:** GDP Büyümesi, Enflasyon, İşsizlik, Kamu Borcu ve Cari Denge olmak üzere 5 temel makroekonomik göstergeden kural tabanlı bir 0–5 risk puanı ve buna bağlı ikili sınıflı (`Güvenli`, `Riskli`) (`risk_class`) türetilmiştir.

---

## 3. Metodoloji ve Strateji: Hedef Sızıntısının Önlenmesi
Makroekonomik modellemelerde karşılaşılan en büyük tuzaklardan biri **hedef sızıntısıdır (Target Leakage)**. Bu projede bu riski bertaraf etmek için iki kritik stratejik karar alınmıştır:
1. **Özellik Seçimi ($X$):** Hedef sınıfın hesaplanmasında kullanılan 5 gösterge model girdilerinden tamamen çıkarılmış; yalnızca hedef puanın doğrudan hesaplanmasında yer almayan kişi başına GSYH (`gdp_per_capita`) değişkeni bağımsız değişken olarak seçilmiştir.
2. **Katı Çapraz Doğrulama (`StratifiedGroupKFold`):** Panel veride aynı ülkenin farklı yıllarının hem eğitim hem test kümesine düşerek ezber (overfitting) yapmasını engellemek amacıyla `StratifiedGroupKFold` kullanılmış; ülke kodu (`iso3`) grup değişkeni olarak atanarak ortak ülke sayısı `0` (sıfır) olacak şekilde katı bir ayrım sağlanmıştır.

---

## 4.Model Eğitimi ve Hiperparametre Optimizasyonu

- **Mimari:** Mesafe tabanlı bir algoritma olan KNN için `Pipeline` yapısı kurulmuş, ölçekleme işlemi (`StandardScaler`) yalnızca her eğitim katında (fold) dinamik olarak gerçekleştirilmiştir.
- **GridSearchCV Sonuçları:**
  - Denenen Komşu Sayıları ($k$): 3 ile 21 arası.
  - En İyi Hiperparametreler: `knn__metric = 'euclidean'`, `knn__n_neighbors = 3`, `knn__weights = 'distance'`.
  - Elde Edilen En İyi CV Macro-F1 Skoru: **0.520** (0.5197).

---

## 5.Test Kümesi Değerlendirmesi ve Hata Analizi

Model, daha önce hiç görmemiş 33 yepyeni ülke üzerinde (441 test gözlemi) test edilmiştir:

- **Genel Başarım:** Test kümesi doğruluğu (Accuracy) ve Macro-F1 skoru, modelin optimize edilmiş hiperparametreleri (`k = 3`, `weights = 'distance'`) doğrultusunda dengeli bir sınıflandırma performansı sergilemiştir.
- **Hata Matrisi (Confusion Matrix) Bulguları:**
  - Model, çoğunluk sınıfı olan "Güvenli" ülkeleri tespit etmede yüksek başarı göstermiştir.
  - Sadece kişi başına gelire (`gdp_per_capita`) dayalı model, makroekonomik şoklar (yüksek borç, enflasyon vb.) nedeniyle gelir düzeyi iyi görünmesine rağmen riskli olan ülkeleri (azınlık sınıfı olan "Riskli" grubu) yakalamakta zorlanmaktadır.

---

## 6. Sınırlılıklar ve Geliştirme Önerileri
* **Proxy Hedef Değişken:** Risk sınıfları kurallı türetilmiş etiketlerdir; gerçek egemen kredi derecelendirmelerinin (S&P, Moody's, Fitch vb.) yerini tutmaz.
* **Tek Değişkenli Sınır:** Hedef sızıntısını önlemek adına diğer makroekonomik göstergelerin modelden çıkarılması, modelin ayırt etme gücünü tek değişkene (`gdp_per_capita`) hapsetmiştir.
* **Gelecek Çalışmalar:** Dış veri kaynağı olarak gerçek uluslararası kredi notları modele dahil edilebilir ve Lojistik Regresyon / Random Forest gibi çok değişkenli alternatif algoritmalar test edilebilir.

---

## 7. Sonuç ve Dışa Aktarım
Bu çalışma, yapay başarılar üreten sızıntılı modeller yerine; metodolojik olarak katı, şeffaf ve gerçeği yansıtan bir temel (baseline) makine öğrenmesi uygulaması sunmaktadır. Model tahminleri tüm veri setine uygulanarak Power BI entegrasyonu için `powerbi_kredi_risk_verisi.csv` dosyası olarak başarıyla dışa aktarılmıştır.

---

## 8. Kaynakça ve Veri Erişimi
* **World Bank, World Development Indicators (WDI):** [World Development Indicators Veri Tabanı](https://databank.worldbank.org/source/world-development-indicators)
* **IMF World Economic Outlook (WEO):** [IMF WEO Veri ve İstatistikleri](https://www.imf.org/en/Publications/WEO)