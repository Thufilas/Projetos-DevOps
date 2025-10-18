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





Primeiros passos para solucionar o problema: 

`docker pull marialazaradev/linux-essentials:latest`

`docker pull`é um comando do Docker que baixa uma imagem de container de um registro (registry) remoto, como o Docker Hub, para o sistema local⁠.

`marialazaradev/linux-essentials` é o nome de uma imagem Docker hospedada no registro Docker Hub⁠

`:latest`é uma tag (etiqueta) do Docker que identifica a versão mais recente de uma imagem de container⁠⁠. Quando você especifica :latest em um comando como docker pull, o Docker baixa automaticamente a versão mais atual disponível daquela imagem no registro, em vez de uma versão específica numerada.

Ao executar este comando no terminal voce irá baixar a imagem de container para seu sistema local, onde voce poderar acessar ele e realizar as devidas verificações.

`docker run -it --name devops-investigation marialazaradev/linux-essentials:latest`

`docker run -it` é um comando Docker que cria e inicia um novo container⁠⁠. A flag -it combina duas opções: -i (interactive) mantém a entrada padrão aberta para permitir interação, e -t (tty) aloca um terminal pseudo-TTY, permitindo que você execute comandos dentro do container.

`--name devops-investigation` é uma flag (opção) do comando Docker que atribui um nome personalizado ao container que está sendo criado⁠⁠. Isso permite identificar e referenciar o container de forma mais fácil usando o nome "devops-investigation" em vez de usar o ID gerado automaticamente pelo Docker.

Ao executar este comando no terminal voce irá realizar a conexão dentro da imagem do container. Permitindo que voce acesso o servidor diretamente na sua maquina. Neste exemplo após a execução deste comando voce irá conectar no servidor e aparecera a seguinte mensagem:

> C:\Users\João Victor>docker run -it --name devops-investigation marialazaradev/linux-essentials:latest
Iniciando billing-api em background...
billing-api iniciado com PID 8
devops@prod-web-01:/srv/app$
> 

Comandos para se utilizar no ambiente para entender aonde você está e como o ambiente está para que voce possa começar a fazer a correção dos erros.

 

`clear`  ou Ctrl+L para limpar o terminal.

**`pwd`**: Irá mostrar o path(caminho) onde você está dentro das pastas do servidor.

> /srv/app
> 

`whoami` irá mostrar qual o seu usuário dentro do servidor.

> devops
> 

`uname` irá mostrar qual o sistema que é este servidor.

> Linux
> 

`uname -a` irá mostrar detalhado qual o sistema que é este servidor.

> Linux 3e902dd062ad 6.6.87.2-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Thu Jun  5 18:30:46 UTC 2025 x86_64 GNU/Linux
> 

`df -h` mostrará o uso do disco deste servidor. Se utilizar junto ao comando -h ele mostrara de uma forma mais legível.

Sem o `-h`

> Filesystem      1K-blocks    Used  Available Use% Mounted on
overlay        1055762868 1548576 1000510820   1% /
tmpfs               65536       0      65536   0% /dev
shm                 65536       0      65536   0% /dev/shm
/dev/sde       1055762868 1548576 1000510820   1% /etc/hosts
tmpfs             4046944       0    4046944   0% /proc/acpi
tmpfs             4046944       0    4046944   0% /proc/scsi
tmpfs             4046944       0    4046944   0% /sys/firmware
> 

com o `-h`

> Filesystem      Size  Used Avail Use% Mounted on
overlay        1007G  1.5G  955G   1% /
tmpfs            64M     0   64M   0% /dev
shm              64M     0   64M   0% /dev/shm
/dev/sde       1007G  1.5G  955G   1% /etc/hosts
tmpfs           3.9G     0  3.9G   0% /proc/acpi
tmpfs           3.9G     0  3.9G   0% /proc/scsi
tmpfs           3.9G     0  3.9G   0% /sys/firmware
> 

`free -h` mostrará o uso da memoria ram deste servidor.

> total          used        free      shared  buff/cache   available
Mem:           7.7Gi       734Mi       5.1Gi       3.8Mi       2.1Gi       7.0Gi
Swap:           2.0Gi             0B       2.0Gi
> 

