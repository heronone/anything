You may find following codes in this index */src/main/java/net.floodlightcontroller.jython/JythonDebugInterface.java*

>     String port = configOptions.get("port");
>         if (port != null) {
>             jythonPort = Integer.parseInt(port);
>         }
>         log.debug("Jython port set to {}", jythonPort);
>         
>         JythonServer debug_server = new JythonServer(jythonHost, jythonPort, locals);

so add `net.floodlightcontroller.jython.JythonDebugInterface.port=6755` at the end of *floodlightdefault.properties* you can change the DebugServer port to be 6755 when you start floodlight with *floodlightdefault.properties*.

Also you can change the port whatever you like,not only 6755.
