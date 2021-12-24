## Configuração do Bind9 (DNS Server)
   

   **1. Vamos instalar o bind9 via apt-get**
   _Primeiro faremos o update depois instalamos o samba_

```bash
 sudo apt-get install bind9 dnsutils bind9-doc 
```
   a. Verifique o status do serviço:
   
```bash
 sudo systemctl status bind9
```
   b. Se não estiver rodando:
```bash
$ sudo systemctl enable bind9
```
>primeira e segunda parte da imagem


## Diretórios do bind
   * Os arquivos do bind ficam na no diretório **/etc/bind**.
   * para verificar digite 
```bash
 ls -la /etc/bind
```
```
total 64
drwxr-sr-x  3 root bind 4096 Oct 15 09:06 .
drwxr-xr-x 94 root root 4096 Oct  8 23:29 ..
-rw-r--r--  1 root root 2761 Aug  7 14:43 bind.keys
-rw-r--r--  1 root root  237 Aug  7 14:43 db.0
-rw-r--r--  1 root root  271 Aug  7 14:43 db.127
-rw-r--r--  1 root root  237 Aug  7 14:43 db.255
-rw-r--r--  1 root root  353 Aug  7 14:43 db.empty
-rw-r--r--  1 root root  270 Aug  7 14:43 db.local
-rw-r--r--  1 root root 3171 Aug  7 14:43 db.root
-rw-r--r--  1 root bind  463 Aug  7 14:43 named.conf
-rw-r--r--  1 root bind  490 Aug  7 14:43 named.conf.default-zones
-rw-r--r--  1 root bind  468 Oct 15 05:42 named.conf.local
-rw-r--r--  1 root bind  881 Sep 19 15:08 named.conf.options
-rw-r-----  1 bind bind   77 Sep 19 14:48 rndc.key
-rw-r--r--  1 root root 1317 Aug  7 14:43 zones.rfc1918
```
> Terceira parte da imagem

![install bind9 & ls -la](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/18.jpg)


### Zonas
   * As zonas são especificadas em arquivos **db**. Vamos criar um diretório para armazendar os arquivos de zonas, que sera o diretório ***/etc/bind/zones***  
   
   2. Vamos criar o diretório zones para adicionar os arquivos "db" dentro de /etc/bind.
```bash
 sudo mkdir /etc/bind/zones
```
![mkdir /etc/bind/zones](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/19.jpg)


   3. Criar arquivos db
   * Criar o arquivo **db** no diretório ***/etc/bind/zones***. 
   * Os arquivos **db** são bancos de dados de resolução de nomes, ou seja, quando se sabe o nome da máquina mas não se conhece o IP. Cada zona no DNS deve ter seu próprio arquivo **db**, por exemplo: a zona *meusite.com.br* terá o arquivo **db.meusite.com.br**, já a zona *outrosite.net* terá o arquivo **db.outrosite.net**. 
   * No nosso caso o domínio/zona local será labredes.ifalarapiraca.local. Assim o arquivo db será db.labredes.ifalarapiraca.local
   
##### zona direta
   * O arquivo db.labredes.ifalarapiraca.local conterá os nomes das máquinas do domínio labredes.ifalarapiraca.local
   * Para isso faremos uma cópia do arquivo /etc/bind/db.empty

```bash
 sudo cp /etc/bind/db.empty /etc/bind/zones/db.labredes.ifalarapiraca.local 
```
![cp /etc/bind/db.empty](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/20.jpg)


##### zona reversa
   * Utilizado quando não se conhece o IP mas sabe-se o nome do host.
   * vamos criar a zona reversa a partir do arquivo /etc/bind/db.127

```bash
 sudo cp /etc/bind/db.127 /etc/bind/zones/db.10.9.14.rev
```
![cp /etc/bind/db.127](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/21.jpg)


   * Assim, o arquivo **db.10.9.14.rev** conterá a zona reversa da rede 10.9.14.0. 

   
### Editar arquivos db:

   #### zona direta: db.labredes.ifalarapiraca.local
   * edite o arquivo  **db.labredes.ifalarapiraca.local** para adcionar as informações do seu domínio
      * As linhas iniciadas com **;** são comentários 
      
```bash   
 sudo nano /etc/bind/zones/db.labredes.ifalarapiraca.local 
```
```
RESULTADO
;
; BIND data file for internal network
;
$ORIGIN labredes.ifalarapiraca.local.
$TTL	3h
@	IN	SOA	ns1.pedro_felipe_914.labredes.ifalarapiraca.local. root.pedro_felipe_914.labredes.ifalarapiraca.local. (
			      1		; Serial
			      3h	; Refresh
			      1h	; Retry
			      1w	; Expire
			      1h )	; Negative Cache TTL
;nameservers
@	IN	NS	ns1.pedro_felipe_914labredes.ifalarapiraca.local.

;hosts
ns1.pedro_felipe_914.labredes.ifalarapiraca.local.	  IN	A	10.9.14.126
srv.pedro_felipe_914.labredes.ifalarapiraca.local.	  IN	A	10.9.14.126
gw.pedro_felipe_914.labredes.ifalarapiraca.local.	  IN 	A	10.9.14.1          
desktophost1    CNAME     srv                 ; CNAME é um apelido
```
![nano /etc/bind/zones/db.labredes.ifalarapiraca.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/22.jpg)


   #### zona reversa: db.10.9.14.rev
   * edite o arquivo **db.10.9.14.rev** para adcionar as informações da zona reversa
      * As linhas iniciadas com **;** são comentários.


