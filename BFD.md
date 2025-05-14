<h1>Notas BFD:</h1>

4.2.d BFD

<h1>BFD:</h1>

<h1>Conceitos Básicos:</h1>

- **BFD Echos:** Menssagem padrão destinada a propria origem (realiza um loopback no vizinho), mais leve, tratado em hardware, UDP 3785.
- **BFD Control:** Menssagem mais pesada em algumas plataformas pois é enviada para CPU, UDP 3784.


<h1>CONFIGURAÇÕES</h1>

<h1>BFD para OSPF:</h1>

<h2>IOS-XE BFD</h2>

    interface GigabitEthernet1
     ip ospf bfd
     bfd interval 100 min_rx 100 multiplier 3

    router ospf 1
     bfd all-interfaces

<h2>IOS-XR BFD</h2>

    router ospf 1
     area 0
      bfd minimum-interval 100
      bfd fast-detect
      bfd multiplier 3

