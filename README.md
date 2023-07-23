# Criando usuários com useradd e adduser
O comando `useradd <nome_usuario>`, sem parâmetro, cria o usuário, mas não define senhas ou outras informações.
```
thiago@thiago-pc:~$ sudo useradd thiago2
[sudo] password for thiago:
thiago@thiago-pc:~$
```
Já o comando `adduser <nome_usuario>` é um script que:
* cria o usuário com o comando `useradd`;
* cria um diretório dentro de `/home` com o mesmo nome do usuário;
* cria o bash do usuário e o perfil;
* cria um grupo com o mesmo nome do usuário, e coloca o novo usuário nesse grupo;
* força a definição da senha e informações pessoais como nome completo, telefone etc.
```
thiago@thiago-pc:~$ sudo adduser thiago3
Adding user `thiago3' ...
Adding new group `thiago3' (1002) ...
Adding new user `thiago3' (1002) with group `thiago3' ...
Creating home directory `/home/thiago3' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for thiago3
Enter the new value, or press ENTER for the default
        Full Name []: Thiago Marçal
        Room Number []: 303
        Work Phone []: 99959-1984
        Home Phone []:
        Other []:
chfn: name with non-ASCII characters: 'Thiago Marçal'
Is the information correct? [Y/n] n
Changing the user information for thiago3
Enter the new value, or press ENTER for the default
        Full Name [Thiago Marçal]: Thiago Marcal
        Room Number [303]:
        Work Phone [99959-1984]:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
thiago@thiago-pc:~$ ls /home/
thiago  thiago3
thiago@thiago-pc:~$
```

Em scripts o melhor seria usar o `useradd` com parâmetros. Mas para criação rápida de usuários, use o comando `adduser`.


Veja como fica o arquivo /etc/passwd depois da execução desses comandos:
```
thiago@thiago-pc:~$ cat /etc/passwd | grep thiago
thiago:x:1000:1000:Thiago:/home/thiago:/bin/bash
thiago2:x:1001:1001::/home/thiago2:/bin/sh
thiago3:x:1002:1002:Thiago Marcal,303,99959-1984,:/home/thiago3:/bin/bash
thiago@thiago-pc:~$
```
> Na maioria das distribuições Linux, os usuários de ID maiores ou iguais a 1000 são usuários comuns. Abaixo desse número estão os IDs dos usuários de sistema, o que inclui o root, de ID zero.

Repare que o path para os bashes dos usuários `thiago2` e `thiago3` são diferentes: `thiago2` usa o terminal padrão `/bin/sh`, enquanto `thiago3` usa o terminal bash `/bin/bash`.

O arquivo `/etc/group` lista os grupos existentes, seguidos dos números dos IDs dos usuários que pertencem a esse grupo. Note que os usuários `thiago2` e `thiago3` não estão no grupo `sudo`.

> No arquivo `/etc/group` temos o nome do grupo, um `x`, o ID do grupo e em seguida os nomes de usuários que participam do grupo.

```
thiago@thiago-pc:~$ cat /etc/group | grep thiago
adm:x:4:syslog,thiago
cdrom:x:24:thiago
sudo:x:27:thiago
dip:x:30:thiago
plugdev:x:46:thiago
lxd:x:110:thiago
thiago:x:1000:
thiago2:x:1001:
thiago3:x:1002:
thiago@thiago-pc:~$
```

# Incluindo usuários no sistema
O nosso sistema precisa armazenar, de algum modo, as senhas dos usuários para realizar a verificação de acesso e permitir que eles tenham acesso ao sistema.

Qual arquivo do Linux é utilizado para armazenar o hash das senhas dos usuários?

> `/etc/shadow`: O `/etc/shadow` armazena o hash das senhas. É importante destacar que nenhum sistema moderno deve realizar o armazenamento de senhas.

* O `/etc/passwd` corresponde ao banco de dados de usuários locais do sistema.
* O `/etc/sudoers` armazena as configurações / permissões para a execução do sudo.

# Adicionando grupos aos usuários

Veja no arquivo `/etc/shadow` que o usuário `thiago2` não tem um hash de senha atribuído.

