# This is the YXA README file. YXA is a set of SIP servers written in Erlang.

To install YXA, you need to have Erlang/OTP R11B-2 installed.
                                 ^^^^^^^^^^^^^^^^^

Quick installation guide :

```bash
$ tar zxvf yxa-version.tar.gz
$ mkdir build
$ cd build
$ ../yxa-version/configure && make && make install
```

and then start configuring your new SIP server(s). These steps are
described in more detail below, so keep on reading.

If you run into problems, check the archive of the yxa-devel mailing list
and if the answer to your question is not to be found in the archives then
subscribe to the list (necessary for posting) and ask.

YXA-devel    : https://lists.su.se/mailman/listinfo/yxa-devel
YXA-announce : https://lists.su.se/mailman/listinfo/yxa-announce

(the announce-list is moderated and only gets e-mail when there is a new
snapshot or release available).




## Applications


Applications included are:

### incomingproxy:

Handles registrations and proxies requests. Implements partial ENUM
and LDAP searches. Handles most everything you need to set up SIP for your
domain.

### outgoingproxy:

Special proxy for clients behind NAT. Makes it possible for clients to
receive all requests to them through a persistent TCP connection the client
has set up with the outgoingproxy. You still need an incomingproxy, and you
need to point your outgoingproxy at it through the 'sipproxy' configuration
parameter.

### pstnproxy:

Designed as an authentication frontend to an insecure PSTN gateway.
Supports the expired-draft-only header Remote-Party-Id to signal
what PSTN phone number to use when sending requests to the PSTN
gateway, and, if an LDAP server is configured, looks for a Display
Name to use for calls from the PSTN gateway. Can route calls from
the PSTN either via ENUM or to a preconfigured SIP gateway.

### appserver:

Provides basic forking and CPL script interpretation. If a user has
multiple active contacts, or a CPL script associated with it, incomingproxy
will send requests for that user on to appserver which is a statefull proxy
and can fork the call to all the users registered user agents or according
to the users CPL script.

### eventserver:

Framework for SIP event packages. Dialog stateful UAS.
Comes with default package handlers for the following packages :

*	presence	RFC3856
*	dialog		RFC4235

> NOTE: The eventserver itself is probably working, but the event packages
are still under development and EXPERIMENTAL. Read more about the packages
below, under the topic "SIP Events".

### testserver:

Not useful for anything except regression testing. Answers 404 Not Found
(or some other failure response) to everything, so that one can do
regression tests of the incomingproxy for example by making sure it
routes requests correctly to the testserver.


## Installation


1) Unpack the source

2) Change working directory to where you unpacked the source

3) Optionally, but preferrably - create a sub-directory called 'build'
   or whatever you desire (this does not have to be a sub-directory to
   where you unpacked the source).

4) Change into the 'build' directory you created in step 3.

5) Run ../configure && make && sudo make install
   If you are building source retreived from CVS, you will have to generate
   the "configure" script by running "autoconf" in the source directory first.
   The autoconf version known to work with the current sources is 2.59.

6) Bootstrap your new installation. See "Bootstrapping" below.

7) Configure the YXA applications you want to use (see "Configuration" below).

8) Start the YXA applications you configured in step 7. To start an
   incomingproxy application, execute the incomingproxy script with the
   single argument 'start' :

```bash
$ incomingproxy start
```
   Then check the log files for incomingproxy to see if it says
   'proxy started'. If you haven't set logger_logbase or logger_logdir, the
   three log files incomingproxy.debug, .log and .error will end up in your
   current directory where you started incomingproxy from. If you run into
   problems, and don't see anything in the logfiles (not even in the .debug
   log file), start incomingproxy (or whichever application it is you are
   having problems with) in the debug mode by using -d. -d implies 'start' :

```bash
   $ incomingproxy -d
```

   You will now end up in an Erlang shell 'console', or see more error
   information hopefully telling you why your YXA application does not start.
   To exit the console, hit Ctrl+C twice, or execute the Erlang command
   "halt()." (without the quotes).


Configure options special to YXA :

   --with-erlang	The path to your Erlang/OTP installation's
			'bin' directory.
   --with-mnesiadir	The path to where you want your Mnesia
			database files. Default is "/var/yxa/db".
   --with-sslcertdir	The path to where you keep your SSL certificate
			files (cert.comb).
   --with-local		The Erlang module to use for local customizations.
                        This is a file containing hooks for all sorts of
			things, making it possible to make your own
			extensions to YXA. The default is
                        'local_default.erl', but 'local_su.erl' and
                        'local_kth.erl' are also included in the
                        distribution. Copy 'local_default.erl' to
                        'local_your_domain.erl' and use
			--with-local='local_your_domain' if you want
			to make your own custom local.erl.


## SSL


If you want to use SSL (normally enabled), either put your combined
private and public X.509 key in ssl/cert.comb or edit the ssl.config
in the source directory and do "make sslkey" to get a self-signed
certificate. The certificate password must be "foobar" if you use
"make sslkey". This SSL is only for use by the Erlang distributed operation
system (see below), it has nothing to do with SIP TLS.

