# Laboratório: Análise de Portas e Ligações de Rede no Windows

## Objetivo

Identificar as ligações de rede e as portas abertas num computador pessoal com Windows, descobrir que programa está por trás de cada uma, confirmar se é legítimo e **reduzir a superfície de ataque**, fechando o que não é necessário.

## Ambiente

| Item | Detalhe |
| --- | --- |
| Equipamento | Microsoft Surface com Windows |
| Ferramentas | CMD como administrador, PowerShell, Definições do Windows, Segurança do Windows |
| Rede | Rede doméstica com router |

## Metodologia

O ciclo usado para cada ligação ou porta:

1. Listar as ligações e as portas
2. Filtrar o que interessa
3. Identificar o processo pelo PID
4. Verificar o caminho do executável
5. Verificar a assinatura digital
6. Decidir: manter, desligar ou bloquear
7. Confirmar o resultado

## Comandos usados

### Listar todas as ligações

```cmd
netstat -ano
```

| Opção | Significado |
| --- | --- |
| `-a` | Mostra todas as ligações e portas à escuta |
| `-n` | Mostra endereços e portas em números, sem converter para nomes |
| `-o` | Mostra o PID (número do processo) responsável |

Colunas: Protocolo (TCP/UDP), Endereço local, Endereço externo, Estado, PID.

### Filtrar ligações ativas

```cmd
netstat -ano | findstr ESTABLISHED
```

A barra vertical `|` envia o resultado para o `findstr`, que mostra só as linhas com a palavra indicada. Só aparece TCP, porque o UDP não estabelece ligações.

### Filtrar portas à escuta

```cmd
netstat -ano | findstr LISTENING
```

### Ver todas as ligações de um processo

```cmd
netstat -ano | findstr 15444
```

### Identificar o programa pelo PID

```cmd
tasklist /fi "PID eq 15444"
```

| Parte | Significado |
| --- | --- |
| `tasklist` | Lista os processos em execução |
| `/fi` | Aplica um filtro |
| `"PID eq 15444"` | O PID é igual (*equal*) a 15444 |

Para ver os serviços dentro de um `svchost.exe`:

```cmd
tasklist /svc /fi "PID eq 3652"
```

### Ver o caminho do executável

```cmd
powershell "Get-Process -Id 3652 | Select-Object Path"
```

### Verificar a assinatura digital

```cmd
powershell "Get-AuthenticodeSignature C:\Windows\System32\svchost.exe"
```

O estado **Valid** confirma que o ficheiro é do editor indicado e não foi alterado.

### Ver aparelhos da rede local

```cmd
arp -a
```

Mostra a tabela ARP: endereços IP e endereços físicos (MAC) dos aparelhos com quem o PC comunicou recentemente.

## Conceitos importantes

| Conceito | Explicação |
| --- | --- |
| PID e portas altas | São temporários e mudam sempre que o programa arranca. Investiga-se pelo programa, não pelo número |
| `127.0.0.1` (localhost) | A porta só aceita ligações do próprio computador |
| `192.168.x.x` | Endereço do PC na rede doméstica; só os aparelhos da rede o alcançam |
| `0.0.0.0` | A porta aceita ligações de qualquer interface de rede |
| ESTABLISHED / LISTENING | Ligação ativa / porta aberta à espera de ligações |
| Porta 443 | HTTPS, tráfego web encriptado |
| Three-way handshake (TCP) | SYN, SYN-ACK, ACK. Só depois a ligação fica ESTABLISHED |
| Unicast / Broadcast / Multicast | Um para um, um para todos (255.255.255.255), um para um grupo (224.0.0.x) |
| Superfície de ataque | Cada funcionalidade ou porta ligada é um possível ponto de entrada |

## Resultados

| Ligação / Porta | Processo | Conclusão | Ação |
| --- | --- | --- | --- |
| 127.0.0.1 (local) | AppleMobileDeviceLauncher.exe | App "Apple Devices" legítima mas não usada, cerca de 11 GB | ✅ Desinstalada |
| Externo :443 | chrome.exe | Tráfego HTTPS do navegador, normal | Manter |
| Processo 3652 | svchost.exe | Caminho em System32 e assinatura Valid: legítimo | Manter |
| 0.0.0.0:7680 | svchost.exe (Otimização de Entrega) | Partilha peer-to-peer de atualizações; desnecessária | ✅ Desligada e porta fechada |
| 0.0.0.0:5040 | svchost.exe (CDPSvc) | Plataforma de dispositivos ligados (Ligação ao Telemóvel, Partilha nas proximidades) | A avaliar / bloquear na firewall |
| 0.0.0.0:49664 | lsass.exe | Processo de autenticação do Windows; nunca terminar | ✅ Proteção LSA confirmada |
| 192.168.x.x:52686 | chrome.exe | Porta temporária do Chrome na rede local | Manter, monitorizar |

**Aparelhos na rede (`arp -a`):** o router e um outro aparelho. Os restantes endereços eram multicast (224.0.0.22, 224.0.0.251 mDNS, 224.0.0.252 LLMNR) e broadcast (255.255.255.255).

## Fechar a porta 7680 (Otimização de Entrega)

1. **Definições** > **Windows Update** > **Opções avançadas**
2. **Otimização de Entrega**
3. Desligar o interruptor principal (transferências de outros dispositivos)
4. Confirmar com `netstat -ano | findstr LISTENING`

## Bloquear uma porta na firewall

Duas formas de fechar uma porta:

1. **Desligar o serviço:** nenhum programa fica à escuta; a porta fecha de facto.
2. **Bloquear na firewall:** o serviço continua à escuta, mas o tráfego não passa.

Usar as duas é **defesa em profundidade**.

Criar a regra (CMD como administrador):

```cmd
netsh advfirewall firewall add rule name="Bloquear TCP 5040" dir=in action=block protocol=TCP localport=5040
```

Desativar temporariamente:

```cmd
netsh advfirewall firewall set rule name="Bloquear TCP 5040" new enable=no
```

Reativar:

```cmd
netsh advfirewall firewall set rule name="Bloquear TCP 5040" new enable=yes
```

Também é possível pela interface gráfica: **Firewall do Windows Defender com Segurança Avançada** > **Regras de Entrada** > **Nova Regra** > **Porta**.

## Riscos estudados

| Risco | Explicação |
| --- | --- |
| Extensões do navegador | Podem ler tudo o que se vê no navegador e enviar dados. Rever em `chrome://extensions` |
| LLMNR | Um atacante na mesma rede pode responder a pedidos de nomes e obter o hash da palavra-passe do Windows. Risco maior em Wi-Fi público |
| lsass.exe | Alvo de ferramentas de roubo de credenciais; manter a proteção LSA ligada |
| svchost.exe | Malware pode imitar o nome; verificar sempre caminho e assinatura |

## Conclusões

- A maioria das ligações eram legítimas (Windows, Chrome).
- Foi removida uma aplicação não usada e fechada uma porta desnecessária.
- A linha de comandos permite filtrar, automatizar e trabalhar remotamente; muita desta informação também existe na interface gráfica (por exemplo, `resmon`, separador Rede).
- Esta análise reduz o risco, mas não garante 100% de deteção: malware avançado pode esconder-se.

## Próximos passos

- [ ] Decidir se bloqueio a porta 5040 (CDPSvc) na firewall
- [ ] Ver todos os aparelhos ligados na página do router
- [ ] Repetir a análise periodicamente e comparar resultados
