# 電腦安全概論HW3
###### tags: `電腦安全概論
# Project III: Ransomware Propagation and Payload





### Setup

* Setup NAT network
* Restart Network Manager(https://askubuntu.com/questions/1088953/ubuntu-18-04-missing-wired-connections-in-settings)
    * 不知道幹嘛設定成這樣 幹你娘
* Reinstall vmware Tools
    * `sudo apt install open-vm-tools`
    * `sudo apt install open-vm-tools-desktop`
* 加desktop icon
    * https://extensions.gnome.org/extension/1465/desktop-icons/
    * 預設會在置頂要求在firefox安裝插件，安裝好重新整理，按下on/off，安裝 即可。
* Shared Folder
    * https://askubuntu.com/questions/29284/how-do-i-mount-shared-folders-in-ubuntu-using-vmware-tools
    * `sudo vmhgfs-fuse .host:/ /mnt/ -o allow_other -o uid=1000`

### Task I
* Itertools
    * 沒什麼問題，vimtim.dat讀進去排列而已
* paramiko(for ssh connect)
    * 正常連線方式，把code抄一抄(https://www.kite.com/python/answers/how-to-ssh-into-a-server-in-python)
    * 密碼錯誤會跳except，用except pass掉
    * 因為想使用multi thread處理(每次都block在server那邊很慢)，隨之而來的問題就是server好像不能一瞬間承受那麼多的ssh連線(e3其他人也有問)，測試6個以上就不行了，也會跳except(值得一提的是，不知道為什麼這邊的except沒辦法跳到上面描述的except來處理)，保險起見只開四個thread
        * 解法: 出現SSH的exception就把密碼丟回task的queue再試一次，測試過應該是OK，要擁擠個thread都沒問題，另外queue改使用LIFO，不然等到最後在試有點可憐
    * 利用queue，讓每個thread只要跑while迴圈
    * thread採用empty偵測和get_nowait有沒有問題還要再想想
* victim.dat
    * 做了一點更動，用來測試csc2021
* thread數量
    * 使用100個thread
    * 理論上越多thread越好，但畢竟server扛不住的話，造成大量的fail，因為我的寫法，就算找到密碼了也要把剩下的已經在跑的跑完才行，所以測試大概抓個100差不多，可以在50秒有效測試1000組密碼
## 執行檔功能
### crack_attack
* Dictionary attack
    * 產生所有的密碼排列
    * 如果失敗: 換下一個測試 
    * 如果成功: 
        * 刪掉所有的排列
        * 把自己的IP+PORT用echo傳給victim(ADDR.txt)
        * 把VIRUSGENERATOR用echo傳給victim
        * 讓victim執行
### VIRUSGENERATOR
* 壓縮cat
* 紀錄cat、cat.zip大小
* 把ADDR.txt抄下來
* 產生C code(PAYLOAD.c，並且附加一些資訊進去)
* 編譯PAYLOAD.c 取代 cat
* 塞入空白byte、cat.zip、deadbeaf
* 刪除cat.zip VIRUSGENERATOR PAYLOAD.c ADDR.txt
### Attack_server
* 接收到"worm"字串則把worm傳給連線者
### worm
* 加密所有`/home/csc2021/Pictures/*.jpg`的檔案
* print GIVE ME RANSOM字串
### Infected cat
* 讀自己這個cat檔，把後面cat.zip的部分抓出來寫入成檔案，unzip
* 和Attack_server拿worm(傳送worm字串給server)
* 執行worm
* 執行cat
    * 如果被ctrl C或ctrl Z，就直接刪除cat.zip和oricat/cat
* 如果正常執行完也還是要刪除cat.zip和oricat/cat
### 123
* 要額外注意，大多檔案都需要先chmod +x 才能執行

就這樣