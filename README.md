# 🌍 Tercih AI — Smart University & Life Index Search Engine

[English](#-english) | [Türkçe](#-turkce)

---

## 🇬🇧 English

# 🎓 Tercih AI — Smart University and Life Index Search Engine

Tercih AI is an innovative web application based on **Artificial Intelligence & Mathematical Optimization** that helps university candidates during their preference periods. It allows students to simultaneously evaluate academic criteria (admission scores, rankings, quotas, etc.) alongside city-based socio-economic life quality metrics (rent index, transportation fees, social activities).

🔗 **Live Demo:** [View Live Site (https://www.tercih-ai.com/)](#)

---

## 🚀 Key Features

- 🔍 **Smart & Dynamic Search:** Instant (debounced) search and filtering by department name, university, or city.
- 📊 **Detailed Academic Filters:** Flexible filtering based on maximum base score and maximum ranking constraints.
- ⚡ **Modern & Fluid UI:** A highly responsive "glassmorphic" dark-themed dashboard built with Tailwind CSS.
- 🎓 **Bachelor's & Associate's Distinction:** Easy toggle between 4-year (Lisans) and 2-year (Önlisans) programs, with sub-groupings by score types (SAY, EA, SÖZ, DİL).
- 🧠 **Life Index Scores:** A mathematically optimized percentage index representing the life quality and economic viability of cities for university students.
- ℹ️ **AI Guide (Tooltip):** Context-aware info boxes populated from the dataset summarizing academic and social life standards for each city.

---

## 🛠️ Tech Stack

- **Frontend:** HTML5, Vanilla JavaScript (ES6+)
- **Styling (CSS):** Tailwind CSS, Google Fonts (Plus Jakarta Sans)
- **Data Storage:** Local JSON databases (`lisans4.json`, `onlisans4.json`, `sehirler2.json`)
- **Optimization/ML:** Python - `scipy.optimize.minimize` (SLSQP algorithm) with fallback Random Search approximation.

---

## 🧠 Mathematical Model and Hybrid-ML Optimization

The core value of Tercih AI lies in its **Life Index Model**, which objectively evaluates the suitability of Turkey's 81 cities for students. The training and optimization steps are detailed below:

### 1. Data Normalization
To make metrics with different scales (e.g., average rent in Turkish Liras, count of libraries, number of cultural events) comparable, **Min-Max Normalization** is applied to each feature:

$$x_{\text{norm}} = \frac{x - x_{\text{min}}}{x_{\text{max}} - x_{\text{min}}}$$

This scales all feature values to the $[0, 1]$ interval.

### 2. Feature Polarity & Orientation
Features are categorized based on their impact on a student's quality of life:
- **Negative Features (Costs - Rent, Transport):** Higher costs must decrease the life index. The normalized score is inverted:
  $$\text{score} = 1 - x_{\text{norm}}$$
  To prevent floating-point inaccuracies from generating negative scores, values are clipped at zero:
  $$\text{score} = \max(0, \text{score})$$
- **Positive Features (Opportunities - Libraries, social events, coastal status, etc.):** The normalized value is directly used as the score.

### 3. Feature Set
- Rent Cost Score ($x_{\text{kira}}$)
- Transport Fee Score ($x_{\text{ulasim}}$)
- Library Count Score ($x_{\text{kutuphane}}$)
- Cultural Event Score ($x_{\text{sinema}}$)
- Social Life & Cafe Score ($x_{\text{sosyal}}$)
- Shopping Mall Score ($x_{\text{avm}}$)
- Coastal/Geographic Score ($x_{\text{deniz}}$)

### 4. Original Heuristic Model (Base Weights)
Initially, a set of heuristic weights was defined based on student surveys and expert opinions:

| Feature | Base Weight ($w_i$) |
| :--- | :--- |
| Rent Cost | 0.39 |
| Transport Fee | 0.14 |
| Libraries | 0.12 |
| Cultural Events | 0.12 |
| Shopping Malls | 0.12 |
| Social Life | 0.12 |
| Coastal Status | 0.10 |

The base score was calculated as:
$$\text{yasam\_orijinal} = \sum_{i} w_i \cdot x_i$$

### 5. Hybrid-ML Optimization
The goal is to optimize the weight vector ($w_i$) using mathematical programming to **maximize the number of student-friendly cities**. A city is considered student-friendly if its index score is above the threshold of $0.5$ (or 50%).

#### A. Sigmoid Smoothing
Since the discrete counting function is non-differentiable (preventing gradient-based optimization), we approximate it using a **Sigmoid** function:

$$\text{softcount}(w) = \sum_{i} \sigma\Big(k \cdot \big(s_i(w) - \text{threshold}\big)\Big)$$

Where:
- $\sigma(z) = \frac{1}{1 + e^{-z}}$ (Sigmoid activation function)
- $\text{threshold} = 0.5$ (Target index score threshold)
- $k = 20$ (Steepness scaling parameter controlling count approximation sharpness)
- $s_i(w) = \mathbf{x}_i \cdot \mathbf{w}$ (Calculated index score for city $i$)

#### B. Objective Function
The optimization is modeled as a minimization of the negative softcount:

$$\min_{w} \text{Obj}(w) = -\text{softcount}(w) + \text{penalty}(w)$$

Subject to the following constraints:
- $w_i \ge 0$ (Non-negativity)
- $\sum_{i} w_i = 1$ (Normalization for interpretability)

#### C. Lower Bound Constraint (Min 35% Rule)
To ensure that even the lowest-scoring city does not fall below a baseline quality of 35% ($0.35$), a quadratic penalty is added to the objective function:

$$\text{penalty}(w) = \lambda \cdot \Big(0.35 - \min_{i} s_i(w)\Big)^2 \quad \text{if } \min_{i} s_i(w) < 0.35$$

where $\lambda = 1000$ is the penalty weight parameter.

### 6. Solver & Post-Scaling
The optimization is executed using Python's `scipy.optimize.minimize` with the **SLSQP (Sequential Least Squares Programming)** solver. 

After optimization, a final post-processing linear transformation ($s' = a \cdot s + b$) is applied to strictly enforce the minimum 35% limit, preventing out-of-bound or uncontrolled indices. The final optimized percentage scores are saved under `yasam_ml_pct` in `sehirler2.json`.

---

## 📂 Project Structure

```bash
tercih-ai/
├── index.html          # Web application UI and search engine logic
├── sehirler2.json      # Optimized city dataset, ML index percentages, and AI analyses
├── lisans4.json        # 4-Year Bachelor's program admission scores and rankings
└── onlisans4.json      # 2-Year Associate's program admission scores and rankings
```

### Sample City Entry (`sehirler2.json`):
```json
"ADANA": {
    "rent_score": 0.6857682619647356,
    "transportation_score": 0.3695652173913043,
    "social_score": 1.0,
    "life_ml_pct": 81.66,
    "analysis": "It is a balanced city with a vibrant social life and reasonable housing costs; cafes and entertainment options are plentiful enough to satisfy students."
}
```

---

## 💻 Setup and Local Usage

Since the project runs entirely on the client side, installation is straightforward:

1. Clone the repository:
   ```bash
   git clone https://github.com/username/tercih-ai.git
   ```
2. Navigate to the project directory:
   ```bash
   cd tercih-ai
   ```
3. Open `index.html` in your web browser, or serve it locally using a server extension (like VS Code's Live Server).

---

## 📜 License

This project is licensed under the **MIT License**. See the `LICENSE` file for details.

---

<br>

## 🇹🇷 Türkçe

# 🎓 Tercih AI — Akıllı Üniversite ve Yaşam Endeksi Arama Motoru

Tercih AI, üniversite adaylarının tercih dönemlerinde hem akademik kriterleri (puan, başarı sırası, kontenjan vb.) hem de şehir bazlı sosyo-ekonomik yaşam kalitesini (kira, ulaşım, sosyal imkanlar) birlikte değerlendirebilmelerini sağlayan, **Yapay Zeka / Matematiksel Optimizasyon** tabanlı yenilikçi bir web uygulamasıdır.

🔗 **Canlı Demo:** [Projeyi Canlıda Görüntüle (https://www.tercih-ai.com/)](#)

---

## 🚀 Öne Çıkan Özellikler

- 🔍 **Akıllı ve Dinamik Arama:** Bölüm adı, üniversite ismi veya şehir adına göre anlık (debounced) filtreleme.
- 📊 **Detaylı Akademik Filtreler:** Taban puanı üst sınırı ve başarı sıralaması üst sınırına göre esnek listeleme.
- ⚡ **Hızlı ve Akıcı Arayüz:** Tailwind CSS ile hazırlanmış, modern, "glassmorphic" (cam efektli) karanlık mod tasarımı.
- 🎓 **Lisans ve Önlisans Ayrımı:** Tek tıkla 4 yıllık (Lisans) ve 2 yıllık (Önlisans) programlar arasında geçiş ve puan türüne (SAY, EA, SÖZ, DİL) göre alt gruplama.
- 🧠 **Yaşam Endeksi (Life Index) Skorları:** Şehirlerin öğrencilere sunduğu yaşam kalitesi ve ekonomik durumun matematiksel olarak optimize edilmiş yüzdesel gösterimi.
- ℹ️ **Yapay Zeka Rehberi (AI Tooltip):** Şehirlerin akademik ve sosyal yaşam standartlarını özetleyen, veri setinden beslenen akıllı analiz pencereleri.

---

## 🛠️ Teknoloji Yığını

- **Frontend:** HTML5, Vanilla JavaScript (ES6+)
- **Styling (CSS):** Tailwind CSS, Google Fonts (Plus Jakarta Sans)
- **Veri Depolama:** JSON tabanlı yerel ilişkisel veri dosyaları (`lisans4.json`, `onlisans4.json`, `sehirler2.json`)
- **Optimizasyon/ML:** Python - `scipy.optimize.minimize` (SLSQP algoritması) ve fallback olarak Random Search (Rastgele Arama) optimizasyonu

---

## 🧠 Matematiksel Model ve Hibrit-ML Optimizasyonu

Projenin en güçlü yönü, Türkiye'deki 81 ilin öğrencilere uygunluğunu objektif verilere dayanarak ölçen **Yaşam Endeksi Modeli**dir. Modelin kurulma ve eğitilme aşamaları aşağıda detaylandırılmıştır:

### 1. Veri Normalizasyonu (Normalization)
Farklı ölçeklerdeki parametreleri (örneğin Türk Lirası cinsinden ortalama kiralar, kütüphane sayıları, sinema/tiyatro etkinlik sayıları) birbiriyle kıyaslanabilir kılmak amacıyla her bir özellik (feature) için **Min-Max Normalizasyonu** uygulanmıştır:

$$x_{\text{norm}} = \frac{x - x_{\text{min}}}{x_{\text{max}} - x_{\text{min}}}$$

Bu dönüşüm sonucunda tüm özellik değerleri $[0, 1]$ aralığına indirgenmiştir.

### 2. Özelliklerin Yönlendirilmesi (Feature Polarity)
Öğrenci yaşamı için negatif ve pozitif etkiye sahip özellikler farklı şekilde işlenmiştir:
- **Negatif Özellikler (Maliyetler - Kira, Ulaşım):** Miktar arttıkça yaşam kalitesi puanı düşmelidir. Bu nedenle normalize edilen değer tersine çevrilmiştir:
  $$\text{skor} = 1 - x_{\text{norm}}$$
  Eğer tersleme sonrası kayan noktalı sayı hassasiyetinden dolayı negatif değer oluşursa, sıfırın altına inilmesini engellemek için kırpma (clipping) uygulanmıştır:
  $$\text{skor} = \max(0, \text{skor})$$
- **Pozitif Özellikler (İmkanlar - Kütüphane sayısı, sosyal etkinlikler, denize kıyısı olma vb.):** Doğrudan normalize edilen değer kullanılmıştır.

### 3. Kullanılan Özellikler (Features)
- Barınma Maliyeti Skoru ($x_{\text{kira}}$)
- Ulaşım Ücreti Skoru ($x_{\text{ulasim}}$)
- Kütüphane Sayısı Skoru ($x_{\text{kutuphane}}$)
- Kültürel Etkinlik Skoru ($x_{\text{sinema}}$)
- Sosyal Hayat & Kafe Skoru ($x_{\text{sosyal}}$)
- Alışveriş Merkezi Skoru ($x_{\text{avm}}$)
- Coğrafi Konum Skoru ($x_{\text{deniz}}$)

### 4. Orijinal Hevristik Model (Base Weights)
Optimizasyon öncesi, uzman görüşlerine ve öğrenci anketlerine dayanarak belirlenen ilk ağırlıklar şu şekildedir:

| Özellik | Başlangıç Ağırlığı ($w_i$) |
| :--- | :--- |
| Kira Maliyeti | 0.39 |
| Ulaşım Ücreti | 0.14 |
| Kütüphane | 0.12 |
| Sinema/Tiyatro | 0.12 |
| AVM | 0.12 |
| Sosyal Hayat | 0.12 |
| Deniz Kıyısı | 0.10 |

Bu temel formül ile ham puanlar şu şekilde hesaplanmıştır:
$$\text{yasam\_orijinal} = \sum_{i} w_i \cdot x_i$$

### 5. Hibrit-ML Optimizasyonu
Amacımız, ağırlık ($w_i$) kombinasyonunu yapay zeka/matematiksel optimizasyon kullanarak güncelleyip, **öğrenci dostu şehirlerin sayısını maksimuma çıkarmaktır**. Hedef, yaşam puanı eşik değer olan $0.5$ (yani %50) üzerinde kalan şehirlerin sayısını en üst düzeye ulaştırmaktır.

#### A. Sigmoid Yumuşatma (Sigmoid Smoothing)
Şehir sayısını sayma fonksiyonu (discrete count) türevlenebilir olmadığından, gradyan tabanlı optimizasyon yöntemlerini kullanabilmek için **Sigmoid** fonksiyonu ile yumuşatma yapılmıştır:

$$\text{softcount}(w) = \sum_{i} \sigma\Big(k \cdot \big(s_i(w) - \text{threshold}\big)\Big)$$

Burada:
- $\sigma(z) = \frac{1}{1 + e^{-z}}$ (Sigmoid fonksiyonu)
- $\text{threshold} = 0.5$ (Hedef yaşam endeksi eşiği)
- $k = 20$ (Skorların netliğini ve sigmoid eğrisinin dikliğini belirleyen hassasiyet katsayısı)
- $s_i(w) = \mathbf{x}_i \cdot \mathbf{w}$ (Şehir $i$ için hesaplanan toplam endeks puanı)

#### B. Amaç Fonksiyonu (Objective Function)
Optimizasyon problemi, negatif softcount değerini minimize etme şeklinde tanımlanmıştır:

$$\min_{w} \text{Obj}(w) = -\text{softcount}(w) + \text{penalty}(w)$$

Ağırlıklar için konulan kısıtlar (constraints):
- $w_i \ge 0$ (Ağırlıklar negatif olamaz)
- $\sum_{i} w_i = 1$ (Toplam ağırlık 1 olmalıdır - Anlamlı yorumlanabilirlik için)

#### C. Alt Sınır Cezalandırması (Min %35 Kuralı)
En kötü durumdaki şehrin bile yaşanabilirlik puanının en az %35 ($0.35$) olması hedeflenmiştir. Bu alt sınırı ihlal eden ağırlık kombinasyonlarını engellemek amacıyla amaç fonksiyonuna yüksek katsayılı ($\lambda = 1000$) bir ceza (penalty) terimi eklenmiştir:

$$\text{penalty}(w) = \lambda \cdot \Big(0.35 - \min_{i} s_i(w)\Big)^2 \quad \text{eğer } \min_{i} s_i(w) < 0.35 \text{ ise}$$

### 6. Çözücü (Solver) ve Ölçeklendirme (Scaling)
Optimizasyon işlemleri Python'da `scipy.optimize.minimize(method='SLSQP')` algoritması kullanılarak koşturulmuştur. 

Optimizasyon tamamlandıktan sonra, kısıtların kesin olarak sağlanması adına tüm şehir skorları lineer bir dönüşümle ($s' = a \cdot s + b$) yeniden ölçeklendirilmiş, böylece minimum skor %35'e çekilirken modelin kontrol dışı taşmalar yapması engellenmiştir. Elde edilen optimize edilmiş nihai yüzdeler veri tabanına (`sehirler2.json`) `yasam_ml_pct` olarak kaydedilmiştir.

---

## 📂 Dosya Yapısı ve Veri Formatı

```bash
tercih-ai/
├── index.html          # Uygulamanın modern arayüzü ve filtreleme mantığı
├── sehirler2.json      # Optimize edilmiş şehir verileri, ML yaşam yüzdeleri ve AI analizleri
├── lisans4.json        # 4 Yıllık Lisans programlarına ait taban puanlar ve sıralama verileri
└── onlisans4.json      # 2 Yıllık Önlisans programlarına ait taban puanlar ve sıralama verileri
```

### Örnek Şehir Verisi (`sehirler2.json`):
```json
"ADANA": {
    "kira_skoru": 0.6857682619647356,
    "ulasim_skoru": 0.3695652173913043,
    "sosyal_skoru": 1.0,
    "yasam_ml_pct": 81.66,
    "analiz": "Canlı sosyal hayatı ve makul barınma maliyetleriyle dengeli bir şehirdir; kafeler ve eğlence imkanları öğrencileri tatmin edecek kadar fazladır."
}
```

---

## 💻 Kurulum ve Yerel Çalıştırma

Bu proje tamamen istemci taraflı (client-side) çalıştığından dolayı kurulumu son derece basittir:

1. Depoyu bilgisayarınıza klonlayın:
   ```bash
   git clone https://github.com/kullanici_adi/tercih-ai.git
   ```
2. Proje dizinine gidin:
   ```bash
   cd tercih-ai
   ```
3. `index.html` dosyasını tarayıcınızda çift tıklayarak açabilir veya yerel bir sunucu (örneğin VS Code Live Server eklentisi) kullanarak çalıştırabilirsiniz.

---

## 📜 Lisans

Bu proje **MIT Lisansı** altında lisanslanmıştır. Daha fazla bilgi için `LICENSE` dosyasına göz atabilirsiniz.