```
thiago@thiago-pc:~$ sudo cat /etc/shadow | grep thiago
[sudo] password for thiago:
thiago:$y$j9T$vLBito6FAioyIWXF53UzP0$B4rTIqADEMAu8Mkbz.Jad2/VMsxjeLg1HdAhioyTj51:19487:0:99999:7:::
thiago2:!:19560:0:99999:7:::
thiago3:$y$j9T$4PIHUY4hsR5PtzlTaUGVG1$yMY3/0dMNeFAP70wj5AqlNAUWX5k./.3lq9/uOmfuZ1:19560:0:99999:7:::
thiago@thiago-pc:~$
```

O comando `groups` mostra os grupos aos quais o usuário atual pertence (compare com o arquivo `/etc/group`):
```
thiago@thiago-pc:~$ groups
thiago adm cdrom sudo dip plugdev lxd
thiago@thiago-pc:~$ groups thiago2
thiago2 : thiago2
thiago@thiago-pc:~$ groups thiago3
thiago3 : thiago3
```
## Trocando de usuário usando o comando `su - <usuario>`
Note que o usuário no terminal muda de `thiago` para `thiago3`:
```
thiago@thiago-pc:~$ su - thiago3
Password:
thiago3@thiago-pc:~$
```

Como o usuário `thiago2` não tem senha, não conseguiríamos logar com ele:
```
thiago3@thiago-pc:~$ su - thiago2
Password:
su: Authentication failure
```

## Criando grupos
Use o comando `groupadd <nome do grupo>` para criar um grupo:

```
thiago@thiago-pc:~$ sudo groupadd projetos
[sudo] password for thiago:
thiago@thiago-pc:~$ cat /etc/group | grep projetos
projetos:x:1003:
```
## Atribuindo usuários a grupos
use o comando `usermod -aG <lista de grupos> <usuario>`. 
> Note que o `G` é maiúsculo, serve para acrescentar o usuário no grupo, não para trocar o usuário de grupo principal (que é o caso do parâmetro com `g` minúsculo).

```
thiago@thiago-pc:~$ sudo usermod -aG projetos thiago3
thiago@thiago-pc:~$ groups thiago3
thiago3 : thiago3 projetos
thiago@thiago-pc:~$
```
Note que o usuário `thiago3` agora está no grupo `thiago3` e no grupo `projetos`.

É possível editar o arquivo `/etc/group` diretamente e acrescentar os usuários logo após os dois pontos no final de cada grupo:
```
thiago@thiago-pc:~$ sudo groupadd desenvolvedores
thiago@thiago-pc:~$ vi /etc/group

... dentro do vi ...
...
projetos:x:1003:thiago3
desenvolvedores:x:1004:
```
Mas fazer isso é arriscado, o funcionamento dos grupos pode ficar comprometido.

## Removendo usuários e grupos
Use os comandos `userdel` e `groupdel`:
```
thiago@thiago-pc:~$ sudo userdel thiago2
thiago@thiago-pc:~$ cat /etc/passwd | grep thiago
thiago:x:1000:1000:Thiago:/home/thiago:/bin/bash
thiago3:x:1002:1002:Thiago Marcal,303,99959-1984,:/home/thiago3:/bin/bash
thiago@thiago-pc:~$ sudo groupdel desenvolvedores
thiago@thiago-pc:~$ cat /etc/group | grep des
thiago@thiago-pc:~$
thiago@thiago-pc:~$ cat /etc/group | grep thiago
adm:x:4:syslog,thiago
cdrom:x:24:thiago
sudo:x:27:thiago
dip:x:30:thiago
plugdev:x:46:thiago
lxd:x:110:thiago
thiago:x:1000:
thiago3:x:1002:
projetos:x:1003:thiago3
thiago@thiago-pc:~$
```
# Entendendo o permissionamento dos arquivos e diretórios
Regra: todo arquivo ou diretório pertence a um único usuário e a um único grupo.
```
thiago@thiago-pc:~$ touch novoteste
thiago@thiago-pc:~$ ls -l novo*
-rw-rw-r-- 1 thiago thiago 0 jul 22 16:08 novoteste
```
Note que o usuário `thiago` criou o arquivo `novoteste`, e ele pertence ao usuário `thiago` e ao grupo `thiago`.

