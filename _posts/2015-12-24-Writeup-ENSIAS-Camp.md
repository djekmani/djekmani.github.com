---
layout: post
title: "[Writeups PART 1] MCSC2016: ENSIAS Qualification "
---
Bonjour tout le monde,

Je sais que je suis très inactif sur ce blog, ce n'est pas à cause de la paresse mais à cause du programme chargé des études, (les projets, les examens...), je ne trouve pas le temps de rédiger, même si j'ai beaucoup d'idées d'articles très instructifs et intéressants. Je pense qu'en 2016 je serai plus actif car j'aurai terminé mon dernier semestre.

Avant hier, le Mardi 22 décembre 2015 nous avons organisé une compétition locale (CTF) au bénéfice des étudiants de l'ENSIAS leur permettant de s'évaluer et d'explorer leurs compétences dans la sécurité informatique, ainsi que de choisir ceux qui vont représenter l'école (ENSIAS) à la compétition nationale MCSC2016.













Le scoreboard final:

![Alt text](/public/images/scoreboard.png "scoreboard")

Cet article dû à la demande de plusieurs étudiants voulant que je fassent des writeups des épreuves. En général, les épreuves on été très facile, ils ne demandaient que de la concentration et quelques compétences techniques.

Sans trop tarder là-dessus on commence la partie la plus chaude :smile:

Le répo suivant contient les binaires et les ressources pour les différentes épreuves.

