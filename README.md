# guide_eap_proxy
Using the files from https://github.com/jaysoffian/eap_proxy these are the instructions to get a fully functional bypass using a EdgeRouter Lite and AT&amp;T router.  Searching multiple places I could only find snippets and partial code.  After piecing these together I got a working config.  Kudos to https://www.reddit.com/user/SScorpio/ that provided most if not all of the script from https://www.reddit.com/r/uverse/comments/9ii4t5/eli5_how_to_use_eaproxy_and_att_uverse/e9fbtoa?utm_source=share&utm_medium=web2x

I've used this only with a EdgeRouter Lite (ERL), but probably would work with most EdgeOS routers.

This guide configures the ERL with the following for the ports:
eth0 = WAN (the connection from the ONT)
eth1 = AT&T Router
eth2 = LAN

