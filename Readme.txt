Airbnb Bangkok
____________________________________________________________________________________________________________________________________________________________________
Airbnb adalah online marketplace yang menyediakan akomodasi bagi orang-orang yang ingin menyewa ataupun menyewakan kamar pribadi, apartemen, villa, maupun rumahnya. 
Bagi pemilik properti, tentunya layanan akomodasi ini dapat menjadi penghasilan tambahan karena mereka dapat menyewakan propertinya. Sementara bagi pengguna, 
properti di Airbnb bisa menjadi alternatif penginapan yang lebih terjangkau dibandingkan dengan hotel.
Di analisa ini Airbnb ingin mengetahui faktor yang mempengaruhi income.

Proses di mulai dengan pengenalan kolom dan tipe data masing2 kolom berikut, lalu di lanjut kan dengan beberapa tahapan.
#   Column                          Non-Null Count  Dtype  
---  ------                          --------------  -----  
 0   Unnamed: 0                      15854 non-null  int64  
 1   id                              15854 non-null  float64
 2   name                            15846 non-null  object 
 3   host_id                         15854 non-null  int64  
 4   host_name                       15853 non-null  object 
 5   neighbourhood                   15854 non-null  object 
 6   latitude                        15854 non-null  float64
 7   longitude                       15854 non-null  float64
 8   room_type                       15854 non-null  object 
 9   price                           15854 non-null  int64  
 10  minimum_nights                  15854 non-null  int64  
 11  number_of_reviews               15854 non-null  int64  
 12  last_review                     10064 non-null  object 
 13  reviews_per_month               10064 non-null  float64
 14  calculated_host_listings_count  15854 non-null  int64  
 15  availability_365                15854 non-null  int64  
 16  number_of_reviews_ltm           15854 non-null  int64  
 17  Min_income                      15854 non-null  int64 

DATA CLEANSING
Dalam proses ini di mulai dengan membuat copy dari dari dataset original nya
   # Copy data
   df_clean = df.copy()

1. Drop Kolom "Unnamed :", Kolom ini di hapus karena sudah ada indexing di dataset ini
2. Cek duplicate data --> Tidak ada data duplicate di dataset ini
3. Cek Price karena pada described data terdapat keanehan nilai min price yang bernilai '0' karena hampir tidak mungkin penyewaan property di Airbnb bernilai '0'
   o Menghitung jumlah listing per price untuk tau brp banyak listing yg memiliki nilai price = 0 --> terdapat 1 baris data dengan nilai price = 0
   o Cek kolom price dengan normalisasi Kolmogorov-Smirnov karena jumlah baris data >2000
   o karena tidakterdistribusi normal, nilai 0 di ganti dengan median dari price
4. Outlier
   Ada 2 variabel yg akan di cek data Outlier nya, yaitu price & minimum night
   o Outlier price di ambil dari nilai barisdata dengan  price > Q3 + 1.5 * IQR
   o Outlier minimum night menggunakan batas max di 31
     Karena jumlah outlier minimum_nights > Q3 + 1.5 * IQR jumlah nya 3168 (terlalu banyak untuk di drop dari dataset)
     kita coba menetapkan batas atas minimum_nights di angka 31 (jumlah max hari dalam 1 bulan). 
     Pertimbangan nya karena untuk sewa property ada batas waktu max penyewaan
   o Dari Pengececekan Jumlah Outlier dari price & minimum night. KIta putuskan untuk menghapus outlier price dan minimum night dengan batas angka minimum night 31 hari
     pertimbangan nya karena jumlah outiernya diharapkan tidak merubah pola data asli nya
   o Total listing data yg di drop dari outlier price > above and minimum_nights > 31 sebanyak 2188 list data atau 13.8% data original nya

ANALISA
1. perbandingan price & room type
   room_type
   Entire home/apt    1740.076625
   Hotel room         1690.757246
   Private room       1432.995311
   Shared room         638.356436
   --> Harga sewa Entire home/apt memiliki nilai rata2 paling tinggi

2. Harga di masing2 Neighborhood
   Rata2 harga sewa dari yang tertinggi berdasarkan neighbourhoods
   neighbourhood
   Parthum Wan       2235.323782
   Bang Rak          1938.990071
   Vadhana           1924.409846
   Samphanthawong    1866.323232
   Nong Chok         1827.545455
   Name: price, dtype: float64
   
   5 neighbourhoods dengan harga sewa rata2 tertinggi adalah Parthum Wan, Bang Rak, Vadhana, Samphanthawong, and Nong Chok merupakan area pusat kota yg di kelilingi 
   oleh pusat perbelanjaan, tempat wisata, kuliner dan seni (https://www.tripadvisor.com/Attractions-g293916-Activities-zfn15620288-Bangkok.html https://id.wikipedia.org/wiki/Bangkok )

   Harga rata sewa properti termurah berdasarkan neighbourhoods
   neighbourhood
   Lak Si           1143.833333
   Nong Khaem       1205.222222
   Don Mueang       1273.910180
   Lat Krabang      1358.523490
   Phasi Charoen    1370.219355

   Lak Si berada di lingkungan rumah bagi kompleks kuil Buddha, 
   sedangkan Nong Khaem merupakan salah satu dari tiga pusat pembuangan limbah padat utama di Bangkok (https://en.wikipedia.org/wiki/Nong_Khaem_district)

3. Potensi income
   Menghitung Minimum income dengan mengkalikan minimum night dan price
   df_clean['min_income'] = df_clean.minimum_nights * df_clean.price)
   
   df_clean.groupby(['neighbourhood','host_name','room_type']).Min_income.sum().sort_values(ascending=False)[:5]
   neighbourhood  host_name      room_type      
   Bang Rak       ISanook Hotel  Private room       2569950
   Vadhana        Joseph         Entire home/apt    1475808
   Ratchathewi    Miu Miu        Entire home/apt    1132860
   Vadhana        Max            Entire home/apt    1059392
   Ratchathewi    Aidan          Entire home/apt     991309
   Name: Min_income, dtype: int64

   Note :
   1. 5 host dengan minimum income terbesar berada memiliki property di top 5 neighbourhood
   2. Host yang memiliki room type Entire home/apt memiliki kemungkinan untuk mendapatkan minimum income yang besar
	
4. korelasi Price, Minimum Nights, and Occupancy Rate 
   Menghitung Occupancy. low <5 nights, medium 5-10 nights, high >10 nights
     room_type        Occupancy
     Entire home/apt  high         2119
                      low          4555
                      medium        817
     Hotel room       high           10
                      low           539
                      medium          3
     Private room     high          504
                      low          4381
                      medium        233
     Shared room      high            2
                      low           491
                      medium         12
     Name: minimum_nights, dtype: int64

     Note :
     - Tingkat Occupancy < 5 hari di kategori low sangat tinggi di tiap room_type. 
     - Entire home/apt memiliki nilai Occupancy > 10 hari yang cukup tinggi. ini bisa jadi potensi untuk meningkatkan income dengan memberikan discount di kelipatan hari tertentu

KORELASI ANTAR VARIABEL

corr = df_clean[['price', 'minimum_nights', 'number_of_reviews', 'reviews_per_month',
     'calculated_host_listings_count', 'number_of_reviews_ltm','min_income']].corr(method= "spearman")
corr
				price	minimum_nights	number_of_reviews	reviews_per_month	calculated_host_listings_count	number_of_reviews_ltm	min_income
price				1.000000	-0.038326	-0.001025	0.166013	0.088685	0.075393	0.469716
minimum_nights		       -0.038326	1.000000	0.065834	-0.035876	-0.097370	0.068636	0.833860
number_of_reviews	       -0.001025	0.065834	1.000000	0.588647	0.224982	0.714891	0.042207
reviews_per_month		0.166013	-0.035876	0.588647	1.000000	0.247848	0.711142	0.039155
calculated_host_listings_count	0.088685	-0.097370	0.224982	0.247848	1.000000	0.277369	-0.036675
number_of_reviews_ltm		0.075393	0.068636	0.714891	0.711142	0.277369	1.000000	0.086579
min_income			0.469716	0.833860	0.042207	0.039155	-0.036675	0.086579	1.000000

Note:
1. min_income sangat di pengaruhi oleh minimum_nights & price
2. jika popularitas dikaitan dengan number_of_reviews dan reviews_per_month maka ke 2 nya ini memiliki korelasi negatif terhadap price (review makin banyak maka price akan berkurang)
3. minimum_nights memiliki korelasi positif terhadap number_of_reviews. makin banyak yg review maka nilai minimum_nights akan naik


RECOMENDATION

1. Neighbourhood di pusat kota sangat potential untuk mendatangkan income lebih, sehingga sangat di rekomendasikan untuk lebih mempublish property di pusat kota
2. Airbnb bisa memberikan saran untuk host perorangan (bukan pemilik hotel) yg memiliki property Entire home/apt untuk menyewakan property nya, 
   karena kemungkinan untuk mendapatkan income yg cukup besar
3. Memberikan tambahan potongan harga untuk penyewaan berikutnya bagi pengguna yg memberikan review, karena review dari pengguna setidaknya mempengaruhi minimum_nights, 
   yang akan berdampak pada peningkatan income bagi host & Airbnb

