## CONFIGURAÇÃO DO SAMBA

Vamos definir o nome da VM como ns1
```bash
 sudo hostnamectl ns1
 ```
 ```bash
 reboot
```
OBS: após o reboot o nome da máquina aparecerá no prompt do shell


   **1. Vamos instalar o servidor samba na VM**
   _Primeiro faremos o update depois instalamos o samba_
   
```bash
 sudo apt update
 ```
 ```bash
 sudo apt install samba
```
   
   **2. Verfificar se o samba está rodando**

```bash
 whereis samba
```
```
RESULTADO
samba: /usr/sbin/samba /usr/lib/x86_64-linux-gnu/samba /etc/samba /usr/share/samba /usr/share/man/man8/samba.8.gz /usr/share/man/man7/samba.7.gz
```
```bash
 sudo systemctl status smbd
```
```
RESULTADO
● smbd.service - Samba SMB Daemon
     Loaded: loaded (/lib/systemd/system/smbd.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-03-22 23:07:17 UTC; 1h 26min ago
       Docs: man:smbd(8)
             man:samba(7)
             man:smb.conf(5)
    Process: 691 ExecStartPre=/usr/share/samba/update-apparmor-samba-profile (code=exited, status=0/SUCCESS)
   Main PID: 697 (smbd)
     Status: "smbd: ready to serve connections..."
      Tasks: 4 (limit: 460)
     Memory: 17.5M
     CGroup: /system.slice/smbd.service
             ├─697 /usr/sbin/smbd --foreground --no-process-group
             ├─737 /usr/sbin/smbd --foreground --no-process-group
             ├─738 /usr/sbin/smbd --foreground --no-process-group
             └─739 /usr/sbin/smbd --foreground --no-process-group

```
```bash
 netstat -an | grep LISTEN
tcp        0      0 0.0.0.0:445             0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:139             0.0.0.0:*               LISTEN   
```

  **3. Façao backup do arquivo de configuração do samba e cria um arquivo novo somente com os comandos necessários.**
  
```bash
 sudo cp /etc/samba/smb.conf{,.backup}
 ```
 
   **4. Aqui vamos verificar se foi feito o backup do arquivo "smb.conf".**
    
 ```bash
 ls -la /etc/samba
-rw-r--r--  1 root root 8942 Mar 22 20:55 smb.conf
-rw-r--r--  1 root root 8942 Mar 23 01:42 smb.conf.backup
```

   **5. Agora use esse comando**
```bash
sudo bash -c 'grep -v -E "^#|^;" /etc/samba/smb.conf.backup | grep . > /etc/samba/smb.conf'
```

   **6. Agora vamos acessar o as configurações do smnb**

```bash
 sudo nano /etc/samba/smb.conf
[global]
   workgroup = WORKGROUP
   server string = %h server (Samba, Ubuntu)
   log file = /var/log/samba/log.%m
   max log size = 1000
   logging = file
   panic action = /usr/share/samba/panic-action %d
   server role = standalone server
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
   pam password change = yes
   map to guest = bad user
   usershare allow guests = yes
[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700
[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
```


  
  
  **7. Edite o arquivo de configuração /etc/samba/smb.conf**


```bash
 sudo nano /etc/samba/smb.conf
```

  * Adicione as seguintes linhas no diretorio [global] 

```bash
  interfaces = 127.0.0.1/8 ens160 ens192
  netbios name = ns1
```

  * Adicone também a parte do diretório [publico] no final do aquivo


```bash
[public]
   comment = public anonymous access
   path = /samba/public
   browsable =yes
   create mask = 0660
   directory mask = 0771
   writable = yes
   guest ok = no
   valid users = @sambashare
   #guest only = yes
   #force user = nobody
   #force create mode = 0777
   #force directory mode = 0777
```
   
   * Renicie o serviço smbd
    
```bash
 sudo systemctl restart smbd
```

   **8. modifica a pasta /samba/public para acesso a somente usuários do grupo sambashare**
 
   a. Crie um usuário do S.O para que possa utilizar o compartilhamento samba:
   * usuário: aluno
   * senha: alunoifal
   
   _No meu caso o usario "aluno' já estava criado, coso não esteja crie_
    
   **9. Vamos criar o usuário "aluno"**
   
```bash
 sudo adduser aluno
```

   * É necessário vincular o usuário do S.O. ao Serviço Samba. Repita a senha de aluno ou crie uma senha nova somente para acessar o compartilhamento de arquivo. 
   * Neste caso repetiremos a senha do usuário "aluno" senha "adminifal"
    
```bash
 sudo smbpasswd -a aluno
New SMB password:
Retype new SMB password:
Added user aluno.
```

   **10. Agora vamos alterar o GID do grupo padrão do usuário "aluno" para "sambashare"**

```bash
 sudo usermod -aG sambashare aluno

```
    
   * O Samba já está instalado, agora precisamos criar um diretório para compartilhá-lo em rede.
    
   _Aqui vamos criar o diretório sambashare dentro de home e de aluno_
      
```bash
 sudo mkdir /home/aluno/sambashare/
```

   _Aqui vamos criar o diretório publico dentro do samba_

```bash
 sudo mkdir -p /samba/publico
```

   * configure as permissões para que qualquer um possa acessar o compartilhamento público.

```bash
sudo chown -R nobody:nogroup /samba/public
```
```bash
sudo chmod -R 0775 /samba/public
```
```bash
sudo chgrp sambashare /samba/public
```

   **11. Cliente do compartilhamento:**
   
   * Em um máquina com Windows (também pode usar linux os MacOS) digite no Winndows Explorer o endereço IP do servidor samba da seguinte forma:
    **\\ip_do_maquina**. No meu caso: \\10.9.14.126
