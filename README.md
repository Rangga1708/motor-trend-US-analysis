# Analisis Data Motor Trend US

Halo semua. Pada kesempatan kali ini, saya akan mencoba menganalisis salah satu dataset yang ada di R, yaitu `mtcars`, tetapi menggunakan Python. Ini adalah hasil analisis data saya pertama kali (di luar perkuliahan) yang saya publikasikan. Saya hanya ingin membagikan ilmu yang sudah saya pelajari selama ini kepada teman-teman semua yang ingin belajar menganalisis data. Tentunya saya juga bisa belajar dari teman-teman yang lebih expert di bidang analisis data melalui komentar yang nantinya teman-teman berikan. 

Di sini, saya akan menyelidiki apa saja yang mempengaruhi penggunaan bahan bakar (mpg) dari sebuah mobil. Beberapa variabel yang ada di dataset `mtcars` ini adalah
```
mpg  : miles per galoon
cyl  : number of cylinders
disp : displacement
hp   : gross horsepower
drat : rear axle ratio
wt   : weight (1000 lbs)
qsec : 1/4 mile time
vs   : engine (0 = V-shaped, 1 = straight)
am   : transmission (0 = automatic, 1 = manual)
gear : number of forward gears
carb : Number of carburetors
```
Untuk selanjutnya, mpg akan saya sebut sebagai variabel dependen sedangkan sisanya saya sebut sebagai variabel independen.

Saya akan menggunakan analisis regresi linear sederhana. Tentunya bagaimana menganalisis data tiap orang berbeda, tetapi saya menggunakan analisis ini karena analisis ini lebih mudah. Untuk dataset `mtcars` bisa teman-teman lihat **di sini** dan untuk syntax yang saya buat dapat dilihat **di sini**. 

Pertama saya akan mengimport beberapa modules python yang saya butuhkan. Modules yang akan saya gunakan adalah sebagai berikut:
```Python
import pandas as pd
import plotly.graph_objs as go
from plotly.subplots import make_subplots
import statsmodels.api as sm
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.sandbox.stats.runs import runstest_1samp
from sklearn.model_selection import train_test_split
from scipy.stats import shapiro
```

## Data Train dan Data Test
Sebagai tahapan awal, saya akan membagi datanya menjadi 2, yaitu data train dan data test. Data train akan saya gunakan untuk membentuk model sedangkan data test akan saya gunakan untuk menguji model yang sudah terbentuk. Dari keseluruhan data, saya akan pilih 80% data secara acak sebagai data train, dan 20% data secara acak sebagai data test.
```Python
#input data
df = pd.read_excel ('mtcars.xlsx')

#bagi antara variabel independen dengan variabel dependen
X = df[['cyl','disp','hp','drat','wt','qsec','vs','am','gear','carb']]
Y = df['mpg']

#bagi data menjadi train dan test
X_train, X_test, Y_train, Y_test = train_test_split(X, Y, test_size=0.2)

#reset index
X_train = X_train.reset_index(drop = True)
X_test = X_test.reset_index(drop = True)
Y_train = Y_train.reset_index(drop = True)
Y_test = Y_test.reset_index(drop = True)
```
Data train dan data test yang saya gunakan dapat teman-teman lihat **di sini** dan **di sini**.

## Asumsi awal
Setelah membagi datanya, selanjutnya akan selidiki beberapa asumsi awal analisis regresi linear. Asumsi yang akan diselidiki adalah
1. Cek apakah ada hubungan linear antara linearitas dari variabel dependen dengan variabel independen. (***Linearitas***)
2. Cek apakah setiap variabel independen saling mempengaruhi satu sama lain. (***Multikolinearitas***)

