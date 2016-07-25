see iptables rules here:

    cat /etc/iptables/rules.v4

use command like this to add rules(also refers to rules.v4):

    iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

to save what has done:

    iptables-save > /etc/iptables/rules.v4



`iptables -L -n` 

`iptables -D INPUT 3`  //删除input的第3条规则  
  
`iptables -t nat -D POSTROUTING 1`  //删除nat表中postrouting的第一条规则  
  
`iptables -F INPUT`   //清空 filter表INPUT所有规则  
  
`iptables -F`    //清空所有规则  
  
`iptables -t nat -F POSTROUTING`   //清空nat表POSTROUTING所有规则  


`iptables -P INPUT DROP`  //设置filter表INPUT默认规则是 DROP  

`iptables -R INPUT 3 -j DROP`    //将规则3改成DROP  

`iptables -t nat -vnL POSTROUTING --line-number`      //查看nat

`iptables -I INPUT 3 -p tcp -m tcp --dport 20 -j ACCEPT`  //-A至尾部，-I指定位置