VLAN中删除端口：

    system-view
    interface GigabitEthernet 0/0/1    
    undo port default vlan
    #or undo as following,depends on link type
    #undo port hybird vlan
    #undo port trunk vlan

删除VLAN：

    undo vlan 1