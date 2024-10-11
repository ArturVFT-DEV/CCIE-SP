<h6>Notas OSPF:</h6>

<h5>1.1 Interior Gateway Protocol</h5>
<h5>1.1.b OSPFv2 and OSPFv3</h5>
<h5>1.1.c Optimize IGP scale and performance</h5>
<h5>5.1.b Routing protocol and LDP authentication and securitys</h5>

<h6>IOS-XE configuration and tunings (csr1kv 16.9.7):</h6>

- IGP Link State, mantém estado da topologia em uma base de dados.
- Tipos de adjacências:
    - Broadcast: Rede multi acesso, formação de DR e BDR.
    - Point-to-Point: Rede ponto a ponto, sem eleição, formação de adjacência mais rápida.
    - NBMA: 

> router ospf 1
>  router-id 172.16.125.1
>  max-lsa 1000 #Maximun number os LSAs the process can handle.
>  prefix-priority high route-map LOOPBACKS
>  prefix-suppression #Advertisement of only /32s.
>  passive-interface Loopback0 #Still need to activate under the interface
>  bfd all-interfaces # Enables BFD on process.
>  area 0 authentication # Type 1 auth