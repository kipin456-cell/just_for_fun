# Matryoshka doll-picoCTF

#deskripsi chall
Matryoshka dolls are a set of wooden dolls of decreasing size placed one inside another. What's the final one? Image: dolls.jpg

Hints :
1.Wait, you can hide files inside files? But how do you find them?

2.Make sure to submit the flag as academy{XXXXX}

##langkah penyelesaian 

pertama jika kita download dan buka dolls.jpg maka akan muncul

<img width="610" height="763" alt="image" src="https://github.com/user-attachments/assets/2e7a4b4c-6dcc-4f29-b13b-16b75b40fc48" />

karena di petunjuk chall kita di kasih tau bahwa "Wait, you can hide files inside files? But how do you find them?" yang artinya "tunggu,kamu bisa menyebunyikan files di dalam file?tapi apakah kamu tahu cara menyari mereka?" jadi karena petunjuk tersebut,saat ku binwalk dolls.jpg maka akan muncul

<img width="1096" height="198" alt="image" src="https://github.com/user-attachments/assets/b26d8829-e555-46e2-9771-2f98328a2778" />

setelah ku binwalk ketahuan bahwa ada file zip yang menempel di dolls.jpg yang isinya file jpg ```272492        0x4286C         Zip archive data, at least v2.0 to extract, compressed size: 378929, uncompressed size: 383919, name: base_images/2_c.jpg``` 
jadi aku menggunakan command ```dd if=dolls.jpg of=output.zip skip=272492 bs=1``` untuk memisah file di alamat 272492 yang ada di dolls.jpg dan akan disimpan di file output.zip

<img width="826" height="208" alt="image" src="https://github.com/user-attachments/assets/b46947ef-0409-4044-b19d-f95fa6ddf322" />

ketika ku unzip output.zip maka akan memunculkan folder base_image yang isinya 2_c.jpg

<img width="336" height="77" alt="image" src="https://github.com/user-attachments/assets/795157b8-4d20-4cdf-bd81-f92de89c5cd6" />


