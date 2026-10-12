# Lab01 — Provisionamento e configuração inicial de servidor Linux na Azure

**Categoria:** Cloud Computing | Infraestrutura | Linux  
**Plataforma:** Microsoft Azure  
**Sistema operacional:** Ubuntu Server 24.04 LTS  
**Status:** Concluído — configuração inicial e validações  
**Data:** Outubro de 2026

## 1. Objetivo do projeto

Provisionar uma máquina virtual Linux na Microsoft Azure e realizar as configurações iniciais necessárias para acesso remoto, atualização do sistema operacional e aplicação de medidas básicas de segurança.

O laboratório foi desenvolvido com foco no aprendizado prático de infraestrutura em nuvem, administração Linux, autenticação por chave SSH, gerenciamento de serviços e configuração de firewalls.

## 2. Arquitetura e ambiente

- **Provedor de nuvem:** Microsoft Azure
- **Grupo de recursos:** `Azure-Lab-Personal`
- **Máquina virtual:** `Azure-Lab-One-Ubuntu-Server`
- **Região:** Spain Central
- **Sistema operacional:** Ubuntu Server 24.04 LTS x64
- **Tamanho da VM:** Standard_B2ls_v2
- **Recursos computacionais:** 2 vCPUs e 4 GiB de RAM
- **Armazenamento:** disco do sistema operacional com aproximadamente 30 GB
- **Rede:** interface de rede virtual com endereço IP privado
- **Acesso administrativo:** SSH com autenticação por chave privada
- **Controle de tráfego:** Network Security Group (NSG) da Azure e UFW do Ubuntu

> Endereços IP, chaves privadas e outras informações de acesso foram omitidos desta documentação por segurança.

## 3. Atividades realizadas

### 3.1. Provisionamento da máquina virtual

Criação de uma máquina virtual Linux na Azure, utilizando uma região permitida pela assinatura Azure for Students e uma configuração adequada para um laboratório de aprendizado.

A VM foi criada com Ubuntu Server 24.04 LTS, permitindo praticar administração Linux em um ambiente real de nuvem.

### 3.2. Configuração do acesso SSH

Configuração do acesso remoto utilizando o OpenSSH e uma chave privada armazenada localmente no Windows.

Durante a primeira tentativa de conexão, foi identificado um problema relacionado às permissões da chave privada. O problema foi corrigido por meio do comando `icacls`, ajustando as permissões de acesso ao arquivo.

Após a correção, a conexão SSH foi estabelecida com sucesso.

Comando utilizado no PowerShell:

```powershell
$key = "CAMINHO_DA_CHAVE_PRIVADA.pem"
ssh -i "$key" usuario@IP_PUBLICO
```

Os valores foram generalizados para evitar a exposição de informações de acesso.

### 3.3. Atualização do sistema operacional

O sistema foi atualizado utilizando o gerenciador de pacotes APT.

Primeiro, foi executado:

```bash
sudo apt update
```

Esse comando atualiza os índices dos pacotes disponíveis nos repositórios configurados.

Em seguida:

```bash
sudo apt upgrade
```

Esse comando instala as atualizações disponíveis para os pacotes já instalados, conforme as dependências e confirmações necessárias.

Após a atualização, foi necessária a reinicialização da máquina virtual para utilizar o novo kernel instalado.

```bash
sudo reboot
```

Depois da reinicialização, a conexão SSH foi restabelecida e o kernel ativo foi validado:

```bash
uname -r
```

**Resultado validado:** `7.0.0-1017-azure`.

Também foi verificado que não havia atualizações de pacotes pendentes no momento da validação.

### 3.4. Verificação do serviço SSH

O estado do serviço SSH foi verificado com:

```bash
sudo systemctl status ssh --no-pager
```

O resultado confirmou que o serviço estava ativo e em execução.

Também foi verificado o estado do socket SSH:

```bash
sudo systemctl is-enabled ssh.socket
```

**Resultado:** `enabled`.

Isso confirmou que o socket SSH estava habilitado para inicialização automática. O serviço SSH estava ativo durante os testes.

### 3.5. Configuração do firewall UFW

Inicialmente, o firewall UFW estava inativo.

Antes de ativá-lo, foi adicionada uma regra para permitir conexões SSH:

```bash
sudo ufw allow OpenSSH
```

Em seguida, o firewall foi ativado:

```bash
sudo ufw enable
```

O estado e as regras foram verificados com:

```bash
sudo ufw status verbose
```

E, posteriormente, com:

```bash
sudo ufw status numbered
```

**Configuração validada:**

- Firewall ativo.
- Tráfego de entrada bloqueado por padrão.
- Tráfego de saída permitido por padrão.
- SSH permitido para IPv4 e IPv6.

