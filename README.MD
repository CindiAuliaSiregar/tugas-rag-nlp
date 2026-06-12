# 📘 RAG Parameter Exploration Assignment

Pada tugas ini kamu akan bereksperimen dengan **6 parameter utama Retrieval-Augmented Generation (RAG)** menggunakan **RAG Playground**.

### 🎯 Tujuan Pembelajaran

Kamu akan:

✅ Memahami **apa fungsi** masing-masing parameter RAG

✅ Mengamati **pengaruh** tuning terhadap kualitas jawaban vs kecepatan

✅ Belajar **trade-off** antara retrieval & generation

✅ Menulis **insight praktis** dari eksperimen yang kamu lakukan

> **Tools sudah disediakan.** Kamu **tidak perlu ngoding**, hanya ubah nilai parameter di file `app.py` & restart server.

### ⚠️ Notes Penting

- **Ikuti video di LMS jika kamu kebingungan.**
- **Sediakan penyimpanan at least 2GB**, akan ada download model yang cukup besar ke laptopmu.
- **Hapus proyek ini kalau sudah selesai**, biar laptop-mu tidak tercekik.
- **Ketika proyek ini sudah diisi & dikerjakan di bawah, upload ke GitHub.**

---

## ⚙️ Parameter yang Diuji

Semua parameter berikut dapat ditemukan di file `app.py`:

```python
# Buka app.py, cari bagian KONFIGURASI INTI
CHUNK_SIZE = 300
TOP_K_INITIAL = 20
FINAL_TOP_M = 5
USE_RERANKER = True
TEMPERATURE = 0.2
GROQ_MODEL = "llama-3.1-8b-instant"
```

---

### 📏 1. CHUNK_SIZE

```python
CHUNK_SIZE = 300  # ubah angka ini
```

**Apa itu CHUNK_SIZE?**

Saat dokumen PDF diupload, teksnya dipotong-potong menjadi bagian kecil (chunk) agar bisa diproses dan dicari. `CHUNK_SIZE` menentukan berapa banyak **kata** per potongan.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari CHUNK_SIZE)*

> Jawab: Mengontrol seberapa besar ukuran potongan teks (jumlah kata) yang akan dibaca dan disimpan oleh sistem dalam satu waktu. Parameter ini berfungsi untuk menentukan batas informasi agar AI bisa memahami konteks kalimat tanpa kehilangan inti bahasannya.

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (100–200)** | *Pencarian sangat spesifik dan akurat pada kata kunci tertentu, tetapi berisiko tinggi kehilangan konteks karena kalimat atau paragraf penting bisa terpotong di tengah jalan. Jawaban AI bisa terasa kurang lengkap.* |
| **Sedang (250–400)** ⭐ Recommended | *Memberikan keseimbangan yang paling pas. Cukup besar untuk menjaga satu paragraf utuh beserta konteks ceritanya, tapi cukup kecil agar AI tidak kebingungan memilah informasi mana yang paling relevan dengan pertanyaan.* |
| **Besar (500–1000)** | *Konteks bacaan sangat luas dan lengkap. Namun, pencarian menjadi kurang presisi (kurang tajam) karena dalam satu potongan terdapat terlalu banyak topik yang berbeda. Selain itu, bisa membuat AI lambat merespons.* |

**Pertanyaan Refleksi:**

* Chunk besar atau kecil yang lebih tepat untuk dokumen yang kamu tes? Mengapa?
  > Jawab: Untuk dokumen yang saya tes, chunk Sedang (250-400) lebih tepat. Karena dokumen saya berisi banyak penjelasan paragraf yang saling terhubung. Jika terlalu kecil, penjelasan AI jadi terputus, sedangkan jika terlalu besar, AI malah memberikan informasi ekstra yang tidak saya tanyakan.

* Apa yang terjadi kalau jawaban terasa "terpotong" atau kurang konteks? Parameter mana yang harus diubah?
  > Jawab: Jika jawaban terasa terpotong atau kehilangan makna aslinya, itu berarti potongan teksnya terlalu sempit untuk menangkap satu gagasan utuh. Parameter yang harus diubah (dinaikkan nilainya) adalah CHUNK_SIZE agar AI bisa membaca konteks yang lebih panjang.

