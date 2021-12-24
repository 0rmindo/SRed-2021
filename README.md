# CRIAÇÃO DE SERVIÇO SAMBA E SERVIDOR DNS
Resolução da Atividade Samba e DNS
> Siga o passo a passo

# Requisitos Basicos
- [X] OpenVpn Configurado
- [X] Terminal Linux
- [X] Tela de IPs
- [ ] CORAGEM
- [ ] TEMPO

## Tabelas de Configurações de Rede

Tabela 1: Definições da rede interna

| DESCRIÇÃO   | IP            |
|:------------|:------------- |
| Rede        | 10.9.14.0     |
| Máscara     | 255.255.255.0 |
| VirtualBox (gateway)     | 10.9.14.1      |
| Broadcast   | 10.9.14..255  |
| ns1         | 10.9.14.126   |
| samba       | 10.9.14.126   |

Tabela 2: Definições da rede externa

|  DESCRIÇÃO  |       IP      |
|:------------|:------------- |
| Rede        | 192.168.0.201 |
| Máscara     | 255.255.255.0 |
| VirtualBox (gateway)     | 192.168.0.1 |
| Broadcast   | 192.168.0.255 |

Tabela 3: Nomes dos servidores

|    Nome da VM     |                       NOME                           |
|:------------------|:-----------------------------------------------------|
| Gateway (gw)      | gw.pedro_filipe_914.labredes.ifalarapiraca.local     |
| NameServer1 (ns1) | ns1.pedro_filipe_914.labredes.ifalarapiraca.local    |
| NameServer2 (ns2) | ns2.pedro_filipe_914.labredes.ifalarapiraca.local    |
| Samba-SRV.        | samba.pedro_filipe_914.labredes.ifalarapiraca.local  |

### Configuração da interface de rede 
(Parte1)

* [Para realizar a configuração de IP estático e DNS ma interface de rede](https://github.com/0rmindo/SRed-2021/blob/main/parte1/readme.md)

### Configuração do gateway server/NAT 
(Parte2)

* [Para realizar a configuração de um servidor de gateway com Iptables/NAT](https://github.com/0rmindo/SRed-2021/blob/main/parte2/readme.md)

### Configuração do SAMBA 
(Parte3)

* [Para realizar a configuração de um servidor de arquivos com Samba](https://github.com/0rmindo/SRed-2021/blob/main/parte3/readme.md)

### Configuração do Bind9 (DNS Server) 
(Parte4)

* [Para realizar a configuração os servidores DNS Master e Slave](https://github.com/0rmindo/SRed-2021/blob/main/parte4/readme.md)
