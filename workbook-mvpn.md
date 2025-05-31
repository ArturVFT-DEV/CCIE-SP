<h1>MVPN</h1>

   ![igp](/Imagens/mvpn.png)

- **Configurações iniciais:** Copie as configurações de /configs/mvpn de cada equipamento e cole no respectivo R1-6.
- **Observações:**
  - Imagens:
    - CE1-7: i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018.bin
    - R1, 2 e 6: csr1000vng-universalk9.16.09.01.Fuji
    - R3, 4 e 5: xrv-6.6.3

<h3>LAB TASKS - Profile 0 (Default MDT-GRE)</h3>

- **Task 1. Habilite roteamento multicast e pim na operadora R1-6 e nas interfaces de VRF.**
- **Task 2. Configure R1 como P-RP.**
- **Task 3. CE4 é o Souce Multicast, realize um ping para o grupo 239.1.2.3, nesse momento CE7 ainda não recebe tráfego.**
- **Task 4. Utilize Default MDT (Profile 0) 239.10.20.30 para resolver a transitividade no backbone da operadora R2,3,5 e 6.**
- **Task 5. CE7 nesse momento já deve conseguir receber tráfego multicast do source CE4 para o grupo 239.1.2.3.**
- **Task 6. Profile0 já funciona nas configurações atuais, mas por recomendação da Cisco é necessário configurar BGP com "address-family mdt" entre os PE's utilize SSM na Default MDT 232.10.20.30.**
- **Task 7. Configure Data MDT 232.rtr.rtr.rtr exemplo R1 232.1.1.1 Utilize 2Kbps com threshould e valide a comutação no IOS-XE desligando a interface entre R3 e CE5 pois o XRv não consegue identificar o threshould em dataplane para comutar para a Data MDT.**

<details>
  <summary><i>Task 1 RESPOSTA</i></summary>
  <b>IOS XE</b>

    ip multicast-routing distributed
    interface Loopback0
     ip pim sparse-mode
    interface GigabitEthernet1
     ip pim sparse-mode

  <b>IOS XR</b>

    multicast-routing
     address-family ipv4
      interface all enable

  - **Explicação: P-PIM tem que estar habilitado em todas as interfaces de core, e C-PIM em todas as interfaces de cliente nos R2,3,5 e 6.**

</details>

<details>
  <summary><i>Task 2 RESPOSTA</i></summary>
  <b>R1</b>

    ip pim bsr-candidate Loopback0 0
    ip pim rp-candidate Loopback0

  - **Explicação: P-PIM precisa ter um RP para propagação do grupo de Default MDT.**
  
</details>

<details>
  <summary><i>Task 3 RESPOSTA</i></summary>
  <b>CE4</b>
    
    CE4#ping 239.1.2.3 repeat 15
    Type escape sequence to abort.
    Sending 15, 100-byte ICMP Echos to 239.1.2.3, timeout is 2 seconds:
    Reply to request 0 from 172.16.255.5, 39 ms
    Reply to request 0 from 172.16.255.3, 45 ms

  - **Explicação: Nesse momento é possível ver que o CE7 ainda não responde, portanto ainda não recebe tráfego multicast para o grupo devido a tree não conseguir se formar pelo core da operadora.**
  
</details>

<details>
  <summary><i>Task 4 RESPOSTA</i></summary>
  <b>IOS XE - R1 e R6</b>
    
    vrf definition MVPN-CUST
     address-family ipv4
      mdt default 239.10.20.30

  <b>IOS XR - R3 e R5</b>
    
    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       mdt source Loopback0
       mdt default ipv4 239.10.20.30

  - **Explicação: Definição do grupo Default MDT, esse será responsável por formar tuneis GRE Multiponto, o trafego multicast do cliente será encapsulado nesses tuneis para alcançar membros do grupo atravez do core da operadora.**

</details>