### Linearitas
Pertama saya akan mengecek apakah terdapat hubungan linear antara variabel dependen dengan variabel independen lainnya. Untuk menyelidikinya, kita bisa melihatnya dari scatter plot di bawah ini.
```Python
#siapkan figure subplot
fig = make_subplots(
    rows = 5, cols = 2,
    subplot_titles = ("Cyl and Mpg", 
                      "Disp and Mpg", 
                      "Hp and Mpg", 
                      "Drat and Mpg", 
                      "WT and Mpg",
                      "Qsec and Mpg", 
                      "VS and Mpg", 
                      "AM and Mpg", 
                      "Gear and Mpg", 
                      "Carb and Mpg")
)

#buat plot masing-masing subplot
index = 0
for i in range(1,6):
    for j in range(1,3):
        fig.add_trace(go.Scatter(x = X[X.keys()[index]], y = Y, mode = 'markers', marker = {'color' : 'black'}), row = i, col = j)
        fig.update_xaxes(title_text = X.keys()[index], row = i, col = j)
        fig.update_yaxes(title_text = 'mpg', row = i, col = j)
        index = index+1

#atur layout        
layout = {
    'height' : 2000,
    #'width' : 1000,
    'title' : {
        'text' : 'Scatter Plot',
        'x' : 0.5
    },
    'showlegend' : False
}

fig.update_layout(layout)

fig.show()
```
![Scatter Plot Lineartias](https://github.com/Rangga1708/Motor_Trend_US_Analysis/blob/master/Scatter_Plot_Linearitas.png)

Berdasarkan scatter plot di atas, tampak bahwa terdapat hubungan linear antara beberapa variabel independen dengan variabel mpg. Kita dapat simpulkan beberapa hal sebagai berikut:
1. Terdapat hubungan linear negatif antara variabel cyl dan mpg. Artinya, semakin banyak silinder pada mobil, semakin sedikit bahan bakar yang terpakai.
2. Terdapat hubungan linear negatif antara variabel disp dan mpg. Artinya, semakin besar displacement pada mobil, semakin sedikit bahan bakar yang terpakai.
3. Terdapat hubungan linear negatif antara variabel hp dan mpg. Artinya, semakin besar horsepower mobil, semakin sedikit bahan bakar yang terpakai.
4. Terdapat hubungan linear positif antara variabel drat dan mpg. Artinya, semakin besar rear axle gear ratio pada mobil, semakin banyak bahan bakar yang terpakai.
5. Terdapat hubungan linear negatif antara variabel wt dan mpg. Artinya, semakin berat mobil, semakin sedikit bahan bakar yang terpakai.
6. Terdapat hubungan linear positif antara variabel qsec dan mpg. Artinya, semakin lama mobil menempuh jarak 1/4 mil , semakin banyak bahan bakar yang terpakai.
7. Konfigurasi silinder yang berbentuk V akan memakai bahan bakar lebih sedikit dibandingkan dengan konfigurasi silinder yang berbentuk garis lurus.
8. Mobil matic akan memakai bahan bakar lebih sedikit dibandingkan dengan mobil manual.
9. Mobil dengan 3 gigi akan memakai bahan bakar lebih sedikit dibandingkan dengan mobil dengan 5 gigi dibandingkan dengan mobil dengan 4 gigi.
10. Terdapat hubungan linear negatif antara variabel carb dan mpg. Artinya, semakin banyak karburator pada mobil, semakin sedikit bahan bakar yang terpakai.

Sekilas mungkin untuk variabel gear tidak memiliki hubungan linear dengan mpg. Tapi kita asumsikan variabel gear memiliki hubungan linear dengan mpg.

## Multikolinearitas
Selanjutnya, kita akan menyelidiki apakah variabel-variabel independen pada data kita saling mempengaruhi atau tidak. Kita akan mengecek nilai VIF dari masing-masing variabel. Jika VIF > 10, maka variabel tersebut memiliki hubungan dengan variabel independen lainnya. Kita akan membuang variabel dengan VIF terbesar kemudian kita akan cek kembali VIF variabel-variabel yang tersisa (Konstanta akan tetap dipertahankan). Kita akan ulangi terus sampai kita peroleh VIF dari semua variabel kurang dari 10. Berikut rangkuman nilai VIF dari setiap iterasi.
```Python
X = sm.add_constant(X)

#hitung VIF
VIF = pd.Series([variance_inflation_factor(X.values, i) 
               for i in range(X.shape[1])], 
              index=X.columns)
print(VIF,'\n')

#jika VIF konstanta terbesar, cari terbesar kedua
if list(VIF).index(max(VIF)) == 0:    
    new_VIF = VIF[1:]
    drop_variable_VIF = max(new_VIF)

while drop_variable_VIF > 10 and len(X.keys())!=0:
    #buang variabel dengan VIF terbesar dan lebih dari 10
    drop_variable_index = list(VIF).index(drop_variable_VIF)                          
    drop_variable = X.keys()[drop_variable_index]
    X = X.drop(columns = drop_variable) 
    
    VIF = pd.Series([variance_inflation_factor(X.values, i) 
               for i in range(X.shape[1])], 
              index=X.columns)
    print(VIF,'\n')
    
    #jika VIF konstanta terbesar, cari terbesar kedua
    if list(VIF).index(max(VIF)) == 0:    
        new_VIF = VIF[1:]
        drop_variable_VIF = max(new_VIF)
```
Iterasi | Variabel | VIF
------- | -------- | ---
1       | const <br> cyl <br> disp <br> hp <br> drat <br> wt <br> qsec <br> vs <br> am <br> gear <br> carb | 2149.280570 <br> 19.691673 <br> 22.176578 <br> 10.308235 <br> 4.114787 <br> 17.319301 <br> 7.284777 <br> 4.702383 <br> 5.234086 <br> 7.002498 <br> 11.759377
2       | const <br> cyl <br> hp <br> drat <br> wt <br> qsec <br> vs <br> am <br> gear <br> carb | 2149.201123 <br> 18.589723
hp          6.615471
drat        4.024375
wt          8.076653
qsec        6.744210
vs          4.654625
am          5.176873
gear        7.000588
carb        7.077009

Dari hasil perhitungan di atas, dapat kita lihat bahwa ternyata nilai dari cyl dan disp dipengaruhi oleh variabel independen lainnya. Dengan demikian, kita perlu membuang variabel tersebut agar analisis regresi dapat dilakukan. 

Selanjutnya, untuk memilih variabel apa saja yang signifikan terhadap model, kita akan menggunakan stepwise regression, khususnya backward elimination. Pertama, kita akan membentuk model menggunakan semua variabel. Kemudian kita akan lihat apakah ada variabel yang tidak signifikan (jika p-value lebih dari $\alpha$). Jika ada variabel yang tidak signifikan, kita keluarkan variabel tersebut (terutama yang p-value nya terbesar) kemudian kita bentuk model yang baru. Langkah ini diteruskan sampai semua variabel signifikan terhadap model. Berikut rangkuman dari model-model yang diperoleh dari hasil perhitungan Python.
**Tabel**

Dari rangkuman model di atas, kita peroleh 6 model sebagai berikut:
1. $mpg = 5.0293 + 0.0155 hp + 2.3856 drat - 0.3515 wt + 0.1904 qsec + 2.4657 vs + 6.0699 am + 1.3572 gear - 2.1244 carb$
2. $mpg = 4.7143 + 0.0146 hp + 2.4828 drat + 0.1141 qsec + 2.6869 vs + 6.2544 am + 1.4795 gear - 2.2185 carb$
3. $mpg = 7.0518 + 0.0132 hp + 2.4601 drat + 2.9060 vs + 6.1124 am + 1.4651 gear - 2.2180 carb$
4. $mpg = 10.1872 + 2.0471 drat + 2.6007 vs + 5.9254 am + 1.3103 gear - 1.8558 carb$
5. $mpg = 12.9007 + 2.2434 drat + 3.1553 vs + 7.1908 am - 1.6252 carb$
6. $mpg = 19.8640 + 4.1979 vs + 8.6664 am - 1.5869 carb$

Walaupun semua variabel pada model ke-6 sudah signifikan, tetapi belum tentu model tersebut adalah model yang terbaik. Kita akan pilih model yang terbaik dengan membandingkan beberapa nilai statistik setiap modelnya, seperti:
1. R-squared (semakin besar nilainya, semakin baik modelnya)
2. Adj R-Squared (semakin besar nilainya, semakin baik modelnya)
3. Sum squared error (SSE) (semakin kecil nilainya, semakin baik modelnya)
4. Log-Likelihood (semakin kecil nilainya, semakin baik modelnya)
5. AIC (semakin kecil nilainya, semakin baik modelnya)
6. BIC (semakin kecil nilainya, semakin baik modelnya)

Berikut rangkuman nilai-nilai statistik dari setiap modelnya.
**Tabel**

Dari beberapa nilai statistik model di atas, diperoleh:
1. Model 1 memiliki nilai R-Squared terbesar
2. Model 3 memiliki nilai Adj R-Squared terbesar
3. Model 1 memiliki nilai Sum Squared Error terkecil
4. Model 6 memiliki nilai Log-Likelihood terkecil
5. Model 5 memiliki nilai AIC terkecil
6. Model 5 memiliki nilai BIC terkecil

Dapat dilihat bahwa model 6 hanya memenuhi 1 kriteria model terbaik, sedangkan model 1 dan model 5 memenuhi 2 kriteria model terbaik. Karena model 1 memiliki lebih banyak variabel independen, kita akan memilih model 1 sebagai model terbaik (untuk sementara). Kita akan melakukan diagnostic checking pada model yang sudah kita pilih. Diagnostic checking yang kan kita lakukan adalah
1. Cek apakah residual berdistribusi normal.
2. Cek apakah terdapat autokorelasi dari residual.
3. Cek homoskedastisitas dari residual.

Untuk mengecek normalitas dari residual, kita akan melakukan uji shapiro-wilk. Hipotesis nol dari uji ini adalah residual berdistribusi normal. Hipotesis nol akan ditolak jika p-value $< 0.05$.

Karena p-value $=0.8178114295005798 > 0.05$, maka hipotesis nol tidak ditolak sehingga residual berdistribusi normal.

Selanjutnya untuk mengecek autokorelasi dari residual, kita akan melakukan uji runs. Hipotesis nol dari uji ini adalah residual bersifat acak (tidak ada autokorelasi). Hipotesis akan ditolak jika p-value $<0.05$.