Veja como isso se aplica ao diretório `/var/local`:
```
thiago@thiago-pc:~$ ls -l /var
total 48
drwxr-xr-x  2 root root   4096 mai 10 00:00 backups
drwxr-xr-x 15 root root   4096 mai  9 23:56 cache
drwxrwxrwt  2 root root   4096 fev 17 17:23 crash
drwxr-xr-x 42 root root   4096 mai  9 23:56 lib
drwxrwsr-x  2 root staff  4096 abr 18  2022 local
lrwxrwxrwx  1 root root      9 fev 17 17:19 lock -> /run/lock
drwxrwxr-x 10 root syslog 4096 jul 22 14:52 log
drwxrwsr-x  2 root mail   4096 fev 17 17:19 mail
drwxr-xr-x  2 root root   4096 fev 17 17:19 opt
lrwxrwxrwx  1 root root      4 fev 17 17:19 run -> /run
drwxr-xr-x  5 root root   4096 fev 17 17:25 snap
drwxr-xr-x  4 root root   4096 fev 17 17:24 spool
drwxrwxrwt  7 root root   4096 jul 22 15:53 tmp
drwxr-xr-x  3 root root   4096 mai  9 23:56 www
```
O diretório `/var/local` é do usuário `root`, mas pertence ao grupo `staff`.

Observe o padrão de 10 letras no início: o primeiro caractere identifica se o objeto se trata de um diretório (d) ou de um arquivo (-). Os grupos de 3 letras seguintes configuram acesso de leitura (r), escrita (w) e execução/permissão para entrar no diretório (x). A primeira tríade de letras é para o usuário; a segunda tríade é para o grupo; e a terceira tríade é para qualquer usuário.

As permissões podem ser representadas numericamente pela base octal (1 a 7):
```
7 = rwx (lê, escreve e executa/abre diretório)
6 = rw- (lê e escreve)
5 = r-x (lê e executa/abre diretório)
4 = r-- (só lê)
3 = -wx (escreve e executa/abre diretório)
2 = -w- (só escreve)
1 = --x (só executa/abre diretório)
0 = --- (sem permissão)
```

# Alterando as permissões, donos e grupos
Cenário: criação do diretório `/projetos`:
```
thiago@thiago-pc:~$ mkdir /projetos
mkdir: cannot create directory ‘/projetos’: Permission denied
thiago@thiago-pc:~$ sudo mkdir /projetos
thiago@thiago-pc:~$ sudo mkdir /projetos
[sudo] password for thiago:
thiago@thiago-pc:~$ ls -l /
total 2097224
lrwxrwxrwx   1 root root          7 fev 17 17:19 bin -> usr/bin
...
dr-xr-xr-x 176 root root          0 jul 22 14:51 proc
drwxr-xr-x   2 root root       4096 jul 22 16:43 projetos
...
drwxr-xr-x  14 root root       4096 mai  9 23:56 var
thiago@thiago-pc:~$
```
## Comando `chmod`
Vamos limitar a leitura, escrita e execução do diretório `/projetos` apenas ao grupo ao dono e ao grupo do diretório com o comando `chmod`:
```
thiago@thiago-pc:~$ chmod 770 /projetos/
chmod: changing permissions of '/projetos/': Operation not permitted
thiago@thiago-pc:~$ sudo chmod 770 /projetos/
thiago@thiago-pc:~$ ls -l /
total 2097224
...
drwxrwx---   2 root root       4096 jul 22 16:43 projetos
...
thiago@thiago-pc:~$
```
Com essa configuração, somente o usuário/dono `root` ou algum usuário do grupo `root` pode acessar o diretório:
```
thiago@thiago-pc:~$ cd /projetos/
-bash: cd: /projetos/: Permission denied
thiago@thiago-pc:~$
```
Vamos acrescentar o usuário `thiago3` ao grupo `projetos`:
```
thiago@thiago-pc:~$ sudo usermod -aG projetos thiago3
thiago@thiago-pc:~$ groups thiago3
thiago3 : thiago3 projetos
thiago@thiago-pc:~$
```
E acrescentar o usuário `thiago4`:
```
thiago@thiago-pc:~$ sudo adduser thiago4
Adding user `thiago4' ...
Adding new group `thiago4' (1001) ...
Adding new user `thiago4' (1001) with group `thiago4' ...
Creating home directory `/home/thiago4' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for thiago4
Enter the new value, or press ENTER for the default
        Full Name []: Thiago Marcal
        Room Number []:
        Work Phone []:
        Home Phone []:
        Other []:
Is the information correct? [Y/n] y
thiago@thiago-pc:~$
```
## Comando `chown`
Vamos mudar o dono do diretório `/projetos` com o comando `chown <novo dono> <diretório/arquivo>`:

```
thiago@thiago-pc:~$ sudo chown thiago /projetos/
thiago@thiago-pc:~$ ls -l /
total 2097224
...
drwxrwx---   2 thiago root       4096 jul 22 16:43 projetos
...
thiago@thiago-pc:~$
```

