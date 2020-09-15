Title: Running CoreDNS for fancy DNS records like SSHFP
Date: 2020-09-14
Category: Writeup

This article will not be going into [sshfp records](https://en.wikipedia.org/wiki/SSHFP_record) themselves but feature how to use them if your DNS vendor's server doesn't support them. I was in this situation with my dns registrar, Namecheap.

I was reccomended [CoreDNS](https://coredns.io/) as a simple to use, docker-deployment-ready, production DNS server. Its pretty quick to follow their setup guide and have some domains resolve with `dig`. An important gotcha: openssh requires the records to be signed under DNSSEC. This is fixed by adding the dnssec plugin into your `Corefile` and generating the applicable keys. `sudo dnssec-keygen -a ECDSAP256SHA256 domain.tld`


In the end my config looked like:

Corefile:

	:::nginx
	jacklaxson.com {
    	    file db.jacklaxson.com
        	log
        	prometheus 0.0.0.0:9153
        	any
	}

the important (and not sensitive bits) of my zonefile:
	
	$ORIGIN jacklaxson.com.
	@	3600 IN	SOA ns.jacklaxson.com. admin.jacklaxson.com. (
					20200831 ; serial
					7200       ; refresh (2 hours)
					3600       ; retry (1 hour)
					1209600    ; expire (2 weeks)
					3600       ; minimum (1 hour)
					)
	server.jacklaxson.com IN SSHFP 4 1 43fda19ed10224240a53193635df39456956fcd370dba277e6b69
	server.jacklaxson.com IN SSHFP 4 2 8d798f349510224240a53193635df39456956fcd370dba277e6b69cab5e1628e9



 my docker run command: `sudo docker run -it -p 127.0.0.1:9153:9153 -p 53:53/tcp -p 53:53/udp -v 
$PWD/db.jacklaxson.com:/db.jacklaxson.com -v $PWD/Corefile:/Corefile -v $PWD/Kjacklaxson.key:/Kjacklaxson.key -v 
$PWD/Kjacklaxson.private:/Kjacklaxson.private --restart=unless-stopped --name coredns coredns/coredns`