<details>
  <summary><i>Task 5 RESPOSTA</i></summary>
  <b>CE4</b>
    
    CE4#ping 239.1.2.3 repeat 1 source lo0
    Type escape sequence to abort.
    Sending 1, 100-byte ICMP Echos to 239.1.2.3, timeout is 2 seconds:
    Packet sent with a source address of 172.16.255.4 
    Reply to request 0 from 172.16.255.3, 1 ms
    Reply to request 0 from 172.16.255.7, 4 ms
    Reply to request 0 from 172.16.255.7, 4 ms
    Reply to request 0 from 172.16.255.3, 1 ms
    Reply to request 0 from 172.16.255.5, 1 ms
    Reply to request 0 from 172.16.255.5, 1 ms

  - **Explicação: Nesse momento o trafego Multicast já estará sendo encapsulado em GRE pelo core da operadora para alcançar CE7, é possível visualizar nas imagens abaixo primeiramente a ida do pacote do ponto de vista do R6 vindo do R3(nesse momento o melhor caminho) o pacote está encapsulado em GRE onde a origem é 10.0.255.3 e o destino é o grupo Default MDT 239.10.20.30, dentro do tunel GRE o pacote IP orginal possui a origem 172.16.255.4 e destino 239.1.2.3. Outro ponto importante de se notar é que a volta do pacote ocorre em unicast como se trata de um ping, então o mesmo retorna pelo L3VPN é possível observar pela imposição de label da VPN que o R6 faz para entregar a R3 no meu caso o label 24013 corresponde ao prefixo vpnv4 172.16.255.4/32.**

  <h3>Trafego indo para CE7</h3>

  ![request](/Imagens/profile0-R7-request.png)

  <h3>Trafego voltando para CE4</h3>

  ![reply](/Imagens/profile0-R7-reply.png)

</details>

<details>
  <summary><i>Task 6 RESPOSTA</i></summary>
  <b>IOS XE - R1</b>

    router bgp 65000
     address-family ipv4 mdt
      neighbor 10.0.255.2 activate
      neighbor 10.0.255.3 activate
      neighbor 10.0.255.5 activate
      neighbor 10.0.255.6 activate

  <b>IOS XE - R2 e R6</b>

    router bgp 65000
    address-family ipv4 mdt 
      neighbor 10.0.255.1 activate
      neighbor 10.0.255.4 activate
    
    ip pim ssm deafult
    vrf definition MVPN-CUS
     address-family ipv4
      no  mdt default 239.10.20.30
      mdt default 232.10.20.30
    
  <b>IOS XR - R4</b>

    router bgp 65000
     address-family ipv4 mdt
     neighbor 10.0.255.2
      address-family ipv4 mdt
       route-reflector-client
     neighbor 10.0.255.3
      address-family ipv4 mdt
       route-reflector-client
     neighbor 10.0.255.5
      address-family ipv4 mdt
       route-reflector-client
     neighbor 10.0.255.6
      address-family ipv4 mdt
       route-reflector-client

  <b>IOS XR - R3 e R5</b>

    router bgp 65000
     address-family ipv4 mdt
     neighbor 10.0.255.1
      address-family ipv4 mdt
     neighbor 10.0.255.4
      address-family ipv4 mdt
    
    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       no mdt default ipv4 239.10.20.30
       mdt default ipv4 232.10.20.30

  - **Explicação: Para grupos SSM 232.0.0.0/8 é necessário utilizar familia MDT do BGP para propagação de Source/Group entre os PEs, na documentação Cisco essa configuração é necessária até mesmo para ASM, mas funciona mesmo sem estas configurações. Dessa forma existirá menos manutenção de estado nos P's de toda a rede sejá de *,G ou S,G não existirá mais a necessidade de RP e por fim o Source não será mais aprendido via Dataplane e sim via Controlplane BGP.**

</details>

