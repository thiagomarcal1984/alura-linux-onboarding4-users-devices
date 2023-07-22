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
Repare que o path para os bashes dos usuários `thiago2` e `thiago3` são diferentes: `thiago2` usa o terminal padrão `/bin/sh`, enquanto `thiago3` usa o terminal bash `/bin/bash`.

O arquivo `/etc/group` lista os grupos existentes, seguidos dos números dos IDs dos usuários que pertencem a esse grupo. Note que os usuários `thiago2` e `thiago3` não estão no grupo `sudo`.

> Na maioria das distribuições Linux, os usuários de ID maiores ou iguais a 1000 são usuários comuns. Abaixo desse número estão os IDs dos usuários de sistema, o que inclui o root, de ID zero.

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
