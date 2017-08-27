# Vagrant

玩雲最常碰到的情境是，開了一堆VM，每一個VM都擁有各自的功能，有些做web portal，有些要拿來做後端，資料庫可能又會另外長，如果功能複雜也會需要有load balancer或者是cache，總之就是會起一堆VM。但要管理一堆VM並不是一件容易的事，若是有專業的hypervisor還好說，但一般人手裡有的資源其實就是一台PC，這時候管理VM通常就是將各個VM的domain設成不一樣，才可以在茫茫VM海裡面一眼就看出來在哪，為了方便管理，還需要修改host的DNS，讓各自的domain能夠對應正確的IP，然後就是滿滿的SSH海。

值得一提的是，在新版的virtualbox\(5.1.x\)已經有提供headless start了，所以茫茫VM海的景象不復存在，但SSH海總還是免不了的。這時候就會希望有一個中控端來扮演類似hypervisor的腳色，那就是本章要介紹的Vagrant。

Vagrant是一套用Ruby寫的管理介面，可以運行在windows, linux和mac上，一開始他管理的guest provider僅僅只有virtualbox，但他進步的很快，現在已經可以支援大部分的VM，VMware這種基本的不說，甚至KVM、QEMU和CentOS的Xen都可以支援。但在本章並不會介紹如何使用vagrant帶起linux上的VM，因為我的工作機是windows PC，而且本人是忠實virtualbox的愛用者，不得不說，五版後的virtualbox改善了很多既有的缺點，包含肥大、反應慢等問題都已經改善許多。

Vagrant的安裝很簡單，去官網\([https://www.vagrantup.com/downloads.html](https://www.vagrantup.com/downloads.html)\)下載對應的安裝包安裝即可。Virtualbox也是必要的，要去官網下載主程式和對應版本的**擴充套件**\(必要\)安裝，若是安裝在非預設路徑，要把VBoxManager.exe的路徑寫入PATH，vagrant的底層其實就是透過VBoxManager在操作virtualbox。若是在windows上使用vagrant，還需要安裝一個cmd下可以執行的ssh client，我個人推薦使用git內建的，而且git現在有portable版本，只需要把PortableGit下的usr/bin放入PATH即可。

如此一來整個環境就設定完畢，接著使用步驟也很簡單，首先先去vagrant提供的VM cloud \([https://app.vagrantup.com/boxes/search](https://app.vagrantup.com/boxes/search)\)上找尋自己想要的VM OS，這在vagrant稱為box，之後對於VM的操作其實都是對box操作。在這邊我以ubuntu 14.04為例子，在cloud上找到ubuntu 14.04，點進去後會有一個New的tab，那就是整個初始化的流程。

```bash
vagrant init ubuntu/trusty64
vagrant up
```

只要這樣下，他就會在當下的目錄初始化一個Vagrantfile，並且將ubuntu 14.04載下來並開啟一個實體VM。若是想要對VM進行操作，只要：

```
vagrant ssh
```

就可以透過SSH連入帶起來的VM。在Vagrantfile內包含了這個VM的所有設定，包含要開多少RAM，要怎麼設定網路，要不要建立shared folder來和host溝通，這些都已經有基本的template在設定檔內部，只需要把註解拿掉即可。問題在於，若是要用同一個box產生複數個VM應該如何？又該怎麼操作？以下是例子。

```
Vagrant.configure("2") do |config|

  config.vm.define "web" do |web|
    web.vm.box = "ubuntu/trusty64"
  end

  config.vm.define "db" do |db|
    db.vm.box = "ubuntu/trusty64"
  end
  
   config.vm.network "public_network"

   config.vm.synced_folder "../data", "/vagrant_data"

   config.vm.provider "virtualbox" do |vb|
     vb.gui = true
  
     vb.memory = "512"
   end
end
```

我把多餘的註解都拔掉，可以看到我這份Vagrantfile設定網路方式是bridge，並且與host的data串聯，然後使用512MB的RAM。重點是，我起了兩個VM，一個叫web一個叫db。當Vagrantfile這樣設定後：

```
vagrant up
```

就可以看到vagrant帶起兩個VM，可以分別進行ssh和其餘的操作，例如：

```
vagrant ssh web
vagrant halt db # 關機
```

透過vagrant可以輕鬆管理大量的VM，並且複製、備份、快照等功能都有，基本上，所有VM上的工作都可以透過簡單的指令完成。好處不僅僅是管理變得方便，當透過簡單指令可以操作VM時，就可以用程式來控制，可以寫python/shell scripts來操作vagrant，來完成unit test或者是自動產生各種實驗環境。想像力就是你的超能力。