---

### 🔍 2. TOP_K_INITIAL

```python
TOP_K_INITIAL = 20
```

**Apa itu TOP_K_INITIAL?**

Setelah pertanyaan diubah jadi vektor, sistem mencari chunk yang paling mirip. `TOP_K_INITIAL` menentukan berapa banyak **kandidat awal** yang diambil dari hasil pencarian vektor sebelum di-filter/rerank.

**Fungsi:** *Menentukan jumlah maksimal potongan teks (kandidat awal) yang ditarik secara kasar dari database sebelum diseleksi dan diurutkan ulang (rerank) untuk mendapatkan konteks yang paling akurat.*

> Jawab: ...

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Kecil (5–10)** | *Kecepatan pencarian (latency) sangat tinggi karena data yang diproses sedikit, namun nilai recall (potensi menemukan jawaban yang benar) sangat rendah. Ada risiko tinggi konteks yang relevan tidak ikut terambil.* |
| **Sedang (15–25)** ⭐ Recommended | *Memberikan keseimbangan yang optimal antara rasio recall dan waktu komputasi. Jumlah kandidat cukup banyak agar reranker punya pilihan akurat, namun tidak terlalu membebani sistem saat proses penyeleksian.* |
| **Besar (30–50)** | *Nilai recall maksimal karena hampir semua kemungkinan konteks terbawa, tetapi waktu respons (latency) menjadi lambat karena sistem harus mengevaluasi ulang terlalu banyak potongan teks.* |

> 💡 **Catatan:** Nilai ini hanya berpengaruh signifikan jika `USE_RERANKER = True`. Tanpa reranker, yang dipakai langsung hanya `FINAL_TOP_M` teratas.

**Pertanyaan Refleksi:**

* Apakah top-k kecil (5–10) cukup untuk dataset kamu? Jelaskan alasannya.
  > Jawab: Tidak cukup. Karena jika dokumen PDF yang dites memiliki banyak topik yang mirip atau kata kunci yang berulang di berbagai halaman, mengambil hanya 5-10 kandidat awal berisiko membuang potongan teks yang ternyata memuat jawaban paling komprehensif. Akibatnya, jawaban akhir AI bisa melenceng atau kurang lengkap.

* Apa hubungan antara `TOP_K_INITIAL` dan `USE_RERANKER`?
  > Jawab: TOP_K_INITIAL bertugas sebagai "pengumpul bahan mentah", sedangkan USE_RERANKER adalah "juri penyeleksinya". Jika reranker aktif, ia membutuhkan bahan mentah (TOP_K_INITIAL) yang cukup besar agar punya banyak pilihan untuk disortir demi mencari yang terbaik. Jika reranker dimatikan, mengambil kandidat awal dalam jumlah besar menjadi kurang efektif karena sistem hanya akan mengambil urutan teratas secara kasar.

---

### 📋 3. FINAL_TOP_M

```python
FINAL_TOP_M = 5
```

**Apa itu FINAL_TOP_M?**

Dari sekian banyak kandidat yang ditemukan, hanya **M teratas** (setelah reranking) yang dikirim ke LLM sebagai konteks untuk menjawab pertanyaan.

**Fungsi:** *Menentukan batasan jumlah potongan teks (konteks) paling relevan yang akan benar-benar dikirimkan dan dibaca oleh otak AI (LLM) sebagai landasan dasar untuk menjawab pertanyaan user.*

> Jawab: ...

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **Sedikit (2–3)** | *Jawaban menjadi sangat terarah dan waktu respons AI sangat cepat karena teks yang dibaca sedikit. Namun, risiko gagal menjawab sangat tinggi jika informasi yang dicari ternyata terpotong atau butuh penjelasan dari paragraf lain.* |
| **Sedang (4–6)** ⭐ Recommended | *Memberikan porsi informasi yang paling optimal. Cukup lengkap bagi AI untuk merangkai jawaban yang utuh dan akurat, tanpa membuat sistem menjadi lemot atau membebani memori (context window).* |
| **Banyak (7–10)** | *Konteks bacaan sangat melimpah, namun waktu komputasi (latency) menjadi sangat lambat. AI juga rentan mengalami kebingungan fokus (Lost in the Middle) di mana ia justru mengabaikan poin penting karena terlalu banyak teks yang dijejalkan.* |