[https://github.com/djekmani/mcsc-writeup/tree/master/local-ensias-camp](https://github.com/djekmani/mcsc-writeup/tree/master/local-ensias-camp)

####Catégorie Pwn (Exploitation)

###### Entry 80pts
<hr>

**Task:**


![Alt text](/public/images/entry.png "entry task")

Le premier réflexe est de vérifier le service:


![Alt text](/public/images/test-serv1.png "testing")

Deux fichiers joints avec le challenge, sont le binaire du service et son code source (J'étais très généreux sur ce challenge)

Code source:

{% gist djekmani/a965de56ee376fbd6df5 %}

En analysant ce code source on remarque deux choses:

1 - Une fonction intéressante `getFlag()`  lisant un fichier `flag.txt`.


2 - La ligne 19: `read(0 , buff, 80);` on lit 80 bytes depuis stdin et on stock le buffer dans la variable `buff` déclaré 
statiquement avec une taille de 64bytes.



Strategie d'exploitation:

La vulnérabilité stack overflow dans la ligne 19 va nous permettre de controller le pointeur d'instruction et appeler la fonction `getFlag()`.

Maintenant on attaquera le binaire pour mettre en place notre payload.

```
file level1
level1: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=2be013c8f0e9766018f8b13d3927f59e3143ea96, not stripped

```


{% gist djekmani/13e153c74d4bf49c30a0 %}

En peut construire notre exploit tranquillement aucune mitigation n'est active, l'offset est 76bytes, notre payload: "A"*76+addr(getFlag)



L'exploit finale ressemble à ça :blush:

{% gist djekmani/10ac151b8db0746bcec4 %}

L'execution en live mode :D

![Alt text](/public/images/exp1.png "pwn1")

vous voyez, c'est très facile :laughing:


###### barpwn 200pts
<hr>

**Task:**


![Alt text](/public/images/barpwn.png "barpwn task")

```

file level3 
level3: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=a8bc20609763809a660867576010a78e1c826ad7, not stripped
root@PenLab:~/mcsc/pwn/level3# gdb ./level3 -q
Reading symbols from ./level3...(no debugging symbols found)...done.
gdb-peda$ checksec
CANARY    : disabled
FORTIFY   : disabled
NX        : ENABLED
PIE       : disabled
RELRO     : disabled
gdb-peda$ 
```


On est face à un binaire 32bits compilé dynamiquement avec une [pile non executable](https://en.wikipedia.org/wiki/NX_bit) et sans [ASLR](https://fr.wikipedia.org/wiki/Address_space_layout_randomization). Ça semble facile à exploiter il nous reste que le contrôle de l'eip.

```
gdb-peda$ r 
Starting program: /root/mcsc/pwn/level3/level3 
your name bro: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
My little bad Boy!! Stop trying to hack system
exiting

[Inferior 1 (process 29731) exited with code 01]
Warning: not running or target is remote
gdb-peda$ disas welcome
Dump of assembler code for function welcome:
   0x080484db <+0>:	push   ebp
   0x080484dc <+1>:	mov    ebp,esp
   0x080484de <+3>:	sub    esp,0x58
   0x080484e1 <+6>:	mov    DWORD PTR [ebp-0xc],0xf1f2f3b4
   0x080484e8 <+13>:	sub    esp,0xc
   0x080484eb <+16>:	push   0x8048610
   0x080484f0 <+21>:	call   0x8048380 <printf@plt>
   0x080484f5 <+26>:	add    esp,0x10
   0x080484f8 <+29>:	mov    eax,ds:0x80498a0
   0x080484fd <+34>:	sub    esp,0xc
   0x08048500 <+37>:	push   eax
   0x08048501 <+38>:	call   0x8048390 <fflush@plt>
   0x08048506 <+43>:	add    esp,0x10
   0x08048509 <+46>:	sub    esp,0x4
   0x0804850c <+49>:	push   0x400
   0x08048511 <+54>:	lea    eax,[ebp-0x4c]
   0x08048514 <+57>:	push   eax
   0x08048515 <+58>:	push   0x0
   0x08048517 <+60>:	call   0x8048370 <read@plt>
   0x0804851c <+65>:	add    esp,0x10
   0x0804851f <+68>:	cmp    DWORD PTR [ebp-0xc],0xf1f2f3b4
   0x08048526 <+75>:	je     0x8048542 <welcome+103>
   0x08048528 <+77>:	sub    esp,0xc
   0x0804852b <+80>:	push   0x8048620
   0x08048530 <+85>:	call   0x80483a0 <puts@plt>
   0x08048535 <+90>:	add    esp,0x10
   0x08048538 <+93>:	sub    esp,0xc
   0x0804853b <+96>:	push   0x1
   0x0804853d <+98>:	call   0x80483c0 <exit@plt>
   0x08048542 <+103>:	sub    esp,0xc
   0x08048545 <+106>:	push   0x8048658
   0x0804854a <+111>:	call   0x80483a0 <puts@plt>
   0x0804854f <+116>:	add    esp,0x10
   0x08048552 <+119>:	leave  
   0x08048553 <+120>:	ret    
End of assembler dump.
gdb-peda$ 
```
On remarque qu'il y a une vérification avant l'épilogue de la fonction vulnérable lors du dépassement du buffer, il en résulte une exception qui affiche le message :


```
My little bad Boy!! Stop trying to hack system
exiting
```

Après l'analyse de la fonction `welcome()` on remarque qu'il y a une comparaison à l'instruction `welcome+68` de `0xf1f2f3b4` avec `(ebp - 0xc)`

#####TODO:

1 - Déterminer la distance entre le debut du buffer et (ebp-0xc)

2 - Réécrire  0xf1f2f3b4 dans (ebp - 0xc)

3 - Déterminer la distance entre (ebp - 0xc) et 'saved eip'

4 - ret2libc

pour ceux qui ne savent rien sur la méthode 'ret2libc' je recommande de lire cet [article](https://www.exploit-db.com/docs/17131.pdf) .

Exploit finale
{%gist djekmani/a3025232f855998f5229 %}

Execution de l'exploit:

![Alt text](/public/images/exp3.png "pwn3")

Voila! :smiley:

###### POPROP 400pts
<hr>

**Task:**


![Alt text](/public/images/poprop.png "poprop task")


Sur ce niveau [NX](https://en.wikipedia.org/wiki/NX_bit) et [ASLR](https://fr.wikipedia.org/wiki/Address_space_layout_randomization) sont activé.

On peut pas refaire ret2libc car ici on est face à l'ASLR randomisant 'libc' qui contient les addresses des fonctions (system , et la famille des fonctions exec). 

Donc on va s'appuyer sur [ROP](https://en.wikipedia.org/wiki/Return-oriented_programming) (Return Oriented Programming), encore une fois pour ceux qui ne savent rien sur le ROP ou comment l'utiliser je vous invite à lire cet article introduisant la technique [ROP](https://www.exploit-db.com/docs/28479.pdf) sur un système 32bits.



```
file level4
level4: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), statically linked, for GNU/Linux 2.6.32, BuildID[sha1]=d9cc289365b7230bd25daf4e49c1fdc9e63f1854, stripped
```


Heureusement notre binaire est compilé statiquement, cela implique que nous avons autant de gadgets qu'on le désire :wink:. Après avoir suivi la documentation sur le ROP votre exploit va ressembler à ceci:


**Exploit:**

{% gist djekmani/3f28bdf91dd43a0dc1e6 %}


L'execution en live :smiley:


![Alt text](/public/images/exp4.png "exp4")

Its easy n'est ce pas :grin:


Pour ne pas rendre l'article ennuyeux je termine ici, et je vous promet un deuxième qui termine celui-ci en traitant les challenges web et ceux du reverse enginnering.

Si vous n'avez pas compris un passage, je serais heureux de répondre à vos questions en commentaires.

Je vous souhaite une bonne journée, à la prochaine :wink:
