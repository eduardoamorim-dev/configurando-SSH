# Guia: Configurando SSH no GitHub

Este guia resolve o problema de autenticação no Git, eliminando a necessidade de digitar senhas ou tokens (PAT) em cada operação.

## Por que usar SSH?

Desde 2021, o GitHub não aceita mais senhas de conta via terminal. O SSH é o método mais seguro e prático porque:

- **Segurança**: Usa criptografia de ponta a ponta
- **Agilidade**: Você não precisa digitar credenciais toda vez que der um push

---

## Passo a Passo

### **Etapa 1: Gerar sua chave SSH**

**1.1** Abra seu terminal (Git Bash no Windows, Terminal no Linux/macOS)

**1.2** Execute o comando abaixo substituindo pelo **seu e-mail do GitHub**:

```bash
ssh-keygen -t ed25519 -C "seu-email@exemplo.com"
```

**1.3** Você verá estas perguntas. Aperte **Enter** em todas para aceitar os padrões:

```
Enter file in which to save the key (/home/user/.ssh/id_ed25519): [Enter]
Enter passphrase (empty for no passphrase): [Enter]
Enter same passphrase again: [Enter]
```

> **Dica**: Não definir senha (passphrase) facilita o uso diário, mas é menos seguro. Escolha conforme sua necessidade.

**1.4** Pronto! Sua chave foi criada em `~/.ssh/id_ed25519`

---

### **Etapa 2: Iniciar o SSH Agent**

**2.1** Inicie o agente SSH com o comando:

```bash
eval "$(ssh-agent -s)"
```

**2.2** Você verá algo como: `Agent pid 12345`

---

### **Etapa 3: Adicionar a chave ao SSH Agent**

**3.1** Escolha o comando de acordo com seu sistema operacional:

**macOS:**
```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

**Windows (Git Bash ou WSL):**
```bash
ssh-add ~/.ssh/id_ed25519
```

**Linux:**
```bash
ssh-add ~/.ssh/id_ed25519
```

**3.2** Você verá: `Identity added: /home/user/.ssh/id_ed25519`

---

### **Etapa 4: Copiar a chave pública**

**4.1** Use o comando adequado para copiar sua chave pública:

**macOS:**
```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

**Windows (PowerShell):**
```bash
cat ~/.ssh/id_ed25519.pub | clip
```

**Windows (Git Bash):**
```bash
cat ~/.ssh/id_ed25519.pub | clip.exe
```

**Linux:**
```bash
cat ~/.ssh/id_ed25519.pub
```
> No Linux, selecione e copie o texto que aparecer (começa com `ssh-ed25519`)

---

### **Etapa 5: Adicionar a chave no GitHub**

**5.1** Acesse [github.com](https://github.com) e faça login

**5.2** Clique na sua foto de perfil (canto superior direito) → **Settings**

**5.3** No menu lateral esquerdo, clique em **SSH and GPG keys**

**5.4** Clique no botão verde **New SSH key**

**5.5** Preencha:
- **Title**: Dê um nome para identificar (ex: "PC Pessoal", "Notebook Trabalho")
- **Key**: Cole a chave que você copiou (Ctrl+V / Cmd+V)

**5.6** Clique em **Add SSH key**

**5.7** Confirme com sua senha do GitHub se solicitado

---

### **Etapa 6: Testar a conexão**

**6.1** No terminal, execute:

```bash
ssh -T git@github.com
```

**6.2** Na primeira vez, você verá:
```
The authenticity of host 'github.com' can't be established...
Are you sure you want to continue connecting (yes/no)?
```
Digite `yes` e aperte Enter

**6.3** Se tudo deu certo, você verá:
```
Hi seu-usuario! You've successfully authenticated, but GitHub does not provide shell access.
```

✅ **Parabéns! Sua chave SSH está configurada!**

---

## Convertendo repositórios existentes de HTTPS para SSH

Se você já tem repositórios clonados via HTTPS e o Git ainda pede senha:

**Passo 1:** Entre na pasta do seu repositório:
```bash
cd /caminho/do/seu/repositorio
```

**Passo 2:** Verifique a URL atual:
```bash
git remote -v
```

Se aparecer algo como `https://github.com/USUARIO/REPOSITORIO.git`, você precisa trocar.

**Passo 3:** Troque para SSH:
```bash
git remote set-url origin git@github.com:USUARIO/REPOSITORIO.git
```

**Passo 4:** Confirme a mudança:
```bash
git remote -v
```

Agora deve aparecer `git@github.com:USUARIO/REPOSITORIO.git`

**Passo 5:** Teste com um push:
```bash
git push
```

Não deve mais pedir senha! 

---

## ❓ Resolução de Problemas

### "Permission denied (publickey)"
- Verifique se adicionou a chave correta no GitHub
- Confirme se o SSH Agent está rodando: `eval "$(ssh-agent -s)"`
- Execute novamente: `ssh-add ~/.ssh/id_ed25519`

### "Could not open a connection to your authentication agent"
- Execute: `eval "$(ssh-agent -s)"` antes de rodar `ssh-add`

### No Windows: "ssh-add: command not found"
- Use o Git Bash, não o CMD ou PowerShell comum
