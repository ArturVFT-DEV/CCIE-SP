<h1>OSPF</h1>

<h2>Supressão de redes</h2>

   ![igp](/Imagens/igp.png)

- **Configurações iniciais:** Copie as configurações de /configs/igp/ospf de cada equipamento e cole no respectivo R1-6.

<h3>LAB TAKS</h3>

- **Task 1. Apenas redes de loopback devem existir nas tabelas de roteamento**
- **Task 2. Optimize o ambiente para que as vizinhanças se estabeleçam mais rapidamente**
- **Task 3. Configure autenticação type 2 em todos os vizinhos**

<details>
  <summary><i>Task 1 RESPOSTA</i></summary>
  <b>IOS XE</b>

    router ospf 1
     prefix-suppression

  <b>IOS XR</b>
    router ospf 1
     prefix-suppression
</details>

<details>
  <summary><i>Task 2 RESPOSTA</i></summary>
  <b>IOS XE</b>

    interface G1
     ip ospf network point-to-point

  <b>IOS XR</b>
    router ospf 1
     area 0
      interface G0/0/0/0
       network point-to-point
</details>