## Comando `chgrp`
Para mudar o grupo, use o comando `chgrp`:
```
thiago@thiago-pc:~$ sudo chgrp thiago3 /projetos/
thiago@thiago-pc:~$ ls -l /
total 2097224
...
drwxrwx---   2 thiago thiago3       4096 jul 22 16:43 projetos
...
thiago@thiago-pc:~$
```
Mas o caminho mais rápido é separar o usuário e o grupo com dois pontos dentro do comando `chown`:
```
thiago@thiago-pc:~$ sudo chown thiago:projetos /projetos/
thiago@thiago-pc:~$ ls -l /
total 2097224
...
drwxrwx---   2 thiago projetos       4096 jul 22 16:43 projetos
...
thiago@thiago-pc:~$
```

## Cenário: criação de um arquivo na pasta `/projetos`
```
thiago@thiago-pc:~$ echo "projeto da nasa" > /projetos/proj1
thiago@thiago-pc:~$ cd /projetos/
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw-r-- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$
```

## Cenário: os usuários thiago3 e thiago4 podem (ou não) acessar a pasta `/projetos`
O usuário `thiago3` é parte do grupo `projetos`, mas o usuário `thiago4` não faz parte:
```
thiago@thiago-pc:/projetos$ groups thiago3
thiago3 : thiago3 projetos
thiago@thiago-pc:/projetos$ groups thiago4
thiago4 : thiago4
thiago@thiago-pc:/projetos$
```

# Permissionamentos restritivos e especiais
Os acessos mais **restritivos** são os que prevalecem.

O usuário `thiago` criou um arquivo que pode ser lido por qualquer usuário (octal 664):
```
thiago@thiago-pc:/projetos$ ls -l /projetos/
total 4
-rw-rw-r-- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$ groups thiago3
thiago3 : thiago3 projetos
thiago@thiago-pc:/projetos$ su - thiago3
Password:
thiago3@thiago-pc:~$ cd /projetos/
thiago3@thiago-pc:/projetos$ ls -l
total 4
-rw-rw-r-- 1 thiago thiago 16 jul 22 17:08 proj1
thiago3@thiago-pc:/projetos$ cat proj1
projeto da nasa
thiago3@thiago-pc:/projetos$
logout
```
Removendo a permissão de leitura para quem não for owner ou não for do grupo do arquivo `/projetos/proj1` (octal 660):
```
thiago@thiago-pc:/projetos$ sudo chmod 660 proj1
[sudo] password for thiago:
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw---- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$ su - thiago3
Password:
thiago3@thiago-pc:~$ cat /projetos/proj1
cat: /projetos/proj1: Permission denied
thiago3@thiago-pc:~$
logout
```
## Usando letras ao invés de números ao aplicar o comando `chmod`
|Comando|Owner| Group | Others/Everyone | As 3 colunas|
|---|---|---|---|---|
|Remover Leitura| chmod u-r | chmod g-r| chmod o-r| chmod a-r|
|Adicionar Leitura| chmod u+r | chmod g+r| chmod o+r| chmod a+r|
|Remover Escrita| chmod u-w | chmod g-w| chmod o-w| chmod a-w|
|Adicionar Escrita| chmod u+w | chmod g+w| chmod o+w| chmod a+w|
|Remover Execução| chmod u-x | chmod g-x| chmod o-x| chmod a-x|
|Adicionar Execução| chmod u+x | chmod g+x| chmod o+x| chmod a+x|

> Generalizando: `user@host:~$ chmod [ugoa][+-][rwx] arquivo`

