## Tujuan Proyek dan Konsep Dasar

Proyek ini bertujuan untuk menguji dan memvisualisasikan hubungan kausalitas (Granger Causality) antara data makroekonomi dari FRED (Federal Reserve Economic Data) dan pergerakan harga berbagai aset finansial (Forex, Komoditas, Kripto) dari Yahoo Finance.

**Konsep Dasar:**

1.  **Granger Causality:** Ini adalah uji statistik untuk menentukan apakah satu deret waktu dapat "memprediksi" deret waktu lainnya. Jika deret A secara signifikan membantu memprediksi pergerakan deret B di masa depan (dengan mempertimbangkan nilai-nilai lampau dari B itu sendiri), maka dikatakan A "Granger-menyebabkan" B. Penting dicatat bahwa ini adalah **kausalitas statistik**, bukan kausalitas dalam pengertian sebab-akibat murni.

2.  **Data Makroekonomi (FRED):** Indikator seperti suku bunga (SOFR), indeks pasar saham (S&P 500), imbal hasil obligasi, dll., dapat mempengaruhi sentimen pasar dan pergerakan harga aset.

3.  **Data Harga (Yahoo Finance):** Harga berbagai aset finansial yang kita analisis.

4.  **VARX (Vector Autoregression with Exogenous variables) / ARX (Autoregressive with Exogenous variables):** Model ekonometrika deret waktu yang digunakan untuk memodelkan hubungan antar banyak variabel deret waktu (VAR) atau satu variabel (AR) dengan memasukkan variabel eksternal (Exogenous Variables - X). Dalam proyek ini:
    *   Variabel endogen (yang diprediksi) adalah pergerakan harga aset (log return).
    *   Variabel eksogen (yang mempengaruhi) adalah data makroekonomi yang ditransformasi dan distasionerkan.

**Alur Kerja Proyek:**

1.  **Pengumpulan Data:** Mengunduh data makro dari FRED dan data harga dari Yahoo Finance.
2.  **Pra-pemrosesan Data:** Membersihkan data, menangani nilai yang hilang, dan menyinkronkan data makro dengan data harga berdasarkan tanggal rilis.
3.  **Normalisasi & Transformasi:** Mengubah data harga menjadi log-return dan data makro menjadi stasioner (melalui differencing atau log transform) agar memenuhi asumsi model deret waktu.
4.  **Uji Stasioneritas:** Memastikan semua deret waktu yang akan digunakan dalam model VARX/ARX bersifat stasioner menggunakan uji ADF dan PP.
5.  **Uji Granger Causality:** Melakukan uji Granger antar pasangan data (Makro ↔ Harga) untuk mengidentifikasi hubungan yang signifikan secara statistik.
6.  **Model VARX/ARX:** Membangun model VARX (jika ada banyak eksogen yang signifikan) atau ARX (jika hanya satu endogen) untuk memprediksi pergerakan harga (log return) berdasarkan data historis harga dan data makro yang teridentifikasi signifikan.
7.  **Analisis Model:** Mengevaluasi kinerja model (menggunakan R², residual) dan menginterpretasikan hasilnya untuk memahami kondisi pasar.
8.  **Prediksi & Restorasi:** Menggunakan model terbaik untuk membuat prediksi log return di masa depan, lalu mengembalikannya ke skala harga asli untuk mendapatkan prediksi harga.

Dengan alur ini, proyek ini berusaha memberikan wawasan tentang bagaimana data makroekonomi mungkin berhubungan dan berpotensi mempengaruhi pergerakan harga aset finansial.

### **Menentukan Waktu Entry Berdasarkan Analisis Proyek**

Berdasarkan analisis yang telah dilakukan, waktu yang tepat untuk melakukan entry trading dapat disarankan dengan menggabungkan wawasan dari **Analisis Kondisi Pasar** (output sel `pYPiTE1tOirr`) dan logika keputusan trading dari fungsi `decide_trade` (dijelaskan di sel `79ae1d17`).

Berikut langkah-langkahnya:

1.  **Periksa Hasil Analisis Kondisi Pasar (Output sel `pYPiTE1tOirr`):**
    *   Lihat kolom **"Market Condition"** untuk pair yang Anda minati. Ini memberikan gambaran umum apakah pasar sedang **trending**, **ranging**, atau **chaotic**, serta tingkat **volatilitas** dan **persistence**-nya.
    *   Perhatikan kolom **"Model Recommendation"** dan **"RR Suggestion"**. Ini memberikan petunjuk jenis strategi yang mungkin cocok dan rasio Risk/Reward yang disarankan untuk kondisi pasar tersebut.

2.  **Dapatkan Prediksi Harga (Output sel `SE8INaajk3an` & `JdrZFPzy14bA`):**
    *   Lihat plot **"Restored Forecast: [Nama Pair]"** atau akses data dari dictionary `restored_forecasts`. Ini menunjukkan prediksi harga 2 langkah ke depan (sesuai `forecast_horizon`).
    *   Identifikasi **arah pergerakan harga yang diprediksi** (naik jika prediksi > harga terakhir, turun jika prediksi < harga terakhir). Ini menjadi dasar untuk menentukan arah trade (Long/Short).

