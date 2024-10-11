<h1>Notas OSPF:</h1>

<h2>1.1 Interior Gateway Protocol</h2>
<h2>1.1.b OSPFv2 and OSPFv3</h2>
<h2>1.1.c Optimize IGP scale and performance</h2>
<h2>5.1.b Routing protocol and LDP authentication and securitys</h2>

<h1>Conceitos Básicos:</h1>

- IGP Link State, mantém estado da topologia em uma base de dados.

<h1>Tipos de Adjacência:</h1>

- Broadcast: Rede multi acesso, formação de DR e BDR.
- Point-to-Point: Rede ponto a ponto, sem eleição, formação de adjacência mais rápida.
- NBMA: 

<h1>IOS-XE configuração básica de vizinhança (csr1kv 16.9.7):</h1>

    router ospf 1
     router-id 172.16.125.1
     max-lsa 1000 #Maximun number os LSAs the process can handle.
     prefix-priority high route-map LOOPBACKS
     prefix-suppression #Advertisement of only /32s.
     passive-interface Loopback0 #Still need to activate under the interface
     bfd all-interfaces # Enables BFD on process.
     area 0 authentication # Type 1 auth