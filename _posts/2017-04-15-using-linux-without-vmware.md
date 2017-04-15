---
layout: post
title: Sử dụng linux mà không cần tới vmware
categories: linux
---

Mình không thích dùng ubuntu dù đã có một thời gian dài dùng nó. Nó rất ngốn pin, thiếu thốn
rất nhiều soft như office, EDA ... và giao diện không thân thiện với mình. <br>
Cài máy ảo là một giải pháp hay nhưng thật sự nó rất chậm. Trong bài viết này mình sẽ nói về việc dùng 
`shell để remote control` máy ảo linux và `mapping file system` của nó lên window để code. <br>

# 1.Chuẩn bị
* [VMware workstation 12](https://tinhte.vn/threads/download-vmware-workstation-12-pro-full-key-phan-mem-tao-may-ao-tot-nhat-2015.2507243/)
* [Ubuntu 16.04](https://www.ubuntu.com/download/desktop)
* [Xshell 5](https://www.netsarang.com/products/xsh_overview.html)

# 2.Cài đặt
## 2.1.Setup máy ảo 

Tạo một máy ảo ubuntu 16.04 trên vmware. Ở đây mình đặt linux username là `vietht` và tên máy là ubuntu. Sau khi cài đặt máy ảo ta boot lên 
và cài đặt một số tool cần thiết như ssh, samba hay vim.

|tên|chức năng|
|---|---------|
|ssh|remote access daemon trên linux tương tự như teamview nhưng access tới terminal|
|samba|file sharing daemon dùng để share file từ ubuntu sang window|
|vim|text editor|

````
sudo apt-get update
sudo apt-get install ssh
sudo apt-get install samba
sudo apt-get install vim
````

## 2.2.Remote access tới máy ảo ubuntu 
Ở bước này ta sẽ access tới máy ảo dùng ssh ubuntu, máy ảo phải được config network ở mode NAT hoặc bridge. Ta cần lấy địa chỉ ip 
của máy ảo. Có 2 interface lo là card loopback có địa chỉ localhost nên ta lấy địa chỉ của interface ens33.

````
vietht@ubuntu:~$ ifconfig
ens33     Link encap:Ethernet  HWaddr 00:0c:29:bd:43:11  
          inet addr:192.168.110.130  Bcast:192.168.110.255  Mask:255.255.255.0
          inet6 addr: fe80::3c95:ce73:154d:26fc/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:4674 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1904 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:6301769 (6.3 MB)  TX bytes:129500 (129.5 KB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:273 errors:0 dropped:0 overruns:0 frame:0
          TX packets:273 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:21996 (21.9 KB)  TX bytes:21996 (21.9 KB)
````
Địa chỉ ip của máy ảo là 192.168.110.130. Ta mở xshell 5 lên và ssh vào địa chỉ này. Hãy chắc chắn rằng bạn ping thông giữa
ubuntu và window.

````
ssh vietht@192.168.110.130
````

> Đôi khi ta không thể ssh tới địa chỉ này và xshell thì đơ luôn. Chỉ cần mở máy ảo và restart ssh daemon.
````
sudo service ssh restart
````
hoặc add rules để accept tất cả các traffic đến máy ảo.
````
iptables -I INPUT -j ACCEPT
```` 

![]({{ site.url }}/images/xshell.png){: .center-image }

## 2.3.Share file giữa máy ảo và window 
Bây giờ ta sẽ setup samba để share file system giữa window và ubuntu, bước này nhằm mục đích cho phép ta dễ dàng chỉnh sửa file system của linux trên window. <br>
Thêm user `vietht` vào samba và setup password cho nó.
````
vietht@ubuntu:~$ sudo smbpasswd -a vietht
New SMB password:
Retype new SMB password:
Added user vietht.
````
Kế đến mở `smb.conf` và thêm config cho share folder. Trong trường hợp này ta sẽ share toàn bộ dữ liệu của user vietht

````
sudo vim /etc/samba/smb.conf
````
Thêm vào config vào cuối của file các option rất rõ ràng và dễ hiểu.
````
[vietht]
path = /home/vietht
browseable = yes
read only = no
````

Bây giờ thì restart smbd daemon

> daemon là tiến trình chạy ngầm, không có terminal để control

````
service smbd restart
````
Kế đến là cài đặt mapping network folder trên window. Click vào **Map network drive** <br>

![]({{ site.url }}/images/mapdrive.png){: .center-image }


và điền đường dẫn theo format `\\ipaddress\vietht` với vietht là trường `[vietht]` trong `smb.conf` file


![]({{ site.url }}/images/address.png){: .center-image }


Kế đến ta điền samba username và password đã tạo trước đó cho credential manage của windows để xác thực user. Cuối cùng ta đã có toàn bộ folder của user vietht tại ổ Y.


![]({{ site.url }}/images/foldertree.png){: .center-image }


# 3.Kết luận
Done :D sau khi setup ssh để remote access và samba để share file. Ta đã có toàn quyền control máy ảo ubuntu trên window mà không cần đụng vào VMware. Ta có thể set nó `run on background`.