Adicionando acesso de leitura aos outros usuários:
```
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw---- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$ sudo chmod o+r proj1
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw-r-- 1 thiago thiago 16 jul 22 17:08 proj1

thiago@thiago-pc:/projetos$ su - thiago3
Password:
thiago3@thiago-pc:~$ cd /projetos/
thiago3@thiago-pc:/projetos$ ls -l
total 4
-rw-rw-r-- 1 thiago thiago 16 jul 22 17:08 proj1
thiago3@thiago-pc:/projetos$ cat proj1
projeto da nasa
thiago3@thiago-pc:/projetos$
logout
```
Removendo acesso de leitura aos outros usuários:
```
thiago@thiago-pc:/projetos$ sudo chmod o-r proj1
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw---- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$

thiago@thiago-pc:/projetos$ su - thiago3
Password:
thiago3@thiago-pc:~$ cat /projetos/proj1
cat: /projetos/proj1: Permission denied
thiago3@thiago-pc:~$
```
## Usando o parâmetro `g+s` no comando `chmod`
Ao aplicarmos a um diretório o parâmetro `g+s` (não `g+x`) com o comando `chmod`, os novos arquivos criados dentro desse diretório recebem as mesmas permissões do grupo ao qual o diretório pertence:
```
thiago@thiago-pc:/projetos$ ls -l /projetos/
total 4
-rw-rw---- 1 thiago thiago 16 jul 22 17:08 proj1
thiago@thiago-pc:/projetos$ touch proj2
thiago@thiago-pc:/projetos$ ls -l
total 4
-rw-rw---- 1 thiago thiago 16 jul 22 17:08 proj1
-rw-rw-r-- 1 thiago thiago  0 jul 23 14:03 proj2
thiago@thiago-pc:/projetos$ sudo chmod g+s /projetos/
thiago@thiago-pc:/projetos$ ls -l /
total 2097224
...
drwxrwsr--   2 thiago projetos       4096 jul 23 14:03 projetos
...

thiago@thiago-pc:/projetos$ touch proj3
thiago@thiago-pc:/projetos$ ls -l /projetos/
total 4
-rw-rw---- 1 thiago thiago   16 jul 22 17:08 proj1
-rw-rw-r-- 1 thiago thiago    0 jul 23 14:03 proj2
-rw-rw-r-- 1 thiago projetos  0 jul 23 14:08 proj3
thiago@thiago-pc:/projetos$
```
> Note que o arquivo `proj3` tem as mesmas permissões de leitura/escrita/execução do diretório `/projetos`, além de pertencer ao grupo `projetos` (não ao grupo `thiago`).

# Links simbólicos e suas utilidades
Links simbólicos são atalhos para diretórios ou arquivos. Eles são criado com o parâmetro `-s` do comando `ln`:
```
thiago@thiago-pc:~$ ln -s /projetos projetos
thiago@thiago-pc:~$ ls -l
total 20
-rw-r--r--  1 thiago thiago  331 abr 22 14:28 00-installer-config.yaml
drwxrwxr-x  3 thiago thiago 4096 mai  9 15:24 backup
drwxrwxr-x  2 thiago thiago 4096 mai  9 15:36 bin
drwxrwxr-x 12 thiago thiago 4096 mai  9 14:52 labs
-rw-rw-r--  1 thiago thiago    0 jul 22 16:08 novoteste
lrwxrwxrwx  1 thiago thiago    9 jul 23 14:24 projetos -> /projetos
-rw-rw-r--  1 thiago thiago 2683 mai  9 14:47 teste.arq
thiago@thiago-pc:~$ cd projetos
thiago@thiago-pc:~/projetos$ pwd
/home/thiago/projetos
thiago@thiago-pc:~/projetos$ ls
proj1  proj2  proj3
thiago@thiago-pc:~/projetos$ ls /projetos/
proj1  proj2  proj3
thiago@thiago-pc:~/projetos$
```

> Perceba a letra `l` que aparece no lugar da letra `d` para diretório. Perceba também que do lado do nome do link simbólico está o caminho para o qual o link simbólico aponta: `projetos -> /projetos`.

Criação de um link simbólico chamado `blaha` para o arquivo `/projetos/proj1`:
```
thiago@thiago-pc:~/projetos$ ln -s /projetos/proj1 blaha
thiago@thiago-pc:~/projetos$ cat blaha
projeto da nasa
```
A remoção do link simbólico **não** remove o arquivo original.
```
thiago@thiago-pc:~/projetos$ rm blaha
thiago@thiago-pc:~/projetos$ cat /projetos/proj1
projeto da nasa
thiago@thiago-pc:~/projetos$ ln -s /projetos/proj1 blaha
thiago@thiago-pc:~/projetos$ ls -l
total 4
lrwxrwxrwx 1 thiago projetos 15 jul 23 14:28 blaha -> /projetos/proj1
-rw-rw---- 1 thiago thiago   16 jul 22 17:08 proj1
-rw-rw-r-- 1 thiago thiago    0 jul 23 14:03 proj2
-rw-rw-r-- 1 thiago projetos  0 jul 23 14:08 proj3
thiago@thiago-pc:~/projetos$
```

