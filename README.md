# Proyek---UTS

> # Identitas 

> Nama : Husna
>
> NIM : 160411100018
>
> Matkul : Penambangan dan Pencarian Web 



## Pengantar

- [ ] Program ini menggunakan Bahasa Python dengan library :

  - Beautifulsoup4
  - requests
  - SQlite3 

- [ ] Mengambil data teks di Website (IMDb Top Box Office) : "https://jatim.sindonews.com/loadmore/0/"

  

### Penjelasan Program 

setelah itu barulah kita bisa mecari teks yang mau kita ambil pada web tersebut, untuk contoh tersebut kita mengambil `` Judul Berita``  dan ``Isi Berita`` , rating hasilnya pada hari `` weekend`` dan ``gross`` , dan untuk memperjelas tempat data yang mau kita ambil pahami terlebih dahulu di html web. 



> Untuk memanggil Library kita menggunakan code seperti di bawah ini 

```
from math import log10
import requests
from bs4 import BeautifulSoup
import sqlite3
from Sastrawi.StopWordRemover.StopWordRemoverFactory import StopWordRemoverFactory
from Sastrawi.Stemmer.StemmerFactory import StemmerFactory
import csv
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
import numpy as np
import warnings
warnings.filterwarnings("ignore")
```

> disini kita menggunakan link buat halaman web yang akan kita ambil datanya, dibagian ``src`` kita mau ngambil judul sedangkan di bagian ``news`` kita mau ngambil bagian isinya berita tersebut. 

```src = "https://jatim.sindonews.com/loadmore/0/"
src = "https://jatim.sindonews.com/loadmore/0/"

page = requests.get(src)

soup = BeautifulSoup(page.content, 'html.parser')

news = soup.findAll(class_='caption')
```

> digunakan untuk membuat database menggunakan ``sqlite3`` dan untuk menampung atau menyimpan data.

```koneksi = sqlite3.connect ('berita.db')
koneksi.execute('''CREATE TABLE if not exists berita
         (NAMA_EVENT VARCHAR NOT NULL,
         DESKRIPSI VARCHAR NOT NULL);''')
```

sedangkan untuk mengambil data tanpa ngikut code yang ada di web kita bisa menggunakan code dibawah ini, akan tetapi kita pahami code web yang mau kita ambil karena untuk penepatan judul atau lainnya berbeda - beda dengan web yang lain.

> disini code untuk mengambil data terus di simpan atau insert ke tabel yang sudah kita buat di databse seblumnya.

```for i in range (len(news)):
    link = news[i]['href']
    page = requests.get(link)
    soup = BeautifulSoup(page.content, 'html.parser')
    ivent = soup.find(class_='tahoma').getText()
    deskripsi = soup.find(class_ = 'article col-md-11')
    paragraf = deskripsi.findAll('p')
    p = ''
    for a in paragraf:
        p+=a.getText() +''
    cek = koneksi.execute("SELECT * FROM berita where NAMA_EVENT=?", (ivent,))
    cek = cek.fetchall()
    if len(cek) == 0 :
        koneksi.execute('INSERT INTO berita values (?,?)', (ivent, p));

koneksi.commit()
tampil = koneksi.execute("SELECT * FROM berita")
with open ('data_crawler.csv', newline='', mode='w')as employee_file :
    employee_writer = csv.writer(employee_file, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)
    for i in tampil:
        employee_writer.writerow(i)

tampil = koneksi.execute("SELECT * FROM berita")
isi = []
for row in tampil:
    isi.append(row[1])
    #print(row)
print("berita")
```

> untuk menghitung kata yang sama dalam banyak artiker disalam 1 link web (VSM)

```factory = StopWordRemoverFactory()
stopword = factory.create_stop_word_remover ()

factory = StemmerFactory ()
stemmer = factory.create_stemmer ()

tmp = ''
for i in isi:
    tmp = tmp + ' ' +i

hasil = []
for i in tmp.split():
    try :
        if i.isalpha() and (not i in hasil) and len(i)>1:
            # Menghilangkan Kata tidak penting
            stop = stopword.remove(i)
            if stop != "":
                stem = stemmer.stem(stop)
                hasil.append(stem)
    except:
        continue
katadasar=np.array(hasil)
print("katadasar")
```

> untuk menyeleksi kata dasar yang ada di dalam web atau artikel yang telah di crawl sesuai dengan kamus KBBI , jadi sebelumnya kita menyiapkan kata daasar yang benar yang disimpan di KBI kemudian di seleksi satu - satu yang sekiranya tidak termasuk dalam kamus maka tidak akan di print atau hanya di lewatkan saja.

```koneksi = sqlite3.connect('KBI.db')
cur_kbi = koneksi.execute("SELECT* from KATA")
    
def LinearSearch (kbi,kata):
    found=False
    posisi=0
    while posisi < len (kata) and not found :
        if kata[posisi]==kbi:
            found=True
        posisi=posisi+1
    return found
berhasil=[]
for kata in cur_kbi :
    ketemu=LinearSearch(kata[0],katadasar)
    if ketemu :
        kata = kata[0]
        berhasil.append(kata)
print(berhasil)
katadasar = np.array(berhasil)
```

