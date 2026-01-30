# Guia Definitivo: Autenticação Git SSH Automática

Este guia resolve o problema de autenticação no Git, eliminando a necessidade de digitar senhas ou tokens (PAT) em cada operação. Com a integração do script de automação, sua chave será carregada no momento em que você abrir o terminal.

---

## 🚀 Por que usar SSH?

Desde 2021, o GitHub não aceita mais senhas de conta via terminal. O SSH é o método mais seguro e prático porque:

- **Segurança**: Usa criptografia de ponta a ponta (**Ed25519**).
- **Agilidade**: Você não precisa digitar credenciais toda vez que der um push.
- **Automação**: Sua chave é gerenciada pelo sistema sem intervenção manual.

---

## Passo a Passo

### **Etapa 1: Gerar sua chave SSH**

1. Abra seu terminal (**Git Bash** no Windows, ou Terminal no Linux/macOS).
2. Execute o comando abaixo substituindo pelo **seu e-mail do GitHub**:

```bash
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"
```

3. Aperte **Enter** em todas as perguntas para aceitar os padrões (local do arquivo e senha vazia).

---

### **Etapa 2: Automatizar o SSH Agent (Script de Otimização)**

Para que você não precise iniciar o agente ou adicionar sua chave manualmente toda vez que abrir o terminal, vamos configurar o carregamento automático no seu perfil.

1. No terminal, abra seu arquivo de configuração:
   - **Windows (Git Bash) / Linux**: `nano ~/.bashrc`
   - **macOS**: `nano ~/.zshrc`

2. Cole o script abaixo exatamente no final do arquivo:

```bash
# --- Auto-launch SSH Agent ---
env=~/.ssh/agent.env

agent_load_env () { test -f "$env" && . "$env" >| /dev/null ; }

agent_start () {
    (umask 077; ssh-agent >| "$env")
    . "$env" >| /dev/null ; }

agent_load_env

# agent_run_state: 0=agent running w/ key; 1=agent w/o key; 2=agent not running
agent_run_state=$(ssh-add -l >| /dev/null 2>&1; echo $?)

if [ ! "$SSH_AUTH_SOCK" ] || [ $agent_run_state = 2 ]; then
    agent_start
    ssh-add ~/.ssh/id_ed25519
elif [ "$SSH_AUTH_SOCK" ] && [ $agent_run_state = 1 ]; then
    ssh-add ~/.ssh/id_ed25519
fi

unset env
# -----------------------------
```

3. Salve e saia (`Ctrl + O`, `Enter`, `Ctrl + X`).
4. Force a atualização do terminal para ativar o script:
```bash
source ~/.bashrc
```

---

### **Etapa 3: Copiar a chave pública**

Use o comando adequado para o seu sistema:

- **Windows (Git Bash)**: 
```bash
cat ~/.ssh/id_ed25519.pub | clip.exe
```
- **macOS**: 
```bash
pbcopy < ~/.ssh/id_ed25519.pub
```
- **Linux**: 
```bash
cat ~/.ssh/id_ed25519.pub
```
*(No Linux, selecione o texto que começa com `ssh-ed25519` e copie manualmente).*

---

### **Etapa 4: Adicionar a chave no GitHub**

1. Acesse [github.com/settings/keys](https://github.com/settings/keys).
2. Clique em **New SSH key**.
3. **Title**: Dê um nome (ex: "PC Trabalho").
4. **Key**: Cole o conteúdo que você copiou.
5. Clique em **Add SSH key**.

---

### **Etapa 5: Testar a conexão**

No terminal, execute:
```bash
ssh -T git@github.com
```
Na primeira vez, digite `yes` e aperte **Enter**. A mensagem de sucesso deve ser:
> *Hi [seu-usuario]! You've successfully authenticated...*

---

## 🔄 Convertendo repositórios de HTTPS para SSH

Se você já tem repositórios clonados que ainda pedem senha, rode isso dentro da pasta do projeto:

1. Troque a URL (substitua pelo seu link SSH do repo):
```bash
git remote set-url origin git@github.com:USUARIO/REPOSITORIO.git
```

2. Teste o envio:
```bash
git push
```

---

## ❓ Resolução de Problemas

- **"Permission denied (publickey)"**: Verifique se colou a chave completa no GitHub (incluindo o `ssh-ed25519`).
- **"ssh-add: command not found"**: No Windows, use o **Git Bash**. O CMD ou PowerShell comum não reconhecem esses comandos nativamente sem configuração extra.