# Gerenciamento de pacote com o apt
Os comando `apt` e `apt-get` tem diferenças sutis, mas irrelevantes para o uso geral.

O comando `apt search` serve pesquisar nomes de pacotes nas descrições de pacote. Essas descrições são atualizadas com o comando `apt update`, mas não refletem necessariamente nos pacotes que estão instalados no computador.

O comando `apt show` mostra os detalhes de um determinado pacote.

O comando `apt list --installed` lista os pacotes instalados no computador.

# Informações e upgrade dos pacotes instalados
Os comandos `apt remove` e `apt autoremove` são diferentes: 
* `apt remove` exige o nome do pacote que será removido;
* `apt autoremove` não exige parâmetros, e apaga pacotes que não são dependências para outros e que não foram apagados com o comando `apt remove`.

```
thiago@thiago-pc:~$ sudo apt remove apache2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Os seguintes pacotes foram instalados automaticamente e já não são necessários:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1
  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0 ssl-cert
Utilize 'sudo apt autoremove' para os remover.
Serão REMOVIDOS os seguintes pacotes:
  apache2
0 pacotes actualizados, 0 pacotes novos instalados, 1 a remover e 122 não actualizados.
Após esta operação, será libertado 546 kB de espaço em disco.
Deseja continuar? [S/n]
```
Executando o `apt autoremove`
```
thiago@thiago-pc:~$ sudo apt autoremove
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
Serão REMOVIDOS os seguintes pacotes:
  apache2-bin apache2-data apache2-utils libapr1 libaprutil1
  libaprutil1-dbd-sqlite3 libaprutil1-ldap liblua5.3-0 ssl-cert
0 pacotes actualizados, 0 pacotes novos instalados, 9 a remover e 122 não actualizados.
Após esta operação, será libertado 7735 kB de espaço em disco.
Deseja continuar? [S/n]

(A ler a base de dados ... 74693 ficheiros e directórios actualmente instalados.)
A remover apache2-bin (2.4.52-1ubuntu4.5) ...
dpkg: aviso: ao remover apache2-bin, o directório '/var/lib/apache2' não estava vazio, por isso não foi removido
A remover apache2-data (2.4.52-1ubuntu4.5) ...
A remover apache2-utils (2.4.52-1ubuntu4.5) ...
A remover libaprutil1-dbd-sqlite3:amd64 (1.6.1-5ubuntu4.22.04.1) ...
A remover libaprutil1-ldap:amd64 (1.6.1-5ubuntu4.22.04.1) ...
A remover libaprutil1:amd64 (1.6.1-5ubuntu4.22.04.1) ...
A remover libapr1:amd64 (1.7.0-8ubuntu0.22.04.1) ...
A remover liblua5.3-0:amd64 (5.3.6-1build1) ...
A remover ssl-cert (1.1.2) ...
A processar 'triggers' para man-db (2.10.2-1) ...
A processar 'triggers' para libc-bin (2.35-0ubuntu3.1) ...
thiago@thiago-pc:~$
```

O comando `apt --` (com dois hifens) é um atalho para ajuda do comando `apt`:
```
thiago@thiago-pc:~$ apt --
apt 2.4.8 (amd64)
Usage: apt [options] command

apt is a commandline package manager and provides commands for
searching and managing as well as querying information about packages.
It provides the same functionality as the specialized APT tools,
like apt-get and apt-cache, but enables options more suitable for
interactive use by default.

Most used commands:
  list - list packages based on package names
  search - search in package descriptions
  show - show package details
  install - install packages
  reinstall - reinstall packages
  remove - remove packages
  autoremove - Remover automaticamente todos os pacotes não utilizados
  update - update list of available packages
  upgrade - upgrade the system by installing/upgrading packages
  full-upgrade - upgrade the system by removing/installing/upgrading packages
  edit-sources - edit the source information file
  satisfy - satisfy dependency strings

See apt(8) for more information about the available commands.
Configuration options and syntax is detailed in apt.conf(5).
Information about how to configure sources can be found in sources.list(5).
Package and version choices can be expressed via apt_preferences(5).
Security details are available in apt-secure(8).
                                   Este APT tem Poderes de Super Vaca.
thiago@thiago-pc:~$
```

O comando `apt upgrade` atualiza os pacotes instalados para as versões mais recentes. Cuidado ao fazer isso em servidores de produção: as aplicações pode ficar com problemas depois do upgrade.