`ps aux` usado para verificar todos os processos ativos no servidor. ( como em um servidor real terão muitos processos ativos e pode conter vários processos de uma mesma aplicação o ideal é utilizar um filtro para filtrar esses processos.)

> USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
devops       1  0.0  0.0   4196  3200 pts/0    Ss   21:59   0:00 /bin/bash
devops       8  0.0  0.1  14100  9088 pts/0    S    21:59   0:00 python3 /usr/local/bin/billing-api-simulator.py
devops      17  7.6  0.0   8096  4096 pts/0    R+   22:53   0:00 ps aux
> 

`ps aux | grep billing`  com a utilização do `|` (pipe) ele irá pegar o resultado da busca realizada pelo ps aux e incrementar o comando `grep` que está adicionando um filtro (billing), mostrando apenas os processos que tenha “billing” no nome deles.

> devops       8  0.0  0.1  14100  9088 pts/0    S    21:59   0:00 python3 /usr/local/bin/billing-api-simulator.py
devops      21  0.0  0.0   3332  1280 pts/0    D+   23:06   0:00 grep billing
> 

`find / -name "billing"`  Comando `find` serve para realizar a busca de arquivos/diretórios no filesysteam. 

Argumento `/` irá começar a busca desde a pasta raiz do filesystem( como o exemplo a baixo ele ira começar a busca desde lá de cima e ir fazendo a varredura em todas a pastas). 

<img width="840" height="348" alt="image" src="https://github.com/user-attachments/assets/4458cb55-6d3c-4987-9011-573ae6074ddc" />


`-name "billing"`  Flag `-name` busca pelo nome exato. ( como eu coloquei “billing” ele me retornar todos os diretórios com este nome).

> find: '/etc/ssl/private': Permission denied
find: '/proc/tty/driver': Permission denied
find: '/root': Permission denied
/var/log/marialazaracloud/billing
find: '/var/cache/apt/archives/partial': Permission denied
find: '/var/cache/ldconfig': Permission denied
/data/billing
> 

Algumas pastas deram acesso negado pois o meu usuário é um usuário comum, para eu verificar qual tipo é o meu usuário basta verificar o caractere que esta no final do meu usuário por exemplo, o meu usuário neste servidor é o devops@prod-web-01:/srv/app$. Note-se que o final é o `$`  isso significa que este usuário é um usuário comum. Se ele fosse um usuário com elevação root(administrador máster) ele teria `#` no lugar.

`2>/dev/null`  filtra para não mostrar os diretórios que não tenho acesso ou arquivos com erros jogando para a lixeira.

> /var/log/marialazaracloud/billing
/data/billing
> 

`cd` comando para acessar alguma pasta.

`ls`  mostra o que tem dentro das pasta.

> app.log
> 

`ls -la` mostra todos itens dentro da pasta. A primeira letra se for `d` está informando que é um diretorio e se for `-`  informa que é um arquivo

> total 572
drwxr-xr-x 1 devops devops   4096 Sep 28 19:55 .
drwxr-xr-x 1 devops devops   4096 Sep 28 19:55 ..
-rw-r--r-- 1 devops devops 562991 Oct 13 01:03 app.log
> 

`head app.log` irá abrir o arquivo mostrando apenas os 10 primeiras linhas. ( não é viável abrir um arquivo de log inteiro, pois ele pode acabar travando o servidor por ter muitas linhas)

> 2024-09-27 14:20:15 INFO Starting billing-api service
2024-09-27 14:20:16 INFO Initializing database connections
2024-09-27 14:20:17 INFO Using data directory: /data/billing
2024-09-27 14:20:18 ERROR Data corruption detected in /data/billing/transactions
2024-09-27 14:20:18 ERROR Cannot process transactions - data validation failed
2024-09-27 14:20:23 INFO Using data directory: /data/billing
2024-09-27 14:20:24 ERROR Data corruption detected in /data/billing/transactions
2024-09-27 14:20:24 ERROR Cannot process transactions - data validation failed
2024-09-27 14:20:29 INFO Using data directory: /data/billing
2024-09-27 14:20:30 ERROR Data corruption detected in /data/billing/transactions
> 

`tail app.log` irá abrir o arquivo e mostrar apenas as 10 ultimas linhas.