> 💡 **Catatan:** Semakin banyak konteks = semakin banyak token yang dikirim ke LLM = latency lebih tinggi. Jaga agar tetap efisien.

**Pertanyaan Refleksi:**

* Benarkah "semakin banyak konteks semakin bagus"? Jelaskan berdasarkan eksperimenmu.
  > Jawab: Tidak benar. Berdasarkan eksperimen, menyuapi AI dengan terlalu banyak konteks justru membuatnya lambat karena jumlah token yang diproses membengkak. Selain itu, informasi yang terlalu banyak seringkali memasukkan noise (kalimat tidak penting), yang membuat AI kebingungan membedakan mana poin utama dan mana sekadar info pelengkap, sehingga jawaban akhirnya malah bertele-tele atau tidak fokus pada pertanyaan.

* Apa yang terjadi kalau konteks yang dikirim ke LLM saling bertentangan?
  > Jawab: AI akan memberikan jawaban yang ambigu (ragu-ragu) atau rentan mengalami halusinasi. Karena AI bertugas merangkum konteks yang disuapkan kepadanya, ia akan mencoba menggabungkan kedua fakta yang bertentangan tersebut, atau memilih salah satunya secara acak tanpa bisa memverifikasi mana yang secara faktual benar.

---

### 🎯 4. USE_RERANKER

```python
USE_RERANKER = True  # atau False
```

**Apa itu Reranker?**

Reranker adalah model tambahan (Cross-Encoder) yang mengevaluasi ulang pasangan `[pertanyaan, chunk]` untuk mengurutkan relevansi lebih akurat daripada hanya cosine similarity biasa.

**Fungsi:** *(Jelaskan dengan kata-katamu sendiri apa fungsi dari USE_RERANKER)*

> Jawab: Bertindak sebagai sistem evaluasi tahap dua (juri penilai) yang membaca ulang dan mengurutkan kembali kandidat potongan teks. Fungsinya untuk memastikan potongan teks tersebut benar-benar memiliki makna yang relevan dengan pertanyaan, bukan sekadar memiliki kata-kata yang mirip.

**Pengaruhnya:**

| Mode | Efek |
| --- | --- |
| `True` | *Presisi dan relevansi konteks meningkat drastis karena AI mampu memahami nuansa makna di balik pertanyaan yang kompleks. Namun, waktu respons (latency) aplikasi akan bertambah (terasa sedikit lebih lambat) karena adanya beban komputasi tambahan.* |
| `False` | *Kecepatan respons (latency) menjadi sangat maksimal. Namun, aplikasi hanya mengandalkan cosine similarity (kemiripan kata dasar), sehingga sangat rentan tertipu oleh teks yang menggunakan kata kunci serupa tetapi konteks bahasannya melenceng sama sekali.* |

> 💡 **Catatan:** Untuk dokumen teknis atau pertanyaan yang ambigu, reranker sangat membantu. Untuk kecepatan maksimal, matikan.

**Pertanyaan Refleksi:**

* Pada query jenis apa reranker paling terasa manfaatnya? Berikan contoh.
  > Jawab: : Reranker paling terasa manfaatnya pada pertanyaan yang kompleks, ambigu, atau sangat spesifik secara teknis. Contohnya pertanyaan menjebak seperti: "Apa bedanya bank tempat menaruh uang dengan bank darah?". Pencarian biasa akan kebingungan membedakan kata "bank", tetapi Reranker mampu memahami perbedaan konteks maknanya dan mencari potongan teks yang tepat.

* Berapa kira-kira tambahan latency saat reranker dinyalakan di eksperimenmu?
  > Jawab: Terdapat tambahan latency, retrieval 271.2ms • generation 2396.5ms adalah saa raranker dimatikan, retrieval 16916.3ms • generation 2392.9ms adalah saat raranker dinyalakan, terdapat sekitar 16,6 detik tambahan latecy saat memproses jawaban dibandingkan saat Reranker dimatikan (False). Tambahan waktu ini sepadan karena kualitas jawaban yang diberikan jauh lebih akurat dan tidak melenceng.