Listando os pacotes atualizáveis com o nome `python3.10`:
```
thiago@thiago-pc:~$ apt list --upgradable | grep python3.10

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libpython3.10-minimal/jammy-updates,jammy-security 3.10.6-1~22.04.2ubuntu1.1 amd64 [upgradable from: 3.10.6-1~22.04.2ubuntu1]
libpython3.10-stdlib/jammy-updates,jammy-security 3.10.6-1~22.04.2ubuntu1.1 amd64 [upgradable from: 3.10.6-1~22.04.2ubuntu1]
libpython3.10/jammy-updates,jammy-security 3.10.6-1~22.04.2ubuntu1.1 amd64 [upgradable from: 3.10.6-1~22.04.2ubuntu1]
python3.10-minimal/jammy-updates,jammy-security 3.10.6-1~22.04.2ubuntu1.1 amd64 [upgradable from: 3.10.6-1~22.04.2ubuntu1]
python3.10/jammy-updates,jammy-security 3.10.6-1~22.04.2ubuntu1.1 amd64 [upgradable from: 3.10.6-1~22.04.2ubuntu1]
thiago@thiago-pc:~$
```
Upgrade dos pacotes (no exemplo temos 123 pacotes atualizáveis):
```
thiago@thiago-pc:~$ thiago@thiago-pc:~$ apt list --upgradable | wc -l

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

123


thiago@thiago-pc:~$ sudo apt upgrade
...

thiago@thiago-pc:~$ sudo apt list --installed | grep python3.10
[sudo] password for thiago:

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

libpython3.10-minimal/jammy-updates,jammy-security,now 3.10.6-1~22.04.2ubuntu1.1 amd64 [installed,automatic]
libpython3.10-stdlib/jammy-updates,jammy-security,now 3.10.6-1~22.04.2ubuntu1.1 amd64 [installed,automatic]
libpython3.10/jammy-updates,jammy-security,now 3.10.6-1~22.04.2ubuntu1.1 amd64 [installed,automatic]
python3.10-minimal/jammy-updates,jammy-security,now 3.10.6-1~22.04.2ubuntu1.1 amd64 [installed,automatic]
python3.10/jammy-updates,jammy-security,now 3.10.6-1~22.04.2ubuntu1.1 amd64 [installed,automatic]
thiago@thiago-pc:~$
```
> Note que onde estava escrito `upgradable`, agora está escrito `installed`.

# Consultando a base instalada com o apt list
Consultando se há algum pacote atualizável:
```
thiago@thiago-pc:~$ sudo apt list --upgradable
Listing... Pronto
thiago@thiago-pc:~$
```
Gerando a lista dos pacotes instalados:
```
thiago@thiago-pc:~$ sudo apt list | grep installed > lista-pacotes.txt

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

thiago@thiago-pc:~$ ls lista*
lista-pacotes.txt
thiago@thiago-pc:~$
```
Outra forma de gerar a lista:
```
thiago@thiago-pc:~$ sudo apt list --installed > lista2.txt

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

thiago@thiago-pc:~$ ls -l lista*
-rw-rw-r-- 1 thiago thiago 44622 jul 23 16:10 lista2.txt
-rw-rw-r-- 1 thiago thiago 44611 jul 23 16:07 lista-pacotes.txt
```
> Note que o tamanho do arquivo `lista2.txt` é maior porque contém a primeira com o texto `Listing... `, enquanto o arquivo `lista-pacotes.txt` não tem essa linha:
>```
>thiago@thiago-pc:~$ head -5 lista-pacotes.txt
>adduser/jammy,now 3.118ubuntu5 all [installed,automatic]
>amd64-microcode/jammy,now 3.20191218.1ubuntu2 amd64 [installed,automatic]
>apache2-bin/jammy-updates,now 2.4.52-1ubuntu4.5 amd64 [installed,automatic]
>apache2-data/jammy-updates,now 2.4.52-1ubuntu4.5 all [installed,automatic]
>apache2-utils/jammy-updates,now 2.4.52-1ubuntu4.5 amd64 [installed,automatic]
>
>thiago@thiago-pc:~$ head -5 lista2.txt
>Listing...
>adduser/jammy,now 3.118ubuntu5 all [installed,automatic]
>amd64-microcode/jammy,now 3.20191218.1ubuntu2 amd64 [installed,automatic]
>apache2-bin/jammy-updates,now 2.4.52-1ubuntu4.5 amd64 [installed,automatic]
>apache2-data/jammy-updates,now 2.4.52-1ubuntu4.5 all [installed,automatic]
>thiago@thiago-pc:~$
>```

