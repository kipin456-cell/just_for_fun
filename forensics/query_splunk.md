1.Hitung total event untuk setiap file source di index="evtx_attck", lalu urutkan dari jumlah terbanyak ke paling sedikit.

``` index="evtx_attck" | stats count by source | sort +count ```

2.Tampilkan jumlah variasi EventCode unik (dc) beserta daftar angkanya untuk masing-masing host.	

``` index="evtx_attck" | stats dc(EventCode) as jumlah_event_code values(EventCode) by host ```

3.Cari 5 Image (proses) yang paling sering muncul (top) dan bandingkan dengan 5 yang paling jarang muncul (rare).

paling sering muncul 
``` "evtx_attck" | top limit=5 Image ```

paling jarang muncul 

``` index="evtx_attck" | rare limit=5 Image ```

4.Hitung berapa banyak Account_Name unik yang tercatat di setiap source, lalu tampilkan daftar nama akun tersebut dalam satu kolom.

``` index="evtx_attck" | stats dc(Account_Name) as jumlah_account values(Account_Name) as account by source ```

5.Tampilkan waktu kemunculan pertama dan terakhir tiap Account_Name dalam format tanggal dan jam yang terbaca manusia (bukan epoch).

``` index="evtx_attck" | stats earliest(_time) as pertama latest(_time) as terakhir by Account_Name | convert ctime(pertama) ctime(terakhir) ```

6.Buat tabel kronologi aktivitas logon yang hanya menampilkan waktu, Account_Name, Logon_Type, dan source, diurutkan dari waktu terlama ke terbaru.

``` index="evtx_attck" | table _time Account_Name Logon_type source | sort -_time ```

7.Hitung total event pembuatan proses baru di log Windows-mu, lalu kelompokkan berdasarkan host atau ComputerName.

``` index="evtx_attck" | stats count dc(EventCode=4688) by host ```

8.Gunakan eval dan where untuk menyaring event yang memiliki isi baris perintah lebih dari 50 karakter agar eksekusi yang tidak biasa langsung terlihat.

``` index="evtx_attck" | eval panjang = len(CommandLine) | where panjang > 50 | table EventCode host CommandLine ```

9.Buat kolom kategori baru menggunakan eval untuk melabeli apakah event berasal dari log Security atau selainnya, lalu hitung total tiap kategori.

``` index="evtx_attck" | eval kategori = if(LogName == "Security" , "log security" , "log lain") | stats count by kategori ```
