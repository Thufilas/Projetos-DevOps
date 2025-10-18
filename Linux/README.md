
# 📚 Guia Essencial de Comandos Linux para DevOps

Este guia prático contém os comandos fundamentais do Linux para navegação, gerenciamento de arquivos, monitoramento de sistema e diagnóstico de problemas, essenciais para qualquer Engenheiro DevOps ou SysAdmin.

## ⚠️ Acesso e Contexto (Dados de Teste)

| **Comando** | **Objetivo** |
| --- | --- |
| `clear` ou `Ctrl+L` | Limpar a tela do terminal. |
| `ip addr show` | Exibir informações de endereço IP do servidor. |
| `pwd` | Imprimir o diretório de trabalho atual (Path). |
| `whoami` | Mostrar o nome do usuário logado. |
| `id` | Mostrar o UID, GID e grupos do usuário atual. |

---

## 📁 Gerenciamento de Arquivos e Diretórios

| **Comando** | **Função** | **Flags Comuns** | **Exemplo de Uso** |
| --- | --- | --- | --- |
| **`touch`** | Cria um ou mais arquivos vazios. | N/A | `touch file1.txt file2.log` |
| **`mkdir`** | Cria um novo diretório (pasta). | `-p` (Cria diretórios pais, se necessário) | `mkdir -p /app/logs` |
| **`rm`** | Deleta arquivos e diretórios. | `-i` (Confirmação antes de excluir) | `rm -i arquivo.txt` |
|  |  | `-r` (Recursivo: exclui diretórios e seu conteúdo) | `rm -r minha_pasta` |
| **`rmdir`** | Remove um diretório **apenas se estiver vazio**. | N/A | `rmdir pasta_vazia` |
| **`cp`** | Copia arquivos e diretórios. | `-r` (Cópia recursiva de diretórios) | `cp arquivo.txt /destino/` |
|  |  |  | `cp -r pasta_origem /destino/` |
| **`mv`** | Move arquivos/diretórios OU Renomeia arquivos/diretórios. | N/A | `mv old.txt new.txt` (Renomear) |
|  |  |  | `mv arquivo.txt /destino/` (Mover) |

---

## 🖥️ Monitoramento de Sistema e Processos

| **Comando** | **Função** | **Flags** | **Exemplo de Uso** |
| --- | --- | --- | --- |
| **`df`** | Mostra o uso do sistema de arquivos (espaço em disco). | `-h` (Human-readable: formato legível) | `df -h` |
| **`free`** | Mostra o uso da memória RAM (Memória Livre). | `-h` (Human-readable: formato legível) | `free -h` |
| **`uname`** | Exibe informações sobre o sistema operacional. | N/A | `uname` (Retorna "Linux") |
|  |  | `-a` (All: Exibe todas as informações detalhadas) | `uname -a` |
| **`ps aux`** | Lista todos os processos ativos no sistema. | N/A | `ps aux` |
| **`kill`** | Encerra um processo usando seu ID (PID). | N/A | `kill 1234` |

### Filtragem de Processos (Pipe e Grep)

O caractere **`|` (pipe)** redireciona a saída de um comando para a entrada do próximo.

Bash

`ps aux | grep billing`

> Explicação: Pega todos os processos (ps aux) e filtra (grep) apenas aqueles que contêm a palavra "billing" no nome.
> 

---

## 🔎 Análise e Busca de Arquivos

| **Comando** | **Função** | **Flags** | **Objetivo** |
| --- | --- | --- | --- |
| **`find`** | Realiza buscas complexas no sistema de arquivos. | `/` (Começa a busca na raiz) | `find / -name "billing"` |
|  |  | `-name` (Busca pelo nome exato) | Retorna arquivos ou diretórios com o nome "billing". |
| **`grep`** | Filtra linhas que correspondem a um padrão. | `*` (Wildcard: Coringa que representa zero ou mais caracteres) | Usado para encontrar configurações: `find / -name "*conf*" |
| **`diff`** | Compara o conteúdo de dois arquivos. | N/A | `diff arquivo1.txt arquivo2.txt` |

### Análise de Logs

| **Comando** | **Função** | **Flags** | **Observação** |
| --- | --- | --- | --- |
| **`head`** | Exibe as **10 primeiras** linhas de um arquivo. | N/A | Ideal para verificar o início de logs grandes. |
| **`tail`** | Exibe as **10 últimas** linhas de um arquivo. | N/A | Ideal para ver eventos mais recentes. |
| **`tail -f`** | Exibe o final do arquivo e **acompanha** em tempo real (`follow`). | `-f` | Pare o acompanhamento com `Ctrl + C`. |
| **`cat`** | Exibe o conteúdo **completo** de um arquivo. | N/A | Use com cautela em arquivos muito grandes. |

### Redirecionamento de Saída (`>`)

O caractere **`>`** redireciona a saída de um comando para um arquivo, sobrescrevendo o conteúdo existente.

Bash

`tail app.log > erro.txt`

> Explicação: Pega as últimas 10 linhas de app.log e as salva no arquivo erro.txt (criando-o se não existir).
> 

---

## ✏️ Edição e Permissões

| **Comando** | **Função** | **Observação** |
| --- | --- | --- |
| **`nano`** | Abre um editor de texto simples no terminal. | Use para fazer edições rápidas e diretas em arquivos. |
| **`chmod`** | Altera as permissões de acesso de um arquivo ou diretório. | Usa notação octal (r=4, w=2, x=1). |

### Exemplo de `chmod` (Notação Octal)

O formato é `chmod [DONO][GRUPO][OUTROS] arquivo`.

| **Permissão (Octal)** | **Descrição** | **Exemplo** |
| --- | --- | --- |
| **`644`** | Dono: Leitura e Escrita (4+2=6). Grupo/Outros: Apenas Leitura (4). | `chmod 644 arquivo.txt` |
| **`755`** | Dono: Leitura, Escrita, Execução (4+2+1=7). Grupo/Outros: Leitura e Execução (4+1=5). | Usado frequentemente para scripts executáveis. |
