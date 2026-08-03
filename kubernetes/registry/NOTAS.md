# Autenticação no Docker Registry (`registry:2`) com htpasswd

## 1. O que é o htpasswd

`htpasswd` é um utilitário (originado do Apache) usado para criar e gerenciar arquivos com pares **usuário:senha**, onde a senha é armazenada de forma criptografada (hash), nunca em texto puro.

A imagem oficial `registry:2` suporta autenticação básica HTTP (*basic auth*) usando exatamente esse tipo de arquivo. Quando configurado, o registry passa a exigir usuário e senha em toda requisição (`docker login`, `docker push`, `docker pull`).

Formato de uma linha do arquivo `htpasswd`:

```
usuario:$2y$05$hash_da_senha_aqui
```

> Importante: o `registry:2` exige que o hash seja **bcrypt** (`$2y$`). Hashes MD5 ou SHA não funcionam.

---

## 2. Gerando o arquivo htpasswd

### 2.1. Localmente, com o comando `htpasswd` instalado na máquina

Se o pacote `apache2-utils` (Debian/Ubuntu) ou `httpd-tools` (RHEL/CentOS/Fedora) já estiver instalado, basta usar o comando diretamente:

```bash
# Debian/Ubuntu
sudo apt-get install -y apache2-utils

# RHEL/CentOS/Fedora
sudo yum install -y httpd-tools
```

Gerando o arquivo (a flag `-c` cria o arquivo do zero — use só na primeira vez):

```bash
htpasswd -Bbn usuario senha123 > htpasswd
```

Adicionando mais usuários ao mesmo arquivo (sem `-c`, para não sobrescrever):

```bash
htpasswd -Bbn usuario2 senha456 >> htpasswd
```

Explicação das flags:
- `-B` → força o uso de bcrypt (obrigatório para o registry)
- `-b` → permite passar a senha diretamente na linha de comando
- `-n` → imprime o resultado no stdout em vez de gravar direto num arquivo (por isso o `> htpasswd` / `>> htpasswd`)

### 2.2. Alternativa via Docker (sem instalar nada)

Se preferir não instalar o pacote na máquina, o mesmo binário pode ser executado a partir da imagem `httpd`:

```bash
docker run --rm --entrypoint htpasswd httpd:2 -Bbn usuario senha123 > htpasswd
docker run --rm --entrypoint htpasswd httpd:2 -Bbn usuario2 senha456 >> htpasswd
```

O arquivo `htpasswd` resultante terá algo como:

```
usuario:$2y$05$abcdef1234567890...
usuario2:$2y$05$0987654321fedcba...
```

---

## 3. Como o registry:2 usa esse arquivo

O container `registry:2` lê duas variáveis de ambiente para habilitar a autenticação:

| Variável | Função |
|---|---|
| `REGISTRY_AUTH` | Define o tipo de autenticação (`htpasswd`) |
| `REGISTRY_AUTH_HTPASSWD_REALM` | Nome do "realm" (rótulo exibido no login) |
| `REGISTRY_AUTH_HTPASSWD_PATH` | Caminho **dentro do container** onde está o arquivo htpasswd |

Exemplo rodando localmente com Docker (sem Kubernetes), só para referência:

```bash
docker run -d -p 5000:5000 \
  -v $(pwd)/htpasswd:/auth/htpasswd \
  -e REGISTRY_AUTH=htpasswd \
  -e REGISTRY_AUTH_HTPASSWD_REALM=Registry \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  registry:2
```

---

## 4. Integrando com Secret no Kubernetes

A ideia é: em vez de montar o arquivo `htpasswd` via ConfigMap/volume comum, você o guarda como um **Secret**, e monta esse Secret como arquivo dentro do Pod.

### 4.1. Criar o Secret a partir do arquivo htpasswd

```bash
kubectl create secret generic registry-htpasswd \
  --from-file=htpasswd=./htpasswd \
  -n registry
```

Isso cria um Secret chamado `registry-htpasswd` com uma chave `htpasswd`, cujo valor é o conteúdo do arquivo (em base64, como todo Secret).

Você pode confirmar com:

```bash
kubectl get secret registry-htpasswd -n registry -o yaml
```
