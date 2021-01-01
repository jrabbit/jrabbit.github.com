Title: Running CoreDNS for fancy DNS records like SSHFP
Date: 2020-10-05
Category: Writeup 
Tags: ssh, incremental security

This article will not be going into [sshfp records](https://en.wikipedia.org/wiki/SSHFP_record) themselves but feature how to use them if your DNS vendor's server doesn't support them. I was in this situation with my dns registrar, Namecheap.

I was recommended [CoreDNS](https://coredns.io/) as a simple to use, docker-deployment-ready, production DNS server. Its pretty quick to follow their setup guide and have some domains resolve with `dig`. TLDR: `docker pull coredns/coredns` An important gotcha: openssh requires the records to be signed under DNSSEC. This is fixed by adding the dnssec plugin into your `Corefile` and generating the applicable keys. `sudo dnssec-keygen -a ECDSAP256SHA256 domain.tld` Make sure you mount the public key and private key into the container. with a volume bind. (This could be in a docker-compose.yml file if you were so inclined or had associated services.)

Good CoreDNS plugins:

  1.  [any](https://coredns.io/plugins/any/) any prevents creepy scraping requests
  2.  [prometheus](https://coredns.io/plugins/metrics/) (adds support for influxdb scraping) I love [influxdb2](https://www.influxdata.com/products/influxdb-overview/influxdb-2-0/) but that's for another time.

In the end my config looked like:

Corefile:

	
	jacklaxson.com {
			dnssec {
				key file Kjacklaxson
			}
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



 my docker run command: 

`sudo docker run -it -p 127.0.0.1:9153:9153 -p 53:53/tcp -p 53:53/udp -v 
$PWD/db.jacklaxson.com:/db.jacklaxson.com -v $PWD/Corefile:/Corefile -v $PWD/Kjacklaxson.key:/Kjacklaxson.key -v 
$PWD/Kjacklaxson.private:/Kjacklaxson.private --restart=unless-stopped --name coredns coredns/coredns`

The stunning result of our labor:

`ssh -o "VerifyHostKeyDNS ask" -o UserKnownHostsFile=/dev/null server.jacklaxson.com` gives you

	The authenticity of host 'server.jacklaxson.com (26.....)' can't be established.
	ED25519 key fingerprint is SHA256:j35df39456956fcd370dba..277Ok.
	Matching host key fingerprint found in DNS.
	Are you sure you want to continue connecting (yes/no/[fingerprint])?

Tells you about the signature records from DNSSEC! If you have a recent ssh you'll have the fingerprint option so you can paste the fingerprint to accept.