As regras locais do UFW permitem SSH a partir de qualquer origem. A restrição de origem é aplicada adicionalmente no NSG da Azure, cuja regra de entrada foi configurada para permitir SSH a partir do endereço IP público do cliente.

### 3.6. Inspeção de portas e sockets

Foi executado:

```bash
sudo ss -tulpn
```

O comando permitiu identificar sockets e portas em escuta ou aguardando conexões.

Entre os resultados observados:

- **TCP/22:** serviço SSH.
- **TCP/UDP 53:** resolvedor DNS local (`systemd-resolved`).
- **UDP/68:** serviço de rede associado ao DHCP.
- **UDP/323:** serviço de sincronização de horário (`chronyd`).

Os serviços DNS e de sincronização de horário observados estavam associados a endereços locais. O SSH estava escutando nas interfaces IPv4 e IPv6.

### 3.7. Validação da configuração de rede da Azure

A regra de entrada SSH do NSG foi revisada no portal Azure.

Configuração observada:

- Origem: endereço IP público específico.
- Porta de destino: TCP/22.
- Serviço: SSH.
- Ação: permitir.
- Prioridade: 1000.

A regra foi conferida visualmente no portal, mantendo a restrição de origem configurada para o cliente.

### 3.8. Teste final de conectividade

Após ativar o UFW, foi aberta uma nova sessão SSH a partir do Windows.

A conexão foi estabelecida com sucesso, confirmando que o acesso remoto continuava funcionando após a alteração do firewall local.

## 4. Comandos praticados

| Comando | Finalidade |
|---|---|
| `ssh -i` | Conectar remotamente utilizando uma chave privada. |
| `icacls` | Consultar e modificar permissões de arquivos no Windows. |
| `sudo apt update` | Atualizar os índices dos repositórios APT. |
| `sudo apt upgrade` | Atualizar os pacotes instalados. |
| `sudo reboot` | Reiniciar o sistema operacional. |
| `uname -r` | Consultar a versão do kernel ativo. |
| `systemctl status` | Verificar o estado de um serviço. |
| `systemctl is-enabled` | Consultar se uma unidade está habilitada para inicialização. |
| `ufw allow OpenSSH` | Criar uma regra para permitir SSH no UFW. |
| `ufw status verbose` | Consultar o estado e as configurações do firewall. |
| `ufw status numbered` | Exibir as regras do UFW com numeração. |
| `ss -tulpn` | Inspecionar sockets, portas e processos associados. |

## 5. Resultados alcançados

Ao final desta etapa, foi possível:

- Provisionar e acessar uma máquina virtual Linux na Azure.
- Corrigir um problema de permissões de chave privada no Windows.
- Estabelecer acesso administrativo remoto via SSH.
- Atualizar o sistema operacional e validar o kernel após reinicialização.
- Inspecionar o estado do serviço SSH e do socket correspondente.
- Configurar e verificar o firewall UFW.
- Identificar portas e serviços em escuta no Linux.
- Revisar a regra de entrada SSH do NSG da Azure.
- Testar novamente a conectividade após a ativação do firewall.

## 6. Conhecimentos praticados

**Cloud Computing:** máquina virtual, região, grupo de recursos, rede virtual e regras de segurança na Azure.

**Linux:** administração inicial do Ubuntu, gerenciamento de pacotes, reinicialização, kernel, serviços e sockets de rede.

**Redes e segurança:** SSH, autenticação por chave pública, portas TCP/UDP, endereços privados, firewall local e Network Security Group.

**Windows:** gerenciamento de permissões de arquivos por meio do `icacls` e conexão SSH pelo PowerShell.

## 7. Próximas etapas

As próximas atividades poderão incluir:

1. Documentar a topologia de rede e os recursos da Azure.
2. Investigar o estado do Azure Linux Agent no portal.
3. Praticar a criação e administração de usuários e grupos Linux.
4. Instalar e configurar um serviço de aplicação web.
5. Testar a publicação de um serviço utilizando regras de rede específicas.
6. Avaliar custos e aplicar práticas de desligamento e desprovisionamento da VM quando ela não estiver em uso.
7. Registrar os comandos, resultados e dificuldades de cada novo exercício.

## 8. Considerações finais

Este laboratório representa uma primeira etapa prática de administração de infraestrutura Linux em nuvem. O foco foi criar um ambiente funcional, configurar acesso remoto, atualizar o sistema e verificar as camadas iniciais de segurança.

A documentação registra as atividades efetivamente realizadas e os resultados observados, servindo como base para novos laboratórios e para a evolução dos estudos em Cloud Computing e infraestrutura.

**Projeto:** Lab01 — Azure Ubuntu Server  
**Responsável:** Autor do laboratório