If you get an error like "problems making Certificate Request" and
 ... "string too long" ... then you did not change the "req_distinguished_name"
in ssl.config to your hostname.


## Distributed operation


If you want to setup a distributed erlang system, you must have the
same ~/.erlang.cookie file on all nodes. Also, if you have SSL (see SSL topic
above) enabled on one node, you must have it enabled on all the other nodes
as well. Distributed operation includes having the same Mnesia-tables readable
on more than the local node, so you most probably should do this if you run
more than one YXA node.



## Configuration


The configuration file is named <application>.config and can contain
these variables:

common:
-------
logger_logbasename		(default: application name) Create log files
				based on this. If you specify
				/var/log/incomingproxy as logger_logbasename for
				your incomingproxy, it will log to the files
				/var/log/incomingproxy.debug (everything)
				/var/log/incomingproxy.log   (informational)
				/var/log/incomingproxy.error (errors)

logger_logdir			(default: current working directory) If
				logger_logbasename is NOT set, a default logfile
				basename will be created using this parameters
				value, and the running applications name.

sipauth_realm			(default: "") HTTP Auth realm

sipauth_password		(default: "") HTTP Auth internal cookie

sipauth_unauth_classlist	(default: []) Classes that anyone can call

sipauth_challenge_expiration	(default: 30) How many seconds a challenge is
                                valid. Set to 0 to disable check for stale
                                authentication.

databaseservers			(required if using remote databases)
				Erlang node specification of the database
				servers

ldap_server			(default: "" (ie. no LDAP)) LDAP server host name

ldap_username			(default: "") LDAP user name

ldap_password			(default: "") LDAP password

ldap_searchbase			(default: "") LDAP search base

ldap_use_ssl			(default: false) Connect to LDAP server using SSL
				if true.

ldap_connection_query_limit	(default: 500) When we have made this many querys
				using an LDAP handle, open a new one (as soon as
				we are idle).

listenport			(default: 5060) Port that the server
				should listen to

myhostnames			(required) List of IP addresses and/or
				hostnames of the host you run your server on.
				The first entry in the list will be used
				to symbolize this host in SIP packets
				(Via headers and Record-Route headers (unless you
				set record_route_url)).
				Because of this, the first entry should really be
				a real hostname for this host, and not a NAPTR/SRV
				service name even if those NAPTR/SRV record points
				at the real host. It doesn't hurt if there is also
				NAPTR/SRV records for the real hostname though.

myips				(no default) For some platforms autodetection of
				the IP addresses of the network interfaces doesn't
				work. If you have that problem, list your IP
				addresses in this configuration parameter.

record_route			(default: false, except for pstnproxy and outgoing-
				proxy where default is true) Enable or disable
				adding of Record-Route header to requests.

record_route_url		(optional) A SIP (or SIPS) URI to use as Record-
				Route, if we are configured to add Record-Route
				headers. This makes it possible to for example
				specifying the name of a cluster of machines in
				the Record-Route headers, or to include port
				number or other parameters if you find that you
				have clients that require it.
				Specifying a port number and a maddr=my-ip-address
				is a way to get as much backwards compatibility as
				possible, but also makes sure noone will try to
				use anything besides UDP when following the Route.

default_max_forwards		(default: 70) The Max-Forwards value we put in
				requests that does not have one. Only change for
				debugging, RFC3261 says this should be 70.

max_max_forwards		(default: 255) Upper limit for Max-Forwards in
				requests we send out. If we receive a request with
				a Max-Forwards greater than this, we will use this
				value instead of the received minus one.

detect_loops			(default: true) Detect looping requests. Leave
				this on.

request_rport			(default: false) Request rport in outgoing
				requests. Useful if you are sending through
				a NAT device.

stateless_challenges		(default: false) Send challenges without state
				if this is true. RFC3261 suggests you should do
				this, but there are problems with doing it...
				See comments in transactionlayer.erl,
				function send_challenge2().

stateless_send_ack_with_backup_plan: (default: true) Bend the stateless proxying
				of request rules in RFC3261 slightly, to not
				fail to proxy ACK of 2xx response because the
				first destination resolved is not available.
				Caveat: This works only for TCP/TLS destinations,
				we currently can't detect that UDP destinations
				are not available (we do check for blacklisted
				destinations though).

tcp_connection_idle_timeout	(default: 300) Number of seconds of inactivity
				before we close a TCP connection.

tcp_connect_timeout		(default: 20) Number of seconds before we time
				out trying to establish a TCP connection to a
				remote host.

udp_max_datagram_size		(default: 1200) Max size of messages before we
				switch from TCP to UDP. If you have phones that
				only do UDP (and therefor suck), set this to
				for example 3000.

