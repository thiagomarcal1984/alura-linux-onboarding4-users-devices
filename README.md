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
