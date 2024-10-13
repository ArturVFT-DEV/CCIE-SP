<h1>Notas IS-IS:</h1>

<h4>1.1.a IS-IS</h4>
<h4>1.1.c Optimize IGP scale and performance</h4>
<h4>1.5.a IS-IS and OSPF extensions</h4>
<h4>1.6.a IS-IS segment rouging control plane for IPv4 and IPv6</h4>
<h4>4.2.a IGP convergence</h4>
<h4>5.1.b Routing protocol and LDP authentication and securitys</h4>

<h1>1-1 IS-IS:</h1>

<h1>1-2 Conceitos Básicos:</h1>

- **IGP Link State:** mantém estado da topologia em uma base de dados.
- **Identificação:** Todo roteador se identifica e identificas suas redes conectadas para o domínio.
- **Troca de informações:** Cada roteador recebe uma copia de informação, mantém para si e repassa a mesma ao seu vizinho.
- **Protocolo OSI**: Não foi originalmente desenhado para redes IP e sim para CLNS
- **RID e AID:** Expresso atravez do NET, sempre em Hex - AFI, AreaID, SysID e NSEL.

- **Adjacências:**
   - Roda em Layer2 necessitando de acesso ao meio para ataques.
   - Maior dificuldade em priorização de pacotes.

<h1>1-3 Tipos de Menssagens:</h1>
- Extendido facilmente para novas features apenas com adição de TLVs.
- Roteador gera apenas dois tipos de LSPs Level1 ou Level2, qualquer outra informação é encapsulada em TLVs.

   ![Cabeçalho ISIS](/Imagens/isis-header.png)

- **IIH:**
    - Formação de vizinhança.
    - Keepalive 
- **CSNP:**
    - 
- **PSNP:**
    - 
- **LSP:**
    - 

- **Subdivisão por tipo de adjacência:**
   - IS-IS subdivide as mensagens em Level1, Level2 e se a mesma é LAN ou PTP de acordo com a adjacência formada:
      - Level 1 LAN IIH
      - Level 2 LAN IIH
      - Point-to-Point IIH
      - Level 1 CSNP
      - Level 2 CSNP
      - Level 1 PSNP
      - Level 2 PSNP
      - Level 1 LSP
      - Level 2 LSP

<h2>IOS-XE configuration and tunings (csr1kv 16.9.7):</h2>

    router isis 1
     net 49.0000.0000.0000.0011.00
     is-type level-2-only
     advertise passive-only #Utilized to suppress routes like /30s and announce only loopbacks usually(Optional).
     authentication mode md5
     authentication key-chain ISIS #Authentication for LSPs, CSNPs and PSNPs.
     metric-style wide #Utilized to extend protocol capabilities(Optional).
     no hello padding #Reduce bandwidth usage post adjacency by suppressing padding on the frame(Optional).
     log-adjacency-changes all
     passive-interface Loopback0 #This is going to be the only interface with its address injected in the IGP(Optional).
    
    interface GigabitEthernet2
     ip address 10.1.11.11 255.255.255.0
     ip router isis 1 #ISIS activation for adjacency establishment
     negotiation auto
     no mop enabled
     no mop sysid
     isis network point-to-point #No need for DIS election on a link with only two IS's(Routers).
     isis authentication mode md5
     isis authentication key-chain ISIS #Authenticarion for IIH.
    end
    
    key chain ISIS #Optional key chain for md5 authentication.
     key 0
      key-string cisco
       cryptographic-algorithm md5

<h2>IOS-XE verification commands (csr1kv 16.9.7):</h2>

    show isis neighbors (detail) #Verify adjacencies Types/Interfaces/IP Addresses/State
    show isis database (detail) #Verify IP Addresses/Metrics/SR SIDs
    show key chain #Verify key string in quotes(Optional).
    show clns interface #ISIS interface details.
    show isis database  verbose | i Hostname|Metric #Simple view of full topology.

<h2>IOS-XR configuration and tunings (xrv9k-full 7.6.1):</h2>

    router isis 1
     is-type level-2-only
     net 49.0000.0000.0000.0002.00
     log adjacency changes
     lsp-password hmac-md5 encrypted 14141B180F0B #Authentication for LSPs, CSNPs and PSNPs.
     address-family ipv4 unicast
      metric-style wide
      advertise passive-only
     !
     interface Loopback0
      passive
      address-family ipv4 unicast
      !
     !
     interface GigabitEthernet0/0/0/0
      point-to-point
      hello-padding disable
      hello-password hmac-md5 encrypted 00071A150754 #Authentication for IIHs.
      address-family ipv4 unicast

<h2>IOS-XE configuration caveats (csr1kv 16.9.7):</h2>

#Starting with Release 16.10.1, "area-password" and "authentication key-chain" can not coexist.
#isis-password Old way of authentication without key-chain.