3.  **Hitung Parameter Trade Menggunakan `decide_trade`:**
    *   Gunakan fungsi `decide_trade` (definisi di sel `12767631`, penjelasan di sel `79ae1d17`) dengan input:
        *   `last_price`: Harga penutupan terakhir dari data aktual (`base_dfs[pair_name].iloc[-1]['Close']`).
        *   `predicted_move`: Perubahan harga yang diprediksi (misalnya, `restored_forecasts[pair_name]['forecast_price'].iloc[-1] - last_price`).
        *   `atr`: Nilai Average True Range terbaru untuk pair tersebut pada timeframe yang relevan (perlu dihitung secara terpisah jika belum ada).
        *   `sigma2` atau `resid_std`: Varians atau standar deviasi residual dari model yang fit untuk pair tersebut (`fitted_models[pair_name]['fitted_model'].sigma2` atau `.resid.std()`).
        *   Parameter risiko Anda (`equity`, `risk_pct`, `k_atr_stop`, `k_model_stop`, `snr_threshold`).
    *   Fungsi ini akan mengembalikan dictionary `trade_decision`.

4.  **Analisis Keputusan Trade:**
    *   Periksa nilai `trade_decision['take']`. Jika `True`, ini adalah sinyal potensial untuk entry. Ini berarti prediksi pergerakan cukup kuat relatif terhadap ketidakpastian (SNR > `snr_threshold`) dan Stop Loss dapat ditetapkan dengan jarak yang memadai.
    *   Jika `trade_decision['take']` adalah `False`, lihat `trade_decision['reason']` untuk memahami mengapa trade tidak disarankan (misalnya, SNR rendah, Stop Loss tidak valid).

**Kesimpulan untuk Waktu Entry:**

Waktu yang tepat untuk entry berdasarkan proyek ini adalah ketika:

*   **Model memprediksi pergerakan harga yang jelas** (naik atau turun) untuk horizon prediksi.
*   Hasil dari fungsi `decide_trade` menunjukkan `take: True`, yang mengindikasikan bahwa **Signal-to-Noise Ratio (SNR) cukup tinggi** dan **Stop Loss yang valid dapat ditetapkan** berdasarkan volatilitas historis (ATR) dan ketidakpastian model.
*   Kondisi pasar yang dianalisis oleh model (trending, ranging, dll.) **sesuai dengan strategi trading** yang ingin Anda terapkan.

Dengan kata lain, entry disarankan saat analisis gabungan (prediksi arah + SNR + SL valid) menunjukkan potensi keuntungan yang memadai relatif terhadap risiko, didukung oleh pemahaman kondisi pasar saat itu. Selalu pertimbangkan hasil ini bersama dengan analisis teknikal dan fundamental Anda sendiri.

**Penjelasan Snippet Python:**

Fungsi `sigma_forecast_from_sigma2`: Mengestimasi standar deviasi forecast (`σ_forecast`) untuk horizon 1 hari berdasarkan `sigma2` (varians residual model) atau `resid_std` (standar deviasi residual model), dikalikan dengan akar kuadrat dari horizon (dalam kasus 1 hari, tetap). Ini mengukur ketidakpastian prediksi model.

Fungsi `decide_trade`: Ini adalah inti dari logika pengambilan keputusan.
1. Menghitung `fstd` (forecast standard deviation) menggunakan fungsi pertama.
2. Menghitung `SNR` (Signal-to-Noise Ratio) sebagai rasio antara pergerakan harga yang diprediksi (`predicted_move`) dan ketidakpastian prediksi (`fstd`). Jika SNR di bawah `snr_threshold` yang ditentukan, trade tidak diambil karena sinyal terlalu lemah dibanding noise/ketidakpastian.
3. Menentukan jarak Stop Loss (`sl_dist`). Ini adalah nilai maksimum antara stop berbasis ATR (menggunakan `k_atr_stop` sebagai pengali) dan stop berbasis ketidakpastian model (menggunakan `k_model_stop` sebagai pengali). Ini memastikan stop loss memperhitungkan baik volatilitas pasar historis maupun ketidakpastian prediksi model.
4. Menentukan arah trade (`long` atau `short`) berdasarkan tanda `predicted_move`.
5. Menghitung harga Stop Loss (`sl_price`) dan Take Profit (`tp_price`). Harga target TP ditetapkan secara konservatif (minimal 1.5x jarak SL atau sebesar `predicted_move`, mana yang lebih besar - ini bisa disesuaikan).
6. Menghitung ukuran posisi (`position_units`) berdasarkan persentase risiko (`risk_pct`) dari total ekuitas dibagi dengan jarak Stop Loss. Ini adalah manajemen risiko berbasis volatilitas/model uncertainty.
7. Mengembalikan dictionary yang berisi keputusan trade (`take`), parameter trade (arah, SL/TP, ukuran posisi), rasio RR, SNR, dan nilai `fstd`.

**Cara Menggunakan:**
- Anda perlu mendapatkan nilai `last_price` (harga penutupan terakhir), `atr` (ATR harian terbaru, perlu dihitung dari data OHLC), `predicted_move` (hasil forecast 1 langkah ke depan dari model Anda), dan `sigma2` atau `resid_std` (dari objek model yang sudah di-fit, misalnya `fitted_models[pair_name]['fitted_model'].sigma2` atau `fitted_models[pair_name]['fitted_model'].resid.std()`).
- Panggil fungsi `decide_trade` dengan nilai-nilai tersebut dan parameter risiko yang Anda inginkan (`equity`, `risk_pct`, `k_atr_stop`, `k_model_stop`, `snr_threshold`).
- Periksa nilai kunci `take` dari dictionary hasil. Jika `True`, berarti trade memenuhi kriteria dan Anda bisa menggunakan parameter lain (`direction`, `sl_price`, `tp_price`, `position_units`) untuk mengeksekusi trade dengan manajemen risiko yang sudah terpasang.
