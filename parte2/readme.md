# CONFIGURAÇÃO DO SERVIDOR GETWAY COMO NAT

## Configuração do firewall/NAT

   * Para configurar um servidor como gateway de rede é necessário configurar o firewall do linux (iptables). 
   * as regras do iptables podem ser digitadas no terminal ou podem ser executadas em um script.
   * Com um script, pode-se inicializar as regras do firewall todas as vezes que a máquina for reinicializada.

### habilitar o firewall 
   * Vamos serguir os passos descritos em [1]:
   
---
   1. habilitar o firewall e permitir o acesso ssh:
```bash
 sudo ufw enable
```
```bash
 sudo ufw allow ssh
```
![ufw eneble & allow ssh](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/2.jpg)

* Nome eu caso o ufw já estava confugurado
---
   2. habilitar o encaminhamento de pacotes das interfaces WAN para LAN, ajustando-se os parâmetros no arquivo **/etc/ufw/sysctl.conf**, removendo-se a marca de comentário (#) da seguinte linha _# net/ipv4/ip_forwarding=1_
```bash
 sudo nano /etc/ufw/sysctl.conf
``` 
```bash
...
net/ipv4/ip_forwarding=1
...
```
![/etc/ufw/sysctl.conf](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/3.jpg)

---
   3. confira o nome das interfaces de rede
```bash
 ifconfig -a
```
```bash
WAN interface: ens160
LAN interface: ens192
```

![ifconfig -a](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/4.jpg)

---
   4. no ubuntu 18.04 o arquivo /etc/rc.local não existe mais. Então é necessário recriá-lo.
```bash
 sudo nano /etc/rc.local
```

---
   5. A seguir, adicione o seguinte script no arquivo (rc.local)
```bash
#!/bin/bash

# /etc/rc.local

# Default policy to drop all incoming packets.
# Politica padrão para bloquear (drop) todos os pacotes de entrada
iptables -P INPUT DROP
iptables -P FORWARD DROP

# Accept incoming packets from localhost and the LAN interface.
# Aceita pacotes de entrada a partir das interfaces localhost e the LAN.
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -i enp0s8 -j ACCEPT

# Accept incoming packets from the WAN if the router initiated the connection.
# Aceita pacotes de entrada a partir da WAN se o roteador iniciou a conexao
iptables -A INPUT -i enp0s3 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# Forward LAN packets to the WAN.
# Encaminha os pacotes da LAN para a WAN
iptables -A FORWARD -i enp0s8 -o enp0s3 -j ACCEPT

# Forward WAN packets to the LAN if the LAN initiated the connection.
# Encaminha os pacotes WAN para a LAN se a LAN inicar a conexao.
iptables -A FORWARD -i enp0s3 -o enp0s8 -m conntrack \
--ctstate ESTABLISHED,RELATED -j ACCEPT

# NAT traffic going out the WAN interface.
# Trafego NAT sai pela interface WAN
iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# rc.local needs to exit with 0
# rc.local precisa sair com 0
exit 0
```

![/etc/rc.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/5.jpg)

---
   6. converte o arquivo em executável e o torna inicializável no boot
```bash
 sudo chmod 755 /etc/rc.local
```

---
   7. verificar se o firewall está funcionando
```bash
 sudo ufw status
```
ou
```bash
 systemctl status ufw.service
```


---
   8.  reiniciar a máquina
```bash
 sudo reboot
```

![chmod-reboot](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/6.jpg)

---
  9. Encaminhamento de portas para acesso externo à serviços da rede interna.
  
  * Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente, adicione as informações do IPTABLES sobre portas, IP e Interface no arquivo /etc/rc.local conforme o exemplo abaixo, depois reinicie a máquina:
  
   a. SAMBA: Para permitir que o serviço de compartilhamento de arquivos esteja disponível externamente:
        * Portas: 445 e 139
        * Interface Externa aqui é a WAN: ens192
        * IP do servidor = 10.9.14.100
        
```bash
#Recebe pacotes na porta 445 da interface externa do gw e encaminha para o servidor interno na porta 445
iptables -A PREROUTING -t nat -i ens192 -p tcp –-dport 445 -j DNAT –-to 10.9.14.100:445
iptables -A FORWARD -p tcp -d 10.9.14.100 –-dport 445 -j ACCEPT

#Recebe pacotes na porta 139 da interface externa do gw e encaminha para o servidor interno na porta 139
iptables -A PREROUTING -t nat -i ens192 -p tcp –-dport 139 -j DNAT –-to 10.9.14.100:139
iptables -A FORWARD -p tcp -d 10.9.14.100 –-dport 445 -j ACCEPT
```
   b. DNS: Para permitir que o serviço de resolução de nomes (DNS) esteja disponível externamente:
        * Porta: 53
        * Interface Externa aqui é a WAN: enp0s3
        * IP do servidor nameserver1 (ns1) = 10.9.14.126
        
```bash
#Recebe pacotes na porta 53 da interface externa do gw e encaminha para o servidor DNS Master interno na porta 53
iptables -A PREROUTING -t nat -i ens160 -p tcp –-dport 53 -j DNAT –-to 10.9.14.126:53
iptables -A FORWARD -p udp -d 10.9.14.126 –-dport 53 -j ACCEPT
```
![porta 445,139,53](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/7.jpg)


### [VOLTAR PARA HOME](https://github.com/0rmindo/SRed-2021/blob/main/README.md)
