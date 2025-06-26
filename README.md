# 📊 BettingDataModeling - Veri Analizi ve Makine Öğrenmesi Projesi

---

## 🚀 Proje Hakkında

Bu projede, bahis sektörüne ait 500 kullanıcılık veri seti üzerinde kapsamlı veri analizi, müşteri segmentasyonu, outlier tespiti ve makine öğrenmesi modelleri kullanılarak kullanıcı davranışları ve bahis miktarları incelenmiştir.  
Amaç, kullanıcı segmentlerini anlamak, risk yönetimi yapmak ve iş stratejileri geliştirmektir.

---

## 📂 Veri Seti Genel Özeti

| Özellik         | Açıklama                                  | Veri Tipi    |
|-----------------|-------------------------------------------|--------------|
| User_ID         | Benzersiz kullanıcı kimliği                | Object       |
| Signup_Date     | Kayıt tarihi                              | Datetime     |
| Bet_Amount      | Toplam bahis/harcama miktarı              | Float        |
| Region          | Coğrafi bölge (6 farklı kıta/ülke grubu) | Kategorik    |
| Account_Status  | Hesap durumu (Aktif, Askıya alınmış, Fraudulent) | Kategorik    |
| Login_Count     | Sisteme giriş sayısı                      | Integer      |
| Customer_Type   | Müşteri tipi (Yeni, Düzenli, Yüksek değerli) | Kategorik    |

---

## 🔍 Veri Kalitesi ve Dağılımlar

- Eksik veri bulunmamaktadır (500/500 kayıt eksiksiz).
- **Bet_Amount** ve **Login_Count** değişkenlerinin yayılımı yüksektir (Coefficient of Variation > %57).
- Bölgesel dağılım dengelidir; en fazla kullanıcı Güney Amerika’da (96), en az Avrupa’da (74).
- Fraudulent ve Suspended hesapların sayısı aktiflere yakın, bu risk yönetimi açısından kritik.
- Müşteri tipleri arasında yeni ve düzenli kullanıcılar neredeyse eşit, yüksek değerli kullanıcılar azınlıktadır.

---

## 📈 Veri Analizi Özeti ve Model Önerileri

| 🔍 Analiz Başlığı             | 🧪 Test / Yöntem                        | 📊 Sonuç / İstatistik                       | 📌 Yorum / Açıklama                                           |
|------------------------------|----------------------------------------|--------------------------------------------|--------------------------------------------------------------|
| 📏 **Dağılım Kontrolleri**     | Shapiro-Wilk Testi                     | Bet_Amount p=1.01e-11 / Login_Count p=3.59e-11 | Normal dağılım yok, non-parametrik yöntem önerilir.         |
|                              | Çarpıklık (Skewness)                   | Bet=0.032 / Login=-0.031                   | Değişkenler simetrik.                                        |
|                              | Basıklık (Kurtosis)                    | Bet=-1.229 / Login=-1.170                  | Her iki dağılım da düz (platykurtik).                        |
| 🔗 **Değişken İlişkileri**     | Spearman Korelasyon                    | -0.0387, p=0.388                            | Bet_Amount ile Login_Count arasında ilişki yok.              |
|                              | Kendall's Tau                          | -0.0237, p=0.430                            | Anlamsız ilişki, lineer model uygun değil.                   |
| ⚖️ **Varyans Homojenliği**     | Levene Testi                           | p > 0.4 tüm gruplar                         | Region, Account_Status ve Customer_Type için eşit varyans.   |
| 🧩 **Gruplar Arası Fark**       | Kruskal-Wallis                         | Tüm p > 0.17                                | Bet_Amount gruplara göre anlamlı farklılık göstermiyor.      |
| 🧠 **Bilgi Katkısı (MI)**       | Mutual Information                     | Max: Customer_Type_High-value = 0.0186     | Genelde düşük, fakat High-value kullanıcı en anlamlı.        |
|                              |                                        | Diğer değişkenler ≤ 0.0066                 | Çok düşük bilgi katkısı → feature engineering önerilir.      |
| 🧮 **Kategorik İlişki**         | Ki-Kare & Cramér's V                   | Tüm p > 0.3, V ≈ 0                          | Kategorik değişkenler arasında anlamlı ilişki yok.           |
| 🔁 **Çoklu Bağlantı (VIF)**     | VIF Analizi                            | Bet_Amount = 2.25 / Login_Count = 2.25     | Multicollinearity yok, model için güvenli.                  |
| ✅ **Model Önerisi**           | -                                      | -                                          | Parametrik olmayan ve ağaç tabanlı modeller önerilir.        |
| 📊 **İş Zekâsı Yorumu**         | -                                      | -                                          | Bet_Amount'ı etkileyen güçlü prediktörler eksik.             |


---

## 🛠️ Model Süreci ve Performans Raporu

### Aykırı Değer Tespiti

