参考mininet的examples自定义一个多控制器拓扑:

    from mininet.net import Mininet
    from mininet.node import RemoteController, OVSSwitch
    from mininet.cli import CLI
    from mininet.log import setLogLevel
    
    def multiControllerNet():
        "Create a network from semi-scratch with multiple controllers."

        net = Mininet( controller=RemoteController, switch=OVSSwitch )

        print "*** Creating (reference) controllers"
        c1 = net.addController( 'c1', controller=RemoteController, ip='192.168.37.130', port=6653 )
        c2 = net.addController( 'c2', contrlooer=RemoteController, ip='192.168.37.130', port=6753 )

        print "*** Creating switches"
        s1 = net.addSwitch( 's1' )
        s2 = net.addSwitch( 's2' )

        print "*** Creating hosts"
        hosts1 = [ net.addHost( 'h%d' % n, ip='10.0.0.%d' % n, mac='00:00:00:00:00:0%d' % n ) for n in 1, 2 ]
        hosts2 = [ net.addHost( 'h%d' % n, ip='10.0.0.%d' % n, mac='00:00:00:00:00:0%d' % n ) for n in 3, 4 ]

        print "*** Creating links"
        for h in hosts1:
            net.addLink( s1, h )
        for h in hosts2:
            net.addLink( s2, h )
        net.addLink( s1, s2 )

        print "*** Starting network"
        net.build()
        c1.start()
        c2.start()
        s1.start( [ c1 ] )
        s2.start( [ c2 ] )

        print "*** Testing network"
        net.pingAll()

        print "*** Running CLI"
        CLI( net )

        print "*** Stopping network"
        net.stop()
    
    if __name__ == '__main__':
        setLogLevel( 'info' )  # for CLI output
        multiControllerNet()