# Instalando e particionando um novo disco
1. Crie o disco fisicamente;
2. Crie as partições no dispositivo;

Desligando a máquina com o comando `poweroff`:
```
thiago@thiago-pc:~$ sudo poweroff
[sudo] password for thiago:
Connection to 192.168.18.254 closed by remote host.
Connection to 192.168.18.254 closed.
```
> Como estamos usando o VirtualBox, criamos um disco rígido a mais antes de continuarmos a configuração dele no Ubuntu.

Usamos o comando `lshw` para listar os hardwares disponíveis (o parâmetro `-c disk` lista apenas os discos):

```
thiago@thiago-pc:~$ sudo lshw -c disk
  *-cdrom
       description: DVD reader
       product: CD-ROM
       vendor: VBOX
       physical id: 0.0.0
       bus info: scsi@1:0.0.0
       logical name: /dev/cdrom
       logical name: /dev/sr0
       version: 1.0
       capabilities: removable audio dvd
       configuration: ansiversion=5 status=nodisc
  *-disk:0
       description: ATA Disk
       product: VBOX HARDDISK
       vendor: VirtualBox
       physical id: 0
       bus info: scsi@2:0.0.0
       logical name: /dev/sda
       version: 1.0
       serial: VB93f8f2d5-cbb758be
       size: 25GiB (26GB)
       capabilities: gpt-1.00 partitioned partitioned:gpt
       configuration: ansiversion=5 guid=08a05fef-b21c-4d24-bdc4-b5a8336415f5 logicalsectorsize=512 sectorsize=512
  *-disk:1
       description: ATA Disk
       product: VBOX HARDDISK
       vendor: VirtualBox
       physical id: 1
       bus info: scsi@3:0.0.0
       logical name: /dev/sdb
       version: 1.0
       serial: VBc5bfdfe3-66bf946b
       size: 5GiB (5368MB)
       configuration: ansiversion=5 logicalsectorsize=512 sectorsize=512
thiago@thiago-pc:~$
```
> Há dois discos no computador: o `sda` e o `sdb`. 

Se executarmos o `fdisk -l`, veremos que o disco `sda` tem 3 partições (`sda1`, `sda2` e `sda3`), enquanto o disco `sdb` não é particionado ainda:

```
thiago@thiago-pc:~$ sudo fdisk -l
Disk /dev/loop0: 63,45 MiB, 66531328 bytes, 129944 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 63,34 MiB, 66412544 bytes, 129712 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 111,95 MiB, 117387264 bytes, 229272 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop3: 53,26 MiB, 55844864 bytes, 109072 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop4: 53,24 MiB, 55824384 bytes, 109032 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 08A05FEF-B21C-4D24-BDC4-B5A8336415F5

Device       Start      End  Sectors Size Type
/dev/sda1     2048     4095     2048   1M BIOS boot
/dev/sda2     4096  4198399  4194304   2G Linux filesystem
/dev/sda3  4198400 52426751 48228352  23G Linux filesystem


Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 11,5 GiB, 12343836672 bytes, 24109056 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
thiago@thiago-pc:~$
```

Vamos criar a partição **no dispositivo** com `fdisk`:
```
thiago@thiago-pc:~$ sudo fdisk /dev/sdb

Welcome to fdisk (util-linux 2.37.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0x537e8275.

Command (m for help):
```
> O único comando que você precisa decorar no `fdisk` é o `p`, que mostra a tabela de partição no dispositivo.

Criando a nova partição com o assistente (use o comando `n`):
```
Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1):
First sector (2048-10485759, default 2048):
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-10485759, default 10485759):

Created a new partition 1 of type 'Linux' and of size 5 GiB.

Command (m for help):
```
Sequência de criação da nova partição:
1. Definimos o tipo de partição a ser criada (primária ou extendida);
2. Numeramos ela com 1 (a primeira partição);
3. Definimos o início da partição (logo depois do último setor com meta informações sobre a partição);
4. Definimos o fim da partição.

Visualizando a partição:
```
Command (m for help): p
Disk /dev/sdb: 5 GiB, 5368709120 bytes, 10485760 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x537e8275

Device     Boot Start      End  Sectors Size Id Type
/dev/sdb1        2048 10485759 10483712   5G 83 Linux

Command (m for help):
```
Salvando as modificações da tabela de partição no disco:
```
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

thiago@thiago-pc:~$
```
