mininet官网上的使用python自定义拓扑实例：

>     """Custom topology example
>     
>     Two directly connected switches plus a host for each switch:
>     
>        host --- switch --- switch --- host
>     
>     Adding the 'topos' dict with a key/value pair to generate our newly defined
>     topology enables one to pass in '--topo=mytopo' from the command line.
>     """
>     
>     from mininet.topo import Topo
>     
>     class MyTopo( Topo ):
>         "Simple topology example."
> 
>         def __init__( self ):
>             "Create custom topo."
> 
>             # Initialize topology
>             Topo.__init__( self )
> 
>             # Add hosts and switches
>             leftHost = self.addHost( 'h1' )
>             rightHost = self.addHost( 'h2' )
>             leftSwitch = self.addSwitch( 's3' )
>             rightSwitch = self.addSwitch( 's4' )
> 
>             # Add links
>             self.addLink( leftHost, leftSwitch )
>             self.addLink( leftSwitch, rightSwitch )
>             self.addLink( rightSwitch, rightHost )
> 
> 
>     topos = { 'mytopo': ( lambda: MyTopo() ) }

需要使用--custom命令执行自定义的拓扑，如下：

>     $ sudo mn --custom ~/mininet/custom/topo-2sw-2host.py --topo mytopo --test pingall
