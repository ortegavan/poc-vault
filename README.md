# Configuração do Vault para ambiente de dev

Para a documentação, acesse [https://www.vaultproject.io/docs](https://www.vaultproject.io/docs)

## Instalação 

### No Mac

```
brew tap hashicorp/tap
brew install hashicorp/tap/vault
```

### No Windows

```
choco install vault
```
Ou baixe o binário em [https://www.vaultproject.io/downloads](https://www.vaultproject.io/downloads)

## Verificar a instalação

```
vault
```

## Iniciar o server 

```
vault server -dev
```

Em um novo terminal:

a) Especifique qual servidor vai responder os requests

```
export VAULT_ADDR='http://127.0.0.1:8200'
```

b) Configure o token de autenticação com o server (substituir pelo token exibido ao verificar a instalação)

```
export VAULT_TOKEN="s.XmpNPoi9sRhYtdKHaQhkHP6x"
```

c) Verifique se o server está rodando

```
vault status
```

## Gerenciar segredos

Todos os segredos precisam estar em "secret/"

a) Cria um segredo

```
vault kv put secret/hello foo=world
```

b) Lê um segredo

```
vault kv get secret/hello
```

c) Exclui um segredo

```
vault kv delete secret/hello
```

d) Exibe a ajuda

```
vault kv -help 
```

## Autenticação 

a) O login no server é feito via token. Para gerar um novo token (que herda as políticas do atual) e se autenticar na sequência, execute:

```
vault token create
vault login s.iyNUhq8Ov4hIAx6snw5mB2nL
```
usando o token criado.

b) Para revogar um token:

```
vault token revoke s.iyNUhq8Ov4hIAx6snw5mB2nL
```

## Autorização

### Criando uma política

a) Escreve uma política

```
vault policy write my-policy - << EOF
path "secret/data/*" {
  capabilities = ["create", "update"]
}

path "secret/data/foo" {
  capabilities = ["read"]
}
EOF
```

b) Lista as políticas

```
vault policy list
```

c) Exibe o conteúdo de uma política

```
vault policy read my-policy
```

### Testando uma política

a) Cria um token e associa a política com este token

```
export VAULT_TOKEN="$(vault token create -field token -policy=my-policy)"
```

b) Escreve um segredo ("data" é adicionado automaticamente no put)

```
vault kv put secret/creds password="my-long-password"
```

c) Tenta escrever um segredo no caminho não permitido na política

```
vault kv put secret/foo robot=beepboop
```

# Deploy Vault

## Configuração 

### Criar a pasta que armazenará os segredos 

```
mkdir -p ./vault/data
```

### Criar o arquivo .hcl conforme abaixo (especificando a pasta criada anteriormente) - em produção usar TLS

```
storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_disable = "true"
}

disable_mlock = true
api_addr     = "http://127.0.0.1:8200"
cluster_addr = "https://127.0.0.1:8201"
ui           = true
```

### Criar o server especificando o arquivo .hcl

```
vault server -config=config.hcl
```

## Iniciar o server

```
export VAULT_ADDR='http://127.0.0.1:8200'
vault operator init
```
Salvar as keys e o token apresentados

## Habilitar o server pra ser usado

```
vault operator unseal
```
Executar 3 vezes cada uma com uma das 5 keys fornecidas no passo anterior

## Se autenticar no server

```
vault login s.KkNJYWF5g0pomcCLEmDdOVCW
```
Usar o token fornecido ao iniciar o server

## Habilitar o engine de chave/valor

```
vault secrets enable -version=1 kv
```

## Gerenciar segredos 

a) Cria um segredo 

```
vault kv put kv/my-secret my-value=s3cr3t
```

b) Lê um segredo 

```
vault kv get kv/my-secret
```

Para saber como ler segredos via API consulte o exemplo em .NET no projeto Teste ou em JavaScript na index.html

## Criar uma política só de leitura

```
 vault policy write my-policy policy.hcl
```

Ver arquivo policy.hcl de exemplo

## Gerar um token com esta política

```
vault token create -policy=my-policy 
```