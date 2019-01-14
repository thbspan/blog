# 设置dns

## 使用全新的命令行工具nmcli

### nmcli connection show
    NAME    UUID                                  TYPE      DEVICE 
    ens33   4a98d9ca-a34b-403e-ab29-b9cbd4657173  ethernet  ens33  
    virbr0  f612e2ed-6ded-4efd-b80d-9e67e6ecf2f7  bridge    virbr0 

### 修改当前网络连接对应的DNS服务器，这里的网络连接可以用名称或者UUID来标识
    nmcli con mod [name|UUID] ipv4.dns "114.114.114.114 8.8.8.8"

