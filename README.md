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
| ns2 (opcional) | 10.9.14.109 |
| samba       | 10.9.14.100   |

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
