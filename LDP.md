<h1>Notas LDP:</h1>

<h4>1.4 Multiprotocol Label Switching</h4>
<h4>1.4.a MPLS forwarding and control plane mechanisms</h4>
<h4>1.4.b LDP</h4>
<h4>1.4.c LDP scale and performance</h4>
<h4>4.2.b LDP convergence</h4>
<h4>5.1.b Routing protocol and LDP authentication and security</h4>

<h1>Conceitos Básicos:</h1>

<h1>Performance e Escalabilidade:</h1>

<h2>IOS-XE IGP synch</h2>

    router ospf 1
     mpls ldp sync

<h2>IOS-XR IGP synch</h2>

    router ospf 1
      mpls ldp sync

- **IGP Synch:** Aumenta o custo IGP de caminhos unnlabaled.

<h2>IOS-XE Session Protection</h2>

    mpls ldp session protection

<h2>IOS-XR Session Protection</h2>
    
    mpls ldp
     session protection

- **Session Protection:** Targeted Hello para RID, deve ser alcançavel pelo IGP.

<h1>Segurança:</h1>

<h2>IOS-XE TTL propagation</h2>

    no mpls ip propagate-ttl forwarded

<h2>IOS-XR TTL propagation</h2>
    
    mpls ip-ttl-propagate disable forwarded

- **TTL propagation:** Não propaga TTL do IP para LDP, núvem LDP invisível para CE.

IOS-XE configuration and tunings (csr1kv 16.9.7):

Methtod 1:
interface GigabitEthernet2
 mpls ip #Only command needed for mpls ldp configuration.
end
Methtod 2:
router isis 1
 mpls ldp sync #IGP best paths must have an LDP adjacency unless it's the only one.
 mpls ldp autoconfig #Auto configuration for any link using this IGP.

IOS-XE verification commands (csr1kv 16.9.7):

show mpls ldp neighbor
show isis mpls ldp #Verify LDP Sync Info.
show mpls ldp bindings #LRIB
show mpls forwarding #LFIB

IOS-XE configuration caveats (csr1kv 16.9.7):

#When you shut down the Loopback interface having the advertise passive-only in ISIS the LDP-ID changes to the higher IP
#it does not come back to the Loopback when you turn it on, making the LDP sessions not come UP even having ldp router-id configured.
#LDP adjacency to IOS-XE works fine but not to other IOS-XE node.

IOS-XR configuration and tunings (xrv9k-full 7.6.1):

router isis 1
 address-family ipv4 unicast
  mpls ldp auto-config
 !
 interface GigabitEthernet0/0/0/2
  address-family ipv4 unicast
   mpls ldp sync #Can only be configured per interface.
  !
 !

mpls ldp #Needs to be enabled globally.
!

IOS-XR configuration caveats (xrv9k-full 7.6.1):

