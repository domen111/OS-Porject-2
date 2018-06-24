# OS Prpject2

## Design
這是一個Master-Slave架構，我們需要讓Master以及Slave device都支援mmap。  
Master side:  
* user_program/master.c:  
將檔案map到user_program記憶體後，再將device記憶體map到user_program，而後利用memcpy將檔案拷貝至該map記憶體，再透過ioctl通知device mapping已完成。
* master_device/master_device.c:  
抓到user_program中master.c傳送的通知後，找到記憶體資料，再透過ksend傳送記憶體資料給slave device。
  
Slave side:  
* slave_device/slave_device.c:  
接收到master device傳來的記憶體資料，將記憶體資料利用memcpy拷貝到buffer中，開啟socket等待user_program/slave.c連線。  
* user_program/slave.c:  
連線後取得資料，將device處記憶體map至user_program，再將內容map到將輸出的檔案。  

## Test
```
[1] Master: fcntl, Slave: fcntl

==file1_in==
Master:
Transmission time: 0.049600 ms, File size: 4 bytes
Slave:
Transmission time: 0.080500 ms, File size: 4 bytes

==file2_in==
Master:
Transmission time: 0.230100 ms, File size: 577 bytes
Slave:
Transmission time: 0.297300 ms, File size: 577 bytes

==file3_in==
Master:
Transmission time: 0.147800 ms, File size: 9695 bytes
Slave:
Transmission time: 0.225200 ms, File size: 9695 bytes

==file4_in==
Master:
Transmission time: 8.909200 ms, File size: 1502860 bytes
Slave:
Transmission time: 9.447900 ms, File size: 1502860 bytes

[2] Master: mmap, Slave: mmap

==file1_in==
Master:
Transmission time: 0.082200 ms, File size: 4 bytes
Slave:
Transmission time: 0.189900 ms, File size: 4 bytes

==file2_in==
Master:
Transmission time: 0.191900 ms, File size: 577 bytes
Slave:
Transmission time: 0.260300 ms, File size: 577 bytes

==file3_in==
Master:
Transmission time: 0.130900 ms, File size: 9695 bytes
Slave:
Transmission time: 0.308100 ms, File size: 9695 bytes

==file4_in==
Master:
Transmission time: 1.833000 ms, File size: 1502860 bytes
Slave:
Transmission time: 4.472500 ms, File size: 1502860 bytes
```

## Analysis
File I/O V.S. Memory-mapped I/O  
Master side:  
檔案漸大時，Master方使用會比使用fcntl快。  

Reason:  
mmap時呼叫system call的次數較少。  

Slave side:  
Slave方使用mmap與fcntl的時間快慢無固定。  

Reason:  
接收檔案無法事先得知大小，導致得事後壓縮，因此不一定。

## Member
蘇多門: Master-Slave programming  
王行健: ioctl研究  
余庭光: Master-Slave device部分研究、report  