---

### 🌡️ 5. TEMPERATURE

```python
TEMPERATURE = 0.2  # range: 0.0 - 1.0
```

**Apa itu Temperature?**

Temperature mengatur distribusi probabilitas saat LLM memilih kata berikutnya. Semakin rendah, semakin "pasti" dan konsisten jawabannya.

**Fungsi:** *Mengontrol tingkat kreativitas atau randomnya AI saat merangkai kata. Nilai yang lebih rendah membuat AI lebih kaku, faktual, dan patuh pada dokumen, sedangkan nilai tinggi bikin AI lebih imajinatif tapi rentan mengarang bebas.*

> Jawab: ...

**Pengaruhnya:**

| Range Nilai | Efek |
| --- | --- |
| **0.0–0.2** ⭐ Recommended untuk RAG | *Jawaban sangat konsisten, logis, dan deterministik. AI hampir selalu memberikan kalimat yang persis sama jika ditanya hal yang sama berulang kali. Sangat faktual dan patuh pada teks aslinya (tingkat halusinasi sangat rendah).* |
| **0.3–0.5** | *Memberikan sedikit keseimbangan. Bahasanya mulai sedikit lebih luwes (ga kaku banget), namun AI masih cukup disiplin berpegang pada fakta yang ada di dokumen referensi.* |
| **0.6–0.8** | *Jawaban menjadi lebih kreatif, variatif, dan ekspresif secara bahasa. Sangat cocok untuk tugas brainstorming atau menulis cerita, namun mulai berisiko menyelipkan informasi tambahan yang tidak tertulis di dokumen.* |
| **0.9–1.0** | *Sangat berbahaya untuk sistem RAG. Gaya bahasanya acak dan berubah-ubah. AI menjadi terlalu berimajinasi sehingga risiko mengalami halusinasi (membuat fakta karangan yang seolah-olah benar) menjadi sangat tinggi.* |

> ⚠️ **Catatan:** Untuk RAG (jawaban berdasarkan dokumen), gunakan temperature rendah (0.1–0.3) agar jawaban tetap *grounded* pada konteks yang diberikan.

**Pertanyaan Refleksi:**

* Untuk QA berbasis dokumen perusahaan, temperatur ideal berapa? Jelaskan alasannya.
  > Jawab: Temperatur ideal adalah 0.0 hingga 0.1. Alasannya, dokumen perusahaan (seperti laporan keuangan, SOP, atau kebijakan privasi) membutuhkan tingkat akurasi mutlak dan nol persen toleransi terhadap karangan bebas. Jawaban AI harus 100% faktual, kaku, dan presisi sesuai data asli demi menghindari misinformasi yang fatal.

* Apa yang terjadi pada jawaban saat kamu naikkan temperature ke 0.9?
  > Jawab: Saat diset ke 0.9, jawaban AI menjadi kurang dapat diprediksi. Gaya bahasanya berubah setiap kali ditanya ulang, dan ia mulai merangkai fakta-fakta baru atau opini yang sebenarnya tidak pernah disebutkan di dalam file PDF aslinya (mengalami halusinasi).

---

### 🤖 6. GROQ_MODEL

```python
GROQ_MODEL = "llama-3.1-8b-instant"  # model default
```

**Model lain yang bisa dicoba:**

```python
# Ganti value GROQ_MODEL ke salah satu di bawah:
GROQ_MODEL = "openai/gpt-oss-120b"
GROQ_MODEL = "llama-3.3-70b-versatile"
```

**Apa bedanya tiap model?**

Setiap model punya ukuran parameter dan kemampuan yang berbeda. Isi tabel di bawah berdasarkan eksperimenmu:

| Model | Ukuran | Pengaruh terhadap Kecepatan & Kualitas |
| --- | --- | --- |
| `llama-3.1-8b-instant` (Default) | 8B parameter | *Waktu respons (latency) sangat cepat. Kualitas jawaban cukup baik untuk pertanyaan sederhana dan merangkum isi dokumen. Sangat cocok dipakai untuk development (uji coba awal) karena tidak perlu menunggu lama.* |
| `openai/gpt-oss-120b` | 120B parameter | *Waktu respons (latency) paling lambat karena beban komputasinya sangat berat. Kualitas jawaban sangat sempurna, logis, dan teliti. Cocok dipakai untuk menganalisis dokumen yang sangat kompleks, seperti jurnal medis atau dokumen hukum.* |
| `llama-3.3-70b-versatile` | 70B parameter | *Waktu respons menengah (sedikit lebih lambat dari 8B). Kualitas penalarannya (reasoning) sangat tinggi dan akurat. Cocok untuk versi production (siap pakai) yang membutuhkan jawaban presisi namun kecepatan aplikasinya masih bisa ditoleransi oleh user.* |

> 💡 **Tips:** Mulai dengan `llama-3.1-8b-instant` untuk development. Saat butuh akurasi tinggi, switch ke `llama-3.3-70b-versatile` atau `openai/gpt-oss-120b`.

**Pertanyaan Refleksi:**

* Apakah peningkatan kualitas jawaban dari model besar sebanding dengan penambahan waktu? Jelaskan.
  > Jawab: Tergantung kasus penggunaannya. Jika pertanyaannya hanya sebatas "Siapa nama penulis dokumen ini?", menggunakan model besar sangat membuang waktu dan komputasi, sehingga tidak sebanding. Namun, jika pertanyaannya rumit seperti "Analisis perbedaan metode A dan metode B berdasarkan dokumen", waktu tunggu yang lama sangat sebanding dengan hasil analisisnya yang mendalam dan akurat.

* Model mana yang paling cocok untuk use-case dokumenmu? Mengapa?
  > Jawab: (Sesuaikan dengan eksperimen Anda, ini adalah contoh yang aman) Model llama-3.1-8b-instant sudah cukup cocok untuk dokumen saya. Karena isi dokumen saya bersifat informatif langsung, tidak membutuhkan analisis logis tingkat tinggi, sehingga saya lebih mengutamakan kecepatan interaksi (AI yang merespons instan) daripada kemampuan berfilosofi yang berlebihan.

---

## ⚙️ Parameter Tambahan (Non-Tunable)

Parameter ini sudah di-set optimal. Ubah hanya jika paham dampaknya.

```python
CHUNK_OVERLAP = 60     # overlap antar chunk (kata)
SCORE_THRESHOLD = 0.30 # skor minimum cosine similarity
```

**Penjelasan:**

| Parameter | Nilai Default | Fungsi |
| --- | --- | --- |
| `CHUNK_OVERLAP` | 60 | *Berfungsi menjaga kesinambungan kalimat dengan menduplikasi beberapa kata di akhir potongan ke awal potongan berikutnya. Jika terlalu kecil, kata ganti orang (dia/mereka) atau inti kalimat akan terputus. Jika terlalu besar, AI akan membaca kalimat yang sama berulang-ulang sehingga memboroskan memori.* |
| `SCORE_THRESHOLD` | 0.30 | *Berfungsi sebagai batas nilai minimal kemiripan dokumen. Jika AI memberikan jawaban yang sering ngelantur (tidak relevan), nilai ini harus dinaikkan (misal ke 0.50). Sebaliknya, jika sistem sering gagal dan memunculkan error "Tidak menemukan dokumen yang cocok", nilai ini harus diturunkan (misal ke 0.15).* |

---

## 📝 Tips Eksperimen

* Pakai PDF minimal 1 halaman, maksimal 10 halaman, biar gak berat
* Coba berbagai jenis pertanyaan: **definisi**, **konteks**, dan **reasoning**
* Catat kapan sistem salah & mengapa
* Ubah **satu parameter pada satu waktu** agar bisa melihat pengaruhnya secara jelas
* Prioritaskan kualitas insight dalam laporan

---

## 🔗 Referensi

* Dokumentasi lengkap parameter: buka `docs.html` di browser atau akses via menu **📖 Dokumentasi** di RAG Playground
* Source code: `app.py` — semua parameter ada di bagian `KONFIGURASI INTI`
