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
