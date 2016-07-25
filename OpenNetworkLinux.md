##ONL##
ONL is a base-level operating system and **only includes example packet forwarding code.** 

###ORC###
Open Route Cache ("ORC") provides the glue code to hook a routing stack (e.g., Quagga) to the underlying packet forwarding hardware. **Currently, only basic IPv4 unicast forwarding.** Support for IPv6, ECMP, multicast, and other advanced features are possible but do not exist at this time.

###OpenNSL###
OpenNSL Port API is the largest set of functions by far. **The OpenNSL API uses small integers to refer to ports. These are generally the physical port numbers on the device.** Thus, it is necessary to know the particulars of the device being updated to address the ports on that device.

###OpenFlow Data Plan Abstraction(OF-DPA)###
The controller maintains a channel with each OpenFlow switch, over which it exchanges OpenFlow protocol messages. At the switches, OpenFlow agents maintain their end of the channel, process received OpenFlow protocol messages and send OpenFlow messages in response to local events.

As OF-DPA maintains all of the state that matches the hardware tables to OpenFlow, the agent is expected to do a relatively straightforward and stateless translation of OpenFlow messages into OF-DPA API calls and vice versa.

**OFDB Layer:** This is the OF-DPA database layer. OFDB is the software database for the flow, group and port tables. It provides APIs to manage these tables. OFDB APIs are invoked by the following layers of OF-DPA:  
**API:** OFDB stores the management state of the system. OFDB APIs are invoked by the API layer for configuration updates like flow addition, deletion, etc.  
**Mapping:** OFDB also stores system status information like port link status, etc. The mapping layer receives these port state updates from the hardware and invokes OFDB APIs to update the port tables in the database.  
**Datapath:** The datapath layer invokes OFDB APIs to traverse through flow tables etc. and perform housekeeping operations like flow aging.

###SAL###
SAL(service abstraction layer) acts like a large registry of services advertised by various modules and binds them to the applications that require them.

When an application, or a consumer, requests a service via a generic API, SAL is responsible for assembling the request by binding producer and consumer into a contract, brokered and serviced by SAL.