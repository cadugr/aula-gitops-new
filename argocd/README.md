# 🚀 ArgoCD - Aula GitOps

Este diretório reúne os manifestos utilizados no curso de GitOps com ArgoCD. O objetivo é demonstrar, na prática, como instalar o ArgoCD em um cluster Kubernetes e gerenciá-lo seguindo o próprio modelo GitOps (o famoso "ArgoCD gerenciando a si mesmo"), além de exemplos de `Application` e `AppProject` cobrindo diferentes cenários de deploy.

## 📂 Estrutura do diretório

```
argocd/
├── install.yaml              # Manifesto oficial de instalação do ArgoCD
├── values.yaml                # Valores customizados (réplicas, Redis HA, etc.)
├── deployment.yaml            # Deployment/Service de exemplo (aplicação webpage)
├── app-argocd.yaml            # Application que faz o ArgoCD gerenciar este próprio repositório
└── applications/
    ├── project.yaml            # AppProject de exemplo (exemplo-project)
    ├── app-web.yaml             # Application "review-filmes"
    ├── app-webcolor.yaml        # Application "web-color" (com syncPolicy detalhado)
    ├── app-webcolor2.yaml       # Application "webcolor" vinculada ao exemplo-project
    ├── app-kustomize.yaml       # Application "conversao" (deploy via Kustomize)
    └── app-prometheus.yaml      # Application "prometheus" (deploy via Helm chart)
```

### ✨ Destaques

- **`app-argocd.yaml`**: Application que aponta para este próprio repositório (`argocd/`), com `directory.recurse: true`, fazendo com que o ArgoCD sincronize automaticamente todos os manifestos deste diretório — incluindo as próprias `Applications` dentro de `applications/`. É o padrão conhecido como *app of apps*.
- **`applications/project.yaml`**: exemplo de `AppProject`, restringindo repositórios de origem, destinos permitidos, whitelist/blacklist de recursos e janelas de sincronização (`syncWindows`).
- **`applications/app-prometheus.yaml`**: exemplo de deploy usando múltiplas fontes (`sources`), combinando um Helm chart público com um `values.yaml` vindo de outro repositório Git.
- **`applications/app-kustomize.yaml`**: exemplo de deploy usando Kustomize como ferramenta de templating.

## ✅ Pré-requisitos

- Um cluster Kubernetes acessível via `kubectl`.
- `kubectl` configurado apontando para o cluster desejado.
- Namespace `argocd` criado (`kubectl create namespace argocd`).

## ⚙️ Instalação básica

```bash
kubectl apply -n argocd -f install.yaml
```

Após a instalação, aplique o `Application` que faz o ArgoCD passar a se auto-gerenciar via GitOps:

```bash
kubectl apply -f app-argocd.yaml
```

---

## 💻 Argo CLI

O `argocd` é a CLI oficial para interagir com o ArgoCD (aplicações, projetos, sincronizações, etc.) diretamente pelo terminal, sem precisar usar a interface web.

### 📦 Instalação

```bash
# macOS (Homebrew)
brew install argocd

# Linux
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

### 🔑 Login em um ArgoCD rodando no cluster Kubernetes

Quando o ArgoCD está rodando dentro do cluster (como neste projeto) e você não tem um Ingress/LoadBalancer exposto publicamente, o caminho mais simples é usar `port-forward` para acessar o serviço da API/UI localmente:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Em outro terminal, recupere a senha inicial do usuário `admin` (gerada automaticamente na instalação e armazenada em um Secret):

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d
```

Com o `port-forward` ativo, faça login pela CLI:

```bash
argocd login localhost:8080 --username admin --password <senha-obtida-acima> --insecure
```

> O `--insecure` é necessário porque o `port-forward` normalmente não possui um certificado TLS válido para `localhost`. Em ambientes produtivos, prefira acessar via um endpoint com certificado válido e remover essa flag.

Após o primeiro login, é recomendável alterar a senha padrão:

```bash
argocd account update-password
```

### 🛠️ Comandos úteis

```bash
# Listar todas as aplicações
argocd app list

# Ver detalhes de uma aplicação específica
argocd app get web-color

# Sincronizar manualmente uma aplicação
argocd app sync web-color

# Acompanhar o status de sincronização em tempo real
argocd app wait web-color

# Ver o histórico de deploys de uma aplicação
argocd app history web-color

# Fazer rollback para uma revisão anterior
argocd app rollback web-color <ID-da-revisao>

# Listar os projetos (AppProject) configurados
argocd proj list

# Ver detalhes de um projeto
argocd proj get exemplo-project

# Listar os clusters conhecidos pelo ArgoCD
argocd cluster list

# Listar os repositórios Git/Helm configurados
argocd repo list

# Adicionar um novo repositório Git
argocd repo add https://github.com/cadugr/web-page-deploy.git

# Encerrar a sessão da CLI
argocd logout localhost:8080
```
