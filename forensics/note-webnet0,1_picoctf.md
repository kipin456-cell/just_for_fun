# webnet 1 dan webnet 0 picocCTF

## webnet 0 
catatan :
- pertama jika kita buka pcap nya pakai wireshark dan cek di bagian protokol hierarcy nya maka hanya akan muncul TLS(Transport Layer Scurity)

  <img width="1077" height="166" alt="image" src="https://github.com/user-attachments/assets/40f24b12-f4b7-4482-aab1-8b6eb2a71e60" />


  lalu jika kita ketik filter TLS juga yang muncul cuma bagian yang kuranng penting

  <img width="1364" height="356" alt="image" src="https://github.com/user-attachments/assets/1f3d2d1d-7ed4-41e1-b0c1-9b22da2b0a8c" />


- setelah kita decrypt menggunakan kunci yang telah di berikan maka akan muncul sebuah http

  <img width="1064" height="202" alt="image" src="https://github.com/user-attachments/assets/991e04c7-3fb9-47d6-a335-7d7d8839db84" />

  yang dimana ketika kita follow http tersebut maka kita akan mendapatkan Flag

  <img width="1354" height="468" alt="image" src="https://github.com/user-attachments/assets/04f8fc79-4ad2-4639-9b21-17025bbc6c1a" />

**flag = academy{nongshim.shrimp.crackers}***

## webnet 1
catatan :
- pertama jika kita buka pcap nya pakai wireshark dan cek di bagian protokol hierarcy nya maka hanya akan muncul TLS,sama seperti webnet0

  <img width="1091" height="187" alt="image" src="https://github.com/user-attachments/assets/c97e0fc1-de0c-4ac0-8603-e2d198e99f92" />

- tapi setelah kita apply kunci yang diberikan (caranya seperti tadi) dan ctrl + r untuk reload maka akan muncul http yang dimana di dalamnya ada fiel jpg

  <img width="1309" height="323" alt="image" src="https://github.com/user-attachments/assets/cf6d366a-e36c-4704-a3be-02ebfa5b9bc2" />

  jadi kita coba file > export object > http lalu pilih yang jpg.nanti kita akan mendapat file jpg nya dan jika kita cek metadatanya dengan exiftool maka kita akan dapat Flag

   <img width="1109" height="369" alt="image" src="https://github.com/user-attachments/assets/630102e5-3d3d-4ca2-9715-f5cb04d0eed3" />

  **Flag = academy{honey.roasted.peanuts}**
   



  
