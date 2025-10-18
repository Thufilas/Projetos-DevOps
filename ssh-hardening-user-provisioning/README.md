# 🛡️ Hardening e Provisionamento de Acesso via SSH

Este guia detalha o processo de aumento da segurança (Hardening) do serviço SSH em um servidor Linux (Ubuntu), desabilitando o acesso por senha e pelo usuário `root`, e padronizando a criação de novos usuários com chaves públicas (o "Esqueleto").

## 🎯 Objetivo

- Garantir que apenas o acesso via **chave pública SSH** seja permitido.
- Desabilitar o login direto para o usuário `root`.
- Padronizar o provisionamento de novos usuários.

## 1. Preparação e Instalação

Antes de configurar o SSH, garanta que o sistema esteja atualizado e o servidor SSH esteja instalado.

| **Passo** | **Comando** | **Descrição** |
| --- | --- | --- |
| **Atualização de Pacotes** | `sudo apt update` | Atualiza a lista de pacotes disponíveis. |
| **Upgrade do Sistema (Opcional)** | `sudo apt upgrade` | Instala todas as atualizações de pacotes. |
| **Instalação do SSH** | `sudo apt install openssh-server` | Instala o servidor SSH (OpenSSH). |
| **Elevação de Permissão** | `sudo su` | Entra no modo de usuário `root` (necessário para editar arquivos críticos). |

## 2. Configuração de Segurança (Hardening) do SSH

Em vez de editar o arquivo principal (`sshd_config`), usaremos um arquivo de provisionamento que é a prática recomendada em sistemas modernos, garantindo que nossas mudanças tenham prioridade.

### 2.1. Criar o Arquivo de Provisionamento

1. **Acessar a Pasta de Configurações Dinâmicas:**Bash
    
    `cd /etc/ssh/sshd_config.d`
    
2. Criar o Arquivo de Configuração:Bash
    
    Usaremos 01-ssh-config.conf para garantir que ele seja lido primeiro e sobrescreva o arquivo principal, se necessário.
    
    `nano 01-ssh-config.conf`
    
3. Adicionar as Configurações de Segurança:Snippet de código
    
    Cole as seguintes linhas no arquivo:
    
    `# 1. Autenticacao via Chave Publica
    PubkeyAuthentication yes
    
    # 2. Desabilitar Acesso via Senha
    PasswordAuthentication no
    PermitEmptyPasswords no
    
    # 3. Desabilitar Acesso Root Direto
    PermitRootLogin no`
    

### 2.2. Aplicar e Verificar o Serviço

Após salvar e fechar o `nano` (`Ctrl+O`, `Enter`, `Ctrl+X`), precisamos reiniciar o serviço para aplicar as novas regras.

| **Comando** | **Objetivo** |
| --- | --- |
| `service ssh restart` | Reinicia o serviço SSH. |
| `service ssh status` | Verifica se o serviço foi reiniciado com sucesso e está ativo (`active (running)`). |

---

## 3. Padronização e Provisionamento de Novos Usuários

Usaremos o diretório `/etc/skel` (Esqueleto) para garantir que todos os novos usuários criados já tenham uma pasta `.ssh` pronta para receber as chaves públicas.

### 3.1. Criar o Esqueleto de Acesso

1. **Acessar o Diretório Esqueleto:**Bash
    
    `cd /etc/skel`
    
2. **Criar a Pasta Oculta para Chaves:**Bash
    
    `mkdir .ssh`
    

### 3.2. Criar e Configurar o Novo Usuário

| **Passo** | **Comando** | **Descrição** |
| --- | --- | --- |
| **Criação do Usuário** | `useradd -m -s /bin/bash joao` | Cria o usuário `joao`, utilizando o diretório inicial (`-m`) e definindo o shell (`-s`) como `bash`. |
| **Definir a Senha** | `passwd joao` | Define uma senha para o usuário (necessário para alguns comandos ou login local, mesmo que o SSH esteja bloqueado para senhas). |

### 3.3. Adicionar a Chave Pública (Autorização)

1. **Acessar o Diretório do Novo Usuário:**Bash
    
    `cd /home/joao
    # Ou 'cd /home/thufilas' se o seu usuário for 'thufilas'`
    
    > Verificação: Use ls -a para confirmar que a pasta .ssh criada no Esqueleto foi copiada para /home/joao.
    > 
2. Autorizar a Chave Pública:Bash
    
    Copie sua chave pública (id_ed25519.pub ou id_rsa.pub) para o arquivo authorized_keys dentro da pasta .ssh.
    
    `# Exemplo de cópia (ajuste o caminho da sua chave pública)
    cat /caminho/para/sua/chave.pub > /home/joao/.ssh/authorized_keys`
    
    > Importante: O arquivo authorized_keys é o que o SSH verifica para permitir o login sem senha.
    > 

---

## 4. Teste Final de Conexão

Com as configurações aplicadas, a conexão só deve funcionar utilizando a chave privada correspondente.

Abra o terminal na sua máquina local (Windows PowerShell, CMD ou Bash) onde a chave privada está salva:

Bash

`# Substitua 'joao' pelo nome do usuário e o IP pela sua máquina
ssh joao@192.168.1.16`

Se a configuração estiver correta, a conexão será estabelecida sem que seja solicitada a senha.

<img width="2410" height="961" alt="image" src="https://github.com/user-attachments/assets/13900d04-06df-4a71-a3d6-de45d4938d9f" />
