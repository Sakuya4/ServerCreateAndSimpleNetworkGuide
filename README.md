# 伺服器建置之網路概念以及網域建置方法-以VMware、Vsphere虛擬化環境為例


## 目錄
---
- [網路概念](##網路概念)
- [AD、網域建置](##AD伺服器建置)
- [建置結果展示以及驗證](##建置結果展示以及驗證)
---

## 網路概念
伺服器建置的網路概念其實不難，為了讓未來想要成為網管或者MIS的大學、研究生可以彌補課堂上的不足，或者剛入行的網管、MIS較不熟悉，可以通過簡單的講義去熟悉網路的概念。筆者利用VMware, VSphere虛擬化的環境，說明實際上公司的系統會遇到的環境。
### 1. OSI七層模型
也許從這邊說明會很複雜，筆者相信，在大學網路概論、計算機網路課程都有學過這個模型，以下用MarkDown語法簡單畫了一個模型圖：

:::info
A（application layer）：DHCP、DNS、FTP、Gopher、HTTP、SPDY、HTTP/2、IMAP4、IRC、NNTP、XMPP、POP3、SIP、SMTP、SNMP、SSH、TELNET、RPC、RTCP、RTP、RTSP、SDP、SOAP、GTP、STUN、NTP、SSDP

-----

P（presentation layer）：null

-----

S（session layer）：null

-----

T（transport layer）：TCP（T/TCP · Fast Open）、UDP、DCCP、SCTP、RSVP、PPTP、TLS/SSL

-----

N（network layer）：IP（v4·v6）、ICMP（v6）、IGMP、IS-IS、IPsec、BGP、RIP、OSPF、RARP

-----

D（data link layer）：Wi-Fi（IEEE 802.11）、ARP、WiMAX（IEEE 802.16）、ATM、DTM、權杖環、乙太網路、FDDI、訊框中繼、GPRS、EV-DO、HSPA、HDLC、PPP、PPPoE、L2TP、ISDN、SPB、STP

------

P  (physical layer)：乙太網路、數據機、電力線通訊、同步光纖網路、G.709、光導纖維、同軸電纜、雙絞線。
:::

不用去硬背，先簡單看過，了解一下大概哪一層在做什麼。未來在Debug的時候腦海中可以套一下OSI七層，或者你自己有一套你自己的模型，以我為例，我腦袋在Debug就直接建立「簡單三層」，由上至下，最上層應用層、中間網路層、最底下實體層。假設虛擬機或主機網路不通，你可以從這三層架構下去排查問題。

### 2. IPv4網路
我們在建置網域的時候需要有AD, 要建置AD必須將機器升級為DC, 這個DC通常部會讓他的IP浮動，舉個例子：一個學校的校長，他的辦公室不是固定一間的。所以我們需要給他一組資訊，這些資訊有：
:::success
1. IP，例如：192.168.1.X，簡單想就是門牌號碼，由管理員或者使用者設定，一個公司的IP位置通常會比較接近。
2. Mask，例如：255.255.255.0，遮罩，背遮罩所蓋住的即是網路部分，沒被蓋住的地方就是我們主機的部分。遮罩分為A、B、C、D四種Class, 常見的是C Class, 也就是範例所舉的3個255，在舉個例子，A Class就是一個255。
3. Gateway，他的用途是讓你出外網的，其需有一個實體裝置，如L3 Switch, Firewall, Router，此裝置會有一個專門出外網的端口，並且給他一個IP，而當本機指向這個IP就可以順利出外網，例如：192.168.1.254。
4. DNS，簡單說就是域名解析，我們可以輸入如Google.com就是依靠DNS，通常我在建AD的時候DNS會指向168.95.1.1，而在測試能不能正常通網路我也會Ping 168.95.1.1。
:::
## AD、網域建置
對AD不熟的同學不要緊張，把他想成網域內的管理者就好，以下示範在虛擬化環境建置虛擬機以及網域、AD。

1. 首先建立虛擬機，特別注意連接埠群組需要先設定好；ISO檔上傳至掛載在VMware上面的NAS、硬碟，並且在建置虛擬機時將其選取。而其他設定如記憶體、處理器、儲存空間就依你自己情況而定。都設定完以後，接續安裝OS。
![image](https://hackmd.io/_uploads/BJqQgOdfC.png)

2. 接下來就是Windows Server裡面的設定，以下說明都是使用Windows Server 2022
:::info
修改IPv4資訊。因為習慣問題，隨後會直接改主機名稱，因為是AD, 你可以給他叫XXXXAD，或者DC1等等名稱。修改完畢重開機。
:::
![image](https://hackmd.io/_uploads/HJOEfuOzC.png)
![image](https://hackmd.io/_uploads/SJxbQu_fA.png)

:::info
接下來增加AD網域功能這個角色給我們的測試主機。升級完成以後，需要令其成為DC, 也就是網域控制站，其有資格成為AD，也就是說AD可以移轉到其他DC上，不過目前不會多提及。

步驟中有一個「DNS選項」，那個不用勾選，也不能勾選，因為你是新建的網域，所以你並沒有所謂的父系，所以DNS功能他會自己幫你新增，所以看到警告不要緊張。接下來下一步直至完成。
:::

![image](https://hackmd.io/_uploads/rk4lNd_G0.png)

![image](https://hackmd.io/_uploads/rkyzBOOM0.png)

![image](https://hackmd.io/_uploads/S14Nr_dfC.png)

![image](https://hackmd.io/_uploads/B1vISdOMR.png)

![image](https://hackmd.io/_uploads/B1VvHdOGA.png)

![image](https://hackmd.io/_uploads/ByuUId_zR.png)

:::info
安裝完成後，可以看到這個登入介面，網域也建好了，未來機器要加入網域，請直接將該主機DNS指向這台測試機。
:::

![image](https://hackmd.io/_uploads/ry8Esduz0.png)

![image](https://hackmd.io/_uploads/Bk_Houuz0.png)

###### tags: `IP` `Windows Server` `Windows AD`
