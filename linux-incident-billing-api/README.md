# 💻 Exercício Iniciante Linux/DevOps: Investigação de Incidente na billing-api

**📝 Contexto Empresarial**

A **MariaLazaraCloud** é uma empresa de tecnologia que fornece soluções SaaS B2B para automação de cobrança e faturamento. Sua arquitetura é baseada em microsserviços empacotados em containers Docker, com um foco crucial em alta disponibilidade.

---


**O Incidente**

| **Detalhe** | **Valor** |
| --- | --- |
| **Data/Hora do Evento** | 27 de setembro de 2024, 14:25 UTC |
| **Serviço Afetado** | `billing-api` (API de pagamentos) |
| **Sintomas Reportados** | Aplicação não processa transações, erros de corrupção de dados nos logs, falha na validação de integridade. O dashboard indica status "degraded" (degradado). |

**Seu Papel**

Você é o DevOps Engineer on-call. Como o sistema é legado e tem pouca documentação, você não conhece bem os caminhos dos arquivos. Vamos explorar o sistema calmamente, usando comandos básicos como `find` apenas quando necessário para localizar arquivos e diretórios. Cada passo inclui explicações detalhadas dos comandos (o que cada parte faz), e reflexões sobre por quê usá-los, quando aplicá-los, o motivo e o objetivo. Assumimos que você é iniciante no Linux, então vamos devagar, com narrativas explicando o que estamos fazendo antes de prosseguir para o próximo passo.


## 🛠️ Simulação do Ambiente

Para replicar o cenário de incidente, utilize a imagem Docker fornecida:

### 1. Obter a Imagem do Container

Baixa a imagem do Docker Hub para a máquina local.

Bash

`docker pull marialazaradev/linux-essentials:latest`

> Explicação:
> 
> - `docker pull`: Baixa uma imagem de container de um registro remoto (Docker Hub).
> - `:latest`: Tag que identifica a versão mais recente da imagem.

### 2. Iniciar e Conectar ao Container

Cria e inicia o container, conectando-se interativamente ao terminal.

Bash

`docker run -it --name devops-investigation marialazaradev/linux-essentials:latest`

> Explicação:
> 
> - `docker run -it`: Cria e inicia o container com um terminal interativo (`i` e `t`).
> - `-name devops-investigation`: Atribui um nome amigável para o container.

**Saída Esperada ao Conectar:**

`Iniciando billing-api em background...
billing-api iniciado com PID 8
devops@prod-web-01:/srv/app$`

---

## 🔍 Investigação e Remediação (Passo a Passo)

### Fase 1: Entendimento Inicial do Sistema

| **Comando** | **Objetivo** | **Saída (Exemplo)** | **Reflexão** |
| --- | --- | --- | --- |
| `pwd` | Onde estamos no filesystem. | `/srv/app` | Localização inicial. |
| `whoami` | Identificação do usuário. | `devops` | Confirma o usuário. O `$` no prompt indica que não somos `root` (`#`). |
| `ps aux | grep billing` | Verificar se a aplicação está ativa. | `... python3 /usr/local/bin/billing-api-simulator.py` | A API está rodando, mas com falha. |
| `df -h` | Verificar uso de disco. | `1% /` | Sem problemas de espaço em disco. |
| `free -h` | Verificar uso de memória. | `7.7Gi` | Sem problemas de memória. |


### Fase 2: Análise dos Logs

O erro inicial aponta para **corrupção de dados**. Precisamos encontrar os logs.

1. **Localizar Diretórios Relacionados à `billing`:**Bash
    
    `find / -name "billing" 2>/dev/null`
    
    > Explicação: O 2>/dev/null descarta mensagens de "Permissão negada" que ocorrerão em pastas restritas (e.g., /root, /etc/ssl/private).
    > 
    
    > Saída:
    > 
    
    > /var/log/marialazaracloud/billing
    /data/billing
    > 
    
2. **Acessar o Diretório de Logs e Listar Conteúdo:**Bash
    
    `cd /var/log/marialazaracloud/billing
    ls -la`
    
    > Saída: O arquivo app.log é encontrado.
    > 
    
3. **Inspecionar o Final do Log:**Bash
    
    `tail app.log`
    
    > Explicação: tail mostra as últimas 10 linhas, onde os erros recentes provavelmente estão. O tail -f app.log é útil para acompanhar em tempo real.
    > 
    
    > Evidência no Log:
    > 
    
    > 2025-10-13 01:15:19 ERROR Failed to access data directory: Data corruption detected in /data/billing/transactions
    2025-10-13 01:15:19 ERROR Cannot process transactions - data validation failed
    > 
    
    > Conclusão da Fase: O erro é recorrente e está relacionado à falha de acesso ou validação do diretório /data/billing/transactions.
    

### Fase 3: Detecção da Causa Raiz

O erro de dados frequentemente aponta para uma configuração errada.

1. **Localizar Arquivos de Configuração (`conf*`):**Bash
    
    `cd /etc/marialazaracloud # Acessando a pasta onde o log indica que as config estão
    find / -name "*conf*" 2>/dev/null | grep billing`
    
    > Explicação: Filtra arquivos de configuração (*conf*) relacionados à billing.
    > 
    
    > Saída:
    > 
    
    > /etc/marialazaracloud/billing-config.yml
    /etc/marialazaracloud/billing-config.yml.backup
    > 
    
2. **Comparar Configuração Ativa com Backup:**Bash
    
    `diff billing-config.yml billing-config.yml.backup`
    
    > Explicação: diff é usado para mostrar as diferenças linha a linha entre os dois arquivos.
    > 
    
    > Saída:
    > 
    
    > 5c5
    < data_directory: "/data/billing”
    ---
    >data_directory: "/opt/seeds”
    > 
    
    > Causa Raiz Confirmada: O arquivo ativo (billing-config.yml) foi alterado para o caminho incorreto (/data/billing) em vez do caminho correto de dados (/opt/seeds).
    > 
    

### Fase 4: Remediação e Validação

1. Corrigir o Arquivo de Configuração:Bash
    
    A maneira mais rápida de reverter é copiar o backup correto sobre o arquivo corrompido.
    
    `cp billing-config.yml.backup billing-config.yml`
    
2. **Parar o Processo Ativo:**Bash
    
    `# (Re)encontrar o PID, que pode ter mudado
    # ps aux | grep billing-api-simulator.py | grep -v grep  -> PID 8
    kill 8`
    
    > Explicação: Encerra o processo da API, permitindo que ele seja reiniciado com a nova configuração.
    > 
    
3. Reiniciar a API:Bash
    
    O sistema possui um script de inicialização.
    
    `/usr/local/bin/start-billing-api.sh`
    
4. **Validar o Serviço (Checar o Log Novamente):**Bash
    
    `tail -f /var/log/marialazaracloud/billing/app.log`
    
    > Saída de Sucesso:
    > 
    
    > 2024-09-27 15:00:02 INFO Using data directory: /opt/seeds  <- CORRETO!
    2024-09-27 15:00:03 INFO Service is fully operational, processing transactions.
    > 

### Fase 5: Pós-Incidente (Hardening)

Para evitar que este erro ocorra novamente, removemos a permissão de escrita do arquivo de configuração, tornando-o "somente leitura".

1. **Restaurar Permissões (Apenas Leitura - 444):**Bash
    
    `chmod 444 billing-config.yml`
    
    > Explicação: chmod altera permissões. 444 significa read (4) para o Dono, Grupo e Outros, prevenindo alterações acidentais.
    > 
    

---

## 📖 Glossário de Comandos Linux

| **Comando** | **Função Principal** | **Exemplo de Uso** |
| --- | --- | --- |
| `find` | Localizar arquivos/diretórios. | `find / -name "arquivo.txt"` |
| `grep` | Filtrar texto dentro de arquivos ou pipelines. | `cat log.txt | grep ERROR` |
| ` | ` (pipe) | Conectar a saída de um comando à entrada de outro. |
| `>` / `2>/dev/null` | Redirecionamento de saída (normal ou de erro). | `find / -name "*.conf" 2>/dev/null` |
| `head` / `tail` | Ver o início/fim de um arquivo. | `tail -f app.log` (acompanhar) |
| `df -h` | Mostrar uso do disco em formato human-readable. | `df -h` |
| `ps aux` | Listar processos. | `ps aux` |
| `kill` | Encerrar um processo pelo PID. | `kill 1234` |
| `cp` / `mv` | Copiar / Mover (ou renomear). | `mv old.txt new.txt` |
| `diff` | Comparar dois arquivos. | `diff file1 file2` |
| `chmod` | Mudar permissões de arquivo. | `chmod 644 script.sh` |