> 2025-10-13 01:15:04 ERROR Cannot process transactions - data validation failed
2025-10-13 01:15:09 INFO Using data directory: /data/billing
2025-10-13 01:15:09 ERROR Failed to access data directory: Data corruption detected in /data/billing/transactions
2025-10-13 01:15:09 ERROR Cannot process transactions - data validation failed
2025-10-13 01:15:14 INFO Using data directory: /data/billing
2025-10-13 01:15:14 ERROR Failed to access data directory: Data corruption detected in /data/billing/transactions
2025-10-13 01:15:14 ERROR Cannot process transactions - data validation failed
2025-10-13 01:15:19 INFO Using data directory: /data/billing
2025-10-13 01:15:19 ERROR Failed to access data directory: Data corruption detected in /data/billing/transactions
2025-10-13 01:15:19 ERROR Cannot process transactions - data validation failed
> 

`tail -f app.log` adicionando o `-f` (significa follow) voce irá acompanhar o processo em tempo real com atualizações a cada 5 seg.

Para parar o processo em tempo real, basta apertar o `ctrl+c` 

`cat`  faz a leitura de todo o conteúdo dento do arquivo.

`nano` abre um editor de texto para que possamos fazer a edição.

`tail app.log > erro.txt` quando se adiciona a tag `>` ela irá pegar o resultado da pesquisa anterior e adicionar no arquivo após a tag. 

No exemplo acima ele pegou a lista que o tail mostrou dos arquivos do app.log e copiou tudo para o arquivo erro.txt

OBS: você pode colocar o nome de um arquivo na frente da tag `>`  que ele irá criar um arquivo com o mesmo nome e copiar os arquivos da pesquisa anterior. Se usar o comando em cima de uma pasta ja existente ele substituirá todos os arquivos pelos novos que você selecionou. 

`cp erro.txt /srv/app`  comando `cp` serve para copiar os arquivos de uma pasta para outra, informe o comando `cp`  em seguida o nome do arquivo que você deseja copiar e depois o caminho que irá colar este arquivo.

`mv` serve para mover o arquivo entre pastas e serve também para renomear a os arquivos, basta adicionar `mv erro.txt logs-erro.txt`  assim o arquivo que tinha o nome de erro.txt passara a ter o nome de logs-erro.txt

`find / -name "*conf*" 2>/dev/null | grep billing`  usando o comando para pesquisar eu posso utilizar o * antes e depois de alguma palavra para que ele considera tudo que tem antes do conf e tudo que tem depois do conf e adicionando o `| grep` eu posso filtrar deixando apenas o resultado do conf que estiver algum billing no nome. 

> /etc/marialazaracloud/billing-config.yml
/etc/marialazaracloud/billing-config.yml.backup
> 

`diff` usado para comparar dois arquivos e mostrar a diferença deles. 

exemplo, se eu utilizar `diff billing-config.yml billing-config.yml.backup`  ele irá retornar a seguinte resposta.

> 5c5
> 

> < data_directory: "/data/billing”
> 

> ---
> 

> >data_directory: "/opt/seeds”
> 

`chmod` comando para mudar as permissões de um arquivo.

<img width="630" height="401" alt="2" src="https://github.com/user-attachments/assets/2cfd8de7-415a-4067-9873-ffae5789df4b" />

<img width="891" height="682" alt="3" src="https://github.com/user-attachments/assets/e2f89c84-5ae3-40b0-aa67-657eaa7ea711" />



Exemplo: 

`chmod 644 arquivo.txt` o dono do arquivo.txt terá acesso a leitura e escrita, o grupo terá apenas leitura e outros terá apenas leitura. 

`kill` server para parar o processo ativo, utiliza o kill e o numero do processo(PID).

<img width="912" height="108" alt="4" src="https://github.com/user-attachments/assets/0686b491-ed78-423a-9220-407522ea592f" />


`/usr/local/bin/start-billing-api.sh` executar a api para voltar a funcionar.

Após a api voltar a ficar online verificar as logs com o `tail -f app.log` 

<img width="562" height="507" alt="5" src="https://github.com/user-attachments/assets/e1151142-aa5f-410e-ba53-5b876376e00d" />

Após a validação voltar o arquivo para somente leitura do jeito que estava anteriormente.

<img width="650" height="168" alt="6" src="https://github.com/user-attachments/assets/2f97cc88-9ea7-4c59-be29-6af1be1ce426" />
