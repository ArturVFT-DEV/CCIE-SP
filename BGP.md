<h1>Notas BGP:</h1>

<h1>1.2.a IBGP, EBGP, and MP-BGP</h1>
<h1>1.2.b BGP route policy enforcement</h1>
<h1>1.2.c BGP path attribute</h1>
<h1>1.2.d BGP scale and performance</h1>
<h1>1.2.e BGP labeled unicast and linked state</h1>
<h1>1.6.c BGP SR</h1>
<h1>2.5.b PE-CE routing protocols (OSPF and BGP)</h1>
<h1>4.2.c BGP Prefix Independent Convergence (BGP-PIC)</h1>
<h1>5.1.c BGP prefix-based and attribute-based filtering</h1>
<h1>5.1.d BGP-RPKI (origin AS validation)</h1>
<h1>5.3.d BGP Flowspec</h1>

<h1>1-1 BGP:</h1>

- **Adjacências:**
    - Vizinhança: Opera na porta TCP 179.
    - Especificação do peer: Diferente de IGPs necessita da identificação manual do IP do peer seja interface diretamente conectada ou recursiva.
    - eBGP: TTL = 1 | atributos de NLRI's como next-hop e as-path se alteram.
    - iBGP: TTL = 255 | atributos de NLRI's não se alteram | propagação não altomática necessitando de Route Reflector ou Confederations.
    - Protocolo OSI: Não foi originalmente desenhado para redes IP e sim para CLNS
    - RID e AID: Expresso atravez do NET, sempre em Hex - AFI, AreaID, SysID e NSEL.

<h2>IOS-XR configuration and tunings (xrv9k-full 7.6.1):</h2>

    router bgp 10
     address-family vpnv4 unicast
      additional-paths receive    # Receber prefixos alem do best-path.
     !
     neighbor 17.17.17.1
      remote-as 10
      update-source Loopback1
      address-family vpnv4 unicast
      !
     !
     vrf PEPSI
      bgp unsafe-ebgp-policy     # Permite o recebimento de prefixos de peer ebgp sem route-policy
      address-family ipv4 unicast
      !
      neighbor 10.3.15.3
       remote-as 1234.1234
       address-family ipv4 unicast
        route-policy SOO-3 in
        maximum-prefix 10 75 discard-extra-paths
        as-override
        site-of-origin 1234:4
       !
      !
     !        
    !