> supaya hasil crawl yang sudah di seleksi kita tertata rapi kita menggunaan code di bawah ini menjadi matrix.

```matrix = []
matrix = []
for row in isi :
    tamp_isi=[]
    for a in katadasar:
        tamp_isi.append(row.lower().count(a))
    matrix.append(tamp_isi)
print("kbi")
```

> untuk menyimpan hasil crawl yang telah menjadi matrik kedalam bentuk CSV.

```with open ('data_matrix.csv', newline='', mode='w')as employee_file :
with open ('data_matrix.csv', newline='', mode='w')as employee_file :
    employee_writer = csv.writer(employee_file, delimiter=',', quotechar='"', 		quoting=csv.QUOTE_MINIMAL)
    employee_writer.writerow(katadasar)
    for i in matrix :
        employee_writer.writerow(i)
```

> kemudian kita hitung kata yang sama dalam 1 dokumen atau 1 link tersebut dengan menggunakan tf - idf 

```df = list()
for d in range (len(matrix[0])):
    total = 0
    for i in range(len(matrix)):
        if matrix[i][d] !=0:
            total += 1
    df.append(total)

idf = list()
for i in df:
    tmp = 1 + log10(len(matrix)/(1+i))
    idf.append(tmp)

tf = matrix
tfidf = []
for baris in range(len(matrix)):
    tampungBaris = []
    for kolom in range(len(matrix[0])):
        tmp = tf[baris][kolom] * idf[kolom]
        tampungBaris.append(tmp)
    tfidf.append(tampungBaris)
tfidf = np.array(tfidf)
print("tfidf")
```

> untuk menyimpan hasil tf - idf dalam bentuk CSV dan untuk menyeleksi fitur, me - Klustering, Silhoute hasil dati data yang di telah di ambil atau crawl dan yang telah di seleksi menggunakan KBI.

```def pearsonCalculate(data, u,v):
def pearsonCalculate(data, u,v):
  "i, j is an index"
    atas=0; bawah_kiri=0; bawah_kanan = 0
    for k in range(len(data)):
        atas += (data[k,u] - meanFitur[u]) * (data[k,v] - meanFitur[v])
        bawah_kiri += (data[k,u] - meanFitur[u])**2
        bawah_kanan += (data[k,v] - meanFitur[v])**2
    bawah_kiri = bawah_kiri ** 0.5
    bawah_kanan = bawah_kanan ** 0.5
    return atas/(bawah_kiri * bawah_kanan)
def meanF(data):
    meanFitur=[]
    for i in range(len(data[0])):
        meanFitur.append(sum(data[:,i])/len(data))
    return np.array(meanFitur)
def seleksiFiturPearson(katadasar, data, threshold):
    global meanFitur
    meanFitur = meanF(data)
    u=0
    while u < len(data[0]):
        dataBaru=data[:, :u+1]
        meanBaru=meanFitur[:u+1]
        katadasarBaru=katadasar[:u+1]
        v = u
        while v < len(data[0]):
            if u != v:
                value = pearsonCalculate(data, u,v)
                if value < threshold:
                    dataBaru = np.hstack((dataBaru, data[:, v].reshape(data.shape[0],1)))
                    meanBaru = np.hstack((meanBaru, meanFitur[v]))
                    katadasarBaru = np.hstack((katadasarBaru, katadasar[v]))
                    
            v+=1
        data = dataBaru
        meanFitur=meanBaru
        katadasar = katadasarBaru
        if u%50 == 0 : print("proses : ", u, data.shape)
        u+=1
    return katadasar, data

katadasarBaru, fiturBaru = seleksiFiturPearson(katadasar, tfidf, 0.8)

kmeans = KMeans(n_clusters=3, random_state=0).fit(fiturBaru);
print(kmeans.labels_)
classnya = kmeans.labels_
s_avg = silhouette_score(fiturBaru, classnya, random_state=0)

print("cluster")
```

> code di bawah ini digunakan untuk menyimpandata yang telah diproses dalam bentuk CSV. 

```with open('hasil_cluster.csv', newline='', mode='w') as employee_file:
with open('hasil_cluster.csv', newline='', mode='w') as employee_file:
   employee_writer = csv.writer(employee_file, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)
    for i in classnya.reshape(-1,1):
        employee_writer.writerow(i)

with open('Seleksi_Fitur.csv', newline='', mode='w') as employee_file:
    employee_writer = csv.writer(employee_file, delimiter=',', quotechar='"', quoting=csv.QUOTE_MINIMAL)
    employee_writer.writerow([katadasarBaru])
    for i in fiturBaru:
        employee_writer.writerow(i)
```



> ## Perlu di ketahui untuk menjalan program tersebut harus terhubung ke internet karena kalau tidak terkoneksi dengan internet code tersebut tidak bakal berjalan atau Error :)