userdb_modules			(default: [sipuserdb_mnesia]) User database
				backend(s) to use. See "User database" section
				below.

enable_v6			(default: false) Enables IPv6 support. Read the
				IPv6 paragraph below before enabling this!

max_logfile_size		(default: 262144000 (250 MB)) Max filesize (in
				bytes) before the logfiles are rotated. This
				option will probably go away when we invent a
				controlling mechanism so that things like
				logfile size can be checked outside of YXA
				instead, but this is needed for now since Erlang
				crashes when the logfiles reach 2 GB. We only
				check if the file size exceeds this limit every
				60 seconds, not after every write. Set to 0 to
				disable rotation (at your own risk).

ssl_require_client_has_cert	(default: false) Reject connections from TLS
                                clients that don't use a client certificate.

ssl_server_certfile		(no default) Filename of your SSL (TLS) server
                                certificate. SIP proxys are required to support
				TLS.

ssl_server_ssloptions           (default: []) SSL server SSL options, see
				ssl(3erl) (the Erlang SSL documentation) for more
				information, but you should for example be able
                                to for example use a separate file for your
				certificates key by setting this to
                                [{keyfile, "/path/to/cert.key"}].

ssl_client_ssloptions		(default: []) SSL client SSL options, see
				ssl(3erl) (the Erlang SSL documentation) for more
				information, but you should for example be able
				to get the SSL client to verify the certificates
				of servers it connects to by setting this to
				[{cacertfile, "/path/to/root-cert.pem"},
				 {verify, 2}, {depth, 2}].

ssl_check_subject_altname	(default: true) Whether we should reject SSL
				server certificates with subjectAltName/CN not
				matching whatever hostname/domainname we expected.

ssl_check_subject_altname_allow_servername (default: true) Treat NAPTR/SRV host-
				name as valid subjectAltName/CN in certificates.

ssl_require_sips_registration	(default: true) If you leave this at 'true', your
				users phones must register with SIPS URIs to be
				contacted for a request with a SIPS Request-URI.
				If you set it to 'false' then the communication
				between your registrar and your phones are
				considered safe even if it is not protected by
				TLS.

tls_listenport			(default: 5061) The port we should listen for TLS
				connections on.

tls_disable_client		(default: false) Set this to 'true' to make this
				proxy not connect to other proxys using TLS.

tls_disable_server		(default: false) Set this to 'true' to not listen
				for TLS connections at all. Proxys are required
				to support SIP over TLS, so setting this to
				'true' is NOT recommended.

timerT1				(default: 500) See RFC 3261. You should
				probably not touch this.

timerT2				(default: 4000) See RFC 3261. You should
				probably not touch this.

timerT4				(default: 5000) See RFC 3261. You should
				probably not touch this.

sipsocket_blacklisting		(default: true) Should we do blacklisting of
				unreachable/unresponsive destinations so that we
				avoid them for some time?

sipsocket_blacklist_duration	(default: 120) The amount of time (measured in
				seconds) that we should blacklist an unresponsive
				destination.

sipsocket_blacklist_max		(default: 3600) An upper limit of how long we
				allow entrys to reside in our blacklist. This is
				needed because the time could come from a
				Retry-After header in a 503 response.

sipsocket_blacklist_probe_delay	(default: 60) After how long time of blacklisting
				should we start a background probe to see if the
				destination has became reachable again? Set this
				to something greater than
				sipsocket_blacklist_duration if you want black-
				listing but not probes. You should probably keep
				this value at less than
				sipsocket_blacklist_duration minus 32 seconds
				(64*T1) to allow probes to time out before the
				blacklisting expires, if the destination is still
				unavailable/unresponsive.

stun_demuxing_on_sip_ports	(default: false (except for outgoingproxy))
				Should we demux and process STUN requests received
				on our SIP ports? This is becoming a popular way
				for clients to keep NAT bindings alive. See
				draft-ietf-sip-outbound-03 for more information.

include_server_info_in_responses (default: true) Add a Server: header to
                                responses we create, to ease debugging/SIP call
                                tracing through a number of proxies.

pstnproxy:
----------
e164_to_pstn			(no default) Regexps for choosing PSTN gateway
				to use for E.164 numbers.

number_to_pstn			(no default) The same as e164_to_pstn, but
				consulted if a number could not be rewritten
				to an E.164 number (like an internal-only
				number).

pstngatewaynames		(required) List of IP addresses and/or
				hostnames of your PSTN gateway.
				Incoming requests will be matched against
				this list when checking if they are from the
				PSTN gateway or not.

default_pstngateway		(no default) Name of gateway to which we should
				route requests to PSTN if a lookup of the number
                                in e164_to_pstn does not result in an address.

classdefs			(default: [{"", unknown}]) Regexps for
				classifying phone numbers. Used to determine if
				a call from a user (or unknown party) should be
				permitted or not.

sipproxy			(default: "") The name of the SIP proxy to route
                                calls from the PSTN gateway to. This is something
                                like a default SIP route. If you enable ENUM in
                                the pstnproxy, ENUM is considered before
                                sipproxy. If you do not enter a sipproxy, and
				the number being called is not found using ENUM,
				503 Service Unavailable will be returned.

enum_domainlist			(no default) List of domains in which to look
				for NAPTR-records for requests _from_ the
				PSTN gateway. If you use this, you probably
				also want to configure internal_to_e164. The
				global E.164 root is e164.arpa, so an example
				would be ["e164.arpa", "e164.example.com"].
				All domains will be queried and after that
				the answers will be sorted and the best one
				will be used.

internal_to_e164		(default: []) Regexps for rewriting
				internal numbers to E.164 numbers. In pstnproxy,
				this is only used to query ENUM - NOT for
				rewriting stuff to E.164 before passing them
				to the PSTN gateway.

remote_party_id			(default: false) If you set this to true, then
				the pstnproxy will try to add information
				about the caller in a header called
				Remote-Party-Id. This is not a standard, but
				current versions of Cisco IOS (12.2.15T)
				supports it so it is nice to have. If you
				also configure LDAP for the pstnproxy, this
				function will try to look up a name to use
				in calls from the PSTN gateway, but I haven't
				seen a phone that displays this yet.

pstnproxy_no_sip_dst_code	(default: 480) SIP error code to return for
				requests from PSTN to SIP when the destination is
				not found in ENUM, and no sipproxy has been
				configured. This is configurable since you might
				want your gateway to act in a special way in this
				situation, for example falling back to using PSTN
				or something.

pstnproxy_redirect_on_enum	(default: false) If you set this to true, we will
				send a redirect (302) back to the caller when we
				find a destination for the request in ENUM,
				instead of proxying to the destination (only
				applys to calls from PSTN to SIP).

x_yxa_peer_auth_secret		(no default, not required) Shared secret for this
				pstnproxy to use for allowing requests forwarded
				to PSTN on behalf of a particular user using
				appserver.

allowed_request_methods		(default: ["INVITE", "ACK", "PRACK", "CANCEL",
					   "BYE", "OPTIONS"]) What SIP requests
				we should allow to reach our PSTN gateways.

pstnproxy_challenge_bye_to_pstn_dst (default: true) Should we challenge BYEs sent
				towards a PSTN gateway? 

> NOTE: This does NOT work if you have 'free' classes, since we are not dialog stateful and therefor can't know that the BYE is sent to a 'free' destination.

pstnproxy_allow_reinvite_to_pstn_dst (default: true) Should re-INVITEs on
				existing dialogs be allowed?

incomingproxy:
--------------
internal_to_e164		(default: []) Regexps for rewriting
				internal numbers to E.164 numbers.

e164_to_pstn			(default: []) Regexps for choosing PSTN gateway/
				pstnproxy to use for E.164 numbers.

number_to_pstn			(default: []) The same as e164_to_pstn, but
				consulted if a number could not be rewritten to
				a E.164 number (like an internal-only number).

defaultroute			(default: "") SIP host name part of default
				route. 

> Note that if you use e164_to_pstn, you don't have to point defaultroute at your PSTN gateway.

homedomain			(required) List of SIP domains this proxy
				should handle requests for.

enum_domainlist			(no default) List of domains in which to look
				for NAPTR-records for requests that are not
				to SIP-users but where the user part matches
				one of the regexps in internal_to_e164. The
				global E.164 root is e164.arpa, so an example
				would be ["e164.arpa", "e164.example.com"].
				All domains will be queried and after that
				the answers will be sorted and the best one
				will be used.

max_register_time		(default: 43200) If registers have an 'expire'
				of more than this many seconds, use this
				value instead.

always_verify_homedomain_user	(default: true) If From: matches our definition
				of 'our' domains (matched by homedomain), then
				challenge user and make sure it is not someone
				impersonating the calling party.

authenticate_in_dialog_requests (default: false) Should we perform the From:
                                check above for all in-dialog requests, or only
                                the first one that establishes the request?
                                Default is to only verify the first one.

appserver			(default: "") URL to where to send requests
				when there are more than a single location
				for a request found in the location database,
				or when the request is destined for a user who
				has a CPL script.
				Example: "appserver.example.com" or
					 "sip:sipserver.example.com:5090".

eventserver			(default: "") URL to where to send SUBSCRIBE
                                and PUBLISH requests. Point this at your
                                eventserver (another YXA application) or some
                                other server that handles RFC3265 SIP events
                                in your domain.
				Example: "eventserver.example.com" or
					 "sip:sipserver.example.com:5040".

eventserver_for_package         (default: []) More fine grained control over
                                which server should handle a specific type of
                                events in your domain. Checked before the
                                'eventserver' configuration parameter.
                                Example:
                                  [{"presence", "sip:presence.example.com"},
                                   {"ua-config", "sip:prov.example.com"}
                                  ]

outgoingproxy:
--------------

sipproxy			(required) The location of your incomingproxy.
				Example: "incomingproxy.example.com" or
					 "sip:incomingproxy.example.com:5090".

max_register_time		(default: 43200) If registers have an 'expire'
				of more than this many seconds, use this
				value instead.

homedomain			(required) List of SIP domains this proxy
				should handle requests for.

allow_foreign_registers 	(default: false) Set to true to have your out-
				goingproxy forward non-local REGISTERs to other
				proxys. Defaults to false since it doesn't make
				much sense for the clients to use the outgoing-
				proxy in the first place if they register with
				a domain that the outgoingproxy doesn't consider
				local (what domains are 'local' is determined
				by the 'homedomain' parameter).

stun_demuxing_on_sip_ports	(default: true) Allow clients to keep NAT
				bindings alive by (quite frequently) send STUN
				requests to the outgoingproxy on the same socket
				used for SIP signalling. See
				draft-ietf-sip-outbound-03 for more information.

appserver:
----------
internal_to_e164		(default: []) Regexps for rewriting
				internal numbers to E.164 numbers. The
				appserver must have identical
				internal_to_e164 with the incomingproxy
				to be able to make the exact same routing
				decisions.

appserver_call_timeout		(default: 40) The number of seconds to ring
				a destination before concluding that noone
				answers.

appserver_forward_timeout	(default: 40) The same as appserver_call_timeout
				but for forwarded calls.

x_yxa_peer_auth			(default: []) List of {host, secret} shared
				secrets to use when sending out requests on
				behalf of a user with forwards or a CPL script.
				Example : [{"pstnproxy.example.com", "secret"}]}.
				See also x_yxa_peer_auth_secret to make pstnproxy
				at pstnproxy.example.com accept the requests.


eventserver:
------------
eventserver_package_handlers	(default: [{"presence", presence_package}])

presence_min_publish_time	(default: 5)

presence_max_publish_time	(default: 3600)

presence_default_publish_time	(default: 600)


At the moment, configuration files must be in Erlang syntax. This involves lots
of brackets and curly brackets, and might be seen as an disadvantage for YXA.
Please bear with this though, as steps have been taken to make it easy to add
other configuration parsing backends. An ini-file format is likely to be written
shortly, and it would be fairly easy to write for example an XML-format parser
although you can't count on me doing it for you.

It is possible to include other files from within the top-level configuration
file (ie. you can't include a file from an included file). You do this by
specifying {include, "filename.conf"} as a parameter where you want the contents
of "filename.conf" to be included. The contents of filename.conf should not
contain any sections - see the example below. "filename.conf" will be considered
relative to the filename of the top-level configuration file, unless it is
absolute.

Brief introduction to the Erlang syntax :

{}  - Curly brackets define a tuple. A tuple is like a list of fixed length, for
      example {Key, Value} or {Key, Value1, Value2}.
[]  - Brackets enclose lists. If a parameter can have any number of values
      (like 'myhostnames' for example), the values will be in a list. Example :
        {homedomain, ["example.com", "example.net", "example.org"]}
      or, if you have just one domain
        {homedomain, ["example.com"]}.



## Application configuration examples


incomingproxy.config:
---------------------

```erlang
[{incomingproxy, [{sipauth_realm, "example.com"},
                  {sipauth_password, "secret"},
                  {defaultroute, "other-sipserver.example.com"},
                  {logger_logbasename, "/var/log/incomingproxy"},
                  {internal_to_e164, [{"^00(.+)$", "+\\1"},
                                      {"^0(.+)$", "+46\\1"},
                                      {"^(.+)$", "+468\\1"}
                                     ]},
		  {e164_to_pstn, [{"^\\+468([1-9][0-9]+)$", "00\\1@pstnproxy.example.com"},
				  {"^\\+46([1-9][0-9]+)$", "000\\1@pstnproxy.example.com"},
				  {"^\\+44([1-9][0-9]+)$", "000\\1@pstnproxy-uk.example.com"},
				  {"^\\+([0-9]+)$", "0000\\1@pstnproxy.example.com"}
                                 ]},
		  {number_to_pstn, [{"^118$", "118@pstnproxy.example.com"}]},
		  {enum_domainlist, ["e164.arpa"]},
                  {homedomain, ["example.com", "example.net", "example.org"]},
		  {myhostnames, ["sipserver.example.com", "sip.example.net", "sip.example.org"]},
		  {appserver, "appserver.example.com:5070"},
		  {eventserver, "eventserver.example.com"},
                  {ldap_server, "ldap.example.com"}
                 ]}].
```

outgoingproxy.config:
---------------------

```erlang
[{outgoingproxy, [{sipauth_realm, "example.com"},
                  {sipauth_password, "secret"},
                  {databaseservers, ['incomingproxy@sip-incoming.example.com']},
		  {myhostnames, ["outgoingproxy1.example.com", "out.example.com"]},
                  {homedomain, ["example.com", "example.net", "example.org"]},
		  {tcp_connection_idle_timeout, 1000000}
                 ]}].
```

pstnproxy.config:
-----------------

```erlang
[{pstnproxy, [{sipauth_realm, "example.com"},
              {sipauth_password, "secret"},
              {sipauth_unauth_classlist, [internal]},
	      {logger_logbasename, "/var/log/pstnproxy"},
              {pstngatewaynames, ["pstn-gw.example.com", "10.10.1.1"]},
              {myhostnames, ["sip-pstn.example.com", "sip2pstn.example.com", "10.10.1.2"]},
              {classdefs, [{"^[1-9]", national},
                           {"^0[1-9]", national},
                           {"^00", international},
                           {"", unknown}
                          ]},
              {databaseservers, ['incomingproxy@sip-incoming.example.com']},
              {internal_to_e164, [{"^00(.+)$", "+\\1"},
                                  {"^0(.+)$", "+46\\1"},
                                  {"^(.+)$", "+468\\1"}
                                 ]},
	      {e164_to_pstn, [{"^\\(+44[1-9][0-9]+)$", "\\1@uk-toll-bypass.tsp.example.net"}]},
	      {number_to_pstn, [{"^118$"}, "sip:118@pstn-gw.example.com"]},
	      {enum_domainlist, ["e164.arpa"]},
             ]}].
```

appserver.config:
-----------------

```erlang
[{appserver, [{sipauth_realm, "example.com"},
	      {sipauth_password, "secret"},
	      {logger_logbasename, "/var/log/appserver"},
	      {internal_to_e164, [{"^00(.+)$", "+\\1"},
				  {"^0(.+)$", "+46\\1"},
				  {"^(.+)$", "+468\\1"}
				 ]},
	      {databaseservers, ['incomingproxy@sip-incoming.example.com']},
	      {myhostnames, ["appserver.example.com", "10.10.1.1"]},
	      {record_route, true},
	      {listenport, 5070}
             ]}].
```

Combined configuration file for more than one application :
-----------------------------------------------------------

yxa.config :

```erlang
[
 %% Include files
 %% --------------------------------------------------------------------
 {include,	"secrets.config"},

 %% Common configuration
 %% --------------------------------------------------------------------
 {common,	[{sipauth_realm,		"example.com"},

		 {logger_logdir,		"/var/log/yxa"},

		 {databaseservers,		['incomingproxy@server.example.com']},

		 {ssl_server_certfile,		"/var/yxa/ssl/cert.comb"},
		 {ssl_client_certfile,		"/var/yxa/ssl/cert.comb"},
		 {ssl_server_ssloptions,	[{verify, 1},
						 {cacertfile, "/var/yxa/ssl/ca-chain.crt"}
						]},
		 {ssl_client_ssloptions,	[{verify, 1},
						 {cacertfile, "/var/yxa/ssl/ca-chain.crt"}
						]}
		]},

 %% Application specific configuration
 %% --------------------------------------------------------------------
 {incomingproxy,	[{appserver,		"appserver.server.example.com"},
			 
			 {myhostnames,		["incomingproxy.server.example.com", "server.example.com"]},
			 {homedomain,		["example.com", "example.net", "example.org"],
			 
			 {enum_domainlist,	["e164.arpa"]}
			]},
 
 {appserver,		[{listenport,		5070},
			 {tls_listenport, 	5071}
	 		]}
].

secrets.config :

[
 {common, [{sipauth_password,		"secret"}
	  ]}
].
```


## User database (sipuserdb_file) example

```erlang
[
 {user, [
	 {name, "ft.sip1"},
	 {password, "secret"},
	 {classes, [internal,national,mobile]}
	]},
 {user, [
	 {name, "foo@example.org"},
	 {password, "secret2"},
	 {classes, [internal]},
	 {addresses, ["sip:info@example.org", "sip:all@example.org"]}
	]},

 {address, [
	    {user, "ft.sip1"},
	    {address, "sip:ft@example.org"}
	   ]},
 {address, [
	    {user, "ft.sip1"},
	    {address, "sip:all@example.org"}
	   ]}

].
```

This is a simple user database. If I have two phones, and let one REGISTER
with 'To: sip:ft@example.org' and authentication username 'ft.sip1', and
the other REGISTER with 'To: sip:info@example.org' and authentication
username 'foo@example.org' (NOTE: no "sip:"), calls to sip:ft@example.org
will cause phone #1 to ring and calls to sip:info@example.org will cause
phone #2 to ring. Calls to sip:all@example.org will cause both phones to
ring, provided that I have an incomingproxy and an appserver set up and
configured. Note that addresses can either be supplied as part of the
'user' declaration (user 'foo@example.org'), or separately (user 'ft.sip1').



## User database


YXA has a modular interface for user database backends. The entry point is
the sipuserdb module, which looks at your configuration to determine which
database backends to query, and in what order. The default is to just look
in the Mnesia database (tables user and numbers), but there is also an
LDAP module, MySQL module and a plain text file backend available. The
schema for storing SIP user information in LDAP is not yet finished, and
is currently not even included in the YXA source, but the sipuserdb_ldap
module is used in production at Stockholm university.

sipuserdb_ldap configuration parameters :
-----------------------------------------

ldap_userattribute		(deafult: sipAuthenticationUser) User
				attribute.
ldap_addressattribute		(default: sipLocalAddress) User SIP address
				attribute.
ldap_telephonenumberattribute	(default: telephoneNumber) User phone number
				attribute.
ldap_passwordattribute		(default: sipPassword) User authentication
				password attribute.

sipuserdb_file configuration parameters :
-----------------------------------------

sipuserdb_file_filename		(required) Filename (including path) of
				your user database. See above (section
				Examples) for format of this file.
sipuserdb_file_refresh_interval	(default: 15) Interval (in seconds) between
				checks for updated user database file.


sipuserdb_mysql configuration parameters :
------------------------------------------

sipuserdb_mysql_host		(required) Server name. Example :
				"mysql-srv.example.org"

sipuserdb_mysql_port		(default: 3306) Server port. Example :
				33060

sipuserdb_mysql_user		(required) MySQL username to log in with.
				Example : "yxa"

sipuserdb_mysql_password	(required) MySQL password to log in with.
				Example : "secret"

sipuserdb_mysql_database	(required) MySQL database to use.
				Example : "yxadb"

MySQL query configuration parameters :
--------------------------------------
The default YXA MySQL userdb querys are simple. They can be somewhat
'personalized' by changing the templates. All querys are made for a single
key (or list of keys, when for example looking for all users matching a
number of addresses), and the query is constructed by inserting the quoted
key value everywhere in the templates where there is a '?'.

If you want to construct more advanced querys than can be done using these
templates, look at creating a custom sipuserdb_mysql_make_sql_statement/2
function in your local.erl.

If you have to do special processing of some return values (for example
decrypting passwords after reading them from the database), override the
get-function you are interested in altering using for example the
sipuserdb_backend_override/3 function in your local.erl.

sipuserdb_mysql_get_user
	(default: "select sipuser from users where sipuser = ?")
	Query to use when trying to validate a users existence. Can also
	be used to map multiple usernames to a single 'primary' username,
	which will be the key used when looking for a users location(s)
	in the location database for example.

sipuserdb_mysql_get_user_for_address
	(default: "select sipuser from addresses where address = ?")

sipuserdb_mysql_get_addresses_for_user
	(default: "select address from addresses where sipuser = ?")

sipuserdb_mysql_get_classes_for_user
	(default: "select class from classes where sipuser = ?")

sipuserdb_mysql_get_password_for_user
	(default: "select password from users where sipuser = ?")

sipuserdb_mysql_get_telephonenumber_for_user
	(default: "select address from addresses where sipuser = ? and
		   is_telnr = 'Y'")

Table definitions matching the default querys :

```sql
	CREATE TABLE users (
	       sipuser     VARCHAR(255) NOT NULL,
	       password    VARCHAR(255) NOT NULL
	       );

	CREATE TABLE addresses (
	       address     VARCHAR(255) NOT NULL,
	       sipuser     VARCHAR(255) NOT NULL,
	       is_telnr    ENUM('Y', 'N') DEFAULT 'N'
	       );

	CREATE TABLE classes (
	       sipuser     VARCHAR(255) NOT NULL,
	       class       VARCHAR(255) NOT NULL
	       );
```


## Bootstrapping


In your build directory, or in the directory you chosed to have the executable
files installed in (default: /usr/local/XXX) you should have a file called
yxa-bootstrap. Execute this script, and it will create a Mnesia schema and all
the needed Mnesia tables for you. By default, the incomingproxy application will
be Mnesia database master. If you do not wish to have some other node be the
Mnesia database master, you should execute yxa-bootstrap on the server that
you will be running incomingproxy on.

Example :

```bash
$ /path/to/yxa-bootstrap
```


You should now have the necessary Mnesia database and tables.

All of the YXA application can run just fine without root permissions.

If you want to have more than one incomingproxy node (and thus more than
one database master node), do the following on incomingproxy server #1 :

   $ /path/to/yxa-bootstrap

and then, on incomingproxy server #2 :

   $ /path/to/yxa-bootstrap -replica incomingproxy@server1.example.org

You should then set the configuration parameter databaseservers like this
on all your non-incomingproxy nodes :

   {databaseservers, ['incomingproxy@server1.example.org',
		      'incomingproxy@server2.example.org']}

If you want some other YXA application node than incomingproxy to be your
Mnesia database master, supply the -name argument to yxa-bootstrap. For
example, if you are only using the pstnproxy, invoke yxa-bootstrap like
this :

   $ /path/to/yxa-bootstrap -name pstnproxy


## Web interface


To use the web interface, you first need to install the Erlang web-server
Yaws version 1.56 or higher (available from http://yaws.hyber.org/).
When you have configured a working Yaws web-server, you need to point your
docroot at the yxa/yaws/docroot directory, and configure Yaws with module
and include search paths to the YXA include and ebin-directorys. Example
parts of yaws.conf configuration file :

  ebin_dir = /path/to/lib/yxa/ebin
  include_dir = /path/to/lib/yxa/include

  <server example.org>
        docroot = /path/to/yxa-source/yaws/docroot
  </server>

> NOTE: All access control must be performed using Yaws methods of access control. Currently, the YXA Yaws SSIs will not perform any kind of authentication or authorization, so you must restrict access to the web interface using Yaws.

> NOTE 2: The files inside yaws/docroot will probably be installed somewhere under /path/to/lib/yxa in the future, I just need to figure out where they belong.

Then you need to start Yaws with "-name yaws" (or some other node name) in
order to enable communication between the Yaws node and your YXA node(s).
You also need to start the Yaws node as the same user as you start your YXA
nodes as, or by other means make sure the Yaws node has the same shared
secret in it's ~/.erlang.cookie file as the YXA nodes has, or they will be
unable to communicate. See the section "Distributed operation" above for
more information about Erlang distribution.

If you use SSL for Erlang distribution (see sections "SSL" and "Distributed
operation" above for more information), you must get Yaws to use SSL for
Erlang distribution too, or the nodes won't be able to talk to each other.
This is a sample Yaws startup command for doing this (only works with Yaws
1.56 or later) :

  yaws -i -name yaws \
    -erlarg "-boot /path/to/lib/yxa/ebin/start_ssl" \
    -erlarg "-proto_dist inet_ssl" \
    -erlarg "-ssl_dist_opt client_certfile /path/to/cert.comb" \
    -erlarg "-ssl_dist_opt server_certfile /path/to/cert.comb" \
    -erlarg "-ssl_dist_opt verify 2"


## IPv6 support


If you enable IPv6 support, you need to add your v6 addresses to the
myhostnames configuration variable, because siphost:myip_list() is currently
not capable of detecting your IPv6 address (Erlang OTP R9C-0). If you don't
do this, then lookup:homedomain() will not work properly and you might get
problems with requests sent with a URI containing your IPv6 address.

IPv6 support is disabled by default, because you need to know what you are
doing before turning it on. If you enable it to make proxy-to-proxy
communication use v6 and a v6-only proxy adds Record-Route headers, then
v4-only clients won't be able to reach the v6-only proxy. This could be
fixed by making YXA always Record-Route when receiving a request over v4/v6
and sending it out using the other but then there is still the problem with
v6-only phones responding to INVITE with a SDP offer containing a v6 address.
How will a v4-only phone be able to send audio to that phone, and receive
audio from that phone? v6 is disabled by default.

If you are using Linux, then you will need to

	# echo 1 > /proc/sys/net/ipv6/bindv6only

since YXA wants to use separate sockets for v6 and v4. IPv6 support has only
been tested on Linux 2.4.x where x >= 20, and Linux 2.6.x where x >= 14.

> Note on Erlang OTP R10B-0 through R10B-6 :
> The resolver order was changed in R10B, so now Erlang primarily tries to use
> the 'native' resolver (meaning a C port driver) for DNS resolution.
> Unfortunately, this C port driver only handles getipnodebyname() and not
> getaddrinfo(). The latter is a POSIX standard, and the one that should be used.
> This means that you might not get IPv6 addresses back from
> dnsutil:get_ip_port(), which in turn means YXA won't make outgoing connections
> to IPv6 addresses. This is a problem with some versions of Linux (for example,
> RH7.3 to RH9 based systems).



## SIP Events (RFC3265)


The YXA application 'eventserver' handles subscriptions for SIP events in a
modular fashion. The goal is to separate subscription handling, with all the
issues regarding expiration and renewing of subscriptions, from the event
packages. Event packages are Erlang modules implementing the behaviour
'event_package'. There following event packages exist, and are enabled per
default :

  presence :    This is a presence state agent (presence server), that stores
                presence that clients send using PUBLISH in a Mnesia database.
                It is possible to have a distributed presence server with more
                than one eventserver, running on more than one Erlang node.
                Currently, all users in your domain are allowed to subscribe
                to all other users in your domains presence. Users outside
                your domain is not allowed to subscribe to your users presence
                at all. The presence status of your users can also be obtained
                through a bidirectional subscription (the eventserver subscri-
                bes to your user when it subscribes to someone else), but this
                is done on a per-User-Agent basis and probably requires you to
                patch the code slightly. Anyone interested in implementing
                XCAP for access control lists etc. should let me know, that
                would be great. I won't do it.

  dialog   :    A dialog state agent. The goal was to get 'shared line'
                working on my Snom phones. Can't say I'm there just yet.

> NOTE that ALL these event packages are currently to be considered experimental.
