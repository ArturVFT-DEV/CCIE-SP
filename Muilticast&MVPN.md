<h1>MVPN Studies:</h1>

<h4>1.3 Multicast</h4>
<h4>1.3.a Design PIM (PIM-SM, PIM-SSM, and BIDIR-PIM)</h4>
<h4>1.3.b Design RP (Auto-RP, BSR, static, anycast RP, and MSDP)</h4>
<h4>1.3.c Design IGMP and MLD</h4>
<h4>1.3.d MLDP</h4>
<h4>1.3.e Tree-SID</h4>
<h4>2.7 Multicast VPN</h4>
<h4>2.7.a NG mVPN (profile 7, 11, 12, 13, 14, 27, 28, and 29)</h4>

<h1>Multicast:</h1>

<h1>Conceitos Básicos:</h1>

Profile 0 (Default MDT/GRE/C-PIM):
#When using ASM groups on the P-PIM no need for MDT address family on BGP because RP is already 
#signaling the *,G and S,G after the configuration of the default mdt for the VRF.

ip multicast-routing distributed #RP and mroutes will only be signalled by using this.
ip multicast-routing vrf Cust distributed #RP and mroutes will only be signalled by using this.

interface Loopback0
 ip address 10.0.0.11 255.255.255.255c
 ip pim sparse-mode #Configure P-PIM on all IGP interfaces.

vrf definition Cust
 address-family ipv4
  mdt default 239.1.1.1 #Multicast group of destination for GRE tunnels.

interface GigabitEthernet1
 vrf forwarding Cust
 ip pim sparse-mode #C-PIM for Signaling.
Profile 0 (Default MDT/GRE/C-PIM) with SSM Groups:

ip pim ssm default

vrf definition Cust
 address-family ipv4
  mdt default 232.1.1.1 #SSM group for GRE tunnels.

router bgp 65000
 address-family ipv4 mdt #AFI 1 SAFI 66 needed to build the SPT to the GRE group address.
 neighbor 10.0.0.1 activate

Profile 0 (Default MDT/GRE/C-PIM) Data MDT:
vrf definition Cust
 address-family ipv4
  mdt data 232.11.11.0 0.0.0.255 threshold 1
  mdt data threshold 1 #New form of the threshold command.

###################################################################################################

Profile 1 (Default MDT/mLDP/C-PIM):
#No need for P-PIM neither BGP MDT SAFI.
vrf definition Cust
 vpn id 1:1 #Opaque value needed.
 address-family ipv4
  mdt default mpls mldp 10.0.0.1 #Root of the mLDP tree.

IOS-XR configuration and tunings (xrv9k-full 7.6.1):

Profile 0 (Default MDT/GRE/C-PIM):
#When using ASM groups on the P-PIM no need for MDT address family on BGP because RP is already
#signaling the *,G and S,G after the configuration of the default mdt for the VRF.

 multicast-routing
 address-family ipv4
  interface Loopback0 #P-PIM on all IGP interfaces
   enable
  !
 vrf Cust
  address-family ipv4
   interface GigabitEthernet0/0/0/0 #C-PIM interface.
    enable
   !
   mdt source Loopback0 #Required on XR.
   mdt default ipv4 239.1.1.1 #Multicast group of destination for GRE tunnels.
  !
 !
!
multicast-routing
Profile 0 (Default MDT/GRE/C-PIM) with SSM Groups:
 multicast-routing
 vrf Cust
  address-family ipv4
     mdt default ipv4 232.1.1.1 #Only change do an SSM range is enough.

router bgp 65000
 address-family ipv4 mdt #Enable globally.
  neighbor 10.0.0.2
    address-family ipv4 mdt #Enable on neighbor.

Profile 0 (Default MDT/GRE/C-PIM) Data MDT:
multicast-routing
 vrf Cust
  address-family ipv4
     mdt data 232.33.33.0/24 #Default threshold is 1Kbps.

###################################################################################################

Profile 1 (Default MDT/mLDP/C-PIM):
vrf Cust
 vpn id 1:1 #Opaque value for mLDP FEC Element

mpls ldp
 mldp #Enabling mLDP
  logging notifications
  address-family ipv4

route-policy rpf
  set core-tree mldp-default #Set the core-tree so IOS-XR can understand messages coming with mpls encapsulation.
end-policy

router pim
 vrf Cust
  address-family ipv4
   rpf topology route-policy rpf #Appling the core-tree

multicast-routing
 address-family ipv4
  interface Loopback0 #Enabling PIM on LoopBack so it can be used for source of the Lmdt interface.
   enable

 vrf Cust
  address-family ipv4
   mdt source Loopback0
   mdt default mldp ipv4 10.0.0.1 #Configuration of the root FEC Element.

IOS-XR configuration caveats (xrv9k-full 7.6.1):
#XRV9K-FULL supports the commands, establishes the C-PIM neighborship using mpls encap, receives the BSR updates but does not forwards traffic.
#XRV9K-FULL apparently does not support MPLS encap for MVPN. 


<h1>Anycast RP e MSDP:</h1>

<h1>Conceitos Básicos:</h1>

- **HA de RP:** Resiliência de RP, usando o menor caminho IGP.
- **Troca de informação de Sources:** RPs trocam informações de Sources via SA do MSDP.
- **IP anycast tem que estar no IGP:** Não usar IPs de RIDs ou qualquer outra interface atrelada a serviços diferentes.
- **MSDP:** TCP 639

<h2>IOS-XE Anycast RP e MSDP.</h2>

    interface GigabitEthernet1
     ip pim sparse-mode
    
    interface Loopback0
     ip address 10.0.0.1 255.255.255.255
     ip pim sparse-mode

    interface Loopback1
     ip address 10.1.1.1 255.255.255.255

    ip multicast-routing

    ip pim rp-address 10.1.1.1

    ip msdp peer 10.0.0.2 source Loopback0

    show ip msdp peer
    show ip msdp sa-cache
    show ip pim rp mapping
    show ip pim neighbor
    show ip mroute