<details>
  <summary><i>Task 7 RESPOSTA</i></summary>
  <b>IOS XE - R2 e R6</b>
    
    vrf definition MVPN-CUST
     address-family ipv4
      mdt data 232.2.2.0 0.0.0.255 threshold 2

    vrf definition MVPN-CUST
     address-family ipv4
      mdt data 232.6.6.0 0.0.0.255 threshold 2

  <b>IOS XR - R3 e R5</b>
    
    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       mdt data 239.3.3.0/24 threshold 2

    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       mdt data 239.5.5.0/24 threshold 2

  - **Explicação: Com essa configuração os PE's vão identificar em dataplane o momento que o trafego multicast da VRF ultrapassar o threshould estarão sinalizando o grupo multicast da Data MDT que deve ser usado pelos PE's interessados naquele trafego, após a virada, apenas os PE's que possuem receivers daquele C-Group farão parte da Data MDT.**

  <h3>Trafego de R2 para R5 ainda não atingindo o threshould</h3>

  ![default-mdt-r5](/Imagens/default-mdt-r5.png)

  <h3>Trafego de R2 para R5 após o threshould ser ultrapassado e R2 sinalizar o Data MDT</h3>

  ![data-mdt-r5](/Imagens/data-mdt-r5.png)

  <h3>Criação da S,G para o Data MDT em R2</h3>

  ![data-mdt-r2](/Imagens/data-mdt-r2.png)

  <h3>Envio do grupo Data MDT a partir R2 quando ultrapassado o threshould</h3>

  ![r2-send](/Imagens/r2-send.png)

</details>

<h3>LAB TASKS - Profile 1 (Default MDT-mLDP)</h3>

- **Task 1. Habilite roteamento multicast e pim nas interfaces de cliente nos R2,3,5 e 6.**
- **Task 2. Configure Profile 1 (Default MDT-MLDP) mLDP deve ser habilitado nos R2,3,5 e 6.**
- **Task 3. Configure o VPN ID e o Root sendo R1 nos R2,3,5 e 6.**
- **Task 4. Configure os PEs IOS-XR R3 e R5 especificando o source-interface mdt e o RPF correto do mdt mpls-default.**

<details>
  <summary><i>Task 1 RESPOSTA</i></summary>
  <b>IOS XE</b>
    ip multicast-routing vrf MVPN-CUST distributed
    interface GigabitEthernet1
     vrf forwarding MVPN-CUST
     ip pim sparse-mode

  <b>IOS XR</b>

    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       interface all enable

  - **Explicação: C-PIM em todas as interfaces de cliente nos R2,3,5 e 6.**

</details>

<details>
  <summary><i>Task 2 RESPOSTA</i></summary>

  <b>IOS XR</b>

    mpls ldp
     mldp
      address-family ipv4


  - **Explicação: mLDP já é habilitado por padrão em IOS e IOS-XE sendo necessário configurar explicitamente no IOS-XR.**

</details>

<details>
  <summary><i>Task 3 RESPOSTA</i></summary>
  <b>IOS XE</b>
    vrf definition MVPN-CUST
     vpn id 1:1

    address-family ipv4
     mdt default mpls mldp 10.0.255.1

  <b>IOS XR</b>

    vrf MVPN-CUST
     vpn id 1:1

    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       mdt source Loopback0
       mdt default mldp ipv4 10.0.255.1

  - **Explicação: VPN ID é necessário ser configurado igual por vrf em todos os PEs para ser utilizando como Opaque Value, o root também sendo etapa obrigatória.**

</details>

<details>
  <summary><i>Task 4 RESPOSTA</i></summary>

 <b>IOS XR</b>

    route-policy CORE-TREE-MLDP-DEFAULT
      set core-tree mldp-default
    end-policy

    multicast-routing
     address-family ipv4
      interface Loopback0
       enable
     vrf MVPN-CUST
      address-family ipv4
       mdt source Loopback0

    router pim
     vrf MVPN-CUST
      address-family ipv4
       rpf topology route-policy CORE-TREE-MLDP-DEFAULT

  - **Explicação: No IOS-XR existem algumas configurações adicionais para o funcionamento correto, deve ser especificado o source interface do MDT e também configurada a Loopback na tabela global de Multicast. RPF não se resolve por padrão a menos que configurada atravez de route-policy.**

</details>