| Yöntem                  | Aykırı Değer Sayısı |
|-------------------------|---------------------|
| Z-Score                 | 0                   |
| IQR                     | 0                   |
| Mahalanobis             | 0                   |
| Isolation Forest        | 5                   |
| Local Outlier Factor    | 5                   |
| One-Class SVM           | 6                   |

- Aykırı değerler çıkarıldıktan sonra veri seti 494 satıra düştü.
- Model performansında anlamlı fark olmadığı için orijinal veri seti kullanıldı.

### Model Performansları

## 🤖 Model Performansları

### 📊 Performans Karşılaştırma Tablosu

| 🧠 Model                     | 📉 Train RMSE | 📈 Train R² | 🧪 Test RMSE | 🧪 Test R² | 📝 Yorum                                                                 |
|-----------------------------|---------------|-------------|--------------|------------|---------------------------------------------------------------------------|
| **Random Forest (İlk)**     | 1058.17       | 0.471       | 1510.08      | -0.089     | Aşırı overfitting, model eğitim verisini ezberlemiş; testte başarısız.   |
| **Random Forest (Optimize)**| 1294.46       | 0.208       | 1466.50      | -0.027     | Ufak iyileşme var ancak test başarısı hâlâ yetersiz.                     |
| **XGBoost**                 | 402.05        | 0.923       | 1863.40      | -0.746     | Eğitimde mükemmel ama testte çok kötü: ciddi overfitting sorunu var.     |
| **PCA + Random Forest**     | 208.34        | 0.979       | 495.90       | 0.883      | Boyut indirgeme sonrası yüksek başarı ve iyi genelleme.                  |
| **PCA + Hiperparametre Opt.**| 272.00       | 0.965       | 465.82       | 0.896      | PCA sonrası tuning ile en dengeli ve güvenilir model elde edildi.        |

---

### 📌 Açıklamalar

- 🔍 İlk kurulan **Random Forest** modeli, eğitim verisini fazla öğrenerek testte başarısız oldu (negatif R²).
- ⚙️ Hiperparametre optimizasyonu ile model biraz iyileşti fakat hâlâ yetersizdi.
- 🚨 **XGBoost**, aşırı güçlü olduğu için küçük veri setini ezberledi; test verisinde ciddi şekilde başarısız oldu.
- 🧪 Bu durum, **veri setinin sınırlı boyutu** ve **özelliklerin yetersizliği** nedeniyle modellerin genelleme yapmakta zorlandığını gösteriyor.

---

### 🧮 PCA (Principal Component Analysis) Etkisi

- PCA ile boyut indirgeme uygulandı ve yalnızca 2 bileşen kullanılarak regresyon modeli kuruldu.
- **Açıklanan Varyans Oranları**:
  - 🟦 PC1: %51.9
  - 🟨 PC2: %48.1

#### ✅ PCA sonrası model başarısı:


Train RMSE: 208.34 | Train R²: 0.979
Test RMSE: 495.90  | Test R²: 0.883


### 📈 PCA ile Performans Artışı

Bu sonuçlar, **PCA'nin** modeldeki karmaşıklığı azaltarak hem eğitim hem de test setinde yüksek performans sağladığını göstermektedir:

> ✅ **Model, artık daha az değişkenle daha fazla açıklayıcılık sunabiliyor.**

---

### 🛠️ Hiperparametre Optimizasyonu

🔍 `GridSearchCV` yöntemi ile **5-fold cross-validation** uygulanarak **216 farklı hiperparametre kombinasyonu** test edilmiştir.

#### 🔧 En iyi bulunan parametreler:


{
  'max_depth': 10,
  'max_features': 'auto',
  'min_samples_leaf': 1,
  'min_samples_split': 5,
  'n_estimators': 300
}

## 📊 Optimize Edilmiş Model Performansı

| Aşama     | RMSE    | R²      |
|-----------|---------|---------|
| **Train** | 272.00  | 0.965   |
| **Test**  | 465.82  | 0.896   |

🎯 **Model**, hem eğitim hem de test verisinde yüksek doğruluk sağlamıştır.

🧪 **Overfitting**, önemli ölçüde azalmış; modelin genelleme gücü artmıştır.

---

## ✅ Genel Yorum

📉 Başlangıçta kullanılan modeller istenen başarıyı sağlayamamış olsa da, analitik iterasyon süreci sonunda güçlü ve güvenilir bir model elde edilmiştir.

### 📌 Başarıda Etkili Olan Faktörler:

🔒 **PCA** ile olası *data leakage* risklerinin ortadan kaldırılması  
⚙️ Modelin yapısal olarak daha basit ama daha anlamlı hale getirilmesi  
🎛️ Hiperparametre optimizasyonu ile en uygun model konfigürasyonuna ulaşılması  

---

💡 Bu süreç, bir **veri bilimi projesinde** deneysel yaklaşımın, tekrarlı testlerin ve optimizasyonun ne kadar kritik olduğunu açıkça ortaya koymuştur.

🎓 **Sonuç:**  
📦 Az veri,  
🧠 Doğru analiz,  
🔧 Güçlü dönüşümler ile  
📈 Anlamlı ve yüksek başarılı modeller geliştirilebilir.









---

## 🔎 Kümeleme (Clustering) Analizi

| Cluster | Kullanıcı Sayısı | Ortalama Bet_Amount | Ortalama Login_Count | Bölge         | Hesap Durumu | Müşteri Tipi  |
|---------|------------------|---------------------|----------------------|---------------|--------------|---------------|
| 0       | 118              | 1831.57             | 100.59               | Australia     | Fraudulent   | Regular       |
| 1       | 124              | 4422.05             | 96.39                | South America | Fraudulent   | New           |
| 2       | 130              | 686.52              | 105.39               | South America | Fraudulent   | Regular       |
| 3       | 128              | 3133.95             | 107.07               | Asia          | Active       | New           |

- Silhouette Skoru: **0.549** (orta ve üzeri ayrışma, iyi ayrışmış kümeler)
- Fraudulent hesaplar Güney Amerika ve Avustralya’da yoğunlaşmış.
- Yüksek bahis yapanlar (Cluster 1) yüksek değerli müşteriler.
- Segmentlere göre giriş sayısı benzer, kullanım sıklığı segmentten bağımsız.

---

## 🌐 Network Analizi – Topluluk Tespiti

| Community | Kullanıcı Sayısı | Fraudulent Oranı |
|-----------|------------------|------------------|
| 0         | 180              | %34.44           |
| 1         | 176              | %35.80           |
| 2         | 144              | %31.94           |

- Fraudulent oranlar topluluklar arasında benzer (%31-36).
- Fraud yaygınlığı müşteri tipine göre ayrışmıyor.
- Risk yönetimi için detaylı segment bazlı önlemler önerilir.

---

## 🔬 PCA ve Model Yorumlama

### Açıklanan Varyans Oranları

| Bileşen | Varyans (%) |
|---------|-------------|
| PC1     | 51.9        |
| PC2     | 48.1        |

### PCA Model Performansı

| Set    | RMSE  | R²    |
|--------|-------|-------|
| Train  | 208.34| 0.979 |
| Test   | 495.90| 0.883 |

### SHAP ve LIME Analizi

| Özellik | Ortalama Mutlak SHAP Değeri |
|---------|-----------------------------|
| PC1     | 646.47                      |
| PC2     | 589.92                      |

- LIME örnek tahmini: 421.74
- PC2 > 0.64 durumunda tahmini +125.35 birim artırıyor.
- PC1 0.04 ile 0.70 arasında tahmini +19.85 birim artırıyor.

### PCA Bileşen Yükleri

| Değişken    | PC1     | PC2     |
|-------------|---------|---------|
| Login_Count | -0.707  | 0.707   |
| cluster     | -0.707  | -0.707  |

- **Login_Count** model için kritik öneme sahip.
- **Cluster** değişkeni hem PC1 hem PC2’de etkili, segmentasyon önemli.

---

## 🎯 İş Stratejisi ve Öneriler

- **Kullanıcı giriş sayısını artırmaya odaklanın:**  
  Login_Count’un modele etkisi yüksek, platforma giriş sıklığını artıracak kampanyalar, bildirimler ve UX iyileştirmeleri öncelikli olmalı.

- **Segmentlere özel pazarlama ve risk yönetimi:**  
  Cluster temelli müşteri segmentasyonuna dayalı özel kampanyalar ve fraud önlemleri geliştirin.

- **Fraudulent hesaplar için bölgesel önlemler:**  
  Güney Amerika ve Avustralya’da fraud yoğunluğu yüksek, bu bölgelerde özel kontrol ve denetim mekanizmaları uygulanmalı.

- **Model ve veri iyileştirmeleri:**  
  Daha fazla veri, yeni özellikler ve ileri mühendislik teknikleri ile model başarısı artırılabilir. PCA ve benzeri boyut indirgeme yöntemleri faydalı.

---

## 📝 Özet

Bu projede, bahis verileri üzerinde detaylı analiz, aykırı değer tespiti, segmentasyon, network analizi ve makine öğrenmesi modelleri ile derinlemesine inceleme yapılmıştır.  
Elde edilen bulgular, kullanıcı davranışlarının anlaşılması, fraud yönetimi ve iş stratejilerinin geliştirilmesi için temel oluşturmuştur.

---

## 📁 Dosya Yapısı

```plaintext
/
├── ModelSüreci              
├── HipotezTestleri           
├── KaynakData      
├── README.md           

## 🛠️ Teknolojiler & Kütüphaneler

- Python 3.x
- pandas, numpy
- scikit-learn
- xgboost
- matplotlib, seaborn
- shap, lime

---

## 📬 İletişim

Her türlü soru ve öneri için [berkanozcelik3.6@gmail.com](mailto:berkanozcelik3.6@gmail.com) adresinden ulaşabilirsiniz.


