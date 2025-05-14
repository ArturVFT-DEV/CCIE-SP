<h1>Notas OSPF:</h1>

<h4>1.1.b OSPFv2 and OSPFv3</h4>
<h4>1.1.c Optimize IGP scale and performance</h4>
<h4>1.5.a IS-IS and OSPF extensions</h4>
<h4>1.6.b OSPFv2 and OSPRv3 segment routing control plane</h4>
<h4>2.5.b PE-CE routing protocols (OSPF and BGP)</h4>
<h4>4.2.a IGP convergence</h4>
<h4>5.1.b Routing protocol and LDP authentication and securitys</h4>

<h1>OSPFv2:</h1>

<h1>Conceitos Básicos:</h1>

- **IGP Link State:** mantém estado da topologia em uma base de dados.
- **Identificação:** Todo roteador se identifica e identificas suas redes conectadas para o domínio.
- **Troca de informações:** Cada roteador recebe uma copia de informação, mantém para si e repassa a mesma ao seu vizinho.
- **RID e AID:** 32bit doted decimal assim como IPv4, AID pode ser expresso em decimal também. 
- **Database:**
    - Apenas uma copia do Header e trocada na fase inicial.
    - Vizinho solicita todos os LSAs que ele não conhece.
    - Consistente dentro da área.
- **Calculo SPF:**
    - Candidate Database alimenta a Tree.
    - Cada Roteador se vê como root, alimentando a Candidate com todos os links e custos.
    - Apenas os melhores são escolhidos para a Tree, se não houve mais entradas na Candidate, encerra o cálculo.
- **Seleção de caminhos:**
    - Versões atuais de IOS obedecem a RFC3101.
    - O > O Ia > N1 > E1 > N2 > E2

<h1>Tipos de Menssagens:</h1>
- 5 mensagens diferentes.
- Tamanhos fixos de 32-bits cabeçalhos.
- Para haver extenções é necessário criação de novos LSAs.
- Roteador pode gerar varios LSAs de um mesmo ou vários tipos.

   ![Cabeçalho OSPF](/Imagens/ospf-header.png)

- **Hello:** Formação de vizinhança e manutenção da mesma.
- **DBD:** Troca de informação inicial entre vizinhos para sincronização de databases.
- **LSR:** Requisição de um LSA perdido ou desatualizado.
- **LSU:** Anuncio de um LSA criado por um novo link descoberto, renovação ou remoção.
- **LSAkc:** Confirmação do recebimento dos Updates.

- **Flooding:**
    - Após flooding inicial de um LSA só haverá novos envios em caso de mudança topologica ou LSA expire (30 min).
    - Aging, Sequence Number e Checksum determinam a veracidade do LSA.
    - LSAs possuem tamanhos diferentes dependendo do Type.
    - Redistribuir uma interface consome mais memoria mas consome menos CPU em calculo de SPF por ser um Type-5, por outro lado torna-la passive vai consumir menos memoria porem terá calculo de SPF para qualquer mudança na topologia por gerar um Type-1.

<h1>Tipos de Adjacência:</h1>

- **Broadcast:** Vizinhança multicast, formação de DR e BDR, 10s Hello e 40s Dead.
- **Point-to-Point:** Vizinhança multicast, sem eleição, 10s Hello e 40s Dead.
- **Point-to-Multipoint:** Vizinhança multicast, sem eleição, 30s Hello e 120s Dead
- **NBMA:** Vizinhança unicast, especificar IP do neighbor, eleição de DR, 30s Hello e 120s 
Dead.

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

<h1>Status de interface:</h1>

- **Down:** Link indisponível, sem envio ou recebimento de menssagens.
- **Loopback:** Geralmente para manutenções, loop em hardware ou software.
- **Waiting:** Verificando se existe DR ou BDR.
- **Point to point:** Vizinho detectado adjacência será iniciada.
- **DR Other:** Vizinhança operacional, não é DR nem BDR.
- **Backup:** BDR, sem sincronização de database.
- **DR:** Vizinhança operacional, sincronização de database realizada.

<h1>Tipos de Áreas:</h1>

- **Backbone e Não Backbone:** Backbone area 0 / Não Backbone qualquer outra.
- **Stub:** LSA 4 e 5 não permitidos, gera default LSA type 3, "E" bit deve ser igual.
- **Totally Stubby:** LSA 3, 4 e 5 e  não permitidos, gera default LSA type 3.
- **Not-So-Stubby:** LSA 7 traduzido para 5 no ABR, gerado por um ASBR na area.

<h1>Configurações Especificas</h1>

<h1>Tipos de Adjacência:</h1>

<h2>IOS-XE Broadcast</h2>

    interface GigabitEthernet1
     ip router ospf 1 area 0

<h2>IOS-XR Broadcast</h2>

    router ospf 1
     area 0
      interface GigabitEthernet0/0/0/0

<h2>IOS-XE Point-to-Point</h2>

    interface GigabitEthernet1
     ip router ospf 1 area 0
     network point-to-point

<h2>IOS-XR Point-to-Point</h2>

    router ospf 1
     area 0
      interface GigabitEthernet0/0/0/0
       network point-to-point

<h2>IOS-XE Point-to-Multipoint</h2>

    interface GigabitEthernet1
     ip router ospf 1 area 0
     network point-to-multipoint

<h2>IOS-XR Point-to-Multipoint</h2>

    router ospf 1
     area 0
      interface GigabitEthernet0/0/0/0
       network point-to-multipoint

<h2>IOS-XE Non-Broadcast</h2>

    interface GigabitEthernet1
     ip router ospf 1 area 0
     network non-broadcast

    router ospf 1
     neighbor 10.0.0.1

<h2>IOS-XR Non-Broadcast</h2>

    router ospf 1
     area 0
      interface GigabitEthernet0/0/0/0
       network non-broadcast
       neighbor 10.0.0.2

<h1>Segurança:</h1>

<h1>Autenticação</h1>

<h2>IOS-XE Autenticação Type0: Sem autenticação.</h2>

    interface GigabitEthernet1
     ip ospf authentication null

<h2>IOS-XR Autenticação Type0: Sem autenticação.</h2>

    router ospf 1
     area 0
      interface GigabitEthernet0/0/0/0
       authentication null

<h2>IOS-XE Autenticação Type1: Autenticação por texto claro.</h2>

    router ospf 1
     area 0 authentication

    interface GigabitEthernet1
     ip ospf authentication null

<h2>IOS-XR Autenticação Type1: Autenticação por texto claro.</h2>

    router ospf 1
     area 0
      authentication
      interface GigabitEthernet0/0/0/0
       authentication-key cisco

<h2>IOS-XE Autenticação Type2: Autenticação com encriptação.</h2>

    router ospf 1
     ip ospf authentication message-digest
     ip ospf message-digest-key 1 md5 cisco

    interface GigabitEthernet1
     ip ospf authentication message-digest
     ip ospf message-digest-key 1 md5 cisco

<h2>IOS-XR Autenticação Type2: Autenticação com encriptação.</h2>

    router ospf 1
     area 0
      authentication
      interface GigabitEthernet0/0/0/0
       authentication message-digest
       message-digest-key 1 md5 encrypted cisco

<h1>TTL Security</h1>

<h2>IOS-XR TTL Security no processo e por interface</h2>

    router ospf 1
     ttl-security all-interfaces hops 2

    interface GigabitEthernet1
     ttl-security hops 2

<h2>IOS-XR TTL Security no processo e por interface</h2>

    router ospf 1
     area 0
      security ttl hops 2

      interface GigabituEthernet 0/0/0/0
       security ttl hops 2

- **TTL Security:** TTL dos pacotes será 255, hops sendo os valores a serem subtraidos.

<h1>Performance e Escalabilidade:</h1>

<h2>IOS-XE Prefix supression: Anuncio apenas de host routes /32.</h2>

    router ospf 1
     prefix-suppression
    
<h2>IOS-XR Prefix supression: Anuncio apenas de host routes /32.</h2>

    router ospf 1
     area 0
      prefix-suppression

<h2>IOS-XE BFD: Detecção de falhas abaixo de 1 segundo.</h2>

    router ospf 1
     bfd all-interfaces

    interface GigabitEthernet1
     bfd interval 50 min_rx 50 multiplier 3

<h2>IOS-XR BFD: Detecção de falhas abaixo de 1 segundo.</h2>

    router ospf 1
     area 0
      bfd minimum-interval 150
      bfd fast-detect
      bfd multiplier 3

<h2>OSPFv2 - IOS-XE configuração básica de vizinhança (csr1kv 16.9.7):</h2>

    router ospf 1
     router-id 172.16.125.1
     max-lsa 1000 #Maximun number os LSAs the process can handle.
     prefix-priority high route-map LOOPBACKS
     prefix-suppression #Advertisement of only /32s.
     passive-interface Loopback0 #Still need to activate under the interface
     bfd all-interfaces # Enables BFD on process.
     area 0 authentication # Type 1 auth

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

<h1>OSPFv3:</h1>

- **IOS-XR IPv4:** IOS-XR suporta apenas a address family ipv6.
- **IOS-XE IPv4 e IPv6:** IOS-XE suporta ambas as address families.


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

<h1>APPENDIX:</h1>

**Adjacências:** RFC2328 - 10.5 - Unnumbered Point-to-Point e Virtual Links são ignorados em relação a subnet.