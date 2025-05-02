<h1>MPLS-TE Studies:</h1>

<h4>1.5 MPLS traffic engineering</h4>
<h4>1.5.a IS-IS and OSPF extensions</h4>
<h4>1.5.b RSVP-TE</h4>
<h4>1.5.c MPLS TE policy enforcement</h4>
<h4>1.5.d MPLS LSP attributes</h4>
<h4>1.5.e Optimize MPLS TE scale and performance</h4>
<h4>2.8.e MPLS TE QoS (MAM, RDM and PBTS)</h4>
<h4>4.2.f MPLS TE FRR</h4>

<h1>IOS-XE configuration and tunings (csr1kv 16.9.7):</h1>

mpls traffic-eng tunnels #Enable global command for TE.

router ospf 1
 mpls traffic-eng router-id Loopback0 #Define the RID under the IGP process.
 mpls traffic-eng area 0 #Define the area for the Opaque Area LSAs.

interface GigabitEthernet1
 mpls traffic-eng tunnels #Enable TE per IGP interface.

#After all this you shold have a MPLS TE Topology with all links and attributes.

interface Tunnel0 #Configuring the tunnel it self.
 ip unnumbered Loopback0
 tunnel mode mpls traffic-eng
 tunnel destination 10.1.1.1
 tunnel mpls traffic-eng autoroute announce #Install a IGP like route, but does not advertise.
 tunnel mpls traffic-eng path-option 1 dynamic #Computes a path based on TE Metric and affinity 0x0 (default).

#Tunnel stiching, PE-P or P-PE or P-P tunnels creating a LSP that goes inside "broken" tunnels.

interface Tunnel0
 mpls ip #Enable MPLS LDP signalling inside the tunnel for Label distribution.
 tunnel mpls traffic-eng autoroute announce #Should be used only if the tunnel destination is the end of the LSP.

ip route x.x.x.x x.x.x.x Tunnel0 #Should be used if the tunnel destination is not the end of the LSP.


IOS-XR configuration and tunings (xrv9k-full 7.6.1):

router ospf 1
 area 0
  mpls traffic-eng #Define the area for the Opaque Area LSAs.
 mpls traffic-eng router-id Loopback0 #Define the RID under the IGP process.

mpls traffic-eng
 interface GigabitEthernet0/0/0/0 #Enable TE per IGP interface.
 ds-te mode ietf
 ds-te bc-model mam
 ds-te te-classes
  te-class 0 class-type 0 priority 0
  te-class 1 class-type 1 priority 0

mpls ldp #Needs to be enabled if using a VPN service on top of the TE, an L3VPN vrf will add no out label and will not work.

#After all this you shold have a MPLS TE Topology with all links and attributes.

interface tunnel-te0 #Configuring the tunnel it self.
 ipv4 unnumbered Loopback0
 autoroute announce #Install a IGP like route, but does not advertise.
 !
 destination 10.6.6.6
 path-option 1 dynamic #Computes a path based on TE Metric and affinity 0x0 (default).

#Tunnel stiching, PE-P or P-PE or P-P tunnels creating a LSP that goes inside "broken" tunnels.

mpls ldp
 address-family ipv4
  discovery targeted-hello accept #XR needs to accept hellos, XE does by default.
 interface tunnel-te0 #Enable MPLS LDP signalling inside the tunnel for Label distribution.

interface tunnel-te0
 autoroute announce #Should be used only if the tunnel destination is the end of the LSP.

router static
 address-family ipv4 unicast
  x.x.x.x/x tunnel-te0 #Should be used if the tunnel destination is not the end of the LSP.

#FRR with BFD

mpls traffic-eng
 interface GigabitEthernet0/0/0/1.23 #Dot1q interface created so traffic can be dropped changing the vlan.
  bfd fast-detect
  backup-path tunnel-te 0 #Tunnel crated at the PLR for the NHOP
 !
 bfd minimum-interval 100
 bfd multiplier 3