```bash   
 sudo nano /etc/bind/zones/db.10.9.14.rev
```   
```
RESULTADO
;
; BIND reverse data file of reverse zone for local area network 10.9.14.0/24
;
$TTL    604800
@       IN      SOA     pedro_felipe_914.labredes.ifalarapiraca.local. root.pedro_felipe_914.labredes.ifalarapiraca.local. (
                              1         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL

; name servers
@      IN      NS      ns1.pedro_felipe_914.labredes.ifalarapiraca.local.

; PTR Records
126   IN      PTR     ns1.pedro_felipe_914.labredes.ifalarapiraca.local.              ; 10.9.14.10
126  IN      PTR     srv.pedro_felipe_914.labredes.ifalarapiraca.local.    	         ; 10.9.14.100
1    IN      PTR     gw.pedro_felipe_914.labredes.ifalarapiraca.local.               ; 10.9.14.1
```
![nano /etc/bind/zones/db.10.9.14.rev](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/23.jpg)

   ### Configuração do named.conf.local
   * Para ativar as zonas descritas nos arquivos **db** deve-se editar o arquivo de configuracão do bind para informar onde eles foram salvos. As zonas são adicionadas em **/etc/bind/named.conf.local**.
   
```bash
 sudo nano /etc/bind/named.conf.local
```
```
RESULTADO
//
// Do any local configuration here
//

// Consider adding the 1918 zones here, if they are not used in your
// organization
//include "/etc/bind/zones.rfc1918";

zone "labredes.ifalarapiraca.local" {
	type master;
	file "/etc/bind/zones/db.labredes.ifalarapiraca.local";
	allow-transfer{ 10.9.14.126; };  
	allow-query{any;};
};

zone "14.9.10.in-addr.arpa" IN {
	type master;
	file "/etc/bind/zones/db.10.9.14.rev";
	allow-transfer{ 10.9.14.126; };
};
```
![nano /etc/bind/named.conf.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/24.jpg)


   ### Verificação de sintaxe 
   
   * Para checar a sintaxe de configuração do BIND deve-se executar o comando named-checkconf. Este scritp checa os arquivos /etc/bind/named.conf.local.*

```bash
 sudo named-checkconf
```

   ###  Verificar a sintaxe dos arquivos de dados
   
   * Para verificar se a formatação da sintaxe dos arquivos db está correta, utiliza-se o script named-checkconf da seguinte forma: 
   
   * Vamos entrar no diretório zones

```bash
 cd /etc/bind/zones
```

   * Agora digite os comandos e veja se está ok
  
```bash
 sudo named-checkzone labredes.ifalarapiraca.local db.labredes.ifalarapiraca.local
```
```bash
RESULTADO
...
zone labredes.ifalarapiraca.local/IN: loaded serial 1
OK
```
```
 sudo named-checkzone 14.9.10.in-addr.arpa db.10.9.14.rev
```
```bash
RESULTADO
...	
zone 14.9.10.in-addr.arpa/IN: loaded serial 1
OK
```
![nano /etc/bind/named.conf.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/25.jpg)


### Configure para somente resolver endereços IPv4

```bash
 sudo nano /etc/default/named
```
   a. Adicione o -4 na seguinte linha
  ***OPTIONS="-u bind"***
  
```bash
RESULTADO
# run resolvconf?
RESOLVCONF=no

# startup options for the server
OPTIONS="-4 -u bind"
```
![nano /etc/bind/named.conf.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/26.jpg)

### Vamos reiniciar o BIND 

```bash
 sudo systemctl restart bind9
```

### Configuração dos clientes
   * Configure o DNS na máquina ns1, adicionando o IP no campo adresses de name server, onde estava o IP do google, interface de rede local (ens160).


```
            nameservers: 
                addresses:
                - 10.9.14.10
                search: [pedro_filipe_914.labredes.ifalarapiraca.local]
```
  
   * O arquivo de configuração do netplan ficará da seguinte forma:

```bash
 sudo nano /etc/netplan/00-installer-config.yaml
```
```bash
RESULTADO
network:
    ethernets:
        ens160:                           # interface local
            addresses: [10.9.14.126/24]   # ip/mascara
            gateway4: 10.9.14.1           # ip do gateway
            dhcp4: false                  # 'false' para conf. estatica 
            nameservers:                  # servidores dns
                addresses:
                - 10.9.14.126             # ip do ns1
                search: [pedro_filipe_194.labredes.ifalarapiraca.local]  # domínio
        ens192:                           # nome da interface que está sendo configurada. Verifique com o comando 'ifconfig -a'
            addresses: [192.168.0.201/25] # IP e Máscara de interface externa.
    version: 2
```
![nano /etc/bind/named.conf.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/27.jpg)

   * O campo search indica o nome do domínio no qual a máquina pertence.
   
### Testando o servidor DNS:

#### Teste de configuração como cliente. 
   * Observe se os campos **DNS servers** e **DNS Domain** estão corretos.
  
```bash
 systemd-resolve --status ens160
```
```
RESULTADO
Link 2 (ens160)
      Current Scopes: DNS
       LLMNR setting: yes
MulticastDNS setting: no
      DNSSEC setting: no
    DNSSEC supported: no
         DNS Servers: 10.9.14.10
                      10.9.14.11
         DNS Domain: pedro_filipe_194.labredes.ifalarapiraca.local
```
> Parte 1 da imagem

   ### Para finalizar é só ver se o nosso serviço DNS resolve o DNS do Google
  
```bash
  ping google.com
```
> Parte 2 a imagem
![nano /etc/bind/named.conf.local](https://github.com/0rmindo/SRed-2021/blob/main/imaegens/29.jpg)


### [VOLTAR PARA HOME](https://github.com/0rmindo/SRed-2021/blob/main/README.md)
