<h1>MVPN</h1>

   ![igp](/Imagens/mvpn.png)

- **Configurações iniciais:** Copie as configurações de /configs/mvpn de cada equipamento e cole no respectivo R1-6.
- **Observações:**
  - Imagens:
    - CE1-7: i86bi_LinuxL3-AdvEnterpriseK9-M2_157_3_May_2018.bin
    - R1, 2 e 6: csr1000vng-universalk9.16.09.01.Fuji
    - R3, 4 e 5: xrv-6.6.3

<h3>LAB TAKS - Profile 0 (Default MDT)</h3>

- **Task 1. Habilite roteamento multicast e pim na operadora R1-6**
- **Task 2. Configure R1 como P-RP**
- **Task 3. CE4 é o Souce Multicast, realize um ping para o grupo 239.1.2.3, nesse momento CE7 ainda não recebe tráfego**
- **Task 4. Utilize Default MDT (Profile 0) para resolver a transitividade no backbone da operadora R2,3,5 e 6**
- **Task 5. CE7 nesse momento já deve conseguir receber tráfego multicast do source CE4 para o grupo 239.1.2.3**
- **Task 6. Profile0 já funciona nas configurações atuais, mas por recomendação da Cisco é necessário configurar BGP com "address-family mdt" entre os PE's**

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
     !
    !

  - **Explicação: P-PIM tem que estar habilitado em todas as interfaces de core.**

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

  - **Explicação: Nesse momento o trafego Multicast já estará sendo encapsulado em GRE pelo core da operadora para alcançar CE7, é possível visualizar nas imagens abaixo primeiramente a ida do pacote do ponto de vista do R6 vindo do R3(nesse momento o melhor caminho) o pacote está encapsulado em GRE onde a origem é 10.0.255.3 e o destino é o grupo Default MDT 239.10.20.30, dentro do tunel GRE o pacote IP orginal possui a origem 172.16.255.4 e destino 239.1.2.3. Outro ponto importante de se notar é que a volta do pacote ocorre em unicast como se trata de um ping, então o mesmo retorna pelo L3VPN é possível observar pela imposição de label da VPN que o R6 faz para entregar a R3 no meu caso o label 24013 corresponde ao prefixo vpnv4 172.16.255.4/32**

  <h3>Trafego indo para CE7</h3>

  ![request](/Imagens/profile0-R7-request.png)

  <h3>Trafego voltando para CE4</h3>

  ![reply](/Imagens/profile0-R7-reply.png)

</details>

<details>
  <summary><i>Task 6 RESPOSTA</i></summary>
  <b>IOS XE - R1 and R6</b>
    
    vrf definition MVPN-CUST
     address-family ipv4
      mdt default 239.10.20.30

  <b>IOS XR - R3 and R5</b>
    
    multicast-routing
     vrf MVPN-CUST
      address-family ipv4
       mdt source Loopback0
       mdt default ipv4 239.10.20.30

  - **Explicação: Definição do grupo Default MDT, esse será responsável por formar tuneis GRE Multiponto, o trafego multicast do cliente será encapsulado nesses tuneis para alcançar membros do grupo atravez do core da operadora.**

</details>