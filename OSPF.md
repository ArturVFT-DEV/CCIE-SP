<h1>Notas OSPF:</h1>

<h4>1.1.b OSPFv2 and OSPFv3</h4>
<h4>1.1.c Optimize IGP scale and performance</h4>
<h4>1.5.a IS-IS and OSPF extensions</h4>
<h4>1.6.b OSPFv2 and OSPRv3 segment routing control plane</h4>
<h4>2.5.b PE-CE routing protocols (OSPF and BGP)</h4>
<h4>4.2.a IGP convergence</h4>
<h4>5.1.b Routing protocol and LDP authentication and securitys</h4>

<h1>1-1 OSPFv2:</h1>

<h1>1-2 Conceitos Básicos:</h1>

- **IGP Link State:** mantém estado da topologia em uma base de dados.
- **Identificação:** Todo roteador se identifica e identificas suas redes conectadas para o domínio.
- **Troca de informações:** Cada roteador recebe uma copia de informação, mantém para si e repassa a mesma ao seu vizinho.
- **Adjacências:**
    - Troca de informação de RID nos Hellos.
    - Mesma subnet IP. (Existe uma particularidade p-t-p e vl **APPENDIX**).
    - Descorberta por Multicast 224.0.0.5 (AllOSPFRouters) | 224.0.0.6 (AllDRouters).
    - TTL por padrão 1.
    - IP Protocol 89.
    - Mesma área OSPF.
    - Typo e string de autenticação iguais, Type 0 null, Type 1 clear, Type 2 md5/sha1.
    - Tempos de Dead e Hello iguais.
    - IP MTU iguais.
    - Devem concordar se a área é ou não Stub.
    - Rodando em cima de IP, mais sujeito a ataques exemplo Spoofing.(IRPAS/Nemesis)
- **Flooding:**
    - Após flooding inicial de um LSA só haverá novos envios em caso de mudança topologica ou LSA expire (30 min).
    - Aging, Sequence Number e Checksum determinam a veracidade do LSA.
- **Database:**
    - Apenas uma copia do Header e trocada na fase inicial.
    - Vizinho solicita todos os LSAs que ele não conhece.
    - Consistente dentro da área.
- **Calculo SPF:**
    - Candidate Database alimenta a Tree.
    - Cada Roteador se vê como root, alimentando a Candidate com todos os links e custos.
    - Apenas os melhores são escolhidos para a Tree, se não houve mais entradas na Candidate, encerra o calculo.

<h1>1-3 Tipos de Menssagens:</h1>
    - Protocolo criado numa época de escassez de recursos computacionais, portanto tem tamanhos fixos de 32-bits.
    - Para haver extenções é necessário criação de novos LSAs.

![Cabeçalho OSPF](/Imagens/ospf-header.png)

<h1>1-4 Tipos de Adjacência:</h1>

- **Broadcast:** Rede multi acesso, formação de DR e BDR.
- **Point-to-Point:** Rede ponto a ponto, sem eleição, formação de adjacência mais rápida.
- **NBMA:** 

<h2>OSPFv2 - IOS-XE configuração básica de vizinhança (csr1kv 16.9.7):</h2>

    router ospf 1
     router-id 172.16.125.1
     max-lsa 1000 #Maximun number os LSAs the process can handle.
     prefix-priority high route-map LOOPBACKS
     prefix-suppression #Advertisement of only /32s.
     passive-interface Loopback0 #Still need to activate under the interface
     bfd all-interfaces # Enables BFD on process.
     area 0 authentication # Type 1 auth

<h2>OSPFv3 - IOS-XE configuração básica de vizinhança (csr1kv 16.9.7):</h2>

    router ospfv3 1
     !
     address-family ipv6 unicast
      passive-interface Loopback0
     exit-address-family

    interface GigabitEthernet2
     ip address 172.16.14.1 255.255.255.0
     ip ospf authentication message-digest #Authentication type 2.
     ip ospf authentication-key cisco #Authentication type 1.
     ip ospf authentication null #Authentication type 0.
     ip ospf message-digest-key 1 md5 cisco #Algorigthm used and password
     ip ospf network point-to-point #Only Router LSAs.
     ip ospf 1 area 0
     negotiation auto
     ipv6 enable
     ipv6 ospf authentication ipsec spi 1040 md5 0123456789ABCDEF0123456789ABCDEF
     ipv6 ospf 1 area 0
     ipv6 ospf network point-to-point
     bfd interval 50 min_rx 50 multiplier 3


<h2>OSPFv2 - IOS-XR configuration and tunings (xrv9k-full 7.6.1):</h2>

    interface GigabitEthernet0/0/0/1
     ipv4 address 172.16.24.2 255.255.255.0

    router ospf 1
     router-id 172.16.125.4
     spf prefix-priority route-policy LOOPBACKS #High priority for convergence.
     area 0
      bfd minimum-interval 150
      bfd fast-detect
      bfd multiplier 3
      authentication #Type1
      prefix-suppression #Advertisement of only /32s.
      interface Loopback0
       passive enable
      interface GigabitEthernet0/0/0/0
       authentication null #Type0
       authentication-key cisco #Type1
       authentication message-digest #Type2
       message-digest-key 1 md5 encrypted 0822455D0A16
       network point-to-point

<h1>APPENDIX:</h1>

**1-2 Adjacências:** RFC2328 - 10.5 - Unnumbered Point-to-Point e Virtual Links são ignorados em relação a subnet.