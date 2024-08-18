# 【成大資安社社課】Forensics Writeup
## Lab 1: 
1. 分析 pcap 檔案的封包組成 : Statics -> Protocal Hierarchy
![image](https://hackmd.io/_uploads/ByWXAb1jR.png)
先從觀察數量特別少的封包開始觀察
* ICMP
![image](https://hackmd.io/_uploads/B1IVyG1jC.png)
* DNS
![image](https://hackmd.io/_uploads/BkYSlGkoC.png)
* HTTP
![image](https://hackmd.io/_uploads/S1Z1WfkoC.png)
可以發現 10.153.11.112 一直去 GET 相同的 Request (/index.php?page=bHMgLg%3d)，所以我們可以去網頁試看看這個 Request 會跑出什麼內容。
![image](https://hackmd.io/_uploads/BkvFbGki0.png)
把 Host Name 改成 chall.nckuctf.org，且這個封包的 Dst Port 為 8100，所以完整網址為 : http://chall.nckuctf.org:8100/index.php?page=bHMgLg%3d ，瀏覽後發現沒什麼特別的地方。
![image](https://hackmd.io/_uploads/HkSnGGJsA.png)
2. 再觀察 HTTP 封包中有沒有特別的封包，發現到一個明顯和其他封包不同的 Request (/index.php?page=bHMgLg%3d)
![image](https://hackmd.io/_uploads/BJzeEMki0.png)
進去看之後是一個很空的網頁，顯示大小寫的 Index.php
![image](https://hackmd.io/_uploads/SyJMBfkjA.png)
再仔細觀察兩者 Request 會發現以下資訊 :
a. **%3DogLgMHb 的翻轉就是 bHMgLg%3d** 
b. bHMgLg%3d 的 Header 資訊蠻少的，%3DogLgMHb 的資訊較多
![2](https://hackmd.io/_uploads/Skqfdf1jA.png)
![螢幕擷取畫面 2024-08-18 144027](https://hackmd.io/_uploads/SkTaPzJoC.png)
c. 從 User-Agent 看到 %3DogLgMHb 是用瀏覽器手動戳的，而 bHMgLg%3d 是用指令 curl 的方式

3. bHMgLg%3d 中看到結尾的 %3d ，猜測這可能是 URL Encode 過的資訊，所以嘗試 Decode URL，結果為 **bHMgLg=**
![image](https://hackmd.io/_uploads/Hy1EoM1j0.png)
bHMgLg= 看起來非常像 **Base64**，Decode 它 !
雖然 Decode 結果為不合法輸入，但看到了特別的輸出 **ls .** !
![image](https://hackmd.io/_uploads/rJ66if1sA.png)

4. 同理，我們去測試看看 %3DogLgMHb，結果為 =ogLgMHb
![image](https://hackmd.io/_uploads/rJQPpfysR.png)
再看反轉它就變成了 Base64 的形式，接著 Decode !
![image](https://hackmd.io/_uploads/rkRsTMJsC.png)
果然正確找出內容為 **ls .**，
![image](https://hackmd.io/_uploads/SJYxCfJjR.png)
5. 從 (4) 我們可以得知這個 Reqest (/Index.php?page=%3DogLgMHb) 其實是在做 **ls .**，所以網頁才會列出 Index.php 和 index.php。
![image](https://hackmd.io/_uploads/BJefemJj0.png)
那這就代表我們可以在 page= 後插入指令 !
6. 根據上方解密流程，逆向建構 Payload : ls / -> Base64 加密 -> 字串反轉 -> urlencode
![image](https://hackmd.io/_uploads/HysYQ7ysC.png)
把 %3DowLqMHb 插入到 url page 後面(/Index.php?page=，注意 I 是大寫)
![image](https://hackmd.io/_uploads/rJqhSQ1iC.png)
找到 FLAG 的資料夾 flag_c603222fc7a23ee4ae2d59c8eb2ba84d，用 cat 進去看有什麼
![image](https://hackmd.io/_uploads/Syxi8mkjA.png)
拿到 FLAG: AIS3{0h!Why_do_U_kn0w_this_sh3ll1!1l!} !!!

7. 完整 Payload
```
curl "http://chall.nckuctf.org:8100/Index.php?page=$(urlencode $(echo "ls /" | base64 | rev))"
curl "http://chall.nckuctf.org:8100/Index.php?page=$(urlencode $(echo "cat /flag_c603222fc7a23ee4ae2d59c8eb2ba84d" | base64 | rev))"
``` 
![image](https://hackmd.io/_uploads/rkFAvXyiA.png)

## Lab 2:

