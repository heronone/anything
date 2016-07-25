##dlux ui##
`http://localhost:8181/index.html`
default login user & pw are both `admin`

##api doc##
Install the feature first`feature:install odl-apidoc`

Then see api in `localhost:8181/apidoc/explorer/index.html`

To see installed features `feature:list -i`

##others##
to connect to a running ODL instance,use this:`ssh -p 8101 -o UserKnownHostsFile=/dev/null -o StrictHostKeyChecking=no karaf@ipaddress`,the default pw is `karaf`
