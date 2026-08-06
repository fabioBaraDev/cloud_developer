# Cloud Developer II — Kubernetes e Serverless

**Apostila da disciplina · 2º semestre**

---

## Ficha da disciplina

| Item | Descrição |
|---|---|
| Disciplina | Cloud Developer II — Orquestração de Containers e Computação Serverless |
| Carga horária | 22 h/aula (11 encontros de 2 horas) + trabalhos extraclasse |
| Pré-requisito | Cloud Developer I (Docker: containers, imagens, Dockerfile, volumes, networks, Docker Hub, segurança) |
| Modalidade | Aula expositiva-dialogada com laboratório guiado; exercícios hands-on realizados em casa |
| Ambiente | Minikube (local) e Amazon EKS (nuvem); alternativa sem custo com kind/k3d |
| Linguagem dos exemplos | Python 3.12 + FastAPI |
| Avaliação | 3 Check Points (trabalhos) + 1 Global Solution (trabalho final, peso de prova) |
| Versão de referência | Kubernetes 1.33/1.34 · agosto de 2026 |

---

## Ementa

Limitações da execução isolada de containers e a necessidade de orquestração. Arquitetura do
Kubernetes: control plane, nós de trabalho e o modelo declarativo de reconciliação. Instalação e
operação de clusters locais e gerenciados. Modelagem de cargas de trabalho: Pods, Deployments,
StatefulSets, DaemonSets, Jobs e CronJobs. Serviços, descoberta, Ingress e políticas de rede.
Namespaces, cotas e isolamento multiambiente. Persistência de dados com volumes, PV, PVC e
StorageClass. Configuração e segredos. Autenticação, autorização (RBAC) e controle de admissão.
Boas práticas de operação, observabilidade e segurança em profundidade. Entrega contínua no modelo
GitOps. Integração com computação serverless: escalonamento dirigido a eventos, escala a zero e
funções gerenciadas.

---

## Objetivos gerais

Ao final da disciplina, o aluno deverá ser capaz de:

1. Explicar a arquitetura de um cluster Kubernetes e o funcionamento do loop de reconciliação.
2. Modelar e implantar aplicações containerizadas por meio de manifestos declarativos versionados.
3. Operar um cluster multi-nó em nuvem pública, incluindo rede, armazenamento e escalonamento.
4. Aplicar controles de segurança em profundidade (imagem, workload, cluster e nuvem).
5. Automatizar a entrega de software com pipelines de CI e reconciliação GitOps.
6. Escolher de forma fundamentada entre containers de longa duração e computação serverless.

---

## Cronograma de aulas

| Aula | Capítulo | Tema | Avaliação |
|---:|---:|---|---|
| 0 | Apêndice E | **Preparação do ambiente (assíncrona, antes da aula 1)** | — |
| 1 | 1 | Introdução aos orquestradores e arquitetura do Kubernetes | — |
| 2 | 2 | Instalação e ambientação em Kubernetes (Minikube) | — |
| 3 | 3 | Criando Pods e configurando Services (arquivos YAML) | — |
| 4 | 4 | Replicação, distribuição e alta disponibilidade | Lançamento do **Check Point 01** |
| 5 | 5 | Kubernetes Networking e Namespaces | Entrega do CP01 |
| 6 | 6 | Criando um cluster Kubernetes na nuvem (múltiplos nós) | — |
| 7 | 7 | Gerenciamento de recursos (volumes) e daemons | Lançamento do **Check Point 02** |
| 8 | 8 | Jobs, Secrets e Endpoints | Entrega do CP02 |
| 9 | 9 | Autenticação e controle de usuário (RBAC) | — |
| 10 | 10 | Boas práticas de operação e segurança | Lançamento do **Check Point 03** |
| 11 | 11 | GitOps (Argo CD e Rancher) | Entrega do CP03 · Lançamento da **Global Solution** |

> **Nota ao professor:** o cronograma acima assume 11 encontros. Se a grade da instituição previr
> semanas adicionais, recomenda-se usá-las como (a) aula de laboratório assistido antes de cada
> entrega de Check Point e (b) sessão de defesa/pitch da Global Solution.

---

## Aula 0 — Preparação do ambiente (obrigatória, antes da primeira aula)

Esta não é uma aula presencial: é uma **tarefa assíncrona de instalação** que o aluno executa em
casa, na semana anterior ao início da disciplina. Ela existe por um motivo prático: quase todos os
comandos de exploração do `kubectl` — incluindo `kubectl api-resources`, `kubectl explain` e até
`kubectl run --dry-run=client` — **precisam falar com um kube-apiserver**. Eles consultam o
*discovery endpoint* e o esquema OpenAPI do servidor para resolver o recurso e validar os campos.
Sem cluster, o retorno é `The connection to the server ... was refused`, e a aula 1 trava no
laboratório.

**O que instalar (passo a passo completo no Apêndice E):**

| Ferramenta | Verificação | Obrigatório em |
|---|---|---|
| Docker Engine / Docker Desktop | `docker run hello-world` | Aula 0 |
| kubectl 1.33+ | `kubectl version --client` | Aula 0 |
| minikube 1.35+ | `minikube version` | Aula 0 |
| Cluster local no ar | `minikube start && kubectl get nodes` | Aula 0 |
| Helm 3.16+, k9s | `helm version` | Até a aula 7 |
| AWS CLI v2, eksctl | `aws --version` | Até a aula 6 |

**Critério de pronto.** O aluno chega à aula 1 com esta saída funcionando:

```bash
minikube start --driver=docker
kubectl get nodes
# NAME       STATUS   ROLES           AGE   VERSION
# minikube   Ready    control-plane   45s   v1.33.1
```

> **Atenção — o erro mais comum do semestre.** Se aparecer
> `The connection to the server 127.0.0.1:<porta> was refused`, o `kubectl` está apontando para um
> cluster que **não existe mais** (Docker Desktop com Kubernetes desligado, ou um `minikube`/`kind`
> apagado). O binário está bem; o kubeconfig é que está órfão. Diagnostique com
> `kubectl config get-contexts` e `kubectl config current-context`, e resolva com `minikube start`
> ou `kubectl config use-context minikube`. A árvore completa de diagnóstico está no **Apêndice C**.

> **Nota ao professor:** vale reservar os primeiros 15 minutos da aula 1 para uma verificação rápida
> de ambiente em sala (`minikube status` e `kubectl get nodes` na tela de cada aluno). Turmas em que
> isso não é feito costumam perder o laboratório inteiro da aula 2 acertando instalação. Alunos com
> máquina insuficiente para virtualização devem ser direcionados ao Play with Kubernetes ou a um
> cluster compartilhado de demonstração — providencie essa alternativa antes da aula 1.

---

## Como usar esta apostila

Cada capítulo corresponde a **uma aula de duas horas** e segue sempre a mesma estrutura:

- **Objetivos de aprendizagem** — o que se espera ao final do encontro.
- **Roteiro da aula (120 min)** — divisão de tempo sugerida ao professor.
- **Desenvolvimento teórico** — o conteúdo conceitual, com diagramas e manifestos completos.
- **Laboratório guiado** — prática conduzida em sala, passo a passo.
- **Ponte com Serverless** — como o tema do capítulo se relaciona com Knative, KEDA, AWS Lambda e
  AWS Fargate. É por meio dessas seções que o eixo *serverless* percorre toda a disciplina.
- **Erros comuns e troubleshooting** — tabela de diagnóstico.
- **Resumo** — pontos-chave para revisão.
- **Exercícios** — **quatro atividades hands-on para casa**, em progressão: fixação, aplicação,
  depuração e desafio integrador. Cada exercício traz objetivo, enunciado, entregável, critério de
  aceite verificável e uma dica opcional.
- **Leitura recomendada** — fontes primárias para aprofundamento.

A **Parte II** reúne os enunciados, rubricas e roteiros de correção dos três Check Points; em
seguida vem a **Global Solution**, o trabalho final de maior peso. Ao fim, estão as **Referências
Bibliográficas** e os **Apêndices** (cheat sheet de `kubectl`, modelos de manifesto, guia de
troubleshooting, glossário, preparação do ambiente e mapa de competências CKA/CKAD).

### O fio condutor: FIAP Store

Todos os capítulos, laboratórios, exercícios e avaliações evoluem **a mesma aplicação**, um pequeno
e-commerce chamado **FIAP Store**:

| Componente | Tecnologia | Papel na disciplina |
|---|---|---|
| `store-api` | Python 3.12 + FastAPI (porta 8000) | Serviço principal: Pods, Deployments, probes, HPA, RBAC |
| `store-front` | Nginx servindo estáticos | Ingress, TLS, canary |
| `store-worker` | Python (processamento assíncrono) | Jobs, CronJobs, KEDA, escala a zero |
| `store-db` | PostgreSQL 16 | StatefulSet, PVC, Secrets |

Namespaces padrão: `store-dev` e `store-prod`. Labels padrão: `app`, `tier`, `env`.

> **Atenção:** os laboratórios em nuvem geram custo real. Todo capítulo que usa AWS traz um
> procedimento de destruição de recursos. Execute-o sempre ao final da prática e configure um
> AWS Budget antes de criar qualquer recurso.

---

## Sumário

**Parte I — Capítulos**

1. [Introdução aos Orquestradores de Containers e a Arquitetura do Kubernetes](#capítulo-1--introdução-aos-orquestradores-de-containers-e-a-arquitetura-do-kubernetes)
2. [Instalação e ambientação em Kubernetes (Minikube)](#capítulo-2--instalação-e-ambientação-em-kubernetes-minikube)
3. [Criando Pods e configurando Serviços (arquivos YAML)](#capítulo-3--criando-pods-e-configurando-serviços-arquivos-yaml)
4. [Replicação, Distribuição e Alta Disponibilidade](#capítulo-4--replicação-distribuição-e-alta-disponibilidade)
5. [Kubernetes Networking e Namespaces](#capítulo-5--kubernetes-networking-e-namespaces)
6. [Criando um Cluster Kubernetes na Nuvem (operação com múltiplos nós)](#capítulo-6--criando-um-cluster-kubernetes-na-nuvem-operação-com-múltiplos-nós)
7. [Gerenciamento de Recursos (Volumes) e Daemons](#capítulo-7--gerenciamento-de-recursos-volumes-e-daemons)
8. [Jobs, Secrets e Endpoints](#capítulo-8--jobs-secrets-e-endpoints)
9. [Autenticação e Controle de Usuário (RBAC)](#capítulo-9--autenticação-e-controle-de-usuário-rbac)
10. [Boas Práticas de Operação e Segurança](#capítulo-10--boas-práticas-de-operação-e-segurança)
11. [GitOps: entrega contínua declarativa com Argo CD e Rancher](#capítulo-11--gitops-entrega-contínua-declarativa-com-argo-cd-e-rancher)

**Parte II — Avaliações**

- Check Point 01 — Fundamentos, YAML e alta disponibilidade no Minikube
- Check Point 02 — Operação em cluster multi-nó: rede, namespaces e armazenamento
- Check Point 03 — Segurança, automação e entrega contínua com GitOps

**Global Solution — Trabalho Final**

**Referências Bibliográficas**

**Apêndices** — A: cheat sheet de kubectl · B: modelos de manifesto · C: troubleshooting ·
D: glossário · E: preparação do ambiente · F: mapa de competências

---

# Parte I — Capítulos

---

## Capítulo 1 — Introdução aos Orquestradores de Containers e a Arquitetura do Kubernetes

> **Aula:** 2 horas · **Pré-requisitos:** Docker (imagens, Dockerfile, volumes, networks, Docker Hub), linha de comando Linux, noções de HTTP e DNS, **Aula 0 concluída** (kubectl instalado e `minikube start` funcionando) · **Artefatos:** arquivos `pod-dry-run.yaml`, `mapa-arquitetura.md`, `relatorio-arquitetura.md`

> **Atenção — dependência de cluster.** A Trilha B do laboratório (seção 1.10) exige um cluster no
> ar. `kubectl api-resources`, `kubectl explain` e `kubectl run --dry-run=client` consultam o
> *discovery endpoint* e o esquema OpenAPI do **servidor**: sem cluster, todos falham com
> `The connection to the server ... was refused`. A Trilha A funciona sem cluster algum.

---

### 1.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as limitações operacionais do Docker e do Docker Compose em ambientes de produção e justificar a necessidade de um orquestrador de containers.
2. **Comparar** as principais plataformas de orquestração (Docker Swarm, HashiCorp Nomad, Apache Mesos, Amazon ECS e Kubernetes), explicando os fatores técnicos e de ecossistema que levaram o Kubernetes à posição dominante.
3. **Explicar** o modelo declarativo do Kubernetes e o funcionamento do *loop de reconciliação* (estado desejado versus estado atual), diferenciando-o do modelo imperativo.
4. **Descrever** cada componente do control plane e dos nós de trabalho, relacionando-os ao fluxo end-to-end disparado por um `kubectl apply`.
5. **Aplicar** os comandos de introspecção da API (`kubectl api-resources`, `kubectl explain`, `--dry-run=client -o yaml`) para investigar objetos, reconhecendo que todos eles dependem de um kube-apiserver acessível.
6. **Construir** um diagrama de arquitetura de deploy da aplicação FIAP Store, justificando a escolha de cada componente do Kubernetes.

---

### 1.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: recapitulação do 1º semestre (Docker) e o "muro" do `docker run` em produção |
| 2 | 20 min | Por que orquestrar? Escala, self-healing, service discovery, rolling update, bin packing |
| 3 | 15 min | Panorama de orquestradores e a história Borg → Kubernetes → CNCF |
| 4 | 15 min | Modelo declarativo e o loop de reconciliação |
| 5 | 10 min | **Intervalo** |
| 6 | 25 min | Arquitetura do Kubernetes: control plane, nós de trabalho, CRI/containerd e o fluxo do `kubectl apply` |
| 7 | 15 min | Laboratório guiado: Trilha A (mapa da arquitetura, sem cluster) e Trilha B (explorando a API com `explain`, `api-resources` e `--dry-run=client`) |
| 8 | 5 min | Ponte com Serverless: espectro IaaS → CaaS → PaaS → FaaS |
| 9 | 5 min | Fechamento, erros comuns e apresentação dos exercícios |

---

### 1.3 O ponto de partida: onde o Docker para de ser suficiente

No primeiro semestre construímos imagens, publicamos no Docker Hub, criamos volumes e redes, e subimos pilhas inteiras com `docker compose up`. Tudo funcionou. A pergunta honesta desta aula é: **por que isso não basta?**

Imagine que a FIAP Store — nosso e-commerce fio condutor do semestre — entrou em produção com um `docker-compose.yml` rodando em uma única máquina virtual na AWS. A arquitetura tem quatro serviços: `store-api` (Python 3.12 + FastAPI, porta 8000), `store-worker` (processamento de pedidos em batch), `store-db` (PostgreSQL 16) e `store-front` (Nginx servindo o estático). Em uma sexta-feira de Black Friday, a seguinte sequência de eventos acontece:

1. Às 19h02, o tráfego triplica. A `store-api` satura a CPU da VM. Você precisa de mais réplicas — mas o `docker compose up --scale store-api=5` só multiplica containers **na mesma máquina**, que já está no limite.
2. Às 19h14, o processo Python de uma das réplicas sofre um vazamento de memória e é morto pelo OOM Killer do kernel. O container morre. Ninguém percebe por 8 minutos, até um alerta de latência disparar.
3. Às 19h30, você provisiona uma segunda VM manualmente e sobe outra cópia do Compose lá. Agora tem dois bancos de dados independentes — um erro de configuração que só será descoberto no dia seguinte, na conciliação de pedidos.
4. Às 20h10, um deploy corretivo é feito com `docker compose up -d --build`. Todos os containers da `store-api` são recriados ao mesmo tempo. O serviço fica indisponível por 40 segundos.
5. Às 21h00, ninguém sabe dizer qual VM tem qual versão da imagem rodando.

Cada um desses cinco incidentes corresponde a **uma capacidade que um orquestrador de containers oferece nativamente**. Vamos nomeá-las.

#### 1.3.1 As seis capacidades que definem um orquestrador

**Escala horizontal entre múltiplas máquinas.** Um orquestrador trata um conjunto de servidores (físicos ou virtuais) como um **pool único de recursos computacionais**. Você não diz "rode este container na VM `srv-03`"; você diz "quero 5 réplicas da `store-api`, cada uma pedindo 250m de CPU e 256Mi de memória", e o orquestrador decide onde elas cabem. Essa mudança de vocabulário — de máquina para capacidade — é a essência da orquestração.

**Self-healing (auto-recuperação).** O orquestrador mantém um observador permanente comparando o que *deveria* existir com o que *existe*. Se um container morre, ele é recriado. Se a máquina inteira desaparece, as cargas que estavam nela são reagendadas em outras máquinas. O tempo entre a falha e a recuperação cai de minutos (humano acordando às 3h) para segundos.

**Service discovery e balanceamento de carga.** Se réplicas nascem e morrem o tempo todo, endereços IP fixos deixam de fazer sentido. O orquestrador fornece um nome estável (`store-api`) que resolve, via DNS interno, para um endereço virtual que distribui as requisições entre as réplicas saudáveis do momento. O `store-front` nunca precisa saber quantas réplicas existem nem onde elas estão.

**Rolling update e rollback.** Trocar a versão 1.0 pela 1.1 sem derrubar o serviço exige coreografia: sobe uma réplica nova, espera ela ficar *pronta* (não apenas "no ar" — pronta para receber tráfego), tira uma réplica velha do balanceador, repete. Se a nova versão falhar nos health checks, o processo para e volta atrás. Fazer isso à mão com Docker puro é possível; fazer isso de forma confiável, 30 vezes por dia, em 12 serviços, não é.

**Bin packing (empacotamento).** Dado um conjunto de máquinas com capacidades diferentes e um conjunto de cargas com requisitos diferentes, o problema de alocar cargas em máquinas maximizando a utilização é uma variação do problema clássico da mochila. O agendador do orquestrador resolve isso continuamente, e o ganho é econômico direto: clusters bem empacotados custam menos por requisição servida.

**Gestão declarativa de configuração e segredos.** Senhas de banco, chaves de API e arquivos de configuração deixam de ser variáveis de ambiente coladas num `docker-compose.yml` versionado por engano no Git, e passam a ser objetos de primeira classe do sistema, com controle de acesso próprio.

> **Nota:** o Docker Compose continua sendo uma ferramenta excelente — para ambientes de desenvolvimento local, para demonstrações e para aplicações de instância única. O erro não é usar Compose; é usá-lo como plataforma de produção multi-máquina. Ferramenta certa, contexto errado.

---

### 1.4 Panorama de orquestradores: quem competiu e por que o Kubernetes venceu

Entre 2014 e 2018 houve uma disputa real de mercado, hoje chamada retrospectivamente de "guerra dos orquestradores". Vale conhecer os competidores porque cada um deixou marcas conceituais.

| Plataforma | Origem | Modelo | Pontos fortes | Por que perdeu terreno |
|---|---|---|---|---|
| **Docker Swarm** | Docker Inc., 2015 | Declarativo simples, integrado ao Docker CLI | Curva de aprendizado quase nula; `docker service create` e pronto | Modelo de extensão pobre; sem API estendível; ecossistema fechado; a própria Docker Inc. reduziu investimento após 2019 |
| **Apache Mesos + Marathon** | Berkeley/Twitter, 2011 | Dois níveis de agendamento (Mesos oferece recursos, frameworks aceitam) | Escala comprovada (Twitter, Airbnb); roda cargas não-container (Spark, Hadoop) | Complexidade operacional alta; container era só um caso de uso entre vários; a DC/OS foi descontinuada |
| **HashiCorp Nomad** | HashiCorp, 2015 | Declarativo (HCL), binário único | Simplicidade operacional real; roda containers, binários, JVM e QEMU; integra com Consul/Vault | Ecossistema muito menor; menos operadores e ferramentas de terceiros; nicho de quem já vive no mundo HashiCorp |
| **Amazon ECS** | AWS, 2014 | Declarativo, proprietário | Integração nativa com IAM, ALB, CloudWatch; sem control plane para gerenciar | *Lock-in*: conhecimento e manifestos não são portáveis; a própria AWS lançou o EKS em 2018 |
| **Kubernetes** | Google, 2014 | Declarativo, orientado a API extensível | Modelo de recursos extensível (CRDs), ecossistema CNCF gigante, neutralidade de fornecedor | — |

O Kubernetes venceu por três razões que vale internalizar, porque elas explicam decisões de design que veremos o semestre inteiro:

**1. A API é o produto.** No Kubernetes, tudo — Pod, Service, Deployment, e também objetos que você mesmo inventar — é um recurso REST versionado numa API única. Isso significa que *estender* o Kubernetes não exige alterar o Kubernetes: basta registrar um novo tipo de recurso (`CustomResourceDefinition`) e escrever um controlador que o reconcilie. Prometheus, Argo CD, Istio, cert-manager e KEDA existem por causa dessa decisão. O Swarm nunca teve equivalente.

**2. Neutralidade de fornecedor.** Em 2015 o Google doou o Kubernetes à recém-criada **CNCF (Cloud Native Computing Foundation)**, sob a Linux Foundation. Nenhuma empresa isolada controla o projeto. Isso permitiu que Red Hat, Microsoft, AWS, VMware e centenas de startups investissem sem financiar um concorrente direto. A AWS, que tinha o ECS, lançou o EKS porque os clientes exigiram Kubernetes.

**3. Herança de operação real em escala.** O Kubernetes não nasceu de um whiteboard: é a terceira geração de um sistema que o Google roda há duas décadas.

#### 1.4.1 De Borg a Kubernetes

O **Borg** é o sistema interno de gerenciamento de clusters do Google, em produção desde ~2003, executando praticamente toda a computação da empresa (Search, Gmail, YouTube) sobre dezenas de milhares de máquinas por célula. Conceitos como *job*, *task*, *alloc*, cotas por namespace e agendamento com prioridade e preempção vêm de lá. O **Omega** (2013) foi uma tentativa de reescrita com estado compartilhado transacional e agendamento otimista — a maior parte não virou produto, mas influenciou o design do `etcd` como fonte única de verdade.

O **Kubernetes** (do grego *κυβερνήτης*, "timoneiro"/"piloto"; daí também o termo *cybernetics*) foi anunciado em junho de 2014 por Joe Beda, Brendan Burns e Craig McLuckie, com Tim Hockin e Brian Grant no núcleo técnico. A abreviação **K8s** conta as oito letras entre o K e o s. A versão 1.0 saiu em julho de 2015, no mesmo dia da fundação da CNCF. Desde então o projeto entrega três *releases* menores por ano; na data desta apostila (agosto/2026) estamos entre as versões **1.33 e 1.34**.

> **Na prática:** o artigo *"Borg, Omega, and Kubernetes"* (Burns et al., ACM Queue, 2016) é a leitura mais eficiente que existe sobre *por que* o Kubernetes é do jeito que é. Está indicado na leitura recomendada.

---

### 1.5 Declarativo versus imperativo e o loop de reconciliação

Esta é a seção conceitualmente mais importante do capítulo. Se você entender o loop de reconciliação, praticamente tudo o que vier depois no semestre — Deployments, HPA, Operators, GitOps — será uma variação do mesmo tema.

#### 1.5.1 Duas formas de pedir a mesma coisa

No modelo **imperativo**, você descreve a **sequência de ações**:

```bash
# Imperativo: eu digo COMO fazer, passo a passo
docker run -d --name store-api-1 -p 8001:8000 fiap/store-api:1.0
docker run -d --name store-api-2 -p 8002:8000 fiap/store-api:1.0
docker run -d --name store-api-3 -p 8003:8000 fiap/store-api:1.0
```

No modelo **declarativo**, você descreve o **estado final desejado**:

```yaml
# Declarativo: eu digo O QUE quero que seja verdade
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
    env: dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
    spec:
      containers:
        - name: store-api
          image: docker.io/seuusuario/store-api:1.0
          ports:
            - containerPort: 8000
```

A diferença não é de sintaxe, é de **responsabilidade**. No modelo imperativo, se um container morrer, o problema é seu — o comando já terminou de executar e ninguém mais está olhando. No modelo declarativo, o documento acima não é um comando: é um **contrato permanente**. Ele afirma "existem 3 réplicas da store-api", e o sistema assume a obrigação de tornar essa afirmação verdadeira, agora e daqui a seis meses.

#### 1.5.2 O loop de reconciliação

O mecanismo que sustenta esse contrato é o **loop de reconciliação** (também chamado *control loop*, por analogia com a teoria de controle da engenharia). A analogia canônica é o **termostato**: você declara "quero 22 °C" (estado desejado); o sensor lê 19 °C (estado atual); o controlador liga o aquecedor (ação corretiva); o ciclo se repete indefinidamente.

```
        +-----------------------------------------------+
        |                                               |
        v                                               |
  [ Ler estado desejado ]                                |
   (spec, vindo do etcd)                                 |
        |                                               |
        v                                               |
  [ Observar estado atual ]                              |
   (status dos objetos reais)                            |
        |                                               |
        v                                               |
  [ Existe divergência? ] -- não --> [ dormir/aguardar ]-+
        |                                               |
       sim                                              |
        |                                               |
        v                                               |
  [ Executar ação corretiva ] --------------------------+
   (criar/atualizar/remover)
```

Três propriedades desse desenho merecem atenção:

- **Nível, não borda (level-triggered, não edge-triggered).** O controlador não reage a "eventos de mudança"; ele reavalia o estado completo periodicamente. Se ele ficar 5 minutos fora do ar e voltar, ele simplesmente compara de novo e conserta. Sistemas edge-triggered perdem eventos e ficam permanentemente inconsistentes.
- **Idempotência.** Aplicar o mesmo manifesto dez vezes produz o mesmo resultado que aplicar uma vez. Isso é o que torna GitOps viável.
- **Composição de controladores independentes.** Não existe *um* loop; existem dezenas, cada um cuidando de um tipo de recurso, todos falando exclusivamente com o `kube-apiserver` e sem conhecimento uns dos outros. O controlador de Deployment cria ReplicaSets; o de ReplicaSet cria Pods; o scheduler atribui Pods a nós; o kubelet materializa containers. Cada um faz uma coisa pequena e o comportamento global emerge dessa composição.

Todo objeto do Kubernetes tem, por isso, duas metades: **`spec`** (o desejado, escrito por você) e **`status`** (o observado, escrito pelo sistema). Você nunca edita `status`; o sistema nunca decide o `spec`.

> **Atenção:** existem comandos imperativos no `kubectl` (`kubectl run`, `kubectl create deployment`, `kubectl scale`). Eles são ótimos para aprender e para emergências, mas em produção o estado deve viver em arquivos versionados no Git. Uma alteração feita com `kubectl scale` é sobrescrita no próximo `apply` — e é exatamente esse comportamento que queremos.

---

### 1.6 Arquitetura do Kubernetes

Um cluster Kubernetes é composto por um **control plane** (o cérebro, que decide) e um conjunto de **nós de trabalho** (worker nodes, os músculos, que executam). Em clusters gerenciados como o **Amazon EKS**, o control plane é operado pela AWS e você só enxerga um endpoint HTTPS; em clusters locais como o minikube, control plane e worker convivem na mesma máquina.

```
                            +----------------------------------+
   kubectl / CI / Argo CD   |                                  |
   -----------------------> |          CONTROL PLANE           |
        HTTPS (6443)        |                                  |
                            |  +----------------------------+  |
                            |  |      kube-apiserver        |<-+---> autenticação,
                            |  |  (unica porta de entrada)  |  |     autorização (RBAC),
                            |  +-----+----------------+-----+  |     admission controllers
                            |        |                |        |
                            |        v                |        |
                            |  +-----------+          |        |
                            |  |   etcd    |          |        |
                            |  | (estado)  |          |        |
                            |  +-----------+          |        |
                            |                         |        |
                            |   +---------------------+-----+  |
                            |   |                           |  |
                            |   v                           v  |
                            | +------------------+  +---------------------------+
                            | | kube-scheduler   |  | kube-controller-manager   |
                            | | (Pod -> Node)    |  | (Deployment, ReplicaSet,  |
                            | +------------------+  |  Node, Job, Endpoint...)  |
                            |                       +---------------------------+
                            |                                  |
                            |                       +---------------------------+
                            |                       | cloud-controller-manager  |
                            |                       | (ELB, EBS, rotas da VPC)  |
                            |                       +---------------------------+
                            +----------------------------------+
                                          ^   ^   ^
                     watch/report (HTTPS)  |   |   |
                +-----------------------+  |   |   +-------------------------+
                |                          |   |                             |
     +----------+-----------+   +----------+---+--------+   +----------------+-----+
     |     WORKER NODE 1    |   |     WORKER NODE 2     |   |    WORKER NODE 3     |
     |                      |   |                       |   |                      |
     | +------------------+ |   | +-------------------+ |   | +------------------+ |
     | |     kubelet      | |   | |     kubelet       | |   | |     kubelet      | |
     | +---------+--------+ |   | +---------+---------+ |   | +---------+--------+ |
     |           | CRI      |   |           | CRI       |   |           | CRI      |
     | +---------v--------+ |   | +---------v---------+ |   | +---------v--------+ |
     | |   containerd     | |   | |   containerd      | |   | |   containerd     | |
     | |     + runc       | |   | |     + runc        | |   | |     + runc       | |
     | +---------+--------+ |   | +---------+---------+ |   | +---------+--------+ |
     |           |          |   |           |           |   |           |          |
     |  +--------v-------+  |   |  +--------v--------+  |   |  +--------v-------+  |
     |  | Pod store-api  |  |   |  | Pod store-api   |  |   |  | Pod store-db   |  |
     |  | Pod store-front|  |   |  | Pod store-worker|  |   |  |                |  |
     |  +----------------+  |   |  +-----------------+  |   |  +----------------+  |
     |                      |   |                       |   |                      |
     | +------------------+ |   | +-------------------+ |   | +------------------+ |
     | |   kube-proxy     | |   | |   kube-proxy      | |   | |   kube-proxy     | |
     | | (iptables/IPVS)  | |   | | (iptables/IPVS)   | |   | | (iptables/IPVS)  | |
     | +------------------+ |   | +-------------------+ |   | +------------------+ |
     |   + CNI plugin       |   |   + CNI plugin        |   |   + CNI plugin       |
     +----------------------+   +-----------------------+   +----------------------+
              |                            |                           |
              +----------------------------+---------------------------+
                          rede plana de Pods (CNI: Calico, Cilium,
                          VPC CNI da AWS, flannel...)
```

#### 1.6.1 Componentes do control plane

**kube-apiserver.** É o único componente com o qual todos os outros falam, e o único que fala com o etcd. Expõe a API REST do Kubernetes em HTTPS (porta 6443 por padrão). Toda requisição passa por quatro estágios: **autenticação** (quem é você — certificado cliente, token de ServiceAccount, OIDC, e no EKS o `aws-iam-authenticator`), **autorização** (você pode fazer isso? — RBAC), **admission controllers** (mutating, que podem alterar o objeto — ex.: injetar sidecars, aplicar defaults; e validating, que podem rejeitá-lo — ex.: Pod Security Admission) e finalmente **validação de esquema e persistência**. É stateless e horizontalmente escalável: rode três instâncias atrás de um balanceador e você tem alta disponibilidade.

**etcd.** Banco de dados chave-valor distribuído, consistente (algoritmo de consenso Raft), que guarda **todo** o estado do cluster. É a fonte única de verdade. Requer número ímpar de membros (3 ou 5) para tolerar falhas — um cluster de 3 tolera 1 falha; de 5, tolera 2. É extremamente sensível a latência de disco e de rede: use SSD e mantenha os membros na mesma região. **Fazer backup do etcd é fazer backup do cluster inteiro** (exceto o conteúdo dos volumes de dados). No EKS a AWS opera e faz backup do etcd por você — um dos principais argumentos econômicos a favor de clusters gerenciados.

**kube-scheduler.** Observa Pods com `spec.nodeName` vazio (ainda não agendados) e decide em qual nó cada um deve rodar. Trabalha em duas fases: **filtering** (elimina nós que não servem — sem CPU/memória suficiente, sem a porta livre, com taint não tolerado, sem o volume disponível na zona certa) e **scoring** (pontua os nós restantes conforme espalhamento de réplicas, afinidade, balanceamento de recursos). O nó vencedor é gravado em `spec.nodeName`. Note: o scheduler **não** inicia container nenhum; ele só escreve uma decisão na API.

**kube-controller-manager.** Um único binário que hospeda dezenas de controladores independentes, cada um com seu loop de reconciliação: `deployment-controller`, `replicaset-controller`, `node-controller` (marca nós como `NotReady` e despeja Pods após o *toleration seconds*), `job-controller`, `endpointslice-controller`, `serviceaccount-controller`, `namespace-controller`, entre outros.

**cloud-controller-manager.** Isola tudo o que é específico de um provedor de nuvem. Cuida de três coisas: criar balanceadores de carga reais quando você cria um `Service` do tipo `LoadBalancer` (no caso da AWS, um NLB ou ALB), gerenciar rotas de rede na VPC, e remover objetos `Node` cuja instância EC2 foi terminada. Foi extraído do controller-manager principal justamente para permitir que a AWS, o Google e a Microsoft mantivessem seus próprios códigos fora do núcleo do Kubernetes.

#### 1.6.2 Componentes dos nós de trabalho

**kubelet.** O agente que roda em cada nó. Recebe do apiserver a lista de Pods atribuídos àquele nó (via *watch*) e faz o que for necessário para que esses Pods existam: puxa imagens, cria containers através do runtime, monta volumes, executa as *probes* de liveness/readiness/startup e reporta o `status` de volta ao apiserver. O kubelet **não** gerencia containers que não vieram do Kubernetes. Um detalhe importante: se o kubelet perder contato com o control plane, ele continua mantendo os containers locais rodando — o cluster degrada, mas não desliga.

**kube-proxy.** Programa as regras de rede do nó para que o IP virtual de um `Service` (ClusterIP) seja traduzido para o IP real de um dos Pods de destino. Historicamente usa `iptables`; em clusters grandes usa `IPVS` (melhor desempenho, pois iptables avalia regras linearmente) e, mais recentemente, existe o modo `nftables`. Vale notar que CNIs modernos como o **Cilium** podem substituir o kube-proxy inteiramente usando eBPF.

**Container runtime e a CRI.** O kubelet não sabe criar containers — ele delega a um runtime através da **CRI (Container Runtime Interface)**, uma API gRPC padronizada. Os runtimes mais usados são o **containerd** (padrão em EKS, GKE, AKS, minikube e na maioria das distribuições) e o **CRI-O** (padrão no OpenShift). Ambos, por sua vez, chamam o **runc**, que efetivamente cria o processo isolado usando namespaces e cgroups do kernel Linux.

> **Nota — a saída do dockershim.** Até a versão 1.20, o kubelet trazia embutido um adaptador chamado **dockershim** que traduzia CRI para a API do Docker Engine. Ele foi depreciado na 1.20 e **removido na 1.24 (2022)**. Isso gerou o pânico do "Kubernetes vai parar de suportar Docker", que era falso em dois sentidos: (a) suas **imagens Docker continuam funcionando normalmente**, porque o formato de imagem é padronizado pela OCI (Open Container Initiative) e não tem nada de proprietário; (b) o Docker Engine, por baixo, sempre usou o containerd. O que saiu foi apenas a camada de tradução redundante. Na prática: você continua fazendo `docker build` e `docker push`; o cluster usa containerd para rodar.

**CNI (Container Network Interface).** O Kubernetes não implementa rede de Pods; ele define um modelo e delega a implementação a um plugin CNI. O modelo exige três coisas: todo Pod tem um IP próprio e roteável; todos os Pods conseguem se comunicar entre si sem NAT; e o Pod se enxerga com o mesmo IP que os outros enxergam. Implementações comuns: **Amazon VPC CNI** (atribui IPs reais da VPC aos Pods — importante para consumo de endereços no EKS), Calico, Cilium e flannel.

#### 1.6.3 O que acontece quando você roda `kubectl apply`

Este é o roteiro que amarra tudo. Suponha `kubectl apply -f deployment-store-api.yaml` com `replicas: 3`.

1. **kubectl lê o kubeconfig** (`~/.kube/config`), descobre o endereço do cluster, o contexto e as credenciais atuais.
2. **kubectl converte YAML em JSON** e monta uma requisição HTTPS. Se o objeto não existe, é um `POST /apis/apps/v1/namespaces/store-dev/deployments`; se existe, é um `PATCH` com estratégia de *merge*. O kubectl também grava a anotação `kubectl.kubernetes.io/last-applied-configuration` para conseguir calcular remoções em aplicações futuras.
3. **kube-apiserver autentica** o chamador (certificado, token ou, no EKS, uma assinatura IAM).
4. **kube-apiserver autoriza** via RBAC: este usuário pode criar Deployments no namespace `store-dev`?
5. **Admission controllers** rodam: os *mutating* podem injetar defaults (ex.: `imagePullPolicy`, ServiceAccount padrão, sidecar de service mesh); os *validating* podem rejeitar (ex.: Pod Security Admission barrando container privilegiado; ResourceQuota barrando excesso de recursos).
6. **Validação de esquema e persistência no etcd.** O objeto Deployment agora existe. **O apiserver responde `deployment.apps/store-api created` ao kubectl — e o comando termina aqui.** Tudo o que vem a seguir é assíncrono.
7. **deployment-controller** (no controller-manager) está em *watch* sobre Deployments. Vê o novo objeto e cria um **ReplicaSet** com `replicas: 3` e um hash do template no nome (`store-api-7d9f8c6b45`).
8. **replicaset-controller** vê o ReplicaSet, conta 0 Pods existentes com aquele selector, e cria **3 objetos Pod** — ainda sem nó atribuído, na fase `Pending`.
9. **kube-scheduler** vê 3 Pods com `spec.nodeName` vazio. Para cada um, filtra e pontua os nós, e escreve a decisão via *binding*.
10. **kubelet do nó escolhido** vê, no seu *watch*, que há um Pod atribuído a ele. Chama o CNI para configurar a rede do Pod (IP, interface, rotas), monta os volumes declarados e pede ao **containerd**, via CRI, que baixe a imagem (`ImagePulling`) e crie os containers.
11. **containerd → runc** cria o processo isolado. O kubelet começa a executar as probes; quando a readiness probe passa, o Pod é marcado `Ready`.
12. **endpointslice-controller** vê Pods `Ready` cujos labels casam com o selector de algum Service e adiciona seus IPs ao EndpointSlice correspondente.
13. **kube-proxy** de cada nó vê o EndpointSlice atualizado e reprograma iptables/IPVS. A partir deste instante, tráfego para o ClusterIP do Service alcança os Pods novos.

Repare que **nenhum componente conversa diretamente com outro**: todos leem e escrevem no apiserver. Essa topologia em estrela (hub-and-spoke) é o que permite substituir peças (outro scheduler, outro CNI, outro runtime) sem quebrar o resto.

---

### 1.7 O modelo de objetos da API

#### 1.7.1 Estrutura comum e GVK/GVR

Todo objeto do Kubernetes tem a mesma forma de quatro campos de topo:

```yaml
apiVersion: v1          # grupo + versão da API
kind: Pod               # tipo do objeto
metadata:               # nome, namespace, labels, annotations, uid...
  name: store-api
  namespace: store-dev
spec:                   # estado desejado (voce escreve)
  containers:
    - name: store-api
      image: docker.io/seuusuario/store-api:1.0
status:                 # estado observado (o sistema escreve)
  phase: Running
```

O par `apiVersion` + `kind` forma o **GVK (Group, Version, Kind)**. Grupos organizam a API em namespaces lógicos: o grupo vazio (chamado *core* ou *legacy*) contém Pod, Service, ConfigMap, Secret, Namespace e Node, e aparece como `apiVersion: v1`; os demais são nomeados, como `apps/v1` (Deployment, StatefulSet, DaemonSet, ReplicaSet), `batch/v1` (Job, CronJob), `networking.k8s.io/v1` (Ingress, NetworkPolicy) e `rbac.authorization.k8s.io/v1` (Role, RoleBinding).

O **GVR (Group, Version, Resource)** é a forma REST correspondente, com o nome do recurso em plural e minúsculo: o GVK `apps/v1 Deployment` corresponde ao GVR `apps/v1 deployments`, servido em `/apis/apps/v1/namespaces/<ns>/deployments`.

As versões seguem um esquema de estabilidade: `v1alpha1` (experimental, pode sumir sem aviso, desabilitado por padrão), `v1beta1` (bem testado, mas o esquema ainda pode mudar) e `v1` (estável, com garantia de compatibilidade). É por isso que o guia da disciplina proíbe `extensions/v1beta1` — foi removido há muitas versões.

#### 1.7.2 Namespaces

Namespaces são divisões lógicas do cluster. Servem para: escopo de nomes (pode haver um Service `store-api` em `store-dev` e outro em `store-prod`), aplicação de **ResourceQuota** e **LimitRange**, e delimitação de permissões RBAC. Não são fronteira de segurança forte por si só — dois Pods em namespaces diferentes conseguem se comunicar pela rede, a menos que existam NetworkPolicies.

Todo cluster nasce com `default`, `kube-system` (componentes do próprio Kubernetes), `kube-public` (dados legíveis por qualquer um) e `kube-node-lease` (heartbeats dos nós). Nesta disciplina usaremos `store-dev` e `store-prod`.

Nem todo recurso é namespaced: `Node`, `PersistentVolume`, `StorageClass`, `ClusterRole` e `Namespace` são recursos de cluster. Você descobre quais são quais na coluna `NAMESPACED` do `kubectl api-resources`.

#### 1.7.3 Labels, selectors e annotations

**Labels** são pares chave/valor usados para **selecionar** conjuntos de objetos. São a cola de todo o Kubernetes: um Service encontra seus Pods por labels; um ReplicaSet controla seus Pods por labels; o scheduler distribui réplicas usando labels de topologia.

```yaml
metadata:
  labels:
    app: store-api
    tier: backend
    env: dev
    app.kubernetes.io/version: "1.0"
```

Selectors vêm em duas formas. **Igualdade**: `app=store-api,env=dev` (vírgula significa E lógico). **Conjunto**: `env in (dev,staging)`, `tier notin (frontend)`, `app` (existe a chave), `!app` (não existe).

```bash
kubectl get pods -l 'app=store-api,env=dev'
kubectl get pods -l 'env in (dev,staging)'
kubectl get pods -L app,env      # -L maiusculo mostra o label como COLUNA
```

**Annotations** também são pares chave/valor, mas **não são selecionáveis**. Servem para carregar metadados arbitrários, frequentemente lidos por ferramentas externas: `prometheus.io/scrape: "true"`, `service.beta.kubernetes.io/aws-load-balancer-type: "nlb"`, `kubernetes.io/change-cause: "atualiza store-api para 1.1"`. Aceitam valores grandes e estruturados; labels são limitados a 63 caracteres e a um alfabeto restrito.

> **Na prática:** adote desde já as labels recomendadas pelo projeto (`app.kubernetes.io/name`, `/instance`, `/version`, `/component`, `/part-of`, `/managed-by`). Ferramentas como Helm, Argo CD e o próprio `kubectl` já as entendem, e sua vida com dashboards de observabilidade fica muito mais simples.

---

### 1.8 Distribuições, conformance e os limites do Kubernetes

#### 1.8.1 O programa Certified Kubernetes

A CNCF mantém o programa **Certified Kubernetes Conformance**: uma distribuição só pode usar a marca "Kubernetes" se passar na suíte de testes de conformidade (executada pela ferramenta **Sonobuoy**). Isso garante que um manifesto YAML válido funcione em qualquer distribuição certificada — a promessa de portabilidade que sustenta o ecossistema.

| Distribuição | Tipo | Contexto de uso | Observações |
|---|---|---|---|
| **Amazon EKS** | Gerenciada (AWS) | **Provedor oficial desta disciplina** | Control plane cobrado por hora por cluster; nós em EC2 ou Fargate; integra IAM, VPC CNI, ELB, IRSA |
| **Google GKE** | Gerenciada (GCP) | Referência histórica de maturidade | Modo Autopilot cobra por Pod, não por nó |
| **Azure AKS** | Gerenciada (Azure) | Integração com Entra ID | Control plane gratuito no tier free |
| **Red Hat OpenShift** | Distribuição corporativa | On-premises e nuvem | Adiciona Routes, BuildConfig, SCC, console próprio; opinativo e mais restritivo por padrão |
| **k3s** | Leve (Rancher/SUSE) | Edge, IoT, ARM, CI | Binário único < 70 MB; usa SQLite por padrão no lugar do etcd |
| **kind** | Local (nós em containers) | CI e testes | *Kubernetes IN Docker*; cria clusters multi-nó em segundos |
| **minikube** | Local (VM ou container) | **Ferramenta do Capítulo 2** | Rico em addons: dashboard, ingress, metrics-server, registry |
| **Rancher / RKE2** | Gestão multi-cluster | Governança de frota | Administra clusters de várias origens sob um painel |

#### 1.8.2 O que o Kubernetes NÃO faz

Tão importante quanto saber o que a ferramenta faz é saber onde ela termina. O Kubernetes **não**:

- **Constrói imagens.** Não existe `kubectl build`. Isso é papel de Docker, BuildKit, Kaniko ou Buildpacks, normalmente dentro do seu pipeline de CI.
- **É um sistema de CI/CD.** Ele é o *destino* do deploy, não o orquestrador do pipeline. GitHub Actions, GitLab CI, Jenkins, Argo CD e Flux cuidam disso.
- **Fornece middleware.** Não há banco de dados, fila ou barramento de mensagens embutidos. Você roda o PostgreSQL ou o Kafka *sobre* o Kubernetes (ou usa RDS/MSK gerenciados), mas o Kubernetes não os provê.
- **É um PaaS.** Não existe `git push` que vira aplicação. Camadas como OpenShift, Knative e Cloud Foundry constroem essa experiência em cima.
- **Faz logging e monitoramento completos.** Ele expõe métricas e logs de container, mas coletar, agregar e reter é trabalho de Prometheus, Grafana, Loki, Fluent Bit, OpenSearch ou CloudWatch.
- **Resolve arquitetura ruim.** Uma aplicação que guarda sessão em memória local, escreve em disco local sem volume, demora 4 minutos para subir ou não tolera ser morta a qualquer momento vai sofrer no Kubernetes — provavelmente mais do que sofria numa VM. O contrato do orquestrador pressupõe cargas que seguem, ao menos parcialmente, os princípios do *Twelve-Factor App*.

---

### 1.9 Ponte com Serverless

O termo "serverless" é infeliz — existem servidores, você só não os administra. Uma forma mais útil de pensar é como um **espectro de responsabilidade delegada**:

```
  VOCE GERENCIA MAIS                                          VOCE GERENCIA MENOS
  <-------------------------------------------------------------------------->

  IaaS              CaaS                  PaaS               FaaS
  (EC2)             (EKS, ECS)            (Elastic Beanstalk, (Lambda)
                                           App Runner, Knative)

  SO, patches       imagem, manifesto,    codigo + runtime    apenas a funcao
  runtime, deploy   escalonamento         gerenciado          + o gatilho
  balanceador       (cluster gerenciado)
```

O **Kubernetes vive no CaaS**: você entrega uma imagem de container e uma declaração de estado desejado; a plataforma cuida de agendamento, recuperação e rede. Você ainda pensa em réplicas, CPU, memória e nós.

O **AWS Lambda vive no FaaS**: você entrega uma função e um gatilho (HTTP via API Gateway, mensagem no SQS, evento do S3, agendamento do EventBridge). Não há réplica para dimensionar, não há nó, não há patch de SO. Você paga por milissegundo de execução e a plataforma escala de zero a milhares de execuções concorrentes automaticamente. O custo dessa conveniência aparece em três frentes: **cold start** (a primeira invocação após um período ocioso paga a inicialização do ambiente), **limites rígidos** (tempo máximo de execução de 15 minutos, tamanho do pacote, memória) e **modelo de programação restrito** (sem estado local persistente, sem conexões de longa duração fáceis de manter).

Entre os dois extremos existe uma faixa importante para nós: os **serverless containers**. O **AWS Fargate** roda Pods do EKS (ou tasks do ECS) sem que você provisione nenhuma instância EC2 — você declara o Pod, a AWS materializa a capacidade e cobra por vCPU-segundo e GB-segundo. É a **mesma API do Kubernetes** com o modelo operacional e de custo do serverless. O **AWS App Runner** vai um passo além, aproximando-se do PaaS.

Dentro do próprio Kubernetes, dois projetos trazem características serverless: o **Knative Serving**, que implementa escalonamento a zero e roteamento por revisão (sua `store-api` fica com zero Pods quando não há tráfego e sobe sob demanda), e o **KEDA (Kubernetes Event-Driven Autoscaling)**, que escala Deployments com base em métricas externas — profundidade de uma fila SQS, lag de um consumer group Kafka, número de mensagens no RabbitMQ. Usaremos ambos com a `store-worker` mais adiante no semestre.

A decisão prática, que você começará a exercitar já no primeiro exercício: **cargas de tráfego constante e previsível tendem a ser mais baratas em Kubernetes; cargas esporádicas, com picos raros e longos vales de ociosidade, tendem a ser mais baratas em FaaS.** Na FIAP Store, a `store-api` (tráfego contínuo) é uma candidata natural a Deployment no EKS, enquanto o processamento de notas fiscais que roda três vezes ao dia é candidato a Lambda ou a um Job escalado por KEDA.

---

### 1.10 Laboratório guiado (feito em sala)

**Contexto.** Este laboratório tem **duas trilhas**, e a ordem importa.

A **Trilha A** é conceitual: roda em qualquer máquina, com papel e caneta ou um editor de texto, sem
cluster e sem `kubectl`. É ela que consolida a arquitetura que acabamos de estudar, e é o plano de
contingência para quem chegou à aula com o ambiente quebrado.

A **Trilha B** explora a API de verdade e **exige um cluster no ar** — é para isso que existe a
Aula 0. Se `kubectl get nodes` não responder, faça a Trilha A agora e a Trilha B depois de resolver
o ambiente (o Capítulo 2 é inteiro dedicado a isso).

> **Por que a Trilha B exige um cluster?** É contraintuitivo, mas `kubectl api-resources`,
> `kubectl explain` e até `kubectl run --dry-run=client` **não são operações puramente locais**.
> Antes de montar qualquer objeto, o `kubectl` precisa descobrir *que recursos existem* e *em qual
> grupo/versão eles vivem*: ele consulta o **discovery endpoint** (`/api` e `/apis`) e baixa o
> **esquema OpenAPI** do servidor. O `--dry-run=client` significa apenas "não persista este objeto",
> e não "não fale com o servidor". Sem cluster, todos falham assim:
>
> ```
> E0806 17:30:54.370165 memcache.go:265] "Unhandled Error" err="couldn't get current server API
> group list: Get \"https://127.0.0.1:62361/api?timeout=32s\": dial tcp 127.0.0.1:62361:
> connect: connection refused"
> The connection to the server 127.0.0.1:62361 was refused - did you specify the right host or port?
> ```
>
> Repare no endereço: `127.0.0.1` com uma porta alta e aleatória é a assinatura de um cluster local
> que **já existiu e não existe mais**. O `kubeconfig` ficou órfão. Isso é diferente do clássico
> `localhost:8080`, que indica ausência total de kubeconfig.

---

#### Trilha A — O mapa da arquitetura (sem cluster) · 15 min

**Passo A1 — Reconstruir o diagrama de memória.**

Em duplas, e **sem consultar o material**, reproduzam em papel (ou no arquivo `mapa-arquitetura.md`)
o diagrama da seção 1.6, rotulando cada componente com **uma frase** que responda "o que acontece se
este componente cair?". Só depois confiram com a tabela abaixo:

| Componente | Se cair... |
|---|---|
| kube-apiserver | nenhuma mudança nova é aceita; Pods existentes continuam rodando |
| etcd | perda de quórum congela toda a escrita; o cluster fica somente leitura ou indisponível |
| kube-scheduler | Pods novos ficam eternamente `Pending`; os já agendados seguem normais |
| controller-manager | não há self-healing nem rollout novo; nada é reconciliado |
| kubelet de um nó | o nó vai a `NotReady` e os Pods dele são despejados após ~5 min |
| kube-proxy de um nó | ClusterIPs param de funcionar naquele nó; Pod-a-Pod direto continua |

**Passo A2 — Prever o comportamento a partir de um manifesto.**

Leia o manifesto abaixo — sem executá-lo — e responda por escrito, ainda em duplas:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
spec:
  replicas: 3
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
    spec:
      containers:
        - name: store-api
          image: docker.io/seuusuario/store-api:1.0
          ports:
            - containerPort: 8000
```

1. Ao aplicar este arquivo, **quais componentes** são acionados, em que ordem, e qual deles decide
   em que nó cada Pod vai rodar?
2. O `kubectl apply` responde `deployment.apps/store-api created`. Nesse instante, quantos Pods
   estão **rodando**? Justifique com o conceito de assincronia entre `spec` e `status`.
3. Se um operador apagar um dos três Pods com `kubectl delete pod`, **quem** o recria e **por que**?
   E se apagar o nó inteiro?
4. Se a imagem `store-api:1.0` não existir no registro, em que ponto exato do fluxo a falha aparece,
   e qual componente a reporta?
5. O campo `selector.matchLabels` pede `app: store-api`, mas o `template` traz **dois** labels. Isso
   é um problema? O que aconteceria se o `template` não tivesse o label `app`?

> **Gabarito resumido:** (1) apiserver → etcd → deployment controller → replicaset controller →
> scheduler → kubelet do nó escolhido; quem decide o nó é o **scheduler**. (2) Zero: `created`
> significa apenas "persistido no etcd". (3) O ReplicaSet controller, porque o estado desejado
> continua sendo 3 e o observado passou a 2; se o nó cair, o mesmo controller recria os Pods em
> outro nó após o node controller marcá-lo `NotReady`. (4) Só na etapa final, no **kubelet**, que
> reporta `ErrImagePull`/`ImagePullBackOff` — o apiserver aceita a imagem sem verificar se existe.
> (5) Não é problema: o selector precisa ser um **subconjunto** dos labels do template. Sem o label
> `app`, a API rejeita o objeto (`selector does not match template labels`).

---

#### Trilha B — Explorando a API de verdade (exige cluster) · 25 min

**Passo B1 — Instalar o kubectl (Linux/WSL2), caso a Aula 0 não tenha sido feita.**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
kubectl version --client
```

Saída esperada:

```
kubectl: OK
Client Version: v1.34.1
Kustomize Version: v5.5.0
```

> **Nota:** no macOS use `brew install kubectl`; no Windows nativo, `winget install -e --id Kubernetes.kubectl` ou `choco install kubernetes-cli`. Recomendamos fortemente o WSL2 no Windows para acompanhar a disciplina.

**Passo B2 — Autocompletar, alias e kubectx.**

```bash
echo 'source <(kubectl completion bash)' >> ~/.bashrc
echo 'alias k=kubectl' >> ~/.bashrc
echo 'complete -o default -F __start_kubectl k' >> ~/.bashrc
source ~/.bashrc

# kubectx/kubens: troca rapida de contexto e de namespace
sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
sudo ln -s /opt/kubectx/kubectx /usr/local/bin/kubectx
sudo ln -s /opt/kubectx/kubens  /usr/local/bin/kubens
```

Se você usa zsh, troque `bash` por `zsh` e `~/.bashrc` por `~/.zshrc`.

**Passo B3 — Verificar para onde o kubectl está apontando (faça isto ANTES de qualquer coisa).**

Este é o passo que evita 90% da frustração da primeira semana:

```bash
kubectl config get-contexts     # lista todos os contextos conhecidos
kubectl config current-context  # qual está ativo
kubectl cluster-info            # o servidor responde?
kubectl get nodes               # prova final
```

Saída esperada de um ambiente saudável:

```
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
*         minikube   minikube   minikube   default

Kubernetes control plane is running at https://192.168.49.2:8443

NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   45s   v1.33.1
```

Se `kubectl cluster-info` devolver `connection refused`, pare aqui e resolva antes de continuar:

| Situação | Sintoma | O que fazer |
|---|---|---|
| Nenhum cluster subiu ainda | `The connection to the server localhost:8080 was refused` | `minikube start --driver=docker` |
| Contexto órfão de cluster apagado | `The connection to the server 127.0.0.1:<porta alta> was refused` | `minikube start` (recria) ou `kubectl config use-context <outro>`; para limpar de vez, `kubectl config delete-context <nome>` |
| Docker Desktop com Kubernetes desligado | contexto `docker-desktop` ativo e recusando | ligar em Settings → Kubernetes → Enable Kubernetes, ou trocar de contexto |
| Vários contextos e o errado ativo | comandos vão para o cluster errado | `kubectl config use-context minikube` ou `kubectx minikube` |

> **Atenção:** o binário do `kubectl` estar instalado **não** significa que ele tem cluster.
> `kubectl version --client` funciona offline; `kubectl version` (sem `--client`) não.

**Passo B4 — Gerar manifestos com `--dry-run=client`.**

A flag `--dry-run=client` faz o kubectl montar o objeto localmente e **não** persisti-lo no servidor. Combinada com `-o yaml`, vira um gerador de esqueletos excelente:

> **Cuidado com o nome da flag.** "client" refere-se a **onde o objeto é montado**, não a "sem
> rede". O kubectl ainda precisa do *discovery* do servidor para resolver o recurso, então este
> comando **falha sem cluster**. O contraste é com `--dry-run=server`, que envia o objeto ao
> apiserver para validação completa (inclusive webhooks de admissão) e descarta o resultado.

```bash
kubectl run store-api \
  --image=docker.io/seuusuario/store-api:1.0 \
  --port=8000 \
  --labels="app=store-api,tier=backend,env=dev" \
  --dry-run=client -o yaml > pod-dry-run.yaml

cat pod-dry-run.yaml
```

Saída esperada:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    app: store-api
    tier: backend
    env: dev
  name: store-api
spec:
  containers:
  - image: docker.io/seuusuario/store-api:1.0
    name: store-api
    ports:
    - containerPort: 8000
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Observe três coisas: (a) o campo `status: {}` já aparece, vazio, porque nada foi observado ainda; (b) o kubectl preencheu defaults que você não pediu (`dnsPolicy`, `restartPolicy`); (c) `creationTimestamp: null` — o servidor é quem preenche isso.

Gere também um Deployment e um Service:

```bash
kubectl create deployment store-api \
  --image=docker.io/seuusuario/store-api:1.0 \
  --replicas=3 --dry-run=client -o yaml > deploy-dry-run.yaml

kubectl create service clusterip store-api \
  --tcp=80:8000 --dry-run=client -o yaml > svc-dry-run.yaml
```

**Passo B5 — Ler o catálogo de recursos da API.**

```bash
kubectl api-resources --sort-by=name | head -25
```

Saída (parcial) esperada:

```
NAME                    SHORTNAMES   APIVERSION            NAMESPACED   KIND
configmaps              cm           v1                    true         ConfigMap
cronjobs                cj           batch/v1              true         CronJob
daemonsets              ds           apps/v1               true         DaemonSet
deployments             deploy       apps/v1               true         Deployment
endpointslices                       discovery.k8s.io/v1   true         EndpointSlice
horizontalpodautoscalers hpa         autoscaling/v2        true         HorizontalPodAutoscaler
ingresses               ing          networking.k8s.io/v1  true         Ingress
jobs                                 batch/v1              true         Job
namespaces              ns           v1                    false        Namespace
nodes                   no           v1                    false        Node
persistentvolumes       pv           v1                    false        PersistentVolume
pods                    po           v1                    true         Pod
replicasets             rs           apps/v1               true         ReplicaSet
secrets                              v1                    true         Secret
services                svc          v1                    true         Service
statefulsets            sts          apps/v1               true         StatefulSet
```

Repare nas colunas `SHORTNAMES` (`po`, `svc`, `deploy` economizam muita digitação) e `NAMESPACED` — esta última diz se o recurso vive dentro de um namespace ou no escopo do cluster inteiro. Note que a lista **varia de cluster para cluster**: um EKS com Argo CD instalado mostrará `applications.argoproj.io`, que o seu minikube não tem. Isso é a prova concreta de que o catálogo vem do servidor, e não do binário.

Filtragens úteis:

```bash
kubectl api-resources --namespaced=false          # recursos de cluster
kubectl api-resources --api-group=apps            # so o grupo apps
kubectl api-versions | sort                       # todas as GV disponiveis
```

**Passo B6 — Ler a documentação embutida com `kubectl explain`.**

O `kubectl explain` é a melhor documentação de campo que existe, porque vem do próprio esquema OpenAPI do cluster:

```bash
kubectl explain pod
kubectl explain pod.spec
kubectl explain pod.spec.containers
kubectl explain pod.spec.containers.livenessProbe --recursive | head -30
kubectl explain deployment.spec.strategy.rollingUpdate
```

Saída esperada (trecho de `kubectl explain pod.spec`):

```
KIND:       Pod
VERSION:    v1

FIELD: spec <PodSpec>

DESCRIPTION:
    Specification of the desired behavior of the pod.

FIELDS:
  activeDeadlineSeconds <integer>
  containers    <[]Container> -required-
    List of containers belonging to the pod. Containers cannot currently be
    added or removed. There must be at least one container in a Pod.
  ...
  restartPolicy <string>
    enum: Always, Never, OnFailure
  ...
```

Como o `explain` também vem do OpenAPI **do servidor**, ele documenta exatamente a versão do seu cluster — incluindo CRDs instalados. É por isso que ele é mais confiável que qualquer blog: `kubectl explain` nunca está desatualizado em relação ao cluster que você está operando.

---

### 1.11 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `The connection to the server localhost:8080 was refused` | Não há kubeconfig; kubectl caiu no default | `kubectl config view` (sai vazio); `ls ~/.kube/config` | Criar o cluster (Cap. 2) ou exportar `KUBECONFIG=/caminho/config` |
| `The connection to the server 127.0.0.1:<porta alta> was refused`, precedido de `couldn't get current server API group list` | Kubeconfig **órfão**: aponta para um cluster local que foi apagado ou está parado (minikube/kind deletado, Docker Desktop com Kubernetes desligado) | `kubectl config current-context` e `kubectl config get-contexts`; `minikube status`; `docker ps` | `minikube start` para recriar; ou `kubectl config use-context <contexto válido>`; para limpar, `kubectl config delete-context <nome>` |
| `kubectl run --dry-run=client` falha sem cluster | `--dry-run=client` significa "não persista", não "offline": o kubectl ainda faz *discovery* no servidor | o mesmo erro de conexão recusada acima | Subir o cluster; sem cluster, escrever o manifesto à mão (Trilha A da seção 1.10) |
| `error: unable to recognize "x.yaml": no matches for kind "Deployment" in version "extensions/v1beta1"` | API removida | `kubectl api-versions \| grep apps` | Trocar para `apps/v1` |
| `kubectl explain` ou `api-resources` retorna erro de conexão | ambos consultam o *discovery* e o esquema OpenAPI **do servidor** | `kubectl version` sem `--client` também falha, provando que é problema de conexão e não de instalação | Conectar a um cluster; alternativamente consultar kubernetes.io/docs/reference |
| `kubectl apply` responde `created` mas nada acontece | Confundir aceitação da API com execução | `kubectl get events -n store-dev --sort-by=.lastTimestamp` | Investigar scheduler/kubelet; lembre que o apply é assíncrono |
| `error validating data: unknown field "replica"` | Erro de digitação em campo do spec | `kubectl explain deployment.spec` | Corrigir para `replicas`; nunca use `--validate=false` para "resolver" |
| Objeto criado no namespace errado | Contexto sem namespace definido; caiu em `default` | `kubectl config get-contexts` | `kubectl config set-context --current --namespace=store-dev` ou `kubens store-dev` |
| Selector não encontra Pod nenhum | Labels do Pod diferentes do selector do Service | `kubectl get pods --show-labels` e `kubectl get endpoints <svc>` | Alinhar labels e selector (veremos em detalhe no Cap. 3) |

---

### 1.12 Resumo do capítulo

O Docker resolveu o empacotamento e a portabilidade da aplicação, mas deixou em aberto tudo o que acontece **depois** que a imagem existe: onde rodar, quantas cópias, o que fazer quando morre, como trocar de versão sem downtime e como um serviço encontra o outro. Orquestradores de containers atacam exatamente esse espaço. O Kubernetes se tornou o padrão de fato por três motivos combinados — uma API extensível que transformou o produto numa plataforma, a neutralidade garantida pela CNCF e a herança de duas décadas de operação em escala no Borg do Google. Seu princípio organizador é o modelo declarativo sustentado por loops de reconciliação independentes que só conversam através do `kube-apiserver`, e sua arquitetura separa control plane (apiserver, etcd, scheduler, controller-manager, cloud-controller-manager) de nós de trabalho (kubelet, kube-proxy, runtime via CRI).

Pontos-chave:

1. Orquestrador = escala horizontal + self-healing + service discovery + rolling update + bin packing + gestão declarativa de configuração.
2. `spec` é o que você deseja; `status` é o que o sistema observa; controladores existem para eliminar a diferença entre os dois, continuamente e de forma idempotente.
3. O `kube-apiserver` é o único ponto de contato: nenhum componente fala diretamente com outro, e é isso que torna o sistema substituível peça a peça.
4. `kubectl apply` é **assíncrono** — a resposta `created` significa apenas "persistido no etcd", não "rodando".
5. O dockershim saiu na 1.24, mas suas imagens continuam funcionando: o formato é OCI, e o runtime padrão hoje é o containerd.
6. Labels são para **selecionar**; annotations são para **anotar**. Essa distinção decide se um Service enxerga ou não os seus Pods.
7. O Kubernetes não constrói imagens, não é CI/CD, não é PaaS e não conserta arquitetura ruim — ele é o substrato sobre o qual essas coisas são construídas.

---

### 1.13 Exercícios (para casa)

#### Exercício 1.1 — Ambientar o kubectl e explorar a API (fixação guiada) · 30 min

> **Requer cluster no ar.** Conclua a Aula 0 (`minikube start`) antes de começar. Se o ambiente
> ainda não estiver pronto, faça primeiro o item (0) abaixo — ele vale nota por si só.

- **Objetivo:** garantir um cluster acessível, configurar produtividade de terminal e navegar pelo catálogo de recursos da API.
- **Enunciado:** Instale o `kubectl` na versão 1.33 ou superior no seu sistema (Linux/WSL2, macOS ou Windows). Configure autocompletar, o alias `k` (com autocompletar funcionando no alias) e instale o `kubectx`/`kubens`. Em seguida responda com **comandos, não com pesquisa no Google**: **(0)** cole a saída de `kubectl config get-contexts`, `kubectl config current-context` e `kubectl get nodes`, provando que existe um cluster acessível — se você recebeu `connection refused`, documente também o comando que usou para resolver; (a) quantos recursos de API existem no total e quantos são *cluster-scoped*; (b) qual o shortname de `horizontalpodautoscalers`, `persistentvolumeclaims` e `networkpolicies`; (c) quais são **todos** os valores aceitos pelo campo `restartPolicy` de um Pod; (d) qual é o valor padrão de `spec.strategy.type` num Deployment e quais são as opções; (e) qual campo de `pod.spec.containers.resources` limita memória. Registre cada comando e a saída correspondente.
- **Entregável:** arquivo `ex1-1.md` contendo, para cada item, o comando executado e a saída (pode ser recortada). Incluir a saída de `kubectl version --client` e de `k get ns` provando que o alias funciona.
- **Critério de aceite:** o item (0) mostra `kubectl get nodes` com pelo menos um nó `Ready`; as cinco respostas seguintes foram obtidas exclusivamente com `kubectl api-resources`, `kubectl explain` e `kubectl api-versions`; o alias `k` completa nomes de recursos com TAB; `kubens` lista os namespaces do cluster.
- <details><summary>Dica</summary>Se aparecer <code>The connection to the server 127.0.0.1:&lt;porta&gt; was refused</code>, o kubeconfig aponta para um cluster que não existe mais: rode <code>minikube start --driver=docker</code> e confira com <code>kubectl config current-context</code>. Depois, use <code>kubectl api-resources --namespaced=false</code> para o segundo número e <code>kubectl explain pod.spec.restartPolicy</code> — o campo <code>enum:</code> na saída já lista os valores válidos.</details>

#### Exercício 1.2 — Quadro comparativo e mini-relatório de arquitetura (aplicação) · 45 min

> **Requer cluster no ar** para a parte (A). A parte (B) é textual e pode ser feita sem cluster.

- **Objetivo:** consolidar o modelo mental da arquitetura produzindo artefatos a partir de comandos reais.
- **Enunciado:** Produza dois artefatos. **(A) Quadro comparativo:** para cada um dos objetos `Pod`, `Deployment`, `Service`, `Job` e `ConfigMap`, execute `kubectl explain <objeto>` e `kubectl create ... --dry-run=client -o yaml` (ou `kubectl run` para Pod) e preencha uma tabela com as colunas: Kind | apiVersion | Namespaced? | Shortname | Campo obrigatório mais importante do `spec` | Uma frase sobre para que serve. **(B) Mini-relatório de arquitetura:** escreva de 400 a 600 palavras descrevendo, do seu jeito, o que acontece entre o instante em que você digita `kubectl apply -f deployment.yaml` e o instante em que a primeira requisição HTTP chega ao container da `store-api`. O relatório deve mencionar, em ordem, **no mínimo**: kubeconfig, autenticação, RBAC, admission controller, etcd, deployment-controller, replicaset-controller, kube-scheduler, kubelet, CRI/containerd, CNI, readiness probe, EndpointSlice e kube-proxy. Para cada componente, escreva também o que aconteceria se ele estivesse indisponível naquele instante.
- **Entregável:** `ex1-2-quadro.md` e `relatorio-arquitetura.md`. Os manifestos gerados com `--dry-run` devem ir junto, na pasta `ex1-2/`.
- **Critério de aceite:** a tabela tem 5 linhas corretas (conferíveis com `kubectl api-resources`); o relatório cita os 14 elementos exigidos, na ordem correta, e cada um vem acompanhado da consequência da sua indisponibilidade.
- <details><summary>Dica</summary>Gere os esqueletos com <code>kubectl create job x --image=busybox --dry-run=client -o yaml</code> e <code>kubectl create configmap x --from-literal=APP_ENV=dev --dry-run=client -o yaml</code>; a coluna "campo obrigatório" sai de <code>kubectl explain &lt;objeto&gt;.spec</code> procurando por <code>-required-</code>.</details>

#### Exercício 1.3 — Depuração conceitual a partir de eventos (análise) · 30 min

> **Não requer cluster.** Pode ser feito com o ambiente ainda quebrado.

- **Objetivo:** interpretar sintomas do cluster e localizar o componente responsável, sem acesso ao cluster.
- **Enunciado:** Um colega tentou subir a `store-api` num cluster de 3 nós e enviou os seguintes recortes. Para **cada um dos quatro casos**, responda: (1) qual componente da arquitetura está envolvido; (2) qual é a causa provável; (3) qual comando `kubectl` você rodaria em seguida para confirmar; (4) qual seria a correção.

  **Caso A**
  ```
  $ kubectl get pods -n store-dev
  NAME                         READY   STATUS    RESTARTS   AGE
  store-api-7d9f8c6b45-2xk9p   0/1     Pending   0          14m

  $ kubectl describe pod store-api-7d9f8c6b45-2xk9p -n store-dev
  Events:
    Type     Reason            Age   From               Message
    ----     ------            ----  ----               -------
    Warning  FailedScheduling  14m   default-scheduler  0/3 nodes are available:
             3 Insufficient cpu. preemption: 0/3 nodes are available: 3 No preemption
             victims found for incoming pod.
  ```

  **Caso B**
  ```
  $ kubectl get nodes
  NAME       STATUS     ROLES           AGE   VERSION
  node-01    Ready      control-plane   42d   v1.34.1
  node-02    NotReady   <none>          42d   v1.34.1
  node-03    Ready      <none>          42d   v1.34.1

  $ kubectl describe node node-02 | grep -A3 Conditions
  Conditions:
    Type             Status    Reason                 Message
    Ready            Unknown   NodeStatusUnknown      Kubelet stopped posting node status.
  ```

  **Caso C**
  ```
  $ kubectl apply -f pod.yaml
  Error from server (Forbidden): error when creating "pod.yaml": pods "store-api"
  is forbidden: violates PodSecurity "restricted:latest": allowPrivilegeEscalation
  != false (container "store-api" must set securityContext.allowPrivilegeEscalation=false)
  ```

  **Caso D**
  ```
  $ kubectl get svc,endpoints -n store-dev
  NAME                TYPE        CLUSTER-IP      PORT(S)   AGE
  service/store-api   ClusterIP   10.96.140.221   80/TCP    6m

  NAME                  ENDPOINTS   AGE
  endpoints/store-api   <none>      6m

  $ kubectl get pods -n store-dev --show-labels
  NAME                         READY   STATUS    LABELS
  store-api-7d9f8c6b45-2xk9p   1/1     Running   app=storeapi,tier=backend,env=dev
  ```
- **Entregável:** `ex1-3.md`, com as quatro respostas estruturadas em (1)(2)(3)(4).
- **Critério de aceite:** Caso A aponta o `kube-scheduler` e a fase de *filtering* por recursos insuficientes; Caso B aponta o `kubelet` do node-02 e o `node-controller` que o marcou; Caso C aponta um *validating admission controller* (Pod Security Admission); Caso D aponta o `endpointslice-controller` e a divergência de label (`storeapi` vs `store-api`). Cada resposta traz um comando de confirmação plausível.
- <details><summary>Dica</summary>Reveja a seção 1.6.3: cada mensagem de erro carrega no campo <code>From</code> (ou no texto) o nome do componente que a emitiu. No Caso D, compare caractere a caractere o label do Pod com o selector que um Service <code>store-api</code> normalmente teria.</details>

#### Exercício 1.4 — Desafio: arquitetura de deploy da FIAP Store (desafio integrador) · 60 min

> **Não requer cluster.** É um exercício de projeto e documentação.

- **Objetivo:** projetar e justificar uma arquitetura Kubernetes completa para a aplicação do semestre, incluindo a decisão container versus serverless.
- **Enunciado:** Você é a pessoa responsável por levar a FIAP Store para produção na AWS. Produza um documento `arquitetura-fiap-store.md` contendo: **(A)** um diagrama **Mermaid** (bloco ```` ```mermaid ```` dentro do próprio `.md`) representando o cluster EKS com os namespaces `store-dev` e `store-prod`, os quatro componentes da aplicação (`store-api`, `store-worker`, `store-db`, `store-front`), a entrada de tráfego (usuário → Ingress/ALB → Service → Pods) e os componentes do control plane gerenciados pela AWS. Use `subgraph` para separar control plane, worker nodes e serviços gerenciados da AWS. **(B)** Uma tabela com uma linha por componente da aplicação e as colunas: Componente | Objeto Kubernetes que você usaria (Deployment, StatefulSet, DaemonSet, Job/CronJob) | Tipo de Service | Justificativa em uma frase | Por que **não** os outros objetos. **(C)** Uma seção "Container ou Serverless?" com 200 a 300 palavras decidindo, para **cada** um dos quatro componentes, se ele deveria rodar como Pod no EKS, em nós Fargate, ou como função Lambda — justificando com pelo menos três critérios entre: padrão de tráfego, estado, tempo de execução, cold start e modelo de custo. **(D)** Uma seção "Custos e destruição" listando os recursos AWS que essa arquitetura criaria e que geram cobrança (control plane EKS por hora, instâncias EC2 ou vCPU-Fargate, ALB, EBS, ECR, NAT Gateway, tráfego de saída) e o procedimento para destruir tudo ao final de um laboratório.
- **Entregável:** `arquitetura-fiap-store.md` com as quatro seções. O diagrama Mermaid deve renderizar sem erro de sintaxe (valide em https://mermaid.live).
- **Critério de aceite:** o diagrama renderiza e contém ao menos 3 `subgraph`, os 4 componentes da aplicação e o caminho completo do tráfego externo até o Pod; a tabela cobre os 4 componentes com justificativa **e** contra-justificativa; a seção (C) toma uma decisão explícita por componente com no mínimo três critérios; a seção (D) lista pelo menos cinco fontes de custo e o comando/procedimento de destruição.
- <details><summary>Dica</summary>No Mermaid, use <code>graph TD</code> e blocos <code>subgraph "Control Plane (gerenciado AWS)" ... end</code>. Para (C), lembre-se: o <code>store-db</code> tem estado e conexões longas (péssimo para Lambda); o <code>store-worker</code> processa em batch e fica ocioso a maior parte do tempo (excelente candidato a Lambda ou a KEDA com scale-to-zero).</details>

---

### 1.14 Leitura recomendada

1. **[DOC]** Kubernetes — *Concepts: Overview / Cluster Architecture*. Disponível em: https://kubernetes.io/docs/concepts/architecture/ — leitura obrigatória das páginas "Nodes", "Control Plane Components" e "Controllers".
2. **[ARTIGO]** BURNS, B.; GRANT, B.; OPPENHEIMER, D.; BREWER, E.; WILKES, J. *Borg, Omega, and Kubernetes: Lessons learned from three container-management systems over a decade*. ACM Queue, v. 14, 2016. Disponível em: https://queue.acm.org/detail.cfm?id=2898444
3. **[LIVRO]** BURNS, B.; BEDA, J.; HIGHTOWER, K.; EVENSON, L. *Kubernetes: Up and Running*. 3. ed. O'Reilly, 2022 — capítulos 1 e 2.
4. **[LIVRO]** LUKSA, M. *Kubernetes in Action*. 2. ed. Manning, 2023 — capítulo 1 e a explicação do loop de reconciliação no capítulo 4.
5. **[DOC]** AWS. *Amazon EKS Best Practices Guide*. Disponível em: https://docs.aws.amazon.com/eks/latest/best-practices/ — seções "Cluster Architecture" e "Cost Optimization".
6. **[VÍDEO]** CNCF. *Kubernetes: The Documentary (Part 1 & 2)*. Disponível no canal da Honeypot no YouTube — contexto histórico da guerra dos orquestradores.

---

## Capítulo 2 — Instalação e ambientação em Kubernetes (Minikube)

> **Aula:** 2 horas · **Pré-requisitos:** Capítulo 1; Docker 27+ instalado e funcionando; `kubectl` 1.33+; 4 vCPU e 8 GB de RAM disponíveis na máquina · **Artefatos:** cluster `fiap-store` no minikube, `Dockerfile`, `main.py`, `requirements.txt`, `setup-lab.sh`

---

### 2.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Comparar** as ferramentas de cluster local (minikube, kind, k3d, Docker Desktop, Rancher Desktop) e **selecionar** a adequada a um cenário dado.
2. **Instalar** e **configurar** um cluster minikube multi-nó com drivers, recursos e versão do Kubernetes definidos explicitamente.
3. **Interpretar** e **manipular** o kubeconfig (clusters, contexts, users), alternando contexto e namespace pela linha de comando.
4. **Construir** a imagem da `store-api` com Dockerfile multi-stage e usuário não-root, carregá-la no cluster sem registry externo e expô-la via NodePort.
5. **Aplicar** o conjunto de comandos de inspeção (`get`, `describe`, `logs`, `exec`, `port-forward`, `top`, `events`) para diagnosticar o estado de uma carga.
6. **Automatizar** o provisionamento do laboratório em um script idempotente, validando o ambiente ao final.

---

### 2.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Panorama de clusters locais e critérios de escolha |
| 2 | 15 min | Instalação do minikube; drivers e requisitos por sistema operacional |
| 3 | 15 min | `minikube start` com flags; anatomia do kubeconfig e `kubectl config` |
| 4 | 15 min | Addons, `minikube ssh`, `image load`, `tunnel`, `service` |
| 5 | 10 min | **Intervalo** |
| 6 | 20 min | Primeiro deploy imperativo e comandos de inspeção |
| 7 | 25 min | Laboratório guiado: build da `store-api`, `image load`, NodePort e `/health` |
| 8 | 5 min | Ponte com Serverless: tempo até o primeiro request; Knative/KEDA como prévia |
| 9 | 5 min | Erros comuns, limpeza e apresentação dos exercícios |

---

### 2.3 Por que um cluster local?

No Capítulo 1 falamos muito e rodamos pouco. Isso muda agora. Mas antes de escolher a ferramenta, vale entender **por que não começar direto no EKS**.

Um cluster EKS de laboratório custa dinheiro real: o control plane é cobrado por hora **por cluster**, independentemente de uso, e cada nó é uma instância EC2 rodando 24 horas por dia se você esquecer de destruí-la. Além do custo, há a latência de aprendizado: criar um cluster EKS leva de 10 a 15 minutos; destruir e recriar para testar uma alteração de rede é insuportável durante o aprendizado. Um cluster local sobe em 40 segundos, é gratuito e pode ser destruído sem culpa.

O ponto essencial é que **a API é a mesma**. Um manifesto que funciona no minikube funciona no EKS. As diferenças aparecem exatamente em três lugares, e vale saber quais são desde já:

- **Service do tipo LoadBalancer.** No EKS, cria um NLB/ALB real com IP público. No minikube, fica eternamente `<pending>` a menos que você rode `minikube tunnel`.
- **Storage.** No EKS, uma PVC dispara o provisionamento de um volume EBS. No minikube, o provisionador padrão cria um diretório no disco do nó (`hostPath`).
- **Identidade e integrações de nuvem.** IRSA, Secrets Manager, ECR privado, ALB Ingress Controller — nada disso existe localmente.

Fora esses três, o comportamento é equivalente. É por isso que passamos os primeiros capítulos no minikube.

---

### 2.4 Panorama de clusters locais

| Ferramenta | Como cria os nós | Tempo de start | Multi-nó | Pontos fortes | Limitações |
|---|---|---|---|---|---|
| **minikube** | VM ou container (driver plugável) | 40–90 s | Sim (`--nodes N`) | Sistema de **addons** rico (ingress, metrics-server, dashboard, registry); múltiplos perfis; `minikube ssh`; documentação enorme | Consome mais recursos que kind/k3d; um pouco mais lento |
| **kind** | Nós como containers Docker | 20–40 s | Sim (config YAML) | Muito rápido; padrão de facto para **CI**; usado nos testes de conformidade do próprio Kubernetes | Sem addons prontos; ingress e storage exigem configuração manual |
| **k3d** | k3s dentro de containers Docker | 10–20 s | Sim | O mais leve de todos; ideal para máquinas modestas; registry local integrado | Roda **k3s**, não o Kubernetes upstream puro (Traefik e servicelb embutidos, alguns componentes substituídos) |
| **Docker Desktop** | VM interna do Docker Desktop | Já vem junto | Não | Um clique para habilitar; zero configuração | Nó único; versão do Kubernetes amarrada ao Docker Desktop; licença paga para empresas grandes |
| **Rancher Desktop** | k3s em VM (lima/WSL) | 60 s | Não | Open source; troca de versão do Kubernetes pela interface; inclui `nerdctl` | Nó único; menos flexível que minikube para cenários avançados |

**Nossa escolha na disciplina é o minikube**, por três razões: os addons cobrem quase tudo que precisamos (ingress, metrics-server para HPA, dashboard, registry) sem instalação manual; o suporte a `--nodes` permite exercitar afinidade, taints e DaemonSets de verdade; e o conceito de **perfis** permite manter clusters isolados por assunto sem conflito.

> **Na prática:** em ambientes de CI (GitHub Actions, GitLab CI), a escolha correta quase sempre é **kind** — ele é mais rápido, roda dentro de um container e não precisa de virtualização aninhada. Minikube para a bancada de estudo, kind para o pipeline.

---

### 2.5 Requisitos e drivers do minikube

O minikube precisa de um **driver**: o mecanismo que cria a máquina onde o cluster vai rodar.

| Driver | Sistemas | Isolamento | Observações |
|---|---|---|---|
| `docker` | Linux, macOS, Windows | Container | **Padrão e recomendado.** Nó vira um container `gcr.io/k8s-minikube/kicbase`. Rápido e leve |
| `podman` | Linux, macOS | Container | Alternativa rootless ao Docker; suporte um pouco menos maduro |
| `kvm2` | Linux | VM completa | Melhor isolamento; exige `libvirt` e virtualização habilitada na BIOS |
| `hyperkit` | macOS Intel | VM | Legado; em Apple Silicon use `docker` ou `qemu` |
| `qemu` | macOS (Apple Silicon), Linux | VM | Opção para M1/M2/M3 quando se quer VM real |
| `virtualbox` | Todos | VM | Portável, mas lento; conflita com Hyper-V no Windows |
| `hyperv` | Windows Pro/Enterprise | VM | Requer Hyper-V habilitado; incompatível com VirtualBox simultaneamente |
| `none` | Linux | Nenhum | Instala direto no host, como root. **Não use em máquina pessoal** |

**Requisitos mínimos:** 2 vCPU, 2 GB de RAM livres e 20 GB de disco. Para acompanhar esta disciplina confortavelmente (cluster de 2 nós + addons + Prometheus mais adiante), recomendamos **4 vCPU e 8 GB de RAM** na máquina host.

#### 2.5.1 Instalação — Linux (e WSL2)

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64
minikube version
```

```
minikube version: v1.36.0
commit: 8f2c9a1e0b4d3f7a2c5e8b1d9f0a3c6e4b7d2a51
```

No **WSL2**, dois pontos merecem atenção: habilite a integração do Docker Desktop com a sua distro (Settings → Resources → WSL Integration) **ou** instale o Docker Engine diretamente dentro do WSL; e confirme que seu usuário está no grupo `docker`:

```bash
sudo usermod -aG docker $USER && newgrp docker
docker run --rm hello-world
```

#### 2.5.2 Instalação — macOS

```bash
brew install minikube
# Em Apple Silicon, o driver docker e a escolha natural:
minikube start --driver=docker
```

#### 2.5.3 Instalação — Windows

Duas rotas. A **recomendada** é usar WSL2 e seguir as instruções de Linux dentro da distro — assim seus caminhos, scripts e permissões batem com os do resto da turma. A alternativa nativa:

```powershell
winget install -e --id Kubernetes.minikube
# ou: choco install minikube
minikube start --driver=docker
```

> **Atenção:** no Windows, evite alternar entre drivers (`hyperv` e `docker`, por exemplo) no mesmo perfil — o minikube guarda estado por perfil e a troca de driver costuma exigir `minikube delete` antes.

---

### 2.6 Subindo o cluster: `minikube start` e suas flags

O comando mínimo funciona:

```bash
minikube start
```

Mas em um contexto didático queremos **explicitar tudo**, para que o ambiente seja reprodutível:

```bash
minikube start \
  --profile=fiap-store \
  --driver=docker \
  --kubernetes-version=v1.34.1 \
  --cpus=2 \
  --memory=4096 \
  --nodes=2 \
  --addons=metrics-server,ingress
```

Saída esperada (abreviada):

```
* [fiap-store] minikube v1.36.0 on Ubuntu 24.04
* Using the docker driver based on user configuration
* Starting "fiap-store" primary control-plane node in cluster "fiap-store"
* Pulling base image v0.0.46 ...
* Creating docker container (CPUs=2, Memory=4096MB) ...
* Preparing Kubernetes v1.34.1 on Docker 27.3.1 ...
  - Generating certificates and keys ...
  - Booting up control plane ...
  - Configuring RBAC rules ...
* Configuring CNI (Container Networking Interface) ...
* Verifying Kubernetes components...
  - Using image registry.k8s.io/metrics-server/metrics-server:v0.7.2
* Enabled addons: storage-provisioner, default-storageclass, metrics-server, ingress
* Starting "fiap-store-m02" worker node in cluster "fiap-store"
* Done! kubectl is now configured to use "fiap-store" cluster and "default" namespace by default
```

| Flag | Para que serve | Nota |
|---|---|---|
| `--profile` / `-p` | Nome do cluster; permite vários clusters independentes | Todo comando subsequente aceita `-p`; ou fixe com `minikube profile fiap-store` |
| `--driver` | Mecanismo de criação do nó | Explicitar evita surpresas entre máquinas da turma |
| `--kubernetes-version` | Versão exata do control plane | Fundamental para reprodutibilidade; permite testar upgrades |
| `--cpus` / `--memory` | Recursos por nó | `--memory=max` e `--cpus=max` usam tudo disponível |
| `--nodes` | Quantidade de nós | 1 control-plane + (N-1) workers |
| `--addons` | Habilita addons já no start | Equivale a rodar `minikube addons enable` depois |
| `--container-runtime` | `containerd`, `cri-o` ou `docker` | Use `containerd` para espelhar o EKS moderno |
| `--extra-config` | Passa flags aos componentes do control plane | Ex.: `--extra-config=kubelet.max-pods=200` |

Verificação imediata:

```bash
kubectl get nodes -o wide
```

```
NAME             STATUS   ROLES           AGE   VERSION   INTERNAL-IP    OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
fiap-store       Ready    control-plane   93s   v1.34.1   192.168.49.2   Ubuntu 22.04.5 LTS   6.8.0-45-generic  docker://27.3.1
fiap-store-m02   Ready    <none>          61s   v1.34.1   192.168.49.3   Ubuntu 22.04.5 LTS   6.8.0-45-generic  docker://27.3.1
```

```bash
kubectl get pods -A
```

```
NAMESPACE       NAME                                     READY   STATUS    RESTARTS   AGE
ingress-nginx   ingress-nginx-controller-7d4b8c9f6-hn2vq 1/1     Running   0          80s
kube-system     coredns-668d6bf9bc-4z8lm                 1/1     Running   0          93s
kube-system     etcd-fiap-store                          1/1     Running   0          99s
kube-system     kindnet-9xk4p                            1/1     Running   0          93s
kube-system     kube-apiserver-fiap-store                1/1     Running   0          99s
kube-system     kube-controller-manager-fiap-store       1/1     Running   0          99s
kube-system     kube-proxy-t6rwz                         1/1     Running   0          93s
kube-system     kube-scheduler-fiap-store                1/1     Running   0          99s
kube-system     metrics-server-7fbb699795-cq8dp          1/1     Running   0          80s
kube-system     storage-provisioner                      1/1     Running   0          97s
```

Reconhece a lista? São exatamente os componentes do diagrama do Capítulo 1. No minikube eles rodam como **static pods** — Pods criados diretamente pelo kubelet a partir de arquivos em `/etc/kubernetes/manifests`, sem passar pelo apiserver (afinal, o apiserver é um deles).

Outros comandos de ciclo de vida:

```bash
minikube status -p fiap-store     # estado de cada componente
minikube pause -p fiap-store      # congela o control plane, libera CPU
minikube unpause -p fiap-store
minikube stop -p fiap-store       # desliga preservando o disco
minikube delete -p fiap-store     # destroi tudo
minikube profile list             # lista todos os perfis
```

---

### 2.7 Anatomia do kubeconfig

O `minikube start` termina dizendo "kubectl is now configured". O que ele fez, exatamente, foi **editar o arquivo `~/.kube/config`**. Entender esse arquivo é o que separa quem usa o kubectl de quem sofre com ele.

```bash
kubectl config view --minify
```

```yaml
apiVersion: v1
kind: Config
current-context: fiap-store
clusters:
- name: fiap-store
  cluster:
    certificate-authority: /home/aluno/.minikube/ca.crt
    server: https://192.168.49.2:8443
contexts:
- name: fiap-store
  context:
    cluster: fiap-store
    user: fiap-store
    namespace: default
users:
- name: fiap-store
  user:
    client-certificate: /home/aluno/.minikube/profiles/fiap-store/client.crt
    client-key: /home/aluno/.minikube/profiles/fiap-store/client.key
```

O kubeconfig tem exatamente três listas e um ponteiro:

- **`clusters`** — *onde* está o cluster: URL do apiserver e a CA que valida seu certificado TLS.
- **`users`** — *quem* você é: certificado cliente, token, ou um `exec` que chama um binário externo (é assim que o EKS funciona: `aws eks get-token`).
- **`contexts`** — a **tripla** (cluster, user, namespace). Um contexto amarra um "onde" a um "quem" e opcionalmente fixa um namespace padrão.
- **`current-context`** — qual das triplas está ativa agora.

Comandos essenciais:

```bash
kubectl config get-contexts
kubectl config use-context fiap-store
kubectl config current-context
kubectl config set-context --current --namespace=store-dev
kubectl config view --raw > backup-kubeconfig.yaml
```

```
CURRENT   NAME         CLUSTER      AUTHINFO     NAMESPACE
*         fiap-store   fiap-store   fiap-store   store-dev
          fiap-net     fiap-net     fiap-net     default
          eks-prod     eks-prod     eks-prod     store-prod
```

> **Atenção:** `kubectl config set-context --current --namespace=X` é o comando mais subestimado do Kubernetes. Sem ele, você digita `-n store-dev` centenas de vezes por dia e, na vez que esquecer, cria um objeto no namespace `default`. Use `kubens store-dev` (do kubectx) para o mesmo efeito com menos digitação.

Você pode compor múltiplos kubeconfigs sem editar nada, via variável de ambiente:

```bash
export KUBECONFIG=~/.kube/config:~/.kube/eks-prod.yaml
kubectl config get-contexts
```

> **Nota — segurança.** O kubeconfig é uma credencial. Um arquivo de EKS com um contexto de produção dá a quem o possui exatamente os poderes que o RBAC daquele usuário concede. Nunca versione kubeconfig no Git, mantenha permissão `600` e prefira contextos que usem `exec` (token de curta duração) a certificados de longa duração.

---

### 2.8 Addons do minikube

Addons são componentes pré-empacotados que o minikube instala e mantém para você.

```bash
minikube addons list -p fiap-store
```

```
|-----------------------------|------------|--------------|
|         ADDON NAME          |  PROFILE   |    STATUS    |
|-----------------------------|------------|--------------|
| dashboard                   | fiap-store | disabled     |
| default-storageclass        | fiap-store | enabled       |
| ingress                     | fiap-store | enabled       |
| metrics-server              | fiap-store | enabled       |
| registry                    | fiap-store | disabled     |
| storage-provisioner         | fiap-store | enabled       |
|-----------------------------|------------|--------------|
```

| Addon | O que faz | Quando usaremos |
|---|---|---|
| `metrics-server` | Coleta CPU/memória dos Pods; habilita `kubectl top` | **Obrigatório** para HPA |
| `ingress` | Instala o ingress-nginx controller | Capítulo de Ingress e canary |
| `dashboard` | Interface web de administração | Exploração visual; abre com `minikube dashboard` |
| `registry` | Registry Docker interno ao cluster | Alternativa ao `image load` |
| `storage-provisioner` | Provisiona PVs dinamicamente com hostPath | PVC/StatefulSet |
| `default-storageclass` | Marca a StorageClass `standard` como padrão | PVC sem `storageClassName` |

```bash
minikube addons enable metrics-server -p fiap-store
minikube addons enable dashboard      -p fiap-store
minikube dashboard -p fiap-store        # abre o navegador (bloqueia o terminal)
minikube dashboard --url -p fiap-store  # so imprime a URL
```

O `metrics-server` leva de 30 a 60 segundos para ter dados:

```bash
kubectl top nodes
```

```
NAME             CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
fiap-store       241m         12%    1180Mi          29%
fiap-store-m02   88m          4%     642Mi           16%
```

---

### 2.9 Ferramentas específicas do minikube

#### 2.9.1 `minikube ssh` — entrar no nó

```bash
minikube ssh -p fiap-store
# dentro do no:
docker ps | head -5
sudo crictl ps
ls /etc/kubernetes/manifests/
exit
```

Para entrar num nó específico de um cluster multi-nó: `minikube ssh -p fiap-store -n fiap-store-m02`.

#### 2.9.2 `minikube image load` — imagens locais sem registry

Este é **o comando mais importante deste capítulo**. Quando você faz `docker build -t store-api:1.0 .`, a imagem fica no daemon Docker da **sua máquina**. O cluster minikube roda dentro de outro container/VM, com seu **próprio** armazenamento de imagens — ele não enxerga a sua. Sem resolver isso, todo Pod que você criar vai falhar com `ErrImagePull`.

Três caminhos:

```bash
# (1) Carregar a imagem local para dentro do cluster
docker build -t store-api:1.0 .
minikube image load store-api:1.0 -p fiap-store
minikube image ls -p fiap-store | grep store-api

# (2) Buildar direto dentro do minikube
minikube image build -t store-api:1.0 . -p fiap-store

# (3) Apontar seu CLI Docker para o daemon do minikube
eval $(minikube docker-env -p fiap-store)
docker build -t store-api:1.0 .     # a imagem ja nasce dentro do cluster
eval $(minikube docker-env -u)      # desfaz
```

> **Atenção — a armadilha do `imagePullPolicy`.** Se a tag da sua imagem for `:latest`, o padrão do Kubernetes passa a ser `imagePullPolicy: Always`, e o kubelet tentará buscar no Docker Hub, ignorando a imagem local — resultado: `ErrImagePull`. Duas soluções: (a) **use tags versionadas** (`store-api:1.0`), quando o padrão vira `IfNotPresent`; (b) declare explicitamente `imagePullPolicy: IfNotPresent` (ou `Never`) no manifesto. Adote sempre a opção (a): tags versionadas são boa prática independentemente disso.

#### 2.9.3 `minikube service` e `minikube tunnel`

```bash
minikube service store-api -n store-dev -p fiap-store --url
```

```
http://192.168.49.2:30080
```

Esse comando descobre o IP do nó e a `nodePort`, monta a URL e (sem `--url`) abre no navegador. Em drivers de container no macOS e Windows, ele cria um túnel local e devolve `http://127.0.0.1:<porta-aleatoria>` — por isso **sempre use a URL que ele imprime**, nunca a que você deduziu.

Já `minikube tunnel` resolve outro problema: um `Service` do tipo `LoadBalancer` fica com `EXTERNAL-IP: <pending>` para sempre no minikube, porque não existe cloud-controller-manager para provisionar um balanceador real. O `tunnel` simula esse papel, criando rotas na sua máquina:

```bash
sudo minikube tunnel -p fiap-store    # deixe rodando em outro terminal
```

```bash
kubectl get svc -n store-dev
```

```
NAME        TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)        AGE
store-api   LoadBalancer   10.104.22.187   10.104.22.187   80:31842/TCP   2m
```

---

### 2.10 Primeiro deploy: o caminho imperativo

Antes de escrever YAML (Capítulo 3), vamos usar os comandos imperativos — eles são ótimos para experimentar e para gerar esqueletos.

```bash
kubectl create namespace store-dev
kubectl config set-context --current --namespace=store-dev

kubectl create deployment web --image=nginx:1.27 --replicas=2
kubectl expose deployment web --port=80 --target-port=80 --type=NodePort
kubectl scale deployment web --replicas=4
kubectl set image deployment/web nginx=nginx:1.27.3
kubectl rollout status deployment/web
kubectl rollout undo deployment/web
```

```
namespace/store-dev created
Context "fiap-store" modified.
deployment.apps/web created
service/web exposed
deployment.apps/web scaled
deployment.apps/web image updated
Waiting for deployment "web" rollout to finish: 3 of 4 updated replicas are available...
deployment "web" successfully rolled out
deployment.apps/web rolled back
```

O ponto pedagógico: repare que `kubectl scale` alterou o cluster mas **não alterou nenhum arquivo**. Se amanhã alguém rodar `kubectl apply -f web.yaml` com `replicas: 2`, sua alteração some. Esse é o comportamento correto do modelo declarativo — e a razão pela qual, a partir do próximo capítulo, o estado vive em arquivos.

#### 2.10.1 Caixa de ferramentas de inspeção

```bash
kubectl get pods -o wide                    # inclui NODE e IP
kubectl get pods -w                         # modo watch
kubectl get pod web-6d8f9c5b7-x2k4p -o yaml # objeto completo, com status
kubectl get deploy,rs,pod,svc               # varios tipos de uma vez

kubectl describe pod web-6d8f9c5b7-x2k4p    # SEMPRE olhe a secao Events

kubectl logs web-6d8f9c5b7-x2k4p            # logs do container
kubectl logs -f -l app=web --tail=50        # segue os logs de TODOS os pods do label
kubectl logs web-6d8f9c5b7-x2k4p --previous # logs do container que MORREU (crucial em CrashLoop)

kubectl exec -it web-6d8f9c5b7-x2k4p -- sh
kubectl port-forward pod/web-6d8f9c5b7-x2k4p 8080:80
kubectl port-forward svc/web 8080:80

kubectl top pods
kubectl get events --sort-by=.lastTimestamp
kubectl events --for pod/web-6d8f9c5b7-x2k4p   # sintaxe moderna (1.30+)
```

> **Na prática:** o fluxo padrão de diagnóstico é sempre o mesmo — `get` para ver o sintoma, `describe` para ler os **Events** (que dizem por que o Pod não sai do lugar), `logs` para ver o que a aplicação disse antes de morrer, e `exec`/`port-forward` para investigar de dentro. Se você internalizar esta ordem, resolve 80% dos problemas sem ajuda.

---

### 2.11 Laboratório guiado (feito em sala)

**Objetivo:** subir o cluster, construir a imagem da `store-api`, carregá-la no minikube, expô-la via NodePort e acessar o endpoint `/health`.

**Passo 1 — Criar a estrutura do projeto.**

```bash
mkdir -p ~/fiap-store/store-api && cd ~/fiap-store/store-api
```

`main.py`:

```python
# main.py
from fastapi import FastAPI, Response
import os, socket

app = FastAPI(title="FIAP Store API")
PRONTO = True

@app.get("/health")
def health():
    return {"status": "ok", "host": socket.gethostname()}

@app.get("/ready")
def ready(response: Response):
    if not PRONTO:
        response.status_code = 503
        return {"status": "starting"}
    return {"status": "ready"}

@app.get("/produtos")
def produtos():
    return [{"id": 1, "nome": "Teclado", "preco": 199.9}]

@app.get("/info")
def info():
    return {"env": os.getenv("APP_ENV", "dev"),
            "db_host": os.getenv("DB_HOST", "nao-configurado"),
            "pod": socket.gethostname()}
```

`requirements.txt`:

```
fastapi==0.115.6
uvicorn[standard]==0.34.0
```

**Passo 2 — Dockerfile multi-stage com usuário não-root.**

```dockerfile
# ---------- Stage 1: build das dependencias ----------
FROM python:3.12-slim AS builder

WORKDIR /build
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip \
 && pip install --no-cache-dir -r requirements.txt

# ---------- Stage 2: imagem final ----------
FROM python:3.12-slim AS runtime

# usuario nao-root com UID fixo (importante para Pod Security Admission)
RUN groupadd --gid 10001 appuser \
 && useradd --uid 10001 --gid 10001 --create-home --shell /usr/sbin/nologin appuser

COPY --from=builder /opt/venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_ENV=dev

WORKDIR /app
COPY --chown=10001:10001 main.py .

USER 10001
EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Por que multi-stage? A imagem final não carrega compiladores, cache do pip nem o `requirements.txt` — só o virtualenv pronto e o código. Por que não-root? Porque um comprometimento do processo dentro do container não deve começar já com UID 0, e porque o **Pod Security Admission** no perfil `restricted` (que veremos no capítulo de segurança) **rejeita** Pods que rodam como root.

**Passo 3 — Buildar e testar localmente.**

```bash
docker build -t store-api:1.0 .
docker run --rm -d -p 8000:8000 --name teste store-api:1.0
curl -s localhost:8000/health
docker stop teste
```

```
{"status":"ok","host":"3f1a9b2c4d5e"}
```

**Passo 4 — Subir o cluster e carregar a imagem.**

```bash
minikube start -p fiap-store --driver=docker --kubernetes-version=v1.34.1 \
  --cpus=2 --memory=4096 --nodes=2 --addons=metrics-server

minikube image load store-api:1.0 -p fiap-store
minikube image ls -p fiap-store | grep store-api
```

```
docker.io/library/store-api:1.0
```

> **Atenção:** em cluster multi-nó, o `minikube image load` distribui a imagem para **todos** os nós. Se você usar `eval $(minikube docker-env)`, a imagem vai só para o nó primário — e o Pod pode ser agendado no `m02` e falhar. Prefira `image load` quando houver mais de um nó.

**Passo 5 — Criar namespace, Deployment e Service.**

```bash
kubectl create namespace store-dev
kubectl config set-context --current --namespace=store-dev

kubectl create deployment store-api --image=store-api:1.0 --replicas=2
kubectl set env deployment/store-api APP_ENV=dev DB_HOST=store-db.store-dev.svc.cluster.local
kubectl expose deployment store-api --port=80 --target-port=8000 --type=NodePort
```

**Passo 6 — Verificar.**

```bash
kubectl get pods -o wide
```

```
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE
store-api-6c9f7d8b54-4nqhp   1/1     Running   0          38s   10.244.1.5   fiap-store-m02
store-api-6c9f7d8b54-r7wcz   1/1     Running   0          38s   10.244.0.7   fiap-store
```

```bash
kubectl get svc store-api
```

```
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
store-api   NodePort   10.101.44.9     <none>        80:31274/TCP   22s
```

**Passo 7 — Acessar `/health` de fora do cluster.**

```bash
minikube service store-api -n store-dev -p fiap-store --url
```

```
http://192.168.49.2:31274
```

```bash
URL=$(minikube service store-api -n store-dev -p fiap-store --url)
curl -s $URL/health
curl -s $URL/info
for i in 1 2 3 4 5 6; do curl -s $URL/health | grep -o '"host":"[^"]*"'; done
```

```
{"status":"ok","host":"store-api-6c9f7d8b54-4nqhp"}
{"env":"dev","db_host":"store-db.store-dev.svc.cluster.local","pod":"store-api-6c9f7d8b54-4nqhp"}
"host":"store-api-6c9f7d8b54-4nqhp"
"host":"store-api-6c9f7d8b54-r7wcz"
"host":"store-api-6c9f7d8b54-4nqhp"
"host":"store-api-6c9f7d8b54-r7wcz"
"host":"store-api-6c9f7d8b54-r7wcz"
"host":"store-api-6c9f7d8b54-4nqhp"
```

O último bloco é o mais interessante: o `hostname` alternando entre os dois Pods é a **prova visual do balanceamento de carga do Service**, feito pelas regras iptables que o `kube-proxy` programou em cada nó.

**Passo 8 — Acesso alternativo por `port-forward`.**

```bash
kubectl port-forward svc/store-api 8080:80 -n store-dev
# em outro terminal:
curl -s localhost:8080/produtos
```

```
[{"id":1,"nome":"Teclado","preco":199.9}]
```

**Passo 9 — Limpeza.**

```bash
kubectl delete namespace store-dev     # remove tudo dentro dele
minikube stop -p fiap-store            # ou: minikube delete -p fiap-store
```

---

### 2.12 Ponte com Serverless

Cronometre honestamente o laboratório que acabamos de fazer. Do zero até a primeira resposta HTTP:

| Etapa | Tempo típico |
|---|---|
| `minikube start` (2 nós, imagem base já em cache) | 60–90 s |
| `docker build` da store-api | 20–40 s |
| `minikube image load` | 10–20 s |
| `kubectl create deployment` + agendamento + start do container | 10–20 s |
| **Total até o primeiro `curl /health`** | **~2 a 3 minutos** |

Agora o mesmo caminho no **AWS Lambda**: você empacota o código (`zip` ou imagem de container), faz `aws lambda create-function`, e em **5 a 20 segundos** existe um endpoint invocável. Não há cluster, não há nó, não há Service, não há NodePort. E, crucialmente, **não há custo quando ninguém chama**.

Esse contraste não significa que o Lambda seja "melhor" — significa que os dois otimizam coisas diferentes. O que o Lambda cobra em troca da velocidade inicial:

- **Cold start.** A primeira invocação após ociosidade paga a inicialização do ambiente de execução: dezenas de milissegundos para runtimes leves, e potencialmente 1 a 3 segundos para pacotes Python grandes com muitas dependências (exatamente o caso de uma FastAPI com SQLAlchemy). No Kubernetes, o Pod já está de pé e a latência do primeiro request é a mesma do centésimo — desde que o `readinessProbe` esteja correto.
- **Modelo de execução restrito.** Máximo de 15 minutos por invocação, sem estado local entre chamadas, sem conexões persistentes triviais (um pool de conexões com PostgreSQL vira problema — daí existir o RDS Proxy).
- **Custo por invocação.** Barato para volumes baixos e médios; a partir de um certo patamar de requisições contínuas, um Deployment em nós EC2 reservados fica mais barato.

O Kubernetes não fica de fora dessa conversa. Dois projetos trazem características serverless para dentro do cluster, e o minikube consegue hospedá-los:

```bash
# KEDA - autoescalonamento dirigido a eventos (fila SQS, Kafka, RabbitMQ, cron)
helm repo add kedacore https://kedacore.github.io/charts && helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace

# Knative Serving - escalonamento a zero e roteamento por revisao
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.16.0/serving-crds.yaml
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.16.0/serving-core.yaml
```

Com o Knative, a `store-api` deixa de ter `replicas: 2` fixas e passa a ter **zero Pods quando não há tráfego**, subindo sob demanda no primeiro request — pagando, aí sim, um cold start de Kubernetes (puxar imagem se necessário, criar container, esperar readiness), que costuma ser maior que o do Lambda mas roda em qualquer nuvem. Com o KEDA, a `store-worker` sobe de 0 para N réplicas conforme a profundidade da fila de pedidos e volta a 0 quando ela esvazia.

A conclusão prática para a FIAP Store: o minikube é o lugar certo para **aprender o modelo**; o Fargate no EKS é onde você elimina a gestão de nós mantendo a API do Kubernetes; e o Lambda é onde você elimina o cluster inteiro, aceitando as restrições do modelo FaaS.

---

### 2.13 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `ErrImagePull` / `ImagePullBackOff` com imagem local | A imagem está no Docker do host, não no cluster | `minikube image ls -p X \| grep <img>`; `kubectl describe pod` (Events) | `minikube image load <img>:<tag> -p X`; usar tag versionada, não `:latest` |
| `ImagePullBackOff` mesmo após `image load` | Tag `:latest` força `imagePullPolicy: Always` | `kubectl get pod -o jsonpath='{..imagePullPolicy}'` | Usar tag versionada ou setar `imagePullPolicy: IfNotPresent` |
| `CrashLoopBackOff` | Processo do container termina logo após iniciar | `kubectl logs <pod> --previous` | Corrigir CMD/ENTRYPOINT, dependência faltante ou variável de ambiente obrigatória |
| `minikube start` falha com `Exiting due to RSRC_INSUFFICIENT_CORES` | `--cpus` maior que o disponível | `nproc`; `free -m` | Reduzir `--cpus`/`--memory` ou aumentar os recursos do Docker Desktop |
| `EXTERNAL-IP` fica `<pending>` para sempre | Service `LoadBalancer` sem cloud-controller-manager | `kubectl get svc` | Rodar `sudo minikube tunnel -p X`, ou usar NodePort |
| `Unable to connect to the server: dial tcp 192.168.49.2:8443: connect: no route to host` | Cluster parado ou contexto apontando para perfil errado | `minikube status -p X`; `kubectl config current-context` | `minikube start -p X`; `kubectl config use-context X` |
| Pod fica `Pending` em cluster multi-nó com a imagem "presente" | Imagem só existe no nó primário (uso de `docker-env`) | `kubectl get pod -o wide` e `minikube ssh -n m02 -- docker images` | Usar `minikube image load` (distribui a todos os nós) |
| `kubectl top pods` retorna `error: Metrics API not available` | metrics-server ausente ou ainda coletando | `kubectl get deploy -n kube-system metrics-server` | `minikube addons enable metrics-server`; aguardar ~60 s |
| Objetos "somem" após reiniciar o terminal | Namespace do contexto não foi fixado; objetos criados em `default` | `kubectl get all -A \| grep store` | `kubectl config set-context --current --namespace=store-dev` |
| `permission denied` ao falar com o Docker | Usuário fora do grupo `docker` | `groups \| grep docker` | `sudo usermod -aG docker $USER && newgrp docker` |

---

### 2.14 Resumo do capítulo

Um cluster local é a bancada de trabalho de quem aprende Kubernetes: gratuito, rápido de recriar e — o que mais importa — **expondo exatamente a mesma API** de um EKS de produção. Escolhemos o minikube pelo sistema de addons, pelo suporte a múltiplos nós e pelo conceito de perfis. Vimos que o `minikube start` é, na prática, um provisionador que também reescreve o seu `~/.kube/config`, e que entender esse arquivo — clusters, users, contexts — é pré-requisito para trabalhar com mais de um ambiente sem acidentes. Construímos a imagem da `store-api` com Dockerfile multi-stage e usuário não-root, aprendemos o ciclo `build → image load → deployment → expose` que evita a necessidade de um registry externo, e comprovamos o balanceamento de carga do Service observando o `hostname` alternar entre réplicas.

Pontos-chave:

1. minikube para a bancada de estudo; kind para CI; k3d para máquinas modestas; EKS quando o assunto for nuvem de verdade.
2. Sempre explicite `--profile`, `--driver`, `--kubernetes-version`, `--cpus` e `--memory`: ambiente reprodutível é ambiente depurável.
3. O kubeconfig é (cluster + user + namespace) = contexto. `kubectl config set-context --current --namespace=X` evita metade dos acidentes de laboratório.
4. Imagem construída localmente **não** existe dentro do cluster: use `minikube image load` — obrigatoriamente, se houver mais de um nó.
5. Tag `:latest` implica `imagePullPolicy: Always` e quebra o uso de imagens locais. Use tags versionadas.
6. Fluxo de diagnóstico canônico: `get` → `describe` (leia os Events!) → `logs` (com `--previous` se houve restart) → `exec`/`port-forward`.
7. `minikube service --url` para NodePort, `minikube tunnel` para LoadBalancer, `kubectl port-forward` para qualquer coisa em qualquer cluster — inclusive EKS.

---

### 2.15 Exercícios (para casa)

#### Exercício 2.1 — Provar o cluster (fixação guiada) · 25 min

- **Objetivo:** instalar o minikube, subir um cluster com parâmetros explícitos e coletar evidências do seu estado.
- **Enunciado:** Instale o minikube no seu sistema. Crie um cluster com perfil `fiap-lab`, driver `docker`, Kubernetes `v1.34.1`, 2 CPUs e 4096 MB de memória. Colete e comente as seguintes evidências: (a) `minikube version` e `kubectl version`; (b) `kubectl get nodes -o wide` — identifique na saída a versão do kubelet, o container runtime e o IP interno; (c) `kubectl get pods -n kube-system` — associe **cada** Pod listado ao componente correspondente do diagrama de arquitetura do Capítulo 1, dizendo em uma linha o que ele faz; (d) `kubectl cluster-info`; (e) um **resumo** do `kubectl cluster-info dump` — não entregue o dump inteiro (são milhares de linhas), mas extraia dele, com `grep`/`jq`, três informações: a versão do apiserver, o CIDR de Pods do cluster e o nome da StorageClass padrão; (f) `minikube addons list`. Ao final, execute `minikube ssh` e liste o conteúdo de `/etc/kubernetes/manifests`, explicando em duas linhas por que esses arquivos existem e o que são *static pods*.
- **Entregável:** `ex2-1.md` com os comandos e saídas comentados, e a explicação final sobre static pods.
- **Critério de aceite:** `kubectl get nodes` mostra `Ready` e versão `v1.34.x`; o item (c) associa corretamente pelo menos etcd, kube-apiserver, kube-scheduler, kube-controller-manager, coredns, kube-proxy e storage-provisioner; o item (e) traz as três informações extraídas por comando (não copiadas da documentação); a listagem de `/etc/kubernetes/manifests` contém os quatro YAMLs do control plane.
- <details><summary>Dica</summary>Para o item (e): <code>kubectl get sc</code>, <code>kubectl cluster-info dump | grep -m1 'cluster-cidr'</code> e <code>kubectl version -o json | jq .serverVersion.gitVersion</code>. Static pods têm o nome do nó como sufixo (ex.: <code>etcd-fiap-lab</code>).</details>

#### Exercício 2.2 — Rodar a store-api no minikube (aplicação) · 45 min

- **Objetivo:** construir a imagem da aplicação do semestre, carregá-la no cluster e expô-la com variáveis de ambiente.
- **Enunciado:** Usando o `main.py` e o Dockerfile multi-stage do laboratório, construa a imagem `store-api:1.0`. **Requisitos obrigatórios da imagem:** base `python:3.12-slim`, dois estágios (`builder` e `runtime`), usuário não-root com UID 10001, e o processo escutando em `0.0.0.0:8000`. Comprove que o container **não** roda como root executando `docker run --rm store-api:1.0 id`. Carregue a imagem no cluster `fiap-lab`, crie o namespace `store-dev`, fixe-o no contexto atual, e crie um Deployment `store-api` com **3 réplicas** e as variáveis `APP_ENV=dev`, `DB_HOST=store-db.store-dev.svc.cluster.local` e `DB_PASSWORD=trocar-depois`. Exponha via NodePort com `port=80` e `targetPort=8000`. Depois: (a) acesse `/health`, `/info` e `/produtos` de fora do cluster; (b) faça 10 requisições a `/health` e mostre que o `host` retornado alterna entre as 3 réplicas; (c) prove, com `kubectl exec`, que as três variáveis de ambiente chegaram dentro do container; (d) mostre o consumo com `kubectl top pods`. Finalize escrevendo em 3 a 5 linhas por que armazenar `DB_PASSWORD` como variável de ambiente em um Deployment é uma prática ruim e o que usaríamos no lugar.
- **Entregável:** pasta `ex2-2/` com `main.py`, `requirements.txt` e `Dockerfile`; arquivo `ex2-2.md` com todos os comandos e saídas, incluindo a saída de `docker run --rm store-api:1.0 id` e a de `kubectl get pods -o wide`.
- **Critério de aceite:** `id` retorna `uid=10001` (não 0); os 3 Pods estão `1/1 Running` e distribuídos entre os nós; `curl $URL/info` devolve `"env":"dev"` e o `db_host` configurado; as 10 requisições exibem ao menos 2 hostnames distintos; a reflexão final menciona `Secret` (e idealmente External Secrets / AWS Secrets Manager).
- <details><summary>Dica</summary><code>kubectl set env deployment/store-api APP_ENV=dev DB_HOST=... DB_PASSWORD=...</code> aplica as três de uma vez; para o item (c) use <code>kubectl exec deploy/store-api -- env | grep -E 'APP_ENV|DB_'</code>.</details>

#### Exercício 2.3 — Diagnóstico de ImagePullBackOff e CrashLoopBackOff (análise) · 40 min

- **Objetivo:** reproduzir deliberadamente as duas falhas mais comuns de Pod e diagnosticá-las com o fluxo `get → describe → logs`.
- **Enunciado:** Você vai **quebrar o cluster de propósito** e consertar. **Parte A — ImagePullBackOff:** no namespace `store-dev`, crie um Deployment `quebrado-1` com a imagem `store-api:9.9` (que não existe em lugar nenhum). Observe o Pod, colete `kubectl get pods`, `kubectl describe pod` (seção Events completa) e responda: qual componente emitiu o evento? Qual a diferença entre os status `ErrImagePull` e `ImagePullBackOff`? Por que o intervalo entre tentativas aumenta? Corrija **sem deletar o Deployment**. **Parte B — CrashLoopBackOff:** crie um Deployment `quebrado-2` a partir da imagem `store-api:1.0` mas sobrescrevendo o comando para `sh -c "echo iniciando; sleep 3; echo caiu; exit 1"`. Colete `kubectl get pods -w` por 2 minutos (mostre a coluna RESTARTS crescendo), `kubectl describe pod`, `kubectl logs <pod>` e `kubectl logs <pod> --previous`. Responda: por que `kubectl logs` sem `--previous` frequentemente falha ou vem vazio nesse caso? Qual `restartPolicy` está em vigor e por quê? Qual é o intervalo máximo do backoff? **Parte C — o caso silencioso:** crie um Deployment `quebrado-3` usando a imagem `store-api:1.0` e um `livenessProbe` HTTP apontando para o caminho **errado** `/healthz` na porta 8000, com `initialDelaySeconds: 5` e `periodSeconds: 5`. Explique por que o Pod chega a ficar `Running` e depois entra em restart cíclico, e qual evento no `describe` denuncia a causa. Para cada uma das três partes, entregue: sintoma, diagnóstico, comando que revelou a causa e correção aplicada.
- **Entregável:** `ex2-3.md` com as três partes documentadas (comandos + saídas + respostas às perguntas), e os manifestos/comandos usados para criar cada Deployment quebrado.
- **Critério de aceite:** as três falhas foram reproduzidas e as saídas colam com o sintoma descrito; a Parte A explica corretamente que `ErrImagePull` é a falha imediata e `ImagePullBackOff` é o estado de espera exponencial; a Parte B mostra `RESTARTS` crescendo e usa `--previous` para obter o log do container morto; a Parte C identifica o evento `Liveness probe failed: HTTP probe failed with statuscode: 404` e a diferença entre estar `Running` e estar saudável. Ao final, `kubectl get pods` mostra todos os Deployments corrigidos e `Running`.
- <details><summary>Dica</summary>Para a Parte B: <code>kubectl create deployment quebrado-2 --image=store-api:1.0 -- sh -c "echo iniciando; sleep 3; exit 1"</code>. Para a Parte C, gere o manifesto com <code>--dry-run=client -o yaml</code>, adicione o <code>livenessProbe</code> à mão e aplique. O backoff do kubelet dobra a cada falha até o teto de 5 minutos.</details>

#### Exercício 2.4 — Desafio: `setup-lab.sh` idempotente (desafio integrador) · 60 min

- **Objetivo:** automatizar todo o provisionamento do laboratório em um script que possa ser executado repetidamente sem efeitos colaterais.
- **Enunciado:** Escreva `setup-lab.sh` em Bash que deixe uma máquina limpa pronta para a disciplina. O script deve, em ordem: **(1) Verificar pré-requisitos** — `docker`, `minikube` e `kubectl` presentes e nas versões mínimas (Docker 27+, minikube 1.35+, kubectl 1.33+), abortando com mensagem clara e código de saída diferente de zero se algo faltar. **(2) Criar (ou reaproveitar) o perfil** `fiap-store` com 2 nós, driver `docker`, Kubernetes `v1.34.1`, 2 CPUs e 4096 MB — se o perfil já existir e estiver rodando, **não recriar**; se existir e estiver parado, apenas dar `start`. **(3) Habilitar os addons** `metrics-server`, `ingress` e `dashboard`, pulando os que já estiverem habilitados. **(4) Criar os namespaces** `store-dev` e `store-prod` se não existirem, e fixar `store-dev` no contexto atual. **(5) Construir e carregar a imagem** `store-api:1.0` a partir da pasta `./store-api`, pulando o build se a imagem já existir no cluster com o mesmo digest. **(6) Validar o ambiente** com uma função `validar()` que checa, imprimindo `[OK]`/`[FALHA]` para cada item: os 2 nós estão `Ready`; os namespaces existem; a imagem está presente no cluster; `kubectl top nodes` responde; o ingress-nginx controller está `Running`. O script deve terminar com código 0 apenas se **todas** as validações passarem. **Requisitos técnicos obrigatórios:** `set -euo pipefail`; uma função `log()` com timestamp; suporte a `--profile NOME` e `--destroy` (que executa `minikube delete -p NOME` após confirmação interativa); nenhuma senha ou token no código. **Prova de idempotência:** execute o script **três vezes seguidas** e mostre que a segunda e a terceira execuções terminam em menos de 30 segundos, sem criar nada novo e sem erro.
- **Entregável:** `setup-lab.sh` (executável) e `ex2-4.md` contendo a saída completa das três execuções consecutivas e a saída de `./setup-lab.sh --destroy`.
- **Critério de aceite:** `bash -n setup-lab.sh` não acusa erro de sintaxe; as três execuções terminam com código 0 (`echo $?`); a 2ª e a 3ª execuções não recriam o cluster nem os namespaces (visível no log); a função `validar()` imprime `[OK]` para os cinco itens; `--destroy` remove o perfil e pede confirmação antes.
- <details><summary>Dica</summary>Idempotência sai de padrões como <code>kubectl create namespace store-dev --dry-run=client -o yaml | kubectl apply -f -</code> e <code>minikube status -p "$PROFILE" &gt;/dev/null 2&gt;&amp;1 || minikube start ...</code>. Para os addons, filtre <code>minikube addons list -o json</code> com <code>jq</code> antes de habilitar.</details>

---

### 2.16 Leitura recomendada

1. **[DOC]** minikube — *Documentation: Get Started, Drivers, Handbook*. Disponível em: https://minikube.sigs.k8s.io/docs/ — ler especialmente "Handbook → Pushing images" e "Handbook → Accessing apps".
2. **[DOC]** Kubernetes — *Organizing Cluster Access Using kubeconfig Files*. Disponível em: https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/
3. **[DOC]** Kubernetes — *kubectl Quick Reference (Cheat Sheet)*. Disponível em: https://kubernetes.io/docs/reference/kubectl/quick-reference/
4. **[LIVRO]** ARUNDEL, J.; DOMINGUS, J. *Cloud Native DevOps with Kubernetes*. 2. ed. O'Reilly, 2022 — capítulos 2 e 3 (ambientes locais e primeiros deploys).
5. **[LIVRO]** RICE, L. *Container Security*. O'Reilly, 2020 — capítulo 6, sobre por que containers não devem rodar como root.
6. **[DOC]** KEDA — *Getting Started* (https://keda.sh/docs/) e Knative — *Install Serving* (https://knative.dev/docs/install/) — prévia para a ponte com serverless.

---

## Capítulo 3 — Criando Pods e configurando Serviços (arquivos YAML)

> **Aula:** 2 horas · **Pré-requisitos:** Capítulos 1 e 2; cluster minikube `fiap-store` operante; imagem `store-api:1.0` carregada no cluster · **Artefatos:** `pod.yaml`, `service-clusterip.yaml`, `service-nodeport.yaml`, `service-headless.yaml`, `pod-multi.yaml`

---

### 3.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Escrever** YAML válido para o Kubernetes, identificando as armadilhas clássicas da linguagem (indentação, tipos implícitos, o "Norway problem").
2. **Descrever** a anatomia de um manifesto (`apiVersion`, `kind`, `metadata`, `spec`, `status`) e o ciclo de vida completo de um Pod, incluindo suas fases e políticas de reinício.
3. **Construir** um Pod com `initContainers`, sidecars, probes de liveness/readiness/startup e definição de `requests`/`limits`, explicando a classe de QoS resultante.
4. **Diferenciar** os tipos de Service (ClusterIP, NodePort, LoadBalancer, ExternalName, headless) e **aplicar** o tipo correto a um cenário dado.
5. **Analisar** a relação entre labels, selectors, Endpoints e EndpointSlice, diagnosticando a causa quando um Service não entrega tráfego.
6. **Comprovar** a resolução de nomes do DNS interno (`svc.namespace.svc.cluster.local`) a partir de um Pod cliente.

---

### 3.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | YAML essencial e suas armadilhas |
| 2 | 15 min | Anatomia do manifesto; Pod em profundidade: fases, restartPolicy, initContainers, sidecars |
| 3 | 20 min | Probes (liveness, readiness, startup) e QoS (requests/limits) |
| 4 | 10 min | env, envFrom, ConfigMap; command/args x ENTRYPOINT/CMD; por que não usar Pod "pelado" |
| 5 | 10 min | **Intervalo** |
| 6 | 25 min | Services: tipos, selector, Endpoints/EndpointSlice, DNS interno, sessionAffinity |
| 7 | 20 min | Laboratório guiado: pod.yaml + service.yaml, teste de DNS, quebra proposital do selector |
| 8 | 5 min | Ponte com Serverless: probes como health check gerenciado; scale-to-zero |
| 9 | 5 min | Erros comuns, `apply` x `create` x `replace`, exercícios |

---

### 3.3 YAML essencial (e as armadilhas que derrubam a turma)

YAML é a linguagem de configuração do Kubernetes. Ela foi projetada para ser legível por humanos, e essa escolha tem um preço: várias construções parecem óbvias e significam outra coisa.

#### 3.3.1 O básico

```yaml
# Mapa (dicionario): chave: valor
nome: store-api
porta: 8000
ativo: true

# Aninhamento por INDENTACAO (2 espacos, NUNCA tab)
metadata:
  name: store-api
  labels:
    app: store-api
    tier: backend

# Lista (sequencia): hifen + espaco
portas:
  - 80
  - 443

# Lista de mapas: o hifen inicia o item, o resto alinha com a primeira chave
containers:
  - name: store-api
    image: store-api:1.0
    ports:
      - containerPort: 8000
  - name: sidecar
    image: busybox:1.36
```

#### 3.3.2 Strings multilinha

```yaml
# "|" (literal block): PRESERVA as quebras de linha
script: |
  #!/bin/sh
  echo "iniciando"
  exec uvicorn main:app --host 0.0.0.0

# ">" (folded block): DOBRA as quebras em espacos, vira uma linha so
descricao: >
  Esta e a API principal
  da FIAP Store.
# resultado: "Esta e a API principal da FIAP Store.\n"

# "|-" e ">-" removem a quebra de linha FINAL (chomping)
comando: |-
  uvicorn main:app
```

O uso mais comum de `|` no Kubernetes é dentro de ConfigMaps que carregam arquivos inteiros (nginx.conf, application.yaml, scripts de inicialização).

#### 3.3.3 Âncoras e aliases

```yaml
labels_padrao: &labels_store
  app: store-api
  tier: backend
  env: dev

metadata:
  labels:
    <<: *labels_store      # merge key: herda todas as chaves
    version: "1.0"         # e adiciona/sobrescreve
```

> **Nota:** âncoras funcionam, mas usam-se pouco em manifestos Kubernetes reais, porque não sobrevivem à ida e volta pela API (o servidor armazena o objeto já resolvido) e porque as ferramentas do ecossistema — Helm, Kustomize — resolvem reuso de forma mais poderosa. Conheça a sintaxe para ler código alheio; prefira Kustomize para escrever o seu.

#### 3.3.4 Multi-documento

O separador `---` permite empacotar vários objetos em um arquivo. É extremamente comum agrupar um Deployment e seu Service:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: store-dev
---
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
spec:
  selector:
    app: store-api
  ports:
    - port: 80
      targetPort: 8000
```

O `kubectl apply -f arquivo.yaml` cria os dois. Também vale `kubectl apply -f ./manifests/` (diretório inteiro) e `kubectl apply -k ./overlays/dev` (Kustomize).

#### 3.3.5 As armadilhas

**O "Norway problem".** YAML 1.1 interpreta um conjunto surpreendente de palavras como booleano: `yes`, `no`, `on`, `off`, `true`, `false`, `y`, `n` — em maiúsculas, minúsculas ou capitalizadas. O nome do problema vem do código ISO da Noruega: `NO`, que vira `false`.

```yaml
# ERRADO - vira booleano
pais: NO
debug: on
resposta: yes

# CERTO - aspas forcam string
pais: "NO"
debug: "on"
resposta: "yes"
```

**Números que deveriam ser strings.** Versões, portas em contextos de string e valores octais são o caso clássico:

```yaml
version: 1.10     # vira o FLOAT 1.1  -- a versao "1.10" virou "1.1"
version: "1.10"   # correto

modo: 0644        # em YAML 1.1 pode ser lido como octal; o Kubernetes usa inteiro decimal
defaultMode: 0644 # em Secret/ConfigMap, escreva 0644 mesmo (o schema e integer)
```

**Tabs.** YAML **proíbe** tabulação para indentação. O erro `found character '\t' that cannot start any token` é sempre isso. Configure seu editor: `.editorconfig` com `indent_style = space` e `indent_size = 2`.

**`null` acidental.** Uma chave sem valor vira `null`, não string vazia:

```yaml
imagePullSecrets:      # isto e null, nao lista vazia -> pode causar erro de validacao
imagePullSecrets: []   # lista vazia explicita
```

**Aspas dentro de valores com `{}` ou `[]`.** O YAML tem sintaxe de fluxo (estilo JSON), então `args: [--verbose]` funciona, mas `args: [--host=0.0.0.0, --port=8000]` pode confundir o parser. Prefira a forma em bloco com hífens.

> **Na prática:** valide sempre antes de aplicar. `kubectl apply --dry-run=server -f pod.yaml` envia o objeto ao apiserver, roda validação e admission controllers, e **não persiste** — é o teste mais fiel possível. O `--dry-run=client` só valida localmente. Ferramentas como `kubeconform` e `kubectl-validate` fazem validação de esquema offline em pipelines de CI.

---

### 3.4 Anatomia de um manifesto

Todo objeto tem cinco campos de topo, dos quais quatro você escreve:

```yaml
apiVersion: v1        # [obrigatorio] grupo/versao da API
kind: Pod             # [obrigatorio] tipo do objeto
metadata:             # [obrigatorio] identidade
  name: store-api     #   [obrigatorio] unico dentro do namespace + kind
  namespace: store-dev
  labels: {}          #   selecionaveis
  annotations: {}     #   nao selecionaveis
spec:                 # [obrigatorio na maioria] estado DESEJADO - voce escreve
  ...
status:               # estado OBSERVADO - o sistema escreve; voce nunca envia
  ...
```

Campos de `metadata` preenchidos pelo servidor e que você não deve escrever: `uid`, `resourceVersion` (usado para controle de concorrência otimista), `creationTimestamp`, `generation`, `managedFields` (rastreamento do Server-Side Apply). Ao fazer `kubectl get pod X -o yaml`, você vê tudo isso — mas ao reaplicar aquele YAML, o servidor ignora ou reclama.

---

### 3.5 Pod em profundidade

O **Pod** é a menor unidade que o Kubernetes agenda. Não é "um container": é um **grupo de um ou mais containers que compartilham namespace de rede (mesmo IP e portas), namespace IPC, e possivelmente volumes**, e que são sempre agendados juntos no mesmo nó, sobem juntos e morrem juntos.

Essa definição tem uma consequência prática imediata: dois containers no mesmo Pod falam entre si por `localhost`, e **não podem** usar a mesma porta. É o mesmo modelo de "processos numa máquina", trazido para o nível do container.

#### 3.5.1 Manifesto completo

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
    env: dev
spec:
  restartPolicy: Always
  containers:
    - name: store-api
      image: store-api:1.0
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 8000
          protocol: TCP
      env:
        - name: APP_ENV
          value: "dev"
        - name: DB_HOST
          value: "store-db.store-dev.svc.cluster.local"
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
      startupProbe:
        httpGet:
          path: /health
          port: http
        failureThreshold: 30
        periodSeconds: 2
      readinessProbe:
        httpGet:
          path: /ready
          port: http
        initialDelaySeconds: 3
        periodSeconds: 5
        timeoutSeconds: 2
        failureThreshold: 3
        successThreshold: 1
      livenessProbe:
        httpGet:
          path: /health
          port: http
        initialDelaySeconds: 10
        periodSeconds: 10
        timeoutSeconds: 2
        failureThreshold: 3
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
```

#### 3.5.2 Fases e estados

O campo `status.phase` de um Pod assume exatamente cinco valores:

| Fase | Significado |
|---|---|
| `Pending` | O objeto foi aceito, mas ao menos um container ainda não está rodando: pode estar aguardando agendamento, download de imagem ou montagem de volume |
| `Running` | O Pod foi ligado a um nó e todos os containers foram criados; ao menos um está rodando ou reiniciando |
| `Succeeded` | Todos os containers terminaram com sucesso (código 0) e não serão reiniciados |
| `Failed` | Todos terminaram e ao menos um falhou (código != 0) ou foi encerrado pelo sistema |
| `Unknown` | O estado não pôde ser obtido — normalmente falha de comunicação com o kubelet do nó |

**Não confunda fase com prontidão.** `Running` significa "os processos existem"; não significa que a aplicação responde. É por isso que existe a condição `Ready`, alimentada pela readiness probe, e é ela que decide se o Pod recebe tráfego de um Service.

Estados de container (`status.containerStatuses[].state`) são mais informativos que a fase e é o que você lê no `describe`: `Waiting` (com `reason: ContainerCreating`, `ErrImagePull`, `ImagePullBackOff`, `CrashLoopBackOff`, `CreateContainerConfigError`), `Running` e `Terminated` (com `reason: Completed`, `Error`, `OOMKilled`).

#### 3.5.3 restartPolicy

Aplica-se ao **Pod inteiro** e vale para todos os seus containers:

| Valor | Comportamento | Usado em |
|---|---|---|
| `Always` (padrão) | Reinicia o container sempre que ele termina, com qualquer código de saída | Deployment, StatefulSet, DaemonSet |
| `OnFailure` | Reinicia apenas se o código de saída for diferente de 0 | Job, CronJob |
| `Never` | Nunca reinicia | Jobs de execução única, tarefas de depuração |

O reinício usa **backoff exponencial**: 10s, 20s, 40s... até o teto de **5 minutos**, e o contador zera após 10 minutos de execução bem-sucedida. É esse mecanismo que produz o famoso status `CrashLoopBackOff` — que, note bem, **não é um erro em si**: é o kubelet dizendo "estou esperando antes de tentar de novo".

> **Atenção:** `restartPolicy` reinicia o container **no mesmo nó, no mesmo Pod**. Ele não reagenda nada. Se o nó morrer, quem recria o Pod em outro lugar é o controlador de nível superior (ReplicaSet/Deployment) — não o Pod.

#### 3.5.4 initContainers

`initContainers` rodam **antes** dos containers principais, em sequência, e cada um precisa terminar com sucesso para o próximo começar. Usos típicos: esperar uma dependência ficar disponível, rodar migração de banco, buscar configuração ou clonar um repositório.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api-init
  namespace: store-dev
  labels:
    app: store-api
spec:
  initContainers:
    - name: espera-db
      image: busybox:1.36
      command:
        - sh
        - -c
        - |
          echo "aguardando o banco em ${DB_HOST}:5432..."
          until nc -z ${DB_HOST} 5432; do
            echo "ainda indisponivel, tentando de novo em 2s"
            sleep 2
          done
          echo "banco disponivel"
      env:
        - name: DB_HOST
          value: "store-db.store-dev.svc.cluster.local"
  containers:
    - name: store-api
      image: store-api:1.0
      ports:
        - containerPort: 8000
```

Enquanto o initContainer roda, o Pod fica `Pending` com o status `Init:0/1`. Se ele falhar e o `restartPolicy` for `Always`, o Pod reinicia o init.

#### 3.5.5 Sidecars — o padrão e a novidade da API

O **padrão sidecar** é um container auxiliar que roda ao lado do container principal, compartilhando rede e volumes: coletor de logs, proxy de service mesh, exportador de métricas, sincronizador de configuração.

Historicamente, um sidecar era simplesmente mais um item em `spec.containers`. Isso funciona, mas tem dois defeitos conhecidos: (a) não há garantia de ordem — o app pode subir antes do proxy estar pronto; (b) em Jobs, o Pod nunca termina, porque o sidecar continua rodando depois que o container principal completou.

A partir do Kubernetes **1.29** (beta) e estável na **1.33**, existe a solução oficial: um **sidecar é um initContainer com `restartPolicy: Always`**.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api-sidecar
  namespace: store-dev
  labels:
    app: store-api
spec:
  volumes:
    - name: logs
      emptyDir: {}
  initContainers:
    # sidecar nativo: inicia ANTES dos containers principais,
    # permanece rodando, e NAO impede o Pod de terminar
    - name: log-shipper
      image: busybox:1.36
      restartPolicy: Always          # <- isto o transforma em sidecar
      command: ["sh", "-c", "tail -F /var/log/app/access.log"]
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
  containers:
    - name: store-api
      image: store-api:1.0
      ports:
        - containerPort: 8000
      volumeMounts:
        - name: logs
          mountPath: /var/log/app
```

O `emptyDir` é um volume efêmero criado quando o Pod é agendado e destruído quando o Pod morre. É o mecanismo padrão para compartilhar arquivos entre containers do mesmo Pod.

#### 3.5.6 Containers efêmeros e `kubectl debug`

Imagens de produção bem construídas são mínimas — `python:3.12-slim`, ou até `distroless` — e não têm `curl`, `ps` nem sequer `sh`. Como depurar um Pod assim? Com **ephemeral containers**: containers injetados em um Pod **em execução**, sem reiniciá-lo.

```bash
# injeta um container de debug no Pod, compartilhando o namespace de processos
kubectl debug -it store-api -n store-dev \
  --image=nicolaka/netshoot --target=store-api -- bash

# dentro dele:
curl -s localhost:8000/health
ps aux
ss -tlnp
nslookup store-api.store-dev.svc.cluster.local
```

Outra variante muito útil cria uma **cópia** do Pod com o comando trocado, sem tocar no original:

```bash
kubectl debug store-api -n store-dev --copy-to=store-api-debug \
  --container=store-api -- sh -c "sleep infinity"
```

E é possível depurar o **nó** inteiro:

```bash
kubectl debug node/fiap-store-m02 -it --image=busybox
# o sistema de arquivos do no aparece em /host
```

---

### 3.6 Probes: como o Kubernetes sabe se sua aplicação está viva

Três probes, três perguntas diferentes. Confundi-las é a fonte de incidentes mais comum em clusters de produção.

| Probe | Pergunta | Ação em caso de falha | Quando usar |
|---|---|---|---|
| **startupProbe** | "Já terminou de inicializar?" | Mata o container (e desabilita as outras probes até passar) | Aplicações com boot lento (JVM, carga de modelo, migração) |
| **readinessProbe** | "Pode receber tráfego agora?" | **Remove o Pod dos Endpoints** — não mata nada | Praticamente sempre |
| **livenessProbe** | "Está travado sem recuperação?" | **Mata e reinicia o container** | Quando há travamento real (deadlock); use com cautela |

Três mecanismos de verificação:

```yaml
# httpGet: 2xx ou 3xx = sucesso
readinessProbe:
  httpGet:
    path: /ready
    port: http              # pode ser o NOME da porta ou o numero
    scheme: HTTP
    httpHeaders:
      - name: X-Probe
        value: kubelet

# tcpSocket: sucesso se a conexao TCP abrir
readinessProbe:
  tcpSocket:
    port: 5432

# exec: sucesso se o comando sair com codigo 0
livenessProbe:
  exec:
    command: ["sh", "-c", "pg_isready -U postgres"]

# grpc (estavel desde 1.27)
readinessProbe:
  grpc:
    port: 9000
```

Parâmetros comuns a todas:

| Parâmetro | Padrão | Significado |
|---|---|---|
| `initialDelaySeconds` | 0 | Espera antes da primeira verificação |
| `periodSeconds` | 10 | Intervalo entre verificações |
| `timeoutSeconds` | 1 | Tempo máximo de resposta antes de considerar falha |
| `successThreshold` | 1 | Sucessos consecutivos para voltar a "sucesso" (liveness/startup só aceitam 1) |
| `failureThreshold` | 3 | Falhas consecutivas para acionar a ação |
| `terminationGracePeriodSeconds` | herdado do Pod | Tempo de graça específico da probe ao matar |

> **Atenção — o erro clássico.** Uma `livenessProbe` que aponta para um endpoint que consulta o **banco de dados** transforma uma indisponibilidade do banco em um `CrashLoopBackOff` de toda a frota da aplicação: o banco cai, todas as réplicas falham a liveness, todas são mortas e reiniciadas, e quando o banco volta você tem uma tempestade de reinícios. A regra é: **liveness verifica apenas o próprio processo** (o event loop está respondendo?); **readiness verifica as dependências** (consigo atender uma requisição de verdade agora?). No `main.py` da FIAP Store, `/health` é liveness e `/ready` é readiness — exatamente por isso.

> **Nota:** o tempo máximo até uma falha ser detectada é `initialDelaySeconds + failureThreshold × periodSeconds`. Com os valores do nosso manifesto (`10 + 3×10`), a liveness leva até 40 segundos para matar um container travado. Ajuste conscientemente.

---

### 3.7 Requests, limits e classes de QoS

```yaml
resources:
  requests:            # o que o SCHEDULER usa para decidir onde cabe; e a garantia
    cpu: "100m"        # 100 milicores = 0,1 de um core
    memory: "128Mi"
  limits:              # o teto imposto pelo runtime (cgroups)
    cpu: "500m"
    memory: "256Mi"
```

CPU e memória se comportam de forma **fundamentalmente diferente** ao estourar o limite:

- **CPU é compressível.** Ultrapassar o limite resulta em *throttling*: o processo é desacelerado, mas não morre. Sintoma: latência alta sem erro nenhum no log.
- **Memória é incompressível.** Ultrapassar o limite resulta em **OOMKill**: o container é morto imediatamente pelo kernel, com `reason: OOMKilled` e código de saída 137.

```bash
kubectl get pod store-api -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'
# OOMKilled
```

As três **classes de QoS** são derivadas automaticamente pelo Kubernetes e determinam quem morre primeiro quando o **nó** fica sem memória:

| Classe | Condição | Prioridade de despejo |
|---|---|---|
| **Guaranteed** | Todos os containers têm requests **e** limits definidos, e `requests == limits` para CPU e memória | Última a ser despejada |
| **Burstable** | Ao menos um container tem request definido, mas não satisfaz Guaranteed | Intermediária |
| **BestEffort** | Nenhum request nem limit em nenhum container | **Primeira a ser despejada** |

```bash
kubectl get pod store-api -o jsonpath='{.status.qosClass}'
# Burstable
```

> **Na prática:** defina **sempre** requests. Sem eles, o scheduler assume que o Pod não consome nada, empacota demais no nó e produz despejos em cascata. Para memória, uma prática consolidada é `requests == limits` (evita OOMKill por vizinho barulhento). Para CPU, muitos times definem request sem limit, para permitir picos oportunistas — a discussão é controversa, mas o consenso mínimo é: nunca deixe um Pod BestEffort em produção.

---

### 3.8 Configuração: env, envFrom, ConfigMap, command e args

#### 3.8.1 Variáveis de ambiente

```yaml
env:
  # (1) valor literal
  - name: APP_ENV
    value: "dev"

  # (2) de um ConfigMap
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: store-config
        key: db_host

  # (3) de um Secret
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: store-secret
        key: db_password

  # (4) da Downward API - metadados do proprio Pod
  - name: POD_NAME
    valueFrom:
      fieldRef:
        fieldPath: metadata.name
  - name: NODE_NAME
    valueFrom:
      fieldRef:
        fieldPath: spec.nodeName
  - name: CPU_LIMIT
    valueFrom:
      resourceFieldRef:
        containerName: store-api
        resource: limits.cpu

# (5) TODAS as chaves de um ConfigMap/Secret de uma vez
envFrom:
  - configMapRef:
      name: store-config
  - secretRef:
      name: store-secret
      optional: true
```

ConfigMap correspondente:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: store-config
  namespace: store-dev
data:
  APP_ENV: "dev"
  DB_HOST: "store-db.store-dev.svc.cluster.local"
  LOG_LEVEL: "info"
  app.properties: |
    timeout=30
    retries=3
```

> **Atenção:** variáveis de ambiente vindas de ConfigMap são resolvidas **no momento da criação do container**. Alterar o ConfigMap depois **não** atualiza os Pods existentes — é preciso reiniciar (`kubectl rollout restart deployment/store-api`). Se o ConfigMap for montado como **volume**, aí sim o conteúdo é atualizado automaticamente (com atraso de até ~60 s), mas a aplicação precisa reler o arquivo.

#### 3.8.2 command e args versus ENTRYPOINT e CMD

Esta correspondência confunde quase todo mundo:

| Dockerfile | Kubernetes | Se você definir no Pod... |
|---|---|---|
| `ENTRYPOINT` | `command` | Sobrescreve o ENTRYPOINT da imagem |
| `CMD` | `args` | Sobrescreve o CMD da imagem |

Regra: se você define apenas `command`, o `CMD` da imagem é **descartado**. Se define apenas `args`, o `ENTRYPOINT` é preservado e recebe os novos argumentos.

```yaml
containers:
  - name: store-api
    image: store-api:1.0
    command: ["uvicorn"]
    args: ["main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "2"]
```

#### 3.8.3 Por que não se usa Pod "pelado" em produção

Um Pod criado diretamente — sem um controlador acima dele — é chamado de *naked pod*. Ele é ótimo para aprender e para depurar, e **inadequado para produção**, por cinco razões:

1. **Não é recriado se o nó morrer.** O `restartPolicy` reinicia o container no mesmo nó; se o nó desaparece, o Pod desaparece com ele e ninguém o traz de volta.
2. **Não escala.** Não existe `kubectl scale pod`. Você teria de criar cópias manualmente com nomes diferentes.
3. **Não tem rolling update.** Trocar a imagem exige deletar e recriar, com downtime.
4. **É despejado em drain.** `kubectl drain` durante manutenção do nó remove o Pod definitivamente.
5. **A maior parte do `spec` é imutável.** Só se pode alterar `image`, `activeDeadlineSeconds`, `tolerations` e alguns poucos campos. Mudar recursos ou probes exige recriar o objeto.

Em produção, usa-se **Deployment** (stateless), **StatefulSet** (identidade e storage estáveis), **DaemonSet** (um por nó) ou **Job/CronJob** (execução finita). Todos eles gerenciam Pods para você — e todos usam, internamente, exatamente o `podTemplate` que estamos aprendendo a escrever agora. **O tempo investido em dominar o `spec` de um Pod se paga em todos os capítulos seguintes.**

---

### 3.9 Services

Pods são efêmeros e seus IPs mudam a cada recriação. O **Service** é a abstração que dá um endereço estável e balanceamento sobre um conjunto dinâmico de Pods.

#### 3.9.1 Como um Service encontra seus Pods

```
  Service store-api                    Pods no namespace store-dev
  spec.selector:                       ----------------------------
    app: store-api        --casa-->    Pod A  labels: app=store-api, env=dev   [Ready]
    env: dev                           Pod B  labels: app=store-api, env=dev   [Ready]
                                       Pod C  labels: app=store-api, env=prod  (nao casa)
                          |                   Pod D  labels: app=store-api, env=dev [NotReady]
                          v
                  EndpointSlice store-api-abc12
                    endpoints:
                      - 10.244.0.7  (Pod A)   ready: true
                      - 10.244.1.5  (Pod B)   ready: true
                          |
                          v
                  kube-proxy programa iptables/IPVS em CADA no
                          |
                          v
                  ClusterIP 10.101.44.9:80  ->  10.244.0.7:8000 | 10.244.1.5:8000
```

Três observações críticas: (a) o selector do Service usa **igualdade simples** — todas as chaves listadas devem casar (E lógico), sem operadores de conjunto; (b) apenas Pods **`Ready`** entram como endpoints prontos — é aqui que a readinessProbe age; (c) o Service **não conhece** Deployments ou ReplicaSets, apenas labels de Pods.

#### 3.9.2 Os cinco tipos

**ClusterIP (padrão)** — IP virtual acessível apenas de dentro do cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
spec:
  type: ClusterIP
  selector:
    app: store-api
    env: dev
  ports:
    - name: http
      port: 80           # porta do SERVICE
      targetPort: 8000   # porta do CONTAINER (aceita nome de porta)
      protocol: TCP
```

**NodePort** — abre a mesma porta em **todos** os nós do cluster, na faixa 30000–32767. Todo NodePort implica um ClusterIP.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api-np
  namespace: store-dev
spec:
  type: NodePort
  selector:
    app: store-api
    env: dev
  ports:
    - name: http
      port: 80
      targetPort: 8000
      nodePort: 30080    # opcional; se omitido, o cluster sorteia
```

**LoadBalancer** — pede ao cloud-controller-manager um balanceador externo (NLB/ALB na AWS). É NodePort + ClusterIP + balanceador. No minikube exige `minikube tunnel`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-front-lb
  namespace: store-prod
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  selector:
    app: store-front
  ports:
    - port: 80
      targetPort: 80
```

> **Atenção — custo.** Cada Service `LoadBalancer` no EKS cria um balanceador **cobrado por hora**. Dez microsserviços expostos assim = dez balanceadores na fatura. O padrão correto em produção é **um Ingress** (um único ALB) roteando por host/path para vários Services ClusterIP. Veremos isso no capítulo de Ingress.

**ExternalName** — não tem selector nem IP; cria um registro CNAME no DNS interno. Serve para representar um serviço externo com um nome interno estável:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-db
  namespace: store-dev
spec:
  type: ExternalName
  externalName: fiap-store.abc123xyz.us-east-1.rds.amazonaws.com
```

A aplicação continua se conectando a `store-db.store-dev.svc.cluster.local`; em desenvolvimento isso aponta para um Pod PostgreSQL, em produção para o RDS. Zero alteração de código.

**Headless (`clusterIP: None`)** — não cria IP virtual nem regras de proxy. O DNS retorna **os IPs de todos os Pods** diretamente.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api-headless
  namespace: store-dev
spec:
  clusterIP: None
  selector:
    app: store-api
    env: dev
  ports:
    - name: http
      port: 8000
      targetPort: 8000
```

```bash
nslookup store-api-headless.store-dev.svc.cluster.local
```

```
Name:   store-api-headless.store-dev.svc.cluster.local
Address: 10.244.0.7
Name:   store-api-headless.store-dev.svc.cluster.local
Address: 10.244.1.5
```

Compare com um ClusterIP, que devolveria **um único** endereço virtual. Headless é essencial para **StatefulSets** (onde cada réplica precisa de identidade própria: `store-db-0.store-db.store-dev.svc.cluster.local`), para clientes que fazem o próprio balanceamento (gRPC, drivers de Cassandra/Kafka) e para descoberta de peers em clusters de aplicação.

#### 3.9.3 Endpoints e EndpointSlice

O objeto **Endpoints** (API `v1`, legado) lista os IP:porta de destino de um Service. Ele tem um problema de escala: é um único objeto que cresce sem limite, e qualquer mudança em um Pod reescreve o objeto inteiro para todos os observadores. Com 5.000 endpoints, isso derruba o desempenho do control plane.

O **EndpointSlice** (API `discovery.k8s.io/v1`, padrão desde a 1.19) resolve isso fatiando a lista em pedaços de até 100 endpoints. Um Service com 500 Pods tem 5 slices; mudar um Pod atualiza 1 slice.

```bash
kubectl get endpoints store-api -n store-dev
kubectl get endpointslices -n store-dev -l kubernetes.io/service-name=store-api
kubectl get endpointslice store-api-x7k2m -n store-dev -o yaml
```

```
NAME        ENDPOINTS                           AGE
store-api   10.244.0.7:8000,10.244.1.5:8000     4m

NAME              ADDRESSTYPE   PORTS   ENDPOINTS               AGE
store-api-x7k2m   IPv4          8000    10.244.0.7,10.244.1.5   4m
```

> **Na prática:** `kubectl get endpoints <svc>` mostrando `<none>` é **o** sintoma canônico de Service quebrado. As três causas, em ordem de frequência: (1) selector do Service não casa com os labels dos Pods; (2) os Pods existem mas nenhum está `Ready` (readinessProbe falhando); (3) `targetPort` aponta para uma porta onde não há nada escutando. Sempre comece por aqui.

#### 3.9.4 DNS interno

O **CoreDNS** roda no namespace `kube-system` e serve o domínio `cluster.local`. O formato canônico é:

```
<service>.<namespace>.svc.cluster.local
```

E o kubelet configura o `/etc/resolv.conf` de cada Pod com uma lista de busca que permite formas abreviadas:

```
search store-dev.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Portanto, de dentro de um Pod no namespace `store-dev`:

| Nome usado | Resolve? | Observação |
|---|---|---|
| `store-api` | Sim | Mesmo namespace, via `search` |
| `store-api.store-dev` | Sim | Namespace explícito |
| `store-api.store-dev.svc` | Sim | — |
| `store-api.store-dev.svc.cluster.local` | Sim | **Forma canônica — use esta em configuração** |
| `store-api` a partir de `store-prod` | **Não** | Cross-namespace exige nome qualificado |

Para Services com portas nomeadas há também registros SRV: `_http._tcp.store-api.store-dev.svc.cluster.local`.

> **Nota — `ndots:5`.** Essa opção significa "nomes com menos de 5 pontos passam pela lista de `search` antes de serem tentados como absolutos". Consultar `api.github.com` (2 pontos) dispara 3 consultas falhas (uma por domínio da lista de `search`) antes da certa. Em aplicações com muito tráfego externo isso vira gargalo de DNS; a correção é usar o nome com ponto final (`api.github.com.`) ou ajustar `dnsConfig` no Pod.

#### 3.9.5 Descoberta por variáveis de ambiente

Existe um mecanismo mais antigo, herdado do Docker links: para cada Service existente **no momento em que o Pod é criado**, o kubelet injeta variáveis:

```bash
kubectl exec store-api -n store-dev -- env | grep STORE_API
```

```
STORE_API_SERVICE_HOST=10.101.44.9
STORE_API_SERVICE_PORT=80
STORE_API_PORT=tcp://10.101.44.9:80
STORE_API_PORT_80_TCP_ADDR=10.101.44.9
```

Duas limitações fatais: só enxerga Services criados **antes** do Pod (ordem importa) e só do **mesmo namespace**. **Use DNS.** As variáveis existem por compatibilidade; podem inclusive ser desabilitadas com `spec.enableServiceLinks: false`, o que é recomendável em namespaces com muitos Services.

#### 3.9.6 sessionAffinity

Por padrão o Service distribui em round-robin (aproximado). Para "grudar" um cliente em um Pod:

```yaml
spec:
  sessionAffinity: ClientIP
  sessionAffinityConfig:
    clientIP:
      timeoutSeconds: 10800      # 3 horas
```

A afinidade é por **IP de origem**, o que a torna pouco confiável quando há NAT ou proxy no caminho (todos os clientes chegam com o mesmo IP). A solução arquiteturalmente correta é a aplicação ser stateless e a sessão viver em Redis ou em um JWT.

#### 3.9.7 `apply` x `create` x `replace`

| Comando | Objeto não existe | Objeto existe | Estratégia |
|---|---|---|---|
| `kubectl create -f` | Cria | **Erro** `AlreadyExists` | Imperativo |
| `kubectl replace -f` | **Erro** `NotFound` | Substitui **inteiro** (o que faltar é apagado) | Imperativo |
| `kubectl apply -f` | Cria | Faz merge das diferenças | **Declarativo** |

O `apply` é inteligente porque guarda uma cópia do último manifesto aplicado na annotation `kubectl.kubernetes.io/last-applied-configuration`. Ao reaplicar, ele calcula um *three-way merge* entre: (1) o último aplicado, (2) o que está no cluster agora e (3) o que você está enviando. Isso permite **remover** um campo que você apagou do arquivo, sem apagar campos que outros controladores adicionaram.

```bash
kubectl apply -f pod.yaml
kubectl get pod store-api -o jsonpath='{.metadata.annotations.kubectl\.kubernetes\.io/last-applied-configuration}' | jq .
kubectl diff -f pod.yaml       # mostra o que MUDARIA, sem aplicar
```

> **Nota:** o **Server-Side Apply** (`kubectl apply --server-side`, estável desde a 1.22) substitui esse mecanismo por rastreamento de propriedade de campos no próprio servidor (`metadata.managedFields`), eliminando a annotation e resolvendo conflitos entre múltiplos autores de forma explícita. É o que Argo CD e Flux usam. Prefira-o em pipelines.

#### 3.9.8 Organização de arquivos YAML

```
fiap-store/
├── base/
│   ├── kustomization.yaml
│   ├── store-api-deployment.yaml
│   ├── store-api-service.yaml
│   ├── store-api-configmap.yaml
│   └── namespace.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── replicas-patch.yaml
    └── prod/
        ├── kustomization.yaml
        └── replicas-patch.yaml
```

Regras que economizam sofrimento: **um objeto por arquivo** (exceto o par Deployment+Service, que costuma andar junto); nome do arquivo = `<app>-<kind>.yaml`; namespace **sempre explícito** no manifesto (evita aplicar no lugar errado); labels padronizadas em todos os objetos; e tudo versionado no Git — o cluster é reflexo do repositório, nunca o contrário.

---

### 3.10 Laboratório guiado (feito em sala)

**Objetivo:** escrever do zero o Pod e os Services da `store-api`, comprovar a resolução DNS a partir de um Pod cliente e diagnosticar um selector quebrado propositalmente.

**Passo 1 — Preparar o ambiente.**

```bash
minikube start -p fiap-store --driver=docker --kubernetes-version=v1.34.1 --cpus=2 --memory=4096
minikube image load store-api:1.0 -p fiap-store
kubectl create namespace store-dev
kubectl config set-context --current --namespace=store-dev
```

**Passo 2 — Escrever `pod.yaml`.**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
    env: dev
spec:
  containers:
    - name: store-api
      image: store-api:1.0
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 8000
      env:
        - name: APP_ENV
          value: "dev"
        - name: DB_HOST
          value: "store-db.store-dev.svc.cluster.local"
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
      readinessProbe:
        httpGet:
          path: /ready
          port: http
        initialDelaySeconds: 3
        periodSeconds: 5
      livenessProbe:
        httpGet:
          path: /health
          port: http
        initialDelaySeconds: 10
        periodSeconds: 10
```

```bash
kubectl apply --dry-run=server -f pod.yaml     # valida sem persistir
kubectl apply -f pod.yaml
kubectl get pod store-api -o wide
```

```
pod/store-api created (server dry run)
pod/store-api created
NAME        READY   STATUS    RESTARTS   AGE   IP           NODE
store-api   1/1     Running   0          12s   10.244.0.9   fiap-store
```

Confirme a QoS e as probes:

```bash
kubectl get pod store-api -o jsonpath='{.status.qosClass}{"\n"}'
kubectl describe pod store-api | grep -A2 -E 'Liveness|Readiness'
```

```
Burstable
    Liveness:   http-get http://:http/health delay=10s timeout=1s period=10s #success=1 #failure=3
    Readiness:  http-get http://:http/ready delay=3s timeout=1s period=5s #success=1 #failure=3
```

**Passo 3 — Escrever `service-clusterip.yaml`.**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
spec:
  type: ClusterIP
  selector:
    app: store-api
    env: dev
  ports:
    - name: http
      port: 80
      targetPort: http
      protocol: TCP
```

```bash
kubectl apply -f service-clusterip.yaml
kubectl get svc,endpoints store-api
```

```
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
service/store-api   ClusterIP   10.101.44.9     <none>        80/TCP    5s

NAME                  ENDPOINTS         AGE
endpoints/store-api   10.244.0.9:8000   5s
```

O endpoint apareceu: selector casou e o Pod está `Ready`.

**Passo 4 — Testar DNS de dentro do cluster.**

```bash
kubectl run curl-test --image=curlimages/curl:8.11.1 -it --rm --restart=Never -- sh
```

Dentro do Pod:

```sh
cat /etc/resolv.conf
curl -s http://store-api/health
curl -s http://store-api.store-dev/health
curl -s http://store-api.store-dev.svc.cluster.local/health
curl -s http://store-api.store-dev.svc.cluster.local/info
env | grep STORE_API
exit
```

```
search store-dev.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5

{"status":"ok","host":"store-api"}
{"status":"ok","host":"store-api"}
{"status":"ok","host":"store-api"}
{"env":"dev","db_host":"store-db.store-dev.svc.cluster.local","pod":"store-api"}
STORE_API_SERVICE_HOST=10.101.44.9
STORE_API_SERVICE_PORT=80
```

Para inspeção de DNS mais detalhada, use busybox (que traz `nslookup`):

```bash
kubectl run dns-test --image=busybox:1.36 -it --rm --restart=Never -- \
  nslookup store-api.store-dev.svc.cluster.local
```

```
Server:    10.96.0.10
Address:   10.96.0.10:53

Name:      store-api.store-dev.svc.cluster.local
Address:   10.101.44.9
```

**Passo 5 — Quebrar o selector de propósito.**

Edite `service-clusterip.yaml` trocando `app: store-api` por `app: store-apis` (com "s" a mais) e reaplique:

```bash
sed -i 's/app: store-api$/app: store-apis/' service-clusterip.yaml
kubectl apply -f service-clusterip.yaml
kubectl get endpoints store-api
```

```
NAME        ENDPOINTS   AGE
store-api   <none>      3m
```

Agora observe o efeito de fora:

```bash
kubectl run curl-test --image=curlimages/curl:8.11.1 -it --rm --restart=Never -- \
  curl -s --max-time 5 http://store-api/health
```

```
command terminated with exit code 28
```

Diagnóstico completo:

```bash
kubectl get pods --show-labels
kubectl describe svc store-api | grep -E 'Selector|Endpoints'
kubectl get pods -l app=store-apis
```

```
NAME        READY   STATUS    LABELS
store-api   1/1     Running   app=store-api,env=dev,tier=backend

Selector:          app=store-apis,env=dev
Endpoints:         <none>

No resources found in store-dev namespace.
```

**Este é o exercício mais valioso do capítulo.** O Service existe, tem ClusterIP, o Pod está `Running` e saudável — e nada funciona, porque o selector não casa. Nenhum evento de erro é gerado: o Kubernetes considera perfeitamente legítimo um Service sem endpoints. Grave o comando `kubectl get endpoints <svc>` como primeiro reflexo diante de "meu serviço não responde".

Corrija e confirme:

```bash
sed -i 's/app: store-apis/app: store-api/' service-clusterip.yaml
kubectl apply -f service-clusterip.yaml
kubectl get endpoints store-api
```

```
NAME        ENDPOINTS         AGE
store-api   10.244.0.9:8000   4m
```

**Passo 6 — Quebrar a readiness e observar o mesmo sintoma por outra causa.**

```bash
kubectl patch pod store-api --type=json \
  -p='[{"op":"replace","path":"/spec/containers/0/readinessProbe/httpGet/path","value":"/nao-existe"}]'
kubectl get pod store-api
kubectl get endpoints store-api
```

```
NAME        READY   STATUS    RESTARTS   AGE
store-api   0/1     Running   0          6m

NAME        ENDPOINTS   AGE
store-api   <none>      6m
```

Repare: `Running` mas `0/1` — o Pod está de pé e **não está pronto**, e por isso saiu dos Endpoints. Este é o comportamento correto e desejado do readiness. (O `patch` de probe funciona em Pod avulso; em um Deployment, altere o template.)

**Passo 7 — Limpeza.**

```bash
kubectl delete -f pod.yaml -f service-clusterip.yaml
# ou, de uma vez:
kubectl delete namespace store-dev
```

---

### 3.11 Ponte com Serverless

As probes que acabamos de configurar são, em essência, **a versão explícita e configurável de algo que o AWS Lambda faz por você, de forma opaca**.

No Lambda, você não escreve `livenessProbe`. A plataforma detecta que uma invocação travou (estourando o timeout configurado, de até 15 minutos), encerra o ambiente de execução e cria um novo na próxima invocação. Não existe `readinessProbe` porque não existe balanceador para o qual você precise anunciar prontidão — o serviço só roteia para um ambiente já inicializado. A fase de inicialização (`INIT`) do Lambda, aquela em que o runtime carrega o pacote e executa o código fora do handler, é conceitualmente o análogo direto da **startupProbe**: um período em que a plataforma sabe que a função ainda não está apta a atender.

A troca é clara: no Kubernetes você **decide** o que significa "pronto" e "vivo" para a sua aplicação, com o custo de ter de acertar valores de `periodSeconds`, `failureThreshold` e a distinção entre `/health` e `/ready`. No Lambda a plataforma decide por você, e você não erra — mas também não pode expressar "estou vivo, porém temporariamente incapaz de atender porque o pool de conexões está esgotado", que é exatamente o que uma readinessProbe expressa.

O segundo ponto da ponte é mais estrutural: **um Pod cru não faz scale-to-zero, e nunca fará.** Um Pod é uma instância única e concreta — ele existe ou não existe. Não há "zero réplicas de um Pod"; há Pod deletado. Escalar a zero e voltar exige três coisas que o Pod sozinho não tem:

1. **Um controlador** que possa criar e destruir réplicas conforme demanda (Deployment, no mínimo).
2. **Um componente que segure a requisição** enquanto não há nenhuma réplica de pé. No Knative, esse componente é o **Activator**: com zero Pods, o tráfego é desviado para ele, que enfileira a requisição, sinaliza ao autoscaler para criar um Pod, espera a readinessProbe passar e só então encaminha. É por isso que a readinessProbe é literalmente parte do caminho crítico do cold start no Knative.
3. **Um sinal de demanda.** No Knative Serving, é a concorrência de requisições HTTP medida pelo *queue-proxy* (um sidecar injetado em cada Pod). No KEDA, é uma métrica externa — profundidade de uma fila SQS, lag de um consumer group Kafka, resultado de uma query no Prometheus.

Na FIAP Store, isso se traduz assim: a `store-api` com Knative teria um cold start de 2 a 5 segundos quando ociosa (contra 200 ms a 2 s de um Lambda Python) — pior tempo de resposta, mas com portabilidade total entre nuvens e sem os limites de 15 minutos e de tamanho de pacote. A `store-worker`, que processa pedidos em batch, é o caso ideal para KEDA: fica em **zero réplica** a maior parte do dia, e sobe conforme a fila cresce. E note: nada disso funciona sem uma readinessProbe correta. **Probes bem escritas não são detalhe de configuração — são a interface entre a sua aplicação e todo o mecanismo de escalonamento da plataforma.**

---

### 3.12 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `error converting YAML to JSON: yaml: line N: did not find expected key` | Indentação inconsistente ou tab | Abrir na linha N; `cat -A arquivo.yaml \| grep -n '\^I'` | Usar 2 espaços; configurar `.editorconfig` |
| `found character '\t' that cannot start any token` | Tab no arquivo | `grep -P '\t' pod.yaml` | Substituir tabs por espaços |
| `error validating data: unknown field "livenessprobe"` | Campo com capitalização errada (YAML é case-sensitive, camelCase) | `kubectl explain pod.spec.containers` | `livenessProbe` |
| `no matches for kind "Deployment" in version "extensions/v1beta1"` | API removida | `kubectl api-resources \| grep -i deployment` | `apps/v1` |
| Service com `ENDPOINTS <none>` e Pods `Running 1/1` | Selector do Service não casa com labels dos Pods | `kubectl get pods --show-labels` vs `kubectl describe svc X \| grep Selector` | Alinhar as chaves/valores exatamente |
| Service com `ENDPOINTS <none>` e Pods `Running 0/1` | Readiness probe falhando | `kubectl describe pod` (Events: `Readiness probe failed`) | Corrigir path/porta da probe ou o bug da aplicação |
| Endpoint existe, mas `curl` dá timeout | `targetPort` aponta para porta onde nada escuta | `kubectl exec <pod> -- ss -tlnp`; `kubectl port-forward pod/X 8000:8000` | Ajustar `targetPort` para a porta real do processo |
| Aplicação escuta em `127.0.0.1` e o Service não funciona | Processo ligado a loopback, não a `0.0.0.0` | `kubectl exec <pod> -- ss -tln` mostra `127.0.0.1:8000` | Bind em `0.0.0.0` (`uvicorn --host 0.0.0.0`) |
| `nslookup` falha para Service de outro namespace | Nome curto só resolve no namespace do Pod | `cat /etc/resolv.conf` dentro do Pod | Usar `svc.namespace.svc.cluster.local` |
| Pod `Running` mas `0/1 READY` para sempre | Readiness nunca passa; ou porta nomeada inexistente | `kubectl describe pod` | Conferir `port:` da probe contra `ports[].name` |
| Container `Terminated` com `reason: OOMKilled`, exit 137 | `limits.memory` insuficiente | `kubectl describe pod \| grep -A3 'Last State'` | Aumentar o limite ou corrigir vazamento |
| `CreateContainerConfigError` | ConfigMap/Secret referenciado não existe | `kubectl describe pod` (Events) | Criar o ConfigMap/Secret ou marcar `optional: true` |
| `kubectl replace` falha com `NotFound` | Objeto não existe ainda | `kubectl get <kind> <name>` | Usar `kubectl apply` |
| Alteração no ConfigMap não chega na aplicação | Variáveis de ambiente são resolvidas na criação do container | `kubectl exec <pod> -- env` | `kubectl rollout restart deploy/X` ou montar como volume |
| `pod "store-api" is forbidden: violates PodSecurity "restricted:latest"` | Pod Security Admission bloqueando | Ler a mensagem: ela nomeia o campo faltante | Adicionar `runAsNonRoot: true`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `seccompProfile` |

---

### 3.13 Resumo do capítulo

Escrever YAML para o Kubernetes é escrever um contrato: `apiVersion` e `kind` dizem que tipo de contrato é, `metadata` dá identidade e as labels que tornam o objeto selecionável, e `spec` declara o estado desejado que os controladores se comprometem a manter. O Pod é a peça central desse vocabulário — não porque você vá criá-lo diretamente em produção (não vai), mas porque o seu `spec` é literalmente o template embutido em todo Deployment, StatefulSet, DaemonSet e Job que você escreverá daqui em diante. Dentro dele moram as decisões que mais impactam a confiabilidade: as três probes (com a distinção crítica entre liveness, que mata, e readiness, que apenas retira do balanceamento), os requests e limits que definem a classe de QoS e a ordem de despejo, e a composição multi-container com initContainers e sidecars sobre volumes compartilhados. Do outro lado, o Service transforma um conjunto instável de IPs em um nome estável — e faz isso exclusivamente por meio de labels, o que torna o par selector/labels a articulação mais frágil e mais frequentemente quebrada de todo o sistema.

Pontos-chave:

1. YAML tem armadilhas reais: indentação com espaços, `NO`/`yes`/`on` viram booleanos, `1.10` vira float. Aspas resolvem quase tudo; `--dry-run=server` pega o resto.
2. Um Pod é um grupo de containers que compartilham IP, IPC e volumes, agendados juntos. Containers do mesmo Pod falam por `localhost` e não podem repetir portas.
3. `Running` não significa saudável. Quem decide se o Pod recebe tráfego é a **readinessProbe**, através da condição `Ready`.
4. **Liveness mata, readiness desconecta.** Liveness deve testar apenas o próprio processo; readiness pode testar dependências. Trocar isso transforma queda de banco em CrashLoop de toda a frota.
5. Sem `requests` o Pod é BestEffort e é o primeiro a ser despejado. Memória estourada = OOMKilled (exit 137); CPU estourada = throttling silencioso.
6. Sidecar moderno = `initContainer` com `restartPolicy: Always` (estável na 1.33): ordem garantida na subida e o Pod consegue terminar em Jobs.
7. Service encontra Pods **só** por labels, e só inclui Pods `Ready`. `kubectl get endpoints <svc>` retornando `<none>` é o primeiro comando de qualquer diagnóstico de rede.
8. DNS canônico: `<service>.<namespace>.svc.cluster.local`. Nome curto só funciona dentro do mesmo namespace. Headless (`clusterIP: None`) devolve os IPs dos Pods, não um IP virtual.

---

### 3.14 Exercícios (para casa)

#### Exercício 3.1 — Manifesto de Pod completo da store-api (fixação guiada) · 35 min

- **Objetivo:** escrever, do zero e sem gerador, um manifesto de Pod com probes, recursos e contexto de segurança, e comprovar cada elemento no cluster.
- **Enunciado:** Escreva `pod-store-api.yaml` **manualmente** (proibido usar `--dry-run` como gerador; use `kubectl explain` para consultar campos) contendo um Pod chamado `store-api` no namespace `store-dev`, com: labels `app=store-api`, `tier=backend`, `env=dev`; a imagem `store-api:1.0` com `imagePullPolicy: IfNotPresent`; a porta 8000 **nomeada** como `http`; as variáveis `APP_ENV=dev`, `DB_HOST=store-db.store-dev.svc.cluster.local` e, via **Downward API**, `POD_NAME` (a partir de `metadata.name`) e `NODE_NAME` (a partir de `spec.nodeName`); `requests` de 100m/128Mi e `limits` de 500m/256Mi; as **três** probes — `startupProbe` em `/health` com `failureThreshold: 30` e `periodSeconds: 2`, `readinessProbe` em `/ready` e `livenessProbe` em `/health`, todas referenciando a porta **pelo nome** `http`; e um `securityContext` com `runAsNonRoot: true`, `runAsUser: 10001`, `allowPrivilegeEscalation: false` e `capabilities.drop: ["ALL"]`. Depois: (a) valide com `kubectl apply --dry-run=server`; (b) aplique; (c) prove que o Pod está `1/1 Running`; (d) descubra e **justifique** a classe de QoS resultante; (e) mostre as três probes na saída do `describe`; (f) prove que `POD_NAME` e `NODE_NAME` chegaram corretos dentro do container; (g) responda em 3 linhas: com esses parâmetros, quanto tempo no máximo o Kubernetes leva para matar o container se a liveness começar a falhar?
- **Entregável:** `pod-store-api.yaml` e `ex3-1.md` com os sete itens documentados (comandos + saídas).
- **Critério de aceite:** `kubectl apply --dry-run=server` passa sem erro; o Pod fica `1/1 Running`; a QoS é `Burstable` e o aluno explica por que não é `Guaranteed`; `kubectl exec store-api -- env | grep -E 'POD_NAME|NODE_NAME'` mostra o nome real do Pod e do nó; o cálculo do item (g) é `initialDelaySeconds + failureThreshold × periodSeconds`.
- <details><summary>Dica</summary><code>kubectl explain pod.spec.containers.env.valueFrom.fieldRef</code> mostra a sintaxe da Downward API. Para a QoS: <code>Guaranteed</code> exige <code>requests == limits</code> em <b>todos</b> os recursos de <b>todos</b> os containers.</details>

#### Exercício 3.2 — Três Services e a prova de DNS (aplicação) · 45 min

- **Objetivo:** criar e diferenciar na prática ClusterIP, NodePort e headless, comprovando o comportamento distinto de resolução DNS de cada um.
- **Enunciado:** Partindo do Pod do Exercício 3.1, crie **mais dois** Pods (`store-api-2` e `store-api-3`) com os mesmos labels e nomes distintos — assim teremos 3 endpoints. Em seguida, escreva três arquivos: `svc-clusterip.yaml` (Service `store-api`, ClusterIP, `port: 80`, `targetPort: http`), `svc-nodeport.yaml` (Service `store-api-np`, NodePort, `nodePort: 30081`) e `svc-headless.yaml` (Service `store-api-hl`, `clusterIP: None`, `port: 8000`). Aplique os três e produza as seguintes evidências: **(a)** `kubectl get svc,endpoints` mostrando os três Services e os **3 endpoints** em cada um; **(b)** `kubectl get endpointslices -l kubernetes.io/service-name=store-api -o wide` e uma explicação de 2 linhas sobre a diferença entre Endpoints e EndpointSlice; **(c)** de dentro de um Pod busybox, execute `nslookup` para os **três** Services usando a forma canônica e **compare as respostas** — o ClusterIP e o NodePort devolvem 1 endereço, o headless devolve 3; explique por quê; **(d)** de dentro de um Pod `curlimages/curl`, faça 12 requisições a `http://store-api/health` e mostre a distribuição dos hostnames retornados; **(e)** acesse o NodePort **de fora** do cluster via `minikube service store-api-np --url`; **(f)** crie o namespace `store-prod` e, de um Pod nele, tente `curl http://store-api/health` (nome curto) — mostre que falha — e depois `curl http://store-api.store-dev.svc.cluster.local/health` — mostre que funciona — explicando o papel do `search` no `/etc/resolv.conf`; **(g)** adicione `sessionAffinity: ClientIP` ao ClusterIP, reaplique, repita as 12 requisições do item (d) e mostre que agora todas vão para o mesmo Pod.
- **Entregável:** os três arquivos YAML mais `ex3-2.md` com os sete itens (comandos + saídas + explicações).
- **Critério de aceite:** os três Services listam 3 endpoints cada; o `nslookup` do headless retorna 3 endereços (`Address:` repetido) enquanto os outros retornam 1; as 12 requisições sem afinidade atingem ao menos 2 hostnames distintos e com `sessionAffinity: ClientIP` atingem exatamente 1; o item (f) demonstra a falha do nome curto cross-namespace e o sucesso do FQDN.
- <details><summary>Dica</summary>Use <code>kubectl run tester --image=busybox:1.36 -it --rm --restart=Never -- sh</code> para ter <code>nslookup</code>. No item (d), <code>for i in $(seq 12); do wget -qO- http://store-api/health; echo; done | sort | uniq -c</code> resume a distribuição.</details>

#### Exercício 3.3 — Consertar o manifesto quebrado (análise/depuração) · 40 min

- **Objetivo:** localizar e corrigir cinco defeitos distintos em manifestos, classificando cada um por tipo de erro.
- **Enunciado:** Salve os dois manifestos abaixo como `quebrado.yaml`. Eles contêm **cinco defeitos**. Para cada um, você deve identificar: (1) a linha; (2) o tipo do defeito (sintaxe YAML / API inválida / valor incoerente com outro objeto / valor incoerente com a aplicação); (3) a mensagem de erro ou o sintoma que ele produz; (4) o comando que revela o problema; (5) a correção.

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: store-api
    namespace: store-dev
    labels:
      app: store-api
       tier: backend
      env: dev
  spec:
    containers:
      - name: store-api
        image: store-api:1.0
        ports:
          - name: http
            containerPort: 8000
        livenessprobe:
          httpGet:
            path: /health
            port: 8000
  ---
  apiVersion: extensions/v1beta1
  kind: Service
  metadata:
    name: store-api
    namespace: store-dev
  spec:
    type: ClusterIP
    selector:
      app: store-api
      env: prod
    ports:
      - port: 80
        targetPort: 9000
  ```

  Trabalhe de forma **incremental**: corrija um defeito, tente aplicar, leia a nova mensagem de erro, corrija o próximo. Documente essa progressão — a sequência de mensagens de erro é parte do entregável, porque ela mostra a ordem em que o Kubernetes valida as coisas (parser YAML → esquema da API → admission → runtime → conectividade). Ao final, entregue `corrigido.yaml` funcionando, com o Service devolvendo um endpoint e um `curl` bem-sucedido a partir de um Pod cliente. **Pergunta de fechamento (obrigatória, 5 linhas):** dois dos cinco defeitos **não** geram nenhum erro no `kubectl apply` — o objeto é criado com sucesso e o problema só aparece em tempo de execução. Quais são, e o que isso ensina sobre confiar no `apply` como validação?
- **Entregável:** `ex3-3.md` com a tabela dos 5 defeitos (colunas: linha, tipo, sintoma, comando revelador, correção), a progressão das mensagens de erro, e o arquivo `corrigido.yaml`.
- **Critério de aceite:** os cinco defeitos identificados são: (i) indentação de `tier: backend` desalinhada; (ii) `livenessprobe` em minúsculas (campo desconhecido); (iii) `apiVersion: extensions/v1beta1` para um Service (deve ser `v1`); (iv) selector com `env: prod` divergente do label `env: dev` do Pod (Endpoints vazio, **sem erro no apply**); (v) `targetPort: 9000` sem processo escutando (timeout, **sem erro no apply**). A resposta de fechamento identifica corretamente (iv) e (v).
- <details><summary>Dica</summary>Aplique com <code>kubectl apply --dry-run=server -f quebrado.yaml</code> para pegar (i), (ii) e (iii) sem sujar o cluster. Para (iv) e (v), o único caminho é <code>kubectl get endpoints</code> e um <code>curl</code> de dentro do cluster — nenhuma validação estática pega esses dois.</details>

#### Exercício 3.4 — Desafio: Pod multi-container com initContainer, sidecar e emptyDir (desafio integrador) · 55 min

- **Objetivo:** compor um Pod com três containers de papéis distintos compartilhando um volume, comprovando a ordem de execução e o compartilhamento de rede e arquivos.
- **Enunciado:** Escreva `pod-multi.yaml` com um Pod `store-api-full` no namespace `store-dev` contendo: **(1) um initContainer clássico** chamado `preparar-logs`, imagem `busybox:1.36`, que cria o arquivo `/var/log/app/access.log`, escreve nele uma linha de marcação com a data e sai com código 0; **(2) um sidecar no modelo moderno** — declarado em `initContainers` com `restartPolicy: Always` — chamado `log-shipper`, imagem `busybox:1.36`, que executa `tail -F /var/log/app/access.log` indefinidamente, simulando um coletor; **(3) o container principal** `store-api`, imagem `store-api:1.0`, com readiness e liveness probes, requests/limits e um `command`/`args` que faça o uvicorn escrever o log de acesso no arquivo compartilhado (dica: `--access-log` do uvicorn escreve em stdout; use `sh -c 'uvicorn ... 2>&1 | tee -a /var/log/app/access.log'`); **(4) um volume `emptyDir`** chamado `logs`, montado em `/var/log/app` nos **três** containers. Depois, produza as evidências: **(a)** `kubectl get pod store-api-full` mostrando a coluna READY com a contagem correta de containers e explique **por que** o sidecar conta como container pronto; **(b)** `kubectl describe pod` evidenciando que `preparar-logs` está `Terminated (Completed)` enquanto `log-shipper` está `Running`; **(c)** `kubectl logs store-api-full -c log-shipper` mostrando as linhas de log geradas pelo container principal — prova de que o `emptyDir` é compartilhado; **(d)** prova de que os containers compartilham a **rede**: `kubectl exec store-api-full -c log-shipper -- wget -qO- http://localhost:8000/health` deve funcionar, e você deve explicar em 2 linhas por que `localhost` funciona entre containers de um mesmo Pod; **(e)** gere tráfego com um Service e um Pod curl, e mostre as novas linhas aparecendo no `log-shipper`; **(f)** use `kubectl debug` para injetar um **container efêmero** com a imagem `nicolaka/netshoot` no Pod em execução, e de dentro dele rode `ss -tlnp` e `curl localhost:8000/produtos`, mostrando que ele enxerga o mesmo namespace de rede. **Pergunta de fechamento (obrigatória, 5 a 8 linhas):** se você transformasse este Pod em um `Job`, o que aconteceria com o sidecar se ele fosse declarado da forma **antiga** (em `spec.containers`)? E por que o sidecar como `initContainer` com `restartPolicy: Always` resolve isso?
- **Entregável:** `pod-multi.yaml`, `svc-multi.yaml` e `ex3-4.md` com os seis itens de evidência e a resposta de fechamento.
- **Critério de aceite:** o Pod fica `2/2 Running` (container principal + sidecar; o initContainer clássico não conta); `kubectl logs -c log-shipper` exibe linhas escritas pelo container `store-api`, comprovando o `emptyDir` compartilhado; o `wget` para `localhost:8000` de dentro do sidecar retorna `{"status":"ok",...}`; o container efêmero injetado com `kubectl debug` consegue ver a porta 8000 em `LISTEN`; a resposta de fechamento explica que o Job nunca chegaria à fase `Succeeded` com o sidecar antigo, porque o Pod só completa quando **todos** os containers de `spec.containers` terminam.
- <details><summary>Dica</summary>Para (f): <code>kubectl debug -it store-api-full --image=nicolaka/netshoot --target=store-api -- bash</code> — a flag <code>--target</code> compartilha o namespace de processos com o container indicado. Se o sidecar não aparecer em <code>kubectl get pod</code> na contagem READY, confirme que ele tem <code>restartPolicy: Always</code> <b>dentro</b> do item de <code>initContainers</code> e que o cluster é 1.29+.</details>

---

### 3.15 Leitura recomendada

1. **[DOC]** Kubernetes — *Workloads: Pods* e *Pod Lifecycle*. Disponível em: https://kubernetes.io/docs/concepts/workloads/pods/ — ler integralmente "Pod Lifecycle", "Init Containers" e "Sidecar Containers".
2. **[DOC]** Kubernetes — *Service* e *DNS for Services and Pods*. Disponível em: https://kubernetes.io/docs/concepts/services-networking/service/ e https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/
3. **[DOC]** Kubernetes — *Configure Liveness, Readiness and Startup Probes*. Disponível em: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
4. **[LIVRO]** LUKSA, M. *Kubernetes in Action*. 2. ed. Manning, 2023 — capítulos 3 (Pods), 5 (Services) e 6 (probes e ciclo de vida).
5. **[LIVRO]** BURNS, B. *Designing Distributed Systems*. 2. ed. O'Reilly, 2024 — capítulos 2 e 3, sobre os padrões sidecar, ambassador e adapter.
6. **[ARTIGO]** Kubernetes Blog. *Sidecar Containers graduate to Stable in v1.33*. Disponível em: https://kubernetes.io/blog/ — histórico e motivação do sidecar nativo.

---

## Capítulo 4 — Replicação, Distribuição e Alta Disponibilidade

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 3 (arquitetura do Kubernetes, minikube funcionando, Pods e Services escritos em YAML), Docker do 1º semestre, cluster minikube com pelo menos 3 nós disponíveis para os exercícios · **Artefatos:** `store-api-deployment.yaml`, `store-api-hpa.yaml`, `store-api-pdb.yaml`, `carga.yaml`, imagens `store-api:1.0` e `store-api:2.0`

Até aqui trabalhamos com Pods individuais. Isso é ótimo para aprender, mas péssimo para produzir: um Pod é um objeto mortal, sem qualquer promessa de ressurreição. Se o nó que o hospeda morrer, o Pod morre junto e ninguém o traz de volta. Neste capítulo damos o salto conceitual mais importante do curso — deixamos de descrever *instâncias* e passamos a descrever *populações*. Vamos dizer ao Kubernetes "quero três cópias saudáveis deste serviço, sempre", e deixar que o cluster resolva o resto: onde colocá-las, como substituí-las, como atualizá-las sem derrubar o site e como multiplicá-las quando a Black Friday chegar.

---

### 4.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** o papel do ReplicaSet no modelo de controladores do Kubernetes e justificar por que ele raramente é escrito à mão.
2. **Construir** manifestos de `Deployment` completos, dominando `spec.replicas`, `selector`, `template`, `strategy`, `minReadySeconds`, `revisionHistoryLimit` e `progressDeadlineSeconds`.
3. **Aplicar** o ciclo de vida de um rollout — `kubectl rollout status`, `history`, `pause`, `resume` e `undo` — e explicar por que uma mudança de imagem gera uma nova ReplicaSet.
4. **Analisar** as quatro estratégias clássicas de release (recreate, rolling, blue-green, canary) e implementá-las apenas com objetos nativos do Kubernetes.
5. **Aplicar** escalonamento automático horizontal com `HorizontalPodAutoscaler` (`autoscaling/v2`), incluindo `behavior` e janelas de estabilização, e situar VPA, Cluster Autoscaler e Karpenter no mesmo mapa.
6. **Construir** regras de distribuição de carga (`nodeSelector`, `affinity`, `topologySpreadConstraints`, taints/tolerations) e proteger a disponibilidade durante manutenção com `PodDisruptionBudget`.

---

### 4.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: por que um Pod solto não serve para produção. Revisão relâmpago do cap. 3 |
| 2 | 15 min | ReplicaSet, reconciliação e o padrão *controller loop* |
| 3 | 25 min | Deployment em profundidade: campos do spec, estratégias, rollout e revisões |
| 4 | 10 min | **Intervalo** |
| 5 | 15 min | Estratégias de release: recreate, rolling, blue-green, canary |
| 6 | 15 min | Escalonamento: HPA (autoscaling/v2), metrics-server, VPA, Cluster Autoscaler/Karpenter |
| 7 | 10 min | Distribuição e resiliência: affinity, topology spread, taints, PDB, drain, HA do control plane |
| 8 | 15 min | Laboratório guiado (4.10) — rollout, rollback e HPA sob carga |
| 9 | 5 min | Ponte com Serverless, fechamento e apresentação dos exercícios |

---

### 4.3 O problema: da instância à população

No capítulo 3 aplicamos um Pod da `store-api` e o vimos rodando. Vamos repetir o experimento mental com números de produção. A FIAP Store recebe 400 requisições por segundo em dia normal. Um Pod da `store-api` aguenta, digamos, 150 req/s antes de a latência estourar. Precisamos de pelo menos três Pods — e de mais alguns na promoção de sexta-feira.

Se criarmos três Pods à mão, com nomes `store-api-1`, `store-api-2` e `store-api-3`, herdamos três problemas:

- **Não há reposição.** Se o `kubelet` do nó 2 parar de reportar, o Pod dele será marcado como perdido e simplesmente sumirá. Ficamos com dois.
- **Não há atualização coordenada.** Subir a versão 2.0 significa apagar e recriar Pods um a um, na mão, torcendo para não derrubar todos ao mesmo tempo.
- **Não há elasticidade.** Escalar para dez significa escrever mais sete arquivos.

O Kubernetes resolve isso com **controladores**: processos que rodam dentro do `kube-controller-manager` e executam, sem parar, o mesmo laço:

```
loop:
    desejado  <- ler o spec do objeto na API
    observado <- listar os objetos reais que correspondem ao selector
    se observado != desejado:
        agir para aproximar observado de desejado
    dormir / esperar evento no watch
```

Esse é o **laço de reconciliação**, e ele é a ideia central do Kubernetes. Você não dá ordens ("crie um Pod"); você declara um estado ("existam três Pods com estas características") e um controlador se encarrega de tornar isso verdade, agora e para sempre. Entender esse ponto é entender por que `kubectl delete pod` em um Pod gerenciado parece "não funcionar": o Pod some, o controlador percebe a divergência em milissegundos e cria outro.

> **Nota:** o laço de reconciliação também explica por que o Kubernetes é resiliente a falhas do próprio control plane. Se o controller-manager cair por cinco minutos, nada é perdido — quando ele voltar, relê o estado desejado, compara com o observado e conserta a diferença. Não há fila de comandos para se perder.

---

### 4.4 ReplicaSet: o controlador de população

O `ReplicaSet` é o controlador mais simples desse tipo. Seu contrato é literalmente: "mantenha N Pods que casem com este `selector`, criando-os a partir deste `template`".

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: store-api-rs
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
spec:
  replicas: 3
  selector:
    matchLabels:
      app: store-api
      tier: backend
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
    spec:
      containers:
        - name: store-api
          image: docker.io/seuusuario/store-api:1.0
          ports:
            - containerPort: 8000
          env:
            - name: APP_ENV
              value: "dev"
          readinessProbe:
            httpGet:
              path: /ready
              port: 8000
            initialDelaySeconds: 3
            periodSeconds: 5
```

Três detalhes merecem atenção:

1. **O `selector` é a alma do objeto.** O ReplicaSet não "lembra" quais Pods criou consultando uma lista interna; ele pergunta à API quais Pods têm as labels do selector. Se você criar manualmente um Pod solto com `app: store-api, tier: backend` no mesmo namespace, o ReplicaSet vai **adotá-lo** e, para compensar, apagar um dos seus. Esse mecanismo de adoção é real e já causou incidentes em produção.
2. **O `selector` e as labels do `template` precisam casar.** A API valida isso e rejeita o manifesto caso contrário, com a mensagem `selector does not match template labels`.
3. **`selector` é imutável** em `apps/v1`. Para mudá-lo é preciso apagar e recriar o objeto.

O ReplicaSet garante contagem, mas **não sabe atualizar**. Se você alterar a imagem no `template`, ele não fará absolutamente nada com os Pods existentes — o novo template só vale para Pods criados dali em diante. É por isso que **quase nunca escrevemos ReplicaSets à mão**. Escrevemos `Deployment`, e o Deployment cria e coordena ReplicaSets para nós.

```
Deployment  store-api
   |
   +-- ReplicaSet store-api-7d4b8c9f6   (revisão 1, imagem 1.0)  replicas: 0
   +-- ReplicaSet store-api-5f9a2c1b3   (revisão 2, imagem 2.0)  replicas: 3
                     |
                     +-- Pod store-api-5f9a2c1b3-x8k2p
                     +-- Pod store-api-5f9a2c1b3-m4t7q
                     +-- Pod store-api-5f9a2c1b3-b1n9r
```

---

### 4.5 Deployment em profundidade

O `Deployment` é o objeto que você vai escrever 90% das vezes. Ele adiciona ao ReplicaSet a noção de **revisão** e de **transição controlada entre revisões**.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
spec:
  replicas: 3
  revisionHistoryLimit: 5
  minReadySeconds: 10
  progressDeadlineSeconds: 300
  selector:
    matchLabels:
      app: store-api
      tier: backend
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
    spec:
      containers:
        - name: store-api
          image: docker.io/seuusuario/store-api:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8000
          env:
            - name: APP_ENV
              value: "dev"
            - name: DB_HOST
              value: "store-db.store-dev.svc.cluster.local"
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 3
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
```

#### 4.5.1 Campo a campo

| Campo | Função | Armadilha comum |
|---|---|---|
| `spec.replicas` | Número desejado de Pods | Se houver HPA, **não** versione este campo no Git: o HPA o sobrescreve e cada `kubectl apply` reverte a escala |
| `spec.selector` | Como o Deployment encontra seus Pods | Imutável; deve ser subconjunto das labels do template |
| `spec.template` | Molde do Pod | Qualquer byte alterado aqui gera nova revisão |
| `strategy.type` | `RollingUpdate` (padrão) ou `Recreate` | `Recreate` causa downtime, mas é obrigatório com volumes RWO ou migrações de schema incompatíveis |
| `maxSurge` | Quantos Pods **acima** de `replicas` podem existir durante o rollout | Aceita número absoluto ou percentual (`25%`) |
| `maxUnavailable` | Quantos Pods podem estar indisponíveis durante o rollout | `maxSurge: 0` + `maxUnavailable: 0` é inválido — trava o rollout |
| `minReadySeconds` | Tempo que um Pod novo precisa ficar Ready antes de ser considerado disponível | Padrão 0; sem ele, um Pod que fica Ready e quebra 2s depois é contado como sucesso |
| `revisionHistoryLimit` | Quantas ReplicaSets antigas (com 0 réplicas) guardar | Padrão 10; se você zerar, perde a capacidade de `rollout undo` |
| `progressDeadlineSeconds` | Tempo máximo sem progresso antes de marcar o rollout como falho | Padrão 600; marca a condition `Progressing=False`, mas **não** faz rollback automático |

> **Atenção:** `progressDeadlineSeconds` não desfaz nada sozinho. Ele apenas registra `reason: ProgressDeadlineExceeded`. O rollback automático é responsabilidade da sua pipeline de CI/CD (ou do Argo CD, que veremos adiante no curso), que deve monitorar essa condition e chamar `kubectl rollout undo`.

#### 4.5.2 A matemática do RollingUpdate

Com `replicas: 3`, `maxSurge: 1` e `maxUnavailable: 0`, o Kubernetes garante:

- Nunca menos de 3 Pods disponíveis (3 − 0).
- Nunca mais de 4 Pods no total (3 + 1).

A sequência é: cria 1 Pod novo (total 4) → espera ficar Ready + `minReadySeconds` → apaga 1 Pod velho (total 3) → repete. Três ciclos, zero indisponibilidade, custo de 1 Pod extra de recurso. Essa é a configuração recomendada para APIs sem estado.

Se invertermos para `maxSurge: 0` e `maxUnavailable: 1`, a sequência vira: apaga 1 velho (total 2) → cria 1 novo → espera Ready → repete. Nenhum recurso extra é consumido, mas a capacidade cai 33% durante a atualização. Use quando o cluster estiver com pouca folga.

> **Na prática:** a escolha `maxSurge: 25%, maxUnavailable: 25%` (o padrão) é razoável para deployments grandes, mas com `replicas: 3` ela arredonda para 1 e 0 respectivamente — e para `replicas: 2` produz um comportamento que surpreende muita gente. Sempre declare os valores explicitamente.

#### 4.5.3 Por que mudar só a imagem gera uma nova ReplicaSet

Quando você aplica uma alteração no `spec.template`, o Deployment calcula um **hash** do conteúdo do template (o `pod-template-hash`). Esse hash vira:

- parte do nome da nova ReplicaSet (`store-api-5f9a2c1b3`);
- uma label adicionada automaticamente aos Pods e ao selector daquela ReplicaSet.

É esse hash que permite a duas ReplicaSets do mesmo Deployment coexistirem sem roubar Pods uma da outra — cada uma tem um selector efetivamente diferente. E é por isso que qualquer mudança no template, mesmo uma anotação, produz nova revisão: o hash muda. Já mexer em `spec.replicas` **não** altera o template, logo não gera revisão nova — apenas ajusta a contagem da ReplicaSet ativa.

#### 4.5.4 O ciclo de vida do rollout

```bash
# dispara o rollout alterando a imagem
kubectl -n store-dev set image deployment/store-api store-api=docker.io/seuusuario/store-api:2.0

# acompanha em tempo real (bloqueia até terminar ou falhar)
kubectl -n store-dev rollout status deployment/store-api
```

```
Waiting for deployment "store-api" rollout to finish: 1 out of 3 new replicas have been updated...
Waiting for deployment "store-api" rollout to finish: 2 out of 3 new replicas have been updated...
Waiting for deployment "store-api" rollout to finish: 1 old replicas are pending termination...
deployment "store-api" successfully rolled out
```

```bash
kubectl -n store-dev rollout history deployment/store-api
```

```
deployment.apps/store-api
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=store-api-deployment.yaml
2         imagem atualizada para 2.0 (correcao do calculo de frete)
```

A coluna `CHANGE-CAUSE` vem da anotação `kubernetes.io/change-cause`. Ela não é preenchida sozinha:

```bash
kubectl -n store-dev annotate deployment/store-api \
  kubernetes.io/change-cause="imagem atualizada para 2.0 (correcao do calculo de frete)"
```

Para voltar atrás:

```bash
kubectl -n store-dev rollout undo deployment/store-api              # volta uma revisão
kubectl -n store-dev rollout undo deployment/store-api --to-revision=1
```

O `undo` não apaga a revisão ruim — ele cria uma **nova revisão** cujo template é igual ao da revisão alvo. Seu histórico vira 1, 2, 3, onde 3 é idêntica a 1. Isso preserva a auditoria.

E para inspecionar um canary antes de continuar:

```bash
kubectl -n store-dev rollout pause deployment/store-api
# ... altera imagem, recursos, env — nada acontece enquanto pausado ...
kubectl -n store-dev rollout resume deployment/store-api
```

`pause` é a única forma nativa de agrupar várias alterações em um único rollout. Sem ele, mudar imagem e depois variáveis de ambiente dispara dois rollouts em sequência.

---

### 4.6 Estratégias de release com objetos nativos

| Estratégia | Downtime | Custo de recursos | Risco | Como fazer no K8s puro |
|---|---|---|---|---|
| **Recreate** | Sim (segundos a minutos) | Nenhum extra | Alto para o usuário, baixo para dados | `strategy.type: Recreate` |
| **Rolling** | Não | +`maxSurge` Pods | Médio: as duas versões convivem | `strategy.type: RollingUpdate` (padrão) |
| **Blue-green** | Não | 2× (dois ambientes completos) | Baixo: troca instantânea e reversível | Dois Deployments + `kubectl patch` no selector do Service |
| **Canary** | Não | +N Pods do canary | Muito baixo: exposição gradual | Dois Deployments com label comum + um Service, proporção pelo número de réplicas |

#### 4.6.1 Blue-green com Service

O truque é ter dois Deployments diferenciados por uma label de versão e um Service cujo selector aponta para apenas uma delas.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
spec:
  type: ClusterIP
  selector:
    app: store-api
    version: blue        # <- o "interruptor"
  ports:
    - name: http
      port: 80
      targetPort: 8000
```

```bash
# sobe o ambiente green (Deployment store-api-green com label version: green), valida, e então:
kubectl -n store-dev patch service store-api \
  -p '{"spec":{"selector":{"app":"store-api","version":"green"}}}'
```

A troca é atômica do ponto de vista do Service: o `kube-proxy` reescreve as regras e todo o tráfego novo vai para green. O rollback é o mesmo comando com `blue`. O custo é manter duas frotas completas no ar durante a janela.

#### 4.6.2 Canary por proporção de réplicas

Aqui os dois Deployments compartilham a label que o Service seleciona:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
spec:
  selector:
    app: store-api        # nao menciona version -> pega os dois
  ports:
    - port: 80
      targetPort: 8000
```

Com `store-api-stable` em 9 réplicas e `store-api-canary` em 1, aproximadamente 10% do tráfego cai no canary — porque o `kube-proxy` distribui entre os endpoints de forma aproximadamente uniforme. É um canary "de pobre", mas funciona e não exige nada além do Kubernetes.

> **Atenção:** a granularidade é limitada pelo número de Pods. Para 1% de canary você precisaria de 100 réplicas. Para percentuais finos e roteamento por header/cookie, é preciso um Ingress Controller com suporte a *traffic splitting* (nginx com `canary-weight`), a Gateway API ou um service mesh — assuntos do capítulo 5.

---

### 4.7 Escalonamento automático

#### 4.7.1 Os três eixos

| Eixo | Objeto | O que muda | Quando usar |
|---|---|---|---|
| Horizontal (Pods) | `HorizontalPodAutoscaler` | Número de réplicas | Cargas stateless, variação de tráfego |
| Vertical (Pod) | `VerticalPodAutoscaler` | `requests`/`limits` do container | Dimensionar corretamente; cargas que não paralelizam |
| Nós | Cluster Autoscaler / Karpenter | Quantidade e tipo de nós | Quando os Pods ficam `Pending` por falta de capacidade |

Os três se combinam, mas **HPA e VPA não devem controlar a mesma métrica no mesmo workload** (o VPA baixaria o request de CPU, o que faria o HPA ver utilização alta e escalar, o que baixaria o uso por Pod, e assim por diante). Use VPA em modo `Off`/`Initial` para recomendação e HPA para a ação.

#### 4.7.2 metrics-server

O HPA baseado em CPU/memória depende do `metrics-server`, que agrega os dados do `kubelet`/cAdvisor e os publica na API `metrics.k8s.io`. Ele **não** vem instalado por padrão.

```bash
minikube addons enable metrics-server
kubectl -n kube-system get deployment metrics-server
kubectl top pods -n store-dev
```

```
NAME                         CPU(cores)   MEMORY(bytes)
store-api-5f9a2c1b3-b1n9r    3m           64Mi
store-api-5f9a2c1b3-m4t7q    2m           61Mi
store-api-5f9a2c1b3-x8k2p    3m           63Mi
```

> **Nota:** o metrics-server serve para autoscaling e para `kubectl top`. Ele **não** é um sistema de monitoramento — não guarda histórico. Para isso existe o Prometheus, e para expor métricas do Prometheus ao HPA existe o `prometheus-adapter`, que implementa as APIs `custom.metrics.k8s.io` e `external.metrics.k8s.io`.

#### 4.7.3 HPA com autoscaling/v2

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: store-api
  namespace: store-dev
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: store-api
  minReplicas: 3
  maxReplicas: 12
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 200Mi
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
        - type: Percent
          value: 25
          periodSeconds: 60
```

Pontos essenciais:

- **`averageUtilization` é percentual do `requests`, não do limite nem da capacidade do nó.** Com `requests.cpu: 100m` e alvo de 60%, o HPA quer manter cada Pod em torno de 60m de CPU. Se o Pod não declara `requests.cpu`, o HPA baseado em `Utilization` simplesmente não funciona e reporta `<unknown>`.
- **A fórmula** é `réplicasDesejadas = ceil(réplicasAtuais × (métricaAtual / métricaAlvo))`, com uma tolerância de 10% para evitar oscilação.
- **Com várias métricas, vence a maior sugestão.** O HPA calcula o desejado para cada métrica e usa o máximo.
- **`behavior`** (disponível desde `autoscaling/v2`) é o que separa um HPA educado de um HPA histérico. Subir rápido (janela 0, até dobrar a cada 15s) e descer devagar (janela de 5 minutos, no máximo 25% por minuto) é o padrão sensato: o custo de escalar demais por engano é dinheiro; o de escalar de menos é indisponibilidade.

```bash
kubectl -n store-dev get hpa store-api
```

```
NAME        REFERENCE              TARGETS                        MINPODS   MAXPODS   REPLICAS   AGE
store-api   Deployment/store-api   cpu: 312%/60%, memory: 70Mi/200Mi   3         12        8          4m
```

#### 4.7.4 VPA, Cluster Autoscaler e Karpenter (visão geral)

- **VPA** observa o consumo real e recomenda (ou aplica) `requests`/`limits`. Nos modos `Auto`/`Recreate` ele **reinicia** o Pod para aplicar — por isso muita equipe usa apenas `updateMode: "Off"` e trata a recomendação como insumo de revisão. O recurso de *in-place resize* de Pods vem amadurecendo nas versões 1.33/1.34 e tende a tornar o VPA muito menos disruptivo.
- **Cluster Autoscaler** é o componente clássico: observa Pods `Pending` que não cabem em nenhum nó e aumenta o *node group* (no EKS, um Auto Scaling Group). Trabalha em unidades de grupo, com tipos de instância pré-definidos.
- **Karpenter** é a abordagem moderna da AWS: em vez de escalar grupos fixos, ele olha os requisitos dos Pods pendentes (CPU, memória, arquitetura, zona, tolerations) e provisiona **a instância certa** diretamente na EC2, em segundos. Também faz consolidação: identifica nós subutilizados e recoloca os Pods para desligar máquinas. Voltaremos a ele no capítulo 6.

---

### 4.8 Distribuição de carga: onde os Pods vão parar

O `kube-scheduler` decide o nó de cada Pod em duas fases: **filtragem** (quais nós são viáveis?) e **pontuação** (qual o melhor entre os viáveis?). Você influencia as duas.

#### 4.8.1 nodeName e nodeSelector

```yaml
spec:
  nodeName: minikube-m02          # forca o no, ignora o scheduler — evite
```

```yaml
spec:
  nodeSelector:
    disktype: ssd                  # so agenda em nos com essa label
```

`nodeName` pula o scheduler inteiro; se o nó não tiver capacidade, o Pod fica travado. É útil apenas para depuração. `nodeSelector` é o filtro simples e ainda muito usado — mas só faz correspondência exata de igualdade.

#### 4.8.2 Affinity e anti-affinity

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: kubernetes.io/arch
                operator: In
                values: ["amd64"]
      preferredDuringSchedulingIgnoredDuringExecution:
        - weight: 80
          preference:
            matchExpressions:
              - key: node.kubernetes.io/instance-type
                operator: In
                values: ["t3.medium"]
    podAntiAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - labelSelector:
            matchLabels:
              app: store-api
          topologyKey: kubernetes.io/hostname
```

Leia `topologyKey` como "a régua da distância". Com `kubernetes.io/hostname`, dois Pods são "vizinhos" se estiverem no mesmo nó; a anti-affinity acima proíbe dois Pods da `store-api` no mesmo nó. Com `topology.kubernetes.io/zone`, a régua vira a zona de disponibilidade.

A diferença entre `required` e `preferred` é dura:

- `requiredDuringScheduling...`: se não houver nó que satisfaça, o Pod fica **Pending para sempre**. Com anti-affinity por hostname e 5 réplicas em um cluster de 3 nós, 2 Pods ficam Pending.
- `preferredDuringScheduling...`: vira pontuação. O scheduler tenta, e se não der, agenda mesmo assim. O `weight` (1–100) pondera.

O sufixo `IgnoredDuringExecution` significa que a regra vale no momento do agendamento; se as labels do nó mudarem depois, o Pod **não** é despejado.

> **Na prática:** anti-affinity `required` por hostname é comum demais e derruba clusters em manutenção. Prefira `preferred` com peso alto, ou — melhor ainda — `topologySpreadConstraints`.

#### 4.8.3 topologySpreadConstraints

```yaml
spec:
  topologySpreadConstraints:
    - maxSkew: 1
      topologyKey: topology.kubernetes.io/zone
      whenUnsatisfiable: DoNotSchedule
      labelSelector:
        matchLabels:
          app: store-api
    - maxSkew: 1
      topologyKey: kubernetes.io/hostname
      whenUnsatisfiable: ScheduleAnyway
      labelSelector:
        matchLabels:
          app: store-api
```

`maxSkew: 1` significa: a diferença entre a zona mais cheia e a mais vazia não pode passar de 1 Pod. É uma expressão muito mais direta de "espalhe uniformemente" do que a anti-affinity, e `whenUnsatisfiable: ScheduleAnyway` dá a válvula de escape que a anti-affinity `required` não tem. A prática recomendada é: **`DoNotSchedule` por zona** (para sobreviver à queda de uma AZ) e **`ScheduleAnyway` por nó** (para melhor distribuição sem travar o rollout).

#### 4.8.4 Taints e tolerations

Enquanto affinity é o Pod escolhendo o nó, taint é o **nó rejeitando Pods**.

```bash
kubectl taint nodes minikube-m03 workload=gpu:NoSchedule
```

```yaml
spec:
  tolerations:
    - key: "workload"
      operator: "Equal"
      value: "gpu"
      effect: "NoSchedule"
```

Os três efeitos:

| Efeito | Comportamento |
|---|---|
| `NoSchedule` | Novos Pods sem toleration não são agendados; os existentes ficam |
| `PreferNoSchedule` | Evita, mas agenda se não houver alternativa |
| `NoExecute` | Despeja Pods sem toleration que já estejam no nó (respeitando `tolerationSeconds`) |

Um detalhe importante: **toleration não atrai**. Tolerar o taint só permite que o Pod vá para lá; para garantir que vá, combine com `nodeSelector`/`nodeAffinity`. O padrão "nós dedicados" é sempre taint + affinity juntos.

#### 4.8.5 PodDisruptionBudget e drenagem de nó

Existem dois tipos de interrupção: **involuntária** (nó pega fogo) e **voluntária** (você faz upgrade do cluster). O PDB protege contra a segunda.

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-api
  namespace: store-dev
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: store-api
```

```bash
kubectl cordon minikube-m02                     # marca como nao-agendavel
kubectl drain minikube-m02 --ignore-daemonsets --delete-emptydir-data
kubectl uncordon minikube-m02                   # devolve o no ao pool
```

O `drain` usa a **Eviction API**, que consulta os PDBs antes de aceitar cada despejo. Com `minAvailable: 2` e 3 réplicas, o drain despeja um Pod, espera o substituto ficar Ready em outro nó, e só então despeja o próximo. Se as 3 réplicas estiverem no mesmo nó, o drain **trava** com:

```
error when evicting pods/"store-api-5f9a2c1b3-x8k2p" -n "store-dev" (will retry after 5s):
Cannot evict pod as it would violate the pod's disruption budget.
```

Isso não é um bug — é o PDB fazendo exatamente o seu trabalho e avisando que sua distribuição está ruim. A correção é espalhar os Pods (topology spread), não remover o PDB.

> **Atenção:** um PDB com `minAvailable` igual ao número de réplicas (ou `maxUnavailable: 0`) bloqueia **qualquer** manutenção, indefinidamente. É a receita para o time de plataforma não conseguir atualizar o cluster.

#### 4.8.6 Alta disponibilidade do control plane

Alta disponibilidade não é só ter réplicas da aplicação. O control plane também precisa sobreviver:

- **etcd usa Raft e exige quórum** de `(N/2)+1` membros. Com 3 membros, tolera 1 falha; com 5, tolera 2. Números **ímpares** apenas — 4 membros toleram a mesma falha que 3 e ainda aumentam a latência de escrita.
- **Múltiplas zonas de disponibilidade.** O padrão em nuvem é 3 AZs, um membro do etcd em cada. No EKS isso é feito pela AWS e você não vê nem paga por instância — o control plane gerenciado tem preço fixo por hora e já é multi-AZ.
- **`kube-apiserver` é stateless** e escala horizontalmente atrás de um balanceador. Já `kube-scheduler` e `kube-controller-manager` rodam em modo ativo-passivo, com eleição de líder via Lease.
- **Os nós de trabalho também devem estar em 3 AZs**, e é aí que `topologySpreadConstraints` por `topology.kubernetes.io/zone` deixa de ser enfeite e vira requisito.

#### 4.8.7 StatefulSet como contraste (breve)

Deployment assume que os Pods são **intercambiáveis**: qualquer um pode atender qualquer requisição, os nomes são aleatórios e a ordem de criação não importa. Bancos de dados quebram todas essas suposições. O `StatefulSet` oferece identidade estável (`store-db-0`, `store-db-1`), ordem determinística de criação e término, e um PVC próprio por réplica via `volumeClaimTemplates`. A `store-db` da FIAP Store será um StatefulSet — assunto aprofundado do capítulo 7.

---

### 4.9 Ponte com Serverless

O HPA e o modelo de concorrência do AWS Lambda resolvem o mesmo problema com filosofias opostas, e comparar os dois esclarece bastante o que estamos fazendo.

No **Lambda**, a unidade de escala é a **execução concorrente**. Cada requisição é atendida por um *execution environment* que processa **uma** invocação por vez (com exceção do modelo de streaming). Se chegam 200 requisições simultâneas, a AWS materializa 200 ambientes; quando o tráfego cessa, tudo é recolhido e o custo vai a zero. Você não configura threshold, janela de estabilização nem `maxSurge` — só um teto de concorrência. O preço da elegância é o **cold start**: o primeiro request de cada ambiente novo paga a inicialização do runtime (centenas de milissegundos a alguns segundos, dependendo da linguagem e do tamanho do pacote). Mitiga-se com *provisioned concurrency* — que, note bem, é exatamente reinventar o `minReplicas` do HPA, com custo por ambiente reservado.

No **HPA**, a unidade de escala é a **réplica**, e uma réplica atende dezenas de requisições concorrentes. O escalonamento é reativo e amostrado: o controlador lê métricas a cada 15 segundos, aplica a fórmula, respeita `behavior`. Entre o pico de tráfego e o Pod novo em Ready há facilmente 30 a 90 segundos — por isso `minReplicas` existe e por isso `scaleUp.stabilizationWindowSeconds: 0` é quase sempre a escolha certa.

O ponto de encontro entre os dois mundos tem nome: **KEDA** e **Knative**. O KEDA (2.16+) instala um HPA "turbinado" capaz de ler *scalers* externos — profundidade de uma fila SQS, lag de um consumer group Kafka, resultado de uma query no Prometheus — e, decisivamente, de fazer **scale-to-zero**: quando a fila esvazia, o Deployment do `store-worker` vai a 0 réplicas; quando chega mensagem, o KEDA ativa um Pod. O Knative Serving faz o mesmo para HTTP, mantendo um componente chamado *Activator* que segura a requisição enquanto o primeiro Pod sobe.

O impacto econômico é o que os alunos costumam subestimar. Compare o `store-worker`, que processa pedidos em batch por 2 horas por dia:

| Modelo | Cobrança | Custo relativo |
|---|---|---|
| Deployment com 2 réplicas fixas | 24 h/dia de CPU e memória reservadas | 100% |
| Deployment + HPA (`minReplicas: 1`) | 24 h de 1 réplica + picos | ~55% |
| Deployment + KEDA (`minReplicaCount: 0`) | ~2 h/dia | ~9% |
| AWS Lambda | Por GB-segundo efetivamente executado | ~5%, mas com teto de 15 min por invocação |

A regra prática: **carga contínua e previsível favorece réplicas** (o custo por requisição de um Pod ocupado é imbatível); **carga esporádica e em rajadas favorece scale-to-zero**. O erro caro é manter réplicas ociosas 22 horas por dia porque "sempre foi assim".

---

### 4.10 Laboratório guiado (feito em sala)

**Objetivo:** subir a `store-api` com 3 réplicas, fazer rolling update para a v2, voltar atrás e, por fim, escalar automaticamente sob carga.

#### Passo 1 — preparar o cluster e o namespace

```bash
minikube start --nodes=3 --cpus=2 --memory=2200 --driver=docker
minikube addons enable metrics-server
kubectl create namespace store-dev
kubectl get nodes
```

```
NAME           STATUS   ROLES           AGE   VERSION
minikube       Ready    control-plane   2m    v1.34.0
minikube-m02   Ready    <none>          98s   v1.34.0
minikube-m03   Ready    <none>          82s   v1.34.0
```

#### Passo 2 — construir as duas versões da imagem

Construa na sua máquina e depois carregue as imagens no cluster com `minikube image load` — em cluster **multi-nó** essa é a única forma de garantir que a imagem exista nos três nós (o `eval $(minikube docker-env)` colocaria a imagem apenas no nó primário, e os Pods agendados em `m02`/`m03` falhariam com `ErrImageNeverPull`).

`main.py` (v1 — o código de referência da apostila) e um `Dockerfile`:

```dockerfile
FROM python:3.12-slim
WORKDIR /app
RUN pip install --no-cache-dir fastapi==0.115.* uvicorn[standard]==0.32.*
COPY main.py .
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

Para a v2, acrescente ao `main.py` um endpoint que consome CPU — precisaremos dele para acionar o HPA:

```python
@app.get("/versao")
def versao():
    return {"versao": "2.0"}

@app.get("/pesado")
def pesado():
    # queima CPU de proposito para exercitar o HPA
    total = 0
    for i in range(3_000_000):
        total += i * i
    return {"resultado": total, "pod": socket.gethostname()}
```

```bash
docker build -t store-api:1.0 .
# ... edite o main.py acrescentando /versao e /pesado ...
docker build -t store-api:2.0 .
docker images | grep store-api

# distribui as duas imagens para TODOS os nos do cluster
minikube image load store-api:1.0
minikube image load store-api:2.0
minikube image ls | grep store-api
```

#### Passo 3 — aplicar o Deployment e o Service

Salve o manifesto da seção 4.5 como `store-api-deployment.yaml`, trocando a imagem por `store-api:1.0` e `imagePullPolicy: Never` (a imagem é local). Acrescente o `topologySpreadConstraints` da seção 4.8.3 (apenas o de hostname, já que o minikube não tem zonas) e o Service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
spec:
  type: ClusterIP
  selector:
    app: store-api
    tier: backend
  ports:
    - name: http
      port: 80
      targetPort: 8000
```

```bash
kubectl apply -f store-api-deployment.yaml
kubectl -n store-dev rollout status deployment/store-api
kubectl -n store-dev get pods -o wide
```

```
NAME                         READY   STATUS    RESTARTS   AGE   IP           NODE
store-api-6c9f7d5b84-2wq8n   1/1     Running   0          41s   10.244.1.4   minikube-m02
store-api-6c9f7d5b84-hl4jd   1/1     Running   0          41s   10.244.2.3   minikube-m03
store-api-6c9f7d5b84-tz6xk   1/1     Running   0          41s   10.244.0.7   minikube
```

Um Pod por nó — o topology spread funcionou.

#### Passo 4 — rolling update para a v2, observando

Em um terminal, monitore:

```bash
kubectl -n store-dev get pods -w
```

Em outro:

```bash
kubectl -n store-dev set image deployment/store-api store-api=store-api:2.0
kubectl -n store-dev annotate deployment/store-api \
  kubernetes.io/change-cause="v2.0 com endpoints /versao e /pesado"
kubectl -n store-dev rollout status deployment/store-api
```

Note no terminal de watch que, com `maxSurge: 1`/`maxUnavailable: 0`, aparece um quarto Pod antes de qualquer um ser terminado. Confirme as duas ReplicaSets:

```bash
kubectl -n store-dev get rs
```

```
NAME                   DESIRED   CURRENT   READY   AGE
store-api-6c9f7d5b84   0         0         0       6m
store-api-7fd54c8b96   3         3         3       52s
```

Valide a v2 de dentro do cluster:

```bash
kubectl -n store-dev run curl --rm -it --image=curlimages/curl --restart=Never -- \
  curl -s http://store-api.store-dev.svc.cluster.local/versao
```

```
{"versao":"2.0"}
```

#### Passo 5 — rollback

```bash
kubectl -n store-dev rollout history deployment/store-api
kubectl -n store-dev rollout undo deployment/store-api
kubectl -n store-dev rollout status deployment/store-api
kubectl -n store-dev get rs
```

A ReplicaSet antiga volta a 3 e a nova vai a 0. Confirme que `/versao` agora responde 404. Depois **refaça o rollout para a v2** — vamos precisar do endpoint `/pesado`.

#### Passo 6 — HPA sob carga

```bash
kubectl apply -f store-api-hpa.yaml     # manifesto da secao 4.7.3
kubectl -n store-dev get hpa -w
```

Em outro terminal, gere carga com o `hey` dentro de um Pod:

```bash
kubectl -n store-dev run carga --image=williamyeh/hey --restart=Never -- \
  -z 180s -c 60 http://store-api.store-dev.svc.cluster.local/pesado
```

Se preferir o `ab` (Apache Bench):

```bash
kubectl -n store-dev run carga --image=httpd:2.4-alpine --restart=Never -- \
  ab -t 180 -c 60 http://store-api.store-dev.svc.cluster.local/pesado
```

Acompanhe:

```
NAME        REFERENCE              TARGETS         MINPODS   MAXPODS   REPLICAS   AGE
store-api   Deployment/store-api   cpu: 4%/60%     3         12        3          30s
store-api   Deployment/store-api   cpu: 289%/60%   3         12        6          75s
store-api   Deployment/store-api   cpu: 174%/60%   3         12        12         105s
store-api   Deployment/store-api   cpu: 58%/60%    3         12        12         3m
```

```bash
kubectl -n store-dev describe hpa store-api | tail -8
```

```
Events:
  Type    Reason             Age   From                       Message
  ----    ------             ----  -------------------------  -------
  Normal  SuccessfulRescale  2m    horizontal-pod-autoscaler  New size: 6;  reason: cpu resource utilization (percentage of request) above target
  Normal  SuccessfulRescale  90s   horizontal-pod-autoscaler  New size: 12; reason: cpu resource utilization (percentage of request) above target
```

Quando a carga terminar, **cronometre a descida**: por causa de `scaleDown.stabilizationWindowSeconds: 300`, nada acontece nos primeiros 5 minutos, e depois a redução é de no máximo 25% por minuto. Esse é o comportamento correto — e é a diferença mais visível em relação ao Lambda, que vai a zero em segundos.

#### Passo 7 — PDB e drain

```bash
kubectl apply -f store-api-pdb.yaml     # minAvailable: 2
kubectl drain minikube-m02 --ignore-daemonsets --delete-emptydir-data
kubectl -n store-dev get pods -o wide
kubectl uncordon minikube-m02
```

Observe que os Pods do `minikube-m02` são recriados nos outros nós antes de o comando retornar, e que o `drain` respeita o orçamento de interrupção.

#### Passo 8 — limpeza

```bash
kubectl delete namespace store-dev
minikube delete
```

---

### 4.11 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `rollout status` fica preso em "Waiting for deployment..." | Imagem inexistente ou sem credenciais | `kubectl get pods` mostra `ErrImagePull`/`ImagePullBackOff`; `kubectl describe pod` traz `Failed to pull image` | Corrigir tag/registry, `imagePullSecrets`, ou `imagePullPolicy: Never` para imagem local do minikube |
| Rollout trava com Pods `Running` mas `0/1 READY` | readinessProbe falhando (path errado, porta errada, app demora a subir) | `kubectl describe pod` → eventos `Readiness probe failed: HTTP probe failed with statuscode: 404`; `kubectl logs` | Ajustar `path`/`port`, aumentar `initialDelaySeconds` ou usar `startupProbe` |
| `ProgressDeadlineExceeded` no `describe deployment` | Rollout sem progresso além de `progressDeadlineSeconds` | `kubectl describe deploy store-api` → condition `Progressing=False` | Investigar a causa raiz e `kubectl rollout undo` |
| HPA mostra `TARGETS: <unknown>/60%` | metrics-server ausente, ou container sem `resources.requests.cpu` | `kubectl top pods` falha; `kubectl describe hpa` → `FailedGetResourceMetric` | `minikube addons enable metrics-server`; declarar `requests` |
| HPA não escala mesmo com CPU alta | Já está em `maxReplicas`, ou não há capacidade e os Pods estão `Pending` | `kubectl get hpa`, `kubectl get pods` (Pending), `kubectl describe pod` → `Insufficient cpu` | Elevar `maxReplicas`; adicionar nós / Cluster Autoscaler |
| Réplicas voltam a 3 sozinhas após `kubectl scale` | `kubectl apply` de um manifesto com `replicas` fixo, brigando com o HPA | Comparar `spec.replicas` do YAML com o HPA | Remover `replicas` do manifesto versionado quando houver HPA |
| Pods `Pending` com `didn't match pod anti-affinity rules` | Anti-affinity `required` com mais réplicas do que nós | `kubectl describe pod` → seção Events do scheduler | Trocar por `preferred` ou `topologySpreadConstraints` com `ScheduleAnyway` |
| `kubectl drain` nunca termina | PDB bloqueando: `minAvailable` alto demais ou Pods concentrados em um nó | Mensagem `Cannot evict pod as it would violate the pod's disruption budget` | Espalhar réplicas, aumentar `replicas`, ou relaxar o PDB — nunca removê-lo em produção sem plano |
| Pods `Pending` com `node(s) had untolerated taint` | Taint no nó sem toleration correspondente | `kubectl describe node <no> \| grep -i taint` | Adicionar toleration ou remover o taint (`kubectl taint nodes <no> chave-`) |
| Deployment recusado no apply | `selector` não casa com as labels do template, ou tentativa de alterar `selector` | Mensagem `selector does not match template labels` / `field is immutable` | Alinhar labels; para mudar o selector, recriar o Deployment |

---

### 4.12 Resumo do capítulo

Passamos da gestão de Pods individuais para a gestão de populações. O ReplicaSet introduz o laço de reconciliação — o coração conceitual do Kubernetes — mantendo N Pods vivos a partir de um selector e de um template. Sobre ele, o Deployment acrescenta revisões e transições controladas, permitindo atualizar sem downtime e voltar atrás em um comando. Com dois Deployments e um Service implementamos blue-green e canary sem ferramenta alguma além do Kubernetes. O HPA nos deu elasticidade horizontal reativa, com `behavior` para escalar rápido e encolher devagar, enquanto affinity, topology spread, taints e PDB garantem que essa elasticidade não coloque todos os ovos na mesma cesta — nem impeça o time de plataforma de fazer manutenção.

Pontos-chave:

1. **Controladores reconciliam**: você declara o desejado, o Kubernetes persegue a diferença continuamente. `kubectl delete pod` em um Pod gerenciado é inócuo.
2. **Deployment é o objeto de trabalho**; ReplicaSet é detalhe de implementação que você lê em diagnósticos, não escreve à mão.
3. **Qualquer mudança no `spec.template` gera novo `pod-template-hash`**, logo nova ReplicaSet e nova revisão. Mudar `replicas` não gera revisão.
4. **`maxSurge` e `maxUnavailable` são a política de risco do rollout**; declare-os explicitamente e use `minReadySeconds` para não confiar em um Ready prematuro.
5. **HPA depende de `requests` e do metrics-server**; sem os dois, ele reporta `<unknown>` e não faz nada. `behavior` evita oscilação.
6. **`topologySpreadConstraints` é superior à anti-affinity `required`** na maioria dos casos: expressa uniformidade e tem válvula de escape.
7. **PDB protege interrupções voluntárias** e é o que torna `kubectl drain` seguro — mas mal dimensionado bloqueia manutenção para sempre.

---

### 4.13 Exercícios (para casa)

#### Exercício 4.1 — Deployment com 4 réplicas, rolling update controlado e leitura do histórico

**Tempo estimado:** 30 min · *Fixação guiada*

- **Objetivo:** dominar o ciclo completo de rollout de um Deployment e a leitura de revisões.
- **Enunciado:** no namespace `store-dev`, crie o Deployment `store-api` com **4 réplicas** da imagem `store-api:1.0`, `strategy.type: RollingUpdate` com `maxSurge: 1`, `maxUnavailable: 0`, `minReadySeconds: 15`, `revisionHistoryLimit: 3` e `progressDeadlineSeconds: 120`. Inclua `readinessProbe` em `/ready` e `resources.requests` de `100m`/`128Mi`. Aplique, aguarde estabilizar e registre o `CHANGE-CAUSE` da revisão 1 com `kubectl annotate`. Em seguida, faça três alterações sucessivas, anotando o `CHANGE-CAUSE` de cada uma: (a) atualizar a imagem para `store-api:2.0`; (b) alterar a variável `APP_ENV` de `dev` para `homolog`; (c) atualizar `resources.limits.memory` para `384Mi`. Durante o item (a), mantenha `kubectl get pods -w` em outro terminal e **anote a contagem máxima de Pods simultâneos observada**. Ao final, execute `kubectl rollout history` e explique, em comentário dentro do próprio YAML, por que existem exatamente 3 revisões visíveis mesmo tendo feito 4 aplicações.
- **Entregável:** arquivo `e41-store-api-deployment.yaml`, um arquivo `e41-comandos.sh` com a sequência exata de comandos usada, e a saída completa de `kubectl -n store-dev rollout history deployment/store-api` e de `kubectl -n store-dev get rs -o wide`.
- **Critério de aceite:** `kubectl -n store-dev get deploy store-api` mostra `4/4` READY; `rollout history` lista no máximo 3 revisões, todas com `CHANGE-CAUSE` preenchido; `kubectl get rs` mostra exatamente uma ReplicaSet com DESIRED igual a 4 e as demais com 0; a contagem máxima de Pods observada durante o rollout foi 5.
- <details><summary>Dica</summary>`revisionHistoryLimit` controla quantas ReplicaSets com 0 réplicas sobrevivem — a mais antiga é coletada. Lembre que `CHANGE-CAUSE` só aparece se você anotar o Deployment (`kubernetes.io/change-cause`) *antes* ou logo após cada mudança.</details>

#### Exercício 4.2 — Distribuição garantida com anti-affinity e topologySpreadConstraints

**Tempo estimado:** 45 min · *Aplicação*

- **Objetivo:** aplicar regras de distribuição e comprovar empiricamente o resultado no cluster.
- **Enunciado:** suba um minikube com 3 nós (`minikube start --nodes=3`). Rotule os nós de trabalho simulando zonas: `kubectl label node minikube topology.kubernetes.io/zone=za`, `minikube-m02` com `zb` e `minikube-m03` com `zc`. Crie **duas versões** do Deployment `store-api` com 6 réplicas: (A) usando `podAntiAffinity` `preferredDuringSchedulingIgnoredDuringExecution` com `weight: 100` e `topologyKey: kubernetes.io/hostname`; (B) usando `topologySpreadConstraints` com `maxSkew: 1` em `topology.kubernetes.io/zone` (`DoNotSchedule`) **e** em `kubernetes.io/hostname` (`ScheduleAnyway`). Para cada versão, colete a distribuição real com `kubectl -n store-dev get pods -o wide`. Depois, escale ambas para **10 réplicas** e colete de novo. Por fim, troque na versão (A) o `preferred` por `required` e observe o que acontece. Monte uma tabela comparando as três configurações (Pods por nó em 6 e em 10 réplicas, e quantos ficaram `Pending`).
- **Entregável:** `e42-antiaffinity.yaml`, `e42-topologyspread.yaml`, `e42-required.yaml`, a tabela comparativa em `e42-resultado.md` e a saída de `kubectl -n store-dev get pods -o wide --sort-by=.spec.nodeName` para cada cenário.
- **Critério de aceite:** na versão (B) com 6 réplicas, a distribuição é exatamente 2/2/2 e nenhum Pod fica `Pending`; com `required` por hostname e 10 réplicas, pelo menos 7 Pods ficam `Pending` e `kubectl describe pod` de um deles exibe a mensagem `didn't match pod anti-affinity rules`.
- <details><summary>Dica</summary>Use `kubectl get pods -o custom-columns=NOME:.metadata.name,NO:.spec.nodeName` para uma tabela limpa. O nó control-plane do minikube é agendável por padrão, então conte com ele nas 3 "zonas".</details>

#### Exercício 4.3 — Depuração: três rollouts quebrados

**Tempo estimado:** 45 min · *Análise / depuração*

- **Objetivo:** diagnosticar sistematicamente falhas de rollout, distinguindo causa raiz de sintoma.
- **Enunciado:** você receberá (ou construirá) três cenários com defeito, e para cada um deve produzir um **laudo** contendo: sintoma observado, comando que revelou a causa, causa raiz e correção aplicada.
  - **Cenário A — imagem inexistente:** faça `kubectl set image deployment/store-api store-api=store-api:9.9`. O `rollout status` deve travar. Explique por que os Pods da versão anterior **continuam servindo tráfego normalmente** e por que isso é uma virtude do RollingUpdate com `maxUnavailable: 0`.
  - **Cenário B — readiness quebrada:** aplique uma revisão em que a `readinessProbe` aponta para `/pronto` (endpoint que não existe) na porta 8000. Diagnostique e explique a diferença de comportamento se o defeito estivesse na `livenessProbe` em vez da readiness.
  - **Cenário C — PDB bloqueando drain:** com `minAvailable: 3` e 3 réplicas, tente `kubectl drain` em um nó que hospeda ao menos um Pod da `store-api`. Explique a mensagem de erro, calcule qual é o maior `minAvailable` que ainda permite drenar um nó com 3 réplicas bem distribuídas, e corrija.
- **Entregável:** `e43-laudo.md` com os três laudos (sintoma, comando de diagnóstico com a saída real colada, causa raiz, correção), mais os manifestos corrigidos.
- **Critério de aceite:** cada laudo cita ao menos um comando de diagnóstico com a saída real (não inventada) — respectivamente `ImagePullBackOff` no `get pods`, `Readiness probe failed: HTTP probe failed with statuscode: 404` no `describe pod`, e `Cannot evict pod as it would violate the pod's disruption budget` no `drain`; ao final dos três, `kubectl -n store-dev get deploy store-api` mostra READY completo e `kubectl drain` conclui com sucesso.
- <details><summary>Dica</summary>No cenário B, lembre que readiness tira o Pod dos Endpoints do Service mas não o reinicia; liveness reinicia o container em loop (`CrashLoopBackOff`). No cenário C, o PDB é avaliado sobre Pods **prontos**, e o substituto precisa ficar Ready antes do próximo despejo.</details>

#### Exercício 4.4 — Desafio: canary manual medido, e o que o KEDA faria diferente

**Tempo estimado:** 60 min · *Desafio integrador (com componente serverless)*

- **Objetivo:** implementar um canary apenas com objetos nativos, medir empiricamente a divisão de tráfego e contrastar com o modelo orientado a eventos do KEDA.
- **Enunciado:** implemente um canary da `store-api` com dois Deployments — `store-api-stable` (imagem `1.0`, labels `app: store-api, version: stable`) e `store-api-canary` (imagem `2.0`, labels `app: store-api, version: canary`) — e **um único** Service `store-api` cujo selector é apenas `app: store-api`. Configure a proporção 9:1 (9 réplicas stable, 1 canary). Modifique o endpoint `/info` para retornar também a versão da imagem. A partir de um Pod cliente, dispare **500 requisições** e conte quantas caíram em cada versão:

  ```bash
  kubectl -n store-dev run cliente --rm -it --image=curlimages/curl --restart=Never -- sh -c \
    'for i in $(seq 1 500); do curl -s http://store-api/info; echo; done' | sort | uniq -c | sort -rn
  ```

  Registre o percentual medido, compare com os 10% teóricos e **explique a diferença** (considere `sessionAffinity`, keep-alive de conexão HTTP e o algoritmo do kube-proxy). Depois ajuste a proporção para 8:2 e refaça a medição. **Parte serverless:** instale o KEDA (`helm repo add kedacore https://kedacore.github.io/charts && helm install keda kedacore/keda -n keda --create-namespace`) e escreva um `ScaledObject` para o `store-worker` com `minReplicaCount: 0`, `maxReplicaCount: 10` e um trigger `cron` (ou, se preferir, `prometheus`). Escreva um comparativo de **até 400 palavras** respondendo: (i) o canary por réplicas consegue granularidade de 1%? A que custo? (ii) o que muda quando o roteamento é por peso no Ingress em vez de por contagem de Pods? (iii) por que o `store-worker` é candidato natural a scale-to-zero e a `store-api` não é? (iv) estime o custo mensal das duas abordagens supondo 2 h/dia de trabalho efetivo.
- **Entregável:** `e44-stable.yaml`, `e44-canary.yaml`, `e44-service.yaml`, `e44-scaledobject.yaml`, a saída bruta da contagem nas duas proporções e o comparativo em `e44-analise.md`.
- **Critério de aceite:** a contagem de 500 requisições mostra o canary recebendo entre 5% e 15% na proporção 9:1 e entre 15% e 25% na 8:2; `kubectl -n store-dev get scaledobject` mostra o objeto `READY=True`; `kubectl -n store-dev get deploy store-worker` mostra `0/0` réplicas fora da janela do trigger e ao menos 1 réplica dentro dela; o comparativo responde objetivamente aos quatro itens.
- <details><summary>Dica</summary>Para a medição ser justa, use `curl` com conexões novas a cada requisição (o comportamento padrão do loop acima) — reutilizar a conexão fixa o destino e distorce o resultado. Confirme os endpoints do Service com `kubectl -n store-dev get endpointslice -l kubernetes.io/service-name=store-api -o wide`.</details>

---

### 4.14 Leitura recomendada

1. **[DOC]** Kubernetes — *Deployments* e *ReplicaSet*. Disponível em: https://kubernetes.io/docs/concepts/workloads/controllers/deployment/ — a seção "Writing a Deployment Spec" cobre campo a campo o que discutimos em 4.5.
2. **[DOC]** Kubernetes — *Horizontal Pod Autoscaling* e *HPA Walkthrough*. Disponível em: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/ — leia especialmente o algoritmo de cálculo e a seção `behavior`.
3. **[DOC]** Kubernetes — *Assigning Pods to Nodes*, *Pod Topology Spread Constraints* e *Disruptions*. Disponível em: https://kubernetes.io/docs/concepts/scheduling-eviction/ — base para os exercícios 4.2 e 4.3.
4. **[LIVRO]** BURNS, B.; BEDA, J.; HIGHTOWER, K.; EVENSON, L. *Kubernetes: Up and Running*. 3. ed. O'Reilly, 2022 — capítulos 10 (Deployments) e 11 (DaemonSets), com excelente discussão sobre estratégias de rollout.
5. **[ARTIGO]** AWS — *EKS Best Practices Guide: Reliability*. Disponível em: https://docs.aws.amazon.com/eks/latest/best-practices/reliability.html — trata de PDB, topology spread por AZ e Karpenter em contexto real de produção.

---

> ### Marco avaliativo — CHECK POINT 01
>
> Neste ponto do semestre é lançado o **Check Point 01 — Fundamentos, YAML e alta disponibilidade
> no Minikube**, que cobre os capítulos 1 a 4. O enunciado completo, os entregáveis, a rubrica de
> 100 pontos e o roteiro de correção estão na **Parte II — Avaliações**.

---

## Capítulo 5 — Kubernetes Networking e Namespaces

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 4 (arquitetura, minikube, Pods/Services em YAML, Deployments e distribuição), noções de rede do 1º semestre (bridge do Docker, portas, DNS) · **Artefatos:** `namespaces.yaml`, `quota-limitrange.yaml`, `store-ingress.yaml`, `netpol-default-deny.yaml`, `netpol-allow.yaml`, `tls-secret.yaml`

Se o capítulo 4 respondeu "quantas cópias e onde", este capítulo responde "como elas se falam". Rede é, disparado, a parte do Kubernetes onde mais gente se perde — não porque seja difícil, mas porque há camadas demais empilhadas: o namespace de rede do Linux, o plugin CNI, o `kube-proxy`, o CoreDNS, o Ingress Controller, a NetworkPolicy. Cada uma resolve um problema diferente, e quando algo quebra é preciso saber em qual delas olhar. Nosso objetivo hoje é desmontar essa pilha camada por camada e, ao final, publicar a FIAP Store inteira sob um único host com dois caminhos, isolada por políticas de rede.

---

### 5.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as quatro regras fundamentais do modelo de rede do Kubernetes e explicar por que elas eliminam o mapeamento de portas que você usava no Docker.
2. **Analisar** o caminho completo de um pacote — do `localhost` dentro do Pod ao `ClusterIP`, passando por CNI, `kube-proxy` e CoreDNS — e localizar em qual camada uma falha ocorreu.
3. **Construir** manifestos de `Ingress` com regras de host/path, `pathType` correto e terminação TLS via `Secret`.
4. **Aplicar** `NetworkPolicy` para estabelecer uma postura *default-deny* e liberar seletivamente o tráfego necessário, com `podSelector`, `namespaceSelector` e `ipBlock`.
5. **Construir** namespaces com `ResourceQuota` e `LimitRange`, distinguindo o que o namespace isola do que ele **não** isola.
6. **Identificar** o papel de um service mesh e da Gateway API na evolução do modelo de rede do Kubernetes.

---

### 5.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: por que rede em K8s parece mágica (e não é). Revisão do Service do cap. 3 |
| 2 | 15 min | As 4 regras do modelo de rede; CNI e plugins; rede dentro do Pod |
| 3 | 10 min | kube-proxy (iptables x IPVS): como o ClusterIP vira regra |
| 4 | 10 min | CoreDNS: Corefile, dnsPolicy, FQDN e search domains |
| 5 | 10 min | **Intervalo** |
| 6 | 20 min | Ingress e Ingress Controllers; TLS; cert-manager; Gateway API |
| 7 | 10 min | NetworkPolicy: default-deny e liberação seletiva |
| 8 | 10 min | Namespaces, ResourceQuota, LimitRange; panorama de service mesh |
| 9 | 20 min | Laboratório guiado (5.11) |
| 10 | 5 min | Ponte com Serverless, fechamento e exercícios |

---

### 5.3 O modelo de rede: quatro regras que mudam tudo

No Docker, dois containers na mesma bridge se falam por IP interno, e para expor algo ao mundo você faz `-p 8080:80`. Esse mapeamento de portas é uma solução para um problema que o Kubernetes decidiu simplesmente **não ter**. O modelo de rede do Kubernetes impõe quatro regras a qualquer implementação:

1. **Todo Pod tem seu próprio endereço IP**, único no cluster inteiro.
2. **Todo Pod pode se comunicar com todo outro Pod, em qualquer nó, sem NAT.**
3. **Todo nó (e seus agentes, como o kubelet) pode se comunicar com todos os Pods, sem NAT.**
4. **O IP que um Pod vê de si mesmo é o mesmo que os outros veem dele** — não há tradução de endereços escondida.

O jargão para isso é *IP-per-Pod* ou "rede plana". A consequência prática é enorme: **não existe mapeamento de portas em Pods**. Se a `store-api` escuta na 8000, ela escuta na 8000 do IP do Pod, ponto. Dois Pods diferentes podem usar a porta 8000 sem conflito, porque têm IPs distintos. O `containerPort` no manifesto é **puramente documental** — remover esse campo não fecha porta nenhuma.

> **Nota:** o Kubernetes define o *contrato*, não a *implementação*. Ele não sabe rotear pacote nenhum. Quem cumpre as quatro regras é o plugin CNI.

Existem, na prática, três espaços de endereçamento no cluster e confundi-los é fonte constante de erro:

| Faixa | Quem atribui | Exemplo | Roteável fora do cluster? |
|---|---|---|---|
| IPs de nó | A infraestrutura (VPC, hipervisor) | `192.168.49.2` | Sim |
| IPs de Pod (`podCIDR`) | O plugin CNI | `10.244.1.7` | Depende do CNI |
| IPs de Service (`serviceCIDR`) | O `kube-apiserver` | `10.96.0.1` | **Não** — é virtual, só existe como regra |

Um ClusterIP não pertence a nenhuma interface de rede em lugar nenhum. Se você fizer `ping 10.96.0.10`, provavelmente não obterá resposta, e isso é normal: ele é um endereço fictício que só faz sentido dentro de uma regra de encaminhamento do `kube-proxy`.

---

### 5.4 CNI: quem realmente entrega o pacote

O **CNI (Container Network Interface)** é uma especificação da CNCF: um binário que o kubelet invoca quando um Pod nasce, passando o namespace de rede, e que devolve uma interface configurada com IP e rotas.

| Plugin | Modelo | Destaques | NetworkPolicy |
|---|---|---|---|
| **Flannel** | Overlay VXLAN (ou host-gw) | O mais simples; ótimo para aprender | Não (precisa de Calico junto) |
| **Calico** | BGP nativo ou overlay IPIP/VXLAN | Padrão de mercado; políticas ricas (GlobalNetworkPolicy) | Sim, completo |
| **Cilium** | eBPF, sem iptables | Políticas L7 (HTTP path, método), observabilidade com Hubble, pode substituir o kube-proxy | Sim, L3/L4/L7 |
| **AWS VPC CNI** | IPs reais da VPC via ENI | Pod recebe IP da subnet; integra com Security Groups | Só com Calico/Cilium por cima |

Vale entender a distinção entre **overlay** e **rede nativa**. Um overlay (VXLAN) encapsula o pacote do Pod dentro de outro pacote UDP para atravessar a rede física — funciona em qualquer lugar, mas custa CPU e reduz a MTU útil (daí problemas intermitentes de "conexão funciona mas download grande trava"). Já o **AWS VPC CNI** não encapsula nada: ele anexa ENIs secundárias à instância EC2 e entrega ao Pod um IP real da subnet. Vantagem: performance nativa, visibilidade no VPC Flow Logs, Security Groups aplicáveis por Pod. Desvantagem: **os IPs da sua VPC acabam**, e há um limite de Pods por instância determinado pelo tipo de EC2 (uma `t3.medium` suporta 17 Pods; uma `m5.large`, 29). Esse limite pega muita gente de surpresa no capítulo 6 — o Pod fica `Pending` sem que falte CPU ou memória.

O **eBPF** é a fronteira atual. Em vez de programar `iptables`, o Cilium injeta programas verificados no kernel que fazem roteamento, balanceamento e política em O(1), sem percorrer listas de regras. Em clusters com milhares de Services, a diferença é dramática.

#### 5.4.1 Dentro do Pod: um único namespace de rede

Os containers de um Pod **compartilham o mesmo network namespace do Linux**. Isso significa uma única interface `eth0`, um único IP, uma única tabela de portas — e que eles se enxergam por `localhost`.

```
+------------------ Pod store-api (10.244.1.7) -----------------+
|                                                               |
|   [pause]        [store-api:8000]      [sidecar-log:9000]     |
|      \_________________|_______________________/              |
|              network namespace compartilhado                  |
|                     eth0 -> 10.244.1.7                        |
+---------------------------------------------------------------+
                              |  veth pair
                      +---------------+
                      |  cni0 / bridge |  no minikube-m02
                      +---------------+
```

O **pause container** (também chamado de *sandbox* ou *infra container*) é o dono do namespace. Ele é um processo minúsculo em C que só chama `pause()` e colhe processos zumbis. Sua existência garante que o namespace de rede sobreviva quando o container da aplicação reinicia — sem ele, o Pod perderia o IP a cada `CrashLoopBackOff`.

Duas consequências práticas:

- **Dois containers do mesmo Pod não podem usar a mesma porta.** Se a `store-api` usa 8000, um sidecar não pode usar 8000.
- **Um sidecar acessa a aplicação por `http://localhost:8000`**, nunca pelo Service. É assim que funcionam sidecars de proxy (Envoy do Istio) e de coleta de métricas.

---

### 5.5 kube-proxy: como o ClusterIP vira regra

O `kube-proxy` roda em cada nó (como DaemonSet), observa Services e EndpointSlices na API e programa o kernel para que o tráfego para um ClusterIP seja redirecionado a um dos Pods.

#### 5.5.1 Modo iptables (padrão)

Para o Service `store-api` (ClusterIP `10.96.51.20`, porta 80) com três Pods, o kube-proxy cria uma cadeia como:

```
-A KUBE-SERVICES -d 10.96.51.20/32 -p tcp --dport 80 -j KUBE-SVC-XYZ

-A KUBE-SVC-XYZ -m statistic --mode random --probability 0.33333 -j KUBE-SEP-AAA
-A KUBE-SVC-XYZ -m statistic --mode random --probability 0.50000 -j KUBE-SEP-BBB
-A KUBE-SVC-XYZ -j KUBE-SEP-CCC

-A KUBE-SEP-AAA -p tcp -j DNAT --to-destination 10.244.0.7:8000
-A KUBE-SEP-BBB -p tcp -j DNAT --to-destination 10.244.1.4:8000
-A KUBE-SEP-CCC -p tcp -j DNAT --to-destination 10.244.2.3:8000
```

Três observações que explicam quase todo o comportamento que você vai observar:

1. **É DNAT puro.** O pacote sai com destino ao ClusterIP e o kernel reescreve o destino para o IP de um Pod. Não há processo intermediário, não há proxy de userspace, não há latência de hop extra.
2. **O balanceamento é aleatório por conexão, não por requisição.** As probabilidades encadeadas (1/3, depois 1/2 do que sobrou, depois o resto) dão distribuição uniforme. Mas a decisão acontece no `conntrack`, no estabelecimento da conexão TCP — todas as requisições HTTP que reutilizarem aquela conexão (keep-alive) vão para o **mesmo Pod**. Foi isso que distorceu a medição do canary no exercício 4.4.
3. **Escala em O(n).** Com 5.000 Services, cada pacote pode percorrer dezenas de milhares de regras, e cada mudança de endpoint exige reescrever a tabela.

#### 5.5.2 Modo IPVS

O IPVS usa hash tables no kernel (o mesmo subsistema de balanceadores L4 do Linux), com complexidade O(1) e algoritmos selecionáveis: `rr` (round-robin), `lc` (least connection), `sh` (source hashing), entre outros.

```bash
minikube start --extra-config=kube-proxy.mode=ipvs
kubectl -n kube-system logs -l k8s-app=kube-proxy | head -3
ipvsadm -Ln     # dentro do no
```

Regra prática: até algumas centenas de Services, iptables é indistinguível. Acima de mil, migre para IPVS ou para um CNI com eBPF que dispense o kube-proxy por completo.

#### 5.5.3 EndpointSlices

O objeto `Endpoints` original guardava todos os IPs de um Service em um único recurso — o que, com 5.000 Pods, gerava objetos enormes reescritos a cada mudança. O `EndpointSlice` (`discovery.k8s.io/v1`) fatia isso em pedaços de até 100 endpoints. É o que você deve inspecionar hoje:

```bash
kubectl -n store-dev get endpointslice -l kubernetes.io/service-name=store-api
```

```
NAME              ADDRESSTYPE   PORTS   ENDPOINTS                        AGE
store-api-4t9xk   IPv4          8000    10.244.0.7,10.244.1.4,10.244.2.3   6m
```

> **Na prática:** um EndpointSlice vazio (`ENDPOINTS: <none>`) é o diagnóstico mais valioso da rede do Kubernetes. Significa que o selector do Service não casa com nenhum Pod **pronto** — ou as labels estão erradas, ou a `readinessProbe` está falhando.

---

### 5.6 CoreDNS: descoberta de serviço

Ninguém quer decorar `10.96.51.20`. O CoreDNS roda como um Deployment em `kube-system`, exposto pelo Service `kube-dns` (o nome ficou por compatibilidade), e traduz nomes de Service em ClusterIPs.

#### 5.6.1 O Corefile

```
.:53 {
    errors
    health {
        lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
        pods insecure
        fallthrough in-addr.arpa ip6.arpa
        ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
        max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}
```

Cada linha é um **plugin**, executados em cadeia. O `kubernetes` responde autoritativamente pelo domínio `cluster.local`; o `forward` manda o que não for do cluster para o resolver do nó (que no EKS é o Route 53 Resolver da VPC); o `cache` guarda respostas por 30s; o `reload` recarrega o Corefile automaticamente quando o ConfigMap muda.

```bash
kubectl -n kube-system get configmap coredns -o yaml
kubectl -n kube-system edit configmap coredns    # cuidado: erro aqui derruba o DNS do cluster
```

#### 5.6.2 FQDN e search domains

O nome completo de um Service é:

```
<service>.<namespace>.svc.<cluster-domain>
store-api.store-dev.svc.cluster.local
```

Mas dentro de um Pod você quase sempre escreve só `store-api`. Isso funciona por causa do `/etc/resolv.conf` que o kubelet injeta:

```
search store-dev.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

Ao resolver `store-api`, a libc tenta em ordem: `store-api.store-dev.svc.cluster.local`, `store-api.svc.cluster.local`, `store-api.cluster.local` e, por fim, `store-api.`. Daí decorrem dois efeitos importantes:

- **Nome curto só funciona dentro do mesmo namespace.** Um Pod em `store-prod` que pede `store-api` resolve `store-api.store-prod...` — não o de `store-dev`. Para cruzar namespaces, use no mínimo `store-api.store-dev`.
- **`ndots:5` é uma armadilha de performance.** Qualquer nome com menos de 5 pontos passa primeiro pelos search domains. Consultar `api.stripe.com` (2 pontos) gera 3 consultas fracassadas antes da que funciona. Em aplicações com muitas chamadas externas, isso quadruplica a carga do CoreDNS. A correção é usar o FQDN com **ponto final** — `api.stripe.com.` — ou reduzir `ndots` via `dnsConfig`.

#### 5.6.3 dnsPolicy e dnsConfig

| `dnsPolicy` | Comportamento |
|---|---|
| `ClusterFirst` | Padrão. Usa o CoreDNS; o que não for do cluster é encaminhado ao upstream |
| `ClusterFirstWithHostNet` | Igual, mas para Pods com `hostNetwork: true` (que senão herdariam o DNS do nó) |
| `Default` | Herda o `/etc/resolv.conf` do **nó** — ou seja, **não resolve nomes de Service** |
| `None` | Ignora tudo e usa exclusivamente o `dnsConfig` |

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api-dnstune
  namespace: store-dev
spec:
  dnsPolicy: ClusterFirst
  dnsConfig:
    options:
      - name: ndots
        value: "2"
      - name: single-request-reopen
  containers:
    - name: store-api
      image: docker.io/seuusuario/store-api:1.0
      ports:
        - containerPort: 8000
```

> **Atenção:** `dnsPolicy: Default` tem o nome mais infeliz da API do Kubernetes — ele **não** é o padrão. O padrão é `ClusterFirst`. Já vi horas perdidas nessa confusão.

Teste de DNS que você deve memorizar:

```bash
kubectl -n store-dev run dnsutils --rm -it --image=registry.k8s.io/e2e-test-images/agnhost:2.47 \
  --restart=Never -- nslookup store-api.store-dev.svc.cluster.local
```

```
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      store-api.store-dev.svc.cluster.local
Address 1: 10.96.51.20 store-api.store-dev.svc.cluster.local
```

---

### 5.7 Ingress: a porta de entrada HTTP

Um Service `LoadBalancer` cria um balanceador na nuvem — **um por Service**. Com 20 microsserviços, são 20 balanceadores e 20 contas. Pior: L4 não entende hosts nem caminhos, então não dá para publicar `loja.fiap.com.br/api` e `loja.fiap.com.br/` em backends diferentes.

O **Ingress** resolve isso sendo um roteador L7 compartilhado. Mas atenção ao ponto que mais confunde: **o objeto Ingress não faz nada sozinho**. Ele é apenas configuração declarativa. Quem lê esse objeto e implementa o roteamento é o **Ingress Controller**, um Deployment com um proxy dentro (nginx, Traefik, HAProxy) ou um controlador que programa um balanceador da nuvem (AWS Load Balancer Controller).

| Controller | Como funciona | Onde brilha |
|---|---|---|
| **ingress-nginx** | Pod com nginx; reescreve `nginx.conf` a cada mudança | Onipresente; addon do minikube; canary por peso e anotações ricas |
| **Traefik** | Proxy Go nativo em K8s | Configuração dinâmica, ACME/Let's Encrypt embutido |
| **AWS Load Balancer Controller** | Não hospeda proxy: programa um **ALB** real na AWS | EKS; WAF, ACM, autenticação Cognito/OIDC |

#### 5.7.1 Ingress com dois paths

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store
  namespace: store-dev
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
    - host: store.local
      http:
        paths:
          - path: /api(/|$)(.*)
            pathType: ImplementationSpecific
            backend:
              service:
                name: store-api
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: store-front
                port:
                  number: 80
```

O `pathType` é obrigatório e tem três valores:

| `pathType` | Semântica |
|---|---|
| `Exact` | Casamento exato, sensível a maiúsculas. `/api` **não** casa `/api/` |
| `Prefix` | Casamento por segmentos de caminho. `/api` casa `/api` e `/api/produtos`, mas **não** `/apix` |
| `ImplementationSpecific` | Delegado ao controller — no nginx, habilita expressões regulares |

O `rewrite-target` acima é o padrão para "montar" um serviço sob um prefixo: o grupo `(.*)` captura o que vem depois de `/api` e o `$2` reescreve a URL enviada ao backend. Sem ele, a `store-api` receberia `/api/produtos` e responderia 404, porque a rota FastAPI é `/produtos`.

> **Nota:** `rewrite-target` é uma **anotação**, e anotações são específicas de cada controller. Trocar de ingress-nginx para Traefik exige reescrever essas anotações. Essa fragmentação é a principal motivação da Gateway API.

#### 5.7.2 TLS

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout store.key -out store.crt \
  -subj "/CN=store.local/O=FIAP" \
  -addext "subjectAltName=DNS:store.local"

kubectl -n store-dev create secret tls store-tls \
  --cert=store.crt --key=store.key
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store-tls
  namespace: store-dev
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - store.local
      secretName: store-tls
  rules:
    - host: store.local
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: store-api
                port:
                  number: 80
```

O Secret precisa ser do tipo `kubernetes.io/tls`, com as chaves `tls.crt` e `tls.key`, e **estar no mesmo namespace do Ingress**. A terminação TLS acontece no controller; do controller para o Pod o tráfego segue em HTTP puro, salvo se um service mesh fizer mTLS.

O **cert-manager** automatiza esse ciclo: você instala o CRD `ClusterIssuer` apontando para o Let's Encrypt, acrescenta a anotação `cert-manager.io/cluster-issuer: letsencrypt-prod` ao Ingress, e ele emite, armazena no Secret e **renova** o certificado sozinho, resolvendo o desafio ACME. Em produção, é praticamente obrigatório.

#### 5.7.3 Gateway API: a sucessora

O Ingress tem limites estruturais: só HTTP/HTTPS, extensibilidade só por anotações, e nenhuma separação entre o papel de quem opera a infraestrutura e o de quem publica a aplicação. A **Gateway API** (`gateway.networking.k8s.io`, GA desde 2023 e hoje o caminho recomendado) reorganiza tudo em recursos com papéis definidos:

| Recurso | Responsável | Papel |
|---|---|---|
| `GatewayClass` | Provedor de infraestrutura | "Existe este tipo de gateway" |
| `Gateway` | Operador do cluster | "Escute na 443 com este certificado" |
| `HTTPRoute` / `GRPCRoute` / `TCPRoute` | Time da aplicação | "`/api` vai para store-api, 10% para canary" |

```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: store-api
  namespace: store-dev
spec:
  parentRefs:
    - name: store-gateway
      namespace: infra
  hostnames:
    - "store.local"
  rules:
    - matches:
        - path:
            type: PathPrefix
            value: /api
      backendRefs:
        - name: store-api
          port: 80
          weight: 90
        - name: store-api-canary
          port: 80
          weight: 10
```

Repare que o canary por peso — que no capítulo 4 exigia contagem de réplicas — aqui é um campo de primeira classe.

---

### 5.8 NetworkPolicy: firewall declarativo

**Por padrão, todo Pod fala com todo Pod.** Um Pod comprometido em `store-dev` alcança o `store-db` de `store-prod` sem obstáculo. A `NetworkPolicy` é o objeto que restringe isso.

Três propriedades são indispensáveis para não errar:

1. **É aditiva e permissiva.** Não existe regra de negação. O tráfego é permitido se **alguma** política o permitir.
2. **É seletiva.** Um Pod que não é alvo de nenhuma política aceita tudo. Assim que **uma** política o seleciona para uma direção (`Ingress` ou `Egress`), ele passa a negar tudo naquela direção exceto o que for explicitamente liberado.
3. **Depende do CNI.** Flannel puro **ignora** NetworkPolicies silenciosamente — o objeto é aceito pela API e não faz nada. Use Calico ou Cilium.

#### 5.8.1 Default-deny

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: store-dev
spec:
  podSelector: {}          # todos os Pods do namespace
  policyTypes:
    - Ingress
    - Egress
```

`podSelector: {}` seleciona todos; a ausência de `ingress` e `egress` significa "nenhuma origem/destino permitido". Isso corta inclusive o **DNS**, então o próximo manifesto é obrigatório:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns
  namespace: store-dev
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

> **Atenção:** esquecer o DNS depois de aplicar default-deny é o erro nº 1 com NetworkPolicy. O sintoma é enganoso — a aplicação não reporta "bloqueado", reporta `Name or service not known` ou timeout de resolução, o que leva o aluno a caçar problema no CoreDNS.

#### 5.8.2 Liberação seletiva

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-front-to-api
  namespace: store-dev
spec:
  podSelector:
    matchLabels:
      app: store-api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: store-front
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8000
```

Há aqui uma sutileza de YAML que derruba muita gente. Compare:

```yaml
  # A: OR — Pods com app=store-front (deste namespace) OU qualquer Pod do namespace X
  from:
    - podSelector:
        matchLabels: {app: store-front}
    - namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: store-prod}

  # B: AND — Pods com app=store-front QUE ESTEJAM no namespace store-prod
  from:
    - podSelector:
        matchLabels: {app: store-front}
      namespaceSelector:
        matchLabels: {kubernetes.io/metadata.name: store-prod}
```

Um hífen a mais transforma E em OU. No caso A são dois itens da lista (união); no B, um item com dois seletores (interseção).

E o egress para o banco:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-egress
  namespace: store-dev
spec:
  podSelector:
    matchLabels:
      app: store-api
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: store-db
      ports:
        - protocol: TCP
          port: 5432
    - to:
        - ipBlock:
            cidr: 0.0.0.0/0
            except:
              - 10.0.0.0/8
              - 172.16.0.0/12
              - 192.168.0.0/16
      ports:
        - protocol: TCP
          port: 443
```

A segunda regra é um padrão útil: "pode chamar HTTPS na internet, mas não pode alcançar redes privadas" — mitigação clássica contra SSRF e movimentação lateral.

> **Nota:** a label `kubernetes.io/metadata.name` é aplicada automaticamente a todo Namespace pelo Kubernetes desde a 1.21. É a forma canônica de referenciar um namespace por nome em `namespaceSelector`, sem precisar rotulá-lo à mão.
>
> Note ainda que NetworkPolicy é **L3/L4 apenas**: ela filtra IP, protocolo e porta. Não sabe o que é um path HTTP ou um método. Para isso é preciso Cilium (CiliumNetworkPolicy com regras L7) ou um service mesh.

---

### 5.9 Namespaces: o que isolam e o que não isolam

Um Namespace é um **escopo de nomes** e um ponto de aplicação de políticas. Não é uma fronteira de segurança por si só.

**O que o namespace isola:**

- Nomes de objetos namespaced (dois Services `store-api` podem coexistir em `store-dev` e `store-prod`).
- Escopo padrão de `Role`/`RoleBinding` (RBAC).
- Escopo de `ResourceQuota` e `LimitRange`.
- Escopo padrão de nomes curtos no DNS.
- Ponto de aplicação do Pod Security Admission (por labels no namespace).

**O que o namespace NÃO isola:**

- **Rede.** Sem NetworkPolicy, qualquer Pod fala com qualquer Pod de qualquer namespace. Este é o ponto crítico.
- **Nós.** Pods de namespaces diferentes rodam lado a lado na mesma máquina, compartilhando kernel.
- **Objetos cluster-scoped**: Nodes, PersistentVolumes, StorageClasses, ClusterRoles, CRDs.
- **Consumo de recursos**, salvo se houver `ResourceQuota`.
- **Serviços do control plane**, como o etcd.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: store-prod
  labels:
    kubernetes.io/metadata.name: store-prod
    env: prod
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/audit: restricted
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: quota-store-prod
  namespace: store-prod
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "30"
    services: "10"
    services.loadbalancers: "2"
    persistentvolumeclaims: "5"
    count/deployments.apps: "10"
---
apiVersion: v1
kind: LimitRange
metadata:
  name: limits-store-prod
  namespace: store-prod
spec:
  limits:
    - type: Container
      default:
        cpu: "500m"
        memory: 512Mi
      defaultRequest:
        cpu: "100m"
        memory: 128Mi
      max:
        cpu: "2"
        memory: 4Gi
      min:
        cpu: "50m"
        memory: 64Mi
```

A interação entre os dois é essencial e costuma cair em prova: **se existe uma `ResourceQuota` sobre `requests.cpu`, todo container do namespace é obrigado a declarar `requests.cpu`**, senão o Pod é rejeitado no admission com `must specify requests.cpu`. O `LimitRange` salva a equipe injetando os valores padrão automaticamente em quem esqueceu.

Outro ponto que confunde: quando a quota é estourada por um **Deployment**, o Deployment é aceito e o *ReplicaSet* é que falha. O erro não aparece em `kubectl get deploy`, e sim em:

```bash
kubectl -n store-prod describe replicaset store-api-6c9f7d5b84
```

```
Warning  FailedCreate  12s  replicaset-controller  Error creating: pods "store-api-6c9f7d5b84-" is forbidden:
exceeded quota: quota-store-prod, requested: requests.cpu=100m, used: requests.cpu=4, limited: requests.cpu=4
```

**Namespaces do sistema:** `default` (não use em produção), `kube-system` (control plane, CoreDNS, kube-proxy — não coloque aplicação aqui), `kube-public` (legível por não autenticados; guarda o `cluster-info`), `kube-node-lease` (Leases de heartbeat dos nós).

**Convenções de nomeação** que funcionam bem: `<produto>-<ambiente>` (`store-dev`, `store-prod`), ou `<time>-<produto>` em clusters multi-time. Evite um namespace por microsserviço — a granularidade certa é a fronteira de confiança e de cobrança.

---

### 5.10 Service mesh em panorama

Quando o número de serviços cresce, quatro necessidades aparecem em toda organização: mTLS entre todos os serviços, roteamento fino (canary por header, espelhamento de tráfego), resiliência (retry, timeout, circuit breaker) e observabilidade uniforme (métricas RED e tracing distribuído sem tocar no código). Um **service mesh** entrega isso movendo a lógica de rede para um proxy ao lado da aplicação.

| Aspecto | Istio | Linkerd |
|---|---|---|
| Proxy | Envoy (C++) | linkerd2-proxy (Rust, ultraleve) |
| Escopo | Amplo: L7 completo, multicluster, políticas ricas | Deliberadamente enxuto |
| Curva | Íngreme | Suave |
| Modo sem sidecar | **Ambient mesh** (ztunnel por nó + waypoint opcional) | Sim, em evolução |

O modelo **sidecar** injeta um container de proxy em cada Pod e redireciona todo o tráfego por ele — poderoso, mas cobra CPU/memória por Pod, aumenta o tempo de início e complica ordem de inicialização e Jobs (o sidecar não termina). O modelo **ambient** do Istio separa as camadas: um `ztunnel` por nó cuida de mTLS e L4, e só quem precisa de L7 ganha um *waypoint proxy*. O custo cai drasticamente.

> **Na prática:** service mesh é uma decisão de plataforma, não de aplicação, e raramente se justifica abaixo de 20-30 serviços. Antes disso, NetworkPolicy + Ingress + boas bibliotecas de retry resolvem 90% do problema com 10% da complexidade operacional.

---

### 5.11 Ponte com Serverless

Tudo o que discutimos sobre porta de entrada tem um análogo direto no mundo serverless — e a comparação ilumina decisões de arquitetura.

O **Amazon API Gateway** é, funcionalmente, o Ingress do Lambda: roteia por host e path, termina TLS com certificado do ACM, aplica autenticação (JWT, Cognito, Lambda Authorizer), rate limiting e cache. A diferença é o modelo de cobrança e de operação: você paga por **requisição** (aproximadamente US$ 1,00 por milhão no HTTP API), não por hora de proxy rodando, e não existe Pod de controller para manter. Um `ingress-nginx` em EKS custa a hora dos Pods do controller mais o NLB à frente — algo em torno de US$ 20-30/mês, independentemente de haver tráfego. Abaixo de alguns milhões de requisições por mês, o API Gateway é mais barato; acima, o Ingress vence com folga. Vale notar também que um **ALB** pode disparar funções Lambda diretamente como target group, o que permite arquiteturas híbridas: `/api/relatorios` vai para uma Lambda pesada e esporádica, `/api/produtos` vai para o Deployment da `store-api` no EKS — atrás do mesmo hostname e do mesmo certificado.

No **Knative Serving**, a rede é o coração do produto. Cada `Service` do Knative cria uma rota que passa por uma camada de ingress — historicamente `Istio`, hoje mais comumente **Kourier** (um Envoy enxuto) ou `Contour`. O componente decisivo é o **Activator**: quando a revisão está em zero réplicas, o roteamento aponta para o Activator, que **segura a requisição em buffer**, sinaliza ao autoscaler que precisa de capacidade e só encaminha quando o primeiro Pod fica Ready. É por isso que o scale-to-zero do Knative não gera erro 503 — ele gera **latência**. O cold start aqui tem duas parcelas: agendar e iniciar o Pod (segundos, dominado pelo pull da imagem) e inicializar o runtime da aplicação.

Duas lições de rede se destacam. Primeira: **keep-alive é inimigo do escalonamento a zero**. Conexões HTTP persistentes mantêm o `conntrack` apontando para um Pod específico, o que atrapalha tanto a distribuição (vimos no canary do cap. 4) quanto a drenagem para zero. Por isso Knative e KEDA privilegiam métricas de concorrência sobre conexões abertas. Segunda: **DNS é caminho crítico e frequentemente esquecido**. Uma função que resolve `store-api.store-dev.svc.cluster.local` a cada invocação, com `ndots:5`, gera três consultas fracassadas antes da certa — e em escala serverless, com milhares de invocações curtas, o CoreDNS vira o gargalo. Cache local (NodeLocal DNSCache) e FQDN com ponto final são mitigações padrão.

---

### 5.12 Laboratório guiado (feito em sala)

**Objetivo:** criar `store-dev` e `store-prod` com quotas, publicar `store-front` e `store-api` sob o host `store.local` com paths distintos, e aplicar default-deny liberando só o necessário.

#### Passo 1 — cluster com CNI que suporta NetworkPolicy

```bash
minikube start --cni=calico --cpus=2 --memory=4096 --driver=docker
kubectl -n kube-system get pods -l k8s-app=calico-node
minikube addons enable ingress
kubectl -n ingress-nginx get pods
```

```
NAME                                       READY   STATUS      RESTARTS   AGE
ingress-nginx-admission-create-l7v2k        0/1     Completed   0          62s
ingress-nginx-controller-7d9c8b5f4-mq3xz    1/1     Running     0          62s
```

#### Passo 2 — namespaces, quota e limit range

Salve como `namespaces.yaml` (adapte o bloco da seção 5.9 para os dois namespaces) e aplique:

```bash
kubectl apply -f namespaces.yaml
kubectl get ns --show-labels
kubectl -n store-dev describe quota
```

```
Name:            quota-store-dev
Namespace:       store-dev
Resource         Used  Hard
--------         ----  ----
limits.cpu       0     8
limits.memory    0     16Gi
pods             0     30
requests.cpu     0     4
requests.memory  0     8Gi
```

#### Passo 3 — subir store-api e store-front

Use o Deployment do capítulo 4 para a `store-api` em `store-dev`. Para o `store-front`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-front
  namespace: store-dev
  labels:
    app: store-front
    tier: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: store-front
  template:
    metadata:
      labels:
        app: store-front
        tier: frontend
        env: dev
    spec:
      containers:
        - name: nginx
          image: nginx:1.27-alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: store-front
  namespace: store-dev
spec:
  selector:
    app: store-front
  ports:
    - port: 80
      targetPort: 80
```

```bash
kubectl apply -f store-front.yaml -f store-api-deployment.yaml
kubectl -n store-dev get pods,svc,endpointslice
```

#### Passo 4 — Ingress com dois paths

Aplique o `store-ingress.yaml` da seção 5.7.1 e registre o host:

```bash
kubectl apply -f store-ingress.yaml
kubectl -n store-dev get ingress
```

```
NAME    CLASS   HOSTS         ADDRESS          PORTS   AGE
store   nginx   store.local   192.168.49.2     80      35s
```

```bash
echo "$(minikube ip) store.local" | sudo tee -a /etc/hosts
curl -s http://store.local/            | head -5
curl -s http://store.local/api/produtos
```

```
[{"id":1,"nome":"Teclado","preco":199.9}]
```

Se `ADDRESS` ficar vazio por mais de um minuto, o controller ainda não processou o objeto — verifique `kubectl -n ingress-nginx logs deploy/ingress-nginx-controller`.

#### Passo 5 — TLS autoassinado

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout store.key -out store.crt \
  -subj "/CN=store.local/O=FIAP" -addext "subjectAltName=DNS:store.local"
kubectl -n store-dev create secret tls store-tls --cert=store.crt --key=store.key
kubectl apply -f store-ingress-tls.yaml
curl -k -s https://store.local/api/produtos
curl -sI http://store.local/api/produtos | head -1
```

```
HTTP/1.1 308 Permanent Redirect
```

O 308 confirma o `ssl-redirect`. O `-k` é necessário porque o certificado é autoassinado.

#### Passo 6 — testar a conectividade ANTES de restringir

```bash
kubectl -n store-dev run tester --image=curlimages/curl --restart=Never -- sleep 3600
kubectl -n store-dev exec tester -- curl -s -m 3 http://store-api/produtos
kubectl -n store-dev exec tester -- curl -s -m 3 -o /dev/null -w "%{http_code}\n" http://store-front/
```

Ambos respondem. Registre isso — é a linha de base.

#### Passo 7 — default-deny e liberação seletiva

```bash
kubectl apply -f netpol-default-deny.yaml
kubectl -n store-dev exec tester -- curl -s -m 3 http://store-api/produtos || echo "BLOQUEADO (esperado)"
```

```
curl: (28) Resolving timed out after 3000 milliseconds
BLOQUEADO (esperado)
```

Repare: falhou no **DNS**, não na conexão. Aplique agora `allow-dns`, `allow-front-to-api` e a política que libera o Ingress Controller:

```bash
kubectl apply -f netpol-allow.yaml
kubectl -n store-dev get networkpolicy
curl -s http://store.local/api/produtos          # de fora, via Ingress: deve funcionar
kubectl -n store-dev exec tester -- curl -s -m 3 http://store-api/produtos || echo "BLOQUEADO"
```

O acesso via Ingress volta a funcionar; o `tester` (que não tem `app: store-front`) continua bloqueado. Prove que o critério é a label:

```bash
kubectl -n store-dev label pod tester app=store-front
kubectl -n store-dev exec tester -- curl -s -m 3 http://store-api/produtos
```

Agora funciona. Esse experimento demonstra, de forma inequívoca, que **NetworkPolicy opera sobre labels, não sobre nomes ou IPs**.

#### Passo 8 — limpeza

```bash
kubectl delete ns store-dev store-prod
sudo sed -i '/store.local/d' /etc/hosts
minikube delete
```

---

### 5.13 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| Service existe mas conexão dá timeout | EndpointSlice vazio: selector não casa ou readiness falha | `kubectl get endpointslice -l kubernetes.io/service-name=<svc>` mostra `<none>` | Alinhar `spec.selector` do Service com as labels do Pod; corrigir a readinessProbe |
| `nslookup store-api` falha, mas o IP funciona | Nome curto usado de outro namespace, ou CoreDNS fora do ar | `kubectl -n kube-system get pods -l k8s-app=kube-dns`; testar com FQDN | Usar `store-api.store-dev.svc.cluster.local`; reiniciar/corrigir o CoreDNS |
| DNS lento; muita consulta NXDOMAIN | `ndots:5` fazendo o resolver tentar os search domains primeiro | `kubectl exec <pod> -- cat /etc/resolv.conf`; métricas do CoreDNS | FQDN com ponto final, ou `dnsConfig` com `ndots: 2`; NodeLocal DNSCache |
| Ingress criado, `ADDRESS` permanece vazio | Nenhum controller instalado, ou `ingressClassName` errado | `kubectl get ingressclass`; logs do controller | `minikube addons enable ingress`; corrigir `ingressClassName: nginx` |
| Ingress responde 404 no path certo | `pathType` inadequado ou falta de `rewrite-target` | `kubectl describe ingress`; logs do nginx mostram a URI repassada | Ajustar `pathType`/regex e a anotação de rewrite |
| Erro de certificado no navegador | Secret ausente, no namespace errado, ou tipo diferente de `kubernetes.io/tls` | `kubectl get secret store-tls -o yaml`; `kubectl describe ingress` | Recriar o Secret no namespace do Ingress com `create secret tls` |
| NetworkPolicy aplicada e nada muda | CNI sem suporte (Flannel puro) | `kubectl get pods -n kube-system` para identificar o CNI | Recriar o cluster com `--cni=calico` ou instalar Cilium |
| Tudo quebra após aplicar default-deny | Egress de DNS (porta 53) bloqueado | Erro é de resolução (`Resolving timed out`), não de conexão | Aplicar a política `allow-dns` para `k8s-app: kube-dns` em `kube-system` |
| Política libera mais/menos do que o esperado | Confusão entre AND e OR na lista `from` | Ler atentamente os hífens do YAML | Um item com dois seletores = AND; dois itens = OR |
| Pod rejeitado com `must specify requests.cpu` | `ResourceQuota` ativa e container sem `requests` | `kubectl -n <ns> describe quota`; `describe replicaset` | Declarar `resources.requests` ou criar um `LimitRange` com `defaultRequest` |
| Deployment READY 0/3 sem evento no Deployment | Quota estourada — o erro está no ReplicaSet | `kubectl describe rs <nome>` → `exceeded quota` | Elevar a quota ou reduzir requests/réplicas |
| Pods no mesmo Pod não sobem; porta em uso | Dois containers do mesmo Pod na mesma porta | `kubectl logs` → `address already in use` | Alterar a porta de um dos containers |

---

### 5.14 Resumo do capítulo

A rede do Kubernetes é uma pilha de camadas com responsabilidades bem separadas. O modelo exige IP por Pod e comunicação sem NAT; o **CNI** implementa isso (com overlay ou IPs reais da VPC); o **kube-proxy** transforma ClusterIPs em regras de DNAT no kernel, com balanceamento por conexão; o **CoreDNS** dá nomes a esses IPs virtuais e injeta search domains que fazem o nome curto funcionar dentro do namespace; o **Ingress** — na verdade, o Ingress Controller — concentra o roteamento HTTP L7 com host, path e TLS em um único ponto de entrada; a **NetworkPolicy** transforma a rede plana e permissiva em uma rede segmentada por labels; e o **Namespace** organiza nomes e políticas, isolando cotas e RBAC, mas jamais a rede por conta própria.

Pontos-chave:

1. **Não há mapeamento de portas em Pods.** `containerPort` é documentação; cada Pod tem IP próprio e portas exclusivas.
2. **ClusterIP é virtual.** Não responde a ping; existe apenas como regra de iptables/IPVS programada pelo kube-proxy a partir das EndpointSlices.
3. **EndpointSlice vazio é o diagnóstico mais rápido** de "Service não funciona": selector errado ou readiness falhando.
4. **Nome curto de DNS só resolve dentro do mesmo namespace**; entre namespaces use no mínimo `servico.namespace`, e prefira FQDN em código crítico.
5. **O objeto Ingress é inerte sem um Ingress Controller**, e as anotações que o configuram não são portáveis — motivo pelo qual a Gateway API existe.
6. **NetworkPolicy é aditiva, seletiva e dependente do CNI**; default-deny sem liberar DNS derruba tudo, e um hífen a mais troca AND por OR.
7. **Namespace não isola rede.** Separar `store-dev` de `store-prod` só é real quando há NetworkPolicy — e, idealmente, clusters distintos para produção.

---

### 5.15 Exercícios (para casa)

#### Exercício 5.1 — Namespaces com ResourceQuota e LimitRange, e a prova do bloqueio

**Tempo estimado:** 30 min · *Fixação guiada*

- **Objetivo:** criar namespaces governados e comprovar experimentalmente o comportamento de quota e limit range.
- **Enunciado:** crie os namespaces `store-dev` e `store-prod` com as labels `env: dev|prod` e `pod-security.kubernetes.io/enforce: baseline`. Em `store-prod`, aplique uma `ResourceQuota` com `requests.cpu: "1"`, `requests.memory: 2Gi`, `pods: "5"` e `count/deployments.apps: "3"`, e um `LimitRange` com `defaultRequest` de `100m`/`128Mi` e `max` de `500m`/`512Mi`. Depois execute e documente **quatro experimentos**: (a) crie um Pod da `store-api` **sem** declarar `resources` e mostre, com `kubectl get pod -o yaml`, que o LimitRange injetou os valores; (b) crie um Deployment com `replicas: 12` e mostre onde a mensagem de erro aparece (dica: não é no Deployment); (c) crie um Pod pedindo `requests.cpu: "3"` e capture o erro do LimitRange; (d) mostre a evolução de `Used` em `kubectl describe quota` antes e depois. Repita o experimento (a) em `store-dev`, que **não** deve ter LimitRange, e explique a diferença.
- **Entregável:** `e51-namespaces.yaml`, `e51-quota.yaml`, `e51-limitrange.yaml`, `e51-experimentos.md` com os quatro comandos e as saídas reais coladas.
- **Critério de aceite:** o Pod do item (a) em `store-prod` apresenta `requests.cpu: 100m` no YAML resultante sem que isso conste no manifesto original; o item (b) produz a mensagem `exceeded quota: ... requested: ..., used: ..., limited: ...` visível em `kubectl -n store-prod describe rs <nome>`; o item (c) é rejeitado no `apply` com mensagem citando `maximum cpu usage per Container`; `kubectl -n store-prod describe quota` mostra `pods` com `Used` maior que zero e menor ou igual a 5.
- <details><summary>Dica</summary>Quota é aplicada no admission, então quem falha é o objeto que **cria o Pod** — o ReplicaSet, não o Deployment. Use `kubectl get events -n store-prod --sort-by=.lastTimestamp` para ver tudo de uma vez.</details>

#### Exercício 5.2 — Ingress com dois paths e TLS autoassinado

**Tempo estimado:** 45 min · *Aplicação*

- **Objetivo:** publicar dois serviços sob um único host com roteamento por path e terminação TLS.
- **Enunciado:** em `store-dev`, suba `store-api` (2 réplicas) e `store-front` (2 réplicas) com seus Services ClusterIP. Habilite o addon `ingress` do minikube. Gere um certificado autoassinado para o host `fiapstore.local` com SAN correspondente e crie um Secret `kubernetes.io/tls`. Escreva **um único** Ingress que: (i) roteie `/` para `store-front` com `pathType: Prefix`; (ii) roteie `/api/...` para `store-api` na porta 80, reescrevendo o caminho de forma que uma requisição a `/api/produtos` chegue à aplicação como `/produtos`; (iii) termine TLS com o Secret criado; (iv) redirecione HTTP para HTTPS. Registre `fiapstore.local` no `/etc/hosts`. Valide com `curl` nos quatro casos: `http://fiapstore.local/` (espera 308), `https://fiapstore.local/`, `https://fiapstore.local/api/produtos` e `https://fiapstore.local/api/health`. Por fim, mostre com `kubectl describe ingress` que ambas as regras estão registradas e capture, nos logs do controller, a linha correspondente a uma das requisições, identificando o upstream escolhido.
- **Entregável:** `e52-ingress.yaml`, `e52-tls.sh` (geração do certificado e criação do Secret), `e52-validacao.md` com as quatro saídas de `curl -i` e o trecho de log do controller.
- **Critério de aceite:** `curl -sI http://fiapstore.local/api/produtos` retorna `308`; `curl -k -s https://fiapstore.local/api/produtos` retorna o JSON `[{"id":1,"nome":"Teclado","preco":199.9}]`; `curl -k -s https://fiapstore.local/` retorna a página do nginx; `kubectl -n store-dev get ingress` mostra `ADDRESS` preenchido com o IP do minikube e a coluna `PORTS` com `80, 443`.
- <details><summary>Dica</summary>Para o rewrite no ingress-nginx: `path: /api(/|$)(.*)` com `pathType: ImplementationSpecific` e a anotação `nginx.ingress.kubernetes.io/rewrite-target: /$2`. Se `/api/produtos` retornar 404 da FastAPI, o rewrite não está sendo aplicado — confira a regex e o número do grupo.</details>

#### Exercício 5.3 — Depuração: três formas de quebrar a comunicação entre Pods

**Tempo estimado:** 45 min · *Análise / depuração*

- **Objetivo:** desenvolver método de diagnóstico em camadas para falhas de rede.
- **Enunciado:** para cada um dos três cenários abaixo, quebre deliberadamente, diagnostique e produza um laudo com: sintoma, **camada** onde está o defeito (CNI, kube-proxy/Service, DNS ou NetworkPolicy), comando que revelou a causa com saída colada, causa raiz e correção.
  - **Cenário A — selector errado:** altere o `spec.selector` do Service `store-api` para `app: store-apii`. O Service continua existindo e com ClusterIP. Explique por que `curl` dá timeout em vez de "connection refused" e identifique o comando que mostra o problema em um segundo.
  - **Cenário B — DNS errado:** de um Pod em `store-prod`, tente acessar `http://store-api/produtos` (nome curto) esperando alcançar a API que está em `store-dev`. Capture o erro, explique o mecanismo de search domains que causou a falha e corrija usando duas alternativas diferentes (FQDN e um Service do tipo `ExternalName`).
  - **Cenário C — NetworkPolicy:** aplique uma política que pretende liberar o tráfego do `store-front` para a `store-api`, mas escrita com o erro do hífen (dois seletores em itens separados quando o desejado era AND). Mostre que ela libera mais do que o pretendido criando um Pod "intruso" em outro namespace que consegue acessar a API. Corrija para AND e reprove o acesso do intruso, mantendo o legítimo.
- **Entregável:** `e53-laudo.md` com os três laudos, os manifestos quebrados (`e53-a-quebrado.yaml`, etc.) e os corrigidos.
- **Critério de aceite:** o laudo A cita a saída de `kubectl -n store-dev get endpointslice -l kubernetes.io/service-name=store-api` com `ENDPOINTS: <none>`; o laudo B mostra o conteúdo de `/etc/resolv.conf` do Pod e o erro de resolução, e apresenta as duas correções funcionando; no laudo C, antes da correção o Pod intruso recebe HTTP 200 e depois recebe timeout, enquanto o `store-front` recebe 200 nos dois momentos.
- <details><summary>Dica</summary>Método em camadas, sempre nesta ordem: (1) o Pod está Ready? (2) o EndpointSlice tem endereços? (3) o DNS resolve? (4) o IP direto do Pod responde? Se o IP direto funciona e o ClusterIP não, o problema é Service/kube-proxy; se nem o IP direto funciona, suspeite de NetworkPolicy ou CNI.</details>

#### Exercício 5.4 — Desafio: isolar store-dev de store-prod e documentar a matriz de conectividade

**Tempo estimado:** 60 min · *Desafio integrador*

- **Objetivo:** projetar, implementar e **comprovar** uma postura de rede zero-trust entre dois ambientes.
- **Enunciado:** com um cluster usando Calico, monte em `store-dev` e `store-prod` a mesma tripla de workloads: `store-front`, `store-api` e `store-db` (PostgreSQL 16, porta 5432). Implemente NetworkPolicies que satisfaçam **exatamente** este contrato, sem exceções a mais:
  1. Nenhum Pod de `store-dev` alcança qualquer Pod de `store-prod`, e vice-versa.
  2. Dentro de cada namespace: `store-front` → `store-api` na 8000 (permitido); `store-api` → `store-db` na 5432 (permitido); `store-front` → `store-db` (**negado**).
  3. Todos os Pods podem resolver DNS.
  4. O Ingress Controller (namespace `ingress-nginx`) alcança `store-front` e `store-api` de ambos os namespaces.
  5. `store-api` pode fazer egress HTTPS (443) para a internet, mas **não** para faixas privadas (`10/8`, `172.16/12`, `192.168/16`).
  6. `store-db` não faz egress algum, exceto DNS.

  Depois, **comprove** cada linha com testes de `curl` (ou `nc`) executados de dentro dos Pods, e monte uma **matriz de conectividade** completa em Markdown: linhas = origem (7 origens, incluindo o Ingress Controller e um Pod "intruso" no namespace `default`), colunas = destino (6 destinos), células = `OK` ou `BLOQUEADO`, com o comando usado. Inclua a saída bruta em um anexo. Como parte final, escreva de **200 a 300 palavras** respondendo: (i) quais dessas seis regras um service mesh com mTLS resolveria melhor do que NetworkPolicy, e por quê; (ii) o que muda nessa matriz se `store-api` for migrada para AWS Lambda atrás de um API Gateway — quais regras deixam de fazer sentido e quais controles as substituem (Security Group, IAM, VPC endpoint).
- **Entregável:** `e54-netpol/` com um arquivo YAML por política (nomes descritivos), `e54-matriz.md` com a matriz preenchida e a análise, e `e54-testes.sh` com o script de verificação automatizado.
- **Critério de aceite:** o script `e54-testes.sh` executa as 42 combinações e imprime um relatório em que **todas** as células batem com o contrato acima; especificamente, o Pod intruso em `default` recebe `BLOQUEADO` para os 6 destinos, `store-front` de `store-dev` recebe `BLOQUEADO` ao tentar a 5432 do `store-db` e `OK` na 8000 da `store-api`, e `store-db` falha ao tentar `curl https://example.com` mas resolve `nslookup kubernetes.default`.
- <details><summary>Dica</summary>Comece com um único `default-deny-all` por namespace + `allow-dns`, e vá liberando uma regra por vez, testando após cada aplicação. Para o teste de porta sem HTTP, use `nc -zv -w 3 store-db 5432`. Lembre que a política de egress da origem **e** a de ingress do destino precisam permitir o tráfego — ambas são avaliadas.</details>

---

### 5.16 Leitura recomendada

1. **[DOC]** Kubernetes — *Cluster Networking*, *Service*, *DNS for Services and Pods*. Disponível em: https://kubernetes.io/docs/concepts/services-networking/ — é a referência canônica para as quatro regras e para os search domains.
2. **[DOC]** Kubernetes — *Network Policies* e *Ingress*. Disponível em: https://kubernetes.io/docs/concepts/services-networking/network-policies/ — inclui a excelente coleção de receitas do repositório `ahmetb/kubernetes-network-policy-recipes`, citada na própria página.
3. **[DOC]** Gateway API. Disponível em: https://gateway-api.sigs.k8s.io/ — leia o guia de conceitos e a comparação direta com o Ingress.
4. **[LIVRO]** LUKSA, M. *Kubernetes in Action*. 2. ed. Manning, 2023 — os capítulos de Services e de segurança de rede detalham o caminho do pacote com diagramas muito claros.
5. **[ARTIGO]** AWS — *Amazon EKS Best Practices Guide: Networking*. Disponível em: https://docs.aws.amazon.com/eks/latest/best-practices/networking.html — essencial para entender o VPC CNI, o limite de Pods por instância e o esgotamento de IPs, temas do capítulo 6.

---

## Capítulo 6 — Criando um Cluster Kubernetes na Nuvem (operação com múltiplos nós)

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 5 (arquitetura, Pods/Services, Deployments, HPA, distribuição por zona, Ingress e NetworkPolicy), Docker do 1º semestre, conta AWS com permissão administrativa **ou** Docker instalado para o plano B com k3d · **Artefatos:** `cluster.yaml`, `store-api-eks.yaml`, `store-api-lb.yaml`, `k3d-cluster.yaml`, `runbook-destruicao.md`, `custos.md`

Até aqui o minikube foi um simulador honesto: aprendemos os objetos, os controladores e a rede sem gastar um centavo. Mas o minikube esconde exatamente aquilo que faz o Kubernetes valer a pena em produção — múltiplas zonas de disponibilidade, balanceadores de verdade, identidade IAM, autoscaling de infraestrutura e uma fatura no fim do mês. Neste capítulo saímos do laptop. Vamos entender o que uma nuvem assume quando você contrata um Kubernetes gerenciado, criar um cluster **Amazon EKS** declarativamente, publicar a `store-api` no **Amazon ECR**, distribuí-la em duas AZs, expô-la ao mundo com um balanceador — e, criticamente, **destruir tudo** antes que a fatura chegue.

> **Atenção — leia antes de qualquer comando:** este é o primeiro capítulo que **gasta dinheiro real**. Um cluster EKS custa aproximadamente US$ 0,10 por hora só de control plane, mesmo sem nenhum Pod rodando. Se você esquecer o cluster ligado por um fim de semana, a conta chega perto de US$ 10 só de control plane, mais nós, mais balanceadores, mais NAT Gateway. Leia a seção 6.10 (custos) **antes** da 6.7 (criação), e mantenha o runbook de destruição aberto em outra aba o tempo todo. Se você não tem cartão de crédito ou não quer arriscar, a seção 6.11 traz o **plano B completo com k3d**, que reproduz tudo o que importa pedagogicamente sem custo algum. Não há nenhum prejuízo acadêmico em fazer o capítulo pelo plano B.

---

### 6.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Comparar** Kubernetes gerenciado e self-managed, identificando com precisão qual parte da operação o provedor assume e qual continua sua.
2. **Identificar** os componentes da anatomia de um cluster EKS: control plane gerenciado, VPC, subnets públicas/privadas, security groups, ENIs, VPC CNI e addons.
3. **Construir** um cluster EKS a partir de um arquivo `cluster.yaml` declarativo, com duas AZs e um managed node group, e configurar o acesso com `aws eks update-kubeconfig`.
4. **Aplicar** o fluxo completo de publicação de imagem em Amazon ECR e de deploy da `store-api` distribuída por zona.
5. **Analisar** o modelo de custos de um cluster gerenciado, estimar o gasto de um laboratório e executar a destruição completa dos recursos.
6. **Construir** um cluster multi-nó local equivalente com `k3d`/`kind` como alternativa sem custo, reconhecendo suas limitações.

---

### 6.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: o que muda ao sair do laptop. Modelo de responsabilidade compartilhada |
| 2 | 15 min | Managed x self-managed; anatomia do EKS (VPC, subnets, SG, ENI, addons) |
| 3 | 15 min | Tipos de nó: managed node group, self-managed, Fargate, Karpenter |
| 4 | 10 min | Pré-requisitos, IAM, autenticação (aws-auth x Access Entries), OIDC/IRSA |
| 5 | 10 min | **Intervalo** |
| 6 | 10 min | Custos: tabela do laboratório, alertas de billing, runbook de destruição |
| 7 | 35 min | Laboratório guiado (6.13): cluster.yaml → ECR → deploy em 2 AZs → LoadBalancer → destruir |
| 8 | 10 min | Upgrade de cluster e node group; observabilidade; plano B com k3d |
| 9 | 5 min | Ponte com Serverless, fechamento e exercícios |

---

### 6.3 Managed x self-managed: quem carrega o pager

Existem três formas de ter um cluster Kubernetes em produção:

| Abordagem | Ferramentas | Você opera | Custo típico |
|---|---|---|---|
| **Self-managed do zero** | `kubeadm`, `kops`, Cluster API | Tudo: etcd, apiserver, certificados, upgrades, backup | Só as instâncias EC2 |
| **Gerenciado** | EKS, GKE, AKS | Nós, workloads, addons, rede da aplicação | Instâncias + taxa do control plane |
| **Gerenciado sem nós** | EKS Auto Mode, Fargate, GKE Autopilot | Apenas os workloads | Por Pod/vCPU-segundo |

O `kubeadm` é a ferramenta oficial para levantar um control plane à mão. Vale conhecer academicamente — é o que a certificação CKA cobra e o que revela o que realmente existe por baixo: gerar a CA do cluster, subir `etcd`, `kube-apiserver`, `kube-scheduler` e `kube-controller-manager` como *static Pods*, distribuir kubeconfigs, unir os nós com `kubeadm join`. O `kops` (Kubernetes Operations) automatiza isso na AWS provisionando também a infraestrutura.

Mas o que você assume ao operar seu próprio control plane é pesado e frequentemente subestimado:

- **etcd**: backup, restore testado, compactação, defrag, monitoramento de latência de disco (etcd é brutalmente sensível a `fsync` lento), quórum em 3 AZs.
- **Certificados**: dezenas deles, com validade de 1 ano, que expiram silenciosamente e derrubam o cluster de madrugada.
- **Upgrades**: sequência control plane → kubelet → addons, respeitando o *skew* suportado (kubelet pode estar no máximo 3 versões menores que o apiserver).
- **Alta disponibilidade**: balanceador na frente dos apiservers, eleição de líder, número ímpar de membros do etcd.

Quando você usa **EKS**, a AWS assume tudo isso. O que você recebe é um endpoint de apiserver com SLA de 99,95%, etcd replicado em três AZs, backups automáticos e upgrades de control plane em um clique. O que continua **inteiramente seu**:

- os nós de trabalho (patch do SO, versão do kubelet, rotação de AMI);
- os workloads, seus limites e sua segurança;
- rede da aplicação: Ingress, NetworkPolicy, DNS;
- RBAC e IAM;
- observabilidade, backup de dados persistentes;
- **a fatura**.

> **Nota:** o equivalente no Google Cloud é o **GKE**, cujo modo *Autopilot* vai além do EKS ao gerenciar também os nós; e na Azure é o **AKS**, que historicamente não cobra pelo control plane no tier gratuito (sem SLA). Os conceitos deste capítulo — node pool, CNI, addons, IAM/Workload Identity — transferem-se quase diretamente entre os três.

---

### 6.4 Anatomia do Amazon EKS

```
                    +-----------------------------------------+
                    |  Control Plane EKS (conta da AWS)       |
                    |  apiserver x N | etcd (3 AZs) | sched   |
                    +--------------------+--------------------+
                                         | ENIs cross-account na SUA VPC
        =================================|=================================
        VPC 10.0.0.0/16 (sua conta)      |
                                         |
  +-------- AZ us-east-1a --------+   +-------- AZ us-east-1b --------+
  | subnet publica  10.0.0.0/20   |   | subnet publica  10.0.16.0/20  |
  |   NAT GW, ALB/NLB             |   |   ALB/NLB                     |
  +-------------------------------+   +-------------------------------+
  | subnet privada  10.0.32.0/19  |   | subnet privada  10.0.64.0/19  |
  |   node ip-10-0-40-12          |   |   node ip-10-0-70-33          |
  |     Pod 10.0.40.55 (ENI sec.) |   |     Pod 10.0.70.91            |
  +-------------------------------+   +-------------------------------+
```

#### 6.4.1 Control plane gerenciado

Roda na conta da AWS, invisível para você — não existe instância EC2 do apiserver na sua lista. A comunicação com os seus nós acontece por **ENIs cross-account** que a AWS cria dentro da sua VPC (você as verá no console como "Amazon EKS <cluster>"). O endpoint do apiserver pode ser:

| Configuração | Acesso |
|---|---|
| Público (padrão) | `kubectl` de qualquer lugar, protegido por IAM + RBAC |
| Público + privado | `kubectl` de fora; tráfego dos nós fica interno |
| Privado apenas | Só de dentro da VPC/VPN — postura recomendada em produção |

#### 6.4.2 VPC, subnets e o desenho de rede

O padrão do `eksctl` cria uma VPC com subnets **públicas** (com rota para o Internet Gateway, onde ficam NAT Gateways e balanceadores) e **privadas** (saída via NAT, onde ficam os nós). Os nós em subnets privadas não têm IP público — quem fala com a internet por eles é o NAT Gateway, e ele é **o item mais caro e mais esquecido do laboratório** (cerca de US$ 0,045/hora + US$ 0,045/GB processado, por AZ).

Duas tags são obrigatórias para que o AWS Load Balancer Controller descubra onde criar balanceadores (o `eksctl` as aplica automaticamente):

```
kubernetes.io/role/elb           = 1     # subnets publicas  -> ALB/NLB internet-facing
kubernetes.io/role/internal-elb  = 1     # subnets privadas  -> balanceadores internos
```

#### 6.4.3 Security groups e ENIs

O EKS cria um *cluster security group* compartilhado que permite o tráfego entre control plane e nós, e entre nós. Como o **AWS VPC CNI** entrega ao Pod um IP real da subnet (via ENIs secundárias anexadas à instância), os security groups da instância se aplicam ao tráfego dos Pods — e é possível ir além com `SecurityGroupPolicy`, atribuindo SGs por Pod.

A consequência prática mais importante já foi antecipada no capítulo 5: **o número máximo de Pods por nó é limitado pelo tipo de instância**, não pela CPU ou memória.

| Instância | ENIs | IPs por ENI | Máx. de Pods (fórmula clássica) |
|---|---|---|---|
| `t3.small` | 3 | 4 | 11 |
| `t3.medium` | 3 | 6 | 17 |
| `m5.large` | 3 | 10 | 29 |
| `m5.2xlarge` | 4 | 15 | 58 |

A fórmula é `(ENIs × (IPs por ENI − 1)) + 2`. Com `t3.medium`, dois DaemonSets (`aws-node` e `kube-proxy`) já consomem 2 dos 17 slots. Um Pod `Pending` cuja mensagem é `Too many pods` — e não `Insufficient cpu` — tem essa causa. A saída moderna é o **prefix delegation** do VPC CNI (`ENABLE_PREFIX_DELEGATION=true`), que aloca blocos /28 por ENI e eleva o limite da `t3.medium` de 17 para 110.

#### 6.4.4 Addons gerenciados

O EKS entrega o control plane, mas um cluster ainda precisa de componentes que rodam **nos seus nós**. Os addons gerenciados deixam a AWS cuidar da versão e do upgrade deles:

| Addon | Função | Obrigatório? |
|---|---|---|
| `vpc-cni` | Plugin CNI (rede dos Pods) | Sim |
| `coredns` | DNS do cluster | Sim, na prática |
| `kube-proxy` | Regras de Service | Sim |
| `aws-ebs-csi-driver` | Volumes EBS para PVCs | Sim, se usar PVC (cap. 7) |
| `aws-efs-csi-driver` | Volumes EFS (RWX) | Opcional |
| `eks-pod-identity-agent` | Identidade IAM por Pod (sucessor do IRSA) | Recomendado |
| `metrics-server` | Métricas para HPA e `kubectl top` | Não vem por padrão |

```bash
aws eks list-addons --cluster-name fiap-store
aws eks describe-addon --cluster-name fiap-store --addon-name vpc-cni
```

> **Atenção:** o `metrics-server` **não** é instalado por padrão no EKS. Se você criar o cluster e o HPA do capítulo 4 aparecer com `<unknown>`, é isso. Instale via Helm ou como addon.

---

### 6.5 Tipos de nó

| Tipo | Quem gerencia | Prós | Contras |
|---|---|---|---|
| **Managed node group** | AWS gerencia o ASG e o ciclo de vida da AMI | Upgrade coordenado com drain automático; simples | Menos flexível em customização de AMI |
| **Self-managed** | Você cria o ASG e o launch template | Controle total (AMI custom, GPU, bottlerocket ajustado) | Você faz drain e upgrade |
| **Fargate profile** | AWS: sem nós visíveis | Zero gestão de SO; isolamento por Pod (uma microVM cada) | Sem DaemonSet, sem privileged, sem GPU, sem armazenamento em bloco; mais caro por vCPU |
| **Karpenter** | Provisiona nós EC2 sob demanda | Escolhe o tipo ideal por Pod pendente; consolidação; spot inteligente | Componente extra a operar |

O **managed node group** é o ponto de partida certo para aprender e para a maioria das cargas. Você declara tipo de instância, mínimo/máximo/desejado, subnets e política de update; a AWS mantém um Auto Scaling Group e, no upgrade, faz `cordon` + `drain` respeitando seus PDBs (aqui o capítulo 4 se conecta diretamente: um PDB mal dimensionado **trava o upgrade do node group**).

O **Fargate** merece um parágrafo. Cada Pod vira uma microVM dedicada, faturada por vCPU-segundo e GB-segundo do que você **requisitou** (arredondado para cima em degraus). Não há nó para você atualizar. Em compensação, DaemonSets não funcionam (logo, agentes de log e de segurança precisam virar sidecars), não há volumes EBS, e o tempo de início de um Pod é de dezenas de segundos. É excelente para cargas esporádicas e para isolar workloads sensíveis; é caro para cargas contínuas e densas.

O **Karpenter**, já apresentado no capítulo 4, vem se tornando o padrão em EKS. Em vez de grupos de tipo fixo, ele lê os requisitos dos Pods `Pending` e lança a instância mais adequada e mais barata (frequentemente spot), em cerca de um minuto. E faz **consolidação**: identifica nós subutilizados, recoloca os Pods e desliga máquinas — o que exige PDBs bem escritos e `terminationGracePeriodSeconds` realista.

---

### 6.6 Pré-requisitos e autenticação

#### 6.6.1 Ferramentas

```bash
# AWS CLI v2
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o awscliv2.zip
unzip -q awscliv2.zip && sudo ./aws/install
aws --version            # aws-cli/2.x.x

# eksctl
curl -sSL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_Linux_amd64.tar.gz" \
  | tar xz -C /tmp && sudo mv /tmp/eksctl /usr/local/bin
eksctl version           # 0.19x.x

aws configure            # Access Key, Secret, regiao (us-east-1), formato (json)
aws sts get-caller-identity
```

```json
{
    "UserId": "AIDASAMPLEUSERID",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/aluno-fiap"
}
```

Se esse comando falhar, **pare**: nada mais vai funcionar.

#### 6.6.2 Permissões IAM

O `eksctl` usa CloudFormation e precisa criar VPC, subnets, roles, ASGs e o cluster. Em ambiente de aprendizado, use um usuário com `AdministratorAccess`; em ambiente corporativo, o mínimo envolve políticas sobre `eks:*`, `ec2:*` (limitado a tags), `iam:CreateRole`/`AttachRolePolicy`, `cloudformation:*` e `autoscaling:*`.

Dois papéis são criados automaticamente e vale saber para que servem:

- **Cluster role** (com `AmazonEKSClusterPolicy`): usada pelo control plane para gerenciar recursos da sua conta.
- **Node role** (com `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly` e `AmazonEKS_CNI_Policy`): usada pelas instâncias. É a `AmazonEC2ContainerRegistryReadOnly` que permite ao nó puxar imagens do ECR sem qualquer `imagePullSecret`.

#### 6.6.3 Autenticação: aws-auth x EKS Access Entries

O EKS separa **autenticação** (IAM: quem é você) de **autorização** (RBAC do Kubernetes: o que pode fazer). O mapeamento entre os dois mundos evoluiu:

**Modo legado — ConfigMap `aws-auth`:**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/eksctl-fiap-store-nodegroup-NodeInstanceRole
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/professor
      username: professor
      groups:
        - system:masters
```

Esse ConfigMap era o único caminho e sempre foi perigoso: um erro de sintaxe ou uma linha apagada **tranca todo mundo para fora do cluster**, incluindo você, sem forma de recuperação além de abrir chamado com a AWS.

**Modo atual — EKS Access Entries**, gerenciado por API:

```bash
aws eks create-access-entry \
  --cluster-name fiap-store \
  --principal-arn arn:aws:iam::123456789012:user/professor \
  --type STANDARD

aws eks associate-access-policy \
  --cluster-name fiap-store \
  --principal-arn arn:aws:iam::123456789012:user/professor \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSClusterAdminPolicy \
  --access-scope type=cluster

aws eks list-access-entries --cluster-name fiap-store
```

Vantagens: auditável no CloudTrail, gerenciável por IaC, sem risco de corromper um ConfigMap crítico. Clusters novos usam por padrão o modo `API_AND_CONFIG_MAP`. **Use Access Entries em qualquer cluster novo.**

#### 6.6.4 OIDC provider e IRSA (conceito)

Como um Pod chama a API da AWS (ler um bucket S3, publicar em uma fila SQS)? A resposta ruim é embutir chaves em um Secret. A resposta certa é **IRSA — IAM Roles for Service Accounts**.

Cada cluster EKS expõe um emissor OIDC. Ao registrá-lo como *identity provider* no IAM, o Pod passa a receber um token JWT projetado em um arquivo, assinado pelo cluster, cujo `sub` é `system:serviceaccount:<namespace>:<serviceaccount>`. O SDK da AWS troca esse token por credenciais temporárias via `sts:AssumeRoleWithWebIdentity`. Resultado: **credenciais de curta duração, escopo por ServiceAccount, zero segredos no cluster**.

```bash
eksctl utils associate-iam-oidc-provider --cluster fiap-store --approve

eksctl create iamserviceaccount \
  --cluster fiap-store \
  --namespace store-prod \
  --name store-api-sa \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --approve
```

O sucessor mais simples é o **EKS Pod Identity** (addon `eks-pod-identity-agent`), que dispensa o provider OIDC e usa uma associação direta ServiceAccount ↔ role. Aprofundaremos segurança e identidade no capítulo 9; por ora, guarde o princípio: **identidade por workload, nunca chave estática**.

---

### 6.7 Criando o cluster

#### 6.7.1 O caminho rápido (para conhecer, não para usar)

```bash
eksctl create cluster \
  --name fiap-store \
  --region us-east-1 \
  --version 1.34 \
  --nodegroup-name ng-app \
  --node-type t3.medium \
  --nodes 2 --nodes-min 2 --nodes-max 4 \
  --managed
```

Funciona, leva de 15 a 20 minutos e não é reprodutível: os parâmetros vivem no seu histórico de shell. Serve para conhecer o comando; não use em nada que precise ser repetido.

#### 6.7.2 O caminho declarativo: `cluster.yaml`

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: fiap-store
  region: us-east-1
  version: "1.34"
  tags:
    projeto: fiap-cloud-developer
    ambiente: laboratorio
    owner: aluno
    destruir-em: "2026-08-07"

vpc:
  cidr: "10.0.0.0/16"
  nat:
    gateway: Single          # Single (1 NAT) e mais barato que HighlyAvailable (1 por AZ)
  clusterEndpoints:
    publicAccess: true
    privateAccess: true

availabilityZones:
  - us-east-1a
  - us-east-1b

iam:
  withOIDC: true

addons:
  - name: vpc-cni
    version: latest
  - name: coredns
    version: latest
  - name: kube-proxy
    version: latest
  - name: metrics-server
    version: latest

managedNodeGroups:
  - name: ng-app
    instanceType: t3.medium
    minSize: 2
    maxSize: 4
    desiredCapacity: 2
    volumeSize: 20
    volumeType: gp3
    privateNetworking: true
    labels:
      workload: geral
      env: prod
    tags:
      k8s.io/cluster-autoscaler/enabled: "true"
      k8s.io/cluster-autoscaler/fiap-store: "owned"
    iam:
      withAddonPolicies:
        imageBuilder: true       # permissao de push/pull no ECR
        autoScaler: true
        cloudWatch: true
        ebs: true
    updateConfig:
      maxUnavailablePercentage: 33

cloudWatch:
  clusterLogging:
    enableTypes: ["api", "audit", "authenticator"]
    logRetentionInDays: 7
```

```bash
eksctl create cluster -f cluster.yaml
```

A execução leva de 15 a 20 minutos e imprime o progresso de duas stacks CloudFormation (a do cluster e a do node group). Ao final:

```
2026-08-06 14:12:03 [i]  waiting for CloudFormation stack "eksctl-fiap-store-cluster"
2026-08-06 14:24:51 [i]  creating managed nodegroup "ng-app"
2026-08-06 14:29:18 [i]  nodegroup "ng-app" has 2 node(s)
2026-08-06 14:29:19 [i]  node "ip-10-0-40-12.ec2.internal" is ready
2026-08-06 14:29:19 [i]  node "ip-10-0-70-33.ec2.internal" is ready
2026-08-06 14:29:22 [OK] EKS cluster "fiap-store" in "us-east-1" region is ready
```

> **Nota:** `nat.gateway: Single` economiza cerca de US$ 32/mês em relação a `HighlyAvailable`, ao custo de perder saída para a internet se aquela AZ cair. Para laboratório, é a escolha certa. Para produção, `HighlyAvailable`.
>
> Se algo falhar no meio, o `eksctl` deixa stacks CloudFormation órfãs que **continuam cobrando**. Sempre confira com `eksctl get cluster` e `aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE ROLLBACK_COMPLETE`.

#### 6.7.3 Configurando o kubeconfig

```bash
aws eks update-kubeconfig --region us-east-1 --name fiap-store
kubectl config current-context
kubectl get nodes -o wide
```

```
NAME                         STATUS   ROLES    AGE   VERSION               INTERNAL-IP   OS-IMAGE
ip-10-0-40-12.ec2.internal   Ready    <none>   6m    v1.34.0-eks-a1b2c3d   10.0.40.12    Amazon Linux 2023
ip-10-0-70-33.ec2.internal   Ready    <none>   6m    v1.34.0-eks-a1b2c3d   10.0.70.33    Amazon Linux 2023
```

```bash
kubectl get nodes -L topology.kubernetes.io/zone
```

```
NAME                         STATUS   ROLES    AGE   VERSION               ZONE
ip-10-0-40-12.ec2.internal   Ready    <none>   6m    v1.34.0-eks-a1b2c3d   us-east-1a
ip-10-0-70-33.ec2.internal   Ready    <none>   6m    v1.34.0-eks-a1b2c3d   us-east-1b
```

Um nó por AZ — agora o `topologySpreadConstraints` do capítulo 4 finalmente tem significado real. O `update-kubeconfig` não grava credenciais no arquivo: ele grava um bloco `exec` que chama `aws eks get-token` a cada operação, produzindo um token de curta duração assinado pelas suas credenciais IAM correntes.

---

### 6.8 Amazon ECR: registry privado

```bash
export AWS_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=us-east-1

aws ecr create-repository \
  --repository-name store-api \
  --region $AWS_REGION \
  --image-scanning-configuration scanOnPush=true \
  --image-tag-mutability IMMUTABLE

aws ecr get-login-password --region $AWS_REGION \
  | docker login --username AWS --password-stdin $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com

docker build -t store-api:1.0 .
docker tag store-api:1.0 $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com/store-api:1.0
docker push $AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com/store-api:1.0

aws ecr describe-images --repository-name store-api --region $AWS_REGION \
  --query 'imageDetails[].{Tag:imageTags[0],Size:imageSizeInBytes,Push:imagePushedAt}' --output table
```

Três decisões embutidas no comando de criação:

- **`scanOnPush=true`** dispara varredura de vulnerabilidades a cada push, sem custo extra no scanning básico.
- **`IMMUTABLE`** proíbe sobrescrever uma tag existente. É uma prática de segurança e de sanidade: `store-api:1.0` significa sempre o mesmo digest, e o `imagePullPolicy` deixa de importar.
- **Lifecycle policy** (recomendado) apaga imagens antigas automaticamente, evitando acumular custo de armazenamento.

#### 6.8.1 imagePullPolicy e tags

| Valor | Comportamento | Quando usar |
|---|---|---|
| `IfNotPresent` | Usa a imagem local se existir | Padrão para tags versionadas imutáveis |
| `Always` | Consulta o registry a cada criação de Pod | Tag `latest` ou mutável (necessário, mas evite a prática) |
| `Never` | Só usa local; falha se não houver | Imagens carregadas manualmente no nó (minikube/k3d) |

O padrão implícito é `Always` quando a tag é `latest` e `IfNotPresent` caso contrário. A recomendação profissional é **nunca usar `latest`** e, no limite, referenciar a imagem por digest: `store-api@sha256:9b2c...`.

Como a node role já traz `AmazonEC2ContainerRegistryReadOnly`, **não é preciso criar `imagePullSecret`** para o ECR da própria conta. Se o Pod falhar com `no basic auth credentials`, o problema está na role do nó, não no manifesto.

---

### 6.9 Expondo a aplicação: LoadBalancer e AWS Load Balancer Controller

Um Service `type: LoadBalancer` no EKS, sem controller adicional, é atendido pelo legado *in-tree* e cria um **CLB (Classic Load Balancer)**. Com as anotações certas, cria um **NLB**:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api-lb
  namespace: store-prod
  annotations:
    # anotacao do controlador in-tree (nao exige o AWS Load Balancer Controller)
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
spec:
  type: LoadBalancer
  selector:
    app: store-api
    tier: backend
  ports:
    - name: http
      port: 80
      targetPort: 8000
```

```bash
kubectl -n store-prod get svc store-api-lb
```

```
NAME           TYPE           CLUSTER-IP      EXTERNAL-IP                                          PORT(S)        AGE
store-api-lb   LoadBalancer   172.20.115.42   k8s-storepro-storeapi-3f2a1b.elb.us-east-1.amazonaws.com   80:31842/TCP   3m
```

O `EXTERNAL-IP` leva de 2 a 4 minutos para aparecer, e o DNS do balanceador pode levar mais um ou dois minutos para propagar. Um `EXTERNAL-IP` eternamente `<pending>` significa: faltam as tags nas subnets, faltam permissões IAM ou — se você usou as anotações do **AWS Load Balancer Controller** — ele não está instalado.

> **Atenção:** as anotações `aws-load-balancer-type: "external"`, `aws-load-balancer-nlb-target-type: "ip"` e `aws-load-balancer-scheme`, que você verá em muitos exemplos, pertencem ao **AWS Load Balancer Controller** e só funcionam com ele instalado (seção 6.9.1). Com o controlador in-tree, use `aws-load-balancer-type: "nlb"` como acima.

#### 6.9.1 NLB x ALB

| Critério | NLB (L4) | ALB (L7) |
|---|---|---|
| Objeto Kubernetes | `Service type: LoadBalancer` | `Ingress` |
| Camada | TCP/UDP | HTTP/HTTPS |
| Roteamento por host/path | Não | Sim |
| WAF, Cognito/OIDC | Não | Sim |
| IP estático / Elastic IP | Sim | Não |
| Latência | Menor | Maior (mas com recursos) |
| Custo | 1 por Service | 1 compartilhado por vários Ingresses |

A regra prática é a mesma do capítulo 5: **um NLB por Service não escala economicamente**. Instale o **AWS Load Balancer Controller** e use `Ingress` (ALB) como porta de entrada HTTP compartilhada.

```bash
eksctl create iamserviceaccount \
  --cluster fiap-store --namespace kube-system --name aws-load-balancer-controller \
  --attach-policy-arn arn:aws:iam::$AWS_ACCOUNT:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve

helm repo add eks https://aws.github.io/eks-charts && helm repo update
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system --set clusterName=fiap-store \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store
  namespace: store-prod
  annotations:
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/group.name: fiap-store
spec:
  ingressClassName: alb
  rules:
    - http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: store-api
                port:
                  number: 80
```

A anotação `group.name` é o que permite vários Ingresses de namespaces diferentes **compartilharem um único ALB** — a diferença entre US$ 16/mês e US$ 320/mês em um cluster com 20 serviços.

#### 6.9.2 Escalonamento de nós e upgrades

```bash
eksctl scale nodegroup --cluster fiap-store --name ng-app --nodes 3 --nodes-max 6
kubectl get nodes
```

O upgrade tem uma ordem obrigatória: **control plane primeiro, nós depois**, e no máximo uma versão menor por vez (1.33 → 1.34, nunca 1.33 → 1.35).

```bash
eksctl upgrade cluster --name fiap-store --version 1.34 --approve    # ~20-30 min
eksctl utils update-coredns    --cluster fiap-store --approve
eksctl utils update-kube-proxy --cluster fiap-store --approve
eksctl upgrade nodegroup --cluster fiap-store --name ng-app --kubernetes-version 1.34
```

O upgrade de um managed node group faz rolling: lança nós novos com a AMI atualizada, aplica `cordon` e `drain` nos antigos respeitando `updateConfig.maxUnavailablePercentage` **e os seus PodDisruptionBudgets**. Um PDB com `minAvailable` igual ao número de réplicas trava esse processo indefinidamente — o gancho direto com o capítulo 4.

A estratégia mais segura é o **blue/green de node group**: crie `ng-app-v2` com a versão nova, migre os workloads com `kubectl drain` nos nós antigos, valide, e só então apague `ng-app`. O rollback é trivial (recriar o antigo), a janela de risco é controlada, e o custo é a soma dos dois grupos durante a transição.

```bash
eksctl create nodegroup -f cluster-v2.yaml
kubectl drain <no-antigo> --ignore-daemonsets --delete-emptydir-data
eksctl delete nodegroup --cluster fiap-store --name ng-app --drain
```

#### 6.9.3 Observabilidade

```bash
kubectl top nodes
kubectl top pods -A --sort-by=cpu
```

```
NAME                         CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%
ip-10-0-40-12.ec2.internal   142m         7%     1204Mi          31%
ip-10-0-70-33.ec2.internal   98m          4%     1102Mi          28%
```

O **CloudWatch Container Insights** entrega dashboards de cluster, nó, Pod e container, além de logs centralizados. Instala-se como addon (`amazon-cloudwatch-observability`). Cuidado: **é uma linha relevante da fatura** — logs de container em alto volume podem custar mais que os nós. Em laboratório, mantenha `logRetentionInDays: 7` e desative Container Insights, usando `kubectl top` e `kubectl logs`.

---

### 6.10 Custos: a parte que ninguém pode pular

#### 6.10.1 Tabela do laboratório (us-east-1, valores de referência de 2026)

| Recurso | Quantidade | Preço unitário | Custo/hora | Custo em 4 h |
|---|---|---|---|---|
| Control plane EKS | 1 | US$ 0,10/h | US$ 0,100 | US$ 0,40 |
| EC2 `t3.medium` (on-demand) | 2 | US$ 0,0416/h | US$ 0,083 | US$ 0,33 |
| Volumes EBS gp3 20 GB | 2 | US$ 0,08/GB-mês | US$ 0,004 | US$ 0,02 |
| NAT Gateway (Single) | 1 | US$ 0,045/h + dados | US$ 0,045 | US$ 0,18 |
| NLB (Service LoadBalancer) | 1 | US$ 0,0225/h + LCU | US$ 0,025 | US$ 0,10 |
| ECR (armazenamento) | ~0,3 GB | US$ 0,10/GB-mês | ~US$ 0,000 | US$ 0,00 |
| Transferência de dados | ~1 GB | US$ 0,09/GB | — | US$ 0,09 |
| **Total** | | | **~US$ 0,26/h** | **~US$ 1,12** |

**Se você esquecer ligado:**

| Período | Custo estimado |
|---|---|
| 1 dia | ~US$ 6,20 |
| 1 fim de semana (3 dias) | ~US$ 18,60 |
| 1 semana | ~US$ 43,00 |
| 1 mês | ~US$ 187,00 |

> **Atenção:** o **control plane cobra mesmo com zero nós**. Escalar o node group para 0 **não** para a cobrança do cluster. A única forma de zerar é `eksctl delete cluster`. E o NAT Gateway cobra por hora **e** por GB processado — em um cluster que puxa imagens grandes, esse item surpreende.

#### 6.10.2 Antes de criar: alertas de billing

```bash
# orcamento de US$ 10 com alerta por e-mail em 80% do previsto
cat > budget.json <<'EOF'
{
  "BudgetName": "fiap-k8s-lab",
  "BudgetLimit": {"Amount": "10", "Unit": "USD"},
  "TimeUnit": "MONTHLY",
  "BudgetType": "COST"
}
EOF

cat > notifications.json <<'EOF'
[{
  "Notification": {
    "NotificationType": "FORECASTED",
    "ComparisonOperator": "GREATER_THAN",
    "Threshold": 80,
    "ThresholdType": "PERCENTAGE"
  },
  "Subscribers": [{"SubscriptionType": "EMAIL", "Address": "seu-email@exemplo.com"}]
}]
EOF

aws budgets create-budget \
  --account-id $(aws sts get-caller-identity --query Account --output text) \
  --budget file://budget.json \
  --notifications-with-subscribers file://notifications.json
```

Habilite também, no console de Billing, "Receive Free Tier Usage Alerts" e "Receive Billing Alerts". Leva dois minutos e evita histórias tristes.

#### 6.10.3 Runbook de destruição

```bash
# 1) apagar Services LoadBalancer ANTES do cluster (senao o ELB fica orfao e cobrando)
kubectl get svc -A --field-selector spec.type=LoadBalancer
kubectl -n store-prod delete svc store-api-lb

# 2) apagar Ingresses (ALBs criados pelo controller)
kubectl get ingress -A
kubectl -n store-prod delete ingress store

# 3) aguardar ~2 min e confirmar que nao ha balanceadores
aws elbv2 describe-load-balancers --query 'LoadBalancers[].LoadBalancerName' --output text
aws elb  describe-load-balancers  --query 'LoadBalancerDescriptions[].LoadBalancerName' --output text

# 4) destruir o cluster (10-15 min)
eksctl delete cluster -f cluster.yaml --wait

# 5) apagar o repositorio ECR
aws ecr delete-repository --repository-name store-api --region us-east-1 --force

# 6) VERIFICACAO FINAL — nada pode sobrar
eksctl get cluster --region us-east-1
aws ec2 describe-instances --filters "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].InstanceId' --output text
aws ec2 describe-nat-gateways --filter "Name=state,Values=available" \
  --query 'NatGateways[].NatGatewayId' --output text
aws ec2 describe-volumes --filters "Name=status,Values=available" \
  --query 'Volumes[].VolumeId' --output text
aws cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE ROLLBACK_COMPLETE DELETE_FAILED \
  --query 'StackSummaries[?contains(StackName,`eksctl`)].StackName' --output text
```

A ordem importa: se você apagar o cluster **antes** dos Services LoadBalancer, os balanceadores viram órfãos — o controller que os removeria não existe mais — e continuam cobrando por tempo indeterminado. O passo 6 é obrigatório: os seis comandos devem retornar vazio.

---

### 6.11 Plano B sem cartão de crédito: k3d/kind multi-nó

**Este plano B é oficialmente aceito na disciplina.** Nenhum exercício deste capítulo exige AWS: onde o enunciado disser EKS, leia k3d, e a análise de arquitetura vale igual.

O `k3d` roda o **k3s** (distribuição leve certificada pela CNCF) em containers Docker, com múltiplos nós e um balanceador na frente — o que reproduz razoavelmente bem a topologia de um cluster de nuvem.

```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
k3d version
```

`k3d-cluster.yaml`:

```yaml
apiVersion: k3d.io/v1alpha5
kind: Simple
metadata:
  name: fiap-store
servers: 1
agents: 3
image: rancher/k3s:v1.34.1-k3s1
ports:
  - port: 8080:80
    nodeFilters:
      - loadbalancer
  - port: 8443:443
    nodeFilters:
      - loadbalancer
registries:
  create:
    name: registro-local
    host: "0.0.0.0"
    hostPort: "5555"
options:
  k3s:
    extraArgs:
      - arg: --disable=traefik
        nodeFilters:
          - server:*
```

```bash
k3d cluster create -c k3d-cluster.yaml
kubectl get nodes
```

```
NAME                      STATUS   ROLES                  AGE   VERSION
k3d-fiap-store-server-0   Ready    control-plane,master   58s   v1.34.1+k3s1
k3d-fiap-store-agent-0    Ready    <none>                 52s   v1.34.1+k3s1
k3d-fiap-store-agent-1    Ready    <none>                 52s   v1.34.1+k3s1
k3d-fiap-store-agent-2    Ready    <none>                 51s   v1.34.1+k3s1
```

Simulando zonas de disponibilidade — é isso que torna o exercício equivalente:

```bash
kubectl label node k3d-fiap-store-agent-0 topology.kubernetes.io/zone=us-east-1a
kubectl label node k3d-fiap-store-agent-1 topology.kubernetes.io/zone=us-east-1b
kubectl label node k3d-fiap-store-agent-2 topology.kubernetes.io/zone=us-east-1a
kubectl get nodes -L topology.kubernetes.io/zone
```

Registry local no lugar do ECR:

```bash
docker build -t localhost:5555/store-api:1.0 .
docker push localhost:5555/store-api:1.0
# nos manifestos, use: image: registro-local:5000/store-api:1.0
```

Alternativa com `kind`:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: fiap-store
nodes:
  - role: control-plane
  - role: worker
    labels:
      topology.kubernetes.io/zone: us-east-1a
  - role: worker
    labels:
      topology.kubernetes.io/zone: us-east-1b
  - role: worker
    labels:
      topology.kubernetes.io/zone: us-east-1a
```

```bash
kind create cluster --config kind-cluster.yaml
kind load docker-image store-api:1.0 --name fiap-store    # sem registry
```

**Equivalências e limitações:**

| Conceito do EKS | Equivalente local | Fidelidade |
|---|---|---|
| Múltiplos nós | `agents: 3` | Alta |
| AZs | Labels `topology.kubernetes.io/zone` | Alta para agendamento, nula para falha real |
| ECR | Registry local do k3d ou `kind load` | Alta |
| Service LoadBalancer | `servicelb` do k3s (mapeia porta do host) | Média — recebe IP, mas não é um NLB |
| Ingress ALB | ingress-nginx | Média — sem WAF, ACM, `group.name` |
| IRSA | Não há equivalente | Nenhuma — apenas conceitual |
| Cluster Autoscaler | Não há | Nenhuma — simule com `k3d node create` |
| Custo | **US$ 0,00** | — |

```bash
k3d cluster delete fiap-store     # destruicao instantanea e gratuita
```

---

### 6.12 Ponte com Serverless

O EKS oferece dois modos de fugir da gestão de nós, e entender a diferença entre eles e o Lambda é o que separa uma decisão de arquitetura fundamentada de um chute.

**EKS on Fargate** mantém a API do Kubernetes intacta — você continua escrevendo Deployments, Services e HPAs — mas cada Pod roda em uma microVM Firecracker dedicada, provisionada sob demanda. Você seleciona quais Pods vão para lá com um *Fargate profile* (por namespace e labels):

```yaml
fargateProfiles:
  - name: fp-serverless
    selectors:
      - namespace: store-batch
        labels:
          compute: fargate
```

A cobrança é por **vCPU-segundo e GB-segundo do que foi requisitado**, arredondado para cima em degraus (um Pod com `requests.cpu: 250m` é faturado no degrau de 0,25 vCPU). Não há DaemonSets, não há volumes EBS, e o Pod leva de 30 a 60 segundos para iniciar. **ECS Fargate** é o primo sem Kubernetes: mesmo runtime, API muito mais simples, sem YAML de Pod — ótimo se você não precisa do ecossistema K8s.

**AWS Lambda** é outra categoria. Não há container de longa duração, não há nó, não há escalonamento configurável; há uma função invocada por evento, faturada em GB-milissegundo do tempo **realmente executado**, com escala de zero a milhares de execuções concorrentes em segundos.

| Dimensão | Node group EC2 | EKS/ECS Fargate | Lambda |
|---|---|---|---|
| Unidade de cobrança | Instância-hora | vCPU-s e GB-s requisitados | GB-ms executado |
| Ocioso custa? | Sim, 100% | Sim, enquanto o Pod existe | Não |
| Escala a zero | Só com KEDA/Knative | Só com KEDA/Knative | Nativo |
| Tempo de início | Segundos (nó já ligado) | 30–60 s | 100 ms – 3 s (cold start) |
| Duração máxima | Ilimitada | Ilimitada | 15 minutos |
| DaemonSet / GPU / EBS | Sim | Não | Não |

O raciocínio de custo é direto. Uma `t3.medium` custa cerca de US$ 30/mês e, bem empacotada, hospeda 10 Pods pequenos — US$ 3 por Pod-mês, rodando 24×7. Uma Lambda com 512 MB invocada 1 milhão de vezes por mês, com 200 ms cada, custa em torno de US$ 1,70. O ponto de equilíbrio depende inteiramente do **fator de ocupação**: com carga contínua, o nó ganha com folga porque você paga por capacidade que está sendo usada; com carga esporádica, a Lambda ganha porque o ocioso custa zero.

Aplicando à FIAP Store: a `store-api`, que recebe tráfego o dia inteiro, pertence a um node group com HPA — a receita dos capítulos 4 e 5. O `store-worker`, que processa pedidos em batch duas horas por dia, é candidato natural a KEDA com `minReplicaCount: 0` (se precisar do ecossistema K8s) ou a Lambda acionada por EventBridge/SQS (se for autocontido). E o processamento de relatórios mensais, que roda uma vez a cada 30 dias e leva 40 minutos, não cabe em Lambda (limite de 15 min): vira um `Job` do Kubernetes em nós spot, ou um AWS Batch. **A decisão é sempre função de três variáveis: duração, frequência e fator de ocupação.**

---

### 6.13 Laboratório guiado (feito em sala)

**Objetivo:** criar o cluster com `cluster.yaml`, publicar a `store-api` no ECR, deployar com 3 réplicas espalhadas em 2 AZs, expor com Service LoadBalancer, validar e destruir tudo.

> Se estiver no plano B, execute o Passo 0-B em vez dos Passos 1 e 2, use o registry local no Passo 3 e pule os Passos 6 e 8. Todo o resto é idêntico.

#### Passo 0 — verificações de segurança financeira

```bash
aws sts get-caller-identity
aws configure get region
# crie o budget da secao 6.10.2 AGORA, antes de qualquer recurso
date -u +"%Y-%m-%d %H:%M UTC"     # anote: este e o horario de inicio da cobranca
```

#### Passo 0-B — plano B com k3d

```bash
k3d cluster create -c k3d-cluster.yaml
kubectl label node k3d-fiap-store-agent-0 topology.kubernetes.io/zone=us-east-1a
kubectl label node k3d-fiap-store-agent-1 topology.kubernetes.io/zone=us-east-1b
kubectl label node k3d-fiap-store-agent-2 topology.kubernetes.io/zone=us-east-1a
kubectl get nodes -L topology.kubernetes.io/zone
```

#### Passo 1 — criar o cluster

Salve o `cluster.yaml` da seção 6.7.2 e execute:

```bash
eksctl create cluster -f cluster.yaml     # 15-20 min: use o tempo para a secao 6.10
```

#### Passo 2 — kubeconfig e mapeamento de zonas

```bash
aws eks update-kubeconfig --region us-east-1 --name fiap-store
kubectl get nodes -L topology.kubernetes.io/zone,node.kubernetes.io/instance-type
kubectl get pods -n kube-system
```

```
NAME                       READY   STATUS    RESTARTS   AGE
aws-node-4kd8x             2/2     Running   0          8m
aws-node-p2mn7             2/2     Running   0          8m
coredns-7f9c5d8b4-jl2vx    1/1     Running   0          12m
coredns-7f9c5d8b4-r8qkc    1/1     Running   0          12m
kube-proxy-9wc4t           1/1     Running   0          8m
kube-proxy-hx6rz           1/1     Running   0          8m
metrics-server-6d8f7-2xkzq 1/1     Running   0          7m
```

#### Passo 3 — construir e publicar no ECR

```bash
export AWS_ACCOUNT=$(aws sts get-caller-identity --query Account --output text)
export AWS_REGION=us-east-1
export ECR=$AWS_ACCOUNT.dkr.ecr.$AWS_REGION.amazonaws.com

aws ecr create-repository --repository-name store-api --region $AWS_REGION \
  --image-scanning-configuration scanOnPush=true
aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR

docker build -t store-api:1.0 .
docker tag store-api:1.0 $ECR/store-api:1.0
docker push $ECR/store-api:1.0
aws ecr describe-images --repository-name store-api --region $AWS_REGION --output table
```

#### Passo 4 — deploy distribuído em duas AZs

`store-api-eks.yaml`:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: store-prod
  labels:
    env: prod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-prod
  labels:
    app: store-api
    tier: backend
spec:
  replicas: 3
  revisionHistoryLimit: 3
  minReadySeconds: 10
  selector:
    matchLabels:
      app: store-api
      tier: backend
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: prod
    spec:
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: topology.kubernetes.io/zone
          whenUnsatisfiable: DoNotSchedule
          labelSelector:
            matchLabels:
              app: store-api
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: store-api
      containers:
        - name: store-api
          image: 123456789012.dkr.ecr.us-east-1.amazonaws.com/store-api:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8000
          env:
            - name: APP_ENV
              value: "prod"
            - name: DB_HOST
              value: "store-db.store-prod.svc.cluster.local"
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 5
            periodSeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 15
            periodSeconds: 10
---
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-prod
spec:
  type: ClusterIP
  selector:
    app: store-api
    tier: backend
  ports:
    - name: http
      port: 80
      targetPort: 8000
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-api
  namespace: store-prod
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: store-api
```

```bash
sed -i "s|123456789012.dkr.ecr.us-east-1.amazonaws.com|$ECR|" store-api-eks.yaml
kubectl apply -f store-api-eks.yaml
kubectl -n store-prod rollout status deployment/store-api
kubectl -n store-prod get pods -o wide
```

Prove a distribuição por zona:

```bash
for p in $(kubectl -n store-prod get pods -l app=store-api -o name); do
  no=$(kubectl -n store-prod get $p -o jsonpath='{.spec.nodeName}')
  az=$(kubectl get node $no -o jsonpath='{.metadata.labels.topology\.kubernetes\.io/zone}')
  echo "$p -> $no -> $az"
done
```

```
pod/store-api-6c9f7d5b84-2wq8n -> ip-10-0-40-12.ec2.internal -> us-east-1a
pod/store-api-6c9f7d5b84-hl4jd -> ip-10-0-70-33.ec2.internal -> us-east-1b
pod/store-api-6c9f7d5b84-tz6xk -> ip-10-0-40-12.ec2.internal -> us-east-1a
```

Com 3 réplicas e 2 zonas, o `maxSkew: 1` produz a divisão 2/1 — que é o esperado e o máximo de uniformidade possível.

#### Passo 5 — expor com Service LoadBalancer

```bash
kubectl apply -f store-api-lb.yaml           # manifesto da secao 6.9
kubectl -n store-prod get svc store-api-lb -w
```

Quando `EXTERNAL-IP` aparecer:

```bash
export LB=$(kubectl -n store-prod get svc store-api-lb -o jsonpath='{.status.loadBalancer.ingress[0].hostname}')
curl -s http://$LB/produtos
for i in $(seq 1 12); do curl -s http://$LB/health; echo; done
```

```
[{"id":1,"nome":"Teclado","preco":199.9}]
{"status":"ok","host":"store-api-6c9f7d5b84-2wq8n"}
{"status":"ok","host":"store-api-6c9f7d5b84-hl4jd"}
{"status":"ok","host":"store-api-6c9f7d5b84-tz6xk"}
...
```

Os três hostnames diferentes comprovam o balanceamento entre os Pods das duas AZs.

#### Passo 6 — observar e escalar

```bash
kubectl top nodes
kubectl top pods -n store-prod
kubectl -n store-prod scale deployment store-api --replicas=8
kubectl -n store-prod get pods -o wide | awk '{print $7}' | sort | uniq -c
eksctl scale nodegroup --cluster fiap-store --name ng-app --nodes 3
```

#### Passo 7 — validar a resiliência

```bash
kubectl -n store-prod scale deployment store-api --replicas=3
NODE=$(kubectl get nodes -o jsonpath='{.items[0].metadata.name}')
kubectl drain $NODE --ignore-daemonsets --delete-emptydir-data
# em outro terminal, mantenha o curl rodando: o servico NAO pode cair
while true; do curl -s -o /dev/null -w "%{http_code} " http://$LB/health; sleep 1; done
kubectl uncordon $NODE
```

Você deve ver uma sequência contínua de `200` — o PDB com `minAvailable: 2` e a distribuição por zona garantiram continuidade durante a drenagem.

#### Passo 8 — DESTRUIR TUDO

Execute o runbook completo da seção 6.10.3, **na ordem**, incluindo a verificação final. Anote o horário e calcule a duração real do laboratório para comparar com a estimativa.

```bash
date -u +"%Y-%m-%d %H:%M UTC"
```

No plano B: `k3d cluster delete fiap-store`.

---

### 6.14 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `eksctl create cluster` falha em ~10 min | Permissões IAM insuficientes ou limite de VPC/EIP da conta atingido | Eventos da stack no CloudFormation; `aws service-quotas list-service-quotas --service-code vpc` | Ampliar permissões; apagar VPCs não usadas; pedir aumento de cota |
| `kubectl` retorna `You must be logged in to the server (Unauthorized)` | Credenciais IAM diferentes das que criaram o cluster; sem Access Entry | `aws sts get-caller-identity`; `aws eks list-access-entries --cluster-name fiap-store` | `aws eks create-access-entry` + `associate-access-policy` com o ARN correto |
| `kubectl get nodes` vazio, mas as EC2 existem | Nós não conseguem registrar: sem rota para o apiserver, ou node role incorreta | `aws eks describe-nodegroup`; logs de user-data na instância | Verificar NAT Gateway/rotas, a node role e o security group |
| Pod `Pending` com `Too many pods` | Limite de IPs por ENI do tipo de instância | `kubectl describe node` (Allocatable `pods: 17`); `kubectl describe pod` | Instância maior, mais nós, ou `ENABLE_PREFIX_DELEGATION=true` no vpc-cni |
| Pod `Pending` com `Insufficient cpu` | Falta capacidade real no node group | `kubectl describe pod`; `kubectl top nodes` | `eksctl scale nodegroup`; instalar Cluster Autoscaler/Karpenter |
| Pod `Pending` com `didn't match topology spread constraints` | `DoNotSchedule` por zona com nós em uma única AZ | `kubectl get nodes -L topology.kubernetes.io/zone` | Adicionar nó na outra AZ ou usar `ScheduleAnyway` |
| `ImagePullBackOff` com `no basic auth credentials` | Node role sem `AmazonEC2ContainerRegistryReadOnly` | `kubectl describe pod`; `aws iam list-attached-role-policies --role-name <node-role>` | Anexar a policy à node role; conferir se a região do ECR bate |
| `ImagePullBackOff` com `manifest unknown` | Tag inexistente, ou imagem construída para arquitetura errada | `aws ecr describe-images`; `docker manifest inspect` | Republicar; usar `docker buildx build --platform linux/amd64` em Mac M1/M2/M3 |
| Service LoadBalancer com `EXTERNAL-IP <pending>` para sempre | Subnets sem as tags `kubernetes.io/role/elb`; sem permissão IAM; sem controller | `kubectl describe svc`; eventos com `failed to ensure load balancer` | Aplicar as tags nas subnets; corrigir a role; instalar o AWS LB Controller |
| Ingress `alb` criado, sem ADDRESS | AWS Load Balancer Controller ausente ou sem IRSA | `kubectl -n kube-system logs deploy/aws-load-balancer-controller` | Instalar o controller com a ServiceAccount IRSA correta |
| `eksctl delete cluster` falha | Balanceadores/ENIs órfãos segurando as subnets | Eventos `DELETE_FAILED` no CloudFormation | Apagar os Services LoadBalancer primeiro; remover ENIs órfãs; repetir o delete |
| Fatura maior que o esperado após destruir | NAT Gateway, EBS `available`, Elastic IPs ou ELBs órfãos | Cost Explorer por serviço; passo 6 do runbook | Apagar os recursos remanescentes um a um |
| Upgrade de node group nunca conclui | PDB impedindo o drain | `kubectl get pdb -A`; eventos do nodegroup | Ajustar `minAvailable` ou aumentar réplicas |

---

### 6.15 Resumo do capítulo

Saímos do laptop e enfrentamos o Kubernetes como ele é operado profissionalmente. Vimos que "gerenciado" significa uma divisão precisa de responsabilidades — a AWS assume etcd, apiserver, certificados e upgrades do control plane; você continua dono dos nós, dos workloads, da rede da aplicação, do IAM e da fatura. Dissecamos a anatomia do EKS: control plane fora da sua VPC conectado por ENIs, subnets públicas e privadas com suas tags obrigatórias, o VPC CNI entregando IPs reais aos Pods (com o limite de Pods por instância que isso acarreta) e os addons gerenciados. Criamos o cluster declarativamente com `cluster.yaml`, publicamos a imagem no ECR, distribuímos três réplicas por duas AZs com `topologySpreadConstraints`, expusemos com um NLB e comprovamos resiliência drenando um nó com o serviço no ar. Acima de tudo, tratamos custo como requisito de primeira classe: estimativa antes de criar, alerta de billing, runbook de destruição verificado — e um plano B em k3d que custa zero e ensina o mesmo.

Pontos-chave:

1. **Gerenciado não é mágico**: a AWS assume o control plane; nós, workloads, rede da aplicação, IAM e custo continuam seus.
2. **Sempre declarativo**: `cluster.yaml` versionado no Git; `eksctl create cluster` com flags é para demonstração, não para trabalho.
3. **O VPC CNI dá IPs reais da VPC aos Pods**, o que traz performance nativa e o limite de Pods por tipo de instância — a causa do `Pending` com `Too many pods`.
4. **Access Entries substituem o `aws-auth`**; IRSA (e o Pod Identity) substituem chaves estáticas dentro do cluster.
5. **Um NLB por Service não escala em custo**: use ALB via Ingress com `group.name` para compartilhar um balanceador.
6. **O control plane cobra mesmo ocioso**; zerar o node group não zera a fatura. Só `eksctl delete cluster` para a cobrança — e a ordem do runbook importa para não deixar órfãos.
7. **k3d/kind multi-nó reproduz nós, zonas simuladas, registry e distribuição** com fidelidade suficiente para todo o aprendizado deste capítulo, a custo zero.

---

### 6.16 Exercícios (para casa)

#### Exercício 6.1 — Criar o cluster e mapear os nós por zona

**Tempo estimado:** 45 min (EKS) / 25 min (k3d) · *Fixação guiada*

- **Objetivo:** provisionar um cluster multi-nó declarativamente e comprovar a distribuição da infraestrutura por zona.
- **Enunciado:** escolha uma das trilhas. **Trilha AWS:** escreva um `cluster.yaml` para um cluster chamado `<seunome>-store` em `us-east-1`, versão 1.34, com duas AZs (`us-east-1a` e `us-east-1b`), `nat.gateway: Single`, `withOIDC: true`, addons `vpc-cni`, `coredns`, `kube-proxy` e `metrics-server`, e um managed node group `ng-app` de `t3.medium` com min 2, max 4 e desired 2, em rede privada. **Antes de criar**, configure um AWS Budget de US$ 10 com alerta em 80% e anote o horário de início. Crie o cluster, configure o kubeconfig e execute a coleta descrita abaixo. **Trilha k3d:** crie um cluster com 1 server e 3 agents, aplique as labels `topology.kubernetes.io/zone` simulando duas AZs (2 nós em `us-east-1a`, 1 em `us-east-1b`) e siga a mesma coleta. Em ambas as trilhas, produza: (a) `kubectl get nodes -o wide`; (b) `kubectl get nodes -L topology.kubernetes.io/zone,node.kubernetes.io/instance-type`; (c) `kubectl get pods -n kube-system -o wide` identificando quais componentes são DaemonSets e por quê; (d) `kubectl describe node <um-no> | grep -A6 Allocatable`, apontando o valor de `pods` e explicando de onde vem esse número. Ao final da trilha AWS, **destrua o cluster** e registre o tempo total e o custo estimado.
- **Entregável:** `e61-cluster.yaml` (ou `e61-k3d.yaml`), `e61-coleta.md` com as quatro saídas coladas e comentadas, e `e61-custo.md` com horário de início, horário de destruição, duração e custo calculado item a item.
- **Critério de aceite:** `kubectl get nodes` mostra pelo menos 2 nós `Ready`; a coluna de zona apresenta **duas** zonas distintas; a explicação do item (d) cita a fórmula `(ENIs × (IPs por ENI − 1)) + 2` na trilha AWS (ou justifica o valor do k3s na trilha local); na trilha AWS, `eksctl get cluster` ao final retorna vazio e os cinco comandos de verificação do runbook não retornam recurso algum.
- <details><summary>Dica</summary>Deixe o runbook de destruição (6.10.3) aberto em outra aba **antes** de criar qualquer coisa. Se o `eksctl create` falhar no meio, verifique stacks CloudFormation órfãs — elas continuam cobrando recursos parcialmente criados.</details>

#### Exercício 6.2 — Publicar a imagem no registry e fazer o deploy a partir dele

**Tempo estimado:** 40 min · *Aplicação*

- **Objetivo:** dominar o fluxo build → registry → deploy no ambiente de nuvem (ou seu equivalente local).
- **Enunciado:** construa duas versões da `store-api`: a `1.0` com o código de referência da apostila e a `1.1` acrescentando o endpoint `/info` que retorna `APP_ENV`, `DB_HOST` e o hostname do Pod. **Trilha AWS:** crie o repositório ECR `store-api` com `scanOnPush=true` e `--image-tag-mutability IMMUTABLE`, faça login, publique as duas tags e liste as imagens com `aws ecr describe-images`. **Trilha k3d:** publique no registry local do cluster. Em seguida, crie o namespace `store-prod` e deploy a `1.0` com 3 réplicas, `resources.requests` de `100m`/`128Mi`, readiness em `/ready`, liveness em `/health` e `imagePullPolicy: IfNotPresent`. Valide com um Pod cliente interno. Depois faça `kubectl set image` para a `1.1`, acompanhe com `rollout status`, valide o novo endpoint `/info` e execute `rollout undo`, comprovando que `/info` volta a retornar 404. Finalmente, tente **republicar** a tag `1.0` com um conteúdo diferente e documente o erro obtido (na trilha AWS, a imutabilidade deve bloquear).
- **Entregável:** `Dockerfile`, `main.py` das duas versões, `e62-comandos.sh` com o fluxo completo, `e62-deploy.yaml`, e `e62-evidencias.md` com as saídas de `describe-images` (ou do registry local), do `rollout status`, da chamada a `/info` e do erro de republicação.
- **Critério de aceite:** o registry lista as duas tags com digests **diferentes**; `kubectl -n store-prod get deploy store-api` mostra `3/3` READY; a chamada a `/info` retorna JSON com `"env":"prod"` e um `pod` correspondente a um Pod real; após o `undo`, `/info` retorna 404; a tentativa de republicar `1.0` na trilha AWS falha com mensagem contendo `ImageTagAlreadyExistsException`.
- <details><summary>Dica</summary>Em Mac com chip Apple Silicon, use `docker buildx build --platform linux/amd64` — imagens ARM64 falham em nós x86 com `exec format error`, um erro que aparece só na execução e confunde bastante.</details>

#### Exercício 6.3 — Depuração: três falhas típicas de cluster em nuvem

**Tempo estimado:** 50 min · *Análise / depuração*

- **Objetivo:** diagnosticar falhas que só existem em ambiente de nuvem, distinguindo problema de Kubernetes de problema de infraestrutura/IAM.
- **Enunciado:** provoque, diagnostique e documente os três cenários. Para cada um, o laudo deve conter: sintoma, **camada** do problema (agendamento K8s / rede da nuvem / IAM), comando de diagnóstico com saída real colada, causa raiz e correção.
  - **Cenário A — Pod Pending por capacidade:** escale a `store-api` para 60 réplicas em um node group de 2 `t3.medium`. Colete `kubectl get pods | grep Pending | wc -l` e o `describe pod` de um Pending. **Distinga com precisão** se a mensagem é `Insufficient cpu` ou `Too many pods` — as causas e as correções são diferentes. Calcule quantos Pods cabem por nó pela fórmula de ENIs, compare com `Allocatable.pods` e proponha **duas** correções distintas (uma que muda a instância, outra que muda a configuração do CNI). Na trilha k3d, provoque o equivalente com `resources.requests.cpu: "1"` e 20 réplicas.
  - **Cenário B — LoadBalancer sem endereço:** crie um Service `type: LoadBalancer` e provoque a falha removendo a tag `kubernetes.io/role/elb` de uma subnet pública (ou, na trilha k3d, criando um segundo Service na mesma porta do host). Documente por quanto tempo o `EXTERNAL-IP` permanece `<pending>`, capture o evento em `kubectl describe svc` e explique por que o Kubernetes **não** reporta erro no `kubectl get svc`, apenas nos eventos.
  - **Cenário C — erro de IAM:** remova a policy `AmazonEC2ContainerRegistryReadOnly` da node role (ou, na trilha k3d, aponte para um registry privado sem credenciais) e crie um Pod com imagem do ECR. Capture o `ImagePullBackOff`, mostre a mensagem exata do `describe pod` e explique por que **não** é preciso `imagePullSecret` quando a role está correta. Restaure e comprove.
- **Entregável:** `e63-laudo.md` com os três laudos completos e os comandos de correção efetivamente aplicados.
- **Critério de aceite:** o laudo A apresenta o número exato de Pods `Pending`, o valor de `Allocatable.pods` do nó e duas correções tecnicamente distintas; o laudo B contém a saída de `kubectl describe svc` com um evento de falha do controlador de load balancer e o tempo medido; o laudo C mostra a mensagem de erro de autenticação no `describe pod` antes da correção e o Pod `Running` depois. Ao final, `kubectl get pods -n store-prod` não apresenta nenhum Pod em estado de erro.
- <details><summary>Dica</summary>`kubectl get events -A --sort-by=.lastTimestamp | tail -30` é o comando mais produtivo dos três cenários: eventos de LoadBalancer e de IAM aparecem lá antes de qualquer outro lugar. Lembre que eventos expiram em 1 hora por padrão.</details>

#### Exercício 6.4 — Desafio: cluster.yaml com spot + on-demand, relatório de custo e runbook

**Tempo estimado:** 60 min · *Desafio integrador (com componente serverless)*

- **Objetivo:** projetar uma topologia de cluster realista com otimização de custo e produzir a documentação operacional que a acompanha.
- **Enunciado:** escreva um `cluster.yaml` completo e comentado para a FIAP Store em produção, contemplando **todos** os itens:
  1. Cluster em `us-east-1`, versão 1.34, **três** AZs, `withOIDC: true`, endpoints público e privado, logging de `api`/`audit`/`authenticator` com retenção de 7 dias e tags de governança (`projeto`, `ambiente`, `owner`, `centro-custo`).
  2. Node group **`ng-critico`** on-demand: `m5.large`, min 2 / max 4 / desired 2, label `workload: critico`, para `store-api` e `store-db`.
  3. Node group **`ng-batch`** spot: `spot: true` com pelo menos **três** `instanceTypes` diferentes de capacidade equivalente (justifique por que a diversificação reduz o risco de interrupção), min 0 / max 10 / desired 1, label `workload: batch` e **taint** `workload=batch:NoSchedule`.
  4. Um `fargateProfile` para o namespace `store-batch` com o selector de labels apropriado.
  5. Addons: `vpc-cni`, `coredns`, `kube-proxy`, `aws-ebs-csi-driver`, `metrics-server`.
  6. Um `iamserviceaccount` para o `store-worker` no namespace `store-prod` com permissão de leitura em SQS.

  Escreva também os manifestos que **usam** essa topologia: o `store-worker` com `tolerations` para o taint de spot e `nodeSelector` `workload: batch`, e a `store-api` com `nodeSelector` `workload: critico`, `topologySpreadConstraints` pelas três AZs e um PDB coerente. Produza um **relatório de custo** (`custos.md`) com: tabela de custo/hora item a item (control plane, cada node group considerando o desconto spot típico de 60-70%, NAT, EBS, ALB, CloudWatch), projeção para 1 dia, 1 semana e 1 mês, e uma comparação de **três cenários** para o `store-worker` (2 h de trabalho por dia): (i) rodando 24×7 no `ng-critico`; (ii) no `ng-batch` spot com KEDA em `minReplicaCount: 0`; (iii) migrado para AWS Lambda acionada por SQS. Calcule o custo mensal dos três e recomende um, justificando com base em duração, frequência e fator de ocupação. Por fim, escreva um `runbook-destruicao.md` executável, na ordem correta, com comandos de **verificação** que provem que nada sobrou.
- **Entregável:** `e64-cluster.yaml` (comentado), `e64-workloads.yaml`, `custos.md` e `runbook-destruicao.md`.
- **Critério de aceite:** o `cluster.yaml` passa em `eksctl create cluster -f e64-cluster.yaml --dry-run` sem erro de schema (o dry-run não cria nada e não gera custo); o arquivo contempla os seis itens da lista; `custos.md` apresenta a tabela com valores numéricos por item, as três projeções temporais e os três cenários do `store-worker` com números e uma recomendação justificada; `runbook-destruicao.md` inclui a remoção dos Services LoadBalancer e Ingresses **antes** do `eksctl delete cluster` e ao menos cinco comandos de verificação final que devem retornar vazio.
- <details><summary>Dica</summary>`eksctl create cluster --dry-run` valida o schema e imprime o ClusterConfig normalizado sem criar recurso algum — use e abuse. Para o comparativo do `store-worker`, lembre que spot dá desconto sobre a EC2 mas o nó continua ligado; só o KEDA em zero e a Lambda eliminam o custo ocioso, e a Lambda tem o teto de 15 minutos por invocação, o que pode inviabilizar batches longos.</details>

---

### 6.17 Leitura recomendada

1. **[DOC]** AWS — *Amazon EKS User Guide*. Disponível em: https://docs.aws.amazon.com/eks/latest/userguide/ — comece por "Clusters", "Managed node groups" e "Cluster authentication" (Access Entries).
2. **[DOC]** eksctl — *Config file schema* e *Managed nodegroups*. Disponível em: https://eksctl.io/usage/schema/ — referência completa de todos os campos do `cluster.yaml`.
3. **[DOC]** AWS — *EKS Best Practices Guide* (Cost Optimization, Reliability, Networking). Disponível em: https://docs.aws.amazon.com/eks/latest/best-practices/ — leitura obrigatória antes de qualquer cluster real.
4. **[DOC]** k3d — *Documentation*. Disponível em: https://k3d.io/ — base do plano B; veja "Config file" e "Registries".
5. **[LIVRO]** ARUNDEL, J.; DOMINGUS, J. *Cloud Native DevOps with Kubernetes*. 2. ed. O'Reilly, 2022 — os capítulos sobre escolha de provedor gerenciado e sobre custo/otimização de cluster dialogam diretamente com este capítulo.
6. **[ARTIGO]** AWS — *Karpenter Documentation*. Disponível em: https://karpenter.sh/ — o caminho recomendado hoje para autoscaling de nós no EKS, alternativa ao Cluster Autoscaler.

---

## Capítulo 7 — Gerenciamento de Recursos (Volumes) e Daemons

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 6 (arquitetura do cluster, minikube, Pods/Services, Deployments/HPA, networking/namespaces e cluster EKS na nuvem); conceito de volume no Docker; noções de sistema de arquivos Linux. · **Artefatos:** `store-db-statefulset.yaml`, `store-api-config.yaml`, `store-api-deployment.yaml`, `fluent-daemonset.yaml`, `pvc-relatorios.yaml`, namespace `store-dev`.

---

### 7.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as consequências do sistema de arquivos efêmero de um container e explicar por que o Kubernetes precisa de uma abstração de volume própria, distinta da do Docker.
2. **Distinguir** os principais tipos de Volume (`emptyDir`, `hostPath`, `configMap`, `secret`, `downwardAPI`, `projected`, `nfs`, `csi`) e escolher o tipo adequado para cada cenário da FIAP Store.
3. **Aplicar** o modelo PersistentVolume / PersistentVolumeClaim / StorageClass, decidindo entre provisionamento estático e dinâmico e configurando `accessModes`, `reclaimPolicy` e `volumeBindingMode`.
4. **Construir** um StatefulSet com `volumeClaimTemplates` e headless Service, comprovando a estabilidade de identidade e a persistência dos dados após a destruição de Pods.
5. **Analisar** falhas de armazenamento (PVC `Pending`, `accessMode` incompatível, StorageClass inexistente, eviction por disco cheio) usando `kubectl describe` e eventos do cluster.
6. **Construir** um DaemonSet de coleta de logs e **comparar** essa abordagem com o modelo de observabilidade serverless gerenciado.

---

### 7.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: o container morreu, cadê meus dados? Demonstração do problema com `store-db` sem volume |
| 2 | 20 min | Tipos de Volume: `emptyDir`, `hostPath`, `configMap`, `secret`, `downwardAPI`, `projected`, `nfs`, `csi` |
| 3 | 20 min | PV, PVC e StorageClass: provisionamento estático x dinâmico, accessModes, reclaimPolicy, volumeBindingMode, expansão |
| — | 10 min | **Intervalo** |
| 4 | 10 min | CSI e drivers na prática: EBS CSI no EKS, EFS para RWX, `hostpath` no minikube |
| 5 | 15 min | StatefulSet: identidade estável, `volumeClaimTemplates`, headless Service, ordenação e atualização |
| 6 | 10 min | DaemonSet: casos de uso, tolerations, `updateStrategy`; garbage collection e `ownerReferences` |
| 7 | 20 min | **Laboratório guiado:** `store-db` como StatefulSet + ConfigMap montado como arquivo + DaemonSet de logs |
| 8 | 5 min | Ponte com Serverless, fechamento e apresentação dos exercícios |

---

### 7.3 O problema: o sistema de arquivos efêmero

Você já viu no primeiro semestre que um container Docker escreve em uma *camada writable* empilhada sobre as camadas somente-leitura da imagem. Essa camada vive e morre com o container. No Kubernetes o problema é o mesmo, porém amplificado por três fatores que não existiam quando você rodava `docker run` na sua máquina:

1. **Pods são descartáveis por projeto.** Um Deployment recria Pods a qualquer momento — em um rolling update, em um `kubectl delete pod`, quando o HPA reduz réplicas, quando o kubelet reinicia. Cada recriação começa com a camada writable zerada.
2. **Pods podem mudar de nó.** Se o nó falhar, o Pod é recriado em outro nó. Qualquer coisa gravada no disco local do nó anterior é inalcançável.
3. **Um Pod tem vários containers.** Precisamos de um mecanismo para compartilhar arquivos *entre* containers do mesmo Pod (o padrão sidecar), coisa que a camada writable individual não resolve.

Há ainda um detalhe sutil que confunde muito aluno: **reiniciar um container não é o mesmo que recriar um Pod**. Se o processo do container morrer e o `restartPolicy` for `Always`, o kubelet cria um *novo container* a partir da imagem, dentro do *mesmo Pod*. A camada writable do container antigo se perde, mas um volume declarado no Pod sobrevive. Vamos provar isso no laboratório.

```
   ┌────────────────── Pod ──────────────────┐
   │  container store-api   container sidecar│
   │   ┌──────────┐          ┌──────────┐    │
   │   │ camada RW│          │ camada RW│    │  <- some ao recriar o container
   │   └────┬─────┘          └─────┬────┘    │
   │        │  /dados              │ /dados  │
   │        └──────────┬───────────┘         │
   │             ┌─────▼──────┐              │
   │             │  Volume    │              │  <- vive enquanto o Pod viver
   │             └─────┬──────┘              │
   └───────────────────┼─────────────────────┘
                 ┌─────▼──────┐
                 │ PV (EBS,   │  <- vive além do Pod
                 │ EFS, NFS)  │
                 └────────────┘
```

O Volume do Kubernetes é, portanto, um **diretório acessível aos containers do Pod**, cujo conteúdo e ciclo de vida são determinados pelo *tipo* do volume. Ele é declarado uma vez em `spec.volumes` e montado por cada container em `spec.containers[].volumeMounts`. Essa separação em dois lugares é proposital: um mesmo volume pode ser montado em caminhos diferentes por containers diferentes.

> **Nota:** No Docker, `docker run -v` mistura três coisas (bind mount, volume nomeado e tmpfs) em uma sintaxe só. O Kubernetes separa cada uma em um tipo de volume distinto, e isso é uma melhoria — cada tipo tem regras de segurança e de portabilidade diferentes.

---

### 7.4 Tipos de Volume

#### 7.4.1 emptyDir — o rascunho do Pod

`emptyDir` cria um diretório vazio quando o Pod é agendado em um nó e o apaga **quando o Pod é removido do nó**. Sobrevive a reinícios de container; não sobrevive à morte do Pod. É o volume certo para cache, arquivos temporários de processamento e comunicação entre containers do mesmo Pod.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api-cache
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
spec:
  containers:
    - name: api
      image: docker.io/seuusuario/store-api:1.0
      ports:
        - containerPort: 8000
      volumeMounts:
        - name: cache
          mountPath: /var/cache/store
        - name: scratch-ram
          mountPath: /var/run/scratch
  volumes:
    - name: cache
      emptyDir:
        sizeLimit: 512Mi
    - name: scratch-ram
      emptyDir:
        medium: Memory
        sizeLimit: 64Mi
```

Com `medium: Memory` o Kubernetes monta um **tmpfs**: a escrita vai para a RAM, é muito mais rápida e nunca toca o disco. Duas consequências importantes e frequentemente esquecidas: (a) o conteúdo conta contra o **limite de memória do container**, ou seja, gravar 200 MiB em um tmpfs de um container com `limits.memory: 128Mi` provoca OOMKill; (b) tmpfs sempre se perde no reinício do nó. Use para dados sensíveis que não podem tocar o disco e para buffers de altíssima frequência.

#### 7.4.2 hostPath — poderoso e perigoso

`hostPath` monta um caminho do **sistema de arquivos do nó** dentro do Pod. É o equivalente ao bind mount do Docker.

```yaml
  volumes:
    - name: varlog
      hostPath:
        path: /var/log
        type: Directory
```

O campo `type` aceita `DirectoryOrCreate`, `Directory`, `FileOrCreate`, `File`, `Socket`, `CharDevice`, `BlockDevice` e vazio (sem verificação). Sempre declare `type` — sem ele, um erro de digitação no caminho cria silenciosamente um diretório vazio e o Pod sobe "funcionando" sem os dados.

> **Atenção:** `hostPath` é um dos maiores vetores de escalada de privilégio no Kubernetes. Montar `/` , `/etc`, `/var/run/docker.sock` ou `/var/lib/kubelet` dá ao Pod poder equivalente ao do nó inteiro. Políticas de Pod Security Admission no nível `baseline` e `restricted` **bloqueiam** `hostPath`. Use apenas em DaemonSets de infraestrutura (coleta de logs, agentes de monitoramento, plugins CSI) e nunca em aplicações de negócio.

Além do risco de segurança, `hostPath` não é portátil: os dados ficam presos a um nó específico. Um Pod reagendado para outro nó encontra um diretório diferente. Em cluster com um nó só (minikube) isso passa despercebido; em produção vira perda de dados.

#### 7.4.3 configMap e secret — configuração como arquivo

Ambos projetam chaves de um objeto do cluster como arquivos dentro do container. É a forma canônica de entregar arquivos de configuração (`.conf`, `.yaml`, `.properties`, certificados) sem reconstruir a imagem.

```yaml
  volumes:
    - name: config
      configMap:
        name: store-api-config
        items:
          - key: app.yaml
            path: app.yaml
            mode: 0444
```

Voltaremos a eles em detalhe na seção 7.8. `secret` funciona igual, mas o kubelet monta o conteúdo em **tmpfs** por padrão, para que segredos não sejam gravados no disco do nó.

#### 7.4.4 downwardAPI — o Pod olhando para si mesmo

Expõe metadados do próprio Pod (nome, namespace, labels, annotations, requests/limits) como arquivos. Útil para logs estruturados e para bibliotecas que precisam do nome do Pod sem depender de variáveis de ambiente.

```yaml
  volumes:
    - name: podinfo
      downwardAPI:
        items:
          - path: "labels"
            fieldRef:
              fieldPath: metadata.labels
          - path: "nome-do-pod"
            fieldRef:
              fieldPath: metadata.name
          - path: "limite-memoria"
            resourceFieldRef:
              containerName: api
              resource: limits.memory
              divisor: 1Mi
```

Diferença prática em relação a `env`: labels que mudam **são atualizadas** no arquivo do downwardAPI, enquanto variáveis de ambiente ficam congeladas no momento da criação do container.

#### 7.4.5 projected — vários em um só ponto de montagem

`projected` combina `configMap`, `secret`, `downwardAPI` e `serviceAccountToken` em **um único diretório**. É o mecanismo por trás do token moderno de ServiceAccount (que veremos no capítulo 9).

```yaml
  volumes:
    - name: tudo-junto
      projected:
        sources:
          - configMap:
              name: store-api-config
          - secret:
              name: store-db-secret
          - downwardAPI:
              items:
                - path: "namespace"
                  fieldRef:
                    fieldPath: metadata.namespace
          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
              audience: sts.amazonaws.com
```

#### 7.4.6 nfs e csi — armazenamento de rede

`nfs` monta um export NFS diretamente; funciona, mas está congelado em funcionalidades. O caminho moderno é **CSI (Container Storage Interface)**: uma especificação que permite a qualquer fornecedor de armazenamento escrever um driver fora da árvore de código do Kubernetes. Praticamente todos os volumes "in-tree" antigos (AWS EBS, GCE PD, Azure Disk) foram migrados para CSI via *CSI migration*, e os plugins in-tree foram removidos nas versões 1.26–1.31. Se você encontrar um manifesto com `awsElasticBlockStore:` na internet, é material desatualizado.

| Tipo | Ciclo de vida | Compartilhável entre Pods | Uso típico na FIAP Store |
|---|---|---|---|
| `emptyDir` | Vida do Pod | Só entre containers do mesmo Pod | Cache de imagens de produto, buffer do sidecar |
| `emptyDir` + `medium: Memory` | Vida do Pod (tmpfs/RAM) | Idem | Chaves temporárias, buffers rápidos |
| `hostPath` | Vida do nó | Sim, entre Pods do mesmo nó | Coleta de logs (`/var/log`) por DaemonSet |
| `configMap` | Vida do objeto | Sim (somente leitura) | `app.yaml` do `store-api` |
| `secret` | Vida do objeto (tmpfs) | Sim (somente leitura) | Senha do `store-db`, certificado TLS |
| `downwardAPI` | Vida do Pod | Não | Nome do Pod nos logs |
| `projected` | Vida do Pod | Não | Token IRSA + config + segredo |
| `nfs` / `csi` (EFS) | Independente do Pod | Sim (RWX) | Uploads compartilhados do `store-front` |
| `csi` (EBS) | Independente do Pod | Não (RWO) | Dados do PostgreSQL `store-db` |

---

### 7.5 PersistentVolume, PersistentVolumeClaim e StorageClass

Montar `nfs:` ou `csi:` direto no Pod acopla a aplicação à infraestrutura: o time de desenvolvimento precisaria saber o endereço do servidor NFS ou o ID do volume EBS. O Kubernetes resolve isso com uma indireção clássica em três objetos.

- **PersistentVolume (PV):** representa uma peça de armazenamento **no cluster**. É um recurso de escopo global (não pertence a namespace), normalmente criado pelo administrador ou automaticamente por um provisionador.
- **PersistentVolumeClaim (PVC):** é o **pedido** de armazenamento feito pela aplicação, dentro de um namespace. Diz "quero 10 GiB, RWO, da classe `gp3`". Não diz *onde*.
- **StorageClass (SC):** descreve uma "qualidade de serviço" de armazenamento e, principalmente, **qual provisionador** criar o volume quando alguém pede.

```
  Desenvolvedor                     Administrador / Provisionador
  ┌──────────┐   pede            ┌──────────────┐   cria      ┌───────┐
  │   Pod    │ ──────────────►   │     PVC      │ ──────────► │  PV   │
  │  volumes │   claimName       │  10Gi, RWO   │  binding    │ EBS   │
  └──────────┘                   └──────┬───────┘             └───────┘
                                        │ storageClassName
                                 ┌──────▼───────┐
                                 │ StorageClass │  provisioner: ebs.csi.aws.com
                                 └──────────────┘
```

#### 7.5.1 Provisionamento estático x dinâmico

No **estático**, alguém cria o PV manualmente e o PVC "encontra" um PV compatível (mesmo tamanho ou maior, mesmos accessModes, mesma classe). É o modelo usado com storage legado, NFS corporativo ou quando existe um volume pré-existente com dados.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-relatorios-nfs
spec:
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-corporativo
  nfs:
    server: 10.0.3.42
    path: /exports/fiapstore/relatorios
```

No **dinâmico**, o PVC referencia uma StorageClass e o provisionador cria o PV sob demanda. É o padrão em nuvem e o que usaremos daqui em diante.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  encrypted: "true"
  fsType: ext4
reclaimPolicy: Delete
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: relatorios-pvc
  namespace: store-dev
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: gp3
  resources:
    requests:
      storage: 2Gi
```

#### 7.5.2 accessModes

| Modo | Sigla | Significado |
|---|---|---|
| `ReadWriteOnce` | RWO | Montável em leitura/escrita por **um nó** por vez. Vários Pods no *mesmo* nó podem usá-lo. |
| `ReadOnlyMany` | ROX | Montável somente-leitura por vários nós. |
| `ReadWriteMany` | RWX | Leitura/escrita por vários nós simultaneamente. |
| `ReadWriteOncePod` | RWOP | Leitura/escrita por **exatamente um Pod** no cluster inteiro (K8s 1.29+ estável). |

O erro conceitual mais comum é ler RWO como "um Pod". É **um nó**. Se você precisa garantir exclusividade real — por exemplo, para um banco que não tolera dois processos no mesmo diretório de dados — use `ReadWriteOncePod`.

O que determina se um modo funciona é o **driver**, não o YAML. EBS é um disco de bloco anexado a uma única instância EC2: ele só faz RWO/RWOP, não importa o que você escreva no PVC. Para RWX na AWS use **EFS** (NFS gerenciado) via `efs.csi.aws.com`, ou FSx for Lustre em cargas de HPC.

#### 7.5.3 reclaimPolicy

Define o que acontece com o PV quando o PVC é apagado:

- **`Delete`** — o PV e o volume real (o EBS) são destruídos. Padrão do provisionamento dinâmico. Excelente para ambientes efêmeros, catastrófico se aplicado a um banco de produção.
- **`Retain`** — o PV vai para o estado `Released` e o dado permanece. Um administrador precisa limpar e liberar manualmente. É a política que você quer em `store-prod`.

Para tornar um volume existente à prova de exclusão acidental:

```bash
kubectl patch pv pvc-8c1f... -p '{"spec":{"persistentVolumeReclaimPolicy":"Retain"}}'
```

> **Na prática:** um padrão saudável é ter duas StorageClasses: `gp3` com `Delete` para `store-dev` e `gp3-retain` com `Retain` para `store-prod`. Assim a política vira decisão de infraestrutura, não de quem escreve o manifesto.

#### 7.5.4 volumeBindingMode

- **`Immediate`** — o PV é criado assim que o PVC aparece. Em cluster multi-AZ isso é um problema: o volume EBS pode nascer em `us-east-1a` e o scheduler colocar o Pod em `us-east-1c`, e aí o Pod fica preso em `Pending` para sempre com `volume node affinity conflict`.
- **`WaitForFirstConsumer`** — o provisionamento espera existir um Pod pendente que use o PVC; só então o volume é criado, **na zona onde o Pod foi agendado**. É o modo correto em qualquer cluster com mais de uma zona de disponibilidade.

O PVC fica em `Pending` com a mensagem `waiting for first consumer to be created before binding` — isso é comportamento esperado, não erro.

#### 7.5.5 Expansão de volume

Se a StorageClass tiver `allowVolumeExpansion: true`, basta editar o `resources.requests.storage` do PVC para um valor maior. Diminuir não é permitido.

```bash
kubectl patch pvc dados-store-db-0 -n store-dev \
  -p '{"spec":{"resources":{"requests":{"storage":"20Gi"}}}}'
kubectl get pvc dados-store-db-0 -n store-dev
```

```
NAME               STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
dados-store-db-0   Bound    pvc-8c1f...  10Gi       RWO            gp3            36m
```

A capacidade ainda mostra 10Gi porque a expansão do sistema de arquivos exige que o Pod seja reiniciado (a menos que o driver suporte expansão online, como o EBS CSI recente). Verifique a condição:

```bash
kubectl describe pvc dados-store-db-0 -n store-dev | grep -A3 Conditions
```

```
Conditions:
  Type                      Status  Reason               Message
  FileSystemResizePending   True    Waiting for user to (re-)start a pod to finish file system resize of volume on node.
```

---

### 7.6 CSI e drivers no mundo real

O CSI define uma interface gRPC com três serviços: *Identity*, *Controller* (criar/apagar/anexar volumes — roda como Deployment) e *Node* (montar no nó — roda como **DaemonSet**, e essa é a primeira ligação natural entre os dois temas deste capítulo).

No **EKS**, o EBS CSI Driver **não vem instalado por padrão**. Sem ele, todo PVC fica `Pending`. Instale como add-on gerenciado:

```bash
eksctl create addon --name aws-ebs-csi-driver \
  --cluster fiap-store \
  --service-account-role-arn arn:aws:iam::123456789012:role/AmazonEKS_EBS_CSI_DriverRole \
  --force

kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-ebs-csi-driver
```

```
NAME                                  READY   STATUS    RESTARTS   AGE
ebs-csi-controller-6d4f9c8b7d-2xk9p   6/6     Running   0          92s
ebs-csi-controller-6d4f9c8b7d-l7vwq   6/6     Running   0          92s
ebs-csi-node-4rt9m                    3/3     Running   0          92s
ebs-csi-node-pq2ls                    3/3     Running   0          92s
```

Repare: o *controller* é um Deployment com duas réplicas; o *node* é um DaemonSet, um Pod por nó.

Para RWX, instale o EFS CSI e crie a StorageClass com o modo de acesso de diretório:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-0a1b2c3d4e5f67890
  directoryPerms: "700"
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

No **minikube**, o addon `storage-provisioner` fornece a classe `standard` com o provisionador `k8s.io/minikube-hostpath`, que cria diretórios em `/tmp/hostpath-provisioner`. Funciona para aprender, mas repare: é `hostPath` disfarçado, portanto os dados vivem no nó do minikube e somem se você der `minikube delete`.

```bash
kubectl get storageclass
```

```
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  14d
```

> **Nota (outras nuvens):** no GKE a classe padrão é `standard-rwo` (`pd.csi.storage.gke.io`) e o RWX vem do Filestore; no AKS, `managed-csi` (`disk.csi.azure.com`) e `azurefile-csi` para RWX. O modelo PV/PVC/SC é idêntico; muda o `provisioner` e os `parameters`.

> **Atenção (custo):** volumes EBS e EFS **continuam sendo cobrados depois que você apaga os Pods**. Ao encerrar o laboratório, apague os PVCs (`kubectl delete pvc --all -n store-dev`) e confirme no console EC2 → Volumes que nada ficou órfão. Com `reclaimPolicy: Retain`, apagar o PVC **não** apaga o EBS: você paga até removê-lo à mão.

---

### 7.7 StatefulSet

Um Deployment trata seus Pods como gado: nomes aleatórios, intercambiáveis, todos ligados ao mesmo (ou a nenhum) volume. Um banco de dados replicado precisa do oposto — cada réplica tem identidade, disco próprio e ordem de inicialização. Esse é o papel do **StatefulSet**.

Quatro garantias:

1. **Nome estável e ordinal:** `store-db-0`, `store-db-1`, `store-db-2`. Se `store-db-1` morrer, o substituto se chama `store-db-1` de novo.
2. **Rede estável:** com um **headless Service** (`clusterIP: None`), cada Pod ganha um DNS previsível: `store-db-0.store-db.store-dev.svc.cluster.local`.
3. **Armazenamento estável:** `volumeClaimTemplates` gera um PVC por réplica (`dados-store-db-0`, `dados-store-db-1`, ...). O PVC **acompanha o ordinal**, não o Pod: o novo `store-db-1` remonta o disco do antigo.
4. **Ordem determinística:** criação e escala para cima em ordem crescente (0, 1, 2), esperando cada Pod ficar `Ready`; término e escala para baixo em ordem decrescente. Com `podManagementPolicy: Parallel` você abre mão dessa ordem em troca de velocidade.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-db
  namespace: store-dev
  labels:
    app: store-db
    tier: data
spec:
  clusterIP: None
  selector:
    app: store-db
  ports:
    - name: postgres
      port: 5432
      targetPort: 5432
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: store-db
  namespace: store-dev
spec:
  serviceName: store-db
  replicas: 1
  selector:
    matchLabels:
      app: store-db
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0
  template:
    metadata:
      labels:
        app: store-db
        tier: data
        env: dev
    spec:
      terminationGracePeriodSeconds: 30
      containers:
        - name: postgres
          image: postgres:16
          ports:
            - name: postgres
              containerPort: 5432
          env:
            - name: POSTGRES_DB
              value: fiapstore
            - name: POSTGRES_USER
              value: store
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: store-db-secret
                  key: POSTGRES_PASSWORD
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          volumeMounts:
            - name: dados
              mountPath: /var/lib/postgresql/data
          readinessProbe:
            exec:
              command: ["sh", "-c", "pg_isready -U store -d fiapstore"]
            initialDelaySeconds: 10
            periodSeconds: 5
          resources:
            requests:
              cpu: 100m
              memory: 256Mi
            limits:
              cpu: "1"
              memory: 1Gi
  volumeClaimTemplates:
    - metadata:
        name: dados
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 2Gi
```

> **Atenção:** `PGDATA: /var/lib/postgresql/data/pgdata` não é capricho. Volumes de bloco montados pelo CSI (e o `lost+found` do ext4) fazem o `initdb` do PostgreSQL falhar com *"directory exists but is not empty"*. Colocar os dados em um subdiretório resolve.

**Atualização.** O campo `partition` do `updateStrategy` permite canary em StatefulSet: com `partition: 2` em um conjunto de 3 réplicas, só `store-db-2` é atualizado; as réplicas 0 e 1 ficam na versão antiga até você baixar a partição. `OnDelete` desliga a atualização automática — o Pod só recebe a nova spec quando você o apaga manualmente, útil para bancos que exigem janela de manutenção.

**Exclusão.** Apagar o StatefulSet **não apaga os PVCs** por padrão — é uma proteção deliberada contra perda de dados. Desde a 1.27 existe `persistentVolumeClaimRetentionPolicy`:

```yaml
spec:
  persistentVolumeClaimRetentionPolicy:
    whenDeleted: Retain   # ou Delete
    whenScaled: Delete    # apaga o PVC ao reduzir réplicas
```

---

### 7.8 ConfigMap como volume de configuração

Já usamos ConfigMap com `envFrom` nos capítulos anteriores. Agora vamos montá-lo como **arquivo**, que é o modo certo quando a aplicação lê um `.yaml`/`.conf`.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: store-api-config
  namespace: store-dev
data:
  app.yaml: |
    app:
      nome: FIAP Store API
      ambiente: dev
      moeda: BRL
    catalogo:
      itens_por_pagina: 20
      cache_segundos: 30
  banner.txt: |
    Bem-vindo a FIAP Store (ambiente de desenvolvimento)
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
    spec:
      containers:
        - name: api
          image: docker.io/seuusuario/store-api:1.0
          ports:
            - containerPort: 8000
          env:
            - name: APP_ENV
              value: dev
            - name: DB_HOST
              value: store-db-0.store-db.store-dev.svc.cluster.local
          volumeMounts:
            - name: config
              mountPath: /etc/store
              readOnly: true
          resources:
            requests:
              cpu: 50m
              memory: 64Mi
            limits:
              cpu: 500m
              memory: 256Mi
              ephemeral-storage: 1Gi
      volumes:
        - name: config
          configMap:
            name: store-api-config
```

Isso cria `/etc/store/app.yaml` e `/etc/store/banner.txt`. Repare que montar um volume em `/etc/store` **substitui** todo o conteúdo original desse diretório na imagem. Se você precisar injetar **um único arquivo** dentro de um diretório que já tem conteúdo, use `subPath`:

```yaml
          volumeMounts:
            - name: config
              mountPath: /app/config/app.yaml
              subPath: app.yaml
```

> **Atenção — a armadilha do `subPath`:** um volume montado normalmente é atualizado automaticamente pelo kubelet quando o ConfigMap muda (em até ~1 minuto, o período de sincronização do kubelet + cache). Com `subPath`, **não é**: o arquivo é copiado uma vez e nunca mais atualizado. Você precisa reiniciar o Pod. Existe ainda um segundo detalhe: mesmo sem `subPath`, a aplicação só percebe a mudança se ela reler o arquivo — o FastAPI não recarrega sozinho. Por isso a prática mais confiável é forçar o rollout:

```bash
kubectl rollout restart deployment/store-api -n store-dev
```

Um padrão elegante usado em produção é anotar o Deployment com o hash do ConfigMap (feito automaticamente pelo Helm com `checksum/config`), de modo que qualquer mudança na configuração gere um novo `podTemplate` e, portanto, um rolling update.

Por fim, ConfigMaps marcados como `immutable: true` não podem ser alterados — o que reduz drasticamente a carga no API server em clusters grandes e elimina mudanças acidentais. Para alterar, cria-se um novo ConfigMap com outro nome e troca-se a referência.

---

### 7.9 DaemonSet

Enquanto o Deployment responde "quero N réplicas", o **DaemonSet** responde **"quero exatamente uma réplica em cada nó"** (ou em cada nó que casar com um `nodeSelector`/`affinity`). Quando um nó novo entra no cluster — por autoscaling do node group, por exemplo — o DaemonSet Controller cria o Pod ali automaticamente. Quando o nó sai, o Pod some junto.

Casos de uso clássicos, todos de infraestrutura:

| Categoria | Exemplos | Por que precisa ser por nó |
|---|---|---|
| Logs | fluent-bit, fluentd, vector, aws-for-fluent-bit | Lê `/var/log/pods` do nó local |
| Métricas | node-exporter, cAdvisor, Datadog Agent, CloudWatch Agent | Coleta métricas de CPU/disco/rede do nó |
| Rede (CNI) | aws-node (VPC CNI), calico-node, cilium | Configura interfaces e regras no nó |
| Armazenamento | `ebs-csi-node`, `efs-csi-node` | Monta volumes no sistema de arquivos do nó |
| Segurança | Falco, agentes de EDR | Instrumenta o kernel do nó |
| Proxy | `kube-proxy` | Programa iptables/IPVS do nó |

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: store-logs
  namespace: store-dev
  labels:
    app: store-logs
    tier: infra
spec:
  selector:
    matchLabels:
      app: store-logs
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 0
  template:
    metadata:
      labels:
        app: store-logs
        tier: infra
    spec:
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
        - key: node.kubernetes.io/disk-pressure
          operator: Exists
          effect: NoSchedule
      containers:
        - name: coletor
          image: busybox:1.36
          command:
            - /bin/sh
            - -c
            - |
              mkdir -p /coleta/fiapstore
              while true; do
                echo "$(date -Iseconds) node=${NODE_NAME} pods=$(ls /var/log/pods 2>/dev/null | wc -l)" \
                  >> /coleta/fiapstore/coleta.log
                sleep 15
              done
          env:
            - name: NODE_NAME
              valueFrom:
                fieldRef:
                  fieldPath: spec.nodeName
          volumeMounts:
            - name: varlogpods
              mountPath: /var/log/pods
              readOnly: true
            - name: coleta
              mountPath: /coleta
          resources:
            requests:
              cpu: 20m
              memory: 32Mi
            limits:
              cpu: 100m
              memory: 128Mi
      volumes:
        - name: varlogpods
          hostPath:
            path: /var/log/pods
            type: DirectoryOrCreate
        - name: coleta
          hostPath:
            path: /var/log/fiapstore
            type: DirectoryOrCreate
```

Três pontos merecem atenção.

**Tolerations.** Nós de control plane têm o taint `node-role.kubernetes.io/control-plane:NoSchedule`. Sem a toleration correspondente, o agente de log não roda ali — e você perde justamente os logs mais críticos. Em cluster gerenciado (EKS) você não vê os nós de control plane, mas a toleration é inofensiva e mantém o manifesto portátil. Também é comum tolerar `disk-pressure` e `memory-pressure`: o coletor de logs precisa continuar rodando *especialmente* quando o nó está com problemas.

**updateStrategy.** `RollingUpdate` com `maxUnavailable: 1` derruba um Pod por vez. `OnDelete` exige exclusão manual — usado para agentes sensíveis, em que você quer controlar a janela.

**hostPath somente-leitura.** Note `readOnly: true` no `/var/log/pods`. O agente precisa ler, nunca escrever. Restringir é gratuito e reduz o risco.

Verificação:

```bash
kubectl get daemonset store-logs -n store-dev
```

```
NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
store-logs   1         1         1       1            1           <none>          45s
```

Em cluster EKS com 3 nós, `DESIRED` seria 3.

---

### 7.10 Garbage collection e ownerReferences

Quando você apaga um StatefulSet, o que acontece com os Pods? E com os ReplicaSets de um Deployment? A resposta está nas **ownerReferences**: cada objeto criado por um controller carrega, em `metadata.ownerReferences`, um ponteiro para o objeto "pai".

```bash
kubectl get pod store-db-0 -n store-dev -o jsonpath='{.metadata.ownerReferences}' | python3 -m json.tool
```

```json
[
    {
        "apiVersion": "apps/v1",
        "blockOwnerDeletion": true,
        "controller": true,
        "kind": "StatefulSet",
        "name": "store-db",
        "uid": "5f8b1c0a-7e2d-4a19-9f31-0b2c6a4d8e77"
    }
]
```

O **garbage collector** do controller-manager varre o cluster procurando objetos cujo dono não existe mais e os apaga. Você controla o comportamento com `--cascade`:

- `--cascade=background` (padrão): o dono é apagado imediatamente; os filhos, em segundo plano.
- `--cascade=foreground`: o dono entra em `deletionTimestamp` com o finalizer `foregroundDeletion` e só some quando todos os filhos com `blockOwnerDeletion: true` sumirem.
- `--cascade=orphan`: os filhos ficam **órfãos** e continuam rodando.

```bash
kubectl delete statefulset store-db -n store-dev --cascade=orphan
kubectl get pods -n store-dev
```

```
NAME         READY   STATUS    RESTARTS   AGE
store-db-0   1/1     Running   0          12m
```

O Pod continua de pé, sem controller. É uma técnica útil para migrar workloads entre controllers sem downtime — e uma fonte comum de "Pods fantasma" quando usada por acidente.

Repare que **PVCs criados por `volumeClaimTemplates` não têm o StatefulSet como dono** (a menos que você configure `persistentVolumeClaimRetentionPolicy`), justamente para não serem coletados pelo GC.

---

### 7.11 Armazenamento efêmero, requests/limits e eviction

Escrita em `emptyDir`, na camada writable do container e nos logs do container conta como **ephemeral-storage**, um recurso de primeira classe como CPU e memória:

```yaml
          resources:
            requests:
              ephemeral-storage: 200Mi
            limits:
              ephemeral-storage: 1Gi
```

Se o container ultrapassar o `limits.ephemeral-storage`, o kubelet o **despeja** (evicted) — não há OOMKill nem SIGKILL do kernel, é o kubelet agindo. Independentemente disso, quando o disco do nó cruza o limiar de `nodefs.available < 10%`, o kubelet entra em **disk pressure**: aplica o taint `node.kubernetes.io/disk-pressure`, tenta liberar espaço apagando imagens não usadas e containers mortos e, se não bastar, começa a despejar Pods — primeiro os `BestEffort`, depois os `Burstable` que excedem seus requests.

```bash
kubectl get pods -n store-dev
```

```
NAME                         READY   STATUS    RESTARTS   AGE
store-api-6f9c7b4d55-2wq8t   0/1     Evicted   0          3m
```

```bash
kubectl describe pod store-api-6f9c7b4d55-2wq8t -n store-dev | tail -5
```

```
Status:   Failed
Reason:   Evicted
Message:  Pod ephemeral local storage usage exceeds the total limit of containers 1Gi.
```

> **Na prática:** o campeão de disk pressure em clusters de aula é log de aplicação em modo debug gravado dentro do container. Sempre escreva logs em `stdout`/`stderr` e deixe o agente do DaemonSet cuidar do resto — é o "12-factor" aplicado ao Kubernetes.

---

### 7.12 Laboratório guiado (feito em sala)

Objetivo: subir o `store-db` como StatefulSet com PVC, provar a persistência, montar um ConfigMap como arquivo no `store-api` e coletar logs com um DaemonSet.

**Passo 0 — preparar o ambiente**

```bash
minikube start --nodes=1 --cpus=2 --memory=4096
kubectl create namespace store-dev
kubectl config set-context --current --namespace=store-dev
kubectl get storageclass
```

```
NAME                 PROVISIONER                RECLAIMPOLICY   VOLUMEBINDINGMODE   ALLOWVOLUMEEXPANSION   AGE
standard (default)   k8s.io/minikube-hostpath   Delete          Immediate           false                  2m
```

**Passo 1 — criar o Secret da senha do banco**

```bash
kubectl create secret generic store-db-secret \
  --from-literal=POSTGRES_PASSWORD='FiapStore#2026' \
  -n store-dev
```

```
secret/store-db-secret created
```

**Passo 2 — aplicar o headless Service e o StatefulSet** (arquivo `store-db-statefulset.yaml` da seção 7.7)

```bash
kubectl apply -f store-db-statefulset.yaml
kubectl get pods -w
```

```
NAME         READY   STATUS              RESTARTS   AGE
store-db-0   0/1     ContainerCreating   0          4s
store-db-0   0/1     Running             0          9s
store-db-0   1/1     Running             0          18s
```

**Passo 3 — inspecionar o PVC e o PV criados automaticamente**

```bash
kubectl get pvc,pv
```

```
NAME                                   STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
persistentvolumeclaim/dados-store-db-0 Bound    pvc-1a7d3e9c-4b52-41c8-9a03-2fd6b7e1c904   2Gi        RWO            standard       58s

NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                        STORAGECLASS   AGE
persistentvolume/pvc-1a7d3e9c-4b52-41c8-9a03-2fd6b7e1c904   2Gi        RWO            Delete           Bound    store-dev/dados-store-db-0   standard       58s
```

O nome do PVC segue exatamente `<nome-do-volumeClaimTemplate>-<nome-do-statefulset>-<ordinal>`.

**Passo 4 — gravar dados**

```bash
kubectl exec -it store-db-0 -- psql -U store -d fiapstore -c \
  "CREATE TABLE pedidos (id serial primary key, cliente text, valor numeric);"
kubectl exec -it store-db-0 -- psql -U store -d fiapstore -c \
  "INSERT INTO pedidos (cliente, valor) VALUES ('Maria', 199.90), ('Joao', 89.50);"
kubectl exec -it store-db-0 -- psql -U store -d fiapstore -c "SELECT * FROM pedidos;"
```

```
 id | cliente | valor
----+---------+--------
  1 | Maria   | 199.90
  2 | Joao    |  89.50
(2 rows)
```

**Passo 5 — provar a persistência (o momento da verdade)**

```bash
kubectl delete pod store-db-0
kubectl get pods -w
kubectl exec -it store-db-0 -- psql -U store -d fiapstore -c "SELECT count(*) FROM pedidos;"
```

```
 count
-------
     2
(1 row)
```

O Pod foi destruído e recriado com o **mesmo nome**, remontou o **mesmo PVC** e os dados estão lá. Confirme que o PVC não foi recriado comparando o `AGE` — ele é mais velho que o Pod:

```bash
kubectl get pvc dados-store-db-0
kubectl get pod store-db-0
```

**Passo 6 — montar o ConfigMap como arquivo no `store-api`**

```bash
kubectl apply -f store-api-config.yaml
kubectl apply -f store-api-deployment.yaml
kubectl exec -it deploy/store-api -- ls -l /etc/store
kubectl exec -it deploy/store-api -- cat /etc/store/app.yaml
```

```
total 0
lrwxrwxrwx 1 root root 17 Aug  6 14:22 app.yaml -> ..data/app.yaml
lrwxrwxrwx 1 root root 20 Aug  6 14:22 banner.txt -> ..data/banner.txt
```

Repare que são **symlinks** para um diretório `..data`. É assim que o kubelet consegue trocar todos os arquivos atomicamente quando o ConfigMap muda.

**Passo 7 — observar a atualização automática**

```bash
kubectl patch configmap store-api-config --type merge \
  -p '{"data":{"banner.txt":"Bem-vindo a FIAP Store (config atualizada em aula)\n"}}'
sleep 70
kubectl exec -it deploy/store-api -- cat /etc/store/banner.txt
```

```
Bem-vindo a FIAP Store (config atualizada em aula)
```

Agora repita o experimento em um Pod que use `subPath` e comprove que o arquivo **não** muda. Essa é a armadilha número um de configuração em Kubernetes.

**Passo 8 — subir o DaemonSet de logs**

```bash
kubectl apply -f fluent-daemonset.yaml
kubectl get ds store-logs
kubectl logs -l app=store-logs --tail=3
POD=$(kubectl get pod -l app=store-logs -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD -- tail -3 /coleta/fiapstore/coleta.log
```

```
2026-08-06T14:31:02+00:00 node=minikube pods=14
2026-08-06T14:31:17+00:00 node=minikube pods=14
2026-08-06T14:31:32+00:00 node=minikube pods=15
```

**Passo 9 — provar que o dado está no nó, não no Pod**

```bash
minikube ssh -- tail -3 /var/log/fiapstore/coleta.log
kubectl delete pod $POD
sleep 20
minikube ssh -- wc -l /var/log/fiapstore/coleta.log
```

O arquivo continua crescendo do ponto em que parou: o `hostPath` sobreviveu ao Pod. E isso é exatamente o risco — se esse nó for terminado por um autoscaler, o arquivo some junto.

**Passo 10 — limpeza**

```bash
kubectl delete -f fluent-daemonset.yaml -f store-api-deployment.yaml -f store-api-config.yaml
kubectl delete -f store-db-statefulset.yaml
kubectl get pvc          # o PVC continua! StatefulSet nao apaga PVC
kubectl delete pvc dados-store-db-0
kubectl get pv           # o PV foi removido junto (reclaimPolicy Delete)
```

---

### 7.13 Ponte com Serverless

O modelo de armazenamento que acabamos de estudar existe porque, no Kubernetes, **o compute é durável o bastante para carregar estado**. Em FaaS, essa premissa desaparece por completo.

Uma função AWS Lambda executa dentro de um *micro-VM* Firecracker que a plataforma cria, congela e destrói sem aviso. Ela oferece `/tmp` com 512 MB por padrão (configurável até 10 GB), que é o equivalente conceitual mais próximo de um `emptyDir`: rápido, local, e garantido apenas durante a vida daquele ambiente de execução. Como a AWS reaproveita o ambiente entre invocações próximas (o "warm start"), você *pode* encontrar o arquivo que gravou na invocação anterior — mas **não pode contar com isso**. Depender desse comportamento produz o pior tipo de bug: aquele que só aparece em produção sob carga variável. É o mesmo raciocínio de `emptyDir` levado ao extremo: use como cache, nunca como fonte de verdade.

Para estado real, o padrão serverless é **"estado fora do compute"**: S3 para objetos e artefatos, RDS/Aurora Serverless para relacional, DynamoDB para chave-valor de baixa latência, ElastiCache para sessão. Repare que é exatamente a recomendação que damos para Kubernetes em produção — o `store-db` em StatefulSet é ótimo para aprender e para clusters on-premise, mas em EKS a escolha madura quase sempre é apontar o `store-api` para um RDS gerenciado e deixar o cluster totalmente stateless. StatefulSet resolve o problema; RDS evita o problema.

Quando um sistema de arquivos POSIX compartilhado é inevitável, o **Lambda pode montar um EFS Access Point** diretamente, o que exige colocar a função em uma VPC. O paralelo com Kubernetes é perfeito: é o mesmo EFS que serviria um PVC `ReadWriteMany` via `efs.csi.aws.com`. Muda o consumidor, não o armazenamento. Vale lembrar dos custos do trade-off: montar EFS obriga a função a ter ENIs na VPC, o que historicamente aumentava o cold start (hoje bem menor com Hyperplane ENIs) e adiciona custo por GB-mês e por throughput.

E os DaemonSets? Em serverless eles simplesmente **não existem** — não há nó para colocar um agente. A coleta de logs é feita pela plataforma: `stdout` da função vai automaticamente para o CloudWatch Logs, e as métricas para o CloudWatch Metrics. Você troca controle (formato, destino, filtragem local, custo de retenção) por zero operação. No EKS há um meio-termo popular: o **Fargate**, onde não existem nós seus e, portanto, DaemonSets **não são suportados** — a saída é usar o *sidecar* de log routing do Fargate (o `aws-observability` ConfigMap com fluent-bit). Reconhecer essa restrição antes de projetar a arquitetura evita uma reescrita dolorosa.

---

### 7.14 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| PVC eternamente `Pending` | Não há StorageClass padrão, ou o nome informado não existe | `kubectl describe pvc <nome>` → evento `no persistent volumes available` ou `storageclass "gp3" not found`; `kubectl get sc` | Criar/corrigir a StorageClass ou marcar uma como padrão com a annotation `storageclass.kubernetes.io/is-default-class: "true"` |
| PVC `Pending` com `waiting for first consumer` | `volumeBindingMode: WaitForFirstConsumer` e nenhum Pod ainda usa o PVC | `kubectl get sc -o wide` | Comportamento esperado — crie o Pod que consome o PVC |
| Pod `Pending` com `volume node affinity conflict` | Volume EBS em AZ diferente do nó escolhido | `kubectl describe pod` → seção Events | Usar `WaitForFirstConsumer` na StorageClass |
| Pod `Pending`, PVC `Bound`, evento `Multi-Attach error` | Volume RWO já montado em outro nó (típico em rolling update de Deployment com PVC) | `kubectl describe pod`; `kubectl get pods -o wide` | Usar `strategy: Recreate` no Deployment, ou StatefulSet, ou storage RWX (EFS) |
| PVC pede RWX e fica `Pending` | Driver não suporta RWX (EBS) | `kubectl describe pvc`; conferir o `provisioner` da SC | Trocar para EFS/NFS ou redesenhar para RWO |
| EBS CSI ausente no EKS: todo PVC `Pending` | Add-on não instalado | `kubectl get pods -n kube-system -l app=ebs-csi-controller` (vazio) | `eksctl create addon --name aws-ebs-csi-driver ...` com IRSA |
| PostgreSQL não inicia: `directory not empty` | `PGDATA` apontando para a raiz do volume com `lost+found` | `kubectl logs store-db-0` | Definir `PGDATA=/var/lib/postgresql/data/pgdata` |
| ConfigMap alterado, aplicação não vê | Montagem com `subPath`, ou a app não relê o arquivo | `kubectl exec -- cat` no arquivo; conferir `subPath` no manifesto | Remover `subPath` e/ou `kubectl rollout restart deploy/store-api` |
| Pod `Evicted` com `ephemeral local storage usage exceeds` | Escrita excessiva em `emptyDir`/camada writable | `kubectl describe pod` → `Reason: Evicted` | Aumentar `limits.ephemeral-storage`, usar PVC, ou mandar logs para stdout |
| DaemonSet com `DESIRED 0` | `nodeSelector` sem nós correspondentes, ou taints sem toleration | `kubectl describe ds store-logs`; `kubectl get nodes --show-labels`; `kubectl describe node \| grep Taints` | Ajustar `nodeSelector` ou adicionar `tolerations` |
| Volume expandido no PVC, mas `df` mostra tamanho antigo | Resize do filesystem pendente | `kubectl describe pvc` → `FileSystemResizePending` | Reiniciar o Pod (`kubectl delete pod`) |
| Apaguei o StatefulSet e o EBS continua sendo cobrado | PVC não é apagado com o StatefulSet; ou `reclaimPolicy: Retain` | `kubectl get pvc,pv`; console EC2 → Volumes | `kubectl delete pvc <nome>` e remover o volume manualmente se `Retain` |

---

### 7.15 Resumo do capítulo

O sistema de arquivos de um container é efêmero por natureza, e no Kubernetes essa característica é agravada pelo fato de os Pods serem descartáveis e móveis entre nós. A resposta do Kubernetes é uma família de tipos de Volume — de `emptyDir` (rascunho do Pod) a `csi` (armazenamento de rede real) — e, acima deles, uma indireção em três objetos: PersistentVolume representa o armazenamento, PersistentVolumeClaim representa o pedido e StorageClass representa a política de provisionamento. Sobre essa base, o StatefulSet dá identidade estável e disco próprio a cada réplica de uma aplicação com estado, enquanto o DaemonSet resolve a necessidade oposta: rodar um agente de infraestrutura em cada nó, quase sempre lendo o disco do nó via `hostPath`.

Pontos-chave:

1. **Volume ≠ persistência.** `emptyDir` sobrevive a reinícios de container, mas morre com o Pod. Só PV/PVC sobrevivem ao Pod.
2. **RWO significa "um nó", não "um Pod".** Para exclusividade real, `ReadWriteOncePod`; para múltiplos nós escrevendo, EFS/NFS (RWX).
3. **`WaitForFirstConsumer` é obrigatório em cluster multi-AZ**, sob pena de Pods presos em `Pending` por conflito de zona.
4. **`reclaimPolicy: Delete` apaga o disco de verdade.** Use `Retain` em produção e lembre-se de que `Retain` significa que você continua pagando pelo volume.
5. **StatefulSet não apaga PVCs** ao ser removido — proteção contra perda de dados, e fonte de custo esquecido na AWS.
6. **`subPath` congela o ConfigMap.** Sem `subPath` o kubelet atualiza os arquivos automaticamente; com ele, só reiniciando o Pod.
7. **DaemonSet é para infraestrutura, não para negócio** — e no Fargate ele nem existe, forçando o padrão de sidecar de logs.

---

### 7.16 Exercícios (para casa)

#### Exercício 7.1 — Persistência básica com PVC e Deployment
*Tempo estimado: 25 min · Nível: fixação guiada*

- **Objetivo:** comprovar, na prática, que um PVC preserva dados entre recriações de Pod.
- **Enunciado:** no namespace `store-dev`, crie um PVC chamado `relatorios-pvc` de 1Gi, `ReadWriteOnce`, usando a StorageClass padrão do seu cluster. Crie um Deployment `store-relatorios` com 1 réplica, imagem `busybox:1.36`, que monte o PVC em `/relatorios` e execute um laço que acrescente uma linha com data/hora e o nome do Pod (via `downwardAPI` ou `fieldRef` em env) ao arquivo `/relatorios/vendas.log` a cada 10 segundos. Deixe rodar por 1 minuto, conte as linhas, apague o Pod com `kubectl delete pod`, espere o novo Pod subir e conte as linhas de novo. Em seguida, repita o experimento inteiro trocando o PVC por um `emptyDir` e registre a diferença. Como o Deployment usa volume RWO, configure `strategy: Recreate` e explique em um comentário no YAML por que isso é necessário.
- **Entregável:** arquivos `pvc-relatorios.yaml` e `deploy-relatorios.yaml`; saída de `kubectl get pvc,pods -n store-dev -o wide` e dos dois `wc -l /relatorios/vendas.log` (antes e depois da exclusão do Pod), nos dois cenários (PVC e `emptyDir`).
- **Critério de aceite:** com PVC, a contagem de linhas **após** a recriação do Pod é estritamente maior que a contagem anterior e o arquivo contém nomes de **dois Pods diferentes**. Com `emptyDir`, a contagem reinicia do zero.
- **Dica:**
  <details><summary>Dica</summary>
  Use <code>command: ["/bin/sh","-c","while true; do echo \"$(date -Iseconds) $POD_NAME\" &gt;&gt; /relatorios/vendas.log; sleep 10; done"]</code> e injete <code>POD_NAME</code> com <code>fieldRef: metadata.name</code>. Sem <code>strategy: Recreate</code>, o Pod novo ficará em <code>Pending</code> esperando o volume RWO ser liberado pelo antigo.
  </details>

#### Exercício 7.2 — StatefulSet de 3 réplicas com volumeClaimTemplates
*Tempo estimado: 40 min · Nível: aplicação*

- **Objetivo:** construir um StatefulSet com armazenamento por réplica e verificar identidade estável de rede e de disco.
- **Enunciado:** crie um headless Service `store-cache` e um StatefulSet `store-cache` com 3 réplicas (use `redis:7-alpine` ou `busybox:1.36`, à sua escolha), com um `volumeClaimTemplates` chamado `dados` de 500Mi RWO montado em `/data`. Em cada Pod, grave dentro de `/data/identidade.txt` o próprio hostname. Depois: (a) liste os 3 PVCs e comprove que os nomes seguem o padrão `dados-store-cache-N`; (b) escale para 5 réplicas e mostre os 2 PVCs novos; (c) escale de volta para 3 e comprove que os PVCs 3 e 4 **continuam existindo**; (d) apague o Pod `store-cache-1` e comprove que o novo Pod lê o mesmo `identidade.txt`; (e) a partir de um Pod auxiliar, resolva por DNS `store-cache-0.store-cache.store-dev.svc.cluster.local` e mostre o IP. Explique em um arquivo `ANALISE.md` por que os PVCs não são apagados no scale-down e como mudar esse comportamento.
- **Entregável:** `store-cache-statefulset.yaml`, `ANALISE.md`, saídas de `kubectl get statefulset,pods,pvc -n store-dev` nas etapas (a), (b) e (c), e a saída do `nslookup`/`getent hosts` da etapa (e).
- **Critério de aceite:** existem exatamente 5 PVCs após a etapa (c) com apenas 3 Pods rodando; o `identidade.txt` do `store-cache-1` recriado contém `store-cache-1`; a resolução DNS do Pod 0 retorna o IP do Pod (não um ClusterIP).
- **Dica:**
  <details><summary>Dica</summary>
  Para o DNS funcionar, o <code>spec.serviceName</code> do StatefulSet precisa bater exatamente com o nome do headless Service. Para o item do <code>ANALISE.md</code>, pesquise <code>persistentVolumeClaimRetentionPolicy.whenScaled</code>.
  </details>

#### Exercício 7.3 — Depuração: três falhas de armazenamento
*Tempo estimado: 35 min · Nível: análise/depuração*

- **Objetivo:** diagnosticar e corrigir falhas reais de provisionamento e vinculação de volumes.
- **Enunciado:** aplique os três cenários abaixo, um de cada vez, e para **cada um** registre: (i) o STATUS observado, (ii) o evento exato que revela a causa (obtido com `kubectl describe` ou `kubectl get events --sort-by=.lastTimestamp`), (iii) a correção mínima aplicada, (iv) a evidência de que voltou ao normal.
  - **Cenário A:** um PVC com `storageClassName: gp3-turbo` (classe inexistente no seu cluster) e um Pod que o consome.
  - **Cenário B:** um PVC estático com `accessModes: [ReadWriteMany]` tentando se ligar a um PV criado manualmente com `accessModes: [ReadWriteOnce]` e capacidade menor que a solicitada.
  - **Cenário C:** um Deployment com 2 réplicas montando o **mesmo** PVC RWO, em cluster com mais de um nó (ou simule reduzindo `nodeSelector`), até obter `Multi-Attach error` ou Pod `Pending` por volume ocupado.
  Escreva as respostas em `DEPURACAO.md`.
- **Entregável:** `DEPURACAO.md` com as 4 informações por cenário, os YAMLs quebrados (`cenario-a.yaml`, `cenario-b.yaml`, `cenario-c.yaml`) e os corrigidos, e os blocos de saída de `kubectl describe pvc/pod` que provam o diagnóstico.
- **Critério de aceite:** cada cenário apresenta a mensagem de evento correta e específica (`storageclass.storage.k8s.io "gp3-turbo" not found`, incompatibilidade de accessMode/capacidade e `Multi-Attach error for volume`), e ao final os três cenários têm todos os Pods em `Running` e os PVCs em `Bound`.
- **Dica:**
  <details><summary>Dica</summary>
  <code>kubectl get events -n store-dev --sort-by=.lastTimestamp | tail -20</code> costuma mostrar a causa antes do <code>describe</code>. No Cenário C, a correção "mínima" pode ser tanto reduzir para 1 réplica quanto migrar para volume RWX — justifique sua escolha.
  </details>

#### Exercício 7.4 — Desafio: DaemonSet de métricas x observabilidade serverless
*Tempo estimado: 55 min · Nível: desafio integrador*

- **Objetivo:** operar um agente por nó e contrastar esse modelo com a observabilidade gerenciada de plataformas serverless.
- **Enunciado:** crie um DaemonSet `store-metrics` no namespace `store-dev` que rode em **todos** os nós (inclusive nós com taint de control plane, se seu cluster os expuser). O agente deve, a cada 30 segundos, coletar do nó pelo menos: uso de disco de `/` , uptime e quantidade de diretórios em `/var/log/pods`; e gravar as amostras em `/var/log/fiapstore/metrics.log` no nó, via `hostPath`. Requisitos adicionais: (a) montar `/var/log/pods` como `readOnly: true`; (b) declarar `requests` e `limits` de CPU, memória e `ephemeral-storage`; (c) usar `updateStrategy.rollingUpdate.maxUnavailable: 1`; (d) usar `downwardAPI` ou `fieldRef` para que cada linha do log identifique o nó (`spec.nodeName`). Se estiver usando minikube, suba o cluster com `minikube start --nodes=2` e comprove `DESIRED=2`. Em seguida, altere a imagem do container e observe o rolling update Pod a Pod. Por fim, escreva `COMPARACAO.md` (mínimo 400 palavras) comparando essa abordagem com uma equivalente serverless: em uma arquitetura de Lambdas, quem faz o papel do agente? Aborde CloudWatch Logs/Metrics, a extensão Lambda (Telemetry API) como o "sidecar" possível, a impossibilidade de DaemonSet em EKS Fargate, e faça uma estimativa comparativa de custo entre reter 30 dias de log via agente próprio em EBS/S3 versus CloudWatch Logs. Termine com uma recomendação justificada para a FIAP Store.
- **Entregável:** `store-metrics-daemonset.yaml`, `COMPARACAO.md`, saídas de `kubectl get ds,pods -o wide -n store-dev`, `kubectl rollout status ds/store-metrics -n store-dev` e um trecho do `metrics.log` de **cada** nó.
- **Critério de aceite:** `DESIRED`, `CURRENT`, `READY` e `AVAILABLE` são iguais ao número de nós do cluster; os arquivos `metrics.log` dos nós contêm valores diferentes de `nodeName`; o rolling update é observável (nunca mais de 1 Pod indisponível por vez); e o `COMPARACAO.md` cita explicitamente as três diferenças pedidas com uma recomendação final.
- **Dica:**
  <details><summary>Dica</summary>
  Para coletar sem instalar nada, <code>df -h /</code>, <code>cat /proc/uptime</code> e <code>ls /var/log/pods | wc -l</code> dentro de um <code>busybox</code> bastam. Para forçar o rolling update sem trocar a imagem de verdade, use <code>kubectl set env ds/store-metrics VERSAO=2 -n store-dev</code>.
  </details>

---

### 7.17 Leitura recomendada

1. [DOC] **Kubernetes — Storage: Volumes, Persistent Volumes, Storage Classes, Ephemeral Volumes.** https://kubernetes.io/docs/concepts/storage/ — leitura obrigatória das seções "Volumes", "Persistent Volumes" e "Storage Classes".
2. [DOC] **Kubernetes — Workloads: StatefulSets e DaemonSet.** https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/ e https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/ — atenção às garantias de ordenação e ao `updateStrategy`.
3. [DOC] **AWS — EKS Best Practices Guide: Storage.** https://docs.aws.amazon.com/eks/latest/best-practices/storage.html — decisão entre EBS e EFS, IRSA para o driver CSI e controle de custo.
4. [LIVRO] **LUKSA, M.** *Kubernetes in Action*, 2. ed. Manning, 2023 — capítulos sobre volumes, ConfigMaps/Secrets e StatefulSets; a melhor explicação didática do modelo PV/PVC disponível.
5. [LIVRO] **BURNS, B.; BEDA, J.; HIGHTOWER, K.; EVENSON, L.** *Kubernetes: Up and Running*, 3. ed. O'Reilly, 2022 — capítulos "Storage Solutions" e "DaemonSets".
6. [ARTIGO] **CNCF — Container Storage Interface (CSI) Specification.** https://github.com/container-storage-interface/spec — para entender por que o Controller é Deployment e o Node é DaemonSet.

---

> ### Marco avaliativo — CHECK POINT 02
>
> Neste ponto do semestre é lançado o **Check Point 02 — Operação em cluster multi-nó: rede,
> namespaces e armazenamento**, que cobre os capítulos 5 a 7. O enunciado completo, os entregáveis,
> a rubrica de 100 pontos e o roteiro de correção estão na **Parte II — Avaliações**.

---

## Capítulo 8 — Jobs, Secrets e Endpoints

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 7 (Pods/Services, Deployments/HPA, networking/namespaces, EKS, volumes/StatefulSet/DaemonSet); noção de `restartPolicy`; sintaxe cron. · **Artefatos:** `fechamento-cronjob.yaml`, `relatorio-job.yaml`, `store-db-secret.yaml`, `store-api-secret-deploy.yaml`, `pagamentos-externo-svc.yaml`, `keda-scaledjob.yaml`, namespace `store-dev`.

---

### 8.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as diferenças entre cargas de trabalho de serviço (Deployment) e cargas de trabalho batch (Job/CronJob), escolhendo o controller adequado ao problema.
2. **Aplicar** os parâmetros de um Job (`completions`, `parallelism`, `backoffLimit`, `activeDeadlineSeconds`, `ttlSecondsAfterFinished`, `completionMode`, `podFailurePolicy`) para modelar processamento paralelo e indexado.
3. **Construir** CronJobs corretos, dominando `concurrencyPolicy`, `startingDeadlineSeconds`, limites de histórico, `suspend` e `timeZone`.
4. **Analisar** criticamente a segurança de Secrets — reconhecendo que base64 não é criptografia — e **aplicar** as formas de consumo (env x volume x `imagePullSecrets`), além de identificar quando migrar para External Secrets/Secrets Manager.
5. **Explicar** o mecanismo de Endpoints/EndpointSlice e **construir** Services sem selector, Endpoints manuais e ExternalName para integrar recursos externos ao cluster.
6. **Comparar** o modelo Job/CronJob com o modelo serverless (EventBridge Scheduler + Lambda, KEDA ScaledJob com escala a zero), avaliando custo e limites de tempo de execução.

---

### 8.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 8 min | Abertura: por que um Deployment é a ferramenta errada para "gerar o relatório de fechamento" |
| 2 | 22 min | Job em profundidade: completions, parallelism, backoffLimit, deadlines, TTL, Indexed, podFailurePolicy |
| 3 | 15 min | CronJob: schedule, concurrencyPolicy, startingDeadline, histórico, suspend, timeZone; padrões de work queue |
| — | 10 min | **Intervalo** |
| 4 | 20 min | Secrets: tipos, criação, base64 x criptografia, env x volume, imagePullSecrets, encryption at rest, ESO/Vault/SOPS |
| 5 | 15 min | Endpoints e EndpointSlice: como o Service acha os Pods; Service sem selector, ExternalName, headless |
| 6 | 22 min | **Laboratório guiado:** CronJob de fechamento + Secret nas duas formas + Service sem selector |
| 7 | 8 min | Ponte com Serverless: EventBridge + Lambda, KEDA ScaledJob e escala a zero |

---

### 8.3 Cargas de trabalho batch: o Job

Todos os controllers vistos até aqui — Deployment, StatefulSet, DaemonSet — assumem que o processo **nunca termina**. Se um container de Deployment sai com código 0, o kubelet o reinicia, porque `restartPolicy: Always` significa literalmente "sempre". Isso torna impossível expressar "rode este processamento uma vez, até o fim, e pare".

O **Job** existe para isso: ele cria um ou mais Pods e acompanha até que um número configurado deles **termine com sucesso** (exit code 0). Um Job concluído permanece no cluster, com o resultado registrado, até ser apagado.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: relatorio-vendas
  namespace: store-dev
spec:
  backoffLimit: 4
  activeDeadlineSeconds: 600
  ttlSecondsAfterFinished: 3600
  template:
    metadata:
      labels:
        app: store-worker
        tier: batch
    spec:
      restartPolicy: OnFailure
      containers:
        - name: worker
          image: docker.io/seuusuario/store-worker:1.0
          command: ["python", "-u", "relatorio.py", "--dia", "ontem"]
          env:
            - name: DB_HOST
              value: store-db-0.store-db.store-dev.svc.cluster.local
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: store-db-secret
                  key: POSTGRES_PASSWORD
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

#### 8.3.1 completions e parallelism

Esses dois campos definem três padrões distintos:

| Padrão | `completions` | `parallelism` | Comportamento |
|---|---|---|---|
| Tarefa única | 1 (ou omitido) | 1 (ou omitido) | Um Pod, uma execução bem-sucedida |
| Paralelismo fixo | N | M | Roda até N sucessos, com no máximo M Pods simultâneos |
| Fila de trabalho | omitido | M | M Pods em paralelo; quando **qualquer um** sai com sucesso, o Job encerra assim que os demais terminarem |

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: processa-pedidos
  namespace: store-dev
spec:
  completions: 12
  parallelism: 4
  backoffLimit: 6
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: worker
          image: busybox:1.36
          command: ["/bin/sh","-c","echo processando lote em $(hostname); sleep 5; echo lote concluido"]
```

```bash
kubectl get job processa-pedidos -n store-dev
```

```
NAME               STATUS     COMPLETIONS   DURATION   AGE
processa-pedidos   Complete   12/12         21s        21s
```

#### 8.3.2 restartPolicy: OnFailure x Never

Em Job, `Always` é proibido (o API server rejeita). A escolha entre os outros dois muda **onde** a nova tentativa acontece:

- **`OnFailure`** — o kubelet reinicia o *container dentro do mesmo Pod*. Mais rápido e mais barato, mas você perde os logs da tentativa anterior e o mesmo nó é reutilizado.
- **`Never`** — o Job Controller cria um *Pod novo* a cada falha. Os Pods falhados ficam no cluster, o que é excelente para depuração (`kubectl logs` de cada tentativa) e permite reagendamento em outro nó.

Para trabalho de aula e para produção que precisa de auditoria, prefira `Never`. Para tarefas triviais e de alta frequência, `OnFailure`.

#### 8.3.3 backoffLimit e activeDeadlineSeconds

`backoffLimit` (padrão 6) é o número de **falhas** toleradas antes de o Job ser marcado como `Failed`. Entre as tentativas há um backoff exponencial: 10s, 20s, 40s... até o teto de 6 minutos. Estourado o limite, o Job para de criar Pods e recebe a condição `Failed` com `reason: BackoffLimitExceeded`.

```bash
kubectl get job relatorio-vendas -n store-dev -o jsonpath='{.status.conditions}' | python3 -m json.tool
```

```json
[
    {
        "lastTransitionTime": "2026-08-06T13:22:41Z",
        "message": "Job has reached the specified backoff limit",
        "reason": "BackoffLimitExceeded",
        "status": "True",
        "type": "Failed"
    }
]
```

`activeDeadlineSeconds` é um limite de **tempo de parede** do Job inteiro, contado do início. Ele tem precedência sobre o `backoffLimit`: estourado o prazo, todos os Pods são terminados e o Job vira `Failed` com `reason: DeadlineExceeded`. É a rede de segurança contra o worker que trava em uma consulta ao banco.

> **Nota:** existe também `activeDeadlineSeconds` no `spec` do Pod. São coisas diferentes: o do Job limita o Job; o do Pod limita cada Pod individualmente.

#### 8.3.4 ttlSecondsAfterFinished

Sem TTL, Jobs concluídos se acumulam eternamente — e um CronJob de 5 em 5 minutos gera 288 Jobs por dia. `ttlSecondsAfterFinished` instrui o TTL controller a apagar o Job (e seus Pods, via ownerReferences) N segundos após ele terminar, com sucesso ou falha. Valor de `0` apaga imediatamente. Um bom padrão é `3600` (uma hora) para dar tempo de inspecionar logs.

#### 8.3.5 completionMode: Indexed

Por padrão (`NonIndexed`), todos os Pods do Job são idênticos e não sabem "quem são". Com `completionMode: Indexed`, cada Pod recebe um índice de `0` a `completions-1`, exposto em:

- na variável de ambiente `JOB_COMPLETION_INDEX`;
- na annotation `batch.kubernetes.io/job-completion-index`;
- no **hostname** do Pod, no formato `<job>-<index>`.

Isso permite particionar dados estaticamente sem fila externa: o Pod 0 processa a partição 0, o Pod 1 a partição 1, e assim por diante. É o modelo de MPI/HPC trazido para o Kubernetes.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: reindexa-catalogo
  namespace: store-dev
spec:
  completions: 5
  parallelism: 5
  completionMode: Indexed
  ttlSecondsAfterFinished: 600
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: worker
          image: busybox:1.36
          command:
            - /bin/sh
            - -c
            - |
              echo "Pod $(hostname) processando particao ${JOB_COMPLETION_INDEX} do catalogo"
              sleep $(( (JOB_COMPLETION_INDEX + 1) * 3 ))
              echo "particao ${JOB_COMPLETION_INDEX} concluida"
          env:
            - name: JOB_COMPLETION_INDEX
              valueFrom:
                fieldRef:
                  fieldPath: metadata.annotations['batch.kubernetes.io/job-completion-index']
```

```bash
kubectl logs -l job-name=reindexa-catalogo -n store-dev --prefix --tail=1
```

```
[pod/reindexa-catalogo-0-hk2vn/worker] particao 0 concluida
[pod/reindexa-catalogo-1-9plq4/worker] particao 1 concluida
[pod/reindexa-catalogo-2-t6xzs/worker] particao 2 concluida
[pod/reindexa-catalogo-3-wm4rd/worker] particao 3 concluida
[pod/reindexa-catalogo-4-c8jf7/worker] particao 4 concluida
```

#### 8.3.6 podFailurePolicy

`backoffLimit` trata todas as falhas igualmente, o que é caro: se o worker sai com código 42 significando "arquivo de entrada corrompido, nunca vai funcionar", repetir 6 vezes é desperdício. Se o Pod foi despejado por preemption do nó, o oposto — não deveria nem contar como falha. `podFailurePolicy` (estável desde a 1.31) resolve isso:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: fechamento-inteligente
  namespace: store-dev
spec:
  completions: 3
  parallelism: 3
  backoffLimit: 6
  podFailurePolicy:
    rules:
      - action: FailJob
        onExitCodes:
          containerName: worker
          operator: In
          values: [42]
      - action: Ignore
        onPodConditions:
          - type: DisruptionTarget
      - action: Count
        onExitCodes:
          containerName: worker
          operator: NotIn
          values: [0]
  template:
    spec:
      restartPolicy: Never
      containers:
        - name: worker
          image: docker.io/seuusuario/store-worker:1.0
          command: ["python", "-u", "fechamento.py"]
```

As três ações são `FailJob` (aborta o Job na hora), `Ignore` (não incrementa o contador de falhas) e `Count` (comportamento padrão). Note que `podFailurePolicy` só funciona com `restartPolicy: Never`.

---

### 8.4 CronJob

O CronJob é um controller que cria **Jobs** segundo uma agenda. Ele não roda Pods diretamente — a cadeia é `CronJob → Job → Pod`, e entender isso resolve metade dos problemas de depuração.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: fechamento-diario
  namespace: store-dev
spec:
  schedule: "0 2 * * *"
  timeZone: "America/Sao_Paulo"
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 300
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  suspend: false
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 1800
      ttlSecondsAfterFinished: 7200
      template:
        metadata:
          labels:
            app: store-worker
            tier: batch
            env: dev
        spec:
          restartPolicy: Never
          containers:
            - name: fechamento
              image: docker.io/seuusuario/store-worker:1.0
              command: ["python", "-u", "/app/fechamento.py"]
              env:
                - name: APP_ENV
                  value: dev
                - name: DB_HOST
                  value: store-db-0.store-db.store-dev.svc.cluster.local
                - name: DB_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: store-db-secret
                      key: POSTGRES_PASSWORD
              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  cpu: "1"
                  memory: 512Mi
```

**`schedule`** usa a sintaxe cron padrão de 5 campos: `minuto hora dia-do-mês mês dia-da-semana`. Aceita também os atalhos `@hourly`, `@daily`, `@weekly`, `@monthly`, `@yearly`.

```
 ┌───────── minuto (0-59)
 │ ┌─────── hora (0-23)
 │ │ ┌───── dia do mes (1-31)
 │ │ │ ┌─── mes (1-12)
 │ │ │ │ ┌─ dia da semana (0-6, domingo=0)
 │ │ │ │ │
 0 2 * * *     -> todo dia as 02:00
 */5 * * * *   -> a cada 5 minutos
 0 8 * * 1-5   -> 08:00 de segunda a sexta
 0 0 1 * *     -> 00:00 do primeiro dia de cada mes
```

**`timeZone`** (estável desde a 1.27) resolve um problema histórico sério: até então o schedule era interpretado no fuso do **kube-controller-manager**, quase sempre UTC. Um "fechamento às 2h" virava 23h do dia anterior no horário de Brasília. Sempre declare `timeZone` explicitamente — é a diferença entre um relatório correto e um relatório de um dia errado.

**`concurrencyPolicy`:**

| Valor | Comportamento | Quando usar |
|---|---|---|
| `Allow` (padrão) | Permite execuções sobrepostas | Tarefas idempotentes e curtas |
| `Forbid` | Pula a nova execução se a anterior ainda roda | Fechamento contábil, migração — qualquer coisa que não pode duplicar |
| `Replace` | Mata a execução anterior e inicia a nova | Sincronização de cache em que só o resultado mais recente importa |

**`startingDeadlineSeconds`** define por quanto tempo, depois do horário previsto, o controller ainda pode iniciar um Job atrasado (por exemplo, se o control plane estava fora do ar). Sem esse campo, e se o CronJob perder mais de **100 horários consecutivos**, o controller desiste e registra o evento `FailedNeedsStart` — o CronJob simplesmente para de funcionar até ser recriado. Defina sempre um valor razoável (60 a 600 segundos).

**`suspend: true`** pausa o CronJob sem apagá-lo. Jobs já em execução continuam.

```bash
kubectl patch cronjob fechamento-diario -n store-dev -p '{"spec":{"suspend":true}}'
kubectl get cronjob -n store-dev
```

```
NAME                SCHEDULE    TIMEZONE             SUSPEND   ACTIVE   LAST SCHEDULE   AGE
fechamento-diario   0 2 * * *   America/Sao_Paulo    True      0        6h              2d
```

Para testar sem esperar a agenda:

```bash
kubectl create job --from=cronjob/fechamento-diario fechamento-manual-01 -n store-dev
```

#### 8.4.1 Padrões de fila de trabalho

Job com `parallelism` é ótimo para trabalho *estático*, conhecido de antemão. Para trabalho *dinâmico* — pedidos chegando o dia inteiro — o padrão é a **work queue**: uma fila externa (Redis, RabbitMQ, Amazon SQS) e workers que consomem até a fila esvaziar.

```
   store-api  ──publica──►  ┌──────────┐
                            │   SQS    │
                            │  fila de │
                            │ pedidos  │
                            └────┬─────┘
                                 │ consome
              ┌──────────────────┼──────────────────┐
        ┌─────▼─────┐      ┌─────▼─────┐      ┌─────▼─────┐
        │ worker-1  │      │ worker-2  │      │ worker-3  │
        └───────────┘      └───────────┘      └───────────┘
                (Job com parallelism=3, completions omitido)
```

Com `completions` omitido e `parallelism: 3`, o Job considera o trabalho concluído quando **um** Pod sai com sucesso — a convenção é que o worker sai com 0 apenas quando encontra a fila vazia. Simples, mas com um limite claro: o número de workers é fixo. Se a fila explode, ninguém escala. É exatamente esse buraco que o **KEDA** preenche, e voltaremos a ele na Ponte com Serverless.

---

### 8.5 Secrets

Um **Secret** é um objeto de namespace que armazena dados sensíveis: senhas, tokens, chaves TLS, credenciais de registry. Estruturalmente é quase idêntico a um ConfigMap, com três diferenças de tratamento pelo cluster: os valores são armazenados em base64 no campo `data`, o kubelet monta Secrets em **tmpfs** (memória), e o RBAC costuma restringir o verbo `get` sobre Secrets muito mais do que sobre ConfigMaps.

#### 8.5.1 Tipos

| Tipo | Uso |
|---|---|
| `Opaque` | Padrão; pares chave/valor arbitrários (senha do banco, API key) |
| `kubernetes.io/dockerconfigjson` | Credenciais de registry privado, consumidas via `imagePullSecrets` |
| `kubernetes.io/tls` | Par certificado/chave (`tls.crt`, `tls.key`), usado por Ingress |
| `kubernetes.io/service-account-token` | Token de ServiceAccount (legado — ver capítulo 9) |
| `kubernetes.io/basic-auth` | Chaves `username` e `password` |
| `kubernetes.io/ssh-auth` | Chave privada SSH (`ssh-privatekey`) |

#### 8.5.2 Criação imperativa e declarativa

```bash
# Opaque a partir de literais
kubectl create secret generic store-db-secret \
  --from-literal=POSTGRES_USER=store \
  --from-literal=POSTGRES_PASSWORD='FiapStore#2026' \
  -n store-dev

# Opaque a partir de arquivo
kubectl create secret generic store-api-env --from-file=./.env.prod -n store-dev

# Registry privado
kubectl create secret docker-registry regcred \
  --docker-server=docker.io \
  --docker-username=seuusuario \
  --docker-password='TOKEN_DO_REGISTRY' \
  --docker-email=aluno@fiap.com.br \
  -n store-dev

# TLS
kubectl create secret tls store-tls \
  --cert=./fiapstore.crt --key=./fiapstore.key -n store-dev
```

Na forma declarativa, `stringData` aceita texto puro (o API server faz o base64), o que é muito mais legível que `data`:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: store-db-secret
  namespace: store-dev
type: Opaque
stringData:
  POSTGRES_USER: store
  POSTGRES_PASSWORD: "FiapStore#2026"
  DATABASE_URL: "postgresql://store:FiapStore%232026@store-db-0.store-db.store-dev.svc.cluster.local:5432/fiapstore"
```

#### 8.5.3 base64 NÃO é criptografia

Este é o ponto mais importante da seção. Base64 é **codificação**, não cifra: qualquer um decodifica em um comando.

```bash
kubectl get secret store-db-secret -n store-dev -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d
```

```
FiapStore#2026
```

Consequências práticas:

- **Nunca** faça commit de um manifesto de Secret em Git. Um `git push` de um YAML com `data:` é vazamento de senha em texto claro, com histórico permanente.
- Quem tem `get secrets` no namespace tem **todas** as senhas do namespace. Trate esse verbo como privilégio administrativo.
- Por padrão, o etcd armazena Secrets **sem criptografia**. Quem lê o disco do etcd (ou um backup dele) lê tudo.

**Encryption at rest.** Habilita-se com um `EncryptionConfiguration` passado ao API server:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - kms:
          apiVersion: v2
          name: aws-kms
          endpoint: unix:///var/run/kmsplugin/socket.sock
      - identity: {}
```

No **EKS**, isso é feito com um clique conceitual: KMS envelope encryption via chave gerenciada por você.

```bash
eksctl utils enable-secrets-encryption \
  --cluster fiap-store \
  --key-arn arn:aws:kms:us-east-1:123456789012:key/1a2b3c4d-... \
  --region us-east-1
```

> **Atenção:** habilitar a criptografia **não** re-criptografa Secrets já existentes automaticamente em todos os casos; é comum precisar reescrevê-los (`kubectl get secrets -A -o json | kubectl replace -f -`). E a operação é irreversível no EKS — não dá para desabilitar.

#### 8.5.4 Consumo: env x volume

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
spec:
  replicas: 2
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
    spec:
      imagePullSecrets:
        - name: regcred
      containers:
        - name: api
          image: docker.io/seuusuario/store-api:1.0
          ports:
            - containerPort: 8000
          env:
            - name: APP_ENV
              value: dev
            - name: DB_HOST
              value: store-db-0.store-db.store-dev.svc.cluster.local
            # forma 1: variavel de ambiente
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: store-db-secret
                  key: POSTGRES_PASSWORD
          volumeMounts:
            # forma 2: arquivo
            - name: credenciais
              mountPath: /etc/secrets
              readOnly: true
      volumes:
        - name: credenciais
          secret:
            secretName: store-db-secret
            defaultMode: 0400
            items:
              - key: POSTGRES_PASSWORD
                path: db_password
```

| Critério | Variável de ambiente | Volume |
|---|---|---|
| Rotação sem reiniciar | Não (congelada na criação do container) | Sim (kubelet atualiza em ~1 min) |
| Vazamento em logs/crash dump | Alto — aparece em `kubectl describe pod`, `/proc/<pid>/environ`, dumps | Baixo |
| Vazamento por subprocessos | Herda para todo processo filho | Não |
| Facilidade de uso na app | Máxima (`os.getenv`) | Requer ler arquivo |
| Aparece em `env` de container comprometido | Sim | Não diretamente |

A recomendação de segurança é clara: **prefira volume** para credenciais de longa duração. Variáveis de ambiente são aceitáveis para conveniência em desenvolvimento e para segredos de baixo impacto.

> **Nota:** `envFrom.secretRef` injeta **todas** as chaves do Secret como variáveis. Prático e perigoso: uma chave nova adicionada ao Secret aparece automaticamente no container, e nomes de chave inválidos como variável são silenciosamente descartados.

#### 8.5.5 Além dos Secrets nativos

Secret nativo resolve o transporte, não a gestão: não há rotação automática, nem auditoria de acesso, nem versionamento. Em produção, o padrão é manter a fonte da verdade fora do cluster:

| Solução | Como funciona | Prós / contras |
|---|---|---|
| **External Secrets Operator (ESO)** | CRD `ExternalSecret` sincroniza de AWS Secrets Manager / SSM / Vault para um Secret nativo | Padrão de mercado no EKS; o Secret ainda existe no etcd |
| **Secrets Store CSI Driver** | Monta o segredo direto do provedor como volume, sem criar Secret | Nada no etcd; exige o driver como DaemonSet |
| **AWS Secrets Manager** | Fonte da verdade com rotação automática (inclusive de senha de RDS) | Custo por segredo/mês e por chamada de API |
| **Sealed Secrets (Bitnami)** | Cifra o Secret com a chave pública do controller; o `SealedSecret` **pode** ir para o Git | Perfeito para GitOps simples; recuperação depende da chave do controller |
| **SOPS (+ age/KMS)** | Cifra apenas os valores do YAML; integra com Helm/Flux | Ótimo em GitOps; requer disciplina de chaves |
| **HashiCorp Vault** | Segredos dinâmicos com TTL (credenciais de banco efêmeras) | Mais poderoso e mais complexo de operar |

Exemplo com ESO, o mais comum em EKS:

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: store-db-secret
  namespace: store-dev
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secretsmanager
    kind: ClusterSecretStore
  target:
    name: store-db-secret
    creationPolicy: Owner
  data:
    - secretKey: POSTGRES_PASSWORD
      remoteRef:
        key: prod/fiapstore/db
        property: password
```

#### 8.5.6 ConfigMap x Secret

| Aspecto | ConfigMap | Secret |
|---|---|---|
| Conteúdo | Configuração não sensível | Dados sensíveis |
| Armazenamento no etcd | Texto claro | Texto claro por padrão; cifrado com KMS/EncryptionConfiguration |
| No nó | Disco (kubelet) | tmpfs (memória) |
| Limite de tamanho | ~1 MiB | ~1 MiB |
| `immutable` | Sim | Sim |
| Aparece em `kubectl describe` | Valores visíveis | Só `<nome>: N bytes` |
| RBAC típico | Amplo (`view` lê) | Restrito (`view` **não** lê Secrets) |

---

### 8.6 Endpoints e EndpointSlice

Você já usa Services desde o capítulo 3. Vamos agora abrir a caixa: **como um Service sabe para quais IPs balancear?**

O **EndpointSlice controller** observa Services e Pods. Para cada Service com `selector`, ele encontra os Pods que casam com o selector **e estão prontos** (`Ready`) e materializa a lista em objetos `EndpointSlice`. O `kube-proxy` (ou o CNI, no modo eBPF do Cilium, ou o AWS VPC CNI com o controller correspondente) observa esses EndpointSlices e programa as regras de iptables/IPVS em cada nó.

```
  Service (selector: app=store-api)
        │  observa
        ▼
  EndpointSlice controller  ──►  EndpointSlice store-api-x7k2p
        │                          addresses: [10.244.1.7, 10.244.2.3]
        │                          conditions: {ready: true}
        ▼  observado por
  kube-proxy em cada no  ──►  regras iptables/IPVS  ──►  Pod
```

O objeto **Endpoints** (sem "Slice") é a versão original: uma única lista por Service. Ele não escalava — um Service com 5.000 Pods gerava um objeto enorme, e qualquer mudança de um Pod reenviava o objeto inteiro para todos os nós. O **EndpointSlice** (`discovery.k8s.io/v1`, padrão desde a 1.21) fatia a lista em pedaços de até 100 endpoints. `Endpoints` ainda existe por compatibilidade e está formalmente **deprecado desde a 1.33**.

```bash
kubectl get endpointslices -n store-dev
```

```
NAME              ADDRESSTYPE   PORTS   ENDPOINTS                 AGE
store-api-x7k2p   IPv4          8000    10.244.1.7,10.244.2.3     14m
store-db-9mq4t    IPv4          5432    10.244.1.12               22m
```

```bash
kubectl get endpointslice store-api-x7k2p -n store-dev -o yaml | head -25
```

```yaml
addressType: IPv4
apiVersion: discovery.k8s.io/v1
endpoints:
  - addresses:
      - 10.244.1.7
    conditions:
      ready: true
      serving: true
      terminating: false
    nodeName: ip-10-0-1-45.ec2.internal
    targetRef:
      kind: Pod
      name: store-api-6f9c7b4d55-2wq8t
      namespace: store-dev
```

> **Na prática:** `kubectl get endpointslices` é a primeira coisa a rodar quando "o Service não responde". Se `ENDPOINTS` estiver `<none>`, o problema **não é rede** — é selector errado, label errada no Pod, ou readinessProbe falhando. Note as três condições: `ready` controla o balanceamento normal; `serving` fica `true` mesmo durante o *terminating*, permitindo que conexões em andamento sejam finalizadas; `terminating` marca Pods em encerramento.

#### 8.6.1 Service sem selector + Endpoints manual

Se um Service **não tem selector**, nenhum controller gerencia seus endpoints — você os declara à mão. Esse é o mecanismo para dar um nome interno do cluster a um recurso **externo**: um banco RDS, uma API de pagamentos de terceiros, um servidor legado no datacenter.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: pagamentos-externo
  namespace: store-dev
spec:
  ports:
    - name: https
      protocol: TCP
      port: 443
      targetPort: 443
---
apiVersion: discovery.k8s.io/v1
kind: EndpointSlice
metadata:
  name: pagamentos-externo-manual
  namespace: store-dev
  labels:
    kubernetes.io/service-name: pagamentos-externo
addressType: IPv4
ports:
  - name: https
    protocol: TCP
    port: 443
endpoints:
  - addresses:
      - 203.0.113.42
    conditions:
      ready: true
```

A label `kubernetes.io/service-name` é o que amarra o EndpointSlice ao Service — sem ela, nada acontece. Agora o `store-api` chama `http://pagamentos-externo.store-dev.svc.cluster.local` e nunca precisa saber o IP real. Trocar de fornecedor vira editar um objeto.

A forma antiga, com o objeto `Endpoints`, ainda funciona e aparece em muito material:

```yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: pagamentos-externo   # DEVE ter o mesmo nome do Service
  namespace: store-dev
subsets:
  - addresses:
      - ip: 203.0.113.42
    ports:
      - name: https
        port: 443
        protocol: TCP
```

#### 8.6.2 ExternalName

Quando o destino externo tem **nome DNS** (e não IP fixo — caso de RDS, ElastiCache, S3), o objeto certo é `ExternalName`: o CoreDNS devolve um `CNAME` e nem sequer há balanceamento ou IP de Service envolvido.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-db-rds
  namespace: store-prod
spec:
  type: ExternalName
  externalName: fiapstore.cluster-abc123.us-east-1.rds.amazonaws.com
```

```bash
kubectl run dns-test --rm -it --image=busybox:1.36 --restart=Never -n store-prod -- \
  nslookup store-db-rds.store-prod.svc.cluster.local
```

```
Name:      store-db-rds.store-prod.svc.cluster.local
Address 1: 10.0.4.87 fiapstore.cluster-abc123.us-east-1.rds.amazonaws.com
```

Isso permite que o mesmo Deployment use `DB_HOST=store-db-rds` em produção (apontando para RDS) e `DB_HOST=store-db-0.store-db` em desenvolvimento (apontando para o StatefulSet local) — mudando só o Service, nunca a aplicação.

> **Atenção:** `ExternalName` não faz proxy TCP e não termina TLS. O certificado apresentado será o do host real, então o cliente precisa validar contra `fiapstore.cluster-...rds.amazonaws.com`, não contra o nome do Service.

#### 8.6.3 Headless para descoberta direta

`clusterIP: None` faz o CoreDNS devolver **todos os IPs dos Pods** no registro A do Service, em vez de um IP virtual. É o que StatefulSets usam e o que clientes com balanceamento próprio (drivers de banco, gRPC, Cassandra) preferem.

| Recurso | ClusterIP | Headless | Sem selector | ExternalName |
|---|---|---|---|---|
| Tem IP virtual | Sim | Não | Sim | Não |
| Endpoints gerenciados | Automático | Automático | **Manual** | Não existem |
| DNS retorna | IP do Service | IPs dos Pods | IP do Service | CNAME externo |
| Uso típico | `store-api` | `store-db` (StatefulSet) | API de pagamentos por IP | RDS/ElastiCache |

---

### 8.7 Laboratório guiado (feito em sala)

**Passo 0 — preparar**

```bash
kubectl create namespace store-dev 2>/dev/null
kubectl config set-context --current --namespace=store-dev
```

**Passo 1 — criar o Secret e provar que base64 não protege nada**

```bash
kubectl create secret generic store-db-secret \
  --from-literal=POSTGRES_USER=store \
  --from-literal=POSTGRES_PASSWORD='FiapStore#2026'

kubectl get secret store-db-secret -o yaml
```

```yaml
apiVersion: v1
data:
  POSTGRES_PASSWORD: RmlhcFN0b3JlIzIwMjY=
  POSTGRES_USER: c3RvcmU=
kind: Secret
metadata:
  name: store-db-secret
  namespace: store-dev
type: Opaque
```

```bash
kubectl get secret store-db-secret -o jsonpath='{.data.POSTGRES_PASSWORD}' | base64 -d; echo
```

```
FiapStore#2026
```

Compare com `kubectl describe secret store-db-secret`, que mostra apenas `POSTGRES_PASSWORD: 14 bytes` — proteção puramente cosmética contra quem olha por cima do ombro.

**Passo 2 — consumir o Secret das duas formas**

Aplique o Deployment da seção 8.5.4 (sem `imagePullSecrets`, se sua imagem for pública) e verifique:

```bash
kubectl apply -f store-api-secret-deploy.yaml
kubectl exec -it deploy/store-api -- printenv DB_PASSWORD
kubectl exec -it deploy/store-api -- ls -l /etc/secrets
kubectl exec -it deploy/store-api -- cat /etc/secrets/db_password; echo
```

```
FiapStore#2026
total 0
lrwxrwxrwx 1 root root 18 Aug  6 15:04 db_password -> ..data/db_password
FiapStore#2026
```

Agora rotacione a senha e observe a diferença crucial:

```bash
kubectl patch secret store-db-secret --type merge \
  -p '{"stringData":{"POSTGRES_PASSWORD":"NovaSenha#2026"}}'
sleep 70
kubectl exec -it deploy/store-api -- cat /etc/secrets/db_password; echo   # atualizado
kubectl exec -it deploy/store-api -- printenv DB_PASSWORD                # antigo!
```

```
NovaSenha#2026
FiapStore#2026
```

Essa é a demonstração mais importante da aula sobre Secrets: **a variável de ambiente ficou desatualizada**. Para atualizá-la, é obrigatório `kubectl rollout restart deployment/store-api`.

**Passo 3 — criar o CronJob de fechamento diário**

Para ver o resultado em aula, use `*/2 * * * *` em vez de `0 2 * * *`:

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: fechamento-diario
  namespace: store-dev
spec:
  schedule: "*/2 * * * *"
  timeZone: "America/Sao_Paulo"
  concurrencyPolicy: Forbid
  startingDeadlineSeconds: 120
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 300
      ttlSecondsAfterFinished: 900
      template:
        metadata:
          labels:
            app: store-worker
            tier: batch
        spec:
          restartPolicy: Never
          containers:
            - name: fechamento
              image: busybox:1.36
              env:
                - name: DB_PASSWORD
                  valueFrom:
                    secretKeyRef:
                      name: store-db-secret
                      key: POSTGRES_PASSWORD
              command:
                - /bin/sh
                - -c
                - |
                  echo "=== FECHAMENTO FIAP STORE ==="
                  echo "Data.......: $(date -Iseconds)"
                  echo "Pod........: $(hostname)"
                  echo "Senha DB len: ${#DB_PASSWORD} caracteres (nao imprimimos o valor)"
                  echo "Pedidos do dia: 137"
                  echo "Faturamento...: R$ 24.310,55"
                  echo "=== FIM ==="
              resources:
                requests:
                  cpu: 50m
                  memory: 64Mi
                limits:
                  cpu: 200m
                  memory: 128Mi
```

```bash
kubectl apply -f fechamento-cronjob.yaml
kubectl get cronjob
```

```
NAME                SCHEDULE      TIMEZONE            SUSPEND   ACTIVE   LAST SCHEDULE   AGE
fechamento-diario   */2 * * * *   America/Sao_Paulo   False     0        <none>          8s
```

**Passo 4 — acompanhar a cadeia CronJob → Job → Pod**

```bash
kubectl get jobs -w
```

```
NAME                         STATUS     COMPLETIONS   DURATION   AGE
fechamento-diario-29387420   Running    0/1                      2s
fechamento-diario-29387420   Complete   1/1            5s         5s
```

```bash
kubectl logs job/fechamento-diario-29387420
```

```
=== FECHAMENTO FIAP STORE ===
Data.......: 2026-08-06T15:12:03+00:00
Pod........: fechamento-diario-29387420-x9qkt
Senha DB len: 14 caracteres (nao imprimimos o valor)
Pedidos do dia: 137
Faturamento...: R$ 24.310,55
=== FIM ===
```

Espere passar 8 minutos (4 execuções) e verifique que só 3 Jobs concluídos permanecem — `successfulJobsHistoryLimit: 3` em ação:

```bash
kubectl get jobs
```

```
NAME                         STATUS     COMPLETIONS   DURATION   AGE
fechamento-diario-29387422   Complete   1/1           5s         6m
fechamento-diario-29387424   Complete   1/1           5s         4m
fechamento-diario-29387426   Complete   1/1           5s         2m
```

**Passo 5 — disparar manualmente e suspender**

```bash
kubectl create job --from=cronjob/fechamento-diario fechamento-teste
kubectl logs job/fechamento-teste
kubectl patch cronjob fechamento-diario -p '{"spec":{"suspend":true}}'
```

**Passo 6 — Service sem selector apontando para serviço externo**

```bash
kubectl apply -f pagamentos-externo-svc.yaml
kubectl get svc pagamentos-externo
kubectl get endpointslices -l kubernetes.io/service-name=pagamentos-externo
kubectl describe svc pagamentos-externo
```

```
NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
pagamentos-externo   ClusterIP   10.96.184.203   <none>        443/TCP   6s

NAME                        ADDRESSTYPE   PORTS   ENDPOINTS      AGE
pagamentos-externo-manual   IPv4          443     203.0.113.42   6s
```

Repare em `Selector: <none>` na saída do `describe` e no fato de haver endpoint mesmo assim. Teste a resolução e a conexão:

```bash
kubectl run curl-test --rm -it --image=curlimages/curl:8.10.1 --restart=Never -- \
  curl -sS -o /dev/null -w '%{http_code}\n' --max-time 5 https://pagamentos-externo/health
```

**Passo 7 — comparar com um Service com selector "quebrado"**

```bash
kubectl create deployment store-api --image=docker.io/seuusuario/store-api:1.0 --port=8000
kubectl expose deployment store-api --port=80 --target-port=8000
kubectl patch svc store-api -p '{"spec":{"selector":{"app":"store-api-errado"}}}'
kubectl get endpointslices -l kubernetes.io/service-name=store-api
```

```
NAME              ADDRESSTYPE   PORTS   ENDPOINTS   AGE
store-api-4h9tz   IPv4          <unset> <unset>     3m
```

`ENDPOINTS` vazio: sintoma clássico de selector errado. Corrija e observe o endpoint reaparecer em segundos.

**Passo 8 — limpeza**

```bash
kubectl delete cronjob fechamento-diario
kubectl delete job --all
kubectl delete svc pagamentos-externo store-api
kubectl delete deployment store-api
kubectl delete secret store-db-secret
```

---

### 8.8 Ponte com Serverless

Job e CronJob são a porta de entrada natural para o pensamento serverless, porque compartilham a mesma semântica: **executar, terminar, sumir**. A diferença está em quem mantém a infraestrutura de pé entre as execuções.

**CronJob x EventBridge Scheduler + Lambda.** Um CronJob de fechamento diário exige um cluster ligado 24 horas por dia para rodar 40 segundos por dia. Você paga por 86.400 segundos de nós para usar 40. O equivalente serverless é uma regra do **Amazon EventBridge Scheduler** (`cron(0 2 * * ? *)`, com suporte nativo a fuso horário e a `flexibleTimeWindow` — o análogo do `startingDeadlineSeconds`) disparando uma função Lambda. Você paga por invocação e por GB-segundo: 30 execuções mensais de 40 segundos com 512 MB custam centavos, contra dezenas de dólares mensais de um node group. Por outro lado, o Lambda tem **teto de 15 minutos** de execução, 10 GB de memória e 10 GB de `/tmp`. Um fechamento contábil que varre milhões de linhas simplesmente não cabe — e aí a resposta é **AWS Batch** ou **ECS/Fargate Task** (sem limite de tempo), ou o próprio Job no Kubernetes. A regra prática: tarefa curta e episódica vai para Lambda; tarefa longa e pesada fica em Job/Batch.

**KEDA e a escala a zero.** O padrão de work queue da seção 8.4.1 tem o defeito de exigir workers fixos. O **KEDA (Kubernetes Event-Driven Autoscaling)** — projeto CNCF graduado — resolve isso trazendo o modelo serverless para dentro do cluster: ele observa a fonte de eventos (SQS, Kafka, RabbitMQ, Redis, Prometheus, Azure Queue) e escala a carga de trabalho **de zero a N e de volta a zero**. O `ScaledObject` faz isso para Deployments; o `ScaledJob` cria **um Job por lote de mensagens**, o que é o casamento perfeito com o tema deste capítulo.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: keda-aws-creds
  namespace: store-dev
type: Opaque
stringData:
  AWS_ACCESS_KEY_ID: "AKIAEXAMPLE"
  AWS_SECRET_ACCESS_KEY: "exemploDeChaveSecreta"
---
apiVersion: keda.sh/v1alpha1
kind: TriggerAuthentication
metadata:
  name: keda-sqs-auth
  namespace: store-dev
spec:
  secretTargetRef:
    - parameter: awsAccessKeyID
      name: keda-aws-creds
      key: AWS_ACCESS_KEY_ID
    - parameter: awsSecretAccessKey
      name: keda-aws-creds
      key: AWS_SECRET_ACCESS_KEY
---
apiVersion: keda.sh/v1alpha1
kind: ScaledJob
metadata:
  name: store-worker-sqs
  namespace: store-dev
spec:
  jobTargetRef:
    backoffLimit: 3
    template:
      spec:
        restartPolicy: Never
        containers:
          - name: worker
            image: docker.io/seuusuario/store-worker:1.0
            command: ["python", "-u", "/app/consome_fila.py"]
            resources:
              requests:
                cpu: 100m
                memory: 128Mi
  pollingInterval: 15
  maxReplicaCount: 20
  minReplicaCount: 0
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 3
  scalingStrategy:
    strategy: default
  triggers:
    - type: aws-sqs-queue
      authenticationRef:
        name: keda-sqs-auth
      metadata:
        queueURL: https://sqs.us-east-1.amazonaws.com/123456789012/fiapstore-pedidos
        queueLength: "5"
        awsRegion: us-east-1
```

Com `minReplicaCount: 0` e fila vazia, **nenhum Pod existe**. Chegam 50 mensagens com `queueLength: 5` e o KEDA cria 10 Jobs. Esvaziada a fila, tudo volta a zero. Em produção, troque as credenciais estáticas por **IRSA** ou **Pod Identity** — assunto do próximo capítulo — e o `TriggerAuthentication` passa a usar `podIdentity: {provider: aws}`.

**O custo do "zero".** Escala a zero no Kubernetes é zero de *Pods*, não zero de *cluster*: os nós e o control plane do EKS (cerca de US$ 0,10/hora, ~US$ 73/mês) continuam sendo cobrados. Zero real só com **Fargate** ou **Lambda**. Em compensação, o cold start de um Job KEDA é da ordem de segundos (pull da imagem + agendamento) contra dezenas ou centenas de milissegundos do Lambda — e sem os limites de 15 minutos e 10 GB. Escolher entre os dois é escolher qual restrição dói menos.

---

### 8.9 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| Job `Failed` com `BackoffLimitExceeded` | Comando saindo com código diferente de 0 | `kubectl describe job <j>`; `kubectl logs -l job-name=<j> --tail=50` | Corrigir o comando/imagem; usar `restartPolicy: Never` para preservar os Pods e ler os logs |
| Job travado em `Running` para sempre | Processo bloqueado (I/O, conexão pendurada) | `kubectl get job -o wide`; `kubectl exec` no Pod | Definir `activeDeadlineSeconds` e timeouts na aplicação |
| `Job.spec.template is invalid: restartPolicy: Unsupported value: "Always"` | Copiou o template de um Deployment | Erro no `kubectl apply` | Usar `OnFailure` ou `Never` |
| CronJob criado, mas nenhum Job aparece | `suspend: true`, schedule inválido, ou 100 horários perdidos | `kubectl get cronjob -o wide`; `kubectl describe cronjob` → evento `FailedNeedsStart` | `suspend: false`, corrigir o cron, definir `startingDeadlineSeconds` |
| CronJob roda no horário errado | Fuso do controller-manager (UTC) | `kubectl get cronjob -o jsonpath='{.spec.timeZone}'` (vazio) | Definir `timeZone: "America/Sao_Paulo"` |
| Jobs se acumulam às centenas | Sem `ttlSecondsAfterFinished` e histórico alto | `kubectl get jobs \| wc -l` | Definir `ttlSecondsAfterFinished` e `successfulJobsHistoryLimit` |
| Duas execuções do fechamento simultâneas | `concurrencyPolicy: Allow` (padrão) e execução longa | `kubectl get jobs`; `.status.active` do CronJob | `concurrencyPolicy: Forbid` |
| Pod `CreateContainerConfigError` | Secret/chave referenciada não existe | `kubectl describe pod` → `secret "store-db-secret" not found` ou `key POSTGRES_PASSWORD not found` | Criar o Secret ou corrigir o nome da chave (case-sensitive) |
| `illegal base64 data at input byte N` | Valor colado no campo `data` sem estar em base64 | `kubectl apply` falha na hora | Usar `stringData`, ou `echo -n 'senha' \| base64` (atenção ao `-n`) |
| Senha rotacionada, app continua com a antiga | Secret consumido por `env` | `kubectl exec -- printenv` x `cat` do arquivo montado | `kubectl rollout restart deploy/store-api`, ou migrar para volume |
| `ImagePullBackOff` em registry privado | Falta `imagePullSecrets` ou credencial errada | `kubectl describe pod` → `unauthorized` | `kubectl create secret docker-registry` e referenciar no Pod/ServiceAccount |
| Service responde `connection refused` | EndpointSlice vazio: selector errado ou readinessProbe falhando | `kubectl get endpointslices -l kubernetes.io/service-name=<svc>` | Alinhar labels do Pod ao selector; corrigir a probe |
| Service sem selector não roteia | Falta a label `kubernetes.io/service-name` no EndpointSlice, ou nome do `Endpoints` diferente do Service | `kubectl get endpointslices --show-labels` | Adicionar a label / renomear o Endpoints para bater com o Service |
| `ExternalName` retorna erro de TLS | Certificado emitido para o host real, não para o nome do Service | `curl -v https://<svc>` | Validar contra o hostname real ou usar proxy reverso |
| KEDA não escala | Trigger sem permissão na fila, ou `TriggerAuthentication` errado | `kubectl describe scaledjob`; logs do `keda-operator` | Ajustar IRSA/credenciais e o `queueURL` |

---

### 8.10 Resumo do capítulo

Nem toda carga de trabalho é um serviço de longa duração. O Job modela processamento que começa, termina e reporta resultado — com controle fino sobre paralelismo (`completions`/`parallelism`), tolerância a falhas (`backoffLimit`, `podFailurePolicy`), prazo (`activeDeadlineSeconds`) e limpeza (`ttlSecondsAfterFinished`). O CronJob agenda Jobs, e sua correção depende de detalhes frequentemente ignorados: fuso horário explícito, política de concorrência e prazo de partida. Secrets carregam os dados sensíveis que essas cargas precisam, mas sua segurança real vem do RBAC, da criptografia no etcd e da forma de consumo — nunca do base64. Por fim, Endpoints e EndpointSlice revelam o mecanismo por trás do Service e abrem a porta para integrar recursos externos ao cluster com Services sem selector e ExternalName.

Pontos-chave:

1. **Job usa `restartPolicy: OnFailure` ou `Never`**; `Always` é rejeitado. `Never` preserva os Pods falhados e facilita a depuração.
2. **`completionMode: Indexed`** dá identidade a cada Pod (`JOB_COMPLETION_INDEX`), permitindo particionamento estático sem fila externa.
3. **CronJob sem `timeZone` roda em UTC** — a causa mais comum de relatório com data errada no Brasil.
4. **`concurrencyPolicy: Forbid`** é o padrão que você quer em tarefas não idempotentes; `Allow` (o default) permite sobreposição.
5. **base64 não é criptografia.** Proteja Secrets com RBAC, encryption at rest (KMS no EKS) e mantendo a fonte da verdade fora do Git (ESO, Sealed Secrets, SOPS, Vault).
6. **Secret por volume rotaciona sozinho; por variável de ambiente, não.** Trocar a senha exige `rollout restart`.
7. **EndpointSlice vazio é diagnóstico, não sintoma:** significa selector errado ou Pod não-`Ready`, nunca problema de rede.
8. **Service sem selector + EndpointSlice manual** (com a label `kubernetes.io/service-name`) dá nome interno a recursos externos; `ExternalName` faz o mesmo quando o destino tem DNS.

---

### 8.11 Exercícios (para casa)

#### Exercício 8.1 — Job paralelo com logs agregados
*Tempo estimado: 25 min · Nível: fixação guiada*

- **Objetivo:** executar processamento batch paralelo e ler o resultado consolidado das réplicas.
- **Enunciado:** no namespace `store-dev`, crie um Job `processa-pedidos` com `completions: 10` e `parallelism: 3`, `backoffLimit: 4` e `ttlSecondsAfterFinished: 600`. Cada Pod deve imprimir o próprio hostname, dormir um tempo aleatório entre 3 e 8 segundos e imprimir uma linha final `LOTE OK <hostname>`. Acompanhe com `kubectl get job -w` e observe que nunca há mais de 3 Pods `Running` ao mesmo tempo (registre a saída de `kubectl get pods` em pelo menos dois momentos distintos durante a execução). Ao final, colete os logs agregados de todos os Pods de uma só vez usando o seletor `job-name`. Em seguida, crie uma segunda versão do Job com `completionMode: Indexed` e `completions: 4`, fazendo cada Pod imprimir a partição que processou; comprove que os quatro índices de 0 a 3 apareceram, sem repetição.
- **Entregável:** `job-paralelo.yaml`, `job-indexed.yaml`, saídas de `kubectl get job,pods -n store-dev` durante a execução, saída de `kubectl logs -l job-name=processa-pedidos -n store-dev --prefix --tail=1` e a listagem dos 4 índices distintos.
- **Critério de aceite:** `COMPLETIONS` chega a `10/10` com STATUS `Complete`; nenhuma amostra de `kubectl get pods` mostra mais de 3 Pods em `Running`; os logs agregados contêm 10 linhas `LOTE OK`; e no Job Indexed aparecem exatamente os índices 0, 1, 2 e 3.
- **Dica:**
  <details><summary>Dica</summary>
  Todo Pod de um Job recebe automaticamente a label <code>job-name=&lt;nome&gt;</code> (e <code>batch.kubernetes.io/job-name</code> nas versões recentes). Para o tempo aleatório em <code>busybox</code>: <code>sleep $(( (RANDOM % 6) + 3 ))</code>.
  </details>

#### Exercício 8.2 — CronJob com concorrência controlada e Secret de duas formas
*Tempo estimado: 40 min · Nível: aplicação*

- **Objetivo:** operar um CronJob real e comprovar experimentalmente a diferença entre consumir Secret por env e por volume.
- **Enunciado:** crie um CronJob `sincroniza-catalogo` com `schedule: "*/5 * * * *"`, `timeZone: "America/Sao_Paulo"`, `startingDeadlineSeconds: 120`, `successfulJobsHistoryLimit: 2` e `failedJobsHistoryLimit: 1`. O container deve durar deliberadamente **6 minutos** (`sleep 360`), de modo que a execução seguinte encontre a anterior ainda ativa. Rode primeiro com `concurrencyPolicy: Allow` por 12 minutos, registrando quantos Jobs ficaram ativos ao mesmo tempo; depois altere para `Forbid`, rode outros 12 minutos e registre a diferença, capturando o evento do CronJob que indica a execução pulada. Paralelamente, crie um Secret `catalogo-secret` com a chave `API_TOKEN` e faça o Pod consumi-lo **simultaneamente** como variável de ambiente `API_TOKEN` e como arquivo em `/etc/catalogo/token` (com `defaultMode: 0400`). Durante uma execução ativa, altere o valor do Secret com `kubectl patch` e comprove, dentro do mesmo Pod, que o arquivo mudou e a variável de ambiente não. Documente tudo em `ANALISE.md`.
- **Entregável:** `sincroniza-cronjob.yaml` (as duas versões), `catalogo-secret.yaml`, `ANALISE.md`, saídas de `kubectl get cronjob,jobs,pods -n store-dev` nos dois cenários, o evento do `kubectl describe cronjob` referente à execução pulada, e as saídas de `printenv API_TOKEN` e `cat /etc/catalogo/token` antes e depois do patch.
- **Critério de aceite:** com `Allow` existem ≥2 Jobs simultaneamente ativos; com `Forbid` existe no máximo 1 e o `describe` mostra o evento `JobAlreadyActive` / "Job already active, skipping"; o histórico nunca ultrapassa 2 Jobs concluídos; e a evidência mostra o arquivo atualizado com a variável de ambiente desatualizada no mesmo Pod.
- **Dica:**
  <details><summary>Dica</summary>
  <code>kubectl get cronjob sincroniza-catalogo -o jsonpath='{.status.active}'</code> lista os Jobs ativos. A atualização do arquivo montado leva até ~60-70 segundos (período de sincronização do kubelet) — não conclua cedo demais.
  </details>

#### Exercício 8.3 — Depuração: Job estourado, Secret inválido e Service sem endpoints
*Tempo estimado: 35 min · Nível: análise/depuração*

- **Objetivo:** diagnosticar três falhas distintas de batch, configuração sensível e descoberta de serviço.
- **Enunciado:** reproduza os três cenários abaixo e, para **cada um**, registre: (i) o STATUS/estado observado, (ii) o comando que revelou a causa e a mensagem exata, (iii) a correção aplicada, (iv) a evidência da correção.
  - **Cenário A:** Job `relatorio-quebrado` com `backoffLimit: 2`, `restartPolicy: Never`, cujo comando sai com código 1 (por exemplo, tentando ler um arquivo inexistente). Observe as tentativas, os Pods `Error` acumulados e a condição final do Job.
  - **Cenário B:** Secret criado por manifesto usando o campo `data:` com um valor **que não é base64 válido** e, na sequência, um segundo Secret com base64 válido porém gerado **com quebra de linha** (`echo 'senha' | base64`, sem o `-n`). Mostre o erro do primeiro e prove, no segundo, que a senha chega ao container com um `\n` a mais — comparando o tamanho esperado com `${#VAR}` ou `wc -c`.
  - **Cenário C:** Deployment `store-api` com label `app: store-api` e Service com `selector: app: storeapi` (sem hífen). Mostre o EndpointSlice vazio, prove a falha de conexão a partir de outro Pod e corrija. Em seguida, provoque a **mesma** falha de forma diferente: mantenha o selector correto mas quebre a `readinessProbe` (aponte para `/pronto`, que não existe) e mostre que o endpoint some com o Pod ainda `Running`.
- **Entregável:** `DEPURACAO.md` com as 4 informações por cenário; os manifestos quebrados e corrigidos; saídas de `kubectl describe job`, `kubectl get pods`, `kubectl get endpointslices -o wide` e do teste de conexão.
- **Critério de aceite:** o Cenário A apresenta `reason: BackoffLimitExceeded` e exatamente 3 Pods em `Error` (1 tentativa + 2 retentativas); o Cenário B apresenta a mensagem `illegal base64 data` e a evidência numérica do caractere extra; o Cenário C mostra `ENDPOINTS` vazio nas duas causas distintas e populado após cada correção.
- **Dica:**
  <details><summary>Dica</summary>
  Em <code>restartPolicy: Never</code>, cada retentativa cria um Pod novo — conte-os. Na readinessProbe, o Pod fica <code>1/1 Running</code>? Confira <code>kubectl get pod</code> na coluna READY e o <code>describe</code> na seção Conditions (<code>Ready: False</code>, <code>ContainersReady: False</code>).
  </details>

#### Exercício 8.4 — Desafio: processamento de fila com escala a zero (KEDA)
*Tempo estimado: 60 min · Nível: desafio integrador*

- **Objetivo:** construir um pipeline event-driven com escala a zero e compará-lo com a alternativa serverless gerenciada.
- **Enunciado:** implemente o processamento da fila de pedidos da FIAP Store com escala a zero comprovada. Escolha **uma** das trilhas:
  - **Trilha A (KEDA):** instale o KEDA (`helm repo add kedacore https://kedacore.github.io/charts && helm install keda kedacore/keda -n keda --create-namespace`). Suba um Redis no cluster (`redis:7-alpine`) como fila e crie um `ScaledJob` (ou `ScaledObject` sobre um Deployment `store-worker`) com trigger `redis` (`listName: pedidos`, `listLength: "5"`), `minReplicaCount: 0` e `maxReplicaCount: 10`. Comprove: (a) com a fila vazia, `kubectl get pods -l app=store-worker` retorna vazio; (b) após inserir 50 itens com `redis-cli RPUSH`, o KEDA cria múltiplos Jobs/Pods; (c) esvaziada a fila, o cluster volta a zero Pods.
  - **Trilha B (simulação com HPA):** se não puder instalar o KEDA, use um HPA sobre métrica customizada (ou, na falta do metrics adapter, um HPA de CPU sobre um worker que consuma CPU proporcionalmente ao tamanho da fila) e documente explicitamente **por que o HPA nativo não consegue escalar a zero** (`minReplicas` mínimo é 1) e o que o KEDA faz de diferente.
  Em ambas as trilhas, produza `COMPARACAO.md` (mínimo 500 palavras) contrastando três arquiteturas para o mesmo problema: (1) Job com `parallelism` fixo, (2) KEDA ScaledJob, (3) SQS + Lambda com event source mapping. Cubra obrigatoriamente: comportamento sob pico de 10.000 mensagens; latência de cold start em cada uma; limite de tempo de execução por item (15 min no Lambda x ilimitado no Job); modelo de custo com números (custo/hora de um node group `t3.medium` x preço por invocação e GB-segundo do Lambda x US$ 0,10/hora do control plane do EKS); e tratamento de falhas (backoffLimit x DLQ do SQS x `podFailurePolicy`). Encerre com uma recomendação justificada para a FIAP Store, considerando que o pico de pedidos ocorre na Black Friday e o volume no restante do ano é baixo.
- **Entregável:** `keda-scaledjob.yaml` (ou `hpa-simulacao.yaml` + justificativa), `COMPARACAO.md`, e evidências em sequência temporal: `kubectl get pods -n store-dev` com zero Pods, durante o pico e após o esvaziamento, mais `kubectl describe scaledjob store-worker-fila -n store-dev` (ou `describe hpa`).
- **Critério de aceite:** existem três capturas distintas de `kubectl get pods` provando o ciclo zero → N → zero (na Trilha B, a documentação explica corretamente a limitação de `minReplicas: 1`); o `COMPARACAO.md` cobre os cinco eixos exigidos com números concretos e apresenta recomendação justificada.
- **Dica:**
  <details><summary>Dica</summary>
  Para popular a fila: <code>kubectl exec -it deploy/redis -- sh -c 'for i in $(seq 1 50); do redis-cli RPUSH pedidos "pedido-$i"; done'</code>. Monitore com <code>watch -n2 'kubectl get pods -n store-dev; kubectl exec deploy/redis -- redis-cli LLEN pedidos'</code>. O KEDA leva até <code>pollingInterval</code> segundos para reagir — seja paciente antes de declarar que não funcionou.
  </details>

---

### 8.12 Leitura recomendada

1. [DOC] **Kubernetes — Jobs, CronJob e Pod Failure Policy.** https://kubernetes.io/docs/concepts/workloads/controllers/job/ e https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/ — inclui os padrões de work queue e o comportamento de `startingDeadlineSeconds`.
2. [DOC] **Kubernetes — Secrets e Encrypting Confidential Data at Rest.** https://kubernetes.io/docs/concepts/configuration/secret/ e https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/ — a seção "Risks" da primeira página é leitura obrigatória.
3. [DOC] **Kubernetes — Service, EndpointSlices e Services without selectors.** https://kubernetes.io/docs/concepts/services-networking/endpoint-slices/ — explica as condições `ready`, `serving` e `terminating`.
4. [DOC] **KEDA — Scaling Jobs e AWS SQS Queue Scaler.** https://keda.sh/docs/latest/concepts/scaling-jobs/ — referência do `ScaledJob` e da autenticação por Pod Identity.
5. [ARTIGO] **AWS — EventBridge Scheduler e Lambda com SQS event source mapping.** https://docs.aws.amazon.com/scheduler/latest/UserGuide/ — para a comparação de custo e limites do exercício 8.4.
6. [LIVRO] **BURNS, B.; BEDA, J.; HIGHTOWER, K.; EVENSON, L.** *Kubernetes: Up and Running*, 3. ed. O'Reilly, 2022 — capítulos "Jobs" e "ConfigMaps and Secrets".
7. [LIVRO] **ARUNDEL, J.; DOMINGUS, J.** *Cloud Native DevOps with Kubernetes*, 2. ed. O'Reilly, 2022 — discussão prática sobre gestão de segredos com SOPS e Sealed Secrets em pipelines GitOps.

---

## Capítulo 9 — Autenticação e Controle de Usuário (RBAC)

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 8 (arquitetura do control plane, namespaces, Secrets, ServiceAccount como conceito, cluster EKS provisionado); noções de PKI (chave privada, CSR, certificado); AWS CLI configurada. · **Artefatos:** `maria-dev.key`, `maria-dev.csr`, `maria-dev.crt`, `csr-maria.yaml`, `role-dev.yaml`, `rolebinding-maria.yaml`, `sa-relatorios-irsa.yaml`, `kubeconfig-maria.yaml`, namespaces `store-dev` e `store-prod`.

---

### 9.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as três fases do controle de acesso no API server — autenticação, autorização e admission control — e o papel de cada uma na negação ou aceitação de uma requisição.
2. **Explicar** por que o Kubernetes não possui um objeto `User` e **distinguir** os modos de autenticação disponíveis (X.509, tokens, ServiceAccount, OIDC, webhook, proxy).
3. **Construir** modelos de autorização com Role, ClusterRole, RoleBinding e ClusterRoleBinding, aplicando `apiGroups`, `resources`, subresources, `verbs` e `resourceNames`.
4. **Aplicar** o fluxo completo de criação de acesso para um novo usuário via CertificateSigningRequest e geração de kubeconfig funcional.
5. **Analisar** erros de autorização reais (`Forbidden`) com `kubectl auth can-i`, inclusive com impersonation (`--as`), corrigindo-os com o mínimo de privilégio necessário.
6. **Aplicar** o mapeamento IAM→RBAC no EKS (Access Entries / aws-auth) e configurar IRSA/Pod Identity para conceder permissões AWS a um Pod, **comparando** o resultado com o modelo de execution role do Lambda.

---

### 9.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | As três fases do controle de acesso; por que não existe objeto `User` |
| 2 | 15 min | Modos de autenticação: X.509, tokens, ServiceAccount projetado, OIDC, webhook, proxy |
| 3 | 25 min | RBAC em profundidade: Role x ClusterRole, bindings, regras, agregação, roles padrão |
| — | 10 min | **Intervalo** |
| 4 | 15 min | `kubectl auth can-i`, impersonation e `auth reconcile`; antipadrões e menor privilégio |
| 5 | 15 min | EKS: Access Entries x aws-auth; IRSA e Pod Identity |
| 6 | 25 min | **Laboratório guiado:** criar "maria.dev" via CSR, kubeconfig, Role no `store-dev`, ServiceAccount com IRSA |
| 7 | 5 min | Admission control (Gatekeeper/Kyverno), Ponte com Serverless e exercícios |

---

### 9.3 As três fases do controle de acesso

Toda requisição ao API server — de um `kubectl get pods`, de um controller, de um Pod usando a API — atravessa três estágios em sequência. Falhar em qualquer um encerra a requisição.

```
   kubectl / SDK / curl
          │
          ▼
   ┌─────────────────────────────────────────────────────────┐
   │                      kube-apiserver                     │
   │                                                         │
   │  1. AUTHENTICATION   Quem e voce?                       │
   │     X.509 | token | OIDC | webhook | proxy              │
   │     falha -> 401 Unauthorized                           │
   │                        │                                │
   │                        ▼                                │
   │  2. AUTHORIZATION    Voce pode fazer isso?              │
   │     Node | RBAC | Webhook | (ABAC)                      │
   │     falha -> 403 Forbidden                              │
   │                        │                                │
   │                        ▼                                │
   │  3. ADMISSION CONTROL   Isso e valido/aceitavel aqui?   │
   │     Mutating -> Validating (Gatekeeper, Kyverno, PSA)   │
   │     falha -> 400 / 403 com a mensagem do webhook        │
   │                        │                                │
   │                        ▼                                │
   │                    etcd (persiste)                      │
   └─────────────────────────────────────────────────────────┘
```

Distinguir 401 de 403 é a primeira habilidade de diagnóstico deste capítulo:

- **401 Unauthorized** — o cluster não conseguiu determinar *quem* você é. Certificado expirado, token inválido, kubeconfig apontando para o cluster errado.
- **403 Forbidden** — o cluster sabe quem você é e decidiu que você **não pode**. A mensagem sempre nomeia o sujeito, o verbo e o recurso.

```
Error from server (Forbidden): pods is forbidden: User "maria.dev" cannot list
resource "pods" in API group "" in the namespace "store-prod"
```

Essa mensagem é um presente: ela entrega o sujeito (`maria.dev`), o verbo (`list`), o recurso (`pods`), o apiGroup (`""`, o core) e o namespace (`store-prod`). Com esses cinco dados você escreve a regra RBAC exata, sem chutar.

---

### 9.4 Autenticação: o Kubernetes não tem usuários

Este é o conceito que mais surpreende quem vem de outros sistemas: **não existe `kubectl create user`**. Não há objeto `User` no etcd, não há tabela de senhas, não há `kubectl get users`. A razão de projeto é deliberada: gerenciar identidades humanas é um problema resolvido por sistemas corporativos (LDAP, Active Directory, Google Workspace, Okta, IAM), e reimplementá-lo dentro do orquestrador significaria mais uma base de credenciais para vazar.

O Kubernetes, então, **delega a autenticação** e se preocupa apenas em extrair de cada requisição três atributos:

- **username** (uma string, por exemplo `maria.dev`);
- **UID** (opcional);
- **groups** (lista de strings, por exemplo `["fiap:desenvolvedores", "system:authenticated"]`).

Quem produz esses atributos é o *authenticator*. O API server tenta cada authenticator habilitado em cadeia; o primeiro que reconhecer a credencial define a identidade.

| Modo | Como a identidade chega | Onde se usa |
|---|---|---|
| **Certificados X.509** | `CN` do certificado vira o username; cada `O` (Organization) vira um group | Administração, acesso de emergência, laboratório |
| **Tokens estáticos** (`--token-auth-file`) | Header `Authorization: Bearer <token>` contra um CSV no disco do control plane | **Obsoleto** — exige reiniciar o API server para revogar; não use |
| **Senhas estáticas básicas** | HTTP Basic Auth | **Removido** desde a 1.19 |
| **ServiceAccount tokens** | JWT assinado pelo cluster; username `system:serviceaccount:<ns>:<nome>` | Identidade de **workloads** (Pods, CI/CD) |
| **OIDC** | JWT de um provedor externo (Okta, Entra ID, Google, Keycloak, Dex) | Identidade de **humanos** em produção |
| **Webhook token** | O API server pergunta a um serviço externo se o token é válido | Integrações customizadas; é como o EKS valida tokens do IAM |
| **Authenticating proxy** | Um proxy confiável injeta os headers `X-Remote-User` e `X-Remote-Group` | Portais e plataformas internas |

> **Nota:** a distinção fundamental é **usuário humano** x **ServiceAccount**. ServiceAccounts são objetos reais do cluster (namespaced, com token gerenciável), feitos para processos. Usuários humanos vêm sempre de fora. Usar ServiceAccount como identidade de pessoa é um antipadrão comum: gera token sem expiração natural, sem auditoria individual e sem revogação centralizada.

#### 9.4.1 Autenticação por certificado X.509

O API server confia em uma CA (definida por `--client-ca-file`, normalmente a CA do cluster). Qualquer certificado válido assinado por ela autentica. O mapeamento é direto:

```
Subject: CN=maria.dev, O=fiap:desenvolvedores, O=fiap:qa
         └────┬────┘  └──────────┬───────────┘
          username              groups
```

É simples e não depende de nenhuma infraestrutura externa — por isso é o método que usaremos no laboratório. Mas tem duas limitações sérias em produção: **não há revogação** (o Kubernetes não consulta CRL nem OCSP; um certificado vazado vale até expirar) e **não há como listar quem tem acesso**, porque não existe registro dos certificados emitidos. Por isso, em produção, certificado é para acesso de emergência (break-glass) e OIDC é para o dia a dia.

#### 9.4.2 ServiceAccounts e o fim dos tokens permanentes

Historicamente, criar uma ServiceAccount gerava automaticamente um Secret do tipo `kubernetes.io/service-account-token` com um JWT **sem expiração**, montado em todo Pod. Se aquele token vazasse — em um log, em um dump, em uma imagem — valia para sempre.

Desde a 1.24 esse comportamento acabou. Hoje o kubelet usa a **TokenRequest API** para obter um token **projetado**, com três propriedades novas:

- **expiração** (`expirationSeconds`, padrão 1 hora, com rotação automática pelo kubelet);
- **audiência** (`aud`), de modo que um token destinado ao API server não sirva para outro serviço;
- **vinculação ao Pod** — o token é invalidado quando o Pod deixa de existir.

O volume `projected` que vimos no capítulo 7 é justamente o mecanismo por trás disso:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api-token-demo
  namespace: store-dev
spec:
  serviceAccountName: store-api-sa
  containers:
    - name: api
      image: docker.io/seuusuario/store-api:1.0
      volumeMounts:
        - name: token-api
          mountPath: /var/run/secrets/tokens
          readOnly: true
  volumes:
    - name: token-api
      projected:
        sources:
          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
              audience: https://kubernetes.default.svc
```

```bash
kubectl exec -it store-api-token-demo -n store-dev -- \
  cat /var/run/secrets/tokens/token | cut -d. -f2 | base64 -d 2>/dev/null
```

```json
{"aud":["https://kubernetes.default.svc"],"exp":1786052400,"iat":1786048800,
 "kubernetes.io":{"namespace":"store-dev","pod":{"name":"store-api-token-demo",
 "uid":"a7c3..."},"serviceaccount":{"name":"store-api-sa","uid":"9f21..."}},
 "sub":"system:serviceaccount:store-dev:store-api-sa"}
```

Repare no `sub`: o username de uma ServiceAccount é sempre `system:serviceaccount:<namespace>:<nome>`, e ela pertence automaticamente aos grupos `system:serviceaccounts` e `system:serviceaccounts:<namespace>`.

Duas configurações importantes:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: store-api-sa
  namespace: store-dev
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/FiapStoreApiRole
automountServiceAccountToken: false
imagePullSecrets:
  - name: regcred
```

`automountServiceAccountToken: false` deve ser o **padrão** para qualquer aplicação que não fala com a API do Kubernetes — e o `store-api` não fala. Montar um token onde ele não é necessário é entregar credencial de graça a quem comprometer o container. Pode ser definido na ServiceAccount (vale para todos os Pods que a usam) ou no Pod (tem precedência).

Para gerar um token manualmente, por exemplo para um pipeline de CI:

```bash
kubectl create token store-api-sa -n store-dev --duration=2h
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IkFCQzEyMyJ9.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjIl0...
```

---

### 9.5 Autorização: os modos e a ordem de avaliação

Passada a autenticação, o API server consulta os *authorizers* habilitados em `--authorization-mode`, em ordem. A lógica é de **união permissiva**: o primeiro que responder `Allow` encerra a decisão com permissão; se um responder `Deny` explícito, encerra negando; se todos responderem `NoOpinion`, o resultado final é negar.

| Modo | Função |
|---|---|
| **Node** | Autorizador especializado que limita cada kubelet a ler apenas os objetos dos Pods do seu próprio nó |
| **ABAC** | Políticas em arquivo JSON no disco do control plane; exige reiniciar o API server para mudar. Legado, não use |
| **RBAC** | Permissões como objetos da API, versionáveis e alteráveis em tempo real. O padrão de fato |
| **Webhook** | Delega a decisão a um serviço externo. É como o EKS avalia Access Entries e políticas do IAM |
| **AlwaysAllow / AlwaysDeny** | Apenas para testes; `AlwaysAllow` desliga a autorização |

Um cluster típico roda `--authorization-mode=Node,RBAC`. No EKS a configuração efetiva inclui o webhook do IAM antes do RBAC.

> **Atenção:** RBAC é **puramente aditivo**. Não existe regra de negação em RBAC — não há `deny`. Se você precisa proibir algo que uma regra concede, a solução não é RBAC, é remover a concessão ou usar admission control (Gatekeeper/Kyverno) ou o `ValidatingAdmissionPolicy` nativo.

---

### 9.6 RBAC em profundidade

Quatro objetos, todos em `rbac.authorization.k8s.io/v1`, e uma regra mental simples:

|  | Define permissões | Concede permissões |
|---|---|---|
| **Namespaced** | `Role` | `RoleBinding` |
| **Cluster-wide** | `ClusterRole` | `ClusterRoleBinding` |

E uma combinação que confunde todo mundo na primeira vez: **um `RoleBinding` pode referenciar um `ClusterRole`**. Nesse caso, as permissões do ClusterRole ficam restritas ao namespace do RoleBinding. É o padrão mais usado na prática — você escreve o ClusterRole `view` uma vez e o concede em vários namespaces com RoleBindings diferentes.

O que **não** funciona: um `RoleBinding` não pode referenciar uma `Role` de outro namespace, e um `ClusterRoleBinding` não pode referenciar uma `Role`.

#### 9.6.1 Anatomia de uma regra

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-store
  namespace: store-dev
rules:
  # 1) recursos do core group (apiGroups: "")
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # 2) subresources: ler logs e abrir shell
  - apiGroups: [""]
    resources: ["pods/log", "pods/status"]
    verbs: ["get", "list"]
  - apiGroups: [""]
    resources: ["pods/exec", "pods/portforward"]
    verbs: ["create"]

  # 3) workloads
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

  # 4) escalar Deployment sem poder recria-lo
  - apiGroups: ["apps"]
    resources: ["deployments/scale"]
    verbs: ["update", "patch"]

  # 5) batch
  - apiGroups: ["batch"]
    resources: ["jobs", "cronjobs"]
    verbs: ["get", "list", "watch", "create", "delete"]

  # 6) resourceNames: acesso a UM secret especifico
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["store-api-config-secret"]
    verbs: ["get"]
```

Pontos a fixar:

**`apiGroups`.** O core group é a **string vazia** `""`, não `"core"` nem `"v1"`. Pods, Services, ConfigMaps, Secrets, Nodes, Namespaces e PVCs estão nele. Deployments e DaemonSets estão em `apps`; Jobs e CronJobs em `batch`; Ingress e NetworkPolicy em `networking.k8s.io`. Errar o apiGroup é a causa número um de "escrevi a Role e continua Forbidden".

Para descobrir o grupo e o nome plural correto de qualquer recurso:

```bash
kubectl api-resources | head -12
```

```
NAME                     SHORTNAMES   APIVERSION   NAMESPACED   KIND
configmaps               cm           v1           true         ConfigMap
endpoints                ep           v1           true         Endpoints
namespaces               ns           v1           false        Namespace
persistentvolumeclaims   pvc          v1           true         PersistentVolumeClaim
pods                     po           v1           true         Pod
secrets                               v1           true         Secret
services                 svc          v1           true         Service
daemonsets               ds           apps/v1      true         DaemonSet
deployments              deploy       apps/v1      true         Deployment
statefulsets             sts          apps/v1      true         StatefulSet
cronjobs                 cj           batch/v1     true         CronJob
jobs                                  batch/v1     true         Job
```

**`resources` usa o nome plural em minúsculas**, exatamente como na coluna NAME acima — nunca o `Kind`.

**Subresources** são escritos com barra e concedidos separadamente. Isso é o que permite separações valiosas: dar `pods` + `pods/log` (ver e ler logs) sem `pods/exec` (abrir shell dentro do container). Em um ambiente com dados de cliente, `pods/exec` é privilégio de produção, não de rotina.

**`verbs`.** Os principais são `get`, `list`, `watch`, `create`, `update`, `patch`, `delete`, `deletecollection`. Dois detalhes práticos: `get` sem `list` permite buscar um objeto pelo nome mas não enumerar; e `kubectl apply` precisa de `patch` **e** `get` (ele lê o objeto atual para calcular o merge) — Roles com apenas `create` e `update` quebram o `apply`.

**`resourceNames`** restringe a regra a objetos com nomes específicos. É poderoso para segredos, mas tem uma limitação importante: **não funciona com `list`, `watch`, `create` nem `deletecollection`**, porque no momento da avaliação dessas operações o nome do objeto ainda não é conhecido ou não é único.

**Wildcards.** `verbs: ["*"]` e `resources: ["*"]` funcionam, e é justamente por isso que aparecem em tantos manifestos copiados da internet. Evite: um wildcard hoje concede também os CRDs que serão instalados amanhã.

#### 9.6.2 Bindings e sujeitos

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: maria-dev-store
  namespace: store-dev
subjects:
  - kind: User
    name: maria.dev
    apiGroup: rbac.authorization.k8s.io
  - kind: Group
    name: fiap:desenvolvedores
    apiGroup: rbac.authorization.k8s.io
  - kind: ServiceAccount
    name: ci-deployer
    namespace: store-dev
roleRef:
  kind: Role
  name: dev-store
  apiGroup: rbac.authorization.k8s.io
```

Detalhes que geram erro de `apply`: sujeitos `User` e `Group` levam `apiGroup: rbac.authorization.k8s.io`; `ServiceAccount` **não leva `apiGroup`** (é core) e **leva `namespace`**. E o campo `roleRef` é **imutável** — para trocar a Role referenciada, apague e recrie o binding.

> **Nota:** o nome do `User` no binding é apenas uma string. Ele **não precisa existir** em lugar nenhum. Você pode criar um RoleBinding para `joao.dev` antes de emitir qualquer certificado — quando alguém se autenticar com esse username, as permissões já estarão lá. É o que significa dizer que "o Kubernetes não tem usuários".

#### 9.6.3 Agregação de ClusterRoles

Uma ClusterRole com `aggregationRule` tem suas `rules` **preenchidas automaticamente** pelo controller, pela união das rules de todas as ClusterRoles cujas labels casem com o seletor. É o mecanismo que permite estender as roles padrão sem editá-las (o que seria desfeito no próximo upgrade do cluster).

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: store-monitoring
  labels:
    rbac.authorization.k8s.io/aggregate-to-view: "true"
rules:
  - apiGroups: ["keda.sh"]
    resources: ["scaledobjects", "scaledjobs"]
    verbs: ["get", "list", "watch"]
```

Aplicando isso, **todo mundo que já tem `view`** passa a enxergar os objetos do KEDA — sem tocar na ClusterRole `view`.

#### 9.6.4 As roles padrão

Todo cluster nasce com um conjunto de ClusterRoles "user-facing":

| ClusterRole | O que concede | Cuidado |
|---|---|---|
| `cluster-admin` | Tudo, em todo lugar (`*`/`*`/`*`) | Equivale a root no cluster |
| `admin` | Leitura/escrita completa em **um namespace** (via RoleBinding), incluindo Roles e RoleBindings; não permite mexer no namespace nem em quotas | Pode conceder permissões a si mesmo dentro do namespace |
| `edit` | Leitura/escrita de workloads em um namespace; **não** gerencia RBAC | **Lê Secrets** — trate como privilégio real |
| `view` | Somente leitura em um namespace; **não lê Secrets** nem roles | O mais próximo de "auditor" pronto de fábrica |

```bash
kubectl get clusterroles | grep -v '^system:' | head
kubectl describe clusterrole view | head -20
```

Concessão típica, sem escrever nenhuma Role própria:

```bash
kubectl create rolebinding qa-view --clusterrole=view --group=fiap:qa -n store-prod
kubectl create rolebinding dev-edit --clusterrole=edit --user=maria.dev -n store-dev
```

Há também as roles de sistema, prefixadas com `system:` — `system:kube-scheduler`, `system:node`, `system:discovery`. Não as edite: o `kubectl auth reconcile` do control plane as restaura a cada reinício.

#### 9.6.5 Escalation prevention

Um usuário não pode criar uma Role com permissões que ele próprio não tenha — caso contrário, qualquer um com `create roles` viraria administrador em dois comandos. As exceções são possuir o verbo especial `escalate` sobre `roles`/`clusterroles`, ou possuir `bind` para associar uma role que você não detém.

```
Error from server (Forbidden): roles.rbac.authorization.k8s.io "super" is forbidden:
user "maria.dev" (groups=["fiap:desenvolvedores" "system:authenticated"]) is attempting
to grant RBAC permissions not currently held
```

---

### 9.7 Ferramentas de verificação

#### 9.7.1 kubectl auth can-i

Antes de abrir um chamado dizendo "não tenho permissão", pergunte ao cluster:

```bash
kubectl auth can-i create deployments -n store-dev
kubectl auth can-i delete pods -n store-prod
kubectl auth can-i get secret/store-db-secret -n store-dev
kubectl auth can-i '*' '*' --all-namespaces
```

```
yes
no
yes
no
```

Para listar tudo o que a identidade atual pode fazer em um namespace:

```bash
kubectl auth can-i --list -n store-dev
```

```
Resources                                       Non-Resource URLs   Resource Names   Verbs
selfsubjectreviews.authentication.k8s.io        []                  []               [create]
pods                                            []                  []               [get list watch create update patch delete]
pods/log                                        []                  []               [get list]
deployments.apps                                []                  []               [get list watch create update patch delete]
secrets                                         []                  [store-api-config-secret]  [get]
```

#### 9.7.2 Impersonation com --as

Um administrador pode **testar as permissões de outra identidade** sem trocar de kubeconfig:

```bash
kubectl auth can-i list pods -n store-prod --as=maria.dev
kubectl auth can-i create deployments -n store-dev --as=maria.dev --as-group=fiap:desenvolvedores
kubectl auth can-i get secrets -n store-dev --as=system:serviceaccount:store-dev:store-api-sa
kubectl get pods -n store-dev --as=maria.dev
```

```
no
yes
no
NAME                         READY   STATUS    RESTARTS   AGE
store-api-6f9c7b4d55-2wq8t   1/1     Running   0          22m
```

Isso é ouro para validar um modelo RBAC **antes** de entregá-lo ao time — é exatamente a técnica que o exercício 9.4 exige como evidência. Note que impersonation é ela própria um privilégio (verbos `impersonate` sobre `users`, `groups`, `serviceaccounts`) e um dos primeiros alvos de escalada: quem pode impersonar `system:masters` é administrador do cluster.

#### 9.7.3 kubectl auth reconcile

`kubectl apply` sobre objetos RBAC pode se comportar de forma indesejada com regras (substituindo listas em vez de mesclar). `kubectl auth reconcile` foi feito especificamente para RBAC: ele cria o que falta e **une** as regras existentes, preservando permissões concedidas por outros.

```bash
kubectl auth reconcile -f role-dev.yaml -f rolebinding-maria.yaml
```

```
rbac.authorization.k8s.io/v1, Kind=Role, namespace=store-dev, name=dev-store reconciled
	reconciliation required create
	missing rules added:
		{Verbs:["get" "list"], APIGroups:[""], Resources:["pods/log"]}
rbac.authorization.k8s.io/v1, Kind=RoleBinding, namespace=store-dev, name=maria-dev-store reconciled
```

Use `--remove-extra-permissions` e `--remove-extra-subjects` quando quiser que o arquivo seja a verdade absoluta (o modo desejável em GitOps).

---

### 9.8 Criando acesso para um novo usuário via CSR

Este é o procedimento completo, do par de chaves ao kubeconfig funcional. É o coração do laboratório.

**Etapa 1 — gerar chave privada e CSR.** A chave privada **nunca sai da máquina da usuária**. O `CN` vira o username; cada `O` vira um group.

```bash
openssl genrsa -out maria-dev.key 2048

openssl req -new -key maria-dev.key -out maria-dev.csr \
  -subj "/CN=maria.dev/O=fiap:desenvolvedores"
```

**Etapa 2 — submeter ao cluster como CertificateSigningRequest.** O campo `request` recebe o CSR em base64 **em uma única linha** (daí o `| tr -d '\n'`).

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: maria-dev-csr
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K...
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 2592000   # 30 dias
  usages:
    - client auth
```

```bash
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: maria-dev-csr
spec:
  request: $(cat maria-dev.csr | base64 | tr -d '\n')
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 2592000
  usages:
    - client auth
EOF
```

O `signerName` importa: `kubernetes.io/kube-apiserver-client` é o único que produz certificados aceitos como credencial de cliente pelo API server. `kubernetes.io/kubelet-serving` e `kubernetes.io/legacy-unknown` servem a outros propósitos.

**Etapa 3 — aprovar.**

```bash
kubectl get csr
```

```
NAME            AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
maria-dev-csr   12s   kubernetes.io/kube-apiserver-client    kubernetes-admin   30d                 Pending
```

```bash
kubectl certificate approve maria-dev-csr
kubectl get csr maria-dev-csr
```

```
NAME            AGE   SIGNERNAME                            REQUESTOR          REQUESTEDDURATION   CONDITION
maria-dev-csr   48s   kubernetes.io/kube-apiserver-client    kubernetes-admin   30d                 Approved,Issued
```

**Etapa 4 — extrair o certificado assinado.**

```bash
kubectl get csr maria-dev-csr -o jsonpath='{.status.certificate}' | base64 -d > maria-dev.crt
openssl x509 -in maria-dev.crt -noout -subject -dates
```

```
subject=CN = maria.dev, O = fiap:desenvolvedores
notBefore=Aug  6 15:40:00 2026 GMT
notAfter=Sep  5 15:40:00 2026 GMT
```

> **Atenção:** o **EKS não assina CSRs com a CA do cluster** para autenticação de cliente — a autenticação é feita pelo IAM. Este fluxo funciona plenamente em minikube, kubeadm, kind e clusters self-managed. No EKS, o caminho equivalente é Access Entries (seção 9.9).

**Etapa 5 — montar o kubeconfig.**

```bash
CLUSTER=$(kubectl config view --minify -o jsonpath='{.clusters[0].name}')
SERVER=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}')
CA=$(kubectl config view --minify --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}')

# se a CA vier vazia (minikube usa arquivo), gere-a a partir do caminho:
[ -z "$CA" ] && CA=$(base64 -w0 "$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.certificate-authority}')")

kubectl config set-cluster $CLUSTER \
  --server=$SERVER --embed-certs=true \
  --certificate-authority=<(echo "$CA" | base64 -d) \
  --kubeconfig=kubeconfig-maria.yaml

kubectl config set-credentials maria.dev \
  --client-certificate=maria-dev.crt \
  --client-key=maria-dev.key \
  --embed-certs=true \
  --kubeconfig=kubeconfig-maria.yaml

kubectl config set-context maria-dev \
  --cluster=$CLUSTER --user=maria.dev --namespace=store-dev \
  --kubeconfig=kubeconfig-maria.yaml

kubectl config use-context maria-dev --kubeconfig=kubeconfig-maria.yaml
```

O resultado (`kubeconfig-maria.yaml`) é um arquivo autocontido, com o certificado e a chave embutidos, pronto para ser entregue. Neste momento a Maria **autentica**, mas ainda não pode fazer nada:

```bash
kubectl --kubeconfig=kubeconfig-maria.yaml get pods -n store-dev
```

```
Error from server (Forbidden): pods is forbidden: User "maria.dev" cannot list
resource "pods" in API group "" in the namespace "store-dev"
```

Isso é a separação entre autenticação e autorização acontecendo diante dos seus olhos. Faltam a Role e o RoleBinding.

**Revogação.** Não existe. Para tirar o acesso da Maria, remova os bindings (`kubectl delete rolebinding maria-dev-store -n store-dev`). O certificado continua autenticando, mas sem permissão alguma. Em caso de comprometimento grave da CA, a única saída real é rotacionar a CA do cluster — operação pesada. Mais um argumento para OIDC em produção.

---

### 9.9 Integração com a nuvem: EKS

#### 9.9.1 IAM → RBAC: Access Entries e aws-auth

No EKS, a autenticação é feita pelo **IAM**: o `aws eks get-token` gera um token assinado (um STS `GetCallerIdentity` pré-assinado), o API server o valida via webhook e traduz a identidade IAM em um username e grupos do Kubernetes. Depois disso, **quem decide o que você pode fazer é o RBAC**, exatamente como estudamos.

O mapeamento historicamente vivia em um ConfigMap chamado `aws-auth` no `kube-system`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::123456789012:role/eksctl-fiap-store-nodegroup-NodeInstanceRole
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - rolearn: arn:aws:iam::123456789012:role/FiapStoreSRERole
      username: sre:{{SessionName}}
      groups:
        - fiap:sre
  mapUsers: |
    - userarn: arn:aws:iam::123456789012:user/maria.dev
      username: maria.dev
      groups:
        - fiap:desenvolvedores
```

Esse ConfigMap era notoriamente perigoso: um erro de YAML podia **trancar todo mundo fora do cluster**, sem recuperação. Desde 2023 a AWS oferece as **EKS Access Entries**, que fazem o mesmo mapeamento via API do IAM/EKS, com validação e sem editar ConfigMap:

```bash
aws eks create-access-entry \
  --cluster-name fiap-store \
  --principal-arn arn:aws:iam::123456789012:user/maria.dev \
  --type STANDARD \
  --username maria.dev \
  --kubernetes-groups fiap:desenvolvedores

# opcional: associar uma policy gerenciada de acesso, com escopo de namespace
aws eks associate-access-policy \
  --cluster-name fiap-store \
  --principal-arn arn:aws:iam::123456789012:user/maria.dev \
  --policy-arn arn:aws:eks::aws:cluster-access-policy/AmazonEKSEditPolicy \
  --access-scope type=namespace,namespaces=store-dev

aws eks list-access-entries --cluster-name fiap-store
```

O modo de autenticação do cluster pode ser `CONFIG_MAP`, `API_AND_CONFIG_MAP` ou `API`. Para clusters novos, use `API`. Depois disso, o kubeconfig da Maria sai pronto de um comando:

```bash
aws eks update-kubeconfig --name fiap-store --region us-east-1 --alias fiap-dev
kubectl auth can-i --list -n store-dev
```

> **Nota (outras nuvens):** no GKE, o mapeamento é direto de identidades Google/IAM para usuários RBAC (com o e-mail como username); no AKS, integração com Microsoft Entra ID via OIDC e Azure RBAC for Kubernetes. Em todos os casos o modelo é o mesmo: a nuvem autentica, o RBAC autoriza.

#### 9.9.2 IRSA e Pod Identity: dando permissão AWS a um Pod

RBAC controla o acesso à API do Kubernetes. Ele **não diz nada** sobre acessar S3, DynamoDB ou SQS. Para isso existe o **IRSA (IAM Roles for Service Accounts)**: o Pod recebe um token OIDC projetado, o SDK da AWS o troca por credenciais temporárias via STS `AssumeRoleWithWebIdentity`, e pronto — nenhuma chave estática dentro do container.

```
  Pod (SA: store-relatorios)
    │ token OIDC projetado em
    │ /var/run/secrets/eks.amazonaws.com/serviceaccount/token
    ▼
  AWS STS  AssumeRoleWithWebIdentity
    │ valida a assinatura contra o OIDC provider do cluster
    │ confere sub == system:serviceaccount:store-dev:store-relatorios
    ▼
  Credenciais temporarias (1h, rotacionadas pelo SDK)
    ▼
  S3 / DynamoDB / SQS
```

```bash
# 1. associar o provedor OIDC ao cluster (uma vez por cluster)
eksctl utils associate-iam-oidc-provider --cluster fiap-store --region us-east-1 --approve

# 2. criar a role IAM ja vinculada a uma ServiceAccount
eksctl create iamserviceaccount \
  --cluster fiap-store \
  --namespace store-dev \
  --name store-relatorios \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --role-name FiapStoreRelatoriosRole \
  --approve
```

A política de confiança gerada é o coração do mecanismo — repare na condição sobre o `sub`, que amarra a role a **uma** ServiceAccount de **um** namespace:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/oidc.eks.us-east-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.us-east-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:aud": "sts.amazonaws.com",
          "oidc.eks.us-east-1.amazonaws.com/id/EXAMPLED539D4633E53DE1B71EXAMPLE:sub": "system:serviceaccount:store-dev:store-relatorios"
        }
      }
    }
  ]
}
```

> **Atenção:** usar `StringLike` com `*` no `sub` (por exemplo `system:serviceaccount:*:*`) transforma a role em uma permissão para **qualquer Pod do cluster**. É um dos achados mais comuns em auditorias de EKS. Sempre `StringEquals`, sempre com namespace e nome explícitos.

A ServiceAccount resultante e o Pod:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: store-relatorios
  namespace: store-dev
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::123456789012:role/FiapStoreRelatoriosRole
---
apiVersion: batch/v1
kind: Job
metadata:
  name: exporta-relatorio-s3
  namespace: store-dev
spec:
  ttlSecondsAfterFinished: 600
  template:
    spec:
      serviceAccountName: store-relatorios
      restartPolicy: Never
      containers:
        - name: aws
          image: amazon/aws-cli:2.17.0
          command: ["/bin/sh","-c","aws sts get-caller-identity && aws s3 ls s3://fiapstore-relatorios/"]
```

```bash
kubectl logs job/exporta-relatorio-s3 -n store-dev
```

```
{
    "UserId": "AROAEXAMPLEID:botocore-session-1786049000",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/FiapStoreRelatoriosRole/botocore-session-1786049000"
}
2026-08-05 02:00:14      18432 fechamento-2026-08-04.csv
```

O `Arn` prova que o Pod assumiu a role — não há nenhuma access key no container.

**EKS Pod Identity** é a evolução (2023+): dispensa o provedor OIDC e as anotações, usando um agente (DaemonSet) e uma associação explícita.

```bash
aws eks create-pod-identity-association \
  --cluster-name fiap-store \
  --namespace store-dev \
  --service-account store-relatorios \
  --role-arn arn:aws:iam::123456789012:role/FiapStoreRelatoriosRole
```

A política de confiança fica muito mais simples (principal `pods.eks.amazonaws.com`), e a mesma role pode ser reutilizada em vários clusters. IRSA continua sendo necessário para casos cross-account e fora do EKS.

#### 9.9.3 Menor privilégio e antipadrões

| Antipadrão | Por que é ruim | Alternativa |
|---|---|---|
| `cluster-admin` para o pipeline de CI | Um token de CI vazado compromete o cluster inteiro | Role por namespace com apenas os verbos de deploy; `deployments`, `services`, `configmaps` |
| `verbs: ["*"]` / `resources: ["*"]` | Concede automaticamente CRDs futuros | Enumerar explicitamente; revisar quando instalar operadores |
| `ClusterRoleBinding` quando `RoleBinding` bastaria | Vaza permissões para todos os namespaces | `RoleBinding` referenciando o ClusterRole |
| Compartilhar o kubeconfig de admin no time | Sem auditoria individual; sem revogação | Uma identidade por pessoa (OIDC/Access Entry) |
| ServiceAccount `default` com token montado | Toda aplicação vira potencial cliente da API | `automountServiceAccountToken: false` |
| `get secrets` amplo para desenvolvedores | Acesso a todas as senhas do namespace | `resourceNames` com os Secrets específicos |
| `StringLike` com `*` no `sub` do IRSA | Qualquer Pod assume a role | `StringEquals` com namespace e SA explícitos |
| `edit` para perfil de auditoria | `edit` lê Secrets e altera workloads | `view` (não lê Secrets) |

---

### 9.10 Admission control (visão geral)

Passadas autenticação e autorização, a requisição ainda enfrenta os **admission controllers**, que rodam em duas ondas: primeiro os **mutating** (podem alterar o objeto — é assim que sidecars são injetados e que `defaultMode` e limites default aparecem), depois os **validating** (só aceitam ou rejeitam).

RBAC responde "*quem* pode criar um Pod". Admission control responde "*que tipo* de Pod pode ser criado". São perguntas ortogonais, e é por isso que RBAC sozinho não impede um desenvolvedor legítimo de criar um Pod `privileged: true` com `hostPath: /`.

- **Pod Security Admission (PSA)** — nativo desde a 1.25, substituto do removido PodSecurityPolicy. Aplica os perfis `privileged`, `baseline` e `restricted` por namespace, via labels:

```bash
kubectl label namespace store-prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/warn=restricted \
  pod-security.kubernetes.io/audit=restricted
```

- **OPA Gatekeeper** — políticas em Rego, com CRDs `ConstraintTemplate` e `Constraint`; maduro e expressivo, com curva de aprendizado.
- **Kyverno** — políticas em YAML, sem nova linguagem; muito popular por isso. Faz validação, mutação e geração de recursos.
- **ValidatingAdmissionPolicy** — nativo (estável na 1.30), usa CEL e roda dentro do API server, sem webhook externo e sem o risco de indisponibilidade do webhook derrubar o cluster.

> **Nota histórica:** `PodSecurityPolicy` (`policy/v1beta1`) foi depreciado na 1.21 e **removido na 1.25**. Se você encontrar um tutorial com PSP, ele tem pelo menos quatro anos. O substituto oficial é o PSA acima.

---

### 9.11 Laboratório guiado (feito em sala)

**Passo 0 — preparar**

```bash
kubectl create namespace store-dev 2>/dev/null
kubectl create namespace store-prod 2>/dev/null
kubectl create deployment store-api --image=docker.io/seuusuario/store-api:1.0 -n store-dev
kubectl create secret generic store-db-secret --from-literal=POSTGRES_PASSWORD='FiapStore#2026' -n store-dev
mkdir -p ~/lab09 && cd ~/lab09
```

**Passo 1 — gerar chave e CSR da Maria**

```bash
openssl genrsa -out maria-dev.key 2048
openssl req -new -key maria-dev.key -out maria-dev.csr \
  -subj "/CN=maria.dev/O=fiap:desenvolvedores"
openssl req -in maria-dev.csr -noout -subject
```

```
subject=CN = maria.dev, O = fiap:desenvolvedores
```

**Passo 2 — submeter e aprovar o CSR** (bloco da seção 9.8)

```bash
kubectl certificate approve maria-dev-csr
kubectl get csr maria-dev-csr
kubectl get csr maria-dev-csr -o jsonpath='{.status.certificate}' | base64 -d > maria-dev.crt
openssl x509 -in maria-dev.crt -noout -subject -dates
```

**Passo 3 — montar o kubeconfig** (bloco da etapa 5 da seção 9.8) e testar antes de qualquer permissão:

```bash
kubectl --kubeconfig=kubeconfig-maria.yaml get pods -n store-dev
```

```
Error from server (Forbidden): pods is forbidden: User "maria.dev" cannot list
resource "pods" in API group "" in the namespace "store-dev"
```

Autenticação OK (não é 401), autorização negada (403). Exatamente o esperado.

**Passo 4 — criar a Role limitada ao namespace store-dev**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: dev-store
  namespace: store-dev
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "persistentvolumeclaims", "events"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["pods/log"]
    verbs: ["get", "list"]
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployments/scale"]
    verbs: ["update", "patch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: maria-dev-store
  namespace: store-dev
subjects:
  - kind: User
    name: maria.dev
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: dev-store
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f role-dev.yaml -f rolebinding-maria.yaml
```

**Passo 5 — testar como Maria**

```bash
kubectl --kubeconfig=kubeconfig-maria.yaml get pods -n store-dev
kubectl --kubeconfig=kubeconfig-maria.yaml scale deployment store-api --replicas=3 -n store-dev
kubectl --kubeconfig=kubeconfig-maria.yaml get secret store-db-secret -n store-dev
kubectl --kubeconfig=kubeconfig-maria.yaml get pods -n store-prod
kubectl --kubeconfig=kubeconfig-maria.yaml delete pod --all -n store-dev
```

```
NAME                         READY   STATUS    RESTARTS   AGE
store-api-6f9c7b4d55-2wq8t   1/1     Running   0          6m

deployment.apps/store-api scaled

Error from server (Forbidden): secrets "store-db-secret" is forbidden: User "maria.dev" cannot get resource "secrets" in API group "" in the namespace "store-dev"

Error from server (Forbidden): pods is forbidden: User "maria.dev" cannot list resource "pods" in API group "" in the namespace "store-prod"

Error from server (Forbidden): pods is forbidden: User "maria.dev" cannot delete resource "pods" in API group "" in the namespace "store-dev"
```

Quatro comprovações em cinco comandos: ela lê, ela escala, ela **não** lê Secrets, ela **não** alcança produção e **não** apaga Pods.

**Passo 6 — validar com `can-i`**

```bash
kubectl --kubeconfig=kubeconfig-maria.yaml auth can-i --list -n store-dev
kubectl auth can-i delete pods -n store-dev --as=maria.dev
kubectl auth can-i patch deployments -n store-dev --as=maria.dev
kubectl auth can-i get secrets -n store-dev --as=maria.dev
kubectl auth can-i list pods -n store-prod --as=maria.dev
```

```
no
yes
no
no
```

Repare que o resultado do `--as` do administrador bate exatamente com o comportamento real da Maria. Essa equivalência é o que torna a impersonation confiável como ferramenta de validação.

**Passo 7 — conceder acesso de leitura em store-prod usando role padrão**

```bash
kubectl create rolebinding maria-view-prod --clusterrole=view --user=maria.dev -n store-prod
kubectl auth can-i list pods -n store-prod --as=maria.dev
kubectl auth can-i get secrets -n store-prod --as=maria.dev
```

```
yes
no
```

Um único comando, um ClusterRole existente, escopo restrito a um namespace — e `view` não lê Secrets, de graça.

**Passo 8 — ServiceAccount com IRSA lendo um bucket S3** (requer cluster EKS)

```bash
eksctl utils associate-iam-oidc-provider --cluster fiap-store --region us-east-1 --approve

aws s3 mb s3://fiapstore-relatorios-$RANDOM --region us-east-1

eksctl create iamserviceaccount \
  --cluster fiap-store --namespace store-dev --name store-relatorios \
  --attach-policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess \
  --role-name FiapStoreRelatoriosRole --approve

kubectl get sa store-relatorios -n store-dev -o yaml | grep -A2 annotations
kubectl apply -f job-exporta-s3.yaml
kubectl logs job/exporta-relatorio-s3 -n store-dev
```

Comprove no log que o `Arn` retornado por `sts get-caller-identity` é `assumed-role/FiapStoreRelatoriosRole/...`. Em seguida, prove o isolamento: crie um Pod **sem** a ServiceAccount anotada e observe o `Unable to locate credentials`.

**Passo 9 — testar a proteção contra escalation**

```bash
cat <<EOF | kubectl --kubeconfig=kubeconfig-maria.yaml apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: super-maria
  namespace: store-dev
rules:
  - apiGroups: ["*"]
    resources: ["*"]
    verbs: ["*"]
EOF
```

```
Error from server (Forbidden): roles.rbac.authorization.k8s.io is forbidden: User "maria.dev" cannot create resource "roles" in API group "rbac.authorization.k8s.io" in the namespace "store-dev"
```

**Passo 10 — revogar e limpar**

```bash
kubectl delete rolebinding maria-dev-store -n store-dev
kubectl delete rolebinding maria-view-prod -n store-prod
kubectl --kubeconfig=kubeconfig-maria.yaml get pods -n store-dev   # Forbidden de novo
kubectl delete role dev-store -n store-dev
kubectl delete csr maria-dev-csr

# recursos AWS (evite cobranca!)
eksctl delete iamserviceaccount --cluster fiap-store --namespace store-dev --name store-relatorios
aws s3 rb s3://fiapstore-relatorios-XXXX --force
```

> **Atenção (custo):** o cluster EKS custa cerca de US$ 0,10/hora só de control plane, mais as instâncias EC2 do node group. Se este for o último laboratório da semana, destrua o cluster: `eksctl delete cluster --name fiap-store --region us-east-1`.

---

### 9.12 Ponte com Serverless

O modelo de identidade que estudamos neste capítulo tem um paralelo quase perfeito no mundo serverless — e a comparação esclarece os dois lados.

**Execution role do Lambda x IRSA.** Toda função Lambda tem uma **execution role**: uma role IAM que a função assume no momento da invocação, recebendo credenciais temporárias injetadas pelo runtime como variáveis de ambiente. O desenvolvedor nunca escreve uma access key. Quando você configura IRSA em um Pod, está reproduzindo exatamente esse modelo dentro do Kubernetes: a ServiceAccount cumpre o papel de "identidade do workload", a anotação `eks.amazonaws.com/role-arn` cumpre o papel de "execution role", e o STS `AssumeRoleWithWebIdentity` cumpre o papel do runtime injetando credenciais. A diferença é que no Lambda a plataforma faz tudo sozinha, e no Kubernetes você precisa associar o provedor OIDC, criar a role, escrever a política de confiança com a condição correta sobre o `sub` e anotar a ServiceAccount. Mais passos, mais chances de errar — e é por isso que o `StringLike` com wildcard aparece tanto em auditorias.

**Políticas de identidade em vez de credenciais no código.** Os dois modelos partem do mesmo princípio, que é a lição central deste capítulo: **credencial estática é dívida**. Uma access key em variável de ambiente, um token de ServiceAccount sem expiração, um certificado de 10 anos — todos são credenciais que valem até alguém percebê-las vazadas. Credencial temporária, emitida sob demanda para uma identidade verificada e expirando em uma hora, transforma um vazamento de catástrofe permanente em incidente de janela curta. É a mesma razão pela qual o Kubernetes abandonou os tokens permanentes de ServiceAccount na 1.24.

**Blast radius.** Aqui a comparação é reveladora. Uma função Lambda comprometida dá ao atacante exatamente as permissões da execution role daquela função, dentro de um micro-VM que será destruído em minutos, sem acesso à rede de outras funções e sem vizinhos. Um Pod comprometido em um cluster mal configurado é outra história: se a ServiceAccount tem token montado com permissões amplas, o atacante fala com a API do Kubernetes; se há `hostPath` liberado, alcança o disco do nó; se o `automountServiceAccountToken` estava ligado por padrão em todo namespace, alcança outras cargas; e se alguém deu `cluster-admin` ao pipeline de CI, é fim de jogo. O raio de alcance no Kubernetes é maior porque a superfície é maior — nós compartilhados, um plano de controle comum, uma rede plana por padrão.

A conclusão prática para a FIAP Store não é "serverless é mais seguro", e sim: **o Kubernetes exige que você construa manualmente o isolamento que o FaaS entrega por padrão**. Namespaces separados, RBAC de menor privilégio, `automountServiceAccountToken: false`, NetworkPolicies, Pod Security Admission `restricted` e IRSA com escopo estrito são o trabalho que a plataforma serverless já fez por você. Quando esse trabalho é feito com disciplina, os dois modelos convergem em segurança — e a escolha volta a ser sobre custo, portabilidade e controle.

---

### 9.13 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| `error: You must be logged in to the server (Unauthorized)` — 401 | Certificado expirado/inválido, CA errada, token vencido | `openssl x509 -in cert.crt -noout -dates`; conferir `certificate-authority-data` | Reemitir o certificado via CSR; regenerar o kubeconfig com a CA correta |
| `Forbidden: User "X" cannot list pods` — 403 | Falta Role/RoleBinding no namespace | Ler o próprio erro (sujeito, verbo, recurso, namespace); `kubectl auth can-i --list --as=X -n <ns>` | Criar a regra mínima correspondente |
| Role criada e ainda `Forbidden` | `apiGroups` errado (`"apps"` para Deployment, `""` para Pod) | `kubectl api-resources \| grep <recurso>` | Corrigir o `apiGroups` e o nome plural do recurso |
| `kubectl logs` negado, mas `get pods` funciona | Falta o subresource `pods/log` | `kubectl auth can-i get pods/log -n <ns> --as=X` | Adicionar `resources: ["pods/log"]` |
| `kubectl exec` negado | Falta `pods/exec` com verbo `create` | `kubectl auth can-i create pods/exec -n <ns> --as=X` | Adicionar a regra — e considerar se o perfil deve mesmo tê-la |
| `kubectl apply` falha para quem só tem `create`/`update` | `apply` faz `get` + `patch` | `kubectl auth can-i patch <recurso> -n <ns> --as=X` | Adicionar `get` e `patch` aos verbos |
| RoleBinding aplicado, sem efeito | Username diferente do `CN` do certificado (case-sensitive), ou binding em outro namespace | `openssl x509 -noout -subject`; `kubectl get rolebinding -A -o wide` | Alinhar exatamente o nome; criar o binding no namespace certo |
| `field is immutable` ao editar um binding | `roleRef` não pode ser alterado | Erro do `kubectl apply` | `kubectl delete` + `kubectl apply` |
| `attempting to grant RBAC permissions not currently held` | Escalation prevention | `kubectl auth can-i --list` do criador | Criar a Role com uma identidade que já tenha as permissões, ou conceder `escalate` conscientemente |
| ServiceAccount não consegue falar com a API | `automountServiceAccountToken: false`, ou nenhum binding para a SA | `kubectl describe pod` (procurar o volume do token); `kubectl auth can-i --as=system:serviceaccount:<ns>:<sa>` | Ligar o automount ou usar volume `projected`; criar o RoleBinding |
| Pod com IRSA: `Unable to locate credentials` | SA sem anotação, Pod sem `serviceAccountName`, ou provedor OIDC não associado | `kubectl get sa <sa> -o yaml`; `aws eks describe-cluster --query cluster.identity.oidc` | Anotar a SA, referenciá-la no Pod, associar o OIDC e **recriar** o Pod (a anotação não é aplicada retroativamente) |
| IRSA: `AccessDenied` no `AssumeRoleWithWebIdentity` | Condição `sub` da trust policy não bate com `system:serviceaccount:<ns>:<sa>` | Ler a trust policy da role no console IAM; decodificar o token | Corrigir o `sub` (e usar `StringEquals`, nunca `StringLike` com `*`) |
| Perdi acesso ao EKS após editar `aws-auth` | YAML inválido ou remoção do próprio mapeamento | `kubectl get cm aws-auth -n kube-system -o yaml` (se ainda conseguir) | Usar a identidade criadora do cluster (sempre admin implícita) ou migrar para Access Entries |
| `kubectl auth can-i` diz `yes`, comando falha | Bloqueio no admission control, não na autorização | Ler a mensagem completa (cita o webhook/policy) | Ajustar o manifesto à política (PSA/Kyverno/Gatekeeper) |

---

### 9.14 Resumo do capítulo

O controle de acesso no Kubernetes é uma sequência de três decisões independentes: autenticação responde "quem é você", autorização responde "você pode", e admission control responde "isso é aceitável aqui". O Kubernetes deliberadamente não gerencia usuários humanos — ele extrai username e groups de credenciais emitidas por certificados X.509, provedores OIDC ou webhooks de nuvem, e a partir daí tudo é RBAC. Role e ClusterRole descrevem permissões; RoleBinding e ClusterRoleBinding as concedem; e como RBAC é puramente aditivo, o menor privilégio se constrói escrevendo pouco, não negando muito. Nas nuvens, esse modelo se conecta à identidade da plataforma: no EKS, IAM autentica via Access Entries e o RBAC autoriza, enquanto IRSA/Pod Identity resolvem o problema inverso — dar a um Pod permissões na AWS sem nenhuma credencial estática.

Pontos-chave:

1. **401 é autenticação, 403 é autorização.** A mensagem de 403 entrega sujeito, verbo, recurso, apiGroup e namespace — use-a para escrever a regra exata.
2. **Não existe objeto `User`.** Um RoleBinding pode nomear alguém que ainda não tem certificado nenhum; a identidade vem sempre de fora.
3. **RoleBinding + ClusterRole = permissões de cluster aplicadas a um namespace.** É o padrão mais útil e o mais mal compreendido.
4. **RBAC não tem `deny`.** Para proibir, remova a concessão ou use admission control (PSA, Kyverno, Gatekeeper, ValidatingAdmissionPolicy).
5. **`view` não lê Secrets; `edit` lê.** Escolher entre eles é uma decisão de segurança, não de conveniência.
6. **Tokens de ServiceAccount hoje são projetados, expiram e são vinculados ao Pod.** Desligue o automount onde a aplicação não fala com a API.
7. **Certificado X.509 não pode ser revogado** — para cortar acesso, remova os bindings. Em produção, use OIDC/Access Entries.
8. **IRSA é a execution role do Lambda trazida para o Pod**; a condição sobre o `sub` deve ser `StringEquals`, com namespace e ServiceAccount explícitos.

---

### 9.15 Exercícios (para casa)

#### Exercício 9.1 — ServiceAccount somente-leitura validada com can-i
*Tempo estimado: 25 min · Nível: fixação guiada*

- **Objetivo:** conceder acesso de leitura a uma identidade de workload e validar objetivamente o resultado.
- **Enunciado:** no namespace `store-dev`, crie a ServiceAccount `monitor-sa`, uma Role `leitura-store` que permita apenas `get`, `list` e `watch` sobre `pods`, `services`, `endpoints`, `configmaps` e o subresource `pods/log`, e o RoleBinding correspondente. Em seguida: (a) valide com `kubectl auth can-i` usando impersonation (`--as=system:serviceaccount:store-dev:monitor-sa`) pelo menos **seis** combinações verbo/recurso, incluindo duas que devem retornar `no` (`get secrets` e `create pods`); (b) gere um token com `kubectl create token monitor-sa -n store-dev --duration=1h`, decodifique o payload JWT (`cut -d. -f2 | base64 -d`) e mostre os campos `sub`, `aud` e `exp`; (c) suba um Pod que use essa ServiceAccount e, de dentro dele, chame a API do Kubernetes com `curl` usando o token montado, listando os Pods do namespace e recebendo `403` ao tentar listar Secrets. Documente as evidências em `EVIDENCIAS.md`.
- **Entregável:** `monitor-rbac.yaml` (SA + Role + RoleBinding), `EVIDENCIAS.md` com as seis saídas de `can-i`, o payload JWT decodificado e as duas respostas HTTP do `curl` (200 e 403) com o corpo da mensagem de erro.
- **Critério de aceite:** `kubectl auth can-i --list -n store-dev --as=system:serviceaccount:store-dev:monitor-sa` mostra apenas os recursos concedidos; `get secrets` e `create pods` retornam `no`; o `sub` do JWT é exatamente `system:serviceaccount:store-dev:monitor-sa`; e o `curl` a `/api/v1/namespaces/store-dev/secrets` retorna HTTP 403 com mensagem de `forbidden`.
- **Dica:**
  <details><summary>Dica</summary>
  Dentro do Pod: <code>TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)</code> e <code>curl -sS -k -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/store-dev/pods</code>. Use <code>-w '%{http_code}'</code> para capturar o status.
  </details>

#### Exercício 9.2 — Usuário por certificado com kubeconfig funcional
*Tempo estimado: 45 min · Nível: aplicação*

- **Objetivo:** executar o fluxo completo de emissão de identidade humana e entregar um kubeconfig autocontido e testado.
- **Enunciado:** crie o usuário `joao.qa`, pertencente ao grupo `fiap:qa`, seguindo todo o fluxo: chave privada com `openssl genrsa`, CSR com `-subj "/CN=joao.qa/O=fiap:qa"`, objeto `CertificateSigningRequest` com `signerName: kubernetes.io/kube-apiserver-client` e `expirationSeconds: 604800` (7 dias), aprovação, extração do certificado e montagem do `kubeconfig-joao.yaml` com `--embed-certs=true` (o arquivo deve funcionar sozinho, sem depender de nenhum outro arquivo no disco). Depois: (a) prove que **antes** de qualquer binding o acesso retorna `403` e não `401`, e explique por escrito a diferença; (b) crie um RoleBinding no namespace `store-dev` associando o **grupo** `fiap:qa` (não o usuário) ao ClusterRole `view`, e comprove que o João passa a listar Pods — demonstrando que a permissão veio do `O=` do certificado; (c) conceda adicionalmente, via Role própria, permissão de `create` sobre `pods/exec` **apenas** no namespace `store-dev`, e comprove que `kubectl exec` funciona em `store-dev` e falha em `store-prod`; (d) revogue todo o acesso e comprove que o certificado continua autenticando (403, não 401). Registre tudo em `RELATORIO.md`.
- **Entregável:** `joao-qa.key`, `joao-qa.csr`, `joao-qa.crt`, `csr-joao.yaml`, `kubeconfig-joao.yaml`, `rbac-qa.yaml`, `RELATORIO.md` com as saídas de cada etapa e a saída de `openssl x509 -in joao-qa.crt -noout -subject -dates`.
- **Critério de aceite:** o `kubeconfig-joao.yaml`, copiado para um diretório vazio, executa `kubectl get pods -n store-dev` com sucesso; o subject do certificado é `CN = joao.qa, O = fiap:qa`; o `exec` funciona em `store-dev` e retorna `Forbidden` em `store-prod`; e após a revogação o erro é `Forbidden` (403), não `Unauthorized` (401).
- **Dica:**
  <details><summary>Dica</summary>
  Se o cluster for EKS, este exercício não funciona (o EKS não assina CSRs de cliente) — use minikube ou kind. Para testar o kubeconfig isolado: <code>mkdir /tmp/teste && cp kubeconfig-joao.yaml /tmp/teste/ && cd /tmp/teste && KUBECONFIG=./kubeconfig-joao.yaml kubectl get pods -n store-dev</code>.
  </details>

#### Exercício 9.3 — Depuração: quatro erros "Forbidden" reais
*Tempo estimado: 40 min · Nível: análise/depuração*

- **Objetivo:** interpretar mensagens de autorização e corrigi-las com o **mínimo** de permissão adicional.
- **Enunciado:** para cada cenário, reproduza o erro, transcreva a mensagem **exata**, identifique os cinco elementos que ela informa (sujeito, verbo, recurso, apiGroup, namespace), e aplique a **menor** alteração de RBAC que o resolve. Prove que a correção foi mínima mostrando um `can-i` que continua retornando `no` para algo adjacente.
  - **Cenário A:** uma Role concede `pods` com `["get","list","watch"]`, mas o usuário roda `kubectl logs`. Explique por que falha e corrija.
  - **Cenário B:** uma Role concede `resources: ["deployments"]` com `apiGroups: [""]` (core). O usuário não consegue listar Deployments. Identifique o erro e corrija sem ampliar mais nada.
  - **Cenário C:** uma Role concede `["create","update"]` sobre `deployments` no apiGroup `apps`. O usuário roda `kubectl apply -f deploy.yaml` e recebe `Forbidden`. Descubra quais verbos o `apply` realmente exige e corrija.
  - **Cenário D:** um `RoleBinding` no namespace `store-dev` referencia a ClusterRole `edit`, mas o usuário tenta operar em `store-prod`. Além de corrigir, explique por que trocar por `ClusterRoleBinding` seria a solução **errada** e qual é a correta.
  Para cada cenário use `kubectl auth can-i --as=<usuario>` como ferramenta principal de diagnóstico.
- **Entregável:** `DEPURACAO.md` contendo, por cenário: a mensagem exata de erro, a tabela dos cinco elementos extraídos, o diff do YAML da Role (antes/depois), a saída de `can-i` provando a correção e a saída de `can-i` provando que a permissão adjacente continua negada.
- **Critério de aceite:** os quatro cenários apresentam mensagens de erro reais capturadas do cluster (não inventadas); a correção do Cenário A adiciona **apenas** `pods/log` (não `pods/exec`); a do Cenário C adiciona **apenas** `get` e `patch`; e no Cenário D a solução é um segundo RoleBinding em `store-prod`, com justificativa escrita contra o `ClusterRoleBinding`.
- **Dica:**
  <details><summary>Dica</summary>
  Rode <code>kubectl apply -v=8</code> para ver as requisições HTTP reais que o comando faz — é o jeito mais direto de descobrir os verbos exigidos no Cenário C.
  </details>

#### Exercício 9.4 — Desafio: modelo RBAC completo para quatro perfis em dois namespaces
*Tempo estimado: 60 min · Nível: desafio integrador*

- **Objetivo:** projetar, implementar, documentar e comprovar um modelo de autorização realista de menor privilégio.
- **Enunciado:** desenhe e implemente o modelo de acesso da FIAP Store para os namespaces `store-dev` e `store-prod`, com quatro perfis, representados por **grupos** (não por usuários individuais):
  - **`fiap:dev`** — controle total sobre workloads em `store-dev` (Deployments, Pods, Services, ConfigMaps, Jobs, HPA), incluindo `pods/log` e `pods/exec`; em `store-prod`, **somente leitura** e **sem** acesso a Secrets nem a `pods/exec`.
  - **`fiap:qa`** — leitura em ambos os namespaces, mais permissão de criar e apagar **Jobs** em `store-dev` (para rodar suítes de teste) e `create` sobre `pods/exec` apenas em `store-dev`.
  - **`fiap:sre`** — controle total em ambos os namespaces, incluindo Secrets, PVCs, StatefulSets e DaemonSets, mais leitura cluster-wide de `nodes`, `persistentvolumes`, `storageclasses` e `namespaces` — **sem** ser `cluster-admin`.
  - **`fiap:auditor`** — leitura de **tudo** em ambos os namespaces, incluindo objetos RBAC (`roles`, `rolebindings`), mas **jamais** o conteúdo de Secrets e **sem nenhum** verbo de escrita em lugar nenhum.
  Requisitos de implementação: use ClusterRoles reaproveitadas por RoleBindings sempre que possível; nenhum wildcard em `verbs` ou `resources`; nenhum `ClusterRoleBinding` exceto onde o acesso for genuinamente cluster-wide (perfil SRE); documente a decisão em comentários no YAML. Crie um usuário por certificado para **cada** perfil (ou, alternativamente, quatro ServiceAccounts) para produzir evidências reais. Por fim, produza uma **matriz de permissões** em `MATRIZ.md`: linhas = (recurso, verbo) com pelo menos 14 combinações relevantes; colunas = os 4 perfis × 2 namespaces (8 colunas); células = `yes`/`no` **obtidos da saída real** de `kubectl auth can-i ... --as-group=<grupo> --as=<usuario>`, não do que você espera. Inclua um script `testa-rbac.sh` que gere a matriz automaticamente.
- **Entregável:** `rbac-modelo.yaml` (todas as ClusterRoles, Roles e bindings, com comentários justificando cada decisão), `testa-rbac.sh`, `MATRIZ.md` com a matriz preenchida a partir da saída do script, e um parágrafo final justificando por que o perfil `auditor` **não** usa o ClusterRole `view` como está (ou por que usa).
- **Critério de aceite:** a matriz tem no mínimo 14 linhas × 8 colunas totalmente preenchidas com saídas reais; `fiap:auditor` retorna `no` para todo verbo de escrita e `no` para `get secrets` nos dois namespaces; `fiap:dev` retorna `yes` para `create pods/exec` em `store-dev` e `no` em `store-prod`; `fiap:sre` retorna `yes` para `get nodes` mas `no` para `create clusterrolebindings`; e nenhum arquivo YAML contém `"*"` em `verbs` ou `resources`.
- **Dica:**
  <details><summary>Dica</summary>
  Esqueleto do script: <code>for g in dev qa sre auditor; do for ns in store-dev store-prod; do for c in "get pods" "create jobs" "get secrets" "create pods/exec"; do echo -n "| $(kubectl auth can-i $c -n $ns --as=teste.$g --as-group=fiap:$g) "; done; done; done</code>. Para o auditor, pesquise por que ler <code>rolebindings</code> é considerado privilégio sensível — e considere <code>resourceNames</code> se precisar de granularidade.
  </details>

---

### 9.16 Leitura recomendada

1. [DOC] **Kubernetes — Authentication, Authorization Overview e Using RBAC Authorization.** https://kubernetes.io/docs/reference/access-authn-authz/rbac/ — referência completa de Role/ClusterRole/bindings, agregação e roles padrão.
2. [DOC] **Kubernetes — Certificate Signing Requests e Certificates and Certificate Signing Requests.** https://kubernetes.io/docs/reference/access-authn-authz/certificate-signing-requests/ — o fluxo do laboratório, com todos os `signerName` disponíveis.
3. [DOC] **AWS — EKS: Grant IAM users access to Kubernetes (Access Entries) e IAM Roles for Service Accounts.** https://docs.aws.amazon.com/eks/latest/userguide/access-entries.html e https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html
4. [ARTIGO] **OWASP Kubernetes Top Ten** — https://owasp.org/www-project-kubernetes-top-ten/ — itens K01 (Insecure Workload Configurations) e K03 (Overly Permissive RBAC) dialogam diretamente com este capítulo.
5. [LIVRO] **RICE, L.** *Container Security*. O'Reilly, 2020 — capítulos sobre identidade de workload, tokens e superfície de ataque de containers.
6. [LIVRO] **SAYFAN, G.** *Mastering Kubernetes*, 4. ed. Packt, 2023 — capítulo de segurança, com discussão prática de RBAC, admission control e multi-tenancy.
7. [DOC] **Kyverno e OPA Gatekeeper.** https://kyverno.io/policies/ e https://open-policy-agent.github.io/gatekeeper/ — bibliotecas de políticas prontas para complementar o RBAC com admission control.

---

## Capítulo 10 — Boas Práticas de Operação e Segurança

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 9 (arquitetura do cluster, Pods/Services, Deployments/HPA, networking e namespaces, EKS, volumes e StatefulSet, Jobs e Secrets, RBAC); Docker do 1º semestre (Dockerfile, multi-stage, usuário não-root) · **Artefatos:** `store-api` endurecida, `Dockerfile` distroless, `namespace store-prod` com Pod Security Admission `restricted`, policy Kyverno, chart Helm/overlays Kustomize da FIAP Store

Chegamos ao ponto do semestre em que a pergunta muda. Até aqui perguntávamos "como faço isto funcionar no Kubernetes?". A partir de agora a pergunta é "como faço isto funcionar às três da manhã de um domingo, sem mim, sem susto e sem virar manchete?". É uma pergunta diferente, e responder a ela é o que separa quem sabe usar Kubernetes de quem sabe operar Kubernetes.

Este capítulo é denso de propósito. Ele reúne o que a indústria aprendeu — muitas vezes da pior forma possível — sobre segurança e operação de clusters. Vamos organizar esse conhecimento em duas grandes metades: **segurança** (o modelo dos 4C, imagem, `securityContext`, Pod Security Admission, control plane, supply chain, OWASP) e **operação** (health checks, graceful shutdown, disrupção, recursos, observabilidade, configuração, upgrade, backup, FinOps). No fim, montamos um checklist de pré-voo e endurecemos a `store-api` de verdade.

---

### 10.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** as quatro camadas do modelo de segurança nativo em nuvem (Cloud, Cluster, Container, Code) e localizar em qual delas um determinado risco se manifesta.
2. **Construir** imagens de container mínimas, reproduzíveis e não-root, com multi-stage build, e **analisar** seu resultado com scanners de vulnerabilidade (Trivy/Grype) e SBOM.
3. **Aplicar** um `securityContext` restrito em Pod e container e **configurar** Pod Security Admission por namespace, comprovando a rejeição de workloads privilegiados.
4. **Escrever** uma política de admissão declarativa (Kyverno) que barre um antipadrão real de produção, e relacioná-la ao OWASP Kubernetes Top 10.
5. **Diagnosticar e corrigir** perda de requisições durante um rolling update, combinando readiness probe, `preStop` hook, `terminationGracePeriodSeconds` e tratamento de `SIGTERM` na aplicação FastAPI.
6. **Avaliar** o dimensionamento de `requests`/`limits` de um workload, explicando o efeito do CPU throttling, e **construir** um pacote de deploy versionado com Helm ou Kustomize (overlays dev/prod).

---

### 10.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | Abertura: por que "funciona no meu cluster" não é segurança. Os 4C. |
| 2 | 20 min | Segurança da imagem: base mínima, multi-stage, não-root, scanning, assinatura, SBOM. |
| 3 | 20 min | `securityContext` completo + Pod Security Admission (privileged/baseline/restricted). |
| 4 | 10 min | **Intervalo** |
| 5 | 15 min | Control plane, etcd, encryption at rest, auditoria; policy engines (Kyverno/OPA) e OWASP K8s Top 10. |
| 6 | 20 min | Operação: probes, graceful shutdown, PDB, requests/limits e CPU throttling, LimitRange/ResourceQuota. |
| 7 | 10 min | Observabilidade (4 sinais dourados, SLI/SLO), Helm x Kustomize, upgrade, backup/DR, FinOps. |
| 8 | 5 min | Ponte com Serverless: superfície de ataque do FaaS e responsabilidade compartilhada. |
| 9 | 10 min | **Laboratório guiado:** endurecendo a `store-api` (PSA restricted + Trivy + preStop + Kyverno). |
| — | 120 min | **Total** |

---

### 10.3 Segurança em camadas: o modelo dos 4C

A CNCF popularizou um modelo mental simples e muito útil: a segurança de uma aplicação nativa em nuvem é composta por quatro camadas concêntricas, e **cada camada só é tão forte quanto a que está fora dela**. Se a nuvem está mal configurada, não importa o quão bonito é o seu `securityContext`.

```
+--------------------------------------------------------------+
|  CLOUD  (AWS: VPC, IAM, Security Groups, KMS, CloudTrail)     |
|  +--------------------------------------------------------+  |
|  |  CLUSTER  (API server, etcd, RBAC, PSA, NetworkPolicy)  |  |
|  |  +--------------------------------------------------+  |  |
|  |  |  CONTAINER  (imagem, runtime, securityContext)    |  |  |
|  |  |  +--------------------------------------------+  |  |  |
|  |  |  |  CODE  (auth, TLS, deps, secrets, input)   |  |  |  |
|  |  |  +--------------------------------------------+  |  |  |
|  |  +--------------------------------------------------+  |  |
|  +--------------------------------------------------------+  |
+--------------------------------------------------------------+
```

**Cloud.** É o alicerce. No EKS: os nós ficam em subnets privadas? O endpoint do API server é público sem restrição de CIDR? Os Security Groups permitem `0.0.0.0/0` na porta 22? Existe IRSA (IAM Roles for Service Accounts) ou todos os Pods herdam a role do nó — o que dá a qualquer Pod comprometido as permissões de toda a instância? O CloudTrail está ligado? A criptografia de EBS e de Secrets via KMS está habilitada?

**Cluster.** É a superfície administrativa. Autenticação e autorização (RBAC, capítulo 9), Pod Security Admission, NetworkPolicy (capítulo 5), admission controllers, isolamento por namespace, auditoria do API server, proteção do etcd.

**Container.** É a unidade de execução. Que imagem estamos rodando, de onde ela veio, quem a assinou, que vulnerabilidades ela carrega, com que privilégios o processo roda, se o filesystem é somente-leitura, que capabilities o kernel concede.

**Code.** É onde 100% das aplicações continuam vulneráveis mesmo com tudo acima perfeito: SQL injection, dependência com CVE crítico, senha em log, endpoint sem autenticação, TLS desativado "só no ambiente interno".

> **Nota:** o modelo é concêntrico, não hierárquico de importância. Um SQL injection na `store-api` derruba a loja mesmo que o cluster seja impecável. Um IAM permissivo na AWS entrega o cluster inteiro mesmo que o código seja perfeito.

> **Na prática:** ao investigar um incidente, percorra as camadas de fora para dentro perguntando "o que uma pessoa aqui consegue alcançar?". Esse exercício se chama modelagem de ameaças e cabe em 30 minutos de reunião.

---

### 10.4 Segurança da imagem de container

Tudo que roda no cluster nasce de uma imagem. Uma imagem grande, desatualizada e root é o presente perfeito para um atacante: ela traz shell, gerenciador de pacotes, `curl`, compilador e permissão para instalar o que quiser.

#### 10.4.1 Imagem base mínima

A regra é: **quanto menos existir dentro da imagem, menos pode ser explorado**. Isso não é só higiene, é redução matemática da superfície de ataque e do trabalho de patching.

| Base | Tamanho típico | Shell? | Gerenciador de pacotes? | Observação |
|---|---|---|---|---|
| `python:3.12` (Debian completo) | ~1 GB | sim | sim (apt) | conveniente, centenas de pacotes que você não usa |
| `python:3.12-slim` | ~150 MB | sim | sim | bom meio-termo para começar |
| `python:3.12-alpine` | ~50 MB | sim (ash) | sim (apk) | musl libc pode quebrar wheels compiladas (`psycopg2`, `numpy`) |
| `gcr.io/distroless/python3-debian12` | ~50 MB | **não** | **não** | sem shell: `kubectl exec` não abre bash; ótimo para produção |
| `cgr.dev/chainguard/python` | ~40 MB | não (variante prod) | não | reconstruída diariamente, meta de zero CVE conhecido, SBOM e assinatura nativas |

> **Atenção:** distroless quebra o hábito de "entrar no container para depurar". Isso é intencional. A alternativa moderna é `kubectl debug -it pod/store-api-xxx --image=busybox --target=store-api`, que anexa um **ephemeral container** compartilhando os namespaces do processo — você depura sem ter shell na imagem de produção.

#### 10.4.2 Multi-stage build e usuário não-root

O multi-stage separa **o que é necessário para construir** do **que é necessário para executar**. Compilador, headers, cache do pip e o código-fonte ficam no estágio de build e não chegam à imagem final.

```dockerfile
# Dockerfile — store-api endurecida
# ---------- estágio 1: build ----------
FROM python:3.12-slim AS builder

WORKDIR /app
RUN python -m venv /opt/venv
ENV PATH="/opt/venv/bin:$PATH"

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# ---------- estágio 2: runtime ----------
FROM gcr.io/distroless/python3-debian12:nonroot

WORKDIR /app
COPY --from=builder /opt/venv /opt/venv
COPY --chown=nonroot:nonroot main.py .

ENV PATH="/opt/venv/bin:$PATH" \
    PYTHONUNBUFFERED=1 \
    PYTHONDONTWRITEBYTECODE=1

# a tag :nonroot já define UID 65532
USER 65532:65532
EXPOSE 8000

ENTRYPOINT ["python", "-m", "uvicorn", "main:app", \
            "--host", "0.0.0.0", "--port", "8000"]
```

Pontos que merecem comentário:

- **`USER 65532:65532` numérico, não `USER nonroot`.** O Kubernetes precisa avaliar `runAsNonRoot` antes de iniciar o container; se o `USER` for um nome, o kubelet não consegue resolver o UID a partir do `/etc/passwd` da imagem em todos os runtimes e pode falhar com `container has runAsNonRoot and image has non-numeric user`.
- **`--no-cache-dir`** evita levar o cache do pip para dentro da camada.
- **Tag imutável.** Em produção, prefira o digest: `gcr.io/distroless/python3-debian12@sha256:...`. Tag é ponteiro móvel; digest é conteúdo.
- **`.dockerignore`** deve conter `.git`, `.env`, `tests/`, `*.pem`. Já vi credencial de produção vazar por um `.git` copiado com `COPY . .`.

#### 10.4.3 Scanning de vulnerabilidades

Um scanner compara os pacotes instalados (SO + linguagem) contra bases de CVE (NVD, GitHub Advisory, avisos das distros). Os dois mais usados em ambiente CNCF são **Trivy** (Aqua Security) e **Grype** (Anchore).

```bash
# instalar Trivy (Linux)
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh \
  | sh -s -- -b /usr/local/bin v0.58.0

# escanear a imagem
trivy image --severity HIGH,CRITICAL fabiobara/store-api:1.0
```

Saída típica de uma imagem baseada em `python:3.12` puro:

```
fabiobara/store-api:1.0 (debian 12.8)
Total: 47 (HIGH: 41, CRITICAL: 6)

┌──────────────┬────────────────┬──────────┬───────────────────┬───────────────┐
│   Library    │ Vulnerability  │ Severity │ Installed Version │ Fixed Version │
├──────────────┼────────────────┼──────────┼───────────────────┼───────────────┤
│ libssl3      │ CVE-2024-5535  │ CRITICAL │ 3.0.11-1~deb12u2  │ 3.0.14-1      │
│ zlib1g       │ CVE-2023-45853 │ CRITICAL │ 1:1.2.13.dfsg-1   │ (won't fix)   │
│ perl-base    │ CVE-2023-31484 │ HIGH     │ 5.36.0-7          │ 5.36.0-7+deb  │
└──────────────┴────────────────┴──────────┴───────────────────┴───────────────┘

Python (python-pkg)
Total: 3 (HIGH: 2, CRITICAL: 1)
│ starlette    │ CVE-2024-47874 │ HIGH     │ 0.37.2            │ 0.40.0        │
```

Depois de migrar para distroless + dependências atualizadas:

```
fabiobara/store-api:1.1 (debian 12.8)
Total: 0 (HIGH: 0, CRITICAL: 0)
```

Para usar em pipeline, `--exit-code 1` transforma o scanner em porta de qualidade:

```bash
trivy image --exit-code 1 --severity CRITICAL --ignore-unfixed \
  fabiobara/store-api:1.1
```

> **Na prática:** `--ignore-unfixed` é essencial. Sem isso o pipeline quebra por CVEs marcados "won't fix" pela distro, ninguém consegue mergear nada e em duas semanas a equipe desativa o scanner. Um gate que ninguém aguenta é pior que gate nenhum.

O Trivy também escaneia o cluster inteiro e manifestos:

```bash
trivy k8s --report summary cluster       # workloads do cluster
trivy config ./k8s/                      # más configurações em YAML/Helm
trivy fs --scanners vuln,secret .        # código e segredos vazados
```

#### 10.4.4 SBOM, assinatura e admissão de imagens confiáveis

**SBOM** (Software Bill of Materials) é a "lista de ingredientes" da imagem, em formato padronizado (SPDX ou CycloneDX). Ela responde à pergunta que toda empresa fez em dezembro de 2021: "eu uso log4j em algum lugar?".

```bash
trivy image --format cyclonedx --output store-api-sbom.json \
  fabiobara/store-api:1.1
```

**Assinatura com Cosign** (projeto Sigstore) garante procedência: esta imagem foi publicada por quem eu penso que foi, e não foi trocada no registry.

```bash
cosign generate-key-pair                     # gera cosign.key / cosign.pub
cosign sign --key cosign.key fabiobara/store-api:1.1
cosign verify --key cosign.pub fabiobara/store-api:1.1
cosign attach sbom --sbom store-api-sbom.json fabiobara/store-api:1.1
```

Assinar não adianta se ninguém verifica. A verificação acontece no **admission controller**: uma policy Kyverno ou o `policy-controller` do Sigstore intercepta a criação do Pod e rejeita imagens sem assinatura válida (veremos o YAML na seção 10.8).

---

### 10.5 `securityContext`: privilégios do Pod e do container

O `securityContext` é onde traduzimos "princípio do menor privilégio" para YAML. Ele existe em dois níveis: **no Pod** (`spec.securityContext`, aplica-se a todos os containers e ao volume) e **no container** (`spec.containers[].securityContext`, que sobrescreve o do Pod).

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-prod
  labels:
    app: store-api
    tier: backend
    env: prod
spec:
  replicas: 3
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: prod
    spec:
      # ---------- nível Pod ----------
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        runAsGroup: 65532
        fsGroup: 65532
        fsGroupChangePolicy: OnRootMismatch
        seccompProfile:
          type: RuntimeDefault
      automountServiceAccountToken: false
      serviceAccountName: store-api-sa
      containers:
        - name: store-api
          image: docker.io/fabiobara/store-api:1.1
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8000
          # ---------- nível container ----------
          securityContext:
            allowPrivilegeEscalation: false
            privileged: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              memory: "256Mi"
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

Campo a campo, com o *porquê*:

| Campo | Nível | O que faz | Por que importa |
|---|---|---|---|
| `runAsNonRoot: true` | Pod/container | kubelet recusa iniciar se o UID efetivo for 0 | rede de segurança: pega imagem que "voltou" a ser root |
| `runAsUser` / `runAsGroup` | Pod/container | força UID/GID do processo | previsibilidade de permissões em volumes |
| `fsGroup` | Pod | GID suplementar aplicado a volumes montados | sem ele, um PVC montado como root fica ilegível para o app não-root |
| `fsGroupChangePolicy: OnRootMismatch` | Pod | só faz `chown` recursivo se necessário | evita minutos de startup em PVCs com milhões de arquivos |
| `allowPrivilegeEscalation: false` | container | seta `no_new_privs` no kernel | impede que um binário setuid (ex.: `sudo`, `ping`) eleve privilégio |
| `privileged: false` | container | sem acesso aos devices do host | `privileged: true` é praticamente igual a root no nó — é fuga de container por design |
| `readOnlyRootFilesystem: true` | container | monta `/` somente-leitura | atacante não escreve webshell nem baixa binário; exige `emptyDir` em `/tmp` |
| `capabilities.drop: ["ALL"]` | container | remove todas as capabilities do Linux | derruba `CAP_NET_RAW`, `CAP_SYS_ADMIN`, etc. |
| `seccompProfile.type: RuntimeDefault` | Pod/container | aplica o perfil seccomp do runtime | bloqueia ~44 syscalls perigosas; é gratuito e quase ninguém liga |
| `automountServiceAccountToken: false` | Pod | não monta o token da ServiceAccount | se o app não fala com o API server, o token só serve para o atacante |

> **Atenção:** `readOnlyRootFilesystem: true` costuma quebrar aplicações na primeira tentativa — uvicorn, bibliotecas de cache e o próprio Python querem escrever em `/tmp`. A solução **não** é desligar a flag, é montar `emptyDir` exatamente nos caminhos necessários. Se precisar de `CAP_NET_BIND_SERVICE` para porta < 1024, a resposta certa é usar porta 8000 no container e mapear via Service.

> **Nota:** capabilities são a fatia de "poderes de root" que o kernel Linux passou a conceder separadamente desde a versão 2.2. Dropar `ALL` e adicionar de volta só o estritamente necessário é a prática recomendada — na `store-api` não precisamos de nenhuma.

---

### 10.6 Pod Security Admission (PSA)

Até o Kubernetes 1.21 existia a **PodSecurityPolicy (PSP)**, um recurso `policy/v1beta1` que era ao mesmo tempo objeto de política e sujeito de RBAC. Era confuso (a PSP escolhida dependia de ordenação alfabética e das permissões da ServiceAccount), difícil de depurar e foi **depreciada em 1.21 e removida em 1.25**. Não use, não cite em prova como solução atual — cite como história.

O substituto embutido é o **Pod Security Admission**, estável desde o 1.25. É um admission controller nativo que avalia Pods contra os **Pod Security Standards**, três perfis pré-definidos:

| Perfil | Descrição | Uso típico |
|---|---|---|
| `privileged` | sem restrição alguma | componentes de infraestrutura (CNI, CSI, agentes de log) |
| `baseline` | bloqueia escalonamentos conhecidos: `privileged`, `hostNetwork`, `hostPID`, `hostPath`, capabilities perigosas | maioria das aplicações legadas |
| `restricted` | tudo do baseline mais: `runAsNonRoot`, `allowPrivilegeEscalation: false`, `capabilities.drop: ALL`, `seccompProfile: RuntimeDefault`, volumes limitados | **alvo para aplicações novas** |

A configuração é feita por **labels no namespace**, com três modos:

- `enforce` — rejeita o Pod que viola.
- `audit` — permite, mas registra no log de auditoria.
- `warn` — permite, mas devolve um aviso ao usuário no `kubectl`.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: store-prod
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/enforce-version: v1.33
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

Estratégia de adoção recomendada, e que funciona no mundo real:

1. Comece com `warn: restricted` e `audit: restricted` em todos os namespaces, **sem** `enforce`. Ninguém quebra.
2. Colete os avisos por algumas semanas e corrija os workloads.
3. Só então promova para `enforce: baseline` e, por fim, `enforce: restricted`.

> **Na prática:** o `enforce-version` fixa a versão dos standards. Sem ele, um upgrade do cluster pode endurecer a política silenciosamente e quebrar deploys num domingo de manutenção.

Tentando aplicar um Pod privilegiado num namespace `restricted`:

```
Error from server (Forbidden): error when creating "pod-mau.yaml":
pods "pod-mau" is forbidden: violates PodSecurity "restricted:v1.33":
privileged (container "hack" must not set securityContext.privileged=true),
allowPrivilegeEscalation != false, unrestricted capabilities, runAsNonRoot != true,
seccompProfile (pod or container must set securityContext.seccompProfile.type
to "RuntimeDefault" or "Localhost")
```

Note que a mensagem lista **todas** as violações de uma vez — é um dos melhores erros do Kubernetes, use-o como checklist.

> **Atenção:** PSA avalia apenas Pods. Se você aplica um Deployment que viola o perfil, o Deployment é **criado com sucesso** e quem falha é o ReplicaSet ao tentar criar o Pod. O sintoma é `kubectl get deploy` mostrando `0/3` e o motivo real só aparece em `kubectl describe rs` ou nos eventos. Isso confunde muita gente.

---

### 10.7 Control plane, etcd, criptografia e auditoria

Em EKS o control plane é gerenciado pela AWS — ela cuida do etcd, do API server, dos certificados e do patching desses componentes. Isso é ótimo, mas não isenta você de decisões:

**Endpoint do API server.** Por padrão o EKS expõe o endpoint publicamente. Restrinja:

```bash
aws eks update-cluster-config --name fiap-store \
  --resources-vpc-config \
  endpointPublicAccess=true,publicAccessCidrs="200.150.10.0/24",endpointPrivateAccess=true
```

**Encryption at rest de Secrets (envelope encryption com KMS).** Sem isso, um Secret está apenas em base64 dentro do etcd — o que significa "em texto claro com um passo a mais".

```bash
aws eks associate-encryption-config --cluster-name fiap-store \
  --encryption-config '[{"resources":["secrets"],"provider":{"keyArn":"arn:aws:kms:us-east-1:123456789012:key/abcd-1234"}}]'
```

Em cluster autogerenciado, o equivalente é o `EncryptionConfiguration` no API server:

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources: ["secrets"]
    providers:
      - aescbc:
          keys:
            - name: chave-1
              secret: <base64 de 32 bytes>
      - identity: {}
```

> **Atenção:** a ordem dos providers importa. O primeiro é usado para **escrever**; todos são tentados para **ler**. Deixar `identity: {}` em primeiro lugar significa gravar tudo em texto claro.

**Auditoria.** O audit log do API server responde "quem fez o quê, quando, em qual recurso e o servidor deixou?". No EKS, habilite os logs de control plane:

```bash
aws eks update-cluster-config --name fiap-store \
  --logging '{"clusterLogging":[{"types":["api","audit","authenticator"],"enabled":true}]}'
```

Uma audit policy mínima e sensata, para cluster próprio:

```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
omitStages:
  - RequestReceived
rules:
  # não polua o log com leituras de rotina
  - level: None
    verbs: ["get", "list", "watch"]
    resources:
      - group: ""
        resources: ["events"]
  # tudo sobre Secrets e ConfigMaps: metadata apenas (não vaze o conteúdo!)
  - level: Metadata
    resources:
      - group: ""
        resources: ["secrets", "configmaps"]
  # exec e port-forward são altíssimo risco: corpo completo
  - level: RequestResponse
    resources:
      - group: ""
        resources: ["pods/exec", "pods/portforward", "pods/attach"]
  # resto: metadata
  - level: Metadata
```

**Rotação de certificados.** O cluster usa PKI própria: CA, certificados de kubelet, de etcd, do API server. Em kubeadm, `kubeadm certs check-expiration` e `kubeadm certs renew all`; os certificados de cliente do kubelet rotacionam sozinhos com `rotateCertificates: true`. Em EKS, a AWS rotaciona. O que continua sendo seu: **credenciais de acesso humano** — nunca distribua o `kubeconfig` de admin; use IAM/OIDC com tokens de curta duração (capítulo 9).

---

### 10.8 Supply chain e políticas de admissão (Kyverno / OPA Gatekeeper)

RBAC responde "quem pode criar um Pod?". Policy engines respondem "**que tipo** de Pod pode ser criado?". Eles se plugam no API server como `ValidatingAdmissionWebhook` (e mutating, quando alteram o objeto).

| | **Kyverno** | **OPA Gatekeeper** |
|---|---|---|
| Linguagem | YAML (nativo Kubernetes) | Rego (linguagem própria) |
| Curva de aprendizado | baixa | média-alta |
| Mutação de recursos | sim, madura | sim, mais limitada |
| Geração de recursos | sim (gera Secret/NetworkPolicy em novo namespace) | não |
| Verificação de assinatura de imagem | nativa (`verifyImages`) | via extensões |
| Expressividade em lógica complexa | menor | maior |
| Adoção | crescendo rápido | consolidada em grandes empresas |

Para uma turma e para 90% das empresas, **Kyverno** é a escolha pragmática: se você sabe escrever YAML de Kubernetes, você sabe escrever policy.

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
```

Policy real nº 1 — **barrar a tag `latest`** (antipadrão clássico: destrói a reprodutibilidade e faz cada Pod novo poder rodar um código diferente dos irmãos):

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: proibir-tag-latest
  annotations:
    policies.kyverno.io/title: Proibir tag latest
    policies.kyverno.io/category: Boas Práticas
spec:
  validationFailureAction: Enforce   # Audit para testar antes
  background: true
  rules:
    - name: exigir-tag-explicita
      match:
        any:
          - resources:
              kinds: ["Pod"]
              namespaces: ["store-dev", "store-prod"]
      validate:
        message: >-
          Imagem com tag ':latest' (ou sem tag) e proibida.
          Use uma tag imutavel, ex.: store-api:1.1
        pattern:
          spec:
            containers:
              - image: "!*:latest"
```

Policy real nº 2 — **exigir assinatura Cosign**, fechando o elo da supply chain:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: exigir-imagem-assinada
spec:
  validationFailureAction: Enforce
  webhookTimeoutSeconds: 30
  rules:
    - name: verificar-assinatura-store
      match:
        any:
          - resources:
              kinds: ["Pod"]
              namespaces: ["store-prod"]
      verifyImages:
        - imageReferences:
            - "docker.io/fabiobara/store-*"
          attestors:
            - entries:
                - keys:
                    publicKeys: |-
                      -----BEGIN PUBLIC KEY-----
                      MFkwEwYHKoZIzj0CAQYIKoZIzj0DAQcDQgAE...
                      -----END PUBLIC KEY-----
```

Policy real nº 3 — **exigir requests e limits**, evitando o Pod que come o nó inteiro:

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: exigir-recursos
spec:
  validationFailureAction: Enforce
  rules:
    - name: requests-e-limits-obrigatorios
      match:
        any:
          - resources:
              kinds: ["Pod"]
              namespaces: ["store-prod"]
      validate:
        message: "Todo container deve declarar requests de cpu/memoria e limit de memoria."
        pattern:
          spec:
            containers:
              - resources:
                  requests:
                    cpu: "?*"
                    memory: "?*"
                  limits:
                    memory: "?*"
```

> **Na prática:** sempre publique uma policy primeiro com `validationFailureAction: Audit`, olhe os `PolicyReport` (`kubectl get polr -A`) por uma semana, e só então mude para `Enforce`. Policy nova em Enforce direto é a forma mais rápida de parar a produção de todo mundo e perder a confiança do time.

---

### 10.9 OWASP Kubernetes Top 10

A OWASP mantém uma lista dos dez riscos mais relevantes em Kubernetes. Vale decorar — é ótimo material de entrevista e melhor ainda de auditoria.

| # | Risco | Comentário e mitigação |
|---|---|---|
| **K01** | Workload inseguro por configuração | O campeão absoluto. `privileged: true`, `hostPath` em `/`, `hostPID`, root. Mitigação: PSA `restricted` + Kyverno + `securityContext` (seções 10.5/10.6). |
| **K02** | Vulnerabilidades da supply chain | Imagem de origem duvidosa, dependência comprometida, registry sem controle. Mitigação: Trivy no CI, Cosign, SBOM, registry privado (ECR) com scan on push. |
| **K03** | Controles de acesso RBAC excessivos | `cluster-admin` distribuído "temporariamente", wildcards em `verbs`/`resources`. Mitigação: capítulo 9 — menor privilégio, revisão periódica, `kubectl auth can-i --list`. |
| **K04** | Falta de políticas centralizadas | Cada time faz do seu jeito e ninguém consegue provar conformidade. Mitigação: policy-as-code versionada em Git (Kyverno/Gatekeeper), aplicada por GitOps (capítulo 11). |
| **K05** | Logging e monitoramento inadequados | O incidente aconteceu e não há trilha. Mitigação: audit log ligado e **exportado para fora do cluster**, retenção definida, alertas sobre `exec` em produção. |
| **K06** | Autenticação quebrada | Certificados de longa duração, tokens de ServiceAccount eternos, usuários compartilhados. Mitigação: OIDC/IRSA, tokens projetados com expiração, MFA no provedor de identidade. |
| **K07** | Controles de segmentação de rede ausentes | Sem NetworkPolicy, **todo Pod fala com todo Pod**, inclusive com o `store-db`. Mitigação: default-deny por namespace + allow explícito (capítulo 5). |
| **K08** | Falhas na gestão de segredos | Secret em ConfigMap, senha no `env` do Deployment, `.env` commitado. Mitigação: encryption at rest, External Secrets/Sealed Secrets, IRSA em vez de chave de acesso. |
| **K09** | Componentes do cluster mal configurados | etcd exposto, kubelet com `anonymous-auth`, dashboard sem autenticação, `--insecure-port`. Mitigação: CIS Benchmark com `kube-bench`, endpoint privado, versões suportadas. |
| **K10** | Componentes desatualizados | Kubernetes fora de suporte, CNI antigo, ingress controller com CVE. Mitigação: calendário de upgrade (uma minor a cada ~4 meses), inventário de add-ons. |

```bash
# auditoria rápida do cluster contra o CIS Benchmark
kubectl apply -f https://raw.githubusercontent.com/aquasecurity/kube-bench/main/job.yaml
kubectl logs -f job/kube-bench
```

---

### 10.10 Operação: mantendo a loja no ar

Segurança evita o desastre raro. Operação evita o desastre diário — que é perder requisição em cada deploy.

#### 10.10.1 Health checks bem feitos

Revisando os três probes com foco em erros de calibração:

| Probe | Pergunta | Falha causa | Erro comum |
|---|---|---|---|
| `startupProbe` | "já terminou de subir?" | reinicia o container após `failureThreshold × periodSeconds` | não usar em app de start lento e ter que afrouxar o liveness |
| `readinessProbe` | "posso receber tráfego agora?" | remove o Pod dos Endpoints do Service | apontar para um endpoint que checa o banco — o banco pisca e **toda** a frota sai do ar |
| `livenessProbe` | "o processo está saudável ou travou?" | mata e reinicia o container | apontar para `/ready` ou para rota que depende de terceiros — cria loop de restart em cascata |

```yaml
          startupProbe:
            httpGet: { path: /health, port: 8000 }
            periodSeconds: 5
            failureThreshold: 30       # tolera até 150s de boot
          readinessProbe:
            httpGet: { path: /ready, port: 8000 }
            periodSeconds: 5
            timeoutSeconds: 2
            failureThreshold: 3
          livenessProbe:
            httpGet: { path: /health, port: 8000 }
            periodSeconds: 10
            timeoutSeconds: 2
            failureThreshold: 3
```

> **Nota:** a regra de ouro é **liveness verifica só o processo; readiness verifica as dependências**. `/health` responde 200 se o event loop está vivo. `/ready` pode checar o pool do Postgres. Nunca o contrário.

#### 10.10.2 Graceful shutdown: o bug invisível de todo rolling update

Este é o problema que mais aparece em produção e menos aparece em aula. A sequência ao deletar um Pod é:

```
kubectl delete pod  /  rolling update
        |
        +--> Pod marcado Terminating (deletionTimestamp)
        |
        +--(A)--> endpoints controller remove o Pod do Service
        |          -> kube-proxy/iptables atualizam em CADA nó (assíncrono!)
        |
        +--(B)--> kubelet executa preStop hook
        |          -> depois envia SIGTERM ao PID 1
        |
        +--> espera terminationGracePeriodSeconds
        |
        +--> SIGKILL se ainda estiver vivo
```

**A e B acontecem em paralelo.** Não há garantia de que (A) termine antes de (B). Se a aplicação morre instantaneamente com o SIGTERM, mas o iptables de um nó ainda não foi atualizado, esse nó continua mandando requisições para um Pod morto → `502 Bad Gateway`, `connection reset`.

A correção tem três partes, e as três são necessárias:

**1. Um `preStop` que apenas espera** — dá tempo para a propagação dos Endpoints:

```yaml
          lifecycle:
            preStop:
              exec:
                command: ["sleep", "10"]
```

Em imagem distroless não há `sleep`. Use o `sleep` nativo do Kubernetes 1.30+:

```yaml
          lifecycle:
            preStop:
              sleep:
                seconds: 10
```

**2. `terminationGracePeriodSeconds` maior que preStop + tempo da requisição mais longa:**

```yaml
      terminationGracePeriodSeconds: 45
```

O relógio do grace period começa **junto com o preStop**, não depois. Se preStop dorme 10s e o grace period é 10s, seu app recebe SIGTERM e SIGKILL quase juntos.

**3. A aplicação precisa tratar SIGTERM.** O uvicorn já faz isso: ao receber SIGTERM ele para de aceitar novas conexões e aguarda as em andamento. O que quebra é o código de negócio com recursos abertos. No FastAPI, use o `lifespan`:

```python
# main.py — store-api com shutdown gracioso
import asyncio, os, signal, socket
from contextlib import asynccontextmanager
from fastapi import FastAPI, Response

PRONTO = True

@asynccontextmanager
async def lifespan(app: FastAPI):
    # ----- startup -----
    app.state.pool = await criar_pool_db()
    yield
    # ----- shutdown (roda ao receber SIGTERM) -----
    global PRONTO
    PRONTO = False                  # /ready passa a responder 503
    await asyncio.sleep(5)          # drena o que ja esta em voo
    await app.state.pool.close()    # fecha conexoes com o Postgres

app = FastAPI(title="FIAP Store API", lifespan=lifespan)

@app.get("/health")
def health():
    return {"status": "ok", "host": socket.gethostname()}

@app.get("/ready")
def ready(response: Response):
    if not PRONTO:
        response.status_code = 503
        return {"status": "shutting-down"}
    return {"status": "ready"}

@app.get("/produtos")
def produtos():
    return [{"id": 1, "nome": "Teclado", "preco": 199.9}]

@app.get("/info")
def info():
    return {"env": os.getenv("APP_ENV", "dev"),
            "db_host": os.getenv("DB_HOST", "nao-configurado"),
            "pod": socket.gethostname()}

async def criar_pool_db():
    # placeholder didatico: em producao use asyncpg/SQLAlchemy
    return type("Pool", (), {"close": lambda self: asyncio.sleep(0)})()
```

> **Atenção:** se o `ENTRYPOINT` for `sh -c "uvicorn ..."`, o PID 1 será o shell, que **não repassa sinais**. O container só morre por SIGKILL e todo o graceful shutdown é ficção. Use sempre a forma exec (`ENTRYPOINT ["python", "-m", "uvicorn", ...]`), como no Dockerfile da seção 10.4.2.

#### 10.10.3 PodDisruptionBudget

O PDB protege contra **disrupções voluntárias** — `kubectl drain` para manutenção, upgrade de nó, Cluster Autoscaler compactando o cluster. Ele não protege contra queda de hardware (isso é disrupção involuntária).

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-api-pdb
  namespace: store-prod
spec:
  minAvailable: 2          # ou maxUnavailable: 1
  selector:
    matchLabels:
      app: store-api
```

> **Atenção:** `minAvailable: 1` com `replicas: 1` **trava o drain para sempre** — o nó nunca esvazia e o upgrade fica pendurado. Para workloads de réplica única, use `maxUnavailable: 1` ou aceite a indisponibilidade. Prefira expressar em percentual (`minAvailable: 50%`) para o PDB acompanhar o HPA.

#### 10.10.4 Requests, limits e o problema do CPU throttling

`requests` é o que o **scheduler** usa para escolher o nó e o que o kernel usa como peso relativo de CPU. `limits` é o teto imposto pelo cgroup. As consequências são assimétricas e isso é fundamental:

- **Memória é incompressível.** Estourou o limit → o kernel mata o processo: `OOMKilled`, exit code 137.
- **CPU é compressível.** Estourou o limit → o processo é **throttled**: o kernel simplesmente não o agenda no restante do período de 100 ms do CFS.

O throttling é traiçoeiro porque **não gera erro, gera latência**. Um Pod com `limits.cpu: 500m` recebe 50 ms de CPU a cada 100 ms. Se uma requisição precisa de 60 ms de CPU, ela é congelada por 50 ms no meio do caminho e o p99 dispara — com o Pod aparentemente "saudável", uso médio em 45% e nenhum erro no log.

```bash
# ver throttling do container
kubectl exec -n store-prod store-api-7c9d -- \
  cat /sys/fs/cgroup/cpu.stat
```

```
usage_usec 184920134
nr_periods 1204331
nr_throttled 271654      <-- 22% dos periodos foram estrangulados
throttled_usec 9812445
```

**Recomendação atual da comunidade** (e do EKS Best Practices Guide):

| Recurso | Requests | Limits |
|---|---|---|
| CPU | **sempre defina** (p95 real de uso) | **normalmente omita** |
| Memória | sempre defina | sempre defina, **igual ao request** |

Omitir `limits.cpu` permite que o Pod aproveite CPU ociosa do nó; o `requests` já garante a fatia mínima e o compartilhamento justo sob pressão. Memória com request igual ao limit coloca o Pod na QoS **Guaranteed**, que é a última a ser despejada.

Classes de QoS, que definem a ordem de eviction sob pressão de memória do nó:

| Classe | Condição | Ordem de eviction |
|---|---|---|
| `Guaranteed` | requests == limits para CPU e memória em todos os containers | última |
| `Burstable` | tem requests, mas requests != limits | meio |
| `BestEffort` | nenhum request nem limit | **primeira a morrer** |

#### 10.10.5 LimitRange e ResourceQuota

`LimitRange` aplica padrões e tetos **por objeto** dentro do namespace; `ResourceQuota` limita o **total agregado** do namespace.

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: limites-padrao
  namespace: store-dev
spec:
  limits:
    - type: Container
      default:                 # limits aplicados se o Pod nao declarar
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:          # requests aplicados se o Pod nao declarar
        cpu: "100m"
        memory: "128Mi"
      max:
        cpu: "2"
        memory: "2Gi"
      min:
        cpu: "50m"
        memory: "64Mi"
---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: cota-store-dev
  namespace: store-dev
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    persistentvolumeclaims: "5"
    count/deployments.apps: "10"
    count/services.loadbalancers: "1"
```

> **Nota:** ao criar um ResourceQuota que inclua `requests.cpu`, **todo** Pod do namespace passa a ser obrigado a declarar requests, ou é rejeitado. Combine sempre com um LimitRange que forneça os defaults, senão os deploys existentes quebram.

---

### 10.11 Observabilidade

Monitoramento responde perguntas que você já sabia fazer. Observabilidade permite fazer perguntas novas sobre um sistema em produção sem precisar de um novo deploy. São três pilares:

**Logs.** No Kubernetes, a aplicação escreve em `stdout`/`stderr` e ponto final. Nunca escreva em arquivo dentro do container — ele some com o Pod, enche o disco do nó e não é coletado. Um agente DaemonSet (Fluent Bit, Vector) lê `/var/log/containers/*.log` do nó e envia para o backend (CloudWatch Logs, Loki, OpenSearch). Prefira **JSON estruturado**, com `request_id` para correlação.

**Métricas.** O padrão é **Prometheus** raspando endpoints `/metrics` no formato OpenMetrics. A stack mínima:

- `prometheus` — coleta e armazena séries temporais.
- `kube-state-metrics` — expõe o **estado dos objetos** (quantas réplicas desejadas x prontas, Pods em CrashLoopBackOff, Jobs falhos). Não confunda com...
- `metrics-server` — expõe uso instantâneo de CPU/memória para o HPA e o `kubectl top`. Não serve para histórico.
- `node-exporter` — métricas do sistema operacional dos nós.
- `grafana` — visualização.

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install kube-prom prometheus-community/kube-prometheus-stack \
  -n monitoring --create-namespace
kubectl port-forward -n monitoring svc/kube-prom-grafana 3000:80
```

Instrumentando a `store-api`:

```python
from prometheus_fastapi_instrumentator import Instrumentator
Instrumentator().instrument(app).expose(app)   # publica /metrics
```

E dizendo ao Prometheus Operator para raspar:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: store-api
  namespace: store-prod
  labels:
    release: kube-prom
spec:
  selector:
    matchLabels:
      app: store-api
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

**Tracing.** Métricas dizem "o p99 subiu"; o trace diz "subiu porque a chamada ao `store-db` na rota `/produtos` passou de 8 ms para 900 ms". O padrão é **OpenTelemetry**, vendor-neutro, exportando para Jaeger, Tempo ou AWS X-Ray.

```python
from opentelemetry.instrumentation.fastapi import FastAPIInstrumentor
FastAPIInstrumentor.instrument_app(app)
```

**Os 4 sinais dourados** (Google SRE Book), o que monitorar quando não se sabe o que monitorar:

| Sinal | O que é | Exemplo na `store-api` |
|---|---|---|
| **Latência** | tempo de resposta — separe sucesso de erro | p50/p95/p99 de `/produtos` |
| **Tráfego** | demanda sobre o sistema | requisições/s |
| **Erros** | taxa de falha | % de respostas 5xx |
| **Saturação** | quão cheio está o recurso mais escasso | CPU throttling, uso de memória, fila do worker |

**SLI, SLO e error budget.** O SLI é a medida (99,3% das requisições em 30 dias responderam < 300 ms). O SLO é a meta (99,5%). O **error budget** é o que sobra: com SLO de 99,5% ao mês, você pode falhar 0,5% — cerca de 3h39min. Enquanto houver orçamento, a equipe lança funcionalidades; quando o orçamento acaba, congela-se o lançamento e prioriza-se confiabilidade. É assim que se transforma "estabilidade" de opinião em número negociável.

---

### 10.12 Gestão de configuração: Helm x Kustomize

Manter 15 arquivos YAML duplicados para dev e prod é insustentável. Existem duas abordagens dominantes.

| | **Helm** | **Kustomize** |
|---|---|---|
| Modelo | templating (Go templates) + gerenciador de pacotes | sobreposição declarativa (patches) |
| Instalação | binário separado | embutido no `kubectl` (`-k`) |
| Variáveis | `values.yaml`, hierárquico | sem variáveis; patches sobre uma base |
| Distribuição para terceiros | excelente (repositórios de charts) | fraca |
| Versionamento/rollback | nativo (`helm rollback`) | via Git |
| Legibilidade do YAML fonte | menor (YAML com `{{ }}` não é YAML válido) | total (YAML puro sempre) |
| Hooks e lógica | ricos (`pre-install`, condicionais, loops) | mínimos |
| Melhor uso | empacotar software para outros consumirem | customizar manifestos dos seus próprios apps |

**Exemplo Helm** — `charts/store-api/values.yaml` e trecho do template:

```yaml
# values.yaml
replicaCount: 2
image:
  repository: docker.io/fabiobara/store-api
  tag: "1.1"
env: dev
resources:
  requests: { cpu: 100m, memory: 128Mi }
  limits:   { memory: 256Mi }
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-store-api
  labels:
    app: store-api
    env: {{ .Values.env }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        env: {{ .Values.env }}
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        seccompProfile: { type: RuntimeDefault }
      containers:
        - name: store-api
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 8000
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
```

```bash
helm install store-dev ./charts/store-api -n store-dev -f values-dev.yaml
helm upgrade store-prod ./charts/store-api -n store-prod -f values-prod.yaml
helm history store-prod
helm rollback store-prod 3
```

**Exemplo Kustomize** — estrutura com base e overlays:

```
k8s/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   └── service.yaml
└── overlays/
    ├── dev/
    │   ├── kustomization.yaml
    │   └── patch-replicas.yaml
    └── prod/
        ├── kustomization.yaml
        └── patch-prod.yaml
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml
commonLabels:
  app: store-api
  tier: backend
```

```yaml
# overlays/prod/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: store-prod
resources:
  - ../../base
labels:
  - pairs:
      env: prod
    includeSelectors: false
images:
  - name: docker.io/fabiobara/store-api
    newTag: "1.1"
replicas:
  - name: store-api
    count: 4
patches:
  - path: patch-prod.yaml
    target:
      kind: Deployment
      name: store-api
```

```yaml
# overlays/prod/patch-prod.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
spec:
  template:
    spec:
      terminationGracePeriodSeconds: 45
      containers:
        - name: store-api
          resources:
            requests: { cpu: "300m", memory: "512Mi" }
            limits:   { memory: "512Mi" }
```

```bash
kubectl kustomize k8s/overlays/prod   # renderiza sem aplicar
kubectl apply -k k8s/overlays/prod
```

> **Na prática:** a combinação mais comum hoje é usar Helm para software de terceiros (ingress-nginx, Prometheus, Argo CD) e Kustomize para os seus próprios apps. Os dois convivem: o Argo CD (capítulo 11) suporta ambos, e o Kustomize consegue inflar um chart Helm via `helmCharts:`.

---

### 10.13 Upgrade, backup e DR

**Upgrade.** O Kubernetes lança uma minor a cada ~4 meses e mantém suporte por ~14 meses (3 minors). Regras:

- Suba **uma minor por vez** (1.32 → 1.33 → 1.34). Pular versões não é suportado.
- Ordem: **control plane → add-ons (CNI, CoreDNS, kube-proxy) → nós de trabalho**.
- O `kubelet` pode estar até 3 minors atrás do API server, nunca à frente.
- Verifique APIs depreciadas antes: `kubectl deprecations` (plugin krew), `pluto detect-all-in-cluster` ou `kubent`.
- Em EKS, prefira **blue/green de node group**: crie um novo managed node group na versão nova, faça `cordon` + `drain` do antigo respeitando os PDBs, e delete o antigo. Rollback é simplesmente não deletar o velho.

```bash
eksctl upgrade cluster --name fiap-store --version 1.34 --approve
eksctl create nodegroup --cluster fiap-store --name ng-1-34 --nodes 3
kubectl drain ip-10-0-1-45.ec2.internal --ignore-daemonsets --delete-emptydir-data
eksctl delete nodegroup --cluster fiap-store --name ng-1-33
```

**Backup.** Duas coisas distintas precisam ser salvas:

1. **Estado do cluster (objetos da API).** Em cluster autogerenciado, snapshot do etcd:

```bash
ETCDCTL_API=3 etcdctl snapshot save /backup/etcd-$(date +%F).db \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key
etcdctl snapshot status /backup/etcd-2026-08-06.db -w table
```

Em EKS você não tem acesso ao etcd — o backup do estado é feito com **Velero** ou, melhor ainda, pelo próprio Git (é aqui que GitOps deixa de ser moda e vira estratégia de DR: se todo o estado desejado está em Git, recriar o cluster é reaplicar o repositório).

2. **Dados persistentes (PVCs).** Snapshots de EBS via `VolumeSnapshot` (CSI) ou Velero com o plugin de snapshot.

```bash
velero install --provider aws --bucket fiap-store-backup \
  --plugins velero/velero-plugin-for-aws:v1.10.0 --use-node-agent
velero backup create store-prod-diario --include-namespaces store-prod --wait
velero schedule create diario --schedule="0 3 * * *" --include-namespaces store-prod
velero restore create --from-backup store-prod-diario
```

**DR.** Defina **RTO** (quanto tempo até voltar) e **RPO** (quanto dado posso perder). Um backup nunca testado não é backup — é esperança. Agende um exercício trimestral de restauração em cluster descartável.

---

### 10.14 Checklist de produção (pré-voo)

Antes de declarar um workload "pronto para produção", percorra a lista. Ela é longa de propósito: cada item existe porque alguém, em algum lugar, teve um incidente.

**Imagem e build**
- [ ] Imagem base mínima (distroless/alpine/chainguard), multi-stage build
- [ ] `USER` numérico não-root no Dockerfile
- [ ] Tag imutável ou digest — **nunca** `:latest`
- [ ] Scan sem CVE CRITICAL corrigível (Trivy no CI com `--exit-code 1`)
- [ ] SBOM gerado e armazenado; imagem assinada com Cosign
- [ ] `.dockerignore` cobrindo `.git`, `.env`, credenciais
- [ ] Registry privado (ECR) com scan on push e política de retenção

**Segurança do workload**
- [ ] `securityContext`: `runAsNonRoot`, `runAsUser`, `allowPrivilegeEscalation: false`
- [ ] `readOnlyRootFilesystem: true` com `emptyDir` nos caminhos graváveis
- [ ] `capabilities.drop: ["ALL"]` e `seccompProfile: RuntimeDefault`
- [ ] Namespace com PSA `enforce: restricted` (ou `baseline` justificado)
- [ ] `automountServiceAccountToken: false` se o app não usa a API
- [ ] ServiceAccount dedicada, RBAC de menor privilégio, sem `cluster-admin`
- [ ] NetworkPolicy default-deny + allow explícito de ingress e egress
- [ ] Secrets fora do Git (External Secrets/Sealed Secrets), encryption at rest ativa
- [ ] Sem `hostNetwork`, `hostPID`, `hostPath` ou `privileged`

**Confiabilidade**
- [ ] `replicas >= 2` e `podAntiAffinity` (ou `topologySpreadConstraints`) entre AZs
- [ ] `readinessProbe` e `livenessProbe` distintos e corretamente calibrados
- [ ] `startupProbe` se o boot passa de ~20 s
- [ ] `preStop` (sleep 5-15 s) + `terminationGracePeriodSeconds` adequado
- [ ] Aplicação trata SIGTERM; `ENTRYPOINT` em forma exec
- [ ] `PodDisruptionBudget` compatível com o número de réplicas
- [ ] `strategy.rollingUpdate` com `maxUnavailable: 0` para zero-downtime
- [ ] HPA configurado com métrica que representa a carga real

**Recursos**
- [ ] `requests` de CPU e memória definidos com base em medição
- [ ] `limits` de memória definidos (= request para QoS Guaranteed)
- [ ] `limits` de CPU omitidos ou justificados; throttling monitorado
- [ ] `LimitRange` e `ResourceQuota` no namespace
- [ ] `priorityClassName` definido para cargas críticas

**Observabilidade**
- [ ] Logs em stdout/stderr, estruturados em JSON, com `request_id`
- [ ] Agregação de logs fora do cluster, com retenção definida
- [ ] `/metrics` exposto e `ServiceMonitor` criado
- [ ] Dashboard com os 4 sinais dourados
- [ ] Alertas acionáveis (que exigem ação humana) e SLO documentado
- [ ] Tracing distribuído instrumentado (OpenTelemetry)

**Operação**
- [ ] Manifestos versionados em Git (Helm/Kustomize), aplicados por GitOps
- [ ] Runbook para os alertas principais
- [ ] Backup automatizado **e restauração testada**
- [ ] Plano de upgrade do cluster e das dependências
- [ ] Ambiente de staging equivalente ao de produção
- [ ] Custos monitorados por namespace/label

---

### 10.15 FinOps de cluster

Kubernetes torna trivial desperdiçar dinheiro: pede-se 2 CPUs "por garantia", multiplica-se por 40 microsserviços e por 3 ambientes. As alavancas principais:

**Rightsizing.** A maioria dos clusters roda com utilização real de CPU entre 10% e 20% do que foi requisitado. Meça o p95 real por 2 semanas e ajuste. Ferramentas: **VPA em modo `Off`** (recomenda sem aplicar), Goldilocks, Kubecost/OpenCost.

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: store-api-vpa
  namespace: store-prod
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: store-api
  updatePolicy:
    updateMode: "Off"     # apenas recomenda; nao reinicia Pods
```

```bash
kubectl describe vpa store-api-vpa -n store-prod
# Recommendation:
#   Target:  cpu: 87m, memory: 143Mi     <-- voce pediu 500m/512Mi
```

**Spot instances.** Até 90% mais baratas, com aviso de interrupção de 2 minutos. Perfeitas para `store-worker` (Jobs, processamento em batch, stateless com réplicas). Combine com `Karpenter`, taints/tolerations e PDBs. Nunca coloque `store-db` em spot.

**Scale-to-zero.** Ambientes de dev não precisam existir à noite nem no fim de semana — são 128 das 168 horas semanais, 76% de economia. Um CronJob com `kubectl scale --replicas=0` resolve. Para escalar a zero **sob demanda**, o caminho é KEDA ou Knative (que já é serverless — veja a Ponte abaixo).

**Outras alavancas:** Cluster Autoscaler/Karpenter com consolidação, Graviton (ARM) com ~20% melhor custo-desempenho, remover PVs órfãos, ajustar retenção de logs, revisar Load Balancers ociosos (cada NLB custa ~US$ 18/mês parado), Savings Plans para a base estável.

> **Atenção (custos do curso):** um cluster EKS custa US$ 0,10/hora só de control plane, mais as instâncias EC2, mais os EBS, mais os Load Balancers. Ao terminar cada laboratório: `eksctl delete cluster --name fiap-store --wait` e confira no console se os Load Balancers e volumes EBS realmente sumiram — Services do tipo `LoadBalancer` deixam recursos órfãos se o namespace for deletado à força.

---

### 10.16 Ponte com Serverless

Quando movemos a `store-api` de um Deployment no EKS para AWS Lambda (ou para um Knative Service), a conversa sobre segurança muda de lugar — mas não desaparece.

**A superfície de ataque encolhe de verdade.** Não há nó para invadir, não há kubelet para explorar, não há container de longa duração para instalar um minerador. O runtime da função vive por minutos e o sandbox (Firecracker microVM, no caso da AWS) é descartado. Persistência — o objetivo do atacante depois da invasão inicial — fica muito mais difícil: não há onde plantar o backdoor. Boa parte dos itens K01 e K09 do OWASP Kubernetes Top 10 simplesmente deixa de existir, porque não existe `securityContext` para configurar errado nem kubelet para deixar com `anonymous-auth`.

**A responsabilidade compartilhada desloca a linha, não a apaga.** Na AWS, o modelo é explícito: a AWS é responsável pela segurança **da** nuvem; você, pela segurança **na** nuvem. Em EC2 puro você patcheia o SO. Em EKS a AWS cuida do control plane, você cuida dos nós, das imagens e dos workloads. Em Fargate a AWS cuida também dos nós. Em Lambda a AWS cuida de tudo abaixo do seu handler. Cada degrau para cima transfere trabalho de patching — mas transfere junto o controle.

**O que continua sendo culpa do dev, sempre:**

- **Dependências.** Um `requirements.txt` com `starlette` vulnerável é exatamente igual de perigoso dentro de uma Lambda. Trivy escaneia layer e pacote de deployment do mesmo jeito. Este item (K02) não muda em nada.
- **IAM.** Aqui o risco até **aumenta**: uma função com `AdministratorAccess` porque "era mais rápido" é o equivalente serverless do `cluster-admin`. Uma execution role por função, com o mínimo necessário.
- **Dados e código.** Injeção de SQL, deserialização insegura, segredo em variável de ambiente em texto claro, log com CPF do cliente — nada disso é responsabilidade do provedor. Use Secrets Manager/Parameter Store com criptografia KMS, não `env` cru.
- **Configuração da borda.** API Gateway sem autorizador, bucket S3 público como gatilho, tópico SNS aberto. O perímetro virou configuração de serviço gerenciado.

**Comparação de patching.** Em Kubernetes, sair de um CVE do OpenSSL exige rebuild da imagem, novo push, rollout dos Pods **e** upgrade da AMI dos nós — dias de trabalho coordenado. Em Lambda com runtime gerenciado, a AWS atualiza o runtime e a próxima invocação já roda corrigido; você só cuida das suas dependências. Em compensação, você não escolhe **quando** essa atualização acontece — um runtime pode ser depreciado e forçar migração de versão do Python no calendário da AWS, não no seu. A escolha, portanto, não é "serverless é mais seguro", e sim: **você prefere controlar o patching ou delegá-lo?**

---

### 10.17 Laboratório guiado: endurecendo a `store-api`

Vamos aplicar na prática. Ambiente: minikube 1.35+ com Kubernetes 1.33.

**Passo 1 — preparar o cluster e o namespace com PSA `restricted`**

```bash
minikube start --kubernetes-version=v1.33.0 --cpus=4 --memory=6g
kubectl create namespace store-prod
kubectl label namespace store-prod \
  pod-security.kubernetes.io/enforce=restricted \
  pod-security.kubernetes.io/enforce-version=v1.33 \
  pod-security.kubernetes.io/warn=restricted
kubectl get ns store-prod --show-labels
```

```
NAME         STATUS   AGE   LABELS
store-prod   Active   8s    kubernetes.io/metadata.name=store-prod,
pod-security.kubernetes.io/enforce=restricted,...
```

**Passo 2 — comprovar que o namespace rejeita um Pod privilegiado**

```yaml
# pod-mau.yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-mau
  namespace: store-prod
spec:
  containers:
    - name: hack
      image: busybox:1.36
      command: ["sleep", "3600"]
      securityContext:
        privileged: true
```

```bash
kubectl apply -f pod-mau.yaml
```

```
Error from server (Forbidden): error when creating "pod-mau.yaml": pods "pod-mau"
is forbidden: violates PodSecurity "restricted:v1.33": privileged (container "hack"
must not set securityContext.privileged=true), allowPrivilegeEscalation != false,
unrestricted capabilities, runAsNonRoot != true, seccompProfile
```

Guarde esta saída — é o entregável do Exercício 10.1.

**Passo 3 — construir a imagem endurecida e escaneá-la**

Use o `Dockerfile` da seção 10.4.2 e o `main.py` da seção 10.10.2.

```bash
eval $(minikube docker-env)
docker build -t fabiobara/store-api:1.1 .
docker images fabiobara/store-api
```

```
REPOSITORY             TAG   IMAGE ID       SIZE
fabiobara/store-api    1.1   3f2a91bc4de7   68.4MB
fabiobara/store-api    1.0   a1b2c3d4e5f6   1.02GB
```

```bash
trivy image --severity HIGH,CRITICAL fabiobara/store-api:1.0   # antes
trivy image --severity HIGH,CRITICAL fabiobara/store-api:1.1   # depois
```

**Passo 4 — aplicar o Deployment endurecido com preStop e grace period**

```yaml
# store-api-hardened.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: store-api-sa
  namespace: store-prod
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-prod
  labels:
    app: store-api
    tier: backend
    env: prod
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: prod
    spec:
      serviceAccountName: store-api-sa
      automountServiceAccountToken: false
      terminationGracePeriodSeconds: 45
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        runAsGroup: 65532
        fsGroup: 65532
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: store-api
          image: fabiobara/store-api:1.1
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8000
          env:
            - name: APP_ENV
              value: "prod"
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              memory: "256Mi"
          readinessProbe:
            httpGet: { path: /ready, port: 8000 }
            periodSeconds: 5
            failureThreshold: 3
          livenessProbe:
            httpGet: { path: /health, port: 8000 }
            periodSeconds: 10
            failureThreshold: 3
          lifecycle:
            preStop:
              sleep:
                seconds: 10
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-prod
spec:
  selector:
    app: store-api
  ports:
    - name: http
      port: 80
      targetPort: 8000
---
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-api-pdb
  namespace: store-prod
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: store-api
```

```bash
kubectl apply -f store-api-hardened.yaml
kubectl get pods -n store-prod -o wide
```

```
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE
store-api-6b8f9c4d7-2xk9p   1/1     Running   0          42s   10.244.0.14  minikube
store-api-6b8f9c4d7-lm4rt   1/1     Running   0          42s   10.244.0.15  minikube
store-api-6b8f9c4d7-q7zwn   1/1     Running   0          42s   10.244.0.16  minikube
```

Confirme o endurecimento de dentro do Pod:

```bash
kubectl exec -n store-prod deploy/store-api -- id
# uid=65532 gid=65532
kubectl auth can-i --list --as=system:serviceaccount:store-prod:store-api-sa
```

**Passo 5 — instalar o Kyverno e barrar a tag `latest`**

```bash
helm repo add kyverno https://kyverno.github.io/kyverno/ && helm repo update
helm install kyverno kyverno/kyverno -n kyverno --create-namespace
kubectl -n kyverno rollout status deploy/kyverno-admission-controller
kubectl apply -f proibir-tag-latest.yaml     # YAML da secao 10.8
```

Testando:

```bash
kubectl run teste-latest -n store-prod --image=nginx:latest \
  --overrides='{"spec":{"securityContext":{"runAsNonRoot":true,"runAsUser":101,"seccompProfile":{"type":"RuntimeDefault"}},"containers":[{"name":"teste-latest","image":"nginx:latest","securityContext":{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}}}]}}'
```

```
Error from server: admission webhook "validate.kyverno.svc-fail" denied the request:

resource Pod/store-prod/teste-latest was blocked due to the following policies

proibir-tag-latest:
  exigir-tag-explicita: 'validation error: Imagem com tag '':latest'' (ou sem tag)
    e proibida. Use uma tag imutavel, ex.: store-api:1.1'
```

**Passo 6 — validar o rolling update sem perda de requisições**

Em um terminal, gere carga contínua:

```bash
kubectl run carga -n store-prod --rm -it --restart=Never \
  --image=busybox:1.36 --overrides='{"spec":{"securityContext":{"runAsNonRoot":true,"runAsUser":1000,"seccompProfile":{"type":"RuntimeDefault"}},"containers":[{"name":"carga","image":"busybox:1.36","command":["sh","-c","while true; do wget -q -O- --timeout=2 http://store-api.store-prod/produtos >/dev/null && echo OK || echo FALHA; sleep 0.2; done"],"securityContext":{"allowPrivilegeEscalation":false,"capabilities":{"drop":["ALL"]}}}]}}'
```

Em outro terminal, dispare o rollout:

```bash
kubectl set image deploy/store-api store-api=fabiobara/store-api:1.2 -n store-prod
kubectl rollout status deploy/store-api -n store-prod
```

Com preStop + readiness + `maxUnavailable: 0`, a saída da carga deve ser uma sequência ininterrupta de `OK`. Remova o `preStop` do Deployment, repita e observe as linhas `FALHA` aparecerem — essa comparação é o coração do Exercício 10.3.

**Limpeza**

```bash
kubectl delete namespace store-prod
helm uninstall kyverno -n kyverno && kubectl delete ns kyverno
minikube stop
```

---

### 10.18 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| Deployment com `0/3` e nenhum Pod criado | PSA rejeitando o Pod; o Deployment foi aceito, o ReplicaSet não | `kubectl describe rs -n store-prod` e olhar `Events` | Ajustar o `securityContext` ao perfil `restricted` |
| `container has runAsNonRoot and image has non-numeric user` | `USER nonroot` (nome) no Dockerfile | `kubectl describe pod` | Usar `USER 65532:65532` ou definir `runAsUser` no manifesto |
| `CreateContainerConfigError` após ativar `readOnlyRootFilesystem` | app tentando escrever em `/tmp` ou `/var/run` | `kubectl logs` + `kubectl describe pod` | Montar `emptyDir` nesses caminhos |
| Pod em `OOMKilled` (exit 137) | `limits.memory` menor que o pico real | `kubectl describe pod` → `Last State: Terminated, Reason: OOMKilled` | Medir com `kubectl top` / VPA e aumentar o limit |
| p99 alto sem erros e CPU média baixa | CPU throttling pelo `limits.cpu` | `cat /sys/fs/cgroup/cpu.stat` → `nr_throttled` alto | Aumentar ou remover `limits.cpu`; manter requests |
| 502/connection reset durante rolling update | Endpoints não propagaram antes do SIGTERM | Carga contínua durante o rollout; logs do ingress | `preStop` sleep 10s + `terminationGracePeriodSeconds` + `maxUnavailable: 0` |
| Container morre só com SIGKILL, ignora SIGTERM | `ENTRYPOINT` em shell form (PID 1 = `sh`) | `kubectl exec -- ps -o pid,comm` | Forma exec: `ENTRYPOINT ["python","-m","uvicorn",...]` |
| Toda a frota fica `NotReady` de uma vez | readiness checando o banco; o banco piscou | `kubectl get ep store-api` vazio; logs da probe | Readiness só do processo, ou readiness tolerante com cache |
| `kubectl drain` nunca termina | PDB com `minAvailable` >= réplicas | `kubectl get pdb -A` → `ALLOWED DISRUPTIONS 0` | Ajustar PDB ou aumentar réplicas |
| Deploy rejeitado por falta de requests logo após criar quota | ResourceQuota sem LimitRange correspondente | `kubectl describe quota -n store-dev` | Criar LimitRange com `defaultRequest` |
| Kyverno bloqueia tudo, inclusive kube-system | policy sem filtro de namespace e em `Enforce` | `kubectl get polr -A`, logs do admission controller | Restringir `match.namespaces`, voltar para `Audit` |
| Secret legível em `etcdctl get` | encryption at rest desabilitada | `etcdctl get /registry/secrets/... | hexdump` | `EncryptionConfiguration` com KMS/aescbc e reescrever os Secrets |
| Trivy acusa CVE que não tem correção | pacote marcado "won't fix" pela distro | ler a coluna `Fixed Version` | `--ignore-unfixed` no gate; trocar a base se for recorrente |

---

### 10.19 Resumo do capítulo

Operar Kubernetes com segurança é um exercício de disciplina distribuída em camadas. O modelo dos 4C organiza o pensamento: a nuvem sustenta o cluster, o cluster sustenta o container, o container sustenta o código — e uma falha em qualquer nível compromete tudo acima dela. Do lado da imagem, aprendemos que reduzir é proteger: base mínima, multi-stage, usuário não-root, scanning contínuo, assinatura e SBOM transformam a imagem de passivo em ativo auditável. No cluster, `securityContext` e Pod Security Admission traduzem o menor privilégio em YAML aplicável, e policy engines como o Kyverno permitem escrever as regras da casa como código versionado, revisável e testável.

Do lado da operação, vimos que a maior parte da indisponibilidade não vem de ataques, e sim de detalhes: uma readiness probe checando o banco, um `preStop` ausente, um `limits.cpu` apertado demais, um PDB impossível de satisfazer, um `ENTRYPOINT` em shell form engolindo o SIGTERM. Observabilidade — logs, métricas e traces sob a lente dos 4 sinais dourados e de SLOs com error budget — é o que transforma essas falhas em fatos mensuráveis, e não em opinião. Por fim, gestão de configuração (Helm/Kustomize), plano de upgrade, backup testado e higiene de custos completam o que se espera de um workload realmente pronto para produção.

**Pontos-chave**

1. Segurança é concêntrica (Cloud → Cluster → Container → Code); a camada mais externa mal configurada anula todas as internas.
2. Imagem mínima, não-root, com tag imutável, escaneada, assinada e com SBOM é a base de toda a supply chain segura.
3. `securityContext` restrito + PSA `restricted` no namespace + Kyverno em `Enforce` cobrem o risco número um do OWASP Kubernetes Top 10 (K01).
4. PSP está removida desde o 1.25; Pod Security Admission por labels de namespace é o substituto nativo, e deve ser adotado por `warn` → `audit` → `enforce`.
5. Zero-downtime exige quatro peças juntas: readiness correta, `preStop` que espera a propagação dos Endpoints, `terminationGracePeriodSeconds` folgado e tratamento real de SIGTERM.
6. Defina sempre requests; defina limit de memória igual ao request; desconfie de `limits.cpu` — throttling gera latência silenciosa, sem erro no log.
7. Manifesto versionado em Git com Helm ou Kustomize é, simultaneamente, gestão de configuração, auditoria e estratégia de recuperação de desastre — a ponte natural para GitOps, tema do próximo capítulo.

---

### 10.20 Exercícios (para casa)

#### Exercício 10.1 — Endurecimento com `securityContext` e PSA (fixação guiada) · 30 min

**Objetivo:** aplicar um `securityContext` restrito à `store-api` e comprovar que o namespace rejeita workloads privilegiados.

**Enunciado:** crie o namespace `store-seguro` com as três labels de Pod Security Admission no perfil `restricted` (`enforce`, `audit`, `warn`), fixando `enforce-version` na versão do seu cluster. Escreva `store-api-restrito.yaml` com um Deployment de 2 réplicas da `store-api` contendo, no mínimo: `runAsNonRoot: true`, `runAsUser: 65532`, `fsGroup: 65532`, `seccompProfile: RuntimeDefault` (nível Pod) e `allowPrivilegeEscalation: false`, `readOnlyRootFilesystem: true`, `capabilities.drop: ["ALL"]` (nível container), mais um `emptyDir` montado em `/tmp`. Em seguida escreva `pod-privilegiado.yaml` com `privileged: true` e tente aplicá-lo no mesmo namespace. Por fim, crie um segundo namespace `store-legado` com `enforce: baseline` e aplique nele o **mesmo** Pod privilegiado, registrando a diferença de comportamento.

**Entregável:** os três arquivos YAML; saída de `kubectl get ns store-seguro --show-labels`; saída completa do erro de `kubectl apply -f pod-privilegiado.yaml -n store-seguro`; saída de `kubectl get pods -n store-seguro -o wide`; saída de `kubectl exec -n store-seguro deploy/store-api -- id`; e um parágrafo comparando os resultados nos dois namespaces.

**Critério de aceite:** os 2 Pods da `store-api` em `Running 1/1` no namespace `restricted`; o `id` retornando `uid=65532`; a aplicação do Pod privilegiado rejeitada com `Error from server (Forbidden): ... violates PodSecurity "restricted"` listando pelo menos 4 violações; e o mesmo Pod também rejeitado (ou explicitamente aceito, com justificativa) no namespace `baseline`.

<details><summary>Dica</summary>
Se o Pod ficar em `CreateContainerConfigError` depois de ligar `readOnlyRootFilesystem`, o uvicorn está tentando escrever em `/tmp` — monte o `emptyDir`. Se o Deployment ficar `0/2` sem criar Pods, o erro está no ReplicaSet: `kubectl describe rs -n store-seguro`.
</details>

---

#### Exercício 10.2 — Scan de vulnerabilidades e correção da imagem base (aplicação) · 45 min

**Objetivo:** medir a superfície de vulnerabilidades da `store-api` e reduzi-la trocando a estratégia de build, com relatório antes/depois.

**Enunciado:** construa `store-api:vuln` a partir de um Dockerfile deliberadamente ruim — `FROM python:3.12` (imagem completa), `pip install` sem multi-stage, sem `USER` (roda como root), `COPY . .` sem `.dockerignore`. Escaneie com `trivy image --severity HIGH,CRITICAL` e salve o relatório em `relatorio-antes.txt`, anotando também o tamanho da imagem e o resultado de `docker run --rm store-api:vuln id`. Em seguida reescreva o Dockerfile com multi-stage e base distroless (`gcr.io/distroless/python3-debian12:nonroot`), atualize as dependências no `requirements.txt` para as versões corrigidas apontadas pelo scanner, construa `store-api:1.1` e gere `relatorio-depois.txt`. Gere também o SBOM em CycloneDX (`trivy image --format cyclonedx`). Finalize com uma tabela comparativa contendo: tamanho da imagem, nº de CRITICAL, nº de HIGH, número de pacotes no SBOM e UID do processo.

**Entregável:** os dois Dockerfiles, `.dockerignore`, `relatorio-antes.txt`, `relatorio-depois.txt`, `store-api-sbom.json`, a tabela comparativa em Markdown e o comando de CI com `--exit-code 1 --ignore-unfixed` que você usaria como porta de qualidade.

**Critério de aceite:** redução de pelo menos 80% no tamanho da imagem; zero CVEs CRITICAL **corrigíveis** no relatório depois; o processo rodando com UID != 0 na imagem nova; o SBOM válido e legível (`jq '.components | length' store-api-sbom.json` retorna um número > 0).

<details><summary>Dica</summary>
Rode `trivy image --scanners vuln --ignore-unfixed` para separar o que você consegue corrigir do que depende da distro. Se alguma wheel não instalar no distroless, instale no estágio builder dentro de um venv e copie o venv inteiro — como no Dockerfile da seção 10.4.2.
</details>

---

#### Exercício 10.3 — Depuração: aplicação matando conexões no rolling update (análise) · 45 min

**Objetivo:** reproduzir, diagnosticar e corrigir perda de requisições durante um rolling update.

**Enunciado:** você recebeu o seguinte chamado: *"toda vez que a equipe faz deploy da `store-api`, o monitoramento acusa um pico de 502 e alguns clientes veem carrinho vazio"*. Reproduza o cenário: aplique um Deployment da `store-api` com 3 réplicas **sem** `readinessProbe`, **sem** `lifecycle.preStop`, com `terminationGracePeriodSeconds: 5`, `strategy.rollingUpdate.maxUnavailable: 1` e com o `ENTRYPOINT` do Dockerfile em shell form (`ENTRYPOINT sh -c "uvicorn main:app --host 0.0.0.0 --port 8000"`). Suba um gerador de carga contínuo contra o Service e dispare um `kubectl set image` para uma nova tag, **contando** quantas requisições falharam. Em seguida, aplique as correções uma a uma, medindo a taxa de falha após cada uma: (a) adicionar `readinessProbe` em `/ready`; (b) adicionar `preStop` com `sleep 10` e `terminationGracePeriodSeconds: 45`; (c) trocar o `ENTRYPOINT` para forma exec; (d) definir `maxUnavailable: 0` e `maxSurge: 1`; (e) implementar o `lifespan` do FastAPI que faz `/ready` responder 503 no shutdown. Documente a taxa de falha em cada etapa.

**Entregável:** o YAML "quebrado" e o YAML final corrigido; o script/comando do gerador de carga; uma tabela com "correção aplicada × requisições falhadas em 100" para as 5 etapas; saída de `kubectl describe pod` de um Pod em `Terminating` mostrando o `preStop`; e um parágrafo explicando **por que** os Endpoints e o SIGTERM correm em paralelo.

**Critério de aceite:** a versão quebrada apresenta falhas mensuráveis (> 0) durante o rollout; a versão final apresenta **zero** falhas em pelo menos 500 requisições consecutivas durante um rollout completo; e a explicação identifica corretamente a corrida entre a atualização dos Endpoints/iptables e o envio do SIGTERM.

<details><summary>Dica</summary>
Conte falhas com `wget -q -O- --timeout=2 ... && echo OK || echo FALHA` num laço, canalizando para `sort | uniq -c`. Para provar que o shell form engole sinais, rode `kubectl exec -- ps -o pid,comm` e veja quem é o PID 1.
</details>

---

#### Exercício 10.4 — Desafio: checklist de produção + empacotamento (desafio integrador) · 60 min

**Objetivo:** aplicar o checklist de produção à FIAP Store completa e empacotar os manifestos com Helm ou Kustomize, com overlays de dev e prod.

**Enunciado:** consolide `store-api`, `store-front` e `store-worker` em um único pacote de deploy, **à sua escolha**:

- **Opção Kustomize:** `k8s/base/` com os manifestos comuns e `k8s/overlays/dev` e `k8s/overlays/prod` aplicando as diferenças.
- **Opção Helm:** `charts/fiap-store/` com `values-dev.yaml` e `values-prod.yaml`.

O overlay/values de **dev** deve ter: namespace `store-dev`, 1 réplica, `APP_ENV=dev`, PSA `baseline`, recursos pequenos, sem PDB. O de **prod**: namespace `store-prod`, 3+ réplicas, `APP_ENV=prod`, PSA `restricted`, `securityContext` completo, PDB, HPA, `preStop` + grace period, ServiceMonitor, NetworkPolicy default-deny, LimitRange e ResourceQuota. Aplique **os dois** ambientes no mesmo cluster e comprove que convivem isolados.

Em seguida, produza `CHECKLIST.md` percorrendo **todos** os itens da seção 10.14, marcando cada um como `[x] atendido`, `[ ] não atendido` ou `[-] não aplicável`, com uma linha de justificativa por item não atendido. Adicione um `Makefile` (ou script) com alvos `deploy-dev`, `deploy-prod`, `scan`, `diff` e `destroy`.

**Componente serverless (obrigatório):** escolha **um** dos itens do checklist que deixa de fazer sentido em AWS Lambda e **um** que continua exatamente igual, e explique em até 10 linhas por que a linha de responsabilidade compartilhada se move em um caso e não no outro. Como extra opcional, escreva o manifesto de um `Knative Service` equivalente à `store-api` e compare quantos itens do checklist ele elimina.

**Entregável:** a árvore completa de diretórios (`tree`), saída de `kubectl kustomize k8s/overlays/prod` ou `helm template`, saída de `kubectl get all,pdb,hpa,netpol,quota -n store-prod` e `-n store-dev`, o `CHECKLIST.md` preenchido, o `Makefile` e o texto da análise serverless.

**Critério de aceite:** um único comando (`make deploy-prod` ou `kubectl apply -k`) sobe o ambiente inteiro; os dois namespaces coexistem com configurações comprovadamente distintas (a saída de `kubectl get deploy -n store-dev` e `-n store-prod` difere em réplicas e imagem/tag); nenhum recurso em `store-prod` viola o PSA `restricted`; ao menos 80% dos itens do checklist marcados `[x]` ou `[-]` com justificativa; e a análise serverless identifica corretamente uma responsabilidade transferida e uma retida.

<details><summary>Dica</summary>
Em Kustomize, use `patches` com `target` por `kind`/`name` para não duplicar YAML, e `images:` no overlay para trocar a tag sem editar a base. Em Helm, `{{- if .Values.pdb.enabled }}` mantém o PDB fora de dev. Valide antes de aplicar com `kubectl apply -k k8s/overlays/prod --dry-run=server`.
</details>

---

### 10.21 Leitura recomendada

1. **[DOC]** Kubernetes — *Pod Security Standards* e *Pod Security Admission*. Disponível em: https://kubernetes.io/docs/concepts/security/pod-security-standards/ — a referência normativa dos três perfis; leia junto com https://kubernetes.io/docs/concepts/security/pod-security-admission/
2. **[DOC]** Kubernetes — *Configure a Security Context for a Pod or Container*. Disponível em: https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ — campo a campo, com exemplos aplicáveis.
3. **[DOC]** AWS — *EKS Best Practices Guide: Security*. Disponível em: https://docs.aws.amazon.com/eks/latest/best-practices/security.html — o guia mais completo e atualizado sobre endurecimento de EKS, incluindo IRSA e encryption at rest.
4. **[ARTIGO]** OWASP — *Kubernetes Top 10*. Disponível em: https://owasp.org/www-project-kubernetes-top-ten/ — a lista da seção 10.9 com detalhamento e cenários de ataque.
5. **[LIVRO]** RICE, Liz. *Container Security: Fundamental Technology Concepts that Protect Containerized Applications*. O'Reilly, 2020 — explica namespaces, cgroups, capabilities e seccomp no nível do kernel; leitura essencial para entender *por que* o `securityContext` funciona.
6. **[LIVRO]** ARUNDEL, John; DOMINGUS, Justin. *Cloud Native DevOps with Kubernetes*. 2. ed. O'Reilly, 2022 — capítulos sobre operação, observabilidade e gestão de configuração (Helm/Kustomize).

---

## Capítulo 11 — GitOps: entrega contínua declarativa com Argo CD e Rancher

> **Aula:** 2 horas · **Pré-requisitos:** capítulos 1 a 10 (Deployments, namespaces, RBAC, Secrets, Helm/Kustomize, boas práticas de operação); Git em nível intermediário (branch, PR, revert) · **Artefatos:** repositório `fiap-store-config`, Argo CD instalado no minikube, `Application` da FIAP Store com sync automático, overlays Kustomize dev/prod, ApplicationSet, workflow de CI no GitHub Actions

Este é o último capítulo da disciplina, e não é por acaso que ele vem depois de tudo. Nos capítulos anteriores aprendemos a **descrever** o estado desejado do sistema em YAML — Deployments, Services, HPA, NetworkPolicy, PDB, `securityContext`. Aprendemos também que esse YAML deve ser versionado, revisado e empacotado (capítulo 10). Falta responder à pergunta final: **quem aplica esse YAML no cluster, quando, com quais credenciais, e o que acontece quando alguém altera o cluster por fora?**

A resposta convencional — um pipeline de CI que roda `kubectl apply` — funciona, mas carrega problemas estruturais que só aparecem em escala. O GitOps propõe inverter a direção do fluxo, e essa inversão resolve, de uma vez, credenciais, drift, auditoria e recuperação de desastre. Vamos entender por quê, e depois colocar o Argo CD para funcionar.

---

### 11.1 Objetivos de aprendizagem

Ao final desta aula, você deverá ser capaz de:

1. **Identificar** os quatro princípios da OpenGitOps e **distinguir** o modelo push (CIOps) do modelo pull (GitOps), argumentando sobre credenciais, drift e auditabilidade.
2. **Estruturar** repositórios de configuração (app repo x config repo, monorepo x polirepo) e **projetar** um fluxo de promoção entre ambientes baseado em pull request.
3. **Instalar** o Argo CD em um cluster e **construir** objetos `Application` com `syncPolicy` automatizada (`prune`, `selfHeal`), `syncOptions` e `ignoreDifferences`.
4. **Aplicar** os padrões App-of-Apps e `ApplicationSet` para gerenciar múltiplos ambientes, e **configurar** `AppProject` com RBAC do Argo CD.
5. **Diagnosticar** os estados `OutOfSync`, `Degraded` e `Unknown`, corrigindo drift manual, falha de sync por Secret ausente e falha por permissão.
6. **Avaliar** as estratégias de gestão de segredos em GitOps (Sealed Secrets, External Secrets, SOPS) e **comparar** Argo CD, Flux e Rancher Fleet para diferentes cenários organizacionais.

---

### 11.2 Roteiro da aula (120 min)

| Bloco | Duração | Atividade |
|---|---|---|
| 1 | 10 min | O problema do `kubectl apply` manual e do pipeline com credencial de cluster. |
| 2 | 15 min | GitOps: os 4 princípios da OpenGitOps; push x pull; drift e reconciliação. |
| 3 | 15 min | Estrutura de repositórios e promoção entre ambientes por PR. |
| 4 | 10 min | **Intervalo** |
| 5 | 25 min | Argo CD: arquitetura, `Application`, sync policy, waves e hooks, App-of-Apps, ApplicationSet, Projects. |
| 6 | 10 min | Segredos em GitOps (Sealed Secrets, External Secrets, SOPS); Argo Rollouts; Flux em comparação. |
| 7 | 10 min | Rancher: arquitetura, multi-cluster, Fleet, quando escolher o quê. Integração com CI. |
| 8 | 5 min | Ponte com Serverless: GitOps para funções, Knative Service no Argo, "tudo é manifesto". |
| 9 | 20 min | **Laboratório guiado:** Argo CD no minikube, Application da FIAP Store, drift, selfHeal, rollback. |
| — | 120 min | **Total** |

---

### 11.3 O problema: como o YAML chega ao cluster?

Ao longo do semestre usamos `kubectl apply -f`. Funciona para aprender; é insustentável para operar. Pense em quatro perguntas que uma empresa precisa responder sobre seu ambiente de produção:

1. **O que exatamente está rodando em produção agora?** Se a resposta depende de alguém rodar `kubectl get`, você não tem inventário — tem arqueologia.
2. **Quem colocou isso lá e por quê?** Um `kubectl apply` de um notebook não deixa rastro além de uma linha de audit log sem contexto de negócio.
3. **Como volto ao estado de ontem às 14h?** Sem um registro do estado desejado, "voltar" significa reconstruir de memória.
4. **Alguém alterou algo manualmente?** É o **drift** — a diferença silenciosa entre o que está no repositório e o que está no cluster. Ele nasce de um `kubectl edit` numa madrugada de incidente e permanece até o próximo deploy o sobrescrever (ou, pior, até o próximo deploy ser sobrescrito por ele).

A evolução natural foi automatizar com CI. O pipeline (Jenkins, GitHub Actions, GitLab CI) constrói a imagem, faz push, e então executa `kubectl apply` contra o cluster. Isso se chama **CIOps** ou **modelo push**, e resolve as duas primeiras perguntas — mas cria outras.

---

### 11.4 GitOps: definição e os 4 princípios

O termo foi cunhado pela Weaveworks em 2017 e depois formalizado pelo grupo **OpenGitOps** (CNCF), que definiu quatro princípios. Decore-os; eles são o critério objetivo para dizer se um processo é ou não GitOps.

> **1. Declarativo.** O sistema é descrito de forma declarativa — o estado desejado é um conjunto de fatos ("quero 3 réplicas"), não uma sequência de comandos ("escale para 3").
>
> **2. Versionado e imutável.** O estado desejado é armazenado de forma que force imutabilidade, versionamento e retenção do histórico completo. Na prática: Git, com commits assinados e histórico preservado.
>
> **3. Puxado automaticamente.** Agentes de software puxam automaticamente as declarações de estado desejado a partir da fonte da verdade. O cluster busca; ninguém empurra para dentro dele.
>
> **4. Reconciliado continuamente.** Agentes observam continuamente o estado real e tentam aplicar o estado desejado. Não é "aplicar no deploy": é um laço de controle permanente.

Note que o quarto princípio é exatamente o **reconciliation loop** que estudamos no capítulo 1, aplicado um nível acima. O Deployment controller reconcilia "3 réplicas desejadas × 2 réplicas reais". O Argo CD reconcilia "repositório Git × cluster inteiro". É o mesmo padrão arquitetural, e é por isso que GitOps encaixa tão naturalmente no Kubernetes: o Kubernetes já é uma máquina de reconciliação.

```
                    MODELO PUSH (CIOps)
  +--------+  push   +-------------+  kubectl apply   +-----------+
  |  Git   |-------->|  CI Runner  |----------------->|  Cluster  |
  +--------+         | (tem creds  |    (de fora      +-----------+
                     |  do cluster)|     para dentro)
                     +-------------+
   * credencial do cluster mora no CI  * cluster exposto ao CI
   * drift invisível                   * nao ha reconciliacao

                    MODELO PULL (GitOps)
  +--------+       +-------------+          +--------------------+
  |  Git   |<------|  CI Runner  |          |      Cluster       |
  +--------+  PR   | (so build   |          |  +--------------+  |
      ^            |  e push da  |          |  |  Agente Argo |  |
      |            |  imagem)    |          |  +------+-------+  |
      |            +-------------+          +---------|----------+
      |                                               |
      +-------------- pull (a cada 3 min) ------------+
   * nenhuma credencial de cluster fora do cluster
   * drift detectado e corrigido automaticamente
   * Git e a fonte unica da verdade e o log de auditoria
```

#### 11.4.1 Push x pull: a comparação que importa

| Aspecto | **CIOps / push** | **GitOps / pull** |
|---|---|---|
| Onde mora a credencial do cluster | no CI (secret do pipeline) | **em lugar nenhum fora do cluster** |
| Direção da conexão | CI → cluster (de fora para dentro) | agente → Git (de dentro para fora) |
| Firewall / cluster privado | exige expor o API server ao CI | funciona com cluster totalmente privado |
| Detecção de drift | inexistente | contínua |
| Correção de drift | manual, no próximo deploy | automática (`selfHeal`) |
| Fonte da verdade | o último `apply` que rodou | o repositório Git |
| Auditoria | logs do CI, efêmeros | histórico do Git, permanente e assinável |
| Rollback | rodar o pipeline antigo (se ainda existir) | `git revert` |
| Superfície de ataque | comprometer o CI = comprometer todos os clusters | comprometer o CI = poder publicar imagem, apenas |
| Multi-cluster | N credenciais no CI | N agentes puxando do mesmo repo |

> **Atenção:** o item mais subestimado dessa tabela é o primeiro. Em CIOps, o runner de CI precisa de credenciais com poder de escrita em produção. Runners executam código de pull requests. Já houve incidentes públicos graves em que um PR malicioso exfiltrou o `KUBECONFIG` do ambiente de CI. Em GitOps, esse vetor simplesmente não existe: o pior que um PR malicioso faz é ser rejeitado no code review.

> **Nota:** GitOps **não elimina** o CI. Ele redivide as responsabilidades: o CI continua responsável por construir, testar, escanear e publicar a imagem — e por abrir um PR no repositório de configuração. O CD (aplicar no cluster) passa para o agente. É "CI empurra artefato, CD puxa configuração".

---

### 11.5 Estrutura de repositórios

Esta é a decisão de arquitetura que mais gera discussão em equipe, e a que mais impacta o dia a dia.

#### 11.5.1 App repo x config repo

A recomendação consolidada é **separar o código-fonte da aplicação da configuração de deploy**:

```
fiap-store-api/          <-- APP REPO
├── main.py
├── requirements.txt
├── Dockerfile
├── tests/
└── .github/workflows/ci.yaml     (build, test, scan, push, bump no config repo)

fiap-store-config/       <-- CONFIG REPO (o que o Argo observa)
├── base/
├── overlays/
└── apps/
```

Por que separar? Três razões concretas:

1. **Evitar o laço infinito.** Se o CI comita a nova tag no mesmo repositório que ele observa, o commit dispara um novo build, que gera novo commit, que dispara novo build. Dá para contornar com `[skip ci]`, mas é frágil.
2. **Permissões distintas.** Desenvolvedores mexem em código o dia todo. Alterar o que roda em produção deveria exigir uma revisão diferente, possivelmente de pessoas diferentes (CODEOWNERS no config repo).
3. **Cadência distinta.** Um mesmo config repo serve N aplicações; uma mudança de política (adicionar `seccompProfile` a todos os workloads) não deveria exigir tocar em N repositórios de código.

#### 11.5.2 Monorepo x polirepo (de configuração)

| | **Monorepo de config** | **Polirepo (um por app/time)** |
|---|---|---|
| Visibilidade do todo | excelente | fragmentada |
| Mudança transversal (política) | um PR | N PRs |
| Blast radius de um erro | maior | menor |
| RBAC granular por time | difícil (CODEOWNERS ajuda) | natural |
| Escala | ótimo até ~dezenas de apps | melhor com centenas de apps e times |
| Complexidade do Argo | menor (um repo, muitos paths) | maior (muitas credenciais de repo) |

Para a FIAP Store e para a maioria das empresas de porte médio, **monorepo de configuração com pastas por ambiente** é a escolha certa.

#### 11.5.3 Pastas por ambiente e promoção por PR

Existem duas escolas para separar ambientes: **branches por ambiente** (`dev`, `staging`, `main`) e **pastas por ambiente** na mesma branch. A comunidade convergiu fortemente para **pastas**, e a razão é dolorosa de aprender na prática: com branches, você acaba fazendo merge de `dev` para `main`, o que arrasta **todas** as diferenças de configuração de dev (réplicas menores, recursos menores, `APP_ENV=dev`) para produção. Cherry-pick vira rotina, conflitos viram rotina, e ninguém consegue dizer com certeza o que difere entre os ambientes.

```
fiap-store-config/
├── apps/                          # App-of-Apps: Applications do Argo
│   ├── store-dev.yaml
│   └── store-prod.yaml
├── base/                          # manifestos comuns (Kustomize)
│   ├── kustomization.yaml
│   ├── store-api-deployment.yaml
│   ├── store-api-service.yaml
│   ├── store-front-deployment.yaml
│   └── store-front-service.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml     # namespace store-dev, 1 replica, tag 1.1
│   │   └── patch-recursos.yaml
│   └── prod/
│       ├── kustomization.yaml     # namespace store-prod, 4 replicas, tag 1.0
│       ├── patch-recursos.yaml
│       ├── pdb.yaml
│       └── hpa.yaml
└── infra/                         # add-ons de cluster
    ├── kyverno-policies/
    └── monitoring/
```

Com essa estrutura, **promover uma versão de dev para prod é literalmente um pull request de uma linha**:

```diff
  # overlays/prod/kustomization.yaml
  images:
    - name: docker.io/fabiobara/store-api
-     newTag: "1.0"
+     newTag: "1.1"
```

Esse diff é revisável, aprovável, auditável e reversível. É o artefato de governança mais barato que existe: substitui formulário de mudança, e-mail de aprovação e planilha de release por um objeto que o Git já sabe versionar.

> **Na prática:** proteja a branch `main` do config repo (require PR, require review, require status checks). Use `CODEOWNERS` para exigir aprovação do time de plataforma em `overlays/prod/` e `infra/`, enquanto `overlays/dev/` pode ser aprovado pelo próprio time da aplicação.

---

### 11.6 Argo CD

O **Argo CD** é o controlador GitOps mais adotado do ecossistema Kubernetes (projeto graduado da CNCF, junto com o restante do Argo Project). Ele é, essencialmente, um controlador que compara "o que está no Git" com "o que está no cluster" e age sobre a diferença.

#### 11.6.1 Arquitetura

```
   +---------------------------------------------------------+
   |                     Namespace argocd                     |
   |                                                          |
   |  +----------------+      +---------------------------+   |
   |  |   API Server   |<---->|          Redis            |   |
   |  |  (gRPC/REST)   |      |  (cache de manifestos     |   |
   |  |  UI + CLI      |      |   renderizados e estado)  |   |
   |  +-------+--------+      +-------------+-------------+   |
   |          |                             ^                 |
   |          v                             |                 |
   |  +----------------+      +-------------+-------------+   |
   |  |      Dex       |      |       Repo Server         |   |
   |  | (SSO: OIDC,    |      | clona o Git e RENDERIZA   |---+---> Git
   |  |  LDAP, GitHub) |      | (helm template/kustomize) |   |
   |  +----------------+      +-------------+-------------+   |
   |                                        |                 |
   |                          +-------------v-------------+   |
   |                          |  Application Controller   |   |
   |                          |  compara desejado x real  |---+---> API server
   |                          |  e executa o sync         |   |     (deste ou de
   |                          +---------------------------+   |      outros clusters)
   +---------------------------------------------------------+
```

| Componente | Responsabilidade | Detalhe operacional |
|---|---|---|
| **API Server** | expõe gRPC/REST para a UI, o CLI e webhooks; faz autenticação e autorização | é o único componente que precisa ser exposto para humanos |
| **Repo Server** | clona os repositórios e **renderiza** os manifestos (`helm template`, `kustomize build`) | é o gargalo de CPU/memória em instalações grandes; escale horizontalmente |
| **Application Controller** | o coração: compara estado desejado x real, aplica, e avalia health | roda o loop de reconciliação (padrão: a cada 3 min) |
| **Redis** | cache de manifestos renderizados e de estado | é **cache**, não banco: perder o Redis causa lentidão, não perda de dados |
| **Dex** | provedor de identidade embutido para SSO (GitHub, Google, LDAP, SAML) | opcional se você já tem um OIDC próprio |
| **ApplicationSet Controller** | gera `Application`s a partir de geradores | instalado junto desde a versão 2.3 |
| **Notifications Controller** | envia alertas de eventos de sync (Slack, e-mail, webhook) | opcional |

> **Nota:** o estado do Argo CD vive em **CRDs no etcd do cluster** (`Application`, `AppProject`, `ApplicationSet`), não em um banco externo. Isso significa que o Argo CD é ele próprio recuperável por GitOps — se você perder a instalação, reinstalá-la e reaplicar os `Application`s reconstrói tudo.

#### 11.6.2 Instalação e primeiro acesso

```bash
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.13.3/manifests/install.yaml
kubectl wait --for=condition=available --timeout=300s \
  deploy/argocd-server -n argocd
kubectl get pods -n argocd
```

```
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          92s
argocd-applicationset-controller-7d9c4b8f5-h2xkq    1/1     Running   0          92s
argocd-dex-server-6f8d9b7c4-pn4vz                   1/1     Running   0          92s
argocd-notifications-controller-5c7b9d84f-t8wlr     1/1     Running   0          92s
argocd-redis-6b8f7c9d5-k3mnx                        1/1     Running   0          92s
argocd-repo-server-8c9d7f6b4-r5qpt                  1/1     Running   0          92s
argocd-server-79b4c8d6f-w9zjx                       1/1     Running   0          92s
```

Senha inicial e acesso:

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
# UI em https://localhost:8080  (usuario: admin)
```

CLI:

```bash
curl -sSL -o /usr/local/bin/argocd \
  https://github.com/argoproj/argo-cd/releases/download/v2.13.3/argocd-linux-amd64
chmod +x /usr/local/bin/argocd

argocd login localhost:8080 --username admin --password '<senha>' --insecure
argocd account update-password
kubectl -n argocd delete secret argocd-initial-admin-secret   # depois de trocar
```

#### 11.6.3 O objeto `Application`

É a unidade central: "este caminho deste repositório deve estar aplicado neste cluster/namespace".

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: store-prod
  namespace: argocd
  finalizers:
    # sem isso, deletar a Application deixa os recursos orfaos no cluster
    - resources-finalizer.argocd.argoproj.io
  labels:
    env: prod
spec:
  project: fiap-store

  source:
    repoURL: https://github.com/fabiobara/fiap-store-config.git
    targetRevision: main          # branch, tag ou SHA de commit
    path: overlays/prod
    kustomize:
      images:
        - docker.io/fabiobara/store-api:1.1

  destination:
    server: https://kubernetes.default.svc   # ou 'name: cluster-prod'
    namespace: store-prod

  syncPolicy:
    automated:
      prune: true        # deleta do cluster o que sumiu do Git
      selfHeal: true     # desfaz alteracoes manuais no cluster
      allowEmpty: false  # nao permite sync que resulte em zero recursos
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
      - ServerSideApply=true
      - RespectIgnoreDifferences=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m

  # campos alterados por outros controladores nao devem gerar OutOfSync
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas          # o HPA gerencia isto
    - group: ""
      kind: Secret
      name: store-db-credentials
      jsonPointers:
        - /data
```

Vamos destrinchar os campos que causam mais dúvida:

**`syncPolicy.automated.prune`.** Sem `prune`, remover um manifesto do Git **não** remove o recurso do cluster — ele fica órfão para sempre. Com `prune: true`, o Git é realmente a fonte da verdade nos dois sentidos. É assustador na primeira vez; é correto.

**`syncPolicy.automated.selfHeal`.** Este é o quarto princípio da OpenGitOps em um booleano. Com `selfHeal: true`, se alguém rodar `kubectl scale deploy/store-api --replicas=10`, o Argo detecta o drift no próximo ciclo e devolve o valor do Git. Sem `selfHeal`, a Application apenas fica marcada `OutOfSync` e espera intervenção.

**`PruneLast=true`.** Faz as remoções acontecerem no fim do sync, depois de tudo estar saudável. Evita a janela em que o recurso novo ainda não subiu e o antigo já foi removido.

**`ServerSideApply=true`.** Usa Server-Side Apply do Kubernetes, que gerencia ownership de campos no servidor. Resolve dois problemas clássicos: CRDs grandes que estouram o limite de 262 kB da anotação `last-applied-configuration`, e conflitos com outros controladores que escrevem no mesmo objeto.

**`ignoreDifferences`.** Indispensável no mundo real. Se um HPA gerencia `spec.replicas` e o Git diz `replicas: 3`, o Argo ficará eternamente `OutOfSync` — e, com `selfHeal`, entrará em guerra com o HPA, escalando para baixo enquanto o HPA escala para cima. O ideal é remover `replicas` do manifesto **e** ignorá-lo aqui.

> **Atenção:** o `finalizer` `resources-finalizer.argocd.argoproj.io` muda o comportamento de `kubectl delete application`. **Com** o finalizer, deletar a Application faz cascata e remove os recursos gerenciados. **Sem** ele, a Application some e os recursos ficam órfãos no cluster. Saiba qual dos dois você quer antes de deletar algo em produção.

Comandos de operação:

```bash
argocd app create store-prod \
  --repo https://github.com/fabiobara/fiap-store-config.git \
  --path overlays/prod --dest-namespace store-prod \
  --dest-server https://kubernetes.default.svc \
  --sync-policy automated --auto-prune --self-heal

argocd app list
argocd app get store-prod
argocd app diff store-prod          # diff entre Git e cluster
argocd app sync store-prod
argocd app history store-prod
argocd app rollback store-prod 12   # rollback pela revisao do Argo
argocd app set store-prod --sync-policy none   # desliga o automated
```

```
$ argocd app get store-prod
Name:               argocd/store-prod
Project:            fiap-store
Server:             https://kubernetes.default.svc
Namespace:          store-prod
Repo:               https://github.com/fabiobara/fiap-store-config.git
Target:             main
Path:               overlays/prod
SyncWindow:         Sync Allowed
Sync Policy:        Automated (Prune, SelfHeal)
Sync Status:        Synced to main (a3f9c21)
Health Status:      Healthy

GROUP  KIND        NAMESPACE   NAME         STATUS  HEALTH   HOOK  MESSAGE
       Namespace               store-prod   Synced
       Service     store-prod  store-api    Synced  Healthy        service/store-api created
apps   Deployment  store-prod  store-api    Synced  Healthy        deployment.apps/store-api configured
policy PodDisrup.. store-prod  store-api-pdb Synced Healthy
```

#### 11.6.4 Sync status x Health status

São duas dimensões independentes, e confundi-las é o erro nº 1 de quem começa:

| **Sync Status** | Significado |
|---|---|
| `Synced` | o cluster corresponde ao que está no Git |
| `OutOfSync` | há diferença entre Git e cluster (por drift ou por commit novo ainda não aplicado) |
| `Unknown` | o Argo não conseguiu determinar (repo inacessível, erro de renderização) |

| **Health Status** | Significado |
|---|---|
| `Healthy` | os recursos estão funcionando (ex.: Deployment com réplicas prontas) |
| `Progressing` | em transição (rollout em andamento) |
| `Degraded` | falhou (Pods em CrashLoopBackOff, rollout estourou `progressDeadlineSeconds`) |
| `Suspended` | pausado (CronJob suspenso, rollout pausado) |
| `Missing` | o recurso está no Git mas não existe no cluster |

**As combinações que importam:** `Synced` + `Degraded` significa "o Argo aplicou exatamente o que o Git pediu, e o que o Git pediu está quebrado" — o problema é seu, não do Argo. `OutOfSync` + `Healthy` significa "está rodando bem, mas não é o que o Git diz" — drift, ou sync pendente.

Health de recursos customizados pode ser ensinado ao Argo com Lua, no ConfigMap `argocd-cm`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-cm
  namespace: argocd
data:
  resource.customizations.health.example.com_MinhaCRD: |
    hs = {}
    if obj.status ~= nil and obj.status.fase == "Pronto" then
      hs.status = "Healthy"
      hs.message = "recurso pronto"
      return hs
    end
    hs.status = "Progressing"
    return hs
```

#### 11.6.5 Sync waves e hooks

Nem tudo pode ser aplicado ao mesmo tempo. Migração de banco precisa rodar antes do novo código; CRDs precisam existir antes dos recursos que os usam. O Argo resolve com **waves** (ordenação) e **hooks** (momento no ciclo).

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: store-db-migration
  namespace: store-prod
  annotations:
    argocd.argoproj.io/hook: PreSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
    argocd.argoproj.io/sync-wave: "-1"
spec:
  backoffLimit: 2
  template:
    metadata:
      labels:
        app: store-db-migration
    spec:
      restartPolicy: Never
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: migration
          image: docker.io/fabiobara/store-api:1.1
          command: ["python", "-m", "alembic", "upgrade", "head"]
          env:
            - name: DB_HOST
              value: store-db.store-prod.svc.cluster.local
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: store-db-credentials
                  key: password
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
```

| Hook | Quando executa | Uso típico |
|---|---|---|
| `PreSync` | antes de aplicar os manifestos | migração de banco, validação de pré-requisitos |
| `Sync` | junto com os manifestos | o padrão implícito |
| `PostSync` | depois de tudo estar `Healthy` | smoke test, notificação, aquecimento de cache |
| `SyncFail` | quando o sync falha | rollback customizado, alerta de plantão |
| `Skip` | nunca (o Argo ignora o recurso) | manifesto gerenciado por outro processo |

**Waves** ordenam dentro de cada fase: `sync-wave: "-1"` vem antes de `"0"` (padrão), que vem antes de `"1"`. O Argo só avança para a próxima wave quando os recursos da atual estiverem `Healthy`. Ordem típica: `-2` namespaces e CRDs, `-1` migrações, `0` ConfigMaps/Secrets/Deployments, `1` Services e Ingress, `2` smoke tests.

#### 11.6.6 App-of-Apps e ApplicationSet

Com 3 aplicações, criar `Application`s à mão funciona. Com 30 aplicações × 3 ambientes, não.

**App-of-Apps** é o padrão em que uma `Application` "raiz" aponta para uma pasta que contém... outras `Application`s. O Argo aplica as filhas, que por sua vez sincronizam as aplicações reais. É recursivo e elegante: **o bootstrap do cluster inteiro cabe em um único `kubectl apply`**.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fiap-store-root
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/fabiobara/fiap-store-config.git
    targetRevision: main
    path: apps            # pasta com store-dev.yaml, store-prod.yaml, infra.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**ApplicationSet** vai além: em vez de manter um arquivo por Application, você declara um **template** e um **gerador**, e o controlador produz as Applications. Geradores disponíveis: `list`, `git` (por diretório ou por arquivo), `cluster`, `matrix`, `merge`, `scmProvider`, `pullRequest`.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: fiap-store-ambientes
  namespace: argocd
spec:
  goTemplate: true
  goTemplateOptions: ["missingkey=error"]
  generators:
    - list:
        elements:
          - ambiente: dev
            namespace: store-dev
            projeto: fiap-store
            autoSync: "true"
          - ambiente: prod
            namespace: store-prod
            projeto: fiap-store
            autoSync: "true"
  template:
    metadata:
      name: 'fiap-store-{{.ambiente}}'
      labels:
        env: '{{.ambiente}}'
    spec:
      project: '{{.projeto}}'
      source:
        repoURL: https://github.com/fabiobara/fiap-store-config.git
        targetRevision: main
        path: 'overlays/{{.ambiente}}'
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{.namespace}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

Variante com o gerador `git` por diretório — adicionar uma pasta em `overlays/` cria a Application sozinha:

```yaml
  generators:
    - git:
        repoURL: https://github.com/fabiobara/fiap-store-config.git
        revision: main
        directories:
          - path: overlays/*
```

E o gerador `cluster`, para implantar a mesma aplicação em todos os clusters registrados:

```yaml
  generators:
    - clusters:
        selector:
          matchLabels:
            ambiente: producao
```

> **Na prática:** use App-of-Apps quando cada aplicação tem configuração bem diferente (é mais explícito e mais fácil de revisar). Use ApplicationSet quando o padrão se repete (mesma app em N ambientes ou N clusters). Os dois se combinam: uma Application raiz que aponta para uma pasta contendo ApplicationSets.

#### 11.6.7 Projects e RBAC do Argo CD

O `AppProject` é a fronteira de multi-tenancy do Argo CD. Ele restringe **de onde** (repositórios), **para onde** (clusters/namespaces) e **o quê** (kinds) uma Application pode implantar.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: fiap-store
  namespace: argocd
spec:
  description: Aplicacoes do e-commerce FIAP Store

  sourceRepos:
    - https://github.com/fabiobara/fiap-store-config.git

  destinations:
    - server: https://kubernetes.default.svc
      namespace: store-dev
    - server: https://kubernetes.default.svc
      namespace: store-prod

  # recursos de escopo de cluster que este projeto pode criar
  clusterResourceWhitelist:
    - group: ""
      kind: Namespace

  # nada de ClusterRoleBinding vindo deste projeto
  clusterResourceBlacklist:
    - group: rbac.authorization.k8s.io
      kind: ClusterRoleBinding

  namespaceResourceWhitelist:
    - group: "*"
      kind: "*"

  roles:
    - name: desenvolvedor
      description: pode ver tudo e sincronizar apenas dev
      policies:
        - p, proj:fiap-store:desenvolvedor, applications, get, fiap-store/*, allow
        - p, proj:fiap-store:desenvolvedor, applications, sync, fiap-store/fiap-store-dev, allow
      groups:
        - fiap:time-loja

  # janela de mudanca: nada de sync em prod na sexta a noite
  syncWindows:
    - kind: deny
      schedule: "0 18 * * 5"
      duration: 60h
      applications:
        - fiap-store-prod
      manualSync: true
```

O RBAC global fica no ConfigMap `argocd-rbac-cm`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-rbac-cm
  namespace: argocd
data:
  policy.default: role:readonly
  policy.csv: |
    p, role:plataforma, applications, *, */*, allow
    p, role:plataforma, clusters, get, *, allow
    p, role:plataforma, repositories, *, *, allow

    p, role:dev-loja, applications, get, fiap-store/*, allow
    p, role:dev-loja, applications, sync, fiap-store/fiap-store-dev, allow
    p, role:dev-loja, applications, action/*, fiap-store/fiap-store-dev, allow
    # nao pode dar sync em prod: a promocao acontece por PR no Git

    g, fiap:plataforma, role:plataforma
    g, fiap:time-loja, role:dev-loja
  scopes: '[groups, email]'
```

> **Nota:** repare na filosofia. O desenvolvedor **não** ganha permissão de sync em produção — não porque não seja confiável, mas porque em GitOps a forma correta de mudar produção é **abrir um PR**, não clicar em "Sync". O botão é a exceção (incidente), o PR é a regra.

#### 11.6.8 Rollback: o Git é o botão de desfazer

Há dois caminhos, e a diferença conceitual é importante:

```bash
# Caminho 1 (correto no dia a dia): reverter no Git
git revert a3f9c21
git push
# o Argo detecta o novo commit e reconcilia sozinho

# Caminho 2 (emergencia): rollback pela API do Argo
argocd app rollback store-prod 12
```

O caminho 2 coloca a Application em estado `OutOfSync` em relação à branch e **desliga o auto-sync** (senão o Argo desfaria o rollback imediatamente no próximo ciclo). É uma medida de contenção: use para parar o sangramento, e em seguida faça o `git revert` para reconciliar a verdade. Se você não fizer isso, o próximo commit qualquer no repositório trará o problema de volta.

---

### 11.7 Segredos em GitOps

Aqui está o atrito fundamental do modelo: **GitOps quer tudo no Git; Secrets não podem ir para o Git**. Um `Secret` do Kubernetes é base64, que é codificação e não criptografia — commitar um é o mesmo que commitar a senha em texto claro, com o agravante de que o histórico do Git é imutável e a senha fica lá para sempre, mesmo depois de "removida".

Três soluções maduras:

| | **Sealed Secrets** (Bitnami) | **External Secrets Operator** | **SOPS** (Mozilla) + KSOPS/age |
|---|---|---|---|
| Onde vive o segredo | criptografado no Git | em um cofre externo (AWS Secrets Manager, Vault) | criptografado no Git |
| O que vai para o Git | `SealedSecret` (cifrado com chave pública do cluster) | `ExternalSecret` (apenas uma **referência**) | YAML com valores cifrados (chaves em claro) |
| Quem decifra | o controller no cluster (chave privada) | o operator, autenticado no cofre | plugin no repo server do Argo |
| Rotação de segredo | exige recriptografar e commitar | automática (`refreshInterval`) | exige recriptografar e commitar |
| Recuperação de desastre | precisa fazer backup da chave privada do controller | trivial (o segredo está no cofre) | precisa da chave KMS/age |
| Multi-cluster | ruim (cada cluster tem chave própria) | excelente | bom |
| Complexidade | baixa | média | média |

**Sealed Secrets** — bom para aprender e para cluster único:

```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.27.1/controller.yaml

kubectl create secret generic store-db-credentials \
  --from-literal=password='S3nh4F0rt3!' \
  -n store-prod \
  --dry-run=client -o yaml \
  | kubeseal --format yaml --controller-namespace kube-system \
  > sealed-store-db.yaml     # ESTE arquivo pode ir para o Git

kubectl apply -f sealed-store-db.yaml
kubectl get secret store-db-credentials -n store-prod   # criado pelo controller
```

**External Secrets Operator** — a escolha para AWS/produção, e a que melhor se integra com IRSA (capítulo 9):

```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: aws-secrets
  namespace: store-prod
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        jwt:
          serviceAccountRef:
            name: external-secrets-sa   # com IRSA anexada
---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: store-db-credentials
  namespace: store-prod
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets
    kind: SecretStore
  target:
    name: store-db-credentials     # Secret que sera criado no cluster
    creationPolicy: Owner
  data:
    - secretKey: password
      remoteRef:
        key: fiap-store/prod/db
        property: password
```

O `ExternalSecret` acima é **seguro no Git**: ele contém apenas o endereço do segredo, nunca o valor.

> **Atenção:** se um segredo já foi commitado em texto claro, **considere-o comprometido e rotacione imediatamente**. Reescrever o histórico com `git filter-repo` remove do repositório, mas não dos forks, clones locais, caches do GitHub e backups. Rotacionar é a única correção real.

---

### 11.8 Argo Rollouts: canary e blue-green declarativos

O `Deployment` nativo só oferece `RollingUpdate` e `Recreate`. Canary de verdade (com análise automática de métricas e rollback autônomo) exige o **Argo Rollouts**, que substitui o `Deployment` por um CRD `Rollout` de mesma estrutura, acrescido de estratégia.

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: store-api
  namespace: store-prod
spec:
  replicas: 6
  revisionHistoryLimit: 3
  selector:
    matchLabels:
      app: store-api
  strategy:
    canary:
      canaryService: store-api-canary
      stableService: store-api-stable
      trafficRouting:
        nginx:
          stableIngress: store-api-ingress
      analysis:
        templates:
          - templateName: taxa-de-sucesso
        startingStep: 2
      steps:
        - setWeight: 10
        - pause: { duration: 2m }
        - setWeight: 30
        - pause: { duration: 5m }
        - setWeight: 60
        - pause: { duration: 5m }
        - setWeight: 100
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: store-api
          image: docker.io/fabiobara/store-api:1.2
          ports:
            - containerPort: 8000
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits:   { memory: 256Mi }
---
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: taxa-de-sucesso
  namespace: store-prod
spec:
  metrics:
    - name: taxa-sucesso
      interval: 30s
      count: 5
      successCondition: result[0] >= 0.99
      failureLimit: 1
      provider:
        prometheus:
          address: http://kube-prom-prometheus.monitoring:9090
          query: |
            sum(rate(http_requests_total{app="store-api",status!~"5.."}[2m]))
            /
            sum(rate(http_requests_total{app="store-api"}[2m]))
```

Se a taxa de sucesso do canário cair abaixo de 99%, o Rollouts **aborta e volta para a versão estável sozinho**, sem intervenção humana e sem alterar o Git. Blue-green usa `strategy.blueGreen` com `activeService`, `previewService` e `autoPromotionEnabled: false` para exigir promoção manual.

```bash
kubectl argo rollouts get rollout store-api -n store-prod --watch
kubectl argo rollouts promote store-api -n store-prod
kubectl argo rollouts abort store-api -n store-prod
```

---

### 11.9 Flux CD em comparação breve

O **Flux CD** é o outro projeto GitOps graduado da CNCF. Filosoficamente, ele é mais "Kubernetes puro": não tem UI própria, é um conjunto de controladores especializados (`source-controller`, `kustomize-controller`, `helm-controller`, `notification-controller`, `image-automation-controller`) e tudo se opera por CRDs e `kubectl`.

| | **Argo CD** | **Flux CD** |
|---|---|---|
| Interface | UI web rica, muito boa para diagnóstico e para times grandes | sem UI oficial (Weave GitOps é de terceiros) |
| Modelo | monolítico, multi-tenant, um Argo para N clusters | modular, tipicamente um Flux por cluster |
| Curva inicial | mais suave (a UI ensina) | mais íngreme (só CRDs) |
| Multi-cluster | nativo (registra clusters remotos) | via hub-and-spoke ou um Flux por cluster |
| Atualização automática de imagem | não nativa (usa Image Updater, um add-on) | **nativa** (`ImagePolicy` + `ImageUpdateAutomation`) |
| Helm | renderiza com `helm template` | usa o SDK do Helm (releases reais) |
| Consumo de recursos | maior | menor |
| Melhor para | organizações com muitos times e necessidade de visibilidade | equipes de plataforma que preferem tudo por CRD e menor footprint |

Ambos são excelentes e implementam os mesmos quatro princípios. A escolha é cultural: **se o time quer olhar uma tela, Argo; se o time quer só `kubectl` e YAML, Flux.**

---

### 11.10 Rancher

O **Rancher** (SUSE) resolve um problema adjacente e maior: não "como aplico manifestos", mas "**como gerencio uma frota de clusters**" — alguns em nuvem, alguns on-premises, alguns na borda.

#### 11.10.1 Arquitetura

```
        +------------------------------------------+
        |            Rancher Server                |
        |  (roda ele mesmo em um cluster K8s)      |
        |  UI unificada · autenticacao central     |
        |  catalogo de apps · Fleet · RBAC global  |
        +----+---------------+---------------+-----+
             |               |               |
     (tunel iniciado pelo agente, de dentro para fora)
             |               |               |
   +---------v----+  +-------v------+  +-----v--------+
   | Cluster EKS  |  | Cluster RKE2 |  | Cluster K3s  |
   | cattle-agent |  | cattle-agent |  | cattle-agent |
   |              |  | (on-premises)|  | (edge/IoT)   |
   +--------------+  +--------------+  +--------------+
```

Componentes e conceitos:

- **Rancher Server** — a plataforma de gestão, que roda como aplicação em um cluster Kubernetes dedicado (o "local cluster"). Concentra UI, autenticação (AD, LDAP, SAML, GitHub), políticas e o catálogo.
- **cattle-cluster-agent** — o agente instalado em cada cluster gerenciado. Ele **inicia** a conexão com o Rancher Server, o que permite gerenciar clusters atrás de NAT ou firewall sem expor o API server.
- **RKE2** — distribuição Kubernetes própria, endurecida, com foco em conformidade (CIS Benchmark, FIPS 140-2, certificação FedRAMP). É a escolha padrão para on-premises regulado.
- **K3s** — distribuição minimalista (binário único, < 100 MB, com SQLite como datastore padrão). Feita para edge, IoT, CI e desenvolvimento local.
- **Projects** — abstração exclusiva do Rancher: um agrupamento **acima** do namespace. Você concede acesso a um Project e a pessoa passa a ter acesso a todos os namespaces dentro dele, presentes e futuros. Resolve um limite real do RBAC nativo, em que não existe nada entre "um namespace" e "o cluster inteiro".
- **Catálogo de apps** — repositórios Helm curados, instaláveis pela UI, com valores editáveis em formulário. Útil para times que não vivem no terminal.
- **Fleet** — o motor GitOps do Rancher.

#### 11.10.2 Fleet: GitOps em escala de frota

O Fleet foi projetado para um caso extremo: implantar a mesma aplicação em **milhares** de clusters (pense em uma rede de varejo com um K3s por loja). A unidade é o `GitRepo`, e os alvos são selecionados por **label de cluster**, não por nome.

```yaml
apiVersion: fleet.cattle.io/v1alpha1
kind: GitRepo
metadata:
  name: fiap-store
  namespace: fleet-default
spec:
  repo: https://github.com/fabiobara/fiap-store-config.git
  branch: main
  paths:
    - overlays/prod
  targets:
    - name: producao
      clusterSelector:
        matchLabels:
          ambiente: producao
          regiao: br-sudeste
```

| | **Argo CD** | **Rancher Fleet** |
|---|---|---|
| Foco | entrega contínua rica em um ou poucos clusters | distribuição em massa para muitos clusters |
| Escala testada | dezenas de clusters | milhares de clusters |
| UI | própria, muito detalhada | dentro da UI do Rancher |
| Customização por cluster | via ApplicationSet | nativa (`targetCustomizations` por label) |
| Independente do Rancher | sim | roda fora, mas o valor está no Rancher |

#### 11.10.3 Quando escolher Rancher x Argo CD

A pergunta está mal formulada, porque **eles não competem**: o Rancher gerencia clusters; o Argo CD entrega aplicações. É perfeitamente comum — e recomendável — usar Rancher para provisionar e governar a frota e Argo CD para o CD das aplicações (o Rancher inclusive oferece o Argo CD em seu catálogo).

| Cenário | Escolha |
|---|---|
| Um ou poucos clusters, foco em entrega de aplicações, time DevOps maduro | **Argo CD** puro |
| Muitos clusters heterogêneos (nuvem + on-prem + edge), necessidade de UI e autenticação corporativa centralizada | **Rancher** (+ Fleet) |
| Ambiente regulado, on-premises, exigência de CIS/FIPS | **Rancher + RKE2** |
| Edge/IoT com centenas de sites | **Rancher + K3s + Fleet** |
| Já usa EKS gerenciado e quer só GitOps | **Argo CD** (ou Flux) |
| Governança de frota **e** CD sofisticado (canary com análise) | **Rancher + Argo CD/Rollouts** |

---

### 11.11 Integração com CI: o fluxo completo

Vamos fechar o ciclo. O CI **não** fala com o cluster; ele constrói, escaneia, publica e **abre um PR** no config repo.

```
 dev faz push no app repo
        |
        v
 [GitHub Actions]
   1. testes unitarios
   2. docker build
   3. trivy scan (--exit-code 1)
   4. cosign sign
   5. docker push  store-api:1.2.3
   6. abre PR no fiap-store-config alterando newTag para 1.2.3
        |
        v
 revisao humana + merge na main
        |
        v
 [Argo CD] detecta o commit (webhook ou poll de 3 min)
        |
        v
 sync -> cluster reconciliado -> Healthy
```

```yaml
# .github/workflows/ci.yaml (no app repo fiap-store-api)
name: CI store-api
on:
  push:
    branches: [main]

permissions:
  contents: read

jobs:
  build-e-promover:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Definir tag pela SHA curta
        id: tag
        run: echo "valor=1.2.${GITHUB_RUN_NUMBER}" >> "$GITHUB_OUTPUT"

      - name: Login no Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

      - name: Build e push
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: docker.io/fabiobara/store-api:${{ steps.tag.outputs.valor }}

      - name: Scan de vulnerabilidades
        uses: aquasecurity/trivy-action@0.28.0
        with:
          image-ref: docker.io/fabiobara/store-api:${{ steps.tag.outputs.valor }}
          severity: CRITICAL
          ignore-unfixed: true
          exit-code: '1'

      - name: Assinar a imagem
        run: |
          cosign sign --key env://COSIGN_KEY \
            docker.io/fabiobara/store-api:${{ steps.tag.outputs.valor }}
        env:
          COSIGN_KEY: ${{ secrets.COSIGN_KEY }}

      - name: Clonar o config repo
        uses: actions/checkout@v4
        with:
          repository: fabiobara/fiap-store-config
          token: ${{ secrets.CONFIG_REPO_TOKEN }}
          path: config

      - name: Atualizar a tag em dev
        run: |
          cd config/overlays/dev
          kustomize edit set image \
            docker.io/fabiobara/store-api=docker.io/fabiobara/store-api:${{ steps.tag.outputs.valor }}

      - name: Abrir PR de promocao
        uses: peter-evans/create-pull-request@v7
        with:
          path: config
          token: ${{ secrets.CONFIG_REPO_TOKEN }}
          branch: bump/store-api-${{ steps.tag.outputs.valor }}
          commit-message: "chore(dev): store-api para ${{ steps.tag.outputs.valor }}"
          title: "Promover store-api ${{ steps.tag.outputs.valor }} para dev"
          body: |
            Imagem construida, escaneada e assinada pelo run ${{ github.run_id }}.
            Merge deste PR faz o Argo CD sincronizar o namespace store-dev.
```

Repare no que **não** existe nesse workflow: nenhum `KUBECONFIG`, nenhum `kubectl`, nenhuma credencial de cluster. O token mais poderoso ali dá acesso de escrita a um repositório Git — cujo `main` está protegido por revisão obrigatória.

Para eliminar o atraso de até 3 minutos do polling, configure um webhook do GitHub apontando para `https://argocd.suaempresa.com/api/webhook`. O sync passa a ser imediato.

> **Na prática:** o **Argo CD Image Updater** é uma alternativa que automatiza o bump sem PR — ele observa o registry e escreve a nova tag direto. É conveniente para dev, mas remove a revisão humana; para produção, prefira o PR. É a diferença entre velocidade e governança, e cada ambiente merece uma resposta diferente.

---

### 11.12 Ponte com Serverless

GitOps não é uma técnica de Kubernetes — é uma técnica de **estado desejado declarativo**. Onde houver descrição declarativa, GitOps se aplica. E serverless é declarativo por natureza.

**GitOps para funções.** Uma AWS Lambda pode ser descrita em `template.yaml` (AWS SAM), em `serverless.yml` (Serverless Framework), em CDK ou em Terraform. Todos produzem, ao final, um CloudFormation stack — um estado desejado. O pipeline então fica: commit → CI valida e empacota → `sam deploy` ou `cdk deploy`. Isso é, tecnicamente, **CIOps**: há um push, com credenciais AWS no runner. É o modelo dominante hoje no mundo serverless puro, e o motivo é simples: a Lambda não roda um agente de reconciliação dentro de si mesma. A mitigação moderna é usar OIDC do GitHub Actions para assumir uma role AWS de curta duração, eliminando as chaves estáticas — o que resolve o pior problema do push, mas não devolve a reconciliação contínua. Quem quer o modelo pull de verdade em recursos de nuvem usa **Crossplane** ou o **AWS Controllers for Kubernetes (ACK)**: eles transformam recursos AWS em objetos Kubernetes, e aí o Argo CD passa a reconciliar sua fila SQS, seu bucket S3 e sua função Lambda exatamente como reconcilia um Deployment.

**Knative Service como recurso declarativo do Argo.** Aqui a integração é perfeita e não exige nenhuma ponte. Um `Service` do Knative Serving é um CRD como qualquer outro: se está no Git, o Argo o gerencia.

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: store-api
  namespace: store-prod
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/minScale: "0"     # escala a zero
        autoscaling.knative.dev/maxScale: "50"
        autoscaling.knative.dev/target: "100"     # requisicoes concorrentes por Pod
    spec:
      containerConcurrency: 100
      containers:
        - image: docker.io/fabiobara/store-api:1.2
          ports:
            - containerPort: 8000
          env:
            - name: APP_ENV
              value: prod
          resources:
            requests: { cpu: 100m, memory: 128Mi }
```

O Argo CD trata esse objeto normalmente: mostra o diff, sincroniza, avalia o health (o Knative expõe `status.conditions`, que o Argo entende) e faz `prune` se o arquivo sumir do Git. **Escala a zero e reconciliação contínua convivem sem conflito**, porque o Argo compara o `spec` (o desejado), não a contagem de Pods (o real, que o Knative gerencia). É o mesmo raciocínio do `ignoreDifferences` para o HPA: o GitOps cuida da intenção, o autoscaler cuida da execução.

**"Tudo é manifesto".** Essa é a ideia final do curso e o fio que amarra os onze capítulos. Um Deployment é um manifesto. Um HPA é um manifesto. Uma NetworkPolicy, uma policy Kyverno, um Knative Service, um `ScaledObject` do KEDA, um `Application` do Argo, uma tabela DynamoDB via ACK — todos são manifestos. Quando **tudo** o que a empresa opera está descrito em arquivos versionados num repositório, três propriedades emergem de graça: a auditoria vira `git log`, o rollback vira `git revert`, e a recuperação de desastre vira `git clone` num cluster novo. Não é sobre a ferramenta; é sobre transformar operação em algo que se revisa em pull request.

---

### 11.13 Laboratório guiado: Argo CD no minikube com a FIAP Store

**Passo 1 — subir o cluster e instalar o Argo CD**

```bash
minikube start --kubernetes-version=v1.33.0 --cpus=4 --memory=6g
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.13.3/manifests/install.yaml
kubectl wait --for=condition=available --timeout=300s deploy/argocd-server -n argocd
```

**Passo 2 — acessar a UI**

```bash
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d; echo
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Abra `https://localhost:8080` (aceite o certificado autoassinado), usuário `admin`.

```bash
argocd login localhost:8080 --username admin --password '<senha>' --insecure
```

**Passo 3 — criar o repositório de configuração**

Crie no GitHub um repositório **público** chamado `fiap-store-config` com esta estrutura:

```yaml
# base/store-api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  labels:
    app: store-api
    tier: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: store-api
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
    spec:
      terminationGracePeriodSeconds: 45
      securityContext:
        runAsNonRoot: true
        runAsUser: 65532
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: store-api
          image: docker.io/fabiobara/store-api:1.0
          ports:
            - name: http
              containerPort: 8000
          env:
            - name: APP_ENV
              value: "dev"
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          resources:
            requests: { cpu: 100m, memory: 128Mi }
            limits:   { memory: 256Mi }
          readinessProbe:
            httpGet: { path: /ready, port: 8000 }
            periodSeconds: 5
          livenessProbe:
            httpGet: { path: /health, port: 8000 }
            periodSeconds: 10
          lifecycle:
            preStop:
              sleep:
                seconds: 10
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

```yaml
# base/store-api-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  labels:
    app: store-api
spec:
  selector:
    app: store-api
  ports:
    - name: http
      port: 80
      targetPort: 8000
```

```yaml
# base/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - store-api-deployment.yaml
  - store-api-service.yaml
```

```yaml
# overlays/dev/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: store-dev
resources:
  - ../../base
labels:
  - pairs:
      env: dev
    includeSelectors: false
images:
  - name: docker.io/fabiobara/store-api
    newTag: "1.0"
replicas:
  - name: store-api
    count: 2
```

```bash
kubectl kustomize overlays/dev    # valide localmente antes de commitar
git add . && git commit -m "manifestos iniciais da FIAP Store" && git push
```

**Passo 4 — criar a Application**

```yaml
# apps/store-dev.yaml (aplique com kubectl, no namespace argocd)
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: fiap-store-dev
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://github.com/SEU-USUARIO/fiap-store-config.git
    targetRevision: main
    path: overlays/dev
  destination:
    server: https://kubernetes.default.svc
    namespace: store-dev
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

```bash
kubectl apply -f apps/store-dev.yaml
argocd app get fiap-store-dev
kubectl get pods -n store-dev
```

```
NAME                        READY   STATUS    RESTARTS   AGE
store-api-9f7d6c8b4-jk2mx   1/1     Running   0          38s
store-api-9f7d6c8b4-pq8vn   1/1     Running   0          38s
```

Na UI, observe o grafo de recursos: `Application → Deployment → ReplicaSet → Pod`, cada nó com seu sync status e health.

**Passo 5 — alterar a réplica no Git e ver a reconciliação**

```bash
# no repositorio local, edite overlays/dev/kustomization.yaml:
#   replicas: count: 2  ->  count: 4
git commit -am "escala store-api para 4 replicas em dev" && git push

argocd app get fiap-store-dev --refresh
```

Sem tocar no cluster, em até 3 minutos (ou imediatamente com `argocd app sync fiap-store-dev`):

```bash
kubectl get pods -n store-dev
```

```
NAME                        READY   STATUS    RESTARTS   AGE
store-api-9f7d6c8b4-jk2mx   1/1     Running   0          6m
store-api-9f7d6c8b4-pq8vn   1/1     Running   0          6m
store-api-9f7d6c8b4-r4tzc   1/1     Running   0          22s
store-api-9f7d6c8b4-x9bwl   1/1     Running   0          22s
```

**Passo 6 — provocar drift manual e observar o `selfHeal`**

Este é o passo mais importante do laboratório. Simule a madrugada de incidente:

```bash
kubectl scale deploy/store-api -n store-dev --replicas=1
kubectl get pods -n store-dev          # imediatamente: 1 Pod
argocd app get fiap-store-dev          # Sync Status: OutOfSync

# aguarde ate ~3 minutos (ou force com: argocd app get fiap-store-dev --refresh)
kubectl get pods -n store-dev          # de volta a 4 Pods
```

```
$ argocd app history fiap-store-dev
ID  DATE                           REVISION
0   2026-08-06 14:02:11 -0300      main (a3f9c21)
1   2026-08-06 14:09:47 -0300      main (7d1e4b8)
```

Tente também um `kubectl edit` alterando a imagem:

```bash
kubectl set image deploy/store-api store-api=nginx:1.27 -n store-dev
argocd app diff fiap-store-dev
```

```
===== apps/Deployment store-dev/store-api ======
23c23
<           image: nginx:1.27
---
>           image: docker.io/fabiobara/store-api:1.0
```

Em poucos minutos, o Argo devolve a imagem correta. **Isto é o quarto princípio da OpenGitOps funcionando.** Compare com o modelo push: nele, essa alteração ficaria lá até o próximo deploy — talvez semanas.

**Passo 7 — rollback por Git**

```bash
# faca uma mudanca ruim de proposito
# em overlays/dev/kustomization.yaml: newTag: "1.0" -> newTag: "9.9-inexistente"
git commit -am "bump para versao inexistente" && git push
argocd app sync fiap-store-dev

kubectl get pods -n store-dev
```

```
NAME                        READY   STATUS             RESTARTS   AGE
store-api-6d4b8f9c2-h7kpq   0/1     ImagePullBackOff   0          45s
```

```bash
argocd app get fiap-store-dev
# Sync Status:   Synced          <-- o Argo aplicou exatamente o que o Git pediu
# Health Status: Degraded        <-- o que o Git pediu esta quebrado
```

Guarde essa observação: **`Synced` + `Degraded` é a assinatura de "o problema está no seu manifesto"**. Agora reverta:

```bash
git revert HEAD --no-edit && git push
argocd app sync fiap-store-dev
kubectl get pods -n store-dev
# volta a Running; Health Status: Healthy
```

**Passo 8 — testar o `prune`**

```bash
git rm base/store-api-service.yaml
# remova a linha correspondente de base/kustomization.yaml
git commit -am "remove o Service" && git push
# em ate 3 minutos:
kubectl get svc -n store-dev
# No resources found in store-dev namespace.
```

Restaure com `git revert` e confirme que o Service volta sozinho.

**Limpeza**

```bash
kubectl delete application fiap-store-dev -n argocd   # o finalizer remove os recursos
kubectl delete namespace argocd store-dev
minikube stop
```

---

### 11.14 Erros comuns e troubleshooting

| Sintoma | Causa provável | Como diagnosticar | Correção |
|---|---|---|---|
| Application `Unknown` logo após criar | repo privado sem credencial, ou URL errada | `argocd app get <app>`; logs do `argocd-repo-server` | `argocd repo add <url> --username ... --password <PAT>` |
| `ComparisonError: rpc error ... unable to resolve` | branch/tag inexistente em `targetRevision`, ou `path` errado | `argocd app get` → seção `CONDITIONS` | corrigir `targetRevision`/`path`; conferir com `kubectl kustomize <path>` local |
| Application eternamente `OutOfSync` sem ninguém mexer | outro controlador escreve no objeto (HPA em `replicas`, webhook mutante, `metadata.annotations`) | `argocd app diff <app>` mostra sempre o mesmo campo | `ignoreDifferences` no campo, e remover `replicas` do manifesto |
| `Synced` + `Degraded` | o manifesto está correto do ponto de vista do Git, mas o app não sobe | `kubectl describe pod`, `kubectl logs` | corrigir o manifesto/imagem no **Git**, nunca no cluster |
| Sync falha: `secrets "store-db-credentials" not found` | Secret não está no Git (corretamente) nem foi provisionado no cluster | `kubectl get events -n store-prod` | instalar Sealed Secrets ou External Secrets e criar o `ExternalSecret` no repo |
| Sync falha com `forbidden: User "system:serviceaccount:argocd:argocd-application-controller" cannot create ...` | RBAC do Argo insuficiente para aquele kind (comum com CRDs e ClusterRole) | logs do `argocd-application-controller` | ajustar o ClusterRole do controller ou o `clusterResourceWhitelist` do AppProject |
| `Application ... is not permitted in project` | `destination` ou `sourceRepos` fora do que o `AppProject` permite | `kubectl describe appproject <proj> -n argocd` | adicionar namespace/servidor/repo ao AppProject |
| `selfHeal` desfaz uma correção urgente durante incidente | é exatamente o comportamento esperado | — | `argocd app set <app> --sync-policy none`, corrigir, commitar no Git, religar |
| Deletar a Application derrubou a produção | finalizer `resources-finalizer` fazendo cascata | `kubectl get application <app> -o yaml` | remover o finalizer **antes** do delete se quiser preservar os recursos |
| Recursos não removidos ao apagar o arquivo do Git | `prune: false` (padrão) | UI mostra o recurso como órfão | `automated.prune: true` ou `argocd app sync --prune` |
| Sync demora até 3 minutos | polling padrão do controller | — | configurar webhook do Git para `/api/webhook` |
| Hook `PreSync` roda toda vez e falha na segunda | Job com nome fixo e sem `hook-delete-policy` | `kubectl get jobs -n <ns>` | `argocd.argoproj.io/hook-delete-policy: HookSucceeded` |
| ApplicationSet não gera nada | `goTemplate` com chave inexistente, ou gerador sem resultados | logs do `argocd-applicationset-controller` | `goTemplateOptions: ["missingkey=error"]` ajuda a expor o erro |
| Segredo em texto claro apareceu no diff da UI | Secret cru commitado no repo | `argocd app diff` | rotacionar a credencial imediatamente e migrar para External Secrets |

---

### 11.15 Resumo do capítulo

GitOps é a conclusão lógica de tudo o que estudamos: se o Kubernetes já é uma máquina de reconciliação que persegue um estado declarado, faz todo o sentido que o estado declarado viva num repositório Git — versionado, revisável, assinável e permanente — e que um agente dentro do cluster o busque continuamente. Os quatro princípios da OpenGitOps (declarativo, versionado/imutável, puxado automaticamente, reconciliado continuamente) transformam essa intuição em critério objetivo. A inversão do fluxo, de push para pull, resolve de uma só vez quatro problemas que pareciam independentes: credenciais de cluster fora do cluster, drift silencioso, ausência de auditoria de negócio e recuperação de desastre.

O Argo CD materializa esse modelo com uma arquitetura simples de entender — repo server renderiza, application controller compara e aplica, API server expõe a UI — e um objeto central, a `Application`, cuja `syncPolicy` com `prune` e `selfHeal` é onde os princípios viram booleanos. À medida que a operação cresce, App-of-Apps e `ApplicationSet` eliminam a repetição, `AppProject` estabelece as fronteiras de multi-tenancy, e sync waves com hooks ordenam o que precisa de ordem. Segredos exigem um tratamento à parte — Sealed Secrets, External Secrets ou SOPS — porque a única regra inegociável é que Secret cru nunca vai para o Git. Por fim, Rancher e Argo CD não competem: um governa a frota de clusters, o outro entrega as aplicações, e juntos cobrem desde a nuvem gerenciada até o K3s na loja da esquina.

**Pontos-chave**

1. Os 4 princípios da OpenGitOps — declarativo, versionado/imutável, puxado automaticamente e reconciliado continuamente — são o critério para dizer se um processo é GitOps de fato.
2. No modelo pull, nenhuma credencial de cluster existe fora do cluster; o CI só constrói, escaneia, assina e abre PR.
3. Prefira pastas por ambiente a branches por ambiente: promoção vira um PR de uma linha, sem arrastar diferenças de configuração indesejadas.
4. `prune: true` faz o Git ser a verdade também nas remoções; `selfHeal: true` é o quarto princípio em um booleano.
5. Sync status e health status são dimensões independentes: `Synced` + `Degraded` significa que o defeito está no seu manifesto, não no Argo.
6. Secret cru nunca vai para o Git; use Sealed Secrets (cluster único), External Secrets (produção em nuvem) ou SOPS, e trate qualquer segredo já commitado como comprometido.
7. GitOps se estende ao serverless: Knative Services são manifestos que o Argo reconcilia normalmente, e Crossplane/ACK trazem recursos de nuvem para o mesmo laço — porque, no fim, **tudo é manifesto**.

---

### 11.16 Exercícios (para casa)

#### Exercício 11.1 — Instalar o Argo CD e sincronizar a primeira Application (fixação guiada) · 40 min

**Objetivo:** instalar o Argo CD no minikube e colocar a `store-api` sob gestão GitOps com sync automático.

**Enunciado:** suba o minikube com Kubernetes 1.33, instale o Argo CD v2.13.x no namespace `argocd`, recupere a senha inicial, troque-a e delete o `argocd-initial-admin-secret`. Crie no GitHub um repositório público `fiap-store-config` com `base/` (Deployment + Service da `store-api`, com o `securityContext` completo do capítulo 10) e `overlays/dev/` (namespace `store-dev`, 2 réplicas, tag `1.0`). Valide localmente com `kubectl kustomize overlays/dev` **antes** de commitar. Crie a `Application` `fiap-store-dev` com `syncPolicy.automated` (`prune: true`, `selfHeal: true`), `syncOptions: [CreateNamespace=true]` e o finalizer `resources-finalizer.argocd.argoproj.io`. Por fim, faça um commit alterando as réplicas de 2 para 4 e **não execute nenhum comando no cluster** — apenas aguarde e comprove a reconciliação.

**Entregável:** link do repositório; `argocd app get fiap-store-dev` mostrando `Synced/Healthy`; `kubectl get pods -n store-dev -o wide` antes e depois do commit; print do grafo de recursos na UI do Argo; e o horário do commit comparado ao horário de criação dos Pods novos (`kubectl get pods -n store-dev --sort-by=.metadata.creationTimestamp`).

**Critério de aceite:** a Application em `Sync Status: Synced` e `Health Status: Healthy`; 4 Pods em `Running` **sem** nenhum `kubectl apply`/`scale` executado por você após a criação da Application; e `argocd app history fiap-store-dev` listando ao menos 2 revisões distintas.

<details><summary>Dica</summary>
Se a Application ficar `Unknown`, o problema é quase sempre o `repoURL` (use a URL `.git` completa) ou o `path` (relativo à raiz do repositório, sem barra inicial). Para não esperar os 3 minutos do polling, use `argocd app get fiap-store-dev --refresh`.
</details>

---

#### Exercício 11.2 — Promoção de dev para prod por PR com Kustomize overlays (aplicação) · 50 min

**Objetivo:** implementar a promoção controlada de uma versão entre ambientes usando pull request, sem duplicar manifestos.

**Enunciado:** partindo do repositório do exercício anterior, crie `overlays/prod/` com: namespace `store-prod`, 4 réplicas, `APP_ENV=prod`, recursos maiores, um `PodDisruptionBudget` (`minAvailable: 2`) e um patch com `terminationGracePeriodSeconds: 60` — tudo reaproveitando `base/` via `resources: [../../base]`, **sem copiar** o Deployment. Crie a Application `fiap-store-prod` apontando para `overlays/prod` com tag `1.0`, e um `AppProject` chamado `fiap-store` que permita **apenas** este repositório e **apenas** os namespaces `store-dev` e `store-prod`, com `clusterResourceWhitelist` restrito a `Namespace`. Ambas as Applications devem pertencer a esse projeto.

Em seguida, simule uma release: publique `store-api:1.1`, altere a tag em `overlays/dev` (commit direto), verifique que dev reconcilia. Depois, **abra um pull request** alterando apenas `overlays/prod/kustomization.yaml` de `1.0` para `1.1`, revise o diff, faça o merge e observe prod reconciliar. Documente o diff exato do PR de promoção.

**Entregável:** árvore do repositório (`tree`); os dois `kustomization.yaml` de overlay; o `AppProject` em YAML; link/print do PR de promoção com o diff; saída de `kubectl get deploy,pdb -n store-dev` e `-n store-prod` mostrando as diferenças; e saída de `argocd app list`.

**Critério de aceite:** o diff do PR de promoção contém **no máximo 2 linhas alteradas**; os dois ambientes coexistem com imagens diferentes comprovadas por `kubectl get deploy -n store-dev -o jsonpath='{.items[*].spec.template.spec.containers[*].image}'` e o equivalente em prod; ambas as Applications aparecem com `PROJECT: fiap-store`; e tentar criar uma Application apontando para `kube-system` é rejeitada pelo AppProject.

<details><summary>Dica</summary>
`kustomize edit set image docker.io/SEU-USUARIO/store-api=docker.io/SEU-USUARIO/store-api:1.1` altera a tag sem editar YAML à mão — é exatamente o comando que o CI usará no exercício 11.4. Para testar a rejeição do AppProject, aplique uma Application com `destination.namespace: kube-system` e leia a mensagem em `kubectl describe application`.
</details>

---

#### Exercício 11.3 — Depuração: Application `OutOfSync` e `Degraded` (análise) · 50 min

**Objetivo:** diagnosticar e corrigir os três modos de falha mais comuns em GitOps, distinguindo problema de sync de problema de aplicação.

**Enunciado:** você é o plantonista. Reproduza, diagnostique e corrija **três cenários**, documentando para cada um: sintoma observado, comando(s) usados no diagnóstico, causa raiz e correção — **sempre corrigindo pelo Git**, nunca pelo cluster.

**Cenário A — drift manual.** Com `selfHeal: true` ativo, execute `kubectl scale deploy/store-api -n store-dev --replicas=1` e depois `kubectl set image deploy/store-api store-api=nginx:1.27 -n store-dev`. Registre a saída de `argocd app diff` **antes** da reconciliação e comprove que o Argo desfez ambas as alterações. Em seguida, desligue o `selfHeal` (`argocd app set ... --sync-policy none`), repita o drift e explique a diferença de comportamento.

**Cenário B — `Synced` + `Degraded`.** Commite uma tag de imagem inexistente. Comprove que o Argo reporta `Synced` e `Degraded` simultaneamente e explique, em 5 linhas, por que essa combinação **não** é um bug do Argo. Corrija com `git revert`.

**Cenário C — sync falhando por dependência ausente.** Adicione ao `base/` um Deployment que consome `DB_PASSWORD` via `secretKeyRef` de um Secret `store-db-credentials` que **não** existe no cluster (e que corretamente não está no Git). Observe o Pod em `CreateContainerConfigError`. Resolva instalando o Sealed Secrets, gerando o `SealedSecret` com `kubeseal`, commitando-o e deixando o Argo criar o Secret real. Comprove que o arquivo commitado **não** contém a senha em texto legível.

**Cenário D (bônus) — falha por RBAC.** Crie um `AppProject` cujo `clusterResourceWhitelist` não inclua `Namespace` e tente sincronizar uma Application com `CreateNamespace=true`. Registre a mensagem de erro.

**Entregável:** um relatório em Markdown com uma seção por cenário (sintoma, diagnóstico, causa raiz, correção), incluindo as saídas de `argocd app get`, `argocd app diff`, `kubectl describe pod` e os logs relevantes do `argocd-application-controller`; mais o `SealedSecret` commitado e a saída de `kubectl get secret store-db-credentials -n store-dev`.

**Critério de aceite:** os três cenários principais reproduzidos e corrigidos, com a Application terminando em `Synced/Healthy` nos três; a explicação do cenário B identifica corretamente que sync compara Git x cluster e health avalia o funcionamento dos recursos; e o `SealedSecret` no repositório não revela a senha (comprove com `grep -i` na senha original retornando vazio).

<details><summary>Dica</summary>
`kubectl logs -n argocd deploy/argocd-application-controller --tail=100` é onde as falhas reais de sync aparecem — a UI muitas vezes só mostra o resumo. Para o cenário C, `kubeseal --format yaml --controller-namespace kube-system` lê um Secret comum do stdin e escreve o `SealedSecret` no stdout.
</details>

---

#### Exercício 11.4 — Desafio: App-of-Apps/ApplicationSet + CI que atualiza a tag (desafio integrador) · 60 min

**Objetivo:** colocar toda a FIAP Store sob um único ponto de entrada GitOps em dois namespaces e automatizar o bump de imagem pelo CI.

**Enunciado:** monte a solução completa em duas partes.

**Parte 1 — gestão declarativa da frota.** Escolha **um** dos padrões e justifique a escolha em 5 linhas:

- **App-of-Apps:** uma Application raiz `fiap-store-root` apontando para `apps/`, que contém `store-dev.yaml`, `store-prod.yaml` e `infra.yaml` (este último instalando as policies Kyverno do capítulo 10).
- **ApplicationSet:** um `ApplicationSet` com gerador `git` por diretório (`directories: [{path: overlays/*}]`) ou gerador `list` com os elementos `dev` e `prod`.

O resultado deve ser que **um único `kubectl apply`** faça subir `store-api`, `store-front` e `store-worker` nos namespaces `store-dev` e `store-prod`, com configurações distintas por ambiente e com o `AppProject` `fiap-store` aplicado. Use `sync-wave` para garantir que o namespace e as policies sejam criados antes das aplicações, e adicione um hook `PreSync` que rode um Job de validação (pode ser um `echo` didático que verifica a conectividade com o `store-db`).

**Parte 2 — CI que promove automaticamente.** Crie no app repo um workflow do GitHub Actions que, a cada push na `main`: rode os testes, construa a imagem com tag baseada em `github.run_number`, escaneie com Trivy (`--exit-code 1 --severity CRITICAL --ignore-unfixed`), faça push, e então clone o config repo, execute `kustomize edit set image` em `overlays/dev` e **abra um PR**. Faça o merge desse PR e comprove que o Argo CD sincroniza `store-dev` com a nova tag sem nenhuma ação sua no cluster. Demonstre que **não existe nenhum `KUBECONFIG` nem `kubectl` no workflow**.

**Componente serverless (obrigatório):** adicione ao repositório um `Knative Service` equivalente à `store-api` (com `minScale: 0` e `maxScale: 20`) em `overlays/serverless/`, e crie uma Application para ele. Se não tiver o Knative instalado, entregue o manifesto e um `kubectl apply --dry-run=client` válido, mais uma análise de 10 a 15 linhas respondendo: (a) por que `minScale: 0` **não** conflita com a reconciliação contínua do Argo; (b) quais campos você precisaria colocar em `ignoreDifferences` e por quê; (c) o que muda no `ApplicationSet` se metade dos ambientes for Knative e metade Deployment.

**Entregável:** árvore completa do config repo; o YAML do App-of-Apps **ou** do ApplicationSet; o `AppProject`; o workflow `.github/workflows/ci.yaml`; link/print do PR aberto automaticamente pelo CI; `argocd app list` mostrando todas as Applications geradas; `kubectl get all -n store-dev` e `-n store-prod`; o manifesto Knative; e o texto da análise serverless.

**Critério de aceite:** um único `kubectl apply -f` (do root/ApplicationSet) resulta em pelo menos 4 Applications no `argocd app list`, todas `Synced/Healthy`; os dois namespaces têm configurações comprovadamente distintas; o PR de bump foi criado pelo CI **sem intervenção manual**; o workflow não contém nenhuma credencial de cluster; e a análise serverless responde corretamente aos três pontos, identificando que o Argo compara `spec` (intenção) e não a contagem de Pods (execução).

<details><summary>Dica</summary>
Com `goTemplate: true` no ApplicationSet, use `{{.path.basename}}` no gerador `git` para nomear a Application a partir do nome da pasta. Para o PR automático, `peter-evans/create-pull-request@v7` exige um PAT com escopo `repo` no config repo (o `GITHUB_TOKEN` padrão não cruza repositórios). Se o hook `PreSync` reexecutar e falhar, faltou `hook-delete-policy: HookSucceeded`.
</details>

---

### 11.17 Leitura recomendada

1. **[DOC]** OpenGitOps (CNCF) — *GitOps Principles v1.0*. Disponível em: https://opengitops.dev/ — o texto normativo dos quatro princípios; leitura de 10 minutos, obrigatória.
2. **[DOC]** Argo CD — *Declarative Setup, Application Specification e Best Practices*. Disponível em: https://argo-cd.readthedocs.io/en/stable/ — a referência completa de `Application`, `AppProject`, `ApplicationSet`, hooks e sync options.
3. **[DOC]** Rancher / SUSE — *Rancher Manager e Fleet (Continuous Delivery)*. Disponível em: https://ranchermanager.docs.rancher.com/ e https://fleet.rancher.io/ — arquitetura multi-cluster, Projects, RKE2/K3s e o modelo de targets por label.
4. **[ARTIGO]** Weaveworks — *Guide to GitOps*. Disponível em: https://www.weave.works/technologies/gitops/ — o texto que originou o termo, útil para entender a motivação histórica do modelo pull.
5. **[LIVRO]** ARUNDEL, John; DOMINGUS, Justin. *Cloud Native DevOps with Kubernetes*. 2. ed. O'Reilly, 2022 — capítulos sobre entrega contínua, gestão de configuração e operação de clusters em produção.
6. **[VÍDEO]** CNCF / Argo Project — *Argo CD Deep Dive* (KubeCon). Disponível em: https://www.youtube.com/@cncf — sessões anuais que cobrem ApplicationSet, multi-tenancy e escala; procure a edição mais recente.

---

> ### Marco avaliativo — CHECK POINT 03 e GLOBAL SOLUTION
>
> Encerrado o conteúdo, são lançados o **Check Point 03 — Segurança, automação e entrega contínua
> com GitOps** (capítulos 8 a 11) e a **Global Solution**, trabalho final de maior peso. Ambos estão
> detalhados nas seções seguintes.

---

# Parte II — Avaliações

Esta parte reúne os instrumentos avaliativos da disciplina **Cloud Developer II — Kubernetes e
Serverless**. São três Check Points aplicados ao longo do semestre e uma Global Solution (trabalho
final, documentada em arquivo próprio).

Os três Check Points **não são independentes**: eles formam uma trilha única sobre a mesma
aplicação-exemplo, a **FIAP Store**. O aluno entrega no CP01 um repositório que será evoluído no
CP02 e novamente no CP03. Quem não entregou o CP01 ainda consegue fazer o CP02 (há um repositório
base fornecido pelo professor), mas perde a continuidade narrativa e a economia de esforço.

## Visão geral dos instrumentos

| Instrumento | Ementa coberta | Aplicação | Modalidade | Peso na média |
|---|---|---|---|---|
| Check Point 01 | Capítulos 1 a 4 | Após a aula 4 | Individual | 1/3 da nota de CP |
| Check Point 02 | Capítulos 5 a 7 | Após a aula 7 | Dupla | 1/3 da nota de CP |
| Check Point 03 | Capítulos 8 a 11 | Após a aula 11 | Dupla | 1/3 da nota de CP |
| Global Solution | Todos os capítulos | Fim do semestre | Grupo de 3 a 5 | Peso de prova |

> **Nota:** a média final sugerida é `MF = 0,4 x MediaCP + 0,6 x GS`, onde `MediaCP` é a média
> aritmética simples dos três Check Points. Ajuste conforme o regulamento da instituição.

## Convenções comuns aos três Check Points

**Repositório.** Toda entrega é um repositório Git público (ou privado com o professor como
colaborador) chamado `fiap-store-k8s-<seu-rm>`. A mesma base evolui a cada Check Point; cada entrega
é marcada com uma tag anotada: `cp01`, `cp02`, `cp03`.

```bash
git tag -a cp01 -m "Entrega do Check Point 01 - RM123456"
git push origin cp01
```

**Formato do vídeo.** MP4 ou link não listado no YouTube/Drive com permissão de leitura para
qualquer pessoa com o link. Áudio obrigatório, câmera opcional, tela compartilhada obrigatória. O
vídeo precisa mostrar **comandos sendo executados ao vivo**, não slides.

**Nomenclatura de arquivos.** Manifestos sempre em `kebab-case` com o kind no nome:
`store-api-deployment.yaml`, `store-api-service.yaml`, `store-api-hpa.yaml`.

**Prints.** PNG ou JPG na pasta `docs/evidencias/`, numerados (`01-pods-running.png`). Print sem
legenda no README não conta como evidência.

---

## Check Point 01 — Fundamentos, YAML e alta disponibilidade no Minikube

### CP01.1 Enquadramento

| Item | Valor |
|---|---|
| Ementa coberta | Capítulos 1 a 4: orquestradores e arquitetura do K8s; instalação e ambientação com Minikube; Pods e Services em YAML; replicação, distribuição e alta disponibilidade |
| Momento de aplicação | Enunciado liberado na aula 4; entrega antes da aula 6 |
| Modalidade | **Individual** (o objetivo é garantir que cada aluno saiba escrever YAML do zero) |
| Prazo sugerido | **10 dias corridos** a partir da liberação |
| Carga horária estimada | 8 a 12 horas de trabalho do aluno |
| Foco | Fundamentos, escrita de manifestos, ciclo de vida do Pod, probes, rolling update, HPA |
| Infraestrutura exigida | Minikube local (sem custo de nuvem) |

### CP01.2 Enunciado principal

#### Cenário

A **FIAP Store** é uma loja online que hoje roda em um único servidor, com a aplicação iniciada
manualmente por `uvicorn main:app` dentro de uma sessão `tmux`. Nas últimas quatro semanas houve
três incidentes: o processo morreu por consumo de memória durante a madrugada e ninguém percebeu até
o expediente seguinte; uma atualização de versão derrubou o site por 40 minutos porque o deploy era
"parar o processo, trocar os arquivos, subir de novo"; e uma promoção relâmpago gerou um pico de
tráfego que o servidor único não absorveu.

A diretoria de tecnologia contratou você como **Cloud Developer** com uma missão de curto prazo:
provar, em um ambiente local reproduzível (Minikube), que a plataforma Kubernetes resolve os três
problemas — **auto-recuperação**, **atualização sem downtime** e **elasticidade**. A prova de
conceito precisa ser convincente o suficiente para justificar o investimento em um cluster
gerenciado na nuvem no trimestre seguinte (que será justamente o CP02).

Você recebeu o código-fonte do `store-api` (Python 3.12 + FastAPI) e liberdade total para decidir
como containerizar e implantar.

#### Requisitos funcionais

| ID | Requisito |
|---|---|
| RF01 | Containerizar o `store-api` com um `Dockerfile` próprio, baseado em imagem oficial `python:3.12-slim`, e publicar a imagem no Docker Hub sob a sua conta, com **duas tags**: `1.0` e `1.1`. |
| RF02 | A versão `1.1` deve conter uma diferença **visível em runtime** (ex.: o endpoint `/info` passa a retornar um campo `versao: "1.1"`), para que o rolling update seja demonstrável. |
| RF03 | Implantar o `store-api` por meio de um `Deployment` (`apps/v1`) com **3 réplicas**, labels `app: store-api`, `tier: backend`, `env: dev`. |
| RF04 | Expor a aplicação internamente por um `Service` do tipo `ClusterIP` na porta 80 mapeando para a porta 8000 do container. |
| RF05 | Expor a aplicação externamente por um `Service` do tipo `NodePort` (ou `minikube service`), permitindo acesso pelo navegador do host. |
| RF06 | Configurar as três probes: `readinessProbe` em `/ready`, `livenessProbe` em `/health` e `startupProbe` em `/health`, com `initialDelaySeconds`, `periodSeconds` e `failureThreshold` justificados no README. |
| RF07 | Injetar as variáveis `APP_ENV=dev` e `DB_HOST=store-db` via `env`, e criar um `ConfigMap` com pelo menos dois pares chave/valor consumidos pelo Pod via `envFrom`. |
| RF08 | Executar um **rolling update** de `1.0` para `1.1` e um **rollback** para `1.0`, evidenciando ambos com `kubectl rollout history` e `kubectl rollout undo`. |
| RF09 | Configurar um `HorizontalPodAutoscaler` (`autoscaling/v2`) com base em CPU, `minReplicas: 2`, `maxReplicas: 8`, alvo de 60% de utilização, e provocar o escalonamento com um gerador de carga. |
| RF10 | Demonstrar auto-recuperação: excluir manualmente um Pod (`kubectl delete pod`) e mostrar o ReplicaSet recriando-o. |

#### Requisitos não funcionais

| ID | Requisito |
|---|---|
| RNF01 | Todos os manifestos devem ser **declarativos** e aplicáveis com `kubectl apply -f k8s/`. Nenhum recurso pode existir apenas por comando imperativo (`kubectl run`, `kubectl expose`). |
| RNF02 | O container **não pode rodar como root**: `securityContext.runAsNonRoot: true` e um usuário não privilegiado criado no Dockerfile. |
| RNF03 | Todo container deve declarar `resources.requests` e `resources.limits` de CPU e memória. Sem requests de CPU o HPA não funciona — explique isso no README. |
| RNF04 | A estratégia de atualização deve ser `RollingUpdate` com `maxUnavailable: 0` e `maxSurge: 1`, garantindo **zero downtime**; comprove com um laço de `curl` durante o update, mostrando 100% de HTTP 200. |
| RNF05 | A imagem publicada deve ter menos de 300 MB (`docker images`). Justifique as escolhas de otimização (multi-stage ou `--no-cache-dir`). |
| RNF06 | Nenhuma senha em texto claro nos manifestos versionados. Se precisar de `DB_PASSWORD` neste Check Point, use um `Secret` criado por comando e **não versionado**, documentando o comando no README. |
| RNF07 | O repositório deve ter `.gitignore` adequado e nenhum arquivo binário desnecessário (imagens de container, `venv/`, `__pycache__/`). |
| RNF08 | O README deve permitir que uma pessoa que nunca viu o projeto reproduza tudo do zero em menos de 15 minutos. |

### CP01.3 Entregáveis

#### Estrutura obrigatória do repositório

```
fiap-store-k8s-<rm>/
├── README.md
├── .gitignore
├── app/
│   ├── main.py
│   ├── requirements.txt
│   └── Dockerfile
├── k8s/
│   ├── 00-configmap.yaml
│   ├── 10-store-api-deployment.yaml
│   ├── 20-store-api-service-clusterip.yaml
│   ├── 21-store-api-service-nodeport.yaml
│   └── 30-store-api-hpa.yaml
├── scripts/
│   ├── build-push.sh
│   ├── carga.sh
│   └── verifica.sh
└── docs/
    ├── evidencias/
    │   ├── 01-pods-running.png
    │   ├── 02-rolling-update.png
    │   ├── 03-rollback.png
    │   ├── 04-hpa-escalando.png
    │   └── 05-autorecuperacao.png
    └── video.md
```

#### Conteúdo obrigatório do README.md

1. **Identificação**: nome completo, RM, turma.
2. **Descrição do problema** em até 10 linhas, com as próprias palavras.
3. **Pré-requisitos** com versões (`minikube version`, `kubectl version --client`, `docker --version`).
4. **Passo a passo de reprodução**, com todos os comandos em blocos copiáveis, na ordem exata.
5. **Tabela de recursos criados**: kind, nome, namespace, função em uma linha.
6. **Justificativa das probes**: por que esses valores de `initialDelaySeconds`/`periodSeconds`; o
   que aconteceria se `livenessProbe` apontasse para `/ready`.
7. **Justificativa de requests/limits**: como você chegou nos números.
8. **Evidências**: cada print incorporado com legenda explicando o que se observa.
9. **Link do vídeo**.
10. **Dificuldades encontradas** e como foram resolvidas (mínimo 3, honestas).
11. **Como destruir tudo**: `kubectl delete -f k8s/` e `minikube delete`.

#### Vídeo de demonstração

**5 a 7 minutos**, roteiro sugerido:

| Tempo | Conteúdo |
|---|---|
| 0:00–0:30 | Identificação (nome, RM, turma) e visão geral do que será mostrado |
| 0:30–1:30 | `kubectl get all` e explicação do que existe no cluster |
| 1:30–2:30 | Acesso à aplicação pelo navegador via NodePort, mostrando `/produtos` e `/info` |
| 2:30–4:00 | Rolling update de 1.0 para 1.1 com o laço de `curl` rodando em paralelo, provando zero downtime |
| 4:00–5:00 | Rollback e `kubectl rollout history` |
| 5:00–6:30 | Geração de carga e HPA escalando (`kubectl get hpa -w`) |
| 6:30–7:00 | Deleção de um Pod e auto-recuperação |

Vídeo sem áudio explicativo, ou que apenas mostra a tela sem narrar o raciocínio, perde metade dos
pontos do critério "Demonstração".

### CP01.4 Rubrica de avaliação (100 pontos)

| Critério | Peso | Insuficiente (0–25%) | Regular (26–55%) | Bom (56–85%) | Excelente (86–100%) |
|---|---|---|---|---|---|
| **C1. Containerização e imagem** | 15 | Não há Dockerfile próprio, ou a imagem não sobe; usa imagem de terceiros pronta | Dockerfile funciona, mas roda como root, sem pin de versão, imagem acima de 600 MB | Dockerfile correto, usuário não-root, versão fixada, imagem abaixo de 300 MB, duas tags publicadas | Além do "Bom": multi-stage ou otimização documentada, `.dockerignore`, healthcheck, imagem abaixo de 180 MB, justificativa técnica das camadas |
| **C2. Deployment e Pods** | 20 | Não aplica, ou Pods em `CrashLoopBackOff`/`ImagePullBackOff` | Deployment sobe com 3 réplicas, mas sem labels padronizados, sem `resources`, `strategy` default não justificada | 3 réplicas Running, labels corretos, requests/limits definidos, `RollingUpdate` com `maxUnavailable: 0` | Além do "Bom": `revisionHistoryLimit`, `terminationGracePeriodSeconds` ajustado, anti-afinidade ou `topologySpreadConstraints`, comentários no YAML |
| **C3. Services e acesso** | 10 | Aplicação inacessível | Apenas um Service, acesso só por `port-forward` | ClusterIP + NodePort funcionando, portas coerentes, `targetPort` nomeado | Além do "Bom": Service headless adicional ou explicação documentada de quando usar cada tipo, teste de resolução DNS interna comprovado |
| **C4. Probes e ciclo de vida** | 15 | Sem probes | Apenas uma probe, valores copiados sem entendimento | As três probes configuradas, endpoints corretos, valores plausíveis | Além do "Bom": demonstração ativa de falha de readiness retirando o Pod do Endpoints, justificativa escrita da diferença liveness vs readiness vs startup |
| **C5. Rolling update e rollback** | 15 | Não demonstrado | Update feito, mas com downtime ou sem evidência | Update e rollback demonstrados, `rollout history` mostrado | Além do "Bom": prova de zero downtime com laço de requisições contando 0 erros, uso de `--record` ou `kubernetes.io/change-cause`, `rollout status` com timeout |
| **C6. HPA e elasticidade** | 15 | Sem HPA | HPA criado mas com `<unknown>` em métricas (metrics-server não habilitado) | HPA `autoscaling/v2` funcional, escalonamento observado sob carga | Além do "Bom": gráfico ou tabela de réplicas x carga x tempo, discussão de `stabilizationWindowSeconds` e do comportamento de scale-down |
| **C7. Documentação (README)** | 10 | Ausente ou com uma linha | Existe, mas não reproduz o ambiente; sem evidências | Completo, reproduzível, com evidências legendadas | Além do "Bom": diagrama da arquitetura, seção de troubleshooting própria, decisões técnicas argumentadas |
| **Total** | **100** | | | | |

**Cálculo por critério:** nota do critério = peso x (percentual do nível atingido). O professor
registra o percentual, não apenas o rótulo. Exemplo: C2 avaliado como "Bom, 70%" vale 14 pontos.

#### Penalidades e bônus fixos

| Situação | Efeito |
|---|---|
| Vídeo ausente | −20 pontos |
| Vídeo fora da faixa de duração (menos de 3 min ou mais de 10 min) | −5 pontos |
| Estrutura de pastas fora do padrão | −5 pontos |
| Segredo em texto claro versionado | −10 pontos |
| Recurso criado apenas imperativamente, sem YAML correspondente | −5 pontos por recurso |
| Bônus: `kustomization.yaml` funcional agrupando os manifestos | +3 pontos |
| Bônus: `store-front` (Nginx) também implantado e apontando para a API | +3 pontos |

O total após bônus é limitado a 100 pontos.

### CP01.5 Roteiro de correção rápida (5 minutos por aluno)

O professor clona o repositório, roda o script abaixo e observa. A entrega que não sobrevive a este
roteiro não pode receber conceito acima de "Regular" nos critérios C2 e C3.

```bash
# 0. Preparação (uma vez por sessão de correção)
minikube delete && minikube start --cpus=4 --memory=6g
minikube addons enable metrics-server

# 1. Clonar e posicionar na tag da entrega
git clone <url-do-aluno> aluno && cd aluno
git checkout cp01

# 2. Aplicar tudo de uma vez - deve funcionar sem edição manual
kubectl apply -f k8s/

# 3. O Deployment converge?
kubectl rollout status deployment/store-api --timeout=120s
kubectl get deploy,rs,pods -o wide

# 4. Labels e réplicas conforme o RF03
kubectl get pods -l app=store-api,tier=backend --show-labels

# 5. Probes declaradas?
kubectl get deploy store-api -o jsonpath='{.spec.template.spec.containers[0].livenessProbe}'; echo
kubectl get deploy store-api -o jsonpath='{.spec.template.spec.containers[0].readinessProbe}'; echo
kubectl get deploy store-api -o jsonpath='{.spec.template.spec.containers[0].startupProbe}'; echo

# 6. Requests e limits (RNF03)
kubectl get deploy store-api -o jsonpath='{.spec.template.spec.containers[0].resources}'; echo

# 7. Estrategia de update (RNF04)
kubectl get deploy store-api -o jsonpath='{.spec.strategy}'; echo

# 8. Nao roda como root (RNF02)
kubectl get deploy store-api -o jsonpath='{.spec.template.spec.securityContext}'; echo
kubectl exec deploy/store-api -- id

# 9. Services respondem?
kubectl get svc
kubectl run curl-teste --rm -it --image=curlimages/curl:8.10.1 --restart=Never -- \
  curl -s http://store-api/produtos

# 10. HPA vivo (nao pode estar <unknown>)
kubectl get hpa store-api

# 11. Auto-recuperacao
kubectl delete pod -l app=store-api --field-selector=status.phase=Running --wait=false | head -1
sleep 15 && kubectl get pods -l app=store-api

# 12. Rolling update real
kubectl set image deploy/store-api store-api=<usuario>/store-api:1.1
kubectl rollout status deploy/store-api --timeout=120s
kubectl rollout history deploy/store-api
kubectl rollout undo deploy/store-api
```

**Checklist binário do professor** (cada item é sim/não, preenchido durante a execução acima):

| # | Verificação | OK |
|---|---|---|
| 1 | `kubectl apply -f k8s/` funciona sem erro e sem edição | |
| 2 | Deployment atinge 3/3 Ready em menos de 120 s | |
| 3 | Labels `app`, `tier`, `env` presentes | |
| 4 | As três probes declaradas com endpoints corretos | |
| 5 | `resources.requests.cpu` presente | |
| 6 | `maxUnavailable: 0` na estratégia | |
| 7 | `id` dentro do container retorna uid diferente de 0 | |
| 8 | Service ClusterIP resolve por DNS e responde `/produtos` | |
| 9 | HPA mostra métrica numérica, não `<unknown>` | |
| 10 | Pod deletado é recriado automaticamente | |
| 11 | `rollout undo` volta para a versão anterior | |
| 12 | README reproduz o ambiente sem intervenção do corretor | |

### CP01.6 Política de plágio, uso de IA e atrasos

**Uso de IA generativa: permitido e regulado.** A disciplina forma Cloud Developers que usarão
assistentes de IA no mercado. O que se avalia é **compreensão**, não digitação. Portanto:

- É permitido usar IA para gerar rascunhos de YAML, Dockerfile e scripts.
- É **obrigatório** declarar o uso no README, em uma seção `## Uso de IA`, informando quais
  ferramentas foram usadas e para quais partes.
- É **proibido** entregar código que o aluno não sabe explicar. Durante a correção o professor pode
  fazer até três perguntas orais/escritas sobre qualquer trecho da entrega. Não saber explicar
  implica **nota zero no critério correspondente**, e, se generalizado, nota zero na entrega.
- Comentários automáticos genéricos ("# This creates a deployment") sem valor explicativo não
  contam como documentação.

**Plágio entre alunos.** Como o Check Point é individual, entregas com manifestos idênticos
(incluindo comentários, ordem de campos e erros iguais) recebem **zero para todos os envolvidos**,
independentemente de quem copiou. Discutir a solução é permitido; compartilhar arquivos não é.
Indicadores usados: `diff` entre repositórios, histórico de commits (repositório com um único commit
"initial commit" contendo tudo pronto é sinal de alerta), e metadados das imagens no Docker Hub.

**Atrasos.**

| Atraso | Penalidade |
|---|---|
| Até 24 h | −10% da nota obtida |
| 24 h a 48 h | −25% da nota obtida |
| 48 h a 72 h | −40% da nota obtida |
| Acima de 72 h | Não aceito; encaminhar para a recuperação (CP01.8) |

Problemas de saúde ou trabalho comprovados devem ser comunicados **antes** do prazo, com documento,
para prorrogação sem penalidade.

### CP01.7 Variações alternativas de tema

**Variação A — FIAP Ticket (bilheteria de eventos).** Substitua a loja por um sistema de venda de
ingressos com o serviço `ticket-api` (FastAPI, endpoints `/eventos`, `/lote`, `/health`, `/ready`).
O gancho narrativo é a venda de ingressos de um show que abre às 10h e esgota em 12 minutos: o
sistema precisa suportar um pico de 40x o tráfego normal em poucos segundos e voltar ao normal
depois. Isso deixa o HPA mais protagonista que no tema original, permitindo explorar
`behavior.scaleUp.policies` com `periodSeconds` curto e `stabilizationWindowSeconds: 0`, além de
discutir por que scale-down deve ser lento. O rolling update é motivado por uma correção emergencial
de bug no cálculo de taxa de conveniência, aplicada com a fila de compra aberta.

**Variação B — FIAP Health (agendamento de consultas).** O serviço `agenda-api` expõe
`/consultas`, `/medicos`, `/health` e `/ready`, e a narrativa é a de uma clínica que não pode
perder agendamentos durante manutenções. O diferencial pedagógico é focar em **readiness e
graceful shutdown**: o aluno deve implementar tratamento de `SIGTERM` no FastAPI (fechando conexões
antes de sair), configurar `terminationGracePeriodSeconds` coerente e um `preStop` com `sleep 5`,
provando com o laço de `curl` que nenhuma requisição em voo é perdida durante o rolling update. A
prova de zero downtime deixa de ser acidental e passa a ser projetada.

**Variação C — FIAP Edu (portal de notas).** O `notas-api` serve boletins e tem uma característica
peculiar: leva 25 segundos para iniciar, porque carrega um arquivo grande de índice na memória. O
tema central passa a ser o **startupProbe** e o entendimento do ciclo de vida do Pod. O aluno deve
demonstrar o que acontece quando se usa apenas `livenessProbe` com `initialDelaySeconds`
insuficiente (o clássico `CrashLoopBackOff` eterno, que deve ser reproduzido de propósito, filmado e
depois corrigido). Acrescente a exigência de um `initContainer` que valida a presença do arquivo de
índice antes de o container principal subir, tornando o CP01 um bom laboratório de diagnóstico.

### CP01.8 Barema de recuperação e plano de contingência

#### Se o aluno não conseguiu rodar o Minikube

O CP01 é intencionalmente **100% local e sem custo**, mas máquinas com pouca memória ou restrições
corporativas podem inviabilizá-lo. Alternativas aceitas, em ordem de preferência:

| Alternativa | Como usar | Observação |
|---|---|---|
| `kind` (Kubernetes in Docker) | `kind create cluster --name fiap` | Aceito integralmente; documente a troca no README. NodePort exige `extraPortMappings` no config do kind |
| `k3d` | `k3d cluster create fiap -p "30080:30080@server:0"` | Aceito integralmente; mais leve que Minikube |
| Killercoda / Play with Kubernetes | Sessões web temporárias | Aceito com ressalva: sessão expira; grave o vídeo em uma sessão só |
| Codespaces / Cloud IDE com kind | GitHub Codespaces 4 cores | Aceito; anexe print do ambiente |
| Cluster do laboratório da faculdade | Namespace dedicado por aluno | Aceito; NodePort pode estar bloqueado, use `port-forward` e documente |

Nenhuma dessas alternativas reduz a nota. O que reduz nota é **não entregar** sem ter comunicado a
dificuldade com pelo menos 72 h de antecedência.

#### Recuperação (para quem ficou abaixo de 60 pontos)

A recuperação **não é uma nova entrega do zero**: é a correção dirigida da entrega original.

1. O professor devolve a rubrica preenchida com os percentuais e um comentário por critério
   reprovado.
2. O aluno tem **5 dias corridos** para corrigir apenas os critérios abaixo de 56%.
3. A entrega da recuperação é um novo commit + tag `cp01-rec`, mais um **vídeo curto de 3 minutos**
   mostrando somente os pontos corrigidos.
4. A nota de recuperação é limitada a **70 pontos** (o teto evita que a recuperação seja mais
   vantajosa que a entrega no prazo).
5. Critérios já avaliados como "Bom" ou "Excelente" são preservados integralmente; o aluno não pode
   perder pontos na recuperação.

#### Barema simplificado de recuperação

| Item verificado na recuperação | Pontos |
|---|---|
| `kubectl apply -f k8s/` sobe o Deployment com 3 réplicas Ready | 25 |
| Service acessível e respondendo `/produtos` | 15 |
| As três probes declaradas e coerentes | 15 |
| Rolling update para 1.1 e rollback demonstrados no vídeo | 15 |
| HPA presente com métricas funcionando | 15 |
| README reproduzível com evidências | 15 |
| **Total (teto de 70 aplicado depois)** | **100** |

---

## Check Point 02 — Operação em cluster multi-nó: rede, namespaces e armazenamento

### CP02.1 Enquadramento

| Item | Valor |
|---|---|
| Ementa coberta | Capítulos 5 a 7: Networking e Namespaces; criação de cluster na nuvem (múltiplos nós / EKS); gerenciamento de recursos (volumes) e daemons |
| Momento de aplicação | Enunciado liberado na aula 7; entrega antes da aula 9 |
| Modalidade | **Dupla** (o custo e a complexidade do cluster na nuvem justificam o trabalho compartilhado; ambos devem ter acesso ao console AWS) |
| Prazo sugerido | **14 dias corridos** |
| Carga horária estimada | 14 a 18 horas por dupla |
| Foco | Operação real: multi-nó, isolamento por namespace, Ingress, NetworkPolicy, persistência, DaemonSet |
| Infraestrutura exigida | EKS (preferencial) **ou** k3d/kind multi-nó (alternativa integral, ver CP02.8) |

> **Atenção (custos):** um cluster EKS cobra a taxa de control plane por hora, mais as instâncias
> EC2 dos nós, mais o NLB/ALB criado pelo Ingress, mais os volumes EBS dos PVCs. A dupla deve
> **destruir o cluster** (`eksctl delete cluster`) imediatamente após gravar o vídeo e incluir no
> README o print do Cost Explorer com o gasto total. Cluster esquecido ligado é problema recorrente:
> o relatório de custos vale pontos justamente para criar esse hábito.

### CP02.2 Enunciado principal

#### Cenário

A prova de conceito do CP01 convenceu a diretoria. A FIAP Store aprovou a migração para um cluster
Kubernetes gerenciado e, junto com ela, três demandas novas que o Minikube não conseguia atender:

1. **Ambientes separados.** Hoje desenvolvimento e produção compartilham o mesmo banco, e um
   `TRUNCATE` mal executado em desenvolvimento apagou o catálogo de produtos em produção. A
   diretoria exige isolamento real: `store-dev` e `store-prod` não podem se enxergar.

2. **Um endereço único e HTTPS.** Os clientes acessam hoje `http://ip-do-servidor:30080`. Marketing
   quer `loja.fiapstore.io` para o front, `api.fiapstore.io` para a API, com TLS. Abrir uma porta
   nova por serviço não escala.

3. **Dados que sobrevivem.** O catálogo de produtos precisa sair do JSON em memória e ir para um
   PostgreSQL cujos dados **sobrevivem à morte do Pod e à troca do nó**. A auditoria também passou a
   exigir que os logs de todos os nós sejam coletados de forma centralizada.

Você e sua dupla são a equipe de plataforma. A entrega é o cluster multi-nó com a FIAP Store rodando
em dois ambientes isolados, com entrada única, rede restrita e dados persistentes.

#### Requisitos funcionais

| ID | Requisito |
|---|---|
| RF01 | Provisionar um cluster com **no mínimo 2 nós worker**: EKS via `eksctl` (preferencial) ou k3d/kind com 1 server + 2 agents. O provisionamento deve ser reproduzível a partir de um arquivo versionado (`infra/cluster.yaml` do eksctl ou `infra/k3d-config.yaml`). |
| RF02 | Criar os namespaces `store-dev` e `store-prod`, cada um com `ResourceQuota` (limite de CPU, memória e número de Pods) e `LimitRange` (default de requests/limits por container). |
| RF03 | Implantar a stack completa nos **dois** namespaces: `store-api` (Deployment), `store-front` (Nginx, Deployment), `store-db` (PostgreSQL 16). Em `store-dev`, 1 réplica de API; em `store-prod`, 3 réplicas. |
| RF04 | O `store-db` deve ser um **StatefulSet** com `volumeClaimTemplates`, Service headless e `storageClassName` explícito (`gp3` na AWS, `local-path` no k3d). |
| RF05 | Comprovar persistência: inserir dados na tabela, deletar o Pod do banco, aguardar o novo Pod e mostrar os dados intactos. |
| RF06 | Instalar um Ingress Controller (NGINX Ingress ou AWS Load Balancer Controller) e criar `Ingress` com **roteamento por host e por path**: `loja.<dominio>` para o front e `api.<dominio>/produtos` para a API. |
| RF07 | Habilitar **TLS** no Ingress. Aceita-se certificado auto-assinado gerado por `openssl` e armazenado em `Secret` do tipo `kubernetes.io/tls`, cert-manager com issuer self-signed, ou ACM na AWS. |
| RF08 | Aplicar `NetworkPolicy` implementando: (a) negação padrão de ingresso em `store-prod`; (b) liberação apenas de `store-front` e do Ingress Controller para `store-api`; (c) liberação apenas de `store-api` para `store-db` na porta 5432; (d) bloqueio total de tráfego entre `store-dev` e `store-prod`. |
| RF09 | Implantar um **DaemonSet** de coleta de logs (Fluent Bit) que roda em todos os nós, monta `/var/log` do host como `hostPath` e escreve a saída em stdout ou em um destino simples. Comprovar com um Pod por nó. |
| RF10 | Demonstrar **distribuição entre nós**: os Pods de `store-api` em `store-prod` devem estar espalhados nos nós, usando `topologySpreadConstraints` ou `podAntiAffinity`. Comprovar com `kubectl get pods -o wide`. |
| RF11 | Comprovar o comportamento de manutenção de nó: `kubectl cordon` + `kubectl drain` em um nó, mostrando os Pods sendo reagendados e a aplicação continuando a responder. |
| RF12 | Usar um `PodDisruptionBudget` para `store-api` em `store-prod` (`minAvailable: 2`) e explicar seu efeito durante o drain. |

#### Requisitos não funcionais

| ID | Requisito |
|---|---|
| RNF01 | Manifestos organizados por ambiente. Use **Kustomize** com `base/` e `overlays/dev`, `overlays/prod`, ou Helm com dois `values` — duplicar YAML por copiar-colar custa pontos. |
| RNF02 | A senha do PostgreSQL deve vir de um `Secret`, nunca de `env` em texto claro. O arquivo do Secret real não é versionado; versiona-se um `secret.example.yaml`. |
| RNF03 | O acesso ao banco deve ser feito exclusivamente pelo Service DNS interno (`store-db.store-prod.svc.cluster.local`), nunca por IP de Pod. |
| RNF04 | O cluster deve ser **destruível por um comando documentado**, e o README deve conter o custo total apurado. |
| RNF05 | Nenhum Pod pode rodar como root, salvo justificativa explícita e documentada (o Fluent Bit pode precisar; explique o porquê e restrinja as capabilities). |
| RNF06 | Todo Deployment deve ter `resources`, probes e labels padronizados herdados do CP01. Regressão nesses pontos é penalizada. |
| RNF07 | A aplicação em `store-prod` deve continuar respondendo (HTTP 200) durante o drain do nó. Comprove com laço de `curl`. |
| RNF08 | O diagrama de arquitetura no README deve mostrar nós, namespaces, Services, Ingress e o caminho do tráfego. Pode ser Mermaid, draw.io exportado ou ASCII. |

### CP02.3 Entregáveis

#### Estrutura obrigatória do repositório (evolução do CP01)

```
fiap-store-k8s-<rm1>-<rm2>/
├── README.md
├── infra/
│   ├── cluster.yaml              # eksctl ClusterConfig ou k3d config
│   ├── criar-cluster.sh
│   └── destruir-cluster.sh
├── app/
│   ├── store-api/                # herdado do CP01
│   └── store-front/
│       ├── Dockerfile
│       └── html/index.html
├── k8s/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── store-api-deployment.yaml
│   │   ├── store-api-service.yaml
│   │   ├── store-front-deployment.yaml
│   │   ├── store-front-service.yaml
│   │   ├── store-db-statefulset.yaml
│   │   ├── store-db-service-headless.yaml
│   │   └── ingress.yaml
│   ├── overlays/
│   │   ├── dev/
│   │   │   ├── kustomization.yaml
│   │   │   ├── namespace.yaml
│   │   │   ├── resourcequota.yaml
│   │   │   └── limitrange.yaml
│   │   └── prod/
│   │       ├── kustomization.yaml
│   │       ├── namespace.yaml
│   │       ├── resourcequota.yaml
│   │       ├── limitrange.yaml
│   │       ├── pdb.yaml
│   │       └── replicas-patch.yaml
│   ├── network-policies/
│   │   ├── default-deny-prod.yaml
│   │   ├── allow-front-to-api.yaml
│   │   ├── allow-api-to-db.yaml
│   │   └── deny-cross-namespace.yaml
│   ├── daemonset/
│   │   └── fluent-bit-daemonset.yaml
│   └── secrets/
│       └── db-secret.example.yaml
├── scripts/
│   ├── teste-networkpolicy.sh
│   ├── teste-persistencia.sh
│   ├── teste-drain.sh
│   └── verifica.sh
└── docs/
    ├── arquitetura.md
    ├── custos.md
    └── evidencias/
```

#### Conteúdo obrigatório do README.md

Além de tudo que era exigido no CP01 (identificação — agora dos dois integrantes — pré-requisitos,
reprodução, evidências, vídeo, dificuldades):

1. **Diagrama da arquitetura** com nós, namespaces, Ingress, Services e o caminho de uma requisição
   externa até o banco.
2. **Tabela de divisão de trabalho** entre os dois integrantes, item a item. Duplas em que um
   integrante não tem nenhum commit no histórico Git são investigadas.
3. **Matriz de NetworkPolicy** documentando o resultado dos testes:

   | Origem | Destino | Porta | Esperado | Obtido |
   |---|---|---|---|---|
   | `store-front` (prod) | `store-api` (prod) | 8000 | Permitido | |
   | Pod avulso (prod) | `store-api` (prod) | 8000 | Bloqueado | |
   | `store-api` (prod) | `store-db` (prod) | 5432 | Permitido | |
   | `store-api` (dev) | `store-db` (prod) | 5432 | Bloqueado | |
   | Pod avulso (dev) | `store-api` (prod) | 8000 | Bloqueado | |

4. **Relatório de custos** (`docs/custos.md`): tempo de cluster ligado, recursos criados, custo por
   item, custo total, print do Cost Explorer ou do Billing. Duplas em k3d preenchem com uma
   **estimativa** do que custaria em EKS pelo Pricing Calculator (o exercício de FinOps é obrigatório
   mesmo para quem não usou nuvem).
5. **Procedimento de destruição** verificado, com o print do console mostrando o cluster ausente.

#### Vídeo de demonstração

**8 a 10 minutos**:

| Tempo | Conteúdo |
|---|---|
| 0:00–0:40 | Identificação da dupla e visão da arquitetura no diagrama |
| 0:40–1:40 | `kubectl get nodes -o wide` e `kubectl get all -n store-prod` |
| 1:40–3:00 | Acesso via Ingress pelos dois hosts, com HTTPS e cadeado (ou aviso de auto-assinado explicado) |
| 3:00–4:30 | Teste de NetworkPolicy: Pod permitido conectando e Pod bloqueado dando timeout |
| 4:30–6:00 | Teste de persistência: INSERT, `delete pod`, SELECT com os dados intactos |
| 6:00–7:30 | Cordon + drain de um nó, com o `curl` em laço provando disponibilidade, e o PDB atuando |
| 7:30–8:30 | DaemonSet: um Pod por nó e trecho de log coletado |
| 8:30–9:30 | Destruição do cluster e comentário sobre o custo apurado |

### CP02.4 Rubrica de avaliação (100 pontos)

| Critério | Peso | Insuficiente (0–25%) | Regular (26–55%) | Bom (56–85%) | Excelente (86–100%) |
|---|---|---|---|---|---|
| **C1. Cluster multi-nó e reprodutibilidade** | 12 | Cluster de nó único ou não comprovado | Cluster multi-nó criado pelo console/manualmente, sem arquivo versionado | Cluster com 2+ nós criado a partir de arquivo versionado, script de criação e destruição funcionando | Além do "Bom": node groups com tipos distintos ou labels/taints com propósito, versão do K8s fixada, cluster autoscaler ou justificativa da capacidade dimensionada |
| **C2. Namespaces, quotas e organização** | 12 | Tudo em `default` | Namespaces criados, sem quota ou limit range | `store-dev` e `store-prod` com ResourceQuota e LimitRange coerentes, Kustomize/Helm sem duplicação | Além do "Bom": demonstração de quota sendo atingida (Pod recusado por falta de cota), overlays enxutos com patches estratégicos |
| **C3. Ingress e TLS** | 15 | Sem Ingress; acesso por NodePort | Ingress instalado, um host só, sem TLS | Ingress com roteamento por host e path, TLS funcionando, controller documentado | Além do "Bom": anotações relevantes justificadas (rewrite, timeouts, `proxy-body-size`), redirecionamento HTTP→HTTPS, cert-manager automatizando emissão |
| **C4. NetworkPolicy** | 15 | Ausente | Policies escritas mas sem efeito (CNI sem suporte, não testado) | Default-deny + regras específicas funcionando, com testes evidenciando permitido e bloqueado | Além do "Bom": matriz completa preenchida com 5+ casos, políticas de egress além de ingress, discussão do comportamento aditivo das policies |
| **C5. Persistência (StatefulSet/PVC)** | 15 | Banco sem volume, dados perdidos | PVC criado mas com Deployment, ou persistência não comprovada | StatefulSet com volumeClaimTemplates, Service headless, persistência comprovada em vídeo | Além do "Bom": discussão de StorageClass e `reclaimPolicy`, teste de sobrevivência a troca de nó (não só de Pod), backup lógico do banco demonstrado |
| **C6. DaemonSet e observação de nós** | 10 | Ausente | DaemonSet aplicado mas com Pods em erro | Fluent Bit rodando em todos os nós, logs visíveis, tolerations coerentes | Além do "Bom": tolerations para nós de controle explicadas, `resources` ajustados, filtro/parser configurado, saída estruturada |
| **C7. Alta disponibilidade e manutenção de nó** | 11 | Não demonstrado | Drain executado mas com indisponibilidade ou sem PDB | Cordon/drain demonstrados, PDB ativo, spread entre nós comprovado | Além do "Bom": prova quantitativa de zero erro durante o drain, análise do que aconteceria com `minAvailable` mal configurado (drain travado) |
| **C8. Documentação, custos e trabalho em dupla** | 10 | README herdado do CP01 sem atualização | Documentação parcial, sem diagrama ou sem custos | Diagrama, matriz de policies, custos apurados, divisão de trabalho clara, commits dos dois | Além do "Bom": análise crítica de custo com propostas de redução (Spot, right-sizing, Fargate), runbook de operação |
| **Total** | **100** | | | | |

#### Penalidades e bônus fixos

| Situação | Efeito |
|---|---|
| Vídeo ausente | −20 pontos |
| Cluster não destruído / evidência de destruição ausente | −10 pontos |
| Senha do banco em texto claro versionada | −15 pontos |
| Regressão de qualidade do CP01 (probes ou resources removidos) | −5 pontos por item |
| Um dos integrantes sem nenhum commit | Nota individual reduzida em 50% para o integrante ausente |
| Bônus: `cert-manager` com emissão automática | +3 pontos |
| Bônus: métricas do cluster com Prometheus + Grafana e um dashboard | +5 pontos |
| Bônus: uso de Amazon ECR em vez de Docker Hub, com política de lifecycle | +3 pontos |

### CP02.5 Roteiro de correção rápida (5 a 8 minutos por dupla)

```bash
# 0. Contexto: o professor usa o cluster da propria dupla, via sessao gravada,
#    ou recria em k3d local com os manifestos entregues.
k3d cluster create corretor --servers 1 --agents 2 \
  -p "8080:80@loadbalancer" -p "8443:443@loadbalancer"

git clone <url-da-dupla> dupla && cd dupla && git checkout cp02

# 1. Multi-no
kubectl get nodes -o wide

# 2. Namespaces, quotas e limites
kubectl get ns store-dev store-prod
kubectl describe quota -n store-prod
kubectl describe limitrange -n store-prod

# 3. Deploy dos dois ambientes via Kustomize (nao pode exigir edicao manual)
kubectl apply -k k8s/overlays/dev
kubectl apply -k k8s/overlays/prod
kubectl get all -n store-dev
kubectl get all -n store-prod

# 4. Replicas diferentes por ambiente (RF03)
kubectl get deploy store-api -n store-dev  -o jsonpath='{.spec.replicas}'; echo
kubectl get deploy store-api -n store-prod -o jsonpath='{.spec.replicas}'; echo

# 5. Distribuicao entre nos (RF10)
kubectl get pods -n store-prod -l app=store-api -o wide
kubectl get deploy store-api -n store-prod \
  -o jsonpath='{.spec.template.spec.topologySpreadConstraints}'; echo

# 6. StatefulSet e PVC (RF04)
kubectl get statefulset,pvc,pv -n store-prod
kubectl get sc

# 7. Persistencia real (RF05)
kubectl exec -n store-prod store-db-0 -- psql -U store -d storedb \
  -c "CREATE TABLE IF NOT EXISTS t(id int); INSERT INTO t VALUES (42);"
kubectl delete pod store-db-0 -n store-prod
kubectl wait --for=condition=ready pod/store-db-0 -n store-prod --timeout=180s
kubectl exec -n store-prod store-db-0 -- psql -U store -d storedb -c "SELECT * FROM t;"
# Esperado: 42

# 8. Ingress e TLS (RF06, RF07)
kubectl get ingress -A
kubectl get secret -n store-prod --field-selector type=kubernetes.io/tls
curl -k -sI -H "Host: api.fiapstore.io" https://localhost:8443/produtos | head -1
curl -k -sI -H "Host: loja.fiapstore.io" https://localhost:8443/ | head -1

# 9. NetworkPolicy - o caso PERMITIDO deve passar
kubectl run permitido -n store-prod --labels="app=store-front" --rm -it \
  --image=curlimages/curl:8.10.1 --restart=Never -- \
  curl -s --max-time 5 http://store-api:80/produtos

# 10. NetworkPolicy - o caso BLOQUEADO deve dar timeout (exit code 28)
kubectl run bloqueado -n store-prod --labels="app=intruso" --rm -it \
  --image=curlimages/curl:8.10.1 --restart=Never -- \
  curl -s --max-time 5 http://store-api:80/produtos ; echo "exit=$?"

# 11. Isolamento entre namespaces
kubectl run xns -n store-dev --rm -it --image=curlimages/curl:8.10.1 --restart=Never -- \
  curl -s --max-time 5 http://store-api.store-prod.svc.cluster.local/produtos ; echo "exit=$?"

# 12. DaemonSet - um Pod por no
kubectl get ds -A
kubectl get pods -A -l app=fluent-bit -o wide

# 13. PDB e drain
kubectl get pdb -n store-prod
kubectl cordon <no-worker-1>
kubectl drain <no-worker-1> --ignore-daemonsets --delete-emptydir-data --timeout=120s
kubectl get pods -n store-prod -o wide
kubectl uncordon <no-worker-1>

# 14. Nenhum segredo versionado
git log -p --all | grep -iE "password|passwd|secret" | grep -v example | head
```

**Checklist binário do professor:**

| # | Verificação | OK |
|---|---|---|
| 1 | Cluster com 2 ou mais nós comprovado | |
| 2 | `store-dev` e `store-prod` com ResourceQuota e LimitRange | |
| 3 | `kubectl apply -k` funciona nos dois overlays sem edição | |
| 4 | Réplicas diferentes entre dev e prod | |
| 5 | Pods de prod distribuídos em nós distintos | |
| 6 | StatefulSet com PVC Bound | |
| 7 | Dado sobrevive à deleção do Pod do banco | |
| 8 | Ingress responde nos dois hosts | |
| 9 | TLS presente com Secret do tipo `kubernetes.io/tls` | |
| 10 | Caso permitido da NetworkPolicy retorna 200 | |
| 11 | Caso bloqueado da NetworkPolicy dá timeout | |
| 12 | Tráfego dev → prod bloqueado | |
| 13 | DaemonSet com um Pod por nó | |
| 14 | PDB existe e drain respeita o mínimo | |
| 15 | Nenhuma senha real no histórico do Git | |
| 16 | Evidência de destruição do cluster e relatório de custos | |

### CP02.6 Política de plágio, uso de IA e atrasos

Valem as mesmas regras do CP01 (declaração obrigatória do uso de IA no README, proibição de entregar
o que não se sabe explicar, arguição de até três perguntas), com as adições específicas do trabalho
em dupla:

- **Responsabilidade solidária.** Os dois integrantes respondem pela entrega inteira. Na arguição, o
  professor escolhe qual dos dois responde cada pergunta. "Essa parte foi meu colega que fez" não é
  resposta aceita e implica perda dos pontos do critério para **quem não soube responder**.
- **Notas individualizadas.** A nota da dupla é a base, mas pode ser ajustada individualmente por
  evidência de participação (histórico de commits, arguição, tabela de divisão de trabalho).
- **Reuso do CP01.** Reaproveitar o próprio CP01 é esperado e desejável. Reaproveitar o CP01 de
  **outro aluno** sem crédito é plágio.
- **Copiar manifestos de tutoriais e da documentação oficial** é permitido, desde que a fonte esteja
  citada em comentário no próprio YAML (`# baseado em: https://...`) e desde que o aluno saiba
  explicar cada campo. YAML colado sem entender e com campos irrelevantes ao contexto (por exemplo,
  anotações de um cloud provider que não está em uso) é sinal de cópia acrítica e custa pontos em
  C3/C4.
- **Atrasos:** mesma tabela do CP01 (−10% / −25% / −40% / não aceito acima de 72 h). No CP02, um
  motivo aceito para prorrogação sem penalidade é a indisponibilidade comprovada de crédito AWS,
  **desde que comunicada antes do prazo** — nesse caso a dupla migra para o plano k3d do CP02.8.

### CP02.7 Variações alternativas de tema

**Variação A — Multi-tenant por namespace (FIAP Store como SaaS).** Em vez de dev e prod, os
namespaces representam **clientes** (`tenant-alfa`, `tenant-beta`, `tenant-gama`), cada um com sua
instância isolada da loja e do banco. O centro do trabalho passa a ser o isolamento: cada tenant
recebe ResourceQuota proporcional ao plano contratado (bronze/prata/ouro), NetworkPolicy de negação
total entre tenants, e um Ingress único que roteia por host (`alfa.fiapstore.io`,
`beta.fiapstore.io`). Acrescente a exigência de demonstrar o efeito "vizinho barulhento": um tenant
tentando consumir CPU acima da sua quota não pode degradar os demais, o que deve ser comprovado com
um Pod de stress e medições antes/durante/depois. É a variação mais próxima do mundo real de
produtos SaaS.

**Variação B — Migração com estado legado (dump e restore).** A narrativa é a migração de um
PostgreSQL que já roda fora do cluster e tem 500 MB de dados. A dupla recebe um arquivo `dump.sql` e
deve: criar o StatefulSet, restaurar o dump em um `initContainer` ou `Job` de migração, validar a
contagem de registros, e só então cortar o tráfego da aplicação para o novo banco. O diferencial é
exercitar `Job` de migração antes mesmo do capítulo 8, `PersistentVolume` dimensionado corretamente
(o que acontece se o PVC for menor que o dump?), `subPath`, e o procedimento de **expansão de
volume** (`allowVolumeExpansion: true` e edição do PVC). Bom para turmas com perfil mais próximo de
banco de dados e operação.

**Variação C — Zona de disponibilidade e falha de nó.** O foco desloca-se de rede para
**resiliência de infraestrutura**. O cluster deve ter nós em pelo menos duas AZs (`eksctl` com dois
node groups em zonas distintas, ou k3d com labels simulando `topology.kubernetes.io/zone`). A dupla
precisa usar `topologySpreadConstraints` com `topologyKey: topology.kubernetes.io/zone` e
`whenUnsatisfiable: DoNotSchedule`, demonstrar o que acontece quando uma AZ inteira "cai" (terminar
as instâncias de um node group, ou `k3d node stop`), medir o tempo até a recuperação total do
serviço e discutir por que um PVC EBS **não** migra sozinho entre AZs — introduzindo naturalmente a
discussão de armazenamento zonal versus regional (EFS). É a variação mais exigente tecnicamente.

### CP02.8 Barema de recuperação e plano de contingência

#### Se a dupla não tem acesso a nuvem

O CP02 foi desenhado para ser **integralmente realizável sem AWS**. A tabela abaixo mapeia cada
requisito para o equivalente local, e **nenhuma dessas substituições reduz nota**:

| Requisito | Na AWS (EKS) | Substituto local aceito |
|---|---|---|
| Cluster multi-nó | `eksctl create cluster --nodes 2` | `k3d cluster create fiap --servers 1 --agents 2` ou `kind` com config multi-node |
| Ingress Controller | AWS Load Balancer Controller | NGINX Ingress Controller ou Traefik (já embutido no k3d) |
| TLS | ACM | `openssl req -x509` + Secret `kubernetes.io/tls`, ou cert-manager com `selfSigned` issuer |
| StorageClass | `gp3` (EBS CSI) | `local-path` (k3d/Rancher) ou `standard` (kind) |
| Registro de imagens | Amazon ECR | Docker Hub ou registro local do k3d (`k3d registry create`) |
| Multi-AZ | Zonas reais | Labels manuais `topology.kubernetes.io/zone` com valores `a` ou `b` nos nós |
| Custos | Cost Explorer | Estimativa no AWS Pricing Calculator, obrigatória mesmo assim |
| NetworkPolicy | VPC CNI com policy ou Calico | k3d/kind com Calico ou Cilium instalado (o Flannel padrão **não** aplica policies) |

> **Atenção:** o erro mais comum desta contingência é escrever NetworkPolicies em um cluster cujo CNI
> as ignora silenciosamente. O aluno acha que bloqueou e não bloqueou. O teste do caso bloqueado
> (item 10 do roteiro de correção) é justamente o que revela isso. Instale Calico
> (`k3d cluster create --flannel-backend=none --disable-network-policy` + manifesto do Calico) ou
> Cilium antes de começar.

#### Créditos e alternativas de nuvem

| Opção | Observação |
|---|---|
| AWS Free Tier / conta pessoal | EKS **não** é free tier: há custo por hora de control plane. Ligar, gravar, destruir no mesmo dia |
| AWS Academy / Learner Lab | Sessões de 4 horas; planejar a gravação dentro da janela; alguns labs limitam tipos de instância |
| Créditos institucionais | Solicitar ao coordenador com 10 dias de antecedência |
| Outra nuvem (GKE/AKS/DigitalOcean) | Aceito com anotação no README explicando as diferenças em relação ao EKS |

#### Recuperação (abaixo de 60 pontos)

1. Devolutiva com a rubrica preenchida e os critérios a corrigir.
2. **7 dias corridos** para a correção dirigida (prazo maior que no CP01 porque pode envolver
   recriar o cluster).
3. Entrega: tag `cp02-rec` + vídeo de 4 minutos focado só nos itens corrigidos.
4. Nota limitada a **70 pontos**.
5. A dupla pode optar por refazer em k3d o que fez em EKS (ou vice-versa) sem penalidade.

#### Barema simplificado de recuperação

| Item verificado na recuperação | Pontos |
|---|---|
| Cluster com 2+ nós e Pods distribuídos | 15 |
| Namespaces `store-dev` e `store-prod` com quota | 15 |
| Ingress roteando por host, com TLS | 20 |
| NetworkPolicy com caso permitido e caso bloqueado comprovados | 20 |
| Persistência comprovada (dado sobrevive ao Pod) | 20 |
| DaemonSet com um Pod por nó | 10 |
| **Total (teto de 70 aplicado depois)** | **100** |

---

## Check Point 03 — Segurança, automação e entrega contínua com GitOps

### CP03.1 Enquadramento

| Item | Valor |
|---|---|
| Ementa coberta | Capítulos 8 a 11: Jobs, Secrets e Endpoints; autenticação e controle de usuário (RBAC); boas práticas de operação e segurança; GitOps com Argo CD e Rancher |
| Momento de aplicação | Enunciado liberado na aula 11; entrega antes da apresentação da Global Solution |
| Modalidade | **Dupla** (preferencialmente a mesma do CP02, para dar continuidade ao repositório) |
| Prazo sugerido | **14 dias corridos** |
| Carga horária estimada | 16 a 20 horas por dupla |
| Foco | Segurança (RBAC, PSA, securityContext, scan de imagem, Secrets), automação (Job/CronJob) e entrega declarativa (Argo CD) |
| Infraestrutura exigida | Cluster do CP02 (EKS, k3d ou kind) + Argo CD instalado no cluster |

### CP03.2 Enunciado principal

#### Cenário

A FIAP Store passou por uma auditoria de segurança e por um incidente operacional na mesma semana.

**A auditoria** apontou cinco não conformidades: (1) todos os desenvolvedores usavam o mesmo
`kubeconfig` de administrador, com poder de deletar qualquer coisa em qualquer namespace; (2) a
senha do banco estava em um `Secret` base64 versionado no Git, o que não é criptografia; (3) os
containers rodavam com sistema de arquivos gravável e capabilities desnecessárias; (4) nenhuma
imagem passava por análise de vulnerabilidades antes de ir para produção; (5) não havia registro de
quem aplicou o quê no cluster — `kubectl apply` era feito da máquina de cada um.

**O incidente** foi diferente: o relatório de fechamento diário, que roda "quando alguém lembra",
não rodou por três dias, e o time financeiro só percebeu no fechamento do mês. Além disso, uma carga
de reprocessamento de 200 mil pedidos foi disparada em um `screen` na máquina de um estagiário que
fechou o notebook no meio.

Sua dupla é responsável pelo plano de remediação. Ao final, **ninguém mais aplica manifesto à mão**:
o cluster passa a ser reconciliado a partir do Git por Argo CD, cada perfil de usuário tem
exatamente as permissões de que precisa, os segredos saem do repositório, as cargas em lote viram
`Job`/`CronJob` do Kubernetes e as imagens só sobem se passarem no scan.

#### Requisitos funcionais

| ID | Requisito |
|---|---|
| RF01 | Implementar um `Job` de reprocessamento em lote (`store-worker`) que processa N pedidos, com `completions: 5`, `parallelism: 2`, `backoffLimit: 3`, `ttlSecondsAfterFinished` e `restartPolicy: OnFailure`. Deve terminar em `Completed`. |
| RF02 | Implementar um `Job` que **falha de propósito** (código de saída diferente de zero) para demonstrar `backoffLimit`, e documentar a diferença observada entre `restartPolicy: OnFailure` e `Never`. |
| RF03 | Implementar um `CronJob` de relatório diário (`schedule: "0 3 * * *"`), com `concurrencyPolicy: Forbid`, `successfulJobsHistoryLimit: 3`, `failedJobsHistoryLimit: 1` e `startingDeadlineSeconds`. Demonstrar a execução forçando com `kubectl create job --from=cronjob/...`. |
| RF04 | Retirar todos os segredos do repositório. Escolher **uma** estratégia e justificar: (a) Sealed Secrets (Bitnami), (b) External Secrets Operator com AWS Secrets Manager, ou (c) SOPS + age. O que fica versionado deve ser ilegível por quem não tem a chave. |
| RF05 | Demonstrar rotação de segredo: alterar a senha do banco pela ferramenta escolhida, reconciliar e comprovar que a aplicação passou a usar a nova senha. |
| RF06 | Criar um `ServiceAccount` dedicado por workload (`store-api`, `store-worker`, `fluent-bit`), desabilitando `automountServiceAccountToken` onde não for necessário. |
| RF07 | Implementar **RBAC por perfil**, com três perfis, cada um com `Role`/`ClusterRole` e binding próprios: **dev-junior** (leitura de Pods, logs e Services em `store-dev`; nenhum acesso a `store-prod`); **dev-pleno** (tudo do júnior + criar/atualizar Deployments e ConfigMaps em `store-dev`, leitura em `store-prod`, sem acesso a Secrets); **sre** (edição em ambos os namespaces, incluindo Secrets, mas **sem** poder deletar namespaces nem editar objetos de cluster). |
| RF08 | Comprovar cada perfil com `kubectl auth can-i` (mínimo 12 asserções, com resultados `yes` e `no`) e com pelo menos um acesso real usando um kubeconfig alternativo (via `ServiceAccount` token ou usuário certificado). |
| RF09 | Aplicar **Pod Security Admission** nos namespaces: `store-prod` com `restricted` em modo `enforce`, `store-dev` com `baseline` em `enforce` e `restricted` em `warn`/`audit`. Demonstrar um Pod privilegiado sendo **recusado** em `store-prod`. |
| RF10 | Endurecer todos os workloads: `runAsNonRoot`, `runAsUser`/`runAsGroup` explícitos, `readOnlyRootFilesystem: true` (com `emptyDir` onde houver necessidade de escrita), `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `seccompProfile: RuntimeDefault`. |
| RF11 | Executar **scan de vulnerabilidades** da imagem `store-api` com Trivy (ou Grype), salvar o relatório em `docs/seguranca/`, corrigir ao menos uma vulnerabilidade HIGH/CRITICAL (atualizando base image ou dependência) e mostrar o antes e depois. |
| RF12 | Instalar o **Argo CD** no cluster e criar um `Application` que sincroniza `k8s/overlays/prod` a partir do repositório Git, com `syncPolicy.automated` (`prune: true`, `selfHeal: true`). |
| RF13 | Demonstrar **self-heal**: alterar um recurso manualmente no cluster (`kubectl scale`, por exemplo) e mostrar o Argo CD revertendo para o estado do Git. |
| RF14 | Demonstrar **deploy por Git**: abrir um commit que altera a imagem para uma nova tag, e mostrar o Argo CD sincronizando sozinho, com o histórico na UI. |
| RF15 | Criar `Endpoints`/`EndpointSlice` manual apontando para um serviço externo (por exemplo, um banco fora do cluster) e explicar o caso de uso — ou, alternativamente, um `Service` do tipo `ExternalName`. |

#### Requisitos não funcionais

| ID | Requisito |
|---|---|
| RNF01 | Nenhum `kubectl apply` manual em `store-prod` após a instalação do Argo CD. A evidência é o Git como única fonte de verdade. |
| RNF02 | O repositório não pode conter nenhum segredo legível. Um `git log -p` filtrado pelo professor não pode encontrar senha em claro em **nenhum commit do histórico**. |
| RNF03 | Nenhum uso de `cluster-admin` para os perfis de aplicação. O `ClusterRoleBinding` para `cluster-admin` só é aceito para o próprio Argo CD, e mesmo assim deve haver justificativa escrita e menção ao que seria necessário para restringi-lo. |
| RNF04 | Todas as imagens devem ser referenciadas por tag imutável (`1.2.0`) ou digest (`@sha256:...`). `latest` é proibido. |
| RNF05 | Todo `Job`/`CronJob` deve declarar `resources` e ter tempo de execução limitado (`activeDeadlineSeconds`). |
| RNF06 | O `store-worker` não pode ter permissão de leitura em Secrets que não sejam os seus. |
| RNF07 | O relatório de segurança em `docs/seguranca/relatorio.md` deve mapear cada uma das cinco não conformidades da auditoria para a solução implementada, com o comando/manifesto que comprova a correção. |
| RNF08 | O repositório deve continuar aplicável por Kustomize/Helm, mantendo a organização do CP02. |

### CP03.3 Entregáveis

#### Estrutura obrigatória do repositório (evolução do CP02)

```
fiap-store-k8s-<rm1>-<rm2>/
├── README.md
├── infra/
├── app/
│   ├── store-api/
│   ├── store-front/
│   └── store-worker/
│       ├── worker.py
│       ├── relatorio.py
│       └── Dockerfile
├── k8s/
│   ├── base/
│   ├── overlays/{dev,prod}/
│   ├── jobs/
│   │   ├── job-reprocessamento.yaml
│   │   ├── job-falha-proposital.yaml
│   │   └── cronjob-relatorio-diario.yaml
│   ├── rbac/
│   │   ├── serviceaccounts.yaml
│   │   ├── role-dev-junior.yaml
│   │   ├── role-dev-pleno.yaml
│   │   ├── role-sre.yaml
│   │   └── bindings.yaml
│   ├── seguranca/
│   │   ├── namespace-psa-labels.yaml
│   │   ├── pod-privilegiado-teste.yaml     # deve ser RECUSADO em prod
│   │   └── securitycontext-patch.yaml
│   ├── secrets/
│   │   ├── sealed-db-secret.yaml           # cifrado, versionável
│   │   └── COMO-GERAR.md
│   ├── external/
│   │   └── service-externo-endpoints.yaml
│   └── argocd/
│       ├── install-notas.md
│       ├── application-prod.yaml
│       └── project-fiap.yaml
├── scripts/
│   ├── teste-rbac.sh
│   ├── scan-imagem.sh
│   └── verifica.sh
└── docs/
    ├── seguranca/
    │   ├── relatorio.md
    │   ├── trivy-antes.txt
    │   ├── trivy-depois.txt
    │   └── matriz-rbac.md
    ├── gitops/
    │   └── fluxo.md
    └── evidencias/
```

#### Conteúdo obrigatório do README.md

Além do herdado dos Check Points anteriores:

1. **Matriz RBAC** completa, no formato:

   | Perfil | Namespace | Recurso | get | list | create | update | delete |
   |---|---|---|---|---|---|---|---|
   | dev-junior | store-dev | pods | sim | sim | não | não | não |
   | dev-junior | store-dev | pods/log | sim | sim | — | — | — |
   | dev-junior | store-dev | secrets | não | não | não | não | não |
   | dev-junior | store-prod | * | não | não | não | não | não |
   | dev-pleno | store-dev | deployments | sim | sim | sim | sim | não |
   | dev-pleno | store-prod | pods | sim | sim | não | não | não |
   | sre | store-dev, store-prod | quase tudo | sim | sim | sim | sim | parcial |

2. **Justificativa da estratégia de Secrets** escolhida, comparando as três opções em uma tabela
   (facilidade, dependência de nuvem, rotação, recuperação de desastre).
3. **Relatório de segurança** ligando as 5 não conformidades às correções.
4. **Fluxo GitOps** desenhado: quem commita, o que dispara o quê, como se faz rollback (dica: no
   GitOps, rollback é `git revert`).
5. **Antes/depois do Trivy**, com a contagem de vulnerabilidades por severidade.

#### Vídeo de demonstração

**8 a 10 minutos**:

| Tempo | Conteúdo |
|---|---|
| 0:00–0:30 | Identificação e resumo do plano de remediação |
| 0:30–2:00 | Jobs: reprocessamento concluindo, Job que falha e o `backoffLimit` atuando, CronJob disparado manualmente |
| 2:00–3:30 | Segredos: mostrar o arquivo cifrado no Git, a rotação de senha e a aplicação usando a nova credencial |
| 3:30–5:30 | RBAC: trocar de contexto para cada perfil e demonstrar `can-i` e uma ação negada de verdade (erro "forbidden") |
| 5:30–6:30 | PSA e hardening: Pod privilegiado sendo recusado em `store-prod`, `readOnlyRootFilesystem` provado com tentativa de escrita |
| 6:30–7:30 | Trivy antes e depois |
| 7:30–9:30 | Argo CD: UI sincronizada, self-heal revertendo um `kubectl scale`, e deploy por commit |

### CP03.4 Rubrica de avaliação (100 pontos)

| Critério | Peso | Insuficiente (0–25%) | Regular (26–55%) | Bom (56–85%) | Excelente (86–100%) |
|---|---|---|---|---|---|
| **C1. Jobs e CronJob** | 12 | Ausentes ou não completam | Job simples completa, sem paralelismo, CronJob sem política de concorrência | Job com `completions`/`parallelism`/`backoffLimit`, caso de falha demonstrado, CronJob com `Forbid` e limites de histórico | Além do "Bom": idempotência do worker discutida, `activeDeadlineSeconds` justificado, tratamento de reentrância, análise do que acontece com CronJob perdido (`startingDeadlineSeconds`) |
| **C2. Gestão de Secrets** | 13 | Secret base64 versionado | Secret criado por comando, fora do Git, mas sem ferramenta de gestão | Sealed Secrets / ESO / SOPS funcionando, rotação demonstrada | Além do "Bom": comparação fundamentada das alternativas, plano de recuperação da chave mestra, escopo mínimo de acesso ao segredo, auditoria de quem pode lê-lo |
| **C3. RBAC** | 20 | Ausente ou tudo `cluster-admin` | Roles criadas mas amplas demais (`verbs: ["*"]`), sem teste | Três perfis com permissões distintas, testados com `auth can-i` e uso real de kubeconfig alternativo | Além do "Bom": princípio do menor privilégio evidente (verbos e resources enxutos), uso correto de `Role` vs `ClusterRole`, `resourceNames` onde couber, matriz completa e testes negativos abundantes |
| **C4. Hardening e Pod Security Admission** | 20 | Containers como root, sem securityContext | Alguns campos de segurança em um workload apenas; PSA ausente | PSA `restricted` em prod, `baseline` em dev, todos os workloads com securityContext completo, Pod privilegiado recusado em vídeo | Além do "Bom": `readOnlyRootFilesystem` com `emptyDir` bem resolvido, `seccompProfile`, ausência de token montado onde desnecessário, análise dos avisos do modo `audit`/`warn` em dev |
| **C5. Scan de imagem e cadeia de suprimentos** | 10 | Ausente | Scan rodado, relatório colado sem análise | Trivy antes/depois, ao menos uma HIGH/CRITICAL corrigida, relatório interpretado | Além do "Bom": scan integrado a pipeline com falha automática por severidade, imagem distroless ou base mínima, SBOM gerado, digest em vez de tag |
| **C6. GitOps com Argo CD** | 20 | Ausente | Argo CD instalado, Application criada mas dessincronizada ou sincronizada só manualmente | Application com `automated`, `prune`, `selfHeal`; self-heal e deploy por commit demonstrados | Além do "Bom": `AppProject` restringindo origens/destinos, estrutura app-of-apps ou múltiplos Applications, rollback por `git revert` demonstrado, RBAC do próprio Argo CD configurado, notas sobre Rancher como alternativa de gestão |
| **C7. Documentação e relatório de segurança** | 5 | Ausente | Superficial | Relatório mapeando as 5 não conformidades, matriz RBAC, fluxo GitOps | Além do "Bom": runbook de incidente, discussão de trade-offs, referência a OWASP Kubernetes Top 10 ou CIS Benchmark com itens específicos |
| **Total** | **100** | | | | |

#### Penalidades e bônus fixos

| Situação | Efeito |
|---|---|
| Vídeo ausente | −20 pontos |
| Qualquer senha legível encontrada no histórico do Git | −20 pontos |
| Uso de `latest` em qualquer imagem de workload | −5 pontos |
| `cluster-admin` concedido a workload de aplicação | −15 pontos |
| Argo CD instalado mas com Application `OutOfSync` sem explicação | −10 pontos |
| Bônus: `kube-bench` ou `kubescape` executado com plano de correção | +5 pontos |
| Bônus: `KEDA` escalando o `store-worker` por fila, com scale-to-zero | +5 pontos |
| Bônus: assinatura de imagem com `cosign` e verificação por policy | +5 pontos |
| Bônus: Rancher instalado gerenciando o cluster, com comparação escrita frente ao Argo CD | +3 pontos |

### CP03.5 Roteiro de correção rápida (6 a 8 minutos por dupla)

```bash
git clone <url-da-dupla> dupla && cd dupla && git checkout cp03

# ---------- SEGREDOS ----------
# 1. Nenhuma senha legivel em NENHUM commit
git log -p --all | grep -inE "(password|senha|passwd)\s*[:=]" | grep -viE "example|placeholder|<" | head
# 2. O que esta versionado deve ser cifrado
kubectl get sealedsecret -A 2>/dev/null || kubectl get externalsecret -A 2>/dev/null

# ---------- JOBS ----------
kubectl apply -f k8s/jobs/
kubectl get jobs,cronjobs -n store-dev
kubectl get job reprocessamento -n store-dev \
  -o jsonpath='{.spec.completions}/{.spec.parallelism}/{.spec.backoffLimit}'; echo
kubectl wait --for=condition=complete job/reprocessamento -n store-dev --timeout=180s
kubectl get cronjob relatorio-diario -n store-dev \
  -o jsonpath='{.spec.schedule} {.spec.concurrencyPolicy}'; echo
kubectl create job manual-teste --from=cronjob/relatorio-diario -n store-dev
kubectl logs job/manual-teste -n store-dev

# ---------- RBAC ----------
kubectl get sa,role,rolebinding,clusterrole,clusterrolebinding -n store-dev \
  | grep -iE "dev-junior|dev-pleno|sre|store-"

# Perfil dev-junior: deve poder ver pods em dev e NAO poder nada em prod
kubectl auth can-i get pods        -n store-dev  --as=system:serviceaccount:store-dev:dev-junior
kubectl auth can-i get pods/log    -n store-dev  --as=system:serviceaccount:store-dev:dev-junior
kubectl auth can-i delete pods     -n store-dev  --as=system:serviceaccount:store-dev:dev-junior   # no
kubectl auth can-i get secrets     -n store-dev  --as=system:serviceaccount:store-dev:dev-junior   # no
kubectl auth can-i get pods        -n store-prod --as=system:serviceaccount:store-dev:dev-junior   # no

# Perfil dev-pleno
kubectl auth can-i update deployments -n store-dev  --as=system:serviceaccount:store-dev:dev-pleno  # yes
kubectl auth can-i get secrets        -n store-dev  --as=system:serviceaccount:store-dev:dev-pleno  # no
kubectl auth can-i get pods           -n store-prod --as=system:serviceaccount:store-dev:dev-pleno  # yes
kubectl auth can-i create deployments -n store-prod --as=system:serviceaccount:store-dev:dev-pleno  # no

# Perfil sre
kubectl auth can-i get secrets        -n store-prod --as=system:serviceaccount:store-prod:sre       # yes
kubectl auth can-i delete namespaces               --as=system:serviceaccount:store-prod:sre       # no
kubectl auth can-i create clusterrolebindings      --as=system:serviceaccount:store-prod:sre       # no

# Nenhum cluster-admin indevido
kubectl get clusterrolebindings -o json \
  | jq -r '.items[] | select(.roleRef.name=="cluster-admin") | .metadata.name'

# ---------- POD SECURITY ADMISSION ----------
kubectl get ns store-prod -o jsonpath='{.metadata.labels}'; echo
# Esperado conter: pod-security.kubernetes.io/enforce=restricted
kubectl apply -f k8s/seguranca/pod-privilegiado-teste.yaml -n store-prod
# Esperado: ERRO de admissao "violates PodSecurity restricted"

# ---------- HARDENING ----------
kubectl get deploy store-api -n store-prod \
  -o jsonpath='{.spec.template.spec.containers[0].securityContext}'; echo
kubectl exec -n store-prod deploy/store-api -- sh -c 'touch /teste 2>&1' # deve falhar: read-only
kubectl get deploy store-api -n store-prod \
  -o jsonpath='{.spec.template.spec.containers[0].image}'; echo   # nao pode ser :latest

# ---------- SCAN ----------
trivy image --severity HIGH,CRITICAL <usuario>/store-api:1.2.0
diff <(head -40 docs/seguranca/trivy-antes.txt) <(head -40 docs/seguranca/trivy-depois.txt) | head

# ---------- ARGO CD ----------
kubectl get applications -n argocd
kubectl get application store-prod -n argocd \
  -o jsonpath='{.status.sync.status} {.status.health.status}'; echo   # Synced Healthy
kubectl get application store-prod -n argocd -o jsonpath='{.spec.syncPolicy}'; echo

# Self-heal: alterar a mao e ver o Argo reverter
kubectl scale deploy/store-api -n store-prod --replicas=1
sleep 60
kubectl get deploy store-api -n store-prod   # deve ter voltado para 3
```

**Checklist binário do professor:**

| # | Verificação | OK |
|---|---|---|
| 1 | Nenhuma senha legível em nenhum commit do histórico | |
| 2 | Segredo versionado está cifrado (SealedSecret/ExternalSecret/SOPS) | |
| 3 | Rotação de segredo demonstrada no vídeo | |
| 4 | Job atinge `Completed` com `completions` e `parallelism` corretos | |
| 5 | Job de falha demonstra `backoffLimit` | |
| 6 | CronJob com `Forbid` e limites de histórico | |
| 7 | Três perfis RBAC existem e são distintos | |
| 8 | `can-i` retorna `no` nos casos negativos esperados | |
| 9 | Nenhum workload de aplicação com `cluster-admin` | |
| 10 | `store-prod` com label PSA `enforce=restricted` | |
| 11 | Pod privilegiado é recusado em `store-prod` | |
| 12 | `readOnlyRootFilesystem` comprovado (escrita falha) | |
| 13 | Nenhuma imagem com tag `latest` | |
| 14 | Relatórios do Trivy antes/depois com redução de HIGH/CRITICAL | |
| 15 | Argo CD Application `Synced` + `Healthy` com `automated/selfHeal` | |
| 16 | Self-heal revertendo alteração manual comprovado | |
| 17 | Deploy por commit comprovado no vídeo | |
| 18 | Relatório de segurança mapeia as 5 não conformidades | |

### CP03.6 Política de plágio, uso de IA e atrasos

Valem as regras dos CPs anteriores. Especificidades do CP03:

- **Arguição obrigatória de segurança.** Este Check Point tem arguição obrigatória (não opcional) de
  5 minutos por dupla, com pelo menos duas perguntas sobre RBAC e uma sobre a estratégia de Secrets.
  Segurança mal compreendida é pior do que segurança ausente, porque gera falsa confiança.
- **Manifesto de RBAC gerado por IA** é aceito como rascunho, mas o aluno deve saber responder, para
  qualquer `Role` entregue: por que este verbo, por que este `apiGroup`, o que quebraria se este
  recurso saísse da lista, e por que `Role` e não `ClusterRole`.
- **Copiar a configuração do Argo CD** da documentação é esperado. O que se avalia é a adequação ao
  repositório da dupla (path correto, `targetRevision`, projeto restrito) e a compreensão do
  significado de `prune` e `selfHeal` — inclusive dos riscos: `prune: true` mal configurado apaga
  recursos legítimos.
- **Segredo real vazado** no repositório (uma senha de verdade, uma chave AWS válida) implica, além
  da penalidade de −20 pontos, orientação imediata de rotação da credencial. Chave AWS válida em
  repositório público é incidente de segurança real, não apenas erro acadêmico.
- **Atrasos:** mesma tabela (−10% / −25% / −40% / não aceito acima de 72 h).

### CP03.7 Variações alternativas de tema

**Variação A — Conformidade LGPD para dados de clientes.** A narrativa muda de auditoria genérica
para uma **adequação à LGPD**: a FIAP Store armazena CPF, endereço e histórico de compras. Os
requisitos ganham foco em confidencialidade e rastreabilidade: os Secrets passam a incluir a chave
de criptografia de campo usada pela aplicação; o RBAC precisa de um quarto perfil, **auditor**, com
permissão de leitura de logs e eventos mas **proibição explícita** de ler Secrets e de executar
`exec` em Pods (bom exercício de `pods/exec` como recurso separado); e o CronJob deixa de ser um
relatório e passa a ser uma **rotina de expurgo** que apaga dados de clientes inativos há mais de 5
anos, com log de auditoria da operação. Acrescente a exigência de habilitar o audit log do cluster
(ou, no k3d, `--kube-apiserver-arg=audit-log-path=...`) e demonstrar o rastro de "quem fez o quê".

**Variação B — Pipeline de imagem com portão de qualidade.** Desloca-se o peso de RBAC para
**cadeia de suprimentos de software**. A dupla deve construir um pipeline (GitHub Actions ou GitLab
CI) que: constrói a imagem, gera SBOM com Syft, roda Trivy com `--exit-code 1 --severity CRITICAL`
(o pipeline **falha** se houver crítica), assina a imagem com cosign, publica no ECR e só então
abre um commit no repositório de manifestos que o Argo CD reconcilia. No cluster, uma policy de
admissão (Kyverno ou OPA Gatekeeper) recusa imagens não assinadas ou vindas de registros não
confiáveis. O RBAC continua exigido, mas com dois perfis apenas. É a variação mais adequada a turmas
com forte perfil de DevOps.

**Variação C — Operação multi-cluster com Rancher.** O foco vai para **gestão de frota**. A dupla
provisiona dois clusters (por exemplo, `fiap-dev` e `fiap-prod`, ambos em k3d para viabilidade),
instala o Rancher gerenciando os dois, e configura projetos, quotas e usuários pela interface do
Rancher, comparando com o RBAC nativo do Kubernetes (o Rancher adiciona a camada de "Project" acima
do namespace). Em paralelo, o Argo CD é configurado com dois destinos, entregando a mesma base de
manifestos com overlays diferentes para cada cluster, e a dupla deve demonstrar uma promoção
dev → prod feita exclusivamente por Git (merge de um PR que altera o `targetRevision` ou a tag da
imagem no overlay de prod). Excelente para discutir governança, mas exige máquina com 16 GB de RAM.

### CP03.8 Barema de recuperação e plano de contingência

#### Se a dupla não tem acesso a nuvem

O CP03 é o mais fácil de executar localmente entre os três, porque tudo o que ele exige (RBAC, PSA,
Jobs, Argo CD) é nativo do Kubernetes.

| Requisito | Na AWS | Substituto local aceito |
|---|---|---|
| Secrets externos | External Secrets Operator + AWS Secrets Manager | Sealed Secrets (Bitnami) ou SOPS + age — sem perda de nota |
| Registro de imagens | Amazon ECR com scan nativo | Docker Hub + Trivy local |
| IAM ligado ao RBAC (IRSA) | `eksctl create iamserviceaccount` | `ServiceAccount` + `Role` nativos; explicar no README o que o IRSA acrescentaria |
| Audit log do API server | EKS control plane logging | `k3d cluster create --k3s-arg "--kube-apiserver-arg=audit-log-path=/var/log/audit.log@server:0"` |
| Argo CD | Instalado no EKS | Instalado no k3d; acesso por `kubectl port-forward svc/argocd-server -n argocd 8080:443` |
| Assinatura de imagem | cosign + ECR | cosign com chave local e registro do k3d |

#### Se a dupla não conseguiu instalar o Argo CD

Alternativa aceita com **até 80% dos pontos de C6**: usar `Flux CD` no lugar do Argo CD, cumprindo
os mesmos requisitos funcionais (reconciliação automática, self-heal, deploy por commit) e
documentando as diferenças. Não é aceito substituir GitOps por "pipeline que roda `kubectl apply`" —
isso é CI/CD por push, não GitOps, e vale no máximo 30% de C6, com a diferença conceitual explicada
no README para receber esses pontos.

#### Recuperação (abaixo de 60 pontos)

1. Devolutiva com rubrica preenchida.
2. **7 dias corridos** para correção dirigida.
3. Entrega: tag `cp03-rec` + vídeo de 4 minutos.
4. Nota limitada a **70 pontos**.
5. Como o CP03 alimenta diretamente a Global Solution, o professor deve priorizar a devolutiva deste
   Check Point: o aluno precisa das correções **antes** de consolidar a arquitetura do trabalho
   final.

#### Barema simplificado de recuperação

| Item verificado na recuperação | Pontos |
|---|---|
| Job com paralelismo concluindo + CronJob com `Forbid` | 15 |
| Nenhum segredo legível no repositório e rotação demonstrada | 15 |
| Três perfis RBAC com `can-i` retornando `yes` e `no` corretos | 25 |
| PSA `restricted` em prod recusando Pod privilegiado | 20 |
| securityContext completo em todos os workloads | 10 |
| Argo CD `Synced/Healthy` com self-heal comprovado | 15 |
| **Total (teto de 70 aplicado depois)** | **100** |

---

## Anexo — Ficha de correção consolidada

Modelo para o professor registrar as três entregas de um mesmo aluno/dupla em uma folha só.

| Campo | Valor |
|---|---|
| Aluno(s) / RM | |
| Turma | |
| Repositório | |

| Check Point | C1 | C2 | C3 | C4 | C5 | C6 | C7 | C8 | Penalidades | Bônus | Total |
|---|---|---|---|---|---|---|---|---|---|---|---|
| CP01 | /15 | /20 | /10 | /15 | /15 | /15 | /10 | — | | | /100 |
| CP02 | /12 | /12 | /15 | /15 | /15 | /10 | /11 | /10 | | | /100 |
| CP03 | /12 | /13 | /20 | /20 | /10 | /20 | /5 | — | | | /100 |
| **Média dos Check Points** | | | | | | | | | | | **/100** |

**Observações qualitativas para a devolutiva** (o que o aluno deve levar para a Global Solution):

| Ponto forte identificado | Ponto a desenvolver na GS |
|---|---|
| | |
| | |
| | |

---

# Global Solution — Trabalho Final

## 1. Apresentação

A **Global Solution (GS)** é o trabalho final da disciplina Cloud Developer II — Kubernetes e
Serverless. Ela não é "mais uma entrega": é a consolidação de tudo o que foi construído nos três
Check Points em um **produto de plataforma completo**, projetado, implantado, documentado, testado
sob carga, testado sob falha e defendido oralmente diante de uma banca.

A diferença essencial entre um Check Point e a Global Solution é a **autonomia de projeto**. Nos
Check Points o enunciado dizia o que fazer, recurso por recurso. Na GS o enunciado descreve um
**problema de negócio** e um conjunto de requisitos; as decisões de arquitetura — quantos
microsserviços, qual estratégia de dados, o que roda como Deployment e o que roda como função
serverless, como isolar ambientes, quanto custa — são do grupo, e é exatamente isso que a banca
avalia. Uma arquitetura defensável com escolhas justificadas vale mais do que uma arquitetura
sofisticada que ninguém do grupo sabe explicar.

### 1.1 Peso e articulação com os Check Points

| Instrumento | Peso sugerido na média final |
|---|---|
| Média dos três Check Points | 40% |
| Global Solution | 60% (peso de prova) |

A GS **absorve** os artefatos dos Check Points. O grupo deve reaproveitar o repositório e evoluí-lo:

| Origem | O que entra na GS |
|---|---|
| CP01 | Containerização, Deployments, Services, probes, rolling update, HPA, disciplina de YAML declarativo |
| CP02 | Cluster multi-nó, namespaces com quotas, Ingress com TLS, NetworkPolicy, persistência com StatefulSet/PVC, DaemonSet de logs, PDB, noção de custo |
| CP03 | Jobs/CronJob, gestão de Secrets fora do Git, RBAC por perfil, Pod Security Admission, hardening, scan de imagem, GitOps com Argo CD |
| Novo na GS | Múltiplos microsserviços integrados, worker assíncrono/serverless, observabilidade (métricas, logs, traces), pipeline CI/CD completo, teste de carga, teste de caos, relatório de custos, defesa oral |

> **Nota:** grupos formados por alunos que fizeram Check Points diferentes (variações de tema)
> devem escolher **um** repositório como base e documentar, no README, de onde veio cada parte.

### 1.2 Formato

| Item | Definição |
|---|---|
| Composição | Grupos de **3 a 5 integrantes**. Grupos de 2 só com autorização do professor e com escopo reduzido negociado; grupos de 6 não são aceitos |
| Duração | **6 semanas**, organizadas em 3 sprints de 2 semanas |
| Acompanhamento | 3 checkpoints intermediários obrigatórios com o professor (fim de cada sprint), de 15 minutos por grupo |
| Entrega | Repositório Git com tag `gs-final` + documento de arquitetura + vídeo + apresentação, todos até a data limite |
| Defesa | **Pitch de 10 minutos** + **10 minutos de arguição** da banca, com todos os integrantes presentes |
| Banca | Professor da disciplina + 1 a 2 convidados (professor de outra disciplina, egresso, profissional de mercado) |
| Presença na defesa | Obrigatória. Integrante ausente sem justificativa formal recebe **zero** na GS, independentemente da nota do grupo |

### 1.3 Regras da defesa

- Todos os integrantes falam. O grupo distribui os 10 minutos como quiser, mas **ninguém pode ficar
  em silêncio** durante o pitch.
- A banca escolhe **quem responde cada pergunta**. Não é permitido "passar a bola".
- A apresentação é feita com o sistema **rodando ao vivo** ou com vídeo gravado como plano B
  (recomenda-se ter o plano B pronto: demonstração ao vivo que falha consome os 10 minutos).
- Slides são opcionais para o pitch, mas o **documento de arquitetura com diagrama é obrigatório**.
- Tempo é cronometrado. Aos 10 minutos o pitch é interrompido, e o que não foi apresentado não é
  avaliado no critério de defesa.

---

## 2. Tema principal — Plataforma cloud-native de e-commerce resiliente

### 2.1 Cenário de negócio

A **FIAP Store** deixou de ser um projeto interno e virou um negócio. Depois de dois anos operando
com um monolito em um servidor único e, mais recentemente, com a prova de conceito em Kubernetes que
vocês construíram nos Check Points, a empresa recebeu aporte e traçou metas agressivas para os
próximos 12 meses:

- Crescer de 2 mil para 60 mil pedidos por dia.
- Suportar campanhas relâmpago (Black Friday, lançamentos) com picos de **30x** o tráfego médio, com
  duração de 2 a 4 horas, sem provisionar 30x de infraestrutura o ano inteiro.
- Reduzir o tempo entre "código aprovado" e "código em produção" de 3 semanas para **menos de 1
  hora**, sem downtime.
- Garantir que **nenhum pedido pago seja perdido**, mesmo com falha de Pods, de nós ou de zona.
- Passar em uma auditoria de segurança de um cliente corporativo, que exige controle de acesso por
  perfil, segredos gerenciados, imagens escaneadas e trilha de auditoria de mudanças.
- Ter previsibilidade de custo: a diretoria financeira quer saber **quanto custa cada mil pedidos**.

O grupo é a equipe de plataforma contratada para desenhar e implantar essa arquitetura.

### 2.2 Personas

| Persona | Papel | O que espera da plataforma | Como isso vira requisito |
|---|---|---|---|
| **Camila** — cliente final | Compra pelo site e pelo app | Site rápido, disponível e seguro (HTTPS); pedido confirmado imediatamente | RNF de latência, disponibilidade, TLS; confirmação síncrona com processamento assíncrono |
| **Rogério** — gerente de operações | Acompanha pedidos e estoque | Painel com status dos pedidos, alerta quando algo falha | Observabilidade, dashboards, alertas |
| **Ana** — desenvolvedora do time de produto | Entrega features | Deploy rápido, ambiente de dev isolado, rollback fácil | CI/CD, GitOps, namespaces, RBAC de dev |
| **Bruno** — SRE / plataforma | Mantém o cluster de pé | Autoscaling, health checks, runbooks, capacidade de drenar nó sem downtime | HPA, probes, PDB, chaos testing |
| **Denise** — CISO da cliente corporativa | Aprova o fornecedor | Menor privilégio, segredos protegidos, imagens sem CVE crítica, auditoria | RBAC, PSA, Secrets externos, scan, audit trail |
| **Fernanda** — CFO | Aprova o orçamento | Custo por transação, sem desperdício de recurso ocioso | FinOps, right-sizing, scale-to-zero do worker |

### 2.3 Requisitos funcionais

| ID | Requisito |
|---|---|
| **RF01** | O sistema deve expor um catálogo de produtos com listagem e detalhe (`GET /produtos`, `GET /produtos/{id}`), servido pelo microsserviço de catálogo. |
| **RF02** | O sistema deve permitir criar um pedido (`POST /pedidos`) informando itens e quantidade, retornando um identificador e o status inicial `RECEBIDO` em resposta síncrona de até 500 ms. |
| **RF03** | O processamento do pedido (validação de estoque, cálculo de frete, confirmação de pagamento simulado) deve ocorrer **de forma assíncrona**, fora do caminho da requisição do usuário. |
| **RF04** | O sistema deve permitir consultar o status de um pedido (`GET /pedidos/{id}`), refletindo a evolução `RECEBIDO → EM_PROCESSAMENTO → CONFIRMADO` ou `RECUSADO`. |
| **RF05** | O sistema deve manter controle de estoque, decrementando a quantidade disponível na confirmação do pedido e recusando pedidos sem estoque, sem permitir estoque negativo. |
| **RF06** | O sistema deve emitir uma notificação (e-mail simulado, webhook, ou registro em log estruturado com marcação clara) a cada mudança de status do pedido. |
| **RF07** | O sistema deve oferecer uma interface web (`store-front`) que consuma as APIs e permita navegar no catálogo e acompanhar um pedido. |
| **RF08** | O sistema deve executar uma rotina agendada diária que gera um relatório consolidado de vendas (quantidade, faturamento, itens mais vendidos) e o persiste. |
| **RF09** | O sistema deve oferecer uma rotina de reprocessamento em lote, acionada sob demanda, capaz de reprocessar pedidos que falharam, de forma idempotente. |
| **RF10** | O sistema deve expor endpoints operacionais em todos os serviços: `/health` (liveness), `/ready` (readiness) e `/metrics` (formato Prometheus). |
| **RF11** | O sistema deve oferecer dois ambientes isolados (`store-dev` e `store-prod`) com a mesma base de manifestos e configurações distintas por ambiente. |
| **RF12** | O sistema deve oferecer uma área administrativa (pode ser apenas endpoints protegidos) acessível somente por usuários com perfil administrativo, exercitando o controle de acesso. |

### 2.4 Requisitos não funcionais

| ID | Requisito |
|---|---|
| **RNF01** | **Disponibilidade.** Nenhum serviço com ponto único de falha: mínimo de 2 réplicas por serviço stateless em produção, distribuídas em nós distintos (`topologySpreadConstraints`), com `PodDisruptionBudget`. A aplicação deve continuar respondendo durante drain de um nó. |
| **RNF02** | **Elasticidade.** `HorizontalPodAutoscaler` em pelo menos dois serviços, com métrica justificada (CPU e/ou métrica customizada). O worker assíncrono deve escalar pelo tamanho da fila (KEDA) ou ter justificativa documentada para não fazê-lo. |
| **RNF03** | **Desempenho.** Latência p95 abaixo de 500 ms para leitura de catálogo e abaixo de 800 ms para criação de pedido, sob carga de referência definida no plano de teste. Comprovado por teste de carga. |
| **RNF04** | **Durabilidade.** Dados de pedidos e estoque em armazenamento persistente que sobrevive à morte do Pod e à recriação do nó. Backup lógico demonstrado (dump + restore). |
| **RNF05** | **Segurança de acesso.** RBAC com no mínimo três perfis (dev, sre, auditor/leitura), princípio do menor privilégio, `ServiceAccount` dedicado por workload, nenhum `cluster-admin` para aplicação. |
| **RNF06** | **Segurança de execução.** Pod Security Admission `restricted` em produção; todos os containers com `runAsNonRoot`, `readOnlyRootFilesystem`, `allowPrivilegeEscalation: false`, `capabilities.drop: ["ALL"]`, `seccompProfile: RuntimeDefault`. |
| **RNF07** | **Segurança de dados e imagens.** Nenhum segredo legível no Git (Sealed Secrets, External Secrets ou SOPS); TLS no Ingress; imagens escaneadas com Trivy/Grype sem CRITICAL não justificada; imagens referenciadas por tag imutável ou digest. |
| **RNF08** | **Rede.** NetworkPolicy com default-deny em produção e liberações explícitas; isolamento comprovado entre `store-dev` e `store-prod`. |
| **RNF09** | **Observabilidade.** Métricas expostas e coletadas (Prometheus), ao menos um dashboard significativo (Grafana), logs centralizados (DaemonSet), e tracing distribuído **ou** correlação de logs por `request-id` propagado entre serviços. Pelo menos uma regra de alerta configurada. |
| **RNF10** | **Entrega e reprodutibilidade.** Toda a plataforma implantável a partir do Git por GitOps (Argo CD ou Flux), com pipeline de CI que constrói, testa, escaneia e publica as imagens. Ambiente recriável do zero em menos de 60 minutos por um terceiro seguindo o README. |

### 2.5 Arquitetura mínima exigida

O grupo tem liberdade de desenho, mas a arquitetura entregue precisa conter, no mínimo:

| Componente | Exigência mínima | Observação |
|---|---|---|
| Microsserviços | **No mínimo 3** serviços de aplicação distintos, com responsabilidades separadas e comunicação entre eles (ex.: `catalogo-api`, `pedidos-api`, `estoque-api`) | Não vale renomear três cópias do mesmo serviço |
| Front-end | 1 serviço web servindo a interface | Nginx com estático já é suficiente |
| Banco de dados | PostgreSQL com persistência real (StatefulSet + PVC) | Pode haver mais de um banco; discuta banco por serviço vs compartilhado |
| Fila / mensageria | Broker (RabbitMQ, Redis Streams, NATS ou SQS) desacoplando a API do processamento | É o que viabiliza RF03 e RNF02 |
| Worker assíncrono | Consumidor da fila, escalável | Aqui entra o KEDA, se usado |
| Componente serverless | **Obrigatório**: ao menos uma função/serviço com escalonamento a zero ou FaaS gerenciada | Ver seção 2.6 |
| Ingress + TLS | Entrada única, roteamento por host/path, certificado | NGINX Ingress, Traefik ou ALB Controller |
| Autoscaling | HPA em ao menos 2 serviços | Cluster Autoscaler/Karpenter é bônus |
| Observabilidade | Prometheus + Grafana + coleta de logs | kube-prometheus-stack via Helm é aceito |
| GitOps | Argo CD (ou Flux) reconciliando produção | Com `automated`, `prune`, `selfHeal` |
| CI/CD | Pipeline versionado no repositório | GitHub Actions, GitLab CI ou similar |
| Segurança | RBAC, PSA, NetworkPolicy, Secrets gerenciados, scan | Herdado e ampliado do CP03 |

### 2.6 O componente serverless (obrigatório)

A disciplina é "Kubernetes **e** Serverless". A GS deve conter pelo menos **um** elemento serverless
real, escolhido entre as opções abaixo e **justificado** no documento de arquitetura (por que este
componente é bom candidato a serverless, e qual o trade-off assumido — cold start, limite de
execução, acoplamento ao provedor):

| Opção | O que implementar | Onde encaixa bem na FIAP Store |
|---|---|---|
| **KEDA** | `ScaledObject` escalando o worker pelo tamanho da fila, com `minReplicaCount: 0` | Reprocessamento e processamento de pedidos: zero réplica fora do horário de pico |
| **Knative Serving** | O serviço de notificação como Knative Service com scale-to-zero | Notificação é esporádica e tolera cold start |
| **AWS Lambda** | Função de geração de thumbnail de imagem de produto ou de emissão de notificação, acionada por evento | Integra com S3/EventBridge; exige EKS ou pelo menos conta AWS |
| **AWS Fargate (perfil no EKS)** | Um namespace ou workload específico rodando em Fargate, sem nó gerenciado | Bom para o worker de picos: capacidade sob demanda |
| **EventBridge + Lambda** | Substituir o CronJob do relatório diário por regra agendada + função | Permite comparar CronJob vs agendamento gerenciado |

> **Nota:** a comparação escrita entre a alternativa Kubernetes e a alternativa serverless para o
> mesmo problema (latência, custo, operação, portabilidade) é parte do que se avalia. Implementar
> KEDA sem saber dizer por que não bastaria um HPA de CPU não pontua no nível "Excelente".

### 2.7 Restrições

| Restrição | Detalhe |
|---|---|
| Kubernetes obrigatório | A plataforma roda em Kubernetes 1.33+; EKS preferencial, k3d/kind multi-nó aceito integralmente |
| Declarativo | Nenhum recurso de produção criado imperativamente; tudo no Git |
| Sem `latest` | Todas as imagens por tag imutável ou digest |
| Sem segredo em claro | Em nenhum commit de nenhum branch |
| Custo controlado | Se usar AWS, o cluster deve ser destruído após as evidências; relatório de custo obrigatório |
| Autoria | Código de terceiros permitido com citação; a integração e as decisões devem ser do grupo |
| Uso de IA | Permitido e declarado, com as mesmas regras dos Check Points (ver seção 9) |
| Prazo | Sem prorrogação coletiva; a data da defesa é fixa por conta da banca |

### 2.8 Fora do escopo

Para evitar que o grupo se perca, os itens abaixo **não são exigidos** e não pontuam (podem até
prejudicar, se consumirem o tempo de itens obrigatórios):

- Integração real com gateway de pagamento (simule; um serviço mock basta).
- Envio real de e-mail/SMS (log estruturado ou webhook para um endpoint de teste é suficiente).
- Front-end elaborado. A avaliação de UI é irrelevante aqui: HTML simples que consome a API basta.
- Autenticação de usuário final com OAuth/OIDC completo (um token estático ou básico é suficiente
  para exercitar o RF12). Autenticação **do cluster** (RBAC) essa sim é exigida.
- Aplicativo móvel.
- Migração de dados legados em volume real.
- Alta disponibilidade multi-região (é item de **bonificação**, não de escopo).

---

## 3. Entregáveis

### 3.1 Repositório — estrutura completa

```
fiap-store-gs-<grupo>/
├── README.md                         # preencher o modelo do Anexo A
├── LICENSE
├── .gitignore
├── .github/
│   └── workflows/
│       ├── ci-build-scan-push.yml    # build, teste, Trivy, push
│       └── cd-bump-manifests.yml     # atualiza a tag no overlay (GitOps)
├── docs/
│   ├── arquitetura.md                # documento de arquitetura (secao 3.2)
│   ├── diagramas/
│   │   ├── c4-contexto.png
│   │   ├── c4-container.png
│   │   └── fluxo-pedido.md           # sequencia em Mermaid
│   ├── adr/                          # Architecture Decision Records
│   │   ├── 001-banco-por-servico.md
│   │   ├── 002-broker-escolhido.md
│   │   ├── 003-serverless-keda.md
│   │   └── 004-gitops-argocd.md
│   ├── seguranca/
│   │   ├── matriz-rbac.md
│   │   ├── trivy-antes.txt
│   │   ├── trivy-depois.txt
│   │   └── ameacas.md                # analise simples de ameacas
│   ├── testes/
│   │   ├── plano-de-carga.md
│   │   ├── k6-script.js
│   │   ├── resultado-carga.md
│   │   ├── plano-de-caos.md
│   │   └── resultado-caos.md
│   ├── custos/
│   │   ├── relatorio-custos.md
│   │   └── evidencias/
│   ├── operacao/
│   │   ├── runbook.md
│   │   └── alertas.md
│   ├── atas/
│   │   ├── sprint-1.md               # modelo do Anexo B
│   │   ├── sprint-2.md
│   │   └── sprint-3.md
│   └── evidencias/
├── infra/
│   ├── cluster.yaml                  # eksctl ClusterConfig ou k3d config
│   ├── criar-cluster.sh
│   ├── destruir-cluster.sh
│   └── bootstrap.sh                  # instala Argo CD, ingress, prometheus, keda
├── services/
│   ├── catalogo-api/    {src/, tests/, Dockerfile, requirements.txt}
│   ├── pedidos-api/     {src/, tests/, Dockerfile, requirements.txt}
│   ├── estoque-api/     {src/, tests/, Dockerfile, requirements.txt}
│   ├── notificacao-fn/  {src/, Dockerfile ou template serverless}
│   ├── store-worker/    {src/, Dockerfile}
│   └── store-front/     {html/, nginx.conf, Dockerfile}
├── k8s/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── catalogo/ pedidos/ estoque/ worker/ front/ db/ broker/
│   ├── overlays/
│   │   ├── dev/     {kustomization.yaml, namespace, quota, patches}
│   │   └── prod/    {kustomization.yaml, namespace, quota, hpa, pdb, patches}
│   ├── rede/        {ingress.yaml, networkpolicies/}
│   ├── seguranca/   {rbac/, psa-labels.yaml, securitycontext-patches/}
│   ├── batch/       {job-reprocessamento.yaml, cronjob-relatorio.yaml}
│   ├── serverless/  {keda-scaledobject.yaml ou knative-service.yaml}
│   ├── observabilidade/ {values-kube-prometheus.yaml, dashboards/, alertas.yaml}
│   └── argocd/      {project.yaml, app-of-apps.yaml, applications/}
├── scripts/
│   ├── carga.sh
│   ├── caos.sh
│   ├── smoke-test.sh
│   └── verifica-tudo.sh
└── apresentacao/
    ├── pitch.pdf
    └── link-video.md
```

> **Nota:** a estrutura acima é a referência. Adaptações são permitidas e até esperadas (Helm em vez
> de Kustomize, por exemplo), **desde que documentadas no README** e desde que a organização
> continue evidente para quem chega de fora.

### 3.2 Documento de arquitetura (`docs/arquitetura.md`)

Documento de 8 a 15 páginas equivalentes, contendo obrigatoriamente:

1. **Contexto e problema** — resumo do cenário com as próprias palavras do grupo.
2. **Visão geral da solução** — parágrafo executivo de até 15 linhas.
3. **Diagrama de contexto (C4 nível 1)** — sistema, usuários e sistemas externos.
4. **Diagrama de containers (C4 nível 2)** — todos os microsserviços, bancos, broker, ingress,
   worker, função serverless, com protocolos e portas nas setas.
5. **Diagrama de implantação Kubernetes** — nós, namespaces, Deployments/StatefulSets, Services,
   Ingress, e o caminho de uma requisição externa até o banco.
6. **Diagrama de sequência de um pedido** (Mermaid), do `POST /pedidos` até `CONFIRMADO`, mostrando
   o ponto de desacoplamento assíncrono.
7. **Decisões de arquitetura (ADRs)** — no mínimo 4, no formato: contexto, opções consideradas,
   decisão, consequências. Exemplos esperados: banco por serviço vs compartilhado; qual broker;
   qual estratégia serverless; Kustomize vs Helm; Argo CD vs Flux.
8. **Estratégia de dados** — o que é persistido onde, consistência, o que acontece se o worker
   processar a mesma mensagem duas vezes (idempotência).
9. **Estratégia de escalabilidade** — o que escala, por qual métrica, até quanto, e qual é o
   gargalo remanescente.
10. **Estratégia de resiliência** — modos de falha previstos e a mitigação de cada um, em tabela:

    | Falha | Detecção | Mitigação | Evidência |
    |---|---|---|---|
    | Pod da API morre | liveness probe | ReplicaSet recria; PDB protege durante manutenção | teste de caos 1 |
    | Nó cai | node controller | topologySpread reagenda em outro nó | teste de caos 2 |
    | Banco indisponível | readiness da API falha | Retry com backoff; fila retém pedidos | teste de caos 3 |
    | Pico de tráfego | métricas de HPA/fila | HPA + KEDA escalam | teste de carga |

11. **Modelo de segurança** — perfis RBAC, superfícies de exposição, tratamento de segredos, PSA.
12. **Modelo de custos** — como a arquitetura consome recursos e onde estão as alavancas de
    economia.
13. **Limitações conhecidas e próximos passos** — o que ficou de fora e por quê. Esta seção
    **valoriza** a nota: reconhecer limitação é maturidade de engenharia.

### 3.3 Manifestos e empacotamento

- Kustomize (`base` + `overlays/dev` + `overlays/prod`) **ou** Helm chart próprio com dois arquivos
  de values. Ambos são aceitos; duplicação de YAML por copiar-colar entre ambientes é penalizada.
- `kubectl apply -k k8s/overlays/prod` (ou `helm install`) deve funcionar em um cluster limpo,
  após o `infra/bootstrap.sh`.
- Todo manifesto deve ter labels padronizados. Recomenda-se o conjunto oficial:
  `app.kubernetes.io/name`, `app.kubernetes.io/component`, `app.kubernetes.io/part-of: fiap-store`,
  `app.kubernetes.io/version`, `app.kubernetes.io/managed-by`.

### 3.4 Pipeline CI/CD

O pipeline versionado no repositório deve conter, no mínimo:

| Etapa | Obrigatório | Evidência |
|---|---|---|
| Lint / formatação | Sim | Log do job |
| Testes unitários dos serviços | Sim (mínimo 1 teste real por serviço) | Log com testes passando |
| Build da imagem | Sim | Tag gerada a partir do SHA ou de versão semântica |
| Scan de vulnerabilidade (Trivy/Grype) | Sim, com falha do pipeline em CRITICAL | Relatório anexado ao job |
| Push para registro (Docker Hub ou ECR) | Sim | Imagem visível no registro |
| Validação de manifestos (`kubeconform`, `kustomize build`) | Sim | Log do job |
| Atualização automática da tag no overlay + commit | Sim (é o elo com o GitOps) | Commit feito pelo bot no histórico |
| Assinatura de imagem (cosign) | Bônus | Verificação com `cosign verify` |

> **Atenção:** o pipeline **não** deve rodar `kubectl apply` em produção. Quem aplica é o Argo CD.
> Se o pipeline aplicar direto, o grupo perde os pontos de GitOps e ganha uma pergunta difícil na
> banca sobre a diferença entre CI/CD por push e GitOps por pull.

### 3.5 Evidências de teste de carga

Entregar `docs/testes/plano-de-carga.md` e `resultado-carga.md` contendo:

1. **Ferramenta** usada (k6, Locust, JMeter, hey, wrk) e script versionado.
2. **Perfil de carga** definido: usuários virtuais, rampa, duração, mix de operações (por exemplo,
   80% leitura de catálogo, 20% criação de pedido).
3. **Cenário base** (carga normal) e **cenário de pico** (rampa até 10x ou mais).
4. **Métricas coletadas**: throughput (req/s), latência p50/p95/p99, taxa de erro, uso de CPU e
   memória por serviço, número de réplicas ao longo do tempo, profundidade da fila.
5. **Tabela de resultados** no formato:

   | Cenário | VUs | Duração | req/s | p95 (ms) | p99 (ms) | Erros | Réplicas API (min→max) |
   |---|---|---|---|---|---|---|---|
   | Base | 50 | 5 min | | | | | |
   | Pico | 500 | 10 min | | | | | |
   | Sustentado pós-pico | 50 | 5 min | | | | | |

6. **Análise**: o HPA reagiu em quanto tempo? Houve erro durante o escalonamento? Qual foi o
   gargalo (CPU da API? conexões do banco? fila?)? O que o grupo mudaria?
7. **Gráficos** exportados do Grafana durante o teste.

Um teste de carga sem análise vale metade dos pontos. O que se avalia é a **interpretação**.

### 3.6 Evidências de teste de caos

Entregar `docs/testes/plano-de-caos.md` e `resultado-caos.md` com, no mínimo, **quatro experimentos**
no formato hipótese → ação → observação → conclusão:

| # | Experimento | Ação | Hipótese | Métrica observada |
|---|---|---|---|---|
| 1 | Morte de Pod | `kubectl delete pod -l app=pedidos-api` durante carga | Zero erro no cliente; réplica recriada em menos de 30 s | Taxa de erro do k6, tempo até Ready |
| 2 | Drenagem de nó | `kubectl drain <no>` com carga ativa | Zero erro graças a PDB + spread | Taxa de erro, Pods reagendados |
| 3 | Queda do banco | `kubectl scale statefulset store-db --replicas=0` por 60 s | API responde 503 no `/ready`, pedidos ficam na fila e são processados na volta | Nenhum pedido perdido |
| 4 | Queda do broker/worker | Derrubar o worker por 2 min sob carga | Fila cresce, nada se perde, KEDA escala na volta e drena o acúmulo | Profundidade da fila ao longo do tempo |
| 5 (opcional) | Estresse de CPU/rede | LitmusChaos `pod-cpu-hog` ou `pod-network-latency` | HPA compensa; p95 degrada mas sem erro | p95, réplicas |

Cada experimento precisa de evidência (print do Grafana, saída de comando, trecho do relatório do
k6) e de uma conclusão explícita: **a hipótese se confirmou?** Hipótese refutada, documentada e
seguida de correção vale **mais** que hipótese confirmada por sorte.

### 3.7 Relatório de custos AWS

`docs/custos/relatorio-custos.md`, obrigatório mesmo para grupos que usaram k3d (nesse caso, com
estimativa pelo AWS Pricing Calculator, claramente identificada como estimativa):

1. **Inventário de recursos** e custo unitário:

   | Recurso | Especificação | Quantidade | Custo/hora (USD) | Custo/mês estimado (USD) |
   |---|---|---|---|---|
   | EKS control plane | — | 1 | | |
   | Nós EC2 | ex.: t3.medium | 3 | | |
   | EBS (gp3) | ex.: 20 GiB x 2 | 2 | | |
   | Load Balancer (NLB/ALB) | 1 | 1 | | |
   | ECR | armazenamento GB | | | |
   | Transferência de dados | saída GB | | | |
   | **Total** | | | | |

2. **Custo real incorrido** durante o projeto, com print do Cost Explorer (ou justificativa de uso
   local).
3. **Custo por mil pedidos**: divida o custo mensal estimado pela capacidade demonstrada no teste de
   carga. Este número é o que a persona Fernanda pediu, e a banca vai perguntar por ele.
4. **Plano de otimização** com pelo menos 4 alavancas quantificadas: instâncias Spot para o worker,
   right-sizing de requests com base nas métricas reais, scale-to-zero fora do horário comercial,
   Graviton (ARM), redução de retenção de logs, `gp3` com IOPS ajustado em vez de `gp2`.
5. **Comparação Kubernetes vs serverless** para o componente assíncrono: quanto custaria processar o
   mesmo volume em Lambda/Fargate versus nós EC2 sempre ligados.

### 3.8 Vídeo

**12 a 15 minutos**, com áudio, tela compartilhada e participação de todos os integrantes:

| Tempo | Conteúdo |
|---|---|
| 0:00–1:00 | Grupo, integrantes, problema e visão da solução |
| 1:00–3:00 | Passeio pela arquitetura no diagrama, com as decisões principais |
| 3:00–5:00 | Demonstração funcional: navegar no catálogo, criar pedido, ver o status evoluir |
| 5:00–7:00 | Kubernetes na prática: `get nodes`, `get all -n store-prod`, Ingress com TLS, NetworkPolicy bloqueando |
| 7:00–9:00 | Observabilidade: dashboards do Grafana com o sistema sob carga; alerta disparando |
| 9:00–11:00 | Teste de caos ao vivo: matar Pod e drenar nó com carga rodando, mostrando zero erro |
| 11:00–13:00 | GitOps: commit alterando a versão, Argo CD sincronizando sozinho; self-heal |
| 13:00–14:00 | Serverless: KEDA/Knative escalando de zero, ou Lambda sendo acionada |
| 14:00–15:00 | Custos, limitações conhecidas e próximos passos |

### 3.9 Apresentação (pitch)

- 10 minutos, cronometrados.
- Sugestão de roteiro: problema (1 min) → arquitetura (3 min) → demonstração ao vivo ou vídeo curto
  (3 min) → resultados quantitativos de carga/caos/custo (2 min) → limitações e próximos passos
  (1 min).
- **Erro comum a evitar:** gastar 6 minutos explicando o que é Kubernetes. A banca conhece
  Kubernetes. O tempo é para mostrar **as decisões do grupo** e **os números obtidos**.

---

## 4. Rubrica de avaliação (100 pontos)

| Dimensão | Peso |
|---|---|
| D1. Arquitetura e design da solução | 20 |
| D2. Kubernetes (implantação, workloads, rede, armazenamento) | 20 |
| D3. Segurança | 15 |
| D4. GitOps e CI/CD | 15 |
| D5. Serverless e automação | 10 |
| D6. Observabilidade e resiliência | 10 |
| D7. Documentação e defesa | 10 |
| **Total** | **100** |

### D1. Arquitetura e design da solução — 20 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–5)** | Serviço único ou serviços artificialmente separados sem responsabilidade distinta; sem diagrama ou diagrama que não corresponde ao que roda; nenhuma decisão justificada; ausência de desacoplamento assíncrono (RF03 não atendido) |
| **Regular (6–11)** | Três serviços existem mas com fronteiras confusas; diagrama genérico sem protocolos/portas; ADRs ausentes ou copiados; fila presente mas sem tratamento de falha/idempotência; requisitos funcionais parcialmente atendidos |
| **Bom (12–17)** | Três ou mais serviços com responsabilidades claras; diagramas de contexto, containers e implantação coerentes com o cluster real; ao menos 4 ADRs próprios; fluxo assíncrono correto; RF01–RF12 atendidos com poucas lacunas justificadas |
| **Excelente (18–20)** | Fronteiras de serviço defensáveis com argumento de domínio; estratégia de dados explícita (idempotência, consistência eventual tratada, chave de deduplicação); diagrama de sequência preciso; ADRs com opções descartadas e consequências; seção de limitações honesta e tecnicamente madura; arquitetura evidentemente pensada, não montada por acumulação de tutoriais |

### D2. Kubernetes — 20 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–5)** | Não sobe em cluster limpo; recursos criados manualmente; réplica única; sem probes ou sem resources; tudo em `default` |
| **Regular (6–11)** | Sobe com ajustes manuais; workloads básicos corretos mas sem HA (sem spread, sem PDB); Ingress ausente ou sem TLS; persistência frágil (PVC com Deployment, ou dados perdidos); namespaces sem quota |
| **Bom (12–17)** | `apply -k` funciona em cluster limpo; 2+ réplicas com spread entre nós e PDB; probes e resources em todos os workloads; Ingress com TLS e roteamento por host/path; StatefulSet com PVC e persistência comprovada; NetworkPolicy default-deny funcionando; dev/prod isolados com quotas; HPA operante |
| **Excelente (18–20)** | Além do "Bom": `topologySpreadConstraints` com política consciente (`whenUnsatisfiable` justificado), graceful shutdown com `preStop` e `terminationGracePeriodSeconds` coerentes, HPA com `behavior` ajustado e justificado, backup/restore do banco demonstrado, expansão de volume ou StorageClass discutida, labels oficiais padronizados, manifestos limpos e comentados |

### D3. Segurança — 15 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–4)** | Segredo em claro no Git; containers como root; sem RBAC; sem TLS |
| **Regular (5–8)** | Secrets fora do Git mas sem ferramenta de gestão; um ou dois campos de securityContext em parte dos workloads; RBAC existente mas com `verbs: ["*"]`; scan não executado |
| **Regular-Bom (9–12)** | Secrets gerenciados (Sealed/ESO/SOPS) com rotação demonstrada; PSA `restricted` em prod recusando Pod privilegiado; securityContext completo em todos os workloads; 3 perfis RBAC testados com `can-i`; Trivy com correção de CRITICAL; NetworkPolicy validada com caso permitido e bloqueado |
| **Excelente (13–15)** | Além do anterior: menor privilégio evidente (verbos e `resourceNames` enxutos, `automountServiceAccountToken: false` onde cabe); análise de ameaças própria; referência a OWASP Kubernetes Top 10 ou CIS com itens específicos endereçados; scan integrado ao pipeline com portão de qualidade; imagens por digest, base mínima/distroless, SBOM ou assinatura |

### D4. GitOps e CI/CD — 15 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–4)** | Sem pipeline e sem GitOps; deploy manual |
| **Regular (5–8)** | Pipeline que apenas constrói imagem; ou Argo CD instalado com Application dessincronizada; deploy ainda depende de `kubectl apply` humano |
| **Regular-Bom (9–12)** | Pipeline com build, teste, scan e push; Argo CD com `automated/prune/selfHeal` sincronizando prod; self-heal e deploy por commit demonstrados; validação de manifestos no CI |
| **Excelente (13–15)** | Além do anterior: `AppProject` restringindo origens e destinos; app-of-apps ou múltiplos Applications organizados; promoção dev→prod por Git com revisão (PR); rollback por `git revert` demonstrado ao vivo; portão de qualidade no scan; separação clara entre repositório de código e de manifestos (ou pastas com racional documentado); RBAC do próprio Argo CD configurado |

### D5. Serverless e automação — 10 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–2)** | Nenhum componente serverless; sem Job/CronJob |
| **Regular (3–5)** | CronJob ou Job presentes mas frágeis (sem `concurrencyPolicy`, sem `backoffLimit`, sem idempotência); componente serverless mencionado no documento mas não implantado |
| **Bom (6–8)** | Job de reprocessamento idempotente e CronJob com políticas corretas; componente serverless implantado e funcional (KEDA/Knative/Lambda/Fargate) com evidência de escalonamento |
| **Excelente (9–10)** | Scale-to-zero comprovado com medição de cold start; comparação quantitativa Kubernetes vs serverless para o mesmo workload (latência e custo); escolha do gatilho justificada (fila vs CPU vs evento); tratamento de reentrância e de mensagem venenosa (dead-letter) |

### D6. Observabilidade e resiliência — 10 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–2)** | Sem métricas, sem logs centralizados, sem teste de carga nem de caos |
| **Regular (3–5)** | Prometheus instalado sem dashboard significativo; teste de carga rodado sem análise; um experimento de caos superficial |
| **Bom (6–8)** | `/metrics` em todos os serviços, dashboard útil no Grafana, logs centralizados, ao menos 1 alerta; teste de carga com base e pico, tabela de resultados e análise; 4 experimentos de caos com hipótese e conclusão |
| **Excelente (9–10)** | Métricas de negócio além das técnicas (pedidos/min, taxa de recusa por estoque); tracing distribuído ou correlação por `request-id`; alertas com racional de SLO/limiar; teste de carga que **encontrou** um gargalo real, corrigido e re-testado com evidência do antes/depois; caos com hipótese refutada documentada e corrigida |

### D7. Documentação e defesa — 10 pontos

| Nível | Descritor |
|---|---|
| **Insuficiente (0–2)** | README mínimo; ambiente não reproduzível; pitch desorganizado; integrantes que não sabem responder |
| **Regular (3–5)** | Documentação existente mas incompleta; pitch estoura ou sobra muito o tempo; respostas superficiais; participação desigual evidente |
| **Bom (6–8)** | README completo no modelo do Anexo A, ambiente reproduzível por terceiro; atas de sprint presentes; pitch dentro do tempo, claro, com todos falando; respostas corretas à maioria das perguntas |
| **Excelente (9–10)** | Documentação de nível profissional (runbook operacional, ADRs, análise de limitações); pitch com narrativa clara e números concretos; respostas que demonstram domínio, incluindo saber dizer "isso não fizemos, e o caminho seria X"; qualquer integrante consegue defender qualquer parte |

### 4.1 Penalidades

| Situação | Efeito |
|---|---|
| Ausência na defesa sem justificativa formal | Zero individual |
| Vídeo ausente | −15 pontos |
| Segredo real (senha, chave AWS) legível no Git | −20 pontos e orientação de rotação imediata |
| Uso de `latest` em produção | −5 pontos |
| Aplicação de manifestos em prod por pipeline em vez de GitOps | −8 pontos (perde o conceito) |
| Relatório de custos ausente | −8 pontos |
| Integrante sem nenhum commit e sem contribuição comprovada | Nota individual reduzida em até 100% |
| Entrega fora do prazo | −10% por dia; acima de 3 dias, não aceita |
| Uso de IA não declarado | −10 pontos e arguição estendida |

---

## 5. Critérios de bonificação (até 10 pontos extras)

Bônus são **somados após a rubrica** e o total final é limitado a 100 pontos. Só pontuam se
implantados, evidenciados e defendidos — bônus citado no documento e não demonstrado vale zero.

| Bônus | Pontos | O que é exigido para pontuar |
|---|---|---|
| **Service mesh** (Istio, Linkerd) | até 4 | mTLS entre serviços comprovado, política de autorização L7, painel de topologia; explicar o custo do sidecar em memória e latência |
| **Argo Rollouts com canary** | até 4 | Estratégia canary com passos e pausas, análise automática por métrica do Prometheus, rollback automático demonstrado ao vivo (subir versão com erro e ver o rollback) |
| **KEDA com scale-to-zero** | até 3 | `ScaledObject` com `minReplicaCount: 0`, réplicas em zero comprovadas em repouso, medição do tempo até a primeira mensagem ser processada (cold start) |
| **Multi-região ou multi-cluster** | até 4 | Dois clusters/regiões com o mesmo GitOps, estratégia de dados discutida (replicação, RPO/RTO), failover demonstrado ou simulado com evidência |
| **Chaos engineering com LitmusChaos** | até 3 | Ao menos 3 experimentos declarativos (`ChaosEngine`) com `probes` de validação automática, resultados coletados pelo operador, não apenas `kubectl delete pod` manual |
| **FinOps aplicado** | até 3 | Kubecost/OpenCost instalado ou análise equivalente; right-sizing feito com base em métricas reais (com antes/depois de requests); adoção de Spot/Graviton com economia quantificada; custo por mil pedidos calculado |
| **Política de admissão** (Kyverno/Gatekeeper) | até 3 | Políticas que recusam imagem sem digest, sem `resources` ou de registro não confiável, com evidência de recusa |
| **Backup automatizado** (Velero) | até 2 | Backup agendado do namespace e **restore comprovado** em cluster ou namespace novo |
| **Tracing distribuído** (OpenTelemetry + Jaeger/Tempo) | até 3 | Trace de ponta a ponta de um pedido, atravessando pelo menos 3 serviços e a fila |
| **Testes de contrato ou e2e no pipeline** | até 2 | Testes que rodam contra o ambiente de dev após o deploy, bloqueando a promoção em caso de falha |

Máximo somado: **10 pontos**, ainda que a soma dos itens ultrapasse.

---

## 6. Cronograma sugerido em sprints

Seis semanas, três sprints. O checkpoint intermediário ao fim de cada sprint é **obrigatório** e
vale como controle de frequência do trabalho; grupo que falta a um checkpoint perde 3 pontos em D7.

| Semana | Sprint | Marcos e entregas internas | Checkpoint com o professor |
|---|---|---|---|
| **1** | S1 | Formação do grupo e papéis; escolha do tema (principal ou alternativo); rascunho do diagrama C4 de contexto e containers; definição dos 3+ microsserviços e do broker; repositório criado com estrutura de pastas; cluster provisionado (EKS ou k3d) | — |
| **2** | S1 | Serviços esqueleto rodando com `/health`, `/ready`, `/metrics`; imagens no registro; Deployments+Services em `store-dev`; Ingress básico; primeiros ADRs escritos | **CP-A (fim da semana 2)** |
| **3** | S2 | Banco com StatefulSet+PVC e persistência comprovada; broker instalado; fluxo assíncrono do pedido funcionando ponta a ponta; worker consumindo; Job e CronJob | — |
| **4** | S2 | Ambiente `store-prod` com quotas, HPA, PDB, spread; NetworkPolicy default-deny; TLS no Ingress; pipeline de CI construindo, testando e escaneando; Argo CD instalado e sincronizando dev | **CP-B (fim da semana 4)** |
| **5** | S3 | Segurança completa (RBAC 3 perfis, PSA restricted, Secrets gerenciados, hardening); componente serverless implantado; Prometheus+Grafana com dashboards e alerta; GitOps em prod com self-heal | — |
| **6** | S3 | Teste de carga executado e analisado; 4 experimentos de caos; relatório de custos; documento de arquitetura finalizado; vídeo gravado; pitch ensaiado com cronômetro; congelamento (`code freeze`) 48 h antes da defesa | **CP-C + Defesa** |

### 6.1 O que o professor verifica em cada checkpoint intermediário

**CP-A (semana 2) — "existe e roda"**

| Verificação | Como |
|---|---|
| Repositório com estrutura e commits dos integrantes | `git shortlog -sn --all` |
| Cluster ativo com 2+ nós | `kubectl get nodes` |
| Ao menos 2 serviços com Pods Ready em `store-dev` | `kubectl get pods -n store-dev` |
| `/health` e `/ready` respondendo | `curl` via port-forward |
| Diagrama de containers em rascunho | Leitura do `docs/arquitetura.md` |
| Divisão de papéis definida | Ata da sprint 1 |

**CP-B (semana 4) — "funciona e é operável"**

| Verificação | Como |
|---|---|
| Fluxo assíncrono do pedido completo | `POST /pedidos` e acompanhar até `CONFIRMADO` |
| Persistência comprovada | Deletar Pod do banco e consultar o dado |
| `store-prod` com quota, HPA e PDB | `kubectl get quota,hpa,pdb -n store-prod` |
| Pipeline de CI verde com scan | Link do último run |
| Argo CD sincronizando ao menos dev | `kubectl get applications -n argocd` |
| Riscos identificados e plano da sprint 3 | Ata da sprint 2 |

**CP-C (semana 6) — "está pronto para a banca"**

| Verificação | Como |
|---|---|
| Ambiente sobe do zero pelo README | Ensaio de reprodução |
| Todos os RNF endereçados ou justificados | Checklist do Anexo A |
| Resultados de carga e caos com análise | Leitura de `docs/testes/` |
| Relatório de custos com custo por mil pedidos | Leitura de `docs/custos/` |
| Pitch dentro de 10 minutos | Ensaio cronometrado |

### 6.2 Papéis sugeridos dentro do grupo

Papéis não isentam ninguém de entender o todo (a banca pergunta a quem quiser), mas ajudam a
organizar:

| Papel | Responsabilidade principal |
|---|---|
| Tech lead / arquiteto | Diagramas, ADRs, coerência técnica, decide empates |
| Dev de aplicação | Código dos microsserviços, testes, Dockerfiles |
| Engenheiro de plataforma | Cluster, manifestos, Ingress, armazenamento, rede |
| Engenheiro de segurança | RBAC, PSA, Secrets, scan, NetworkPolicy, análise de ameaças |
| SRE / observabilidade | Prometheus, Grafana, alertas, teste de carga e de caos, runbook |

Em grupos de 3, acumule papéis (arquiteto+dev, plataforma+segurança, SRE+CI/CD) e documente na ata.

---

## 7. Roteiro de defesa — perguntas da banca

A banca dispõe de 10 minutos de arguição e escolhe entre 4 e 6 perguntas desta lista, direcionando
cada uma a um integrante específico. As respostas esperadas abaixo servem de gabarito para o
professor **e** de guia de estudo para o grupo: publique esta seção junto com o enunciado.

### P01 — "Por que vocês separaram a aplicação nesses microsserviços, e não em dois, ou em oito?"

*O que se espera:* justificativa por **responsabilidade de domínio** e por **razão de mudança** ou
de escala, não por moda. Boas respostas citam: catálogo é leitura intensiva e escala diferente de
pedidos; estoque precisa de consistência forte e é o gargalo de escrita; o worker escala por fila,
não por HTTP. Deve reconhecer o custo da separação: chamadas de rede, consistência eventual,
complexidade operacional. Resposta fraca: "porque microsserviços são melhores que monolito".

### P02 — "O que acontece, passo a passo, quando o Pod do `pedidos-api` morre no exato instante em que um cliente clica em Comprar?"

*O que se espera:* narrativa completa do ciclo — a requisição em voo é perdida se já foi aceita e
não respondida (por isso o cliente deve reagir a erro/timeout, e por isso existe `preStop` +
`terminationGracePeriodSeconds` para o caso de encerramento **planejado**); o kubelet detecta,
Endpoints/EndpointSlice remove o Pod, o Service para de enviar tráfego, o ReplicaSet cria um novo
Pod, a readiness probe libera o tráfego. O aluno deve distinguir **morte abrupta** de **encerramento
gracioso**, e mencionar que o pedido já publicado na fila não se perde.

### P03 — "Mostre onde está a senha do banco. Quem, no cluster, consegue lê-la?"

*O que se espera:* o grupo abre o repositório e mostra o arquivo **cifrado**, explica a ferramenta
(Sealed Secrets/ESO/SOPS) e onde está a chave que decifra. Depois responde à segunda parte: o
`ServiceAccount` do serviço que a consome, os perfis RBAC com verbo `get` em `secrets` naquele
namespace, e quem tem acesso ao etcd/nós. Resposta excelente menciona que Secret do Kubernetes é
base64, **não** criptografia, e cita criptografia em repouso no etcd (`EncryptionConfiguration` ou
KMS no EKS).

### P04 — "Seu HPA está configurado para CPU a 60%. Por que 60 e não 80? E por que CPU e não a fila?"

*O que se espera:* entendimento de que o alvo precisa de folga para absorver o tempo de reação
(coleta de métrica + decisão + scheduling + startup + readiness), da ordem de dezenas de segundos; a
80% o serviço pode saturar antes de a nova réplica ficar pronta. Sobre a métrica: CPU é proxy ruim
para trabalho assíncrono; a profundidade da fila é a métrica que reflete o backlog real, e é por
isso que o worker usa KEDA. Deve citar `behavior`/`stabilizationWindowSeconds` para evitar
oscilação (flapping).

### P05 — "Eu, como SRE, preciso reiniciar o nó 2 agora. O que acontece com a loja?"

*O que se espera:* `kubectl cordon` para impedir novos agendamentos, `kubectl drain` para evacuar; o
PDB garante `minAvailable`, então a evacuação respeita o mínimo (e pode ficar bloqueada se o PDB for
mal configurado, o que é um risco real que o grupo deve reconhecer); `topologySpreadConstraints`
garante que já havia réplica em outro nó; o `preStop` + graceful shutdown evitam requisições
perdidas; DaemonSets são ignorados no drain. Deve citar a evidência do teste de caos 2, com a taxa
de erro medida.

### P06 — "Se eu apagar todos os recursos de `store-prod` com `kubectl delete ns store-prod`, quanto tempo até a loja voltar? O que se perde?"

*O que se espera:* o Argo CD recria tudo o que está no Git em poucos minutos (self-heal/sync), mas
**o que não está no Git não volta**: os dados do PVC (se o `reclaimPolicy` for `Delete`, o volume
some junto), os Secrets gerados fora do fluxo, o estado da fila. A resposta madura separa
**configuração** (recuperável por GitOps) de **estado** (recuperável só por backup) e aponta o
Velero/dump como a resposta para a segunda categoria. É a pergunta que separa quem entende GitOps de
quem só instalou o Argo CD.

### P07 — "Por que esse componente é serverless e os outros não? O que vocês perderam ao escolher isso?"

*O que se espera:* critério explícito — carga intermitente, tolerância a cold start, execução curta,
sem estado local, custo dominado por ociosidade. E os trade-offs assumidos: latência da primeira
requisição, limites de tempo de execução, dificuldade de depuração, acoplamento ao provedor
(Lambda) ou complexidade adicional no cluster (Knative). Deve trazer o número: quanto tempo levou do
zero até a primeira mensagem processada.

### P08 — "O `dev-junior` do time de vocês tentou ver os logs de produção para debugar um incidente. O que aconteceu, e isso está certo?"

*O que se espera:* demonstração da Role: `can-i get pods/log -n store-prod --as=...` retorna `no`,
e a tentativa real retorna `Forbidden`. A segunda parte é de julgamento: o grupo deve discutir o
equilíbrio entre menor privilégio e capacidade de resposta a incidente, e propor o caminho correto
(acesso de leitura temporário concedido pelo SRE, ou logs centralizados no Grafana/Loki com acesso
controlado fora do RBAC do cluster — que é geralmente a resposta certa). Reconhecer que "segurança
que impede o trabalho é contornada na prática" é sinal de maturidade.

### P09 — "Seu teste de carga mostra p95 de X ms. Onde estava o gargalo, e o que vocês fizeram?"

*O que se espera:* número concreto e uma cadeia causal apoiada em métricas: CPU da API saturada
antes do banco? Pool de conexões do PostgreSQL esgotado? Fila crescendo mais rápido do que o worker
consome? A melhor resposta descreve o antes e o depois de uma correção real (aumentar requests,
ajustar pool, aumentar `maxReplicas`, adicionar índice) com evidência gráfica. Resposta fraca: "não
teve gargalo, ficou tudo bem" — isso normalmente significa que a carga foi baixa demais.

### P10 — "Quanto custa processar mil pedidos na plataforma de vocês? E como vocês baixariam esse custo pela metade?"

*O que se espera:* o número calculado no relatório de custos, com a conta explicada (custo mensal
dividido pela capacidade demonstrada). Depois, alavancas concretas e quantificadas: Spot para
worker, right-sizing com base em métricas reais (a maioria dos grupos superdimensiona requests),
scale-to-zero fora do horário comercial, Graviton, reduzir retenção de logs, consolidar nós. Deve
reconhecer o que **não** dá para cortar sem afetar RNF01 e RNF03.

### P11 — "Como vocês fariam o rollback de uma versão ruim que já está em produção há 10 minutos?"

*O que se espera:* no modelo GitOps, o rollback correto é **`git revert` do commit** que alterou a
tag, com o Argo CD reconciliando — porque `kubectl rollout undo` seria desfeito pelo `selfHeal` na
sequência. O grupo deve saber demonstrar isso. Resposta ainda melhor discute o que fazer se a versão
ruim executou uma migração de banco incompatível: rollback de código não desfaz migração, o que leva
à prática de migrações compatíveis com as duas versões (expand/contract). Se o grupo implementou
Argo Rollouts, deve mostrar o rollback automático por métrica.

### P12 — "O que vocês fariam diferente se recomeçassem hoje? E o que falta para isso ir para produção de verdade?"

*O que se espera:* autocrítica técnica específica, não genérica. Boas respostas citam decisões
concretas que custaram tempo (escolha do broker, banco compartilhado que virou acoplamento, Helm vs
Kustomize) e lacunas conhecidas (backup não testado em cenário de desastre real, DR entre regiões,
gestão de migrações de esquema, testes e2e, política de retenção de dados, SLO formal com error
budget). Esta pergunta costuma diferenciar o grupo "Bom" do "Excelente" em D1 e D7.

### 7.1 Perguntas de reserva (curinga)

| # | Pergunta | Foco |
|---|---|---|
| R1 | "Mostre no cluster, agora, que a NetworkPolicy funciona." | Verificação prática |
| R2 | "Se o mesmo pedido for consumido duas vezes da fila, o que acontece com o estoque?" | Idempotência |
| R3 | "Por que `readOnlyRootFilesystem: true` não quebrou sua aplicação? Onde ela escreve?" | Hardening |
| R4 | "Qual a diferença prática entre a liveness e a readiness probe de vocês? Troque as duas mentalmente e diga o que quebra." | Ciclo de vida |
| R5 | "O que o `prune: true` do Argo CD faria se alguém removesse por engano um arquivo do repositório?" | Risco de GitOps |
| R6 | "Por que StatefulSet e não Deployment para o banco? O que muda de concreto?" | Armazenamento |

---

## 8. Temas alternativos de Global Solution

Cinco temas completos para o professor rotacionar entre turmas e semestres. Todos mantêm a mesma
rubrica, os mesmos entregáveis e a mesma arquitetura mínima da seção 2.5 — muda o domínio e o
**diferencial técnico exigido**, que substitui ou acrescenta requisitos específicos.

### Tema Alternativo 1 — Plataforma de telemetria IoT

Uma empresa de logística instalou sensores em 5.000 caminhões que enviam, a cada 10 segundos,
posição GPS, temperatura do baú refrigerado, velocidade e nível de combustível — cerca de 500
mensagens por segundo em regime normal, com rajadas de 5.000/s quando a frota volta da área sem
cobertura e descarrega o buffer local. A plataforma precisa ingerir esse fluxo sem perda, processar
em tempo quase real para gerar alertas (temperatura acima do limite por mais de 5 minutos em carga
refrigerada é prejuízo e problema sanitário), armazenar séries temporais para análise histórica e
oferecer um painel com o mapa da frota e o estado de cada veículo. As personas incluem o operador
de central que precisa reagir a alertas em menos de 1 minuto, o gestor de frota que analisa
tendências semanais e o cliente que rastreia sua carga.

O **diferencial técnico exigido** é a **ingestão de alto volume com escalonamento por backlog**: o
grupo deve usar um broker adequado (Kafka, NATS JetStream ou Redis Streams), dimensionar partições
e consumidores, e escalar os consumidores com **KEDA pelo lag da fila** — não por CPU. É obrigatório
demonstrar o comportamento sob rajada: injetar 10 minutos de mensagens acumuladas de uma vez e
mostrar, em gráfico, o lag subindo, o KEDA escalando, o lag drenando e as réplicas voltando ao
mínimo. O armazenamento deve usar banco de séries temporais (TimescaleDB ou Prometheus de longa
duração) com política de retenção e downsampling explícita, e o grupo precisa calcular o custo de
armazenar 1 bilhão de pontos. A idempotência é crítica: mensagens duplicadas do buffer do caminhão
não podem gerar alerta duplicado.

### Tema Alternativo 2 — Processamento de imagens orientado a eventos

Um marketplace recebe fotos de produtos enviadas por 30 mil vendedores. Cada upload precisa
disparar uma cadeia de processamento: validar formato e tamanho, gerar quatro variações de
thumbnail, remover metadados EXIF (privacidade: fotos de vendedores frequentemente carregam
geolocalização da casa), aplicar marca d'água, executar moderação automática de conteúdo (pode ser
um serviço mock ou um modelo simples) e, ao final, publicar a imagem no catálogo. O volume é
irregular: 90% do tempo há pouquíssimos uploads, mas nas madrugadas de segunda-feira os grandes
vendedores sobem lotes de 20 mil imagens.

O **diferencial técnico exigido** é a **arquitetura event-driven com scale-to-zero e fan-out**: o
pipeline deve ser composto por etapas desacopladas por eventos (EventBridge + Lambda + S3 na AWS,
ou Knative Eventing + Knative Serving no cluster), com cada etapa escalando independentemente de
zero até dezenas de instâncias. É obrigatório medir e reportar o **cold start** de cada etapa, o
custo por mil imagens processadas nas duas abordagens (funções serverless versus Deployment sempre
ligado), e implementar **dead-letter queue** com reprocessamento para imagens que falham. O grupo
deve demonstrar o comportamento com uma carga de 5.000 imagens injetadas de uma vez, mostrando o
sistema partindo do zero, escalando, drenando e voltando a zero, com o gráfico de custo acumulado.

### Tema Alternativo 3 — Plataforma SaaS multi-tenant

Uma startup vende um sistema de gestão para clínicas de pequeno porte. São 200 clínicas clientes,
cada uma com seus usuários, seus dados e seu plano contratado (Bronze, Prata, Ouro), que define
limites de armazenamento, de usuários simultâneos e de recursos computacionais. A promessa comercial
é que uma clínica jamais veja dado de outra e que uma clínica que abuse do sistema não degrade as
demais. Além disso, o time comercial precisa provisionar uma clínica nova em menos de 10 minutos,
sem chamar o time de engenharia.

O **diferencial técnico exigido** é o **isolamento multi-tenant e o provisionamento automatizado**.
O grupo deve escolher e defender um modelo de isolamento (namespace por tenant, banco por tenant,
schema por tenant ou linha por tenant com `tenant_id`), implementando no mínimo o isolamento por
namespace com `ResourceQuota` proporcional ao plano, `LimitRange`, NetworkPolicy de negação total
entre tenants e RBAC por tenant. O onboarding de um tenant novo deve ser **automatizado por GitOps**:
adicionar um arquivo ou uma entrada de configuração no repositório e o Argo CD provisiona namespace,
quotas, banco, políticas e Ingress do subdomínio (`clinica-x.saas.io`) sozinho — demonstrado ao vivo,
do commit ao ambiente pronto. É obrigatório demonstrar o efeito "vizinho barulhento": um tenant
rodando stress de CPU não pode elevar a latência dos demais, comprovado com medição antes/durante/
depois. Bônus natural: operador customizado (CRD + controller) para o recurso `Tenant`.

### Tema Alternativo 4 — Sistema de saúde com dados sensíveis e compliance

Uma rede de laboratórios de análises clínicas precisa de uma plataforma para registrar exames,
disponibilizar resultados a pacientes e médicos, e integrar com o sistema do convênio. Os dados são
os mais sensíveis possíveis pela LGPD (dados de saúde são categoria especial): resultado de exame,
CPF, histórico clínico. Um vazamento é incidente de notificação obrigatória à ANPD. Os requisitos de
negócio incluem: paciente vê só os próprios exames, médico vê os exames dos próprios pacientes,
atendente registra a coleta mas não vê o resultado, auditor vê o registro de acessos mas nenhum dado
clínico.

O **diferencial técnico exigido** é a **segurança e a auditabilidade de ponta a ponta**. Além de todo
o baseline de segurança da GS, o grupo deve implementar: criptografia dos campos sensíveis na
aplicação, com a chave gerenciada fora do cluster (AWS KMS ou equivalente com Sealed Secrets);
criptografia em repouso do etcd documentada; audit log do API server habilitado, coletado e
consultável, com demonstração de rastreabilidade ("quem leu o Secret X e quando"); trilha de
auditoria **na aplicação** registrando cada acesso a resultado de exame; política de retenção e
rotina de expurgo (CronJob) de dados de pacientes inativos, com log da operação; um perfil RBAC
`auditor` que lê eventos e logs mas tem `pods/exec` explicitamente negado; e política de admissão
(Kyverno/Gatekeeper) recusando qualquer Pod que não atenda ao baseline de segurança. O documento de
arquitetura deve conter uma **análise de ameaças** (STRIDE simplificado) e um mapeamento
requisito-LGPD → controle técnico implementado. Este é o tema mais indicado para turmas com perfil
de segurança.

### Tema Alternativo 5 — Plataforma de streaming de eventos financeiros

Uma fintech processa transações de cartão em tempo real: autorização, antifraude, liquidação e
conciliação. O volume é de 3.000 transações por segundo em horário de pico, com requisito de
resposta de autorização em **menos de 200 ms p99** — acima disso a bandeira nega a transação e o
cliente perde a venda. Nenhuma transação pode ser perdida ou processada duas vezes: cobrar duas
vezes o mesmo cartão é incidente regulatório. Ao fim do dia, um processo de conciliação compara o
que foi autorizado com o que foi liquidado e gera o arquivo de fechamento para o banco.

O **diferencial técnico exigido** é a combinação de **baixa latência com garantia de exatamente-uma-
vez lógica**. O grupo deve: separar claramente o caminho síncrono crítico (autorização, que precisa
de latência previsível, logo sem cold start, com réplicas quentes, requests/limits ajustados,
possivelmente `Guaranteed` QoS e afinidade de nó) do caminho assíncrono (liquidação e conciliação,
que toleram latência e escalam por fila); implementar idempotência real com chave de deduplicação
persistida e demonstrar, no teste, o reenvio da mesma transação sem efeito duplicado; garantir
ordenação onde ela importa (partição por cartão ou conta); e comprovar o p99 sob carga real com
teste dedicado, incluindo o efeito de um scale-up no meio do pico (o novo Pod frio degrada o p99?
por quanto tempo?). A conciliação diária é um `CronJob` ou Job que deve ser reexecutável sem gerar
duplicidade. O relatório de resiliência deve tratar explicitamente o cenário "o worker caiu depois
de debitar e antes de confirmar" e como o sistema se recupera.

### 8.1 Como o professor escolhe o tema da turma

| Perfil da turma | Tema recomendado |
|---|---|
| Turma padrão, primeira oferta | Tema principal (e-commerce resiliente) |
| Turma com forte perfil de dados | Alternativo 1 (telemetria IoT) |
| Turma com interesse em serverless | Alternativo 2 (processamento de imagens) |
| Turma com perfil de produto/SaaS | Alternativo 3 (multi-tenant) |
| Turma com perfil de segurança/compliance | Alternativo 4 (saúde e LGPD) |
| Turma avançada, com bom repertório | Alternativo 5 (financeiro em tempo real) |

Grupos podem propor **tema próprio**, desde que aprovado pelo professor até o fim da semana 1,
mediante uma página descrevendo cenário, personas, RF/RNF e o diferencial técnico proposto — que
deve ser equivalente em dificuldade aos temas listados.

---

## 9. Política de plágio, uso de IA, autoria e atrasos

**Uso de IA generativa: permitido, obrigatório declarar, proibido não entender.** A seção
`## Uso de IA` do README deve listar as ferramentas usadas e para quais partes do trabalho
(geração de manifestos, código dos serviços, scripts de teste, redação da documentação). A banca
parte do princípio de que houve uso de IA; o que se avalia é o **domínio sobre o resultado**.
Qualquer integrante pode ser questionado sobre qualquer linha entregue, e o não entendimento
reduz a nota do critério correspondente a zero.

**Autoria e trabalho em grupo.**

- O histórico do Git é a evidência primária de participação. Recomenda-se commits frequentes e
  atribuídos corretamente (configure `user.name` e `user.email`); commits em par devem usar
  `Co-authored-by`.
- A tabela de divisão de trabalho no README e as três atas de sprint são evidência secundária.
- Integrante com participação nula recebe nota individual reduzida em até 100%, mesmo que o grupo vá
  bem. O inverso também vale: o grupo não é penalizado pela ausência de um integrante, desde que
  comunique o problema ao professor **antes** do CP-B (semana 4).

**Plágio.**

- Reuso do próprio material dos Check Points: esperado e incentivado.
- Uso de código de terceiros (charts oficiais, exemplos da documentação, bibliotecas): permitido com
  citação no arquivo e no README.
- Entrega substancialmente igual à de outro grupo, do mesmo semestre ou de semestre anterior: nota
  **zero** para todos os grupos envolvidos e encaminhamento à coordenação.
- Repositório público de terceiros apresentado como próprio: nota zero e encaminhamento.

**Segurança da entrega.** Chave AWS, token de registro ou senha real encontrada no repositório
público é incidente real: além da penalidade, o grupo deve rotacionar a credencial imediatamente e
registrar o fato no README, como se faz na indústria.

**Atrasos.** A data da defesa é fixa. Entregas de artefatos (repositório, documento, vídeo) até 3
dias após o prazo perdem 10% ao dia; acima disso não são aceitas. A defesa em si não tem segunda
chamada, exceto por motivo legal documentado, caso em que a coordenação define a data.

---

## 10. Plano de contingência: grupos sem acesso à nuvem

Vale integralmente a tabela de equivalências do Check Point 02, ampliada:

| Requisito da GS | AWS | Alternativa local sem perda de nota |
|---|---|---|
| Cluster multi-nó | EKS via eksctl | `k3d cluster create gs --servers 1 --agents 3` |
| Ingress + TLS | ALB Controller + ACM | NGINX Ingress + cert-manager `selfSigned` |
| Armazenamento | EBS gp3 | `local-path` (k3d) |
| Registro | ECR | Docker Hub ou registro local do k3d |
| Fila | SQS / MSK | RabbitMQ, NATS ou Redis via Helm no cluster |
| Serverless | Lambda / Fargate | **KEDA com scale-to-zero** ou Knative Serving no cluster |
| Segredos | Secrets Manager + ESO | Sealed Secrets ou SOPS + age |
| Observabilidade | CloudWatch / AMP | kube-prometheus-stack + Loki |
| Custos | Cost Explorer | AWS Pricing Calculator (estimativa obrigatória mesmo assim) |

> **Atenção:** o CNI padrão do k3d (Flannel) **não aplica NetworkPolicy**. Crie o cluster com
> `--flannel-backend=none --disable-network-policy` e instale Calico ou Cilium, senão as políticas
> serão ignoradas silenciosamente e o RNF08 não será atendido de fato.

Requisito mínimo de máquina para o cenário local completo: 16 GB de RAM e 6 vCPUs em ao menos uma
máquina do grupo (ou distribuição dos componentes entre máquinas dos integrantes, documentada).
Grupos sem nenhuma máquina compatível devem procurar o professor até o fim da semana 1 para
alocação de recurso do laboratório.

---

## Anexo A — Modelo de README do grupo

Copie o conteúdo abaixo para o `README.md` do repositório e preencha todos os campos. Campos
deixados em branco contam como não entregues.

````markdown
# FIAP Store — Plataforma Cloud-Native Resiliente
Global Solution — Cloud Developer II (Kubernetes e Serverless)

## 1. Identificação
| Integrante | RM | Turma | Papel principal | GitHub |
|---|---|---|---|---|
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |

- Tema: (principal / alternativo N / próprio aprovado em __/__)
- Link do vídeo (12–15 min):
- Link da apresentação:
- Ambiente utilizado: (EKS região __ / k3d / kind / outro)

## 2. Resumo executivo
(Até 15 linhas: qual problema, qual solução, qual o resultado mais relevante obtido — com número.)

## 3. Arquitetura
(Diagrama de containers embutido. Detalhamento completo em `docs/arquitetura.md`.)

| Serviço | Tecnologia | Responsabilidade | Réplicas dev/prod | Escala por |
|---|---|---|---|---|
| catalogo-api |  |  |  |  |
| pedidos-api |  |  |  |  |
| estoque-api |  |  |  |  |
| store-worker |  |  |  |  |
| notificacao-fn |  |  |  |  |
| store-front |  |  |  |  |
| store-db |  |  |  |  |
| broker |  |  |  |  |

## 4. Pré-requisitos
| Ferramenta | Versão usada | Comando de verificação |
|---|---|---|
| kubectl |  | `kubectl version` |
| eksctl / k3d |  |  |
| helm |  | `helm version` |
| docker |  | `docker --version` |
| argocd CLI |  |  |
| k6 / Trivy |  |  |

## 5. Como subir tudo do zero
```bash
# 1. Cluster
./infra/criar-cluster.sh

# 2. Componentes de plataforma (ingress, argocd, prometheus, keda, cert-manager)
./infra/bootstrap.sh

# 3. Aplicacoes (a partir daqui quem aplica e o Argo CD)
kubectl apply -f k8s/argocd/app-of-apps.yaml

# 4. Verificacao
./scripts/verifica-tudo.sh
./scripts/smoke-test.sh
```
Tempo esperado até o sistema pronto: ___ minutos.

## 6. Como usar
| Ação | Comando ou URL |
|---|---|
| Abrir a loja | https://loja.____ |
| Listar produtos | `curl https://api.____/produtos` |
| Criar pedido | `curl -X POST ... ` |
| Consultar pedido | `curl https://api.____/pedidos/{id}` |
| Grafana | http://____ (usuário/senha: ____) |
| Argo CD | http://____ |

## 7. Atendimento aos requisitos
| ID | Requisito | Atendido | Onde comprovar |
|---|---|---|---|
| RF01 |  | Sim/Parcial/Não | arquivo, print ou minuto do vídeo |
| ... | (repetir até RF12 e RNF01..RNF10) | | |

## 8. Segurança
- Estratégia de Secrets escolhida e por quê:
- Perfis RBAC (matriz completa em `docs/seguranca/matriz-rbac.md`):
- Pod Security Admission: dev = ____ / prod = ____
- Resultado do scan (antes → depois): CRITICAL __ → __ | HIGH __ → __

## 9. Resultados de teste de carga
| Cenário | VUs | req/s | p95 | p99 | Erros | Réplicas min→max |
|---|---|---|---|---|---|---|
| Base |  |  |  |  |  |  |
| Pico |  |  |  |  |  |  |

Gargalo encontrado: ____________ | Correção aplicada: ____________

## 10. Resultados de teste de caos
| Experimento | Hipótese | Resultado | Hipótese confirmada? |
|---|---|---|---|
| Morte de Pod |  |  |  |
| Drain de nó |  |  |  |
| Queda do banco |  |  |  |
| Queda do worker |  |  |  |

## 11. Custos
- Custo mensal estimado (USD):
- Custo real incorrido no projeto (USD):
- **Custo por mil pedidos (USD):**
- Principais alavancas de redução:

## 12. Divisão de trabalho
| Integrante | Entregas sob sua responsabilidade | Commits |
|---|---|---|
|  |  |  |

## 13. Uso de IA
| Ferramenta | Onde foi usada | Revisão feita por |
|---|---|---|
|  |  |  |

## 14. Limitações conhecidas e próximos passos
1.
2.
3.

## 15. Como destruir tudo
```bash
./infra/destruir-cluster.sh
```
Evidência da destruição: `docs/custos/evidencias/____.png`
````

---

## Anexo B — Modelo de ata de reunião de sprint

Uma ata por sprint, em `docs/atas/sprint-N.md`. As três atas são entregáveis obrigatórios e são
consultadas em caso de divergência sobre participação individual.

````markdown
# Ata de Sprint N — FIAP Store GS
Grupo: ____________________  |  Data: __/__/____  |  Duração: ___ min
Local/plataforma: __________  |  Sprint: N (semanas __ a __)

## 1. Presentes e ausentes
| Integrante | Presente | Justificativa (se ausente) |
|---|---|---|
|  | Sim/Não |  |

## 2. Revisão da sprint anterior
| Compromisso anterior | Responsável | Status | Observação |
|---|---|---|---|
|  |  | Concluído / Parcial / Não iniciado |  |

## 3. Demonstração do que ficou pronto
(O que foi mostrado funcionando nesta reunião — comandos executados, telas, evidências.)
- 
- 

## 4. Métricas da sprint
| Indicador | Valor |
|---|---|
| Itens planejados |  |
| Itens concluídos |  |
| Commits no período |  |
| Serviços com Pods Ready em dev |  |
| Serviços com Pods Ready em prod |  |
| Cobertura dos RF (n/12) |  |
| Cobertura dos RNF (n/10) |  |

## 5. Decisões técnicas tomadas
| # | Decisão | Alternativas descartadas | Vira ADR? |
|---|---|---|---|
| 1 |  |  | Sim/Não — `docs/adr/___.md` |

## 6. Impedimentos e riscos
| Risco / impedimento | Impacto (A/M/B) | Probabilidade (A/M/B) | Mitigação | Responsável | Prazo |
|---|---|---|---|---|---|
|  |  |  |  |  |  |

## 7. Compromissos para a próxima sprint
| # | Entrega | Responsável | Prazo | Critério de pronto |
|---|---|---|---|---|
| 1 |  |  | __/__ |  |
| 2 |  |  | __/__ |  |
| 3 |  |  | __/__ |  |

## 8. Retrospectiva rápida
- Manter:
- Parar:
- Começar:

## 9. Observações do professor (preenchido no checkpoint)
- Pontos fortes:
- Riscos que o professor identificou:
- Recomendação para a próxima sprint:

Assinaturas (nome + RM de todos os presentes):
````

---

## Anexo C — Checklist final de entrega (autoavaliação do grupo)

Marque tudo antes de submeter. Este checklist é o mesmo que o professor usa na triagem inicial.

| # | Item | OK |
|---|---|---|
| 1 | Tag `gs-final` criada e enviada (`git push origin gs-final`) | |
| 2 | README preenchido integralmente no modelo do Anexo A | |
| 3 | `docs/arquitetura.md` com os 4 diagramas e no mínimo 4 ADRs | |
| 4 | 3 ou mais microsserviços com responsabilidades distintas rodando | |
| 5 | Fluxo assíncrono de pedido funcionando de ponta a ponta | |
| 6 | Banco com StatefulSet + PVC e persistência comprovada | |
| 7 | Ingress com TLS roteando por host/path | |
| 8 | NetworkPolicy default-deny em prod, com teste permitido e bloqueado | |
| 9 | HPA em pelo menos 2 serviços, com evidência de escalonamento | |
| 10 | PDB + spread entre nós, com drain demonstrado | |
| 11 | RBAC com 3 perfis testados por `auth can-i` | |
| 12 | PSA `restricted` em prod recusando Pod privilegiado | |
| 13 | Nenhum segredo legível em nenhum commit de nenhum branch | |
| 14 | Scan de imagem com relatório antes/depois | |
| 15 | Pipeline de CI verde, com scan bloqueante | |
| 16 | Argo CD `Synced/Healthy` com `automated/prune/selfHeal` | |
| 17 | Componente serverless implantado e demonstrado | |
| 18 | Job idempotente + CronJob com `concurrencyPolicy` | |
| 19 | Prometheus + Grafana com dashboard e ao menos 1 alerta | |
| 20 | Teste de carga com tabela de resultados e análise do gargalo | |
| 21 | 4 experimentos de caos com hipótese e conclusão | |
| 22 | Relatório de custos com custo por mil pedidos | |
| 23 | 3 atas de sprint preenchidas | |
| 24 | Vídeo de 12–15 min publicado e acessível | |
| 25 | Pitch ensaiado dentro de 10 minutos, com todos falando | |
| 26 | Cluster de nuvem destruído (se aplicável) e evidência anexada | |

---

# Referências Bibliográficas

Esta seção reúne as fontes usadas na construção da apostila **Cloud Developer II — Kubernetes e
Serverless** e as leituras recomendadas para aprofundamento. As referências seguem a **ABNT NBR
6023:2018**. Sempre que uma obra possui tradução para o português, a edição brasileira aparece na
bibliografia complementar, mas recomenda-se a leitura do original quando o tema envolver recursos
recentes da API do Kubernetes — as traduções costumam ficar uma ou duas versões atrás.

> **Nota:** o Kubernetes tem ciclo de release rápido (três versões menores por ano). Livros
> envelhecem em detalhes de API; documentação oficial, não. Use os livros para entender **conceitos
> e o porquê** das decisões de arquitetura, e a documentação oficial para conferir **campos, versões
> de API e comportamento atual**.

---

## Bibliografia básica

As obras a seguir sustentam a espinha dorsal do semestre. Espera-se que o aluno tenha acesso a pelo
menos três delas (biblioteca física ou assinatura digital da instituição).

**BURNS, Brendan; BEDA, Joe; HIGHTOWER, Kelsey; EVENSON, Lachlan. *Kubernetes: up and running*:
dive into the future of infrastructure. 3. ed. Sebastopol: O'Reilly Media, 2022. ISBN
978-1-098-11020-8.**

Escrito por três dos criadores do Kubernetes (Burns, Beda e Hightower) somados a um mantenedor de
longa data. É o livro de referência da disciplina inteira: cobre arquitetura do control plane
(capítulo 1 da apostila), Pods e Services (capítulo 3), ReplicaSets e Deployments (capítulo 4),
volumes e ConfigMaps (capítulo 7), Jobs e Secrets (capítulo 8) e RBAC (capítulo 9). O tom é direto e
os exemplos são pequenos o bastante para reproduzir em um minikube. Leia os capítulos de
"Deployments" e "RoleBasedAccessControl" antes dos Check Points 2 e 3.

**LUKŠA, Marko; CONNER, Kevin. *Kubernetes in action*. 2. ed. Shelter Island: Manning, 2026. ISBN
978-1-61729-761-8.**

A obra mais didática da lista para quem quer entender *como o Kubernetes funciona por dentro*. A
segunda edição reorganiza o material em torno do modelo de objetos da API, com capítulos longos
sobre o ciclo de vida do Pod, probes, controladores e o loop de reconciliação — exatamente o
vocabulário exigido nos capítulos 1, 3, 4 e 7 da apostila. Os desenhos de fluxo entre kubelet,
API Server e etcd são os melhores disponíveis em livro. Recomendada como leitura de fundo do
semestre inteiro.

**DOMINGUS, Justin; ARUNDEL, John. *Cloud native DevOps with Kubernetes*: building, deploying, and
scaling modern applications in the cloud. 2. ed. Sebastopol: O'Reilly Media, 2022. ISBN
978-1-098-11682-8.**

Foco em operação e cultura, não em API. É a fonte principal para os capítulos 6 (cluster gerenciado
na nuvem), 10 (boas práticas, requests/limits, observabilidade, custo) e 11 (entrega contínua e
GitOps). O capítulo sobre "Managing Pods" e o sobre "Deploying Kubernetes Applications" ancoram a
discussão de Helm e Kustomize feita na apostila. Também traz uma discussão honesta sobre quando
*não* usar Kubernetes, útil para a ponte com serverless.

**IBRYAM, Bilgin; HUSS, Roland. *Kubernetes patterns*: reusable elements for designing cloud native
applications. 2. ed. Sebastopol: O'Reilly Media, 2023. ISBN 978-1-098-13168-5.**

Catálogo de padrões no estilo GoF, mas para workloads em Kubernetes: Sidecar, Ambassador, Adapter,
Init Container, Health Probe, Batch Job, Periodic Job, Stateful Service, Elastic Scale. É a
referência conceitual dos capítulos 3, 4, 7 e 8, e o vocabulário que os alunos devem usar ao
justificar decisões de design nos Check Points. A segunda edição inclui os padrões de Operator e de
escalonamento elástico, que preparam o terreno para KEDA e Knative.

**SAYFAN, Gigi; IBRYAM, Bilgin. *Mastering Kubernetes*: dive into Kubernetes and learn how to create
and operate world-class cloud-native systems. 4. ed. Birmingham: Packt Publishing, 2023. ISBN
978-1-80461-139-5.**

O mais abrangente em superfície: aborda alta disponibilidade do control plane, storage, networking
(CNI), segurança, multi-tenancy, service mesh e serverless em Kubernetes. É a referência de apoio
dos capítulos 5 (networking e namespaces), 6 (EKS) e 10 (segurança). Denso; use-o como consulta
dirigida a um tema, não como leitura linear.

**BURNS, Brendan. *Designing distributed systems*: patterns and paradigms for scalable, reliable
systems using Kubernetes. 2. ed. Sebastopol: O'Reilly Media, 2024. ISBN 978-1-098-15635-0.**

Livro curto e conceitual sobre os blocos de construção de sistemas distribuídos, usando Kubernetes
como substrato. Cobre padrões de contêiner único (sidecar, adapter, ambassador), padrões de
serviço replicado e sharding, e padrões batch (queue, scatter/gather). Sustenta as discussões de
arquitetura dos capítulos 1, 4 e 8 e é excelente leitura para o desafio integrador do capítulo 11.

**RICE, Liz. *Container security*: fundamental technology concepts that protect containerized
applications. Sebastopol: O'Reilly Media, 2020. ISBN 978-1-492-05670-6.**

Explica o que realmente isola um contêiner — namespaces, cgroups, capabilities, seccomp, AppArmor,
user namespaces — e por que "contêiner não é máquina virtual". Base obrigatória para os capítulos 9
(RBAC) e 10 (segurança, Pod Security Admission, imagens mínimas, execução como não-root). Retoma e
aprofunda o conteúdo de Docker visto no primeiro semestre.

---

## Bibliografia complementar

**BURNS, Brendan; VILLALBA, Eddie; STREBEL, Dave; EVENSON, Lachlan. *Kubernetes best practices*:
blueprints for building successful applications on Kubernetes. 2. ed. Sebastopol: O'Reilly Media,
2023. ISBN 978-1-098-14216-2.**

Coletânea de recomendações práticas organizadas por tema: setup de desenvolvimento, GitOps,
monitoramento, gerenciamento de estado, admission control, política e multi-cluster. Leitura direta
para o capítulo 10 e para justificar as escolhas do capítulo 11.

**ROSSO, Josh; LANDER, Rich; BRAND, Alex; HARRIS, John. *Production Kubernetes*: building successful
application platforms. Sebastopol: O'Reilly Media, 2021. ISBN 978-1-492-09230-8.**

Escrito por consultores que implantaram dezenas de clusters em produção. Trata das decisões que a
apostila só tangencia: escolha de CNI, ingress controller, estratégia de tenancy, pipeline de
imagens e política. Ótimo complemento aos capítulos 5, 6 e 10.

**MARTIN, Andrew; HAUSENBLAS, Michael. *Hacking Kubernetes*: threat-driven analysis and defense.
Sebastopol: O'Reilly Media, 2021. ISBN 978-1-492-08173-9.**

Aborda o cluster pela ótica do atacante: escape de contêiner, abuso de ServiceAccount, exposição do
kubelet, supply chain. Cada ataque vem acompanhado da defesa correspondente. Leitura de apoio dos
capítulos 9 e 10 e do laboratório de RBAC.

**YUEN, Billy; MATYUSHENTSEV, Alexander; EKENSTAM, Todd; SUEN, Jesse. *GitOps and Kubernetes*:
continuous deployment with Argo CD, Jenkins X, and Flux. Shelter Island: Manning, 2021. ISBN
978-1-61729-727-4.**

Escrito por engenheiros da Intuit, criadores do Argo CD. É a referência bibliográfica direta do
capítulo 11: modelo pull vs. push, drift detection, estratégias de repositório (mono vs. multi),
gestão de segredos em GitOps e progressive delivery.

**BUTCHER, Matt; FARINA, Matt; DOLITSKY, Josh. *Learning Helm*: managing apps on Kubernetes.
Sebastopol: O'Reilly Media, 2021. ISBN 978-1-492-08365-8.**

Escrito por criadores do Helm. Cobre estrutura de chart, templates, valores, dependências,
hooks, provenance e distribuição via OCI. Apoia os capítulos 10 e 11, onde Helm aparece como
empacotador dos manifestos da FIAP Store.

**VYAS, Jay; LOVE, Chris. *Core Kubernetes*. Shelter Island: Manning, 2022. ISBN
978-1-61729-755-7.**

Vai fundo nos componentes internos: etcd, scheduler, kube-proxy (iptables vs. IPVS), CNI, CSI e
CRI. Indicado para quem quer entender o capítulo 1 além do diagrama e para depurar problemas de
rede do capítulo 5.

**HOHN, Alan. *The book of Kubernetes*: a complete guide to container orchestration. San Francisco:
No Starch Press, 2022. ISBN 978-1-7185-0264-2.**

Constrói o entendimento de baixo para cima, começando por namespaces e cgroups do Linux e chegando
ao cluster. Excelente ponte entre o primeiro semestre (Docker) e o capítulo 1 desta apostila.

**BEYER, Betsy; JONES, Chris; PETOFF, Jennifer; MURPHY, Niall Richard (org.). *Site reliability
engineering*: how Google runs production systems. Sebastopol: O'Reilly Media, 2016. ISBN
978-1-491-92912-4. Disponível em: https://sre.google/sre-book/table-of-contents/. Acesso em:
6 ago. 2026.**

O "livro do SRE". Define SLI, SLO, error budget, toil e postmortem sem culpa — conceitos usados no
capítulo 4 (disponibilidade, PodDisruptionBudget) e no capítulo 10 (observabilidade e operação).
Disponível gratuitamente em HTML no site oficial.

**CHESTER, Jacques. *Knative in action*. Shelter Island: Manning, 2021. ISBN 978-1-61729-664-2.**

Único livro dedicado ao Knative na lista. Explica Serving (Revisions, Routes, escala a zero),
Eventing (Brokers, Triggers, CloudEvents) e o modelo de autoscaling por concorrência. Referência das
seções "Ponte com Serverless" dos capítulos 4, 8 e 11.

**SBARSKI, Peter; CUI, Yan; NAIR, Ajay. *Serverless architectures on AWS*. 2. ed. Shelter Island:
Manning, 2022. ISBN 978-1-61729-542-3.**

Serverless do lado da AWS: Lambda, API Gateway, EventBridge, Step Functions, padrões de integração e
modelo de custo. Complementa o capítulo 6 (nuvem) e todas as seções "Ponte com Serverless" que
mencionam Lambda e Fargate.

**ROBERTS, Mike; CHAPIN, John. *What is serverless?*. Sebastopol: O'Reilly Media, 2017. ISBN
978-1-491-98416-1.**

Relatório curto (cerca de 60 páginas, distribuição gratuita pela O'Reilly) que define BaaS e FaaS,
discute trade-offs, lock-in e custo. Leitura de 40 minutos, ideal como texto introdutório de
serverless na primeira semana.

**BURNS, Brendan; BEDA, Joe; HIGHTOWER, Kelsey. *Kubernetes básico*: mergulhe no futuro da
infraestrutura. São Paulo: Novatec, 2020. ISBN 978-85-7522-824-1.**

Tradução da 2ª edição de *Kubernetes: up and running*. Recomendada aos alunos com menos conforto em
inglês técnico como apoio aos capítulos 1 a 8; atenção, porém, ao fato de descrever APIs anteriores
ao Kubernetes 1.25 — confira sempre os campos na documentação oficial antes de aplicar um manifesto.

**IBRYAM, Bilgin; HUSS, Roland. *Padrões para Kubernetes*: elementos reutilizáveis no design de
aplicações nativas de nuvem. São Paulo: Novatec, 2019. ISBN 978-85-7522-814-2.**

Tradução da 1ª edição de *Kubernetes patterns*. Os padrões fundamentais, comportamentais e
estruturais permanecem válidos; os padrões avançados foram bastante ampliados na 2ª edição em
inglês.

**ARUNDEL, John; DOMINGUS, Justin. *DevOps nativo de nuvem com Kubernetes*: como construir,
implantar e escalar aplicações modernas na nuvem. São Paulo: Novatec, 2019. ISBN
978-85-7522-778-7.**

Tradução da 1ª edição de *Cloud native DevOps with Kubernetes*. Boa porta de entrada em português
para os temas de operação, custo e cultura DevOps discutidos nos capítulos 6 e 10.

**FERNANDO, Jeferson. *Descomplicando o Kubernetes*. [S. l.]: LINUXtips, 2020. Repositório GitHub,
licença GPL-3.0. Disponível em: https://github.com/badtuxx/DescomplicandoKubernetes. Acesso em:
6 ago. 2026.**

Curso-livro gratuito em português brasileiro, organizado em "days" progressivos, com laboratórios
práticos. Excelente material de reforço em PT-BR para os capítulos 2 a 9; a linguagem é informal,
mas os comandos são corretos e frequentemente atualizados pela comunidade.

---

## Documentação oficial e fontes primárias

A regra da disciplina é simples: **em caso de divergência entre um livro e a documentação oficial,
vence a documentação oficial** — desde que se observe a versão correspondente. Todos os endereços
abaixo foram verificados na data de acesso indicada.

### Kubernetes

**THE KUBERNETES AUTHORS. *Kubernetes documentation*. [S. l.]: The Linux Foundation, 2026.
Disponível em: https://kubernetes.io/docs/home/. Acesso em: 6 ago. 2026.**

Portal principal. Selecione a versão do cluster no seletor superior antes de copiar qualquer
manifesto.

**THE KUBERNETES AUTHORS. *Concepts*. In: *Kubernetes documentation*. [S. l.]: The Linux Foundation,
2026. Disponível em: https://kubernetes.io/docs/concepts/. Acesso em: 6 ago. 2026.**

Seção conceitual: arquitetura do cluster, workloads, Services, storage, configuração, segurança e
scheduling. Base dos capítulos 1, 3, 4, 5, 7 e 8.

**THE KUBERNETES AUTHORS. *Tasks*. In: *Kubernetes documentation*. [S. l.]: The Linux Foundation,
2026. Disponível em: https://kubernetes.io/docs/tasks/. Acesso em: 6 ago. 2026.**

Receitas passo a passo (configurar probes, injetar Secrets, expor aplicações, depurar Pods). Use-a
durante os laboratórios.

**THE KUBERNETES AUTHORS. *Reference*. In: *Kubernetes documentation*. [S. l.]: The Linux
Foundation, 2026. Disponível em: https://kubernetes.io/docs/reference/. Acesso em: 6 ago. 2026.**

Referência normativa da API, incluindo a *API Reference* por versão, flags dos componentes, labels e
annotations bem conhecidas e a política de depreciação.

**THE KUBERNETES AUTHORS. *kubectl quick reference (cheat sheet)*. In: *Kubernetes documentation*.
[S. l.]: The Linux Foundation, 2026. Disponível em:
https://kubernetes.io/docs/reference/kubectl/quick-reference/. Acesso em: 6 ago. 2026.**

Folha de referência oficial do kubectl; o Apêndice A desta apostila é uma versão comentada e
ampliada dela.

**THE KUBERNETES AUTHORS. *Kubernetes blog*. [S. l.]: The Linux Foundation, 2026. Disponível em:
https://kubernetes.io/blog/. Acesso em: 6 ago. 2026.**

Notas de release e anúncios de depreciação. Consulte antes de atualizar um cluster.

**THE KUBERNETES AUTHORS. *minikube documentation*. [S. l.]: The Linux Foundation, 2026. Disponível
em: https://minikube.sigs.k8s.io/docs/. Acesso em: 6 ago. 2026.**

Referência do capítulo 2: drivers, addons, perfis, `minikube tunnel` e `minikube mount`.

### CNCF e ecossistema

**CLOUD NATIVE COMPUTING FOUNDATION. *CNCF: Cloud Native Computing Foundation*. San Francisco, 2026.
Disponível em: https://www.cncf.io/. Acesso em: 6 ago. 2026.**

Entidade que hospeda Kubernetes, Argo, KEDA, Knative, Prometheus e dezenas de outros projetos.

**CLOUD NATIVE COMPUTING FOUNDATION. *CNCF cloud native interactive landscape*. San Francisco, 2026.
Disponível em: https://landscape.cncf.io/. Acesso em: 6 ago. 2026.**

Mapa do ecossistema por categoria e nível de maturidade (Sandbox, Incubating, Graduated). Útil no
capítulo 1 para dimensionar o espaço de soluções.

**CLOUD NATIVE COMPUTING FOUNDATION. *Cloud native definition v1.1*. San Francisco, 2024.
Repositório GitHub. Disponível em: https://github.com/cncf/toc/blob/main/DEFINITION.md. Acesso em:
6 ago. 2026.**

Definição canônica de "cloud native", citada no capítulo 1.

### AWS

**AMAZON WEB SERVICES. *Amazon EKS user guide*. Seattle, 2026. Disponível em:
https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html. Acesso em: 6 ago. 2026.**

Guia oficial do serviço gerenciado usado no capítulo 6: criação de cluster, node groups, Fargate,
add-ons, IRSA/Pod Identity, atualização de versão.

**AMAZON WEB SERVICES. *Amazon EKS best practices guide*. Seattle, 2026. Disponível em:
https://docs.aws.amazon.com/eks/latest/best-practices/. Acesso em: 6 ago. 2026.**

Guia de boas práticas em cinco eixos (segurança, confiabilidade, eficiência, custo e excelência
operacional). O código-fonte é mantido publicamente em
https://github.com/aws/aws-eks-best-practices. Referência dos capítulos 6 e 10.

**AMAZON WEB SERVICES. *AWS Fargate for Amazon EKS*. Seattle, 2026. Disponível em:
https://docs.aws.amazon.com/eks/latest/userguide/fargate.html. Acesso em: 6 ago. 2026.**

Base da seção "Ponte com Serverless" do capítulo 6.

**AMAZON WEB SERVICES. *AWS Lambda developer guide*. Seattle, 2026. Disponível em:
https://docs.aws.amazon.com/lambda/latest/dg/welcome.html. Acesso em: 6 ago. 2026.**

Referência de FaaS usada nas comparações de modelo de custo e cold start.

**AMAZON WEB SERVICES. *AWS Command Line Interface user guide (version 2)*. Seattle, 2026.
Disponível em: https://docs.aws.amazon.com/cli/latest/userguide/. Acesso em: 6 ago. 2026.**

Instalação e configuração do AWS CLI v2 (Apêndice E).

**WEAVEWORKS; AMAZON WEB SERVICES. *eksctl: the official CLI for Amazon EKS*. [S. l.], 2026.
Disponível em: https://eksctl.io/. Acesso em: 6 ago. 2026.**

Ferramenta usada no laboratório do capítulo 6 para criar e destruir o cluster EKS.

### Empacotamento e entrega

**HELM AUTHORS. *Helm documentation*. [S. l.]: Cloud Native Computing Foundation, 2026. Disponível
em: https://helm.sh/docs/. Acesso em: 6 ago. 2026.**

Charts, templates, valores, releases e repositórios OCI.

**SIG CLI. *Kustomize documentation*. [S. l.]: The Kubernetes Authors, 2026. Disponível em:
https://kubectl.docs.kubernetes.io/references/kustomize/. Acesso em: 6 ago. 2026.**

Referência de campos do `kustomization.yaml`; visão geral em https://kustomize.io/.

**ARGO PROJECT. *Argo CD documentation*. [S. l.]: Cloud Native Computing Foundation, 2026.
Disponível em: https://argo-cd.readthedocs.io/en/stable/. Acesso em: 6 ago. 2026.**

Referência principal do capítulo 11: Applications, Projects, sync policies, ApplicationSet e
gestão de acesso.

**ARGO PROJECT. *Argo Rollouts documentation*. [S. l.]: Cloud Native Computing Foundation, 2026.
Disponível em: https://argo-rollouts.readthedocs.io/en/stable/. Acesso em: 6 ago. 2026.**

Progressive delivery: blue-green, canary e análise automatizada. Usado no capítulo 11.

**SUSE. *Rancher Manager documentation*. [S. l.], 2026. Disponível em:
https://ranchermanager.docs.rancher.com/. Acesso em: 6 ago. 2026.**

Gestão multi-cluster com Rancher, tema da segunda metade do capítulo 11.

**SUSE. *Fleet: GitOps at scale*. [S. l.], 2026. Disponível em: https://fleet.rancher.io/. Acesso
em: 6 ago. 2026.**

Motor de GitOps embarcado no Rancher, comparado ao Argo CD no capítulo 11.

### Serverless sobre Kubernetes

**KEDA AUTHORS. *KEDA: Kubernetes Event-driven Autoscaling — documentation*. [S. l.]: Cloud Native
Computing Foundation, 2026. Disponível em: https://keda.sh/docs/. Acesso em: 6 ago. 2026.**

ScaledObject, ScaledJob, scalers e escala a zero. Referência das pontes com serverless dos capítulos
4 e 8.

**KNATIVE AUTHORS. *Knative documentation*. [S. l.]: Cloud Native Computing Foundation, 2026.
Disponível em: https://knative.dev/docs/. Acesso em: 6 ago. 2026.**

Knative Serving e Eventing; base conceitual de FaaS sobre Kubernetes.

**CLOUDEVENTS AUTHORS. *CloudEvents specification*. [S. l.]: Cloud Native Computing Foundation,
2026. Disponível em: https://cloudevents.io/. Acesso em: 6 ago. 2026.**

Formato padronizado de evento usado pelo Knative Eventing.

### Observabilidade, segurança e política

**PROMETHEUS AUTHORS. *Prometheus documentation*. [S. l.]: Cloud Native Computing Foundation, 2026.
Disponível em: https://prometheus.io/docs/. Acesso em: 6 ago. 2026.**

Modelo de dados, PromQL, service discovery em Kubernetes e alerting. Apoia o endpoint `/metrics` do
`store-api` e o capítulo 10.

**AQUA SECURITY. *Trivy documentation*. [S. l.], 2026. Disponível em: https://trivy.dev/. Acesso em:
6 ago. 2026.**

Scanner de vulnerabilidades, misconfigurations, segredos e SBOM em imagens e manifestos. Usado no
laboratório de segurança do capítulo 10.

**NIRMATA. *Kyverno documentation*. [S. l.]: Cloud Native Computing Foundation, 2026. Disponível em:
https://kyverno.io/docs/. Acesso em: 6 ago. 2026.**

Policy engine declarativo para Kubernetes (validate, mutate, generate, verifyImages). Alternativa ao
OPA/Gatekeeper discutida no capítulo 10.

**THE KUBERNETES AUTHORS. *Pod Security Admission*. In: *Kubernetes documentation*. [S. l.]: The
Linux Foundation, 2026. Disponível em: https://kubernetes.io/docs/concepts/security/pod-security-admission/.
Acesso em: 6 ago. 2026.**

Substituto das PodSecurityPolicies (removidas na versão 1.25); níveis privileged, baseline e
restricted. Central no capítulo 10.

**OWASP FOUNDATION. *OWASP Kubernetes Top Ten*. [S. l.], 2022. Disponível em:
https://owasp.org/www-project-kubernetes-top-ten/. Acesso em: 6 ago. 2026.**

Dez principais riscos de segurança em Kubernetes, de "Insecure Workload Configurations" (K01) a
"Vulnerable Components" (K10). Roteiro do laboratório de auditoria do capítulo 10.

**SOUPPAYA, Murugiah; MORELLO, John; SCARFONE, Karen. *Application container security guide*.
Gaithersburg: National Institute of Standards and Technology, 2017. (NIST Special Publication
800-190). DOI 10.6028/NIST.SP.800-190. Disponível em:
https://nvlpubs.nist.gov/nistpubs/specialpublications/nist.sp.800-190.pdf. Acesso em: 6 ago. 2026.**

Documento normativo que estrutura riscos de contêineres em cinco camadas (imagem, registry,
orquestrador, contêiner e host) com contramedidas. Referência formal citada no capítulo 10 e a
fonte adequada quando o trabalho exigir uma norma, e não um blog.

**CENTER FOR INTERNET SECURITY. *CIS Kubernetes benchmark*. East Greenbush, 2026. Disponível em:
https://www.cisecurity.org/benchmark/kubernetes. Acesso em: 6 ago. 2026.**

Checklist de hardening auditável (requer cadastro gratuito para download). Base das ferramentas de
verificação automatizada de conformidade.

---

## Artigos e papers

**VERMA, Abhishek; PEDROSA, Luis; KORUPOLU, Madhukar; OPPENHEIMER, David; TUNE, Eric; WILKES, John.
Large-scale cluster management at Google with Borg. In: EUROPEAN CONFERENCE ON COMPUTER SYSTEMS
(EuroSys), 10., 2015, Bordeaux. *Proceedings* [...]. New York: ACM, 2015. p. 1-17. DOI
10.1145/2741948.2741964. Disponível em: https://research.google/pubs/large-scale-cluster-management-at-google-with-borg/.
Acesso em: 6 ago. 2026.**

O paper fundacional. Descreve o Borg, sistema interno do Google que inspirou o Kubernetes: modelo de
jobs e tasks, alocação por prioridade, quotas, *bin packing*, e o achado de que a utilização média
de um datacenter dobra quando cargas batch e de latência compartilham as mesmas máquinas. Leitura
obrigatória do capítulo 1 — em especial as seções 2 (arquitetura), 5 (lições aprendidas) e a
discussão sobre o "alloc" como antecessor do Pod.

**BURNS, Brendan; GRANT, Brian; OPPENHEIMER, David; BREWER, Eric; WILKES, John. Borg, Omega, and
Kubernetes. *Communications of the ACM*, New York, v. 59, n. 5, p. 50-57, maio 2016. DOI
10.1145/2890784. Disponível em: https://queue.acm.org/detail.cfm?id=2898444. Acesso em:
6 ago. 2026.**

Retrospectiva escrita pelos próprios autores dos três sistemas. Explica *por que* o Kubernetes é
declarativo, por que a API é orientada a objetos com labels em vez de hierarquia rígida, e por que
o controlador é um loop de reconciliação. Texto curto (8 páginas) e o melhor resumo possível da
filosofia de design discutida no capítulo 1.

**BEYER, Betsy; JONES, Chris; PETOFF, Jennifer; MURPHY, Niall Richard (org.). *Site reliability
engineering*: how Google runs production systems. Sebastopol: O'Reilly Media, 2016. Disponível em:
https://sre.google/sre-book/table-of-contents/. Acesso em: 6 ago. 2026.**

Consultar especialmente os capítulos "Service Level Objectives", "Eliminating Toil" e "Embracing
Risk" ao estudar disponibilidade (capítulo 4) e operação (capítulo 10). O volume seguinte, *The
site reliability workbook* (2018), disponível em https://sre.google/workbook/table-of-contents/,
traz exercícios de definição de SLO aplicáveis diretamente ao `store-api`.

**JONAS, Eric; SCHLEIER-SMITH, Johann; SREEKANTI, Vikram; TSAI, Chia-Che; KHANDELWAL, Anurag; PU,
Qifan; SHANKAR, Vaishaal; CARREIRA, Joao; KRAUTH, Karl; YADWADKAR, Neeraja; GONZALEZ, Joseph E.;
POPA, Raluca Ada; STOICA, Ion; PATTERSON, David A. *Cloud programming simplified*: a Berkeley view
on serverless computing. Berkeley: EECS Department, University of California, Berkeley, 2019.
(Technical Report UCB/EECS-2019-3). Disponível em:
https://www2.eecs.berkeley.edu/Pubs/TechRpts/2019/EECS-2019-3.html. Acesso em: 6 ago. 2026.**

Análise acadêmica de referência sobre serverless: define o modelo, enumera as limitações (estado,
comunicação entre funções, cold start, previsibilidade de custo) e projeta a evolução da área. É a
base conceitual de todas as seções "Ponte com Serverless" da apostila. Também disponível como
preprint no arXiv (arXiv:1902.03383).

**OPENGITOPS. *GitOps principles v1.0.0*. [S. l.]: Cloud Native Computing Foundation, 2021.
Disponível em: https://github.com/open-gitops/documents/blob/v1.0.0/PRINCIPLES.md. Acesso em:
6 ago. 2026.**

Documento curto que fixa os quatro princípios de GitOps — declarativo, versionado e imutável,
puxado automaticamente (*pulled automatically*) e continuamente reconciliado. É a definição que a
apostila adota no capítulo 11; o portal do grupo de trabalho está em https://opengitops.dev/.

**CLOUD NATIVE COMPUTING FOUNDATION. *CNCF annual survey*. San Francisco, 2025. Disponível em:
https://www.cncf.io/reports/. Acesso em: 6 ago. 2026.**

Pesquisa anual com dados de adoção de Kubernetes, service mesh, GitOps, serverless e WebAssembly.
Fonte de números para justificar decisões nos Check Points — cite sempre a edição e o ano do
relatório consultado.

**CLOUD NATIVE COMPUTING FOUNDATION. *CNCF technology radar*. San Francisco. Disponível em:
https://radar.cncf.io/. Acesso em: 6 ago. 2026.**

Posicionamento (adopt, trial, assess, hold) de ferramentas por tema — entrega contínua,
observabilidade, segurança de supply chain. Útil para justificar a escolha de ferramentas no
trabalho final.

---

## Cursos, certificações e trilhas

As certificações abaixo são **práticas** (exame em terminal, cluster real, consulta permitida à
documentação oficial durante a prova) e alinham-se diretamente ao conteúdo desta apostila. O
Apêndice F traz o mapeamento capítulo a capítulo com os domínios de cada exame.

| Trilha | Instituição | Foco | Relação com a apostila | Link |
|---|---|---|---|---|
| CKA — Certified Kubernetes Administrator | The Linux Foundation / CNCF | Operação e administração do cluster | Capítulos 1, 2, 4, 5, 7, 8, 9 | https://training.linuxfoundation.org/certification/certified-kubernetes-administrator-cka/ |
| CKAD — Certified Kubernetes Application Developer | The Linux Foundation / CNCF | Projeto e implantação de aplicações | Capítulos 3, 4, 7, 8, 10 | https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/ |
| CKS — Certified Kubernetes Security Specialist | The Linux Foundation / CNCF | Segurança de cluster e workload (exige CKA válido) | Capítulos 9, 10 | https://training.linuxfoundation.org/certification/certified-kubernetes-security-specialist-cks/ |
| KCNA — Kubernetes and Cloud Native Associate | The Linux Foundation / CNCF | Fundamentos, prova de múltipla escolha | Capítulos 1, 2, 11 | https://training.linuxfoundation.org/certification/kubernetes-cloud-native-associate/ |
| AWS Certified Developer – Associate | Amazon Web Services | Desenvolvimento de aplicações na AWS, incluindo Lambda e ECR | Capítulos 6 e pontes com serverless | https://aws.amazon.com/certification/certified-developer-associate/ |
| AWS Certified DevOps Engineer – Professional | Amazon Web Services | Automação, CI/CD, monitoramento e IaC | Capítulos 6, 10, 11 | https://aws.amazon.com/certification/certified-devops-engineer-professional/ |
| Catálogo completo de certificações | The Linux Foundation | Todas as trilhas Kubernetes e bundles | — | https://training.linuxfoundation.org/certification-catalog/ |

**Simuladores e plataformas de prática**

**KILLER.SH. *Kubernetes exam simulator*. [S. l.], 2026. Disponível em: https://killer.sh/. Acesso
em: 6 ago. 2026.** — Simulador oficial incluído na compra dos exames CKA, CKAD e CKS. Duas sessões
de 36 horas por exame adquirido; o nível de dificuldade é deliberadamente superior ao da prova real.

**KODEKLOUD. *Kubernetes courses and labs*. [S. l.], 2026. Disponível em: https://kodekloud.com/.
Acesso em: 6 ago. 2026.** — Cursos com laboratórios em navegador (KodeKloud Engineer / playgrounds).
As trilhas "Kubernetes for the Absolute Beginners" e "CKA/CKAD Certification Course" acompanham bem
os capítulos 2 a 9.

**HIGHTOWER, Kelsey. *Kubernetes the hard way*. [S. l.], 2026. Repositório GitHub. Disponível em:
https://github.com/kelseyhightower/kubernetes-the-hard-way. Acesso em: 6 ago. 2026.** — Tutorial que
monta um cluster do zero, componente por componente, sem nenhuma ferramenta de automação: gerar CA e
certificados, subir etcd, kube-apiserver, kube-controller-manager, kube-scheduler, kubelet e
kube-proxy manualmente. Não é o caminho para *usar* Kubernetes, é o caminho para *entendê-lo*.
Sugerido como desafio opcional após o capítulo 1; reserve de 4 a 6 horas.

**THE LINUX FOUNDATION. *Introduction to Kubernetes (LFS158)*. [S. l.], 2026. Disponível em:
https://training.linuxfoundation.org/training/introduction-to-kubernetes/. Acesso em: 6 ago. 2026.**
— Curso introdutório gratuito, também ofertado via edX. Boa revisão dos capítulos 1 a 5.

**AMAZON WEB SERVICES. *AWS Skill Builder*. Seattle, 2026. Disponível em:
https://skillbuilder.aws/. Acesso em: 6 ago. 2026.** — Trilhas gratuitas de EKS, ECR, Fargate e
Lambda, incluindo laboratórios guiados. Complemento direto do capítulo 6.

**AMAZON WEB SERVICES. *AWS EKS workshop*. Seattle, 2026. Disponível em: https://www.eksworkshop.com/.
Acesso em: 6 ago. 2026.** — Workshop prático oficial de EKS, com módulos de autoscaling,
observabilidade, segurança e GitOps. Estimativa de custo indicada em cada módulo.

> **Atenção — custos:** os laboratórios de EKS, ECR, ALB e NAT Gateway são cobrados por hora, mesmo
> ociosos. Ao terminar qualquer prática do capítulo 6, execute `eksctl delete cluster --name <nome>`
> e confira o console de faturamento no dia seguinte. Um cluster EKS esquecido por um mês consome
> facilmente todo o crédito educacional disponível.

---

## Canais e comunidades

| Canal | O que é | Endereço |
|---|---|---|
| CNCF no YouTube | Palestras completas das KubeCon, webinars técnicos e sessões dos projetos | https://www.youtube.com/@cncf |
| Kubernetes no YouTube | Canal oficial do projeto: office hours, releases e tutoriais | https://www.youtube.com/@KubernetesCommunity |
| Kubernetes Slack | Espaço oficial da comunidade; mais de 200 canais (`#kubernetes-novice`, `#kubectl`, `#sig-network`, `#argo-cd`, `#keda`, `#br-users`) | https://slack.k8s.io/ |
| Kubernetes Community | Calendário de reuniões dos SIGs, atas e processo de contribuição | https://github.com/kubernetes/community |
| KubeCon + CloudNativeCon | Conferência oficial do ecossistema (Europa, América do Norte, Índia, Japão, China); há edição virtual e bolsas de estudo | https://www.cncf.io/events/ |
| CNCF Community Groups | Encontros locais, incluindo os grupos brasileiros (São Paulo, Rio de Janeiro, Belo Horizonte, Porto Alegre, Recife) | https://community.cncf.io/ |
| Kubernetes Contributor Site | Como dar o primeiro passo como contribuidor (issues `good first issue`) | https://www.kubernetes.dev/ |
| Stack Overflow — tag `kubernetes` | Perguntas e respostas; excelente acervo de mensagens de erro reais | https://stackoverflow.com/questions/tagged/kubernetes |
| Reddit r/kubernetes | Discussões, notícias e relatos de produção | https://www.reddit.com/r/kubernetes/ |
| Discuss Kubernetes | Fórum oficial (mais formal que o Slack, com histórico pesquisável) | https://discuss.kubernetes.io/ |
| LINUXtips (PT-BR) | Comunidade brasileira; vídeos, "Descomplicando" e Discord ativo | https://linuxtips.io/ |
| DevOps Brasil / Kubernetes Brasil | Comunidades brasileiras em Telegram e Discord; canal `#br-users` no Slack oficial | https://community.cncf.io/ (buscar por grupos no Brasil) |
| Learnk8s (blog) | Artigos longos e ilustrados sobre internals, networking e autoscaling | https://learnk8s.io/blog |
| Last Week in Kubernetes Development | Boletim semanal do desenvolvimento upstream | https://lwkd.info/ |

> **Na prática:** entrar no Slack oficial e acompanhar `#kubernetes-novice` por um semestre ensina
> tanto quanto um livro. A regra de etiqueta é simples: pesquise antes, descreva o que já tentou,
> cole a saída completa de `kubectl describe` (removendo dados sensíveis) e informe a versão do
> cluster.

---

## Como citar em ABNT

Este mini-guia cobre o que basta para os Check Points e o trabalho final da disciplina. A norma de
referência é a **ABNT NBR 6023:2018** (referências) combinada com a **ABNT NBR 10520:2023**
(citações em documentos).

### Regras gerais

- O **sobrenome do autor** vai em CAIXA ALTA na lista de referências; o prenome, com inicial
  maiúscula. Exemplo: `LUKŠA, Marko`.
- Até três autores, todos são citados, separados por ponto e vírgula. **Com quatro ou mais**, é
  permitido indicar apenas o primeiro seguido de `et al.` — mas, para papers avaliados, prefira
  listar todos.
- O **destaque do título** (negrito, itálico ou sublinhado) deve ser uniforme em todo o trabalho.
  Nesta apostila usa-se itálico para o título e texto normal para o subtítulo, separados por dois
  pontos.
- A lista de referências vai no **fim do trabalho**, em ordem alfabética, alinhada à esquerda
  (sem justificar) e sem numeração.
- Quando o local de publicação não é identificável, use `[S. l.]`; sem editora, `[s. n.]`; sem data,
  estime entre colchetes, por exemplo `[2024]`.

### Modelos de referência

| Tipo | Modelo |
|---|---|
| Livro | SOBRENOME, Nome. *Título*: subtítulo. Edição. Local: Editora, ano. |
| Capítulo de livro | SOBRENOME, Nome. Título do capítulo. In: SOBRENOME, Nome (org.). *Título do livro*. Local: Editora, ano. p. inicial-final. |
| Artigo de periódico | SOBRENOME, Nome. Título do artigo. *Nome do Periódico*, Local, v. X, n. Y, p. inicial-final, mês ano. |
| Trabalho em evento | SOBRENOME, Nome. Título. In: NOME DO EVENTO, n., ano, Local. *Anais* [...]. Local: Editora, ano. p. inicial-final. |
| Documento eletrônico | AUTOR ou INSTITUIÇÃO. *Título*. Local, ano. Disponível em: URL. Acesso em: dia mês abreviado ano. |
| Norma técnica | INSTITUIÇÃO. *Código*: título. Local, ano. |
| Repositório de código | AUTOR. *Nome do repositório*. Local, ano. Repositório GitHub. Disponível em: URL. Acesso em: data. |

> **Atenção:** o mês na data de acesso é **abreviado em português e em minúsculas**, com ponto,
> exceto maio: `jan.`, `fev.`, `mar.`, `abr.`, `maio`, `jun.`, `jul.`, `ago.`, `set.`, `out.`,
> `nov.`, `dez.`. A forma correta aqui é `Acesso em: 6 ago. 2026.`

### Citação direta curta (até três linhas)

Vai no corpo do texto, entre aspas duplas, com autor, ano e página.

```
Segundo Burns et al. (2016, p. 51), "o Kubernetes deve muito de seu design ao Borg, mas foi
construído em torno de uma API declarativa e centrada em objetos".
```

Ou, com a autoria entre parênteses:

```
A API declarativa é apontada como a principal ruptura em relação aos sistemas anteriores
(BURNS et al., 2016, p. 51).
```

### Citação direta longa (mais de três linhas)

Recuo de 4 cm da margem esquerda, fonte menor (por exemplo, 10 pt), espaçamento simples e **sem
aspas**.

```
       O Borg gerencia dezenas de milhares de máquinas em cada célula e é responsável por
       executar praticamente todas as cargas de trabalho de produção do Google, misturando
       serviços de longa duração e jobs em lote na mesma infraestrutura para elevar a
       utilização média do datacenter (VERMA et al., 2015, p. 1).
```

### Citação indireta (paráfrase)

Não leva aspas nem página obrigatória, mas exige autor e ano.

```
A separação entre estado desejado e estado observado é o que permite ao Kubernetes convergir
automaticamente após falhas de nó (LUKŠA; CONNER, 2026).
```

### Citação de documentação eletrônica

Documentação oficial normalmente **não tem autor pessoal**. Use a entidade responsável como autor e
inclua a data de acesso. No texto:

```
Desde a versão 1.25, as PodSecurityPolicies foram removidas em favor do Pod Security Admission
(THE KUBERNETES AUTHORS, 2026).
```

E na lista de referências:

```
THE KUBERNETES AUTHORS. Pod Security Admission. In: Kubernetes documentation. [S. l.]:
The Linux Foundation, 2026. Disponível em:
https://kubernetes.io/docs/concepts/security/pod-security-admission/. Acesso em: 6 ago. 2026.
```

Quando a página indicar explicitamente a versão do produto, inclua-a no título — por exemplo,
`Kubernetes documentation (v1.34)` —, pois a URL muda de conteúdo entre releases.

### Citação de trecho de código ou de arquivo de configuração

Trechos de código copiados de terceiros também precisam de crédito. Coloque a referência
imediatamente abaixo do bloco:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.27
          ports:
            - containerPort: 80
```

`Fonte: adaptado de THE KUBERNETES AUTHORS (2026).`

### Erros mais comuns nos Check Points

| Erro | Por que está errado | Forma correta |
|---|---|---|
| Colar apenas a URL "crua" como referência | Falta autoria, título e data de acesso | Use o modelo de documento eletrônico completo |
| `Acesso em: 06/08/2026` | Formato de data fora da norma | `Acesso em: 6 ago. 2026.` |
| Citar "Kubernetes.io" como autor | O site não é o autor; a entidade responsável é | `THE KUBERNETES AUTHORS` |
| Referência sem correspondência no texto | A norma exige que toda referência listada tenha sido citada | Cite no texto ou remova da lista |
| Citação no texto sem entrada na lista | Quebra a rastreabilidade | Acrescente a referência completa ao final |
| Copiar parágrafo inteiro entre aspas sem página | Citação direta exige indicação de página | Inclua `p. XX` ou, em documento web sem paginação, a seção |
| Traduzir livremente um trecho e apresentá-lo como citação direta | Tradução é paráfrase | Use citação indireta ou indique `tradução nossa` |
| Usar conteúdo gerado por IA sem verificação | Modelos inventam ISBN, edição e autoria | Confira sempre em catálogo da editora ou base bibliográfica antes de citar |

> **Na prática:** gerenciadores como Zotero (gratuito, https://www.zotero.org/) e Mendeley aplicam o
> estilo ABNT automaticamente e importam metadados a partir do DOI ou do ISBN. Mesmo assim, **revise
> a saída**: eles frequentemente erram o campo de edição e a capitalização de subtítulos.

---

# Apêndices

Os apêndices reúnem o material de consulta rápida da disciplina. Diferentemente dos capítulos, eles
não são feitos para ser lidos do começo ao fim: são feitos para ser abertos durante o laboratório,
durante o Check Point e durante a prova prática. Mantenha esta seção impressa ou em uma aba fixa do
navegador.

Todos os comandos e manifestos foram escritos para **Kubernetes 1.33/1.34**, com `kubectl` 1.33 ou
superior. Onde a API pode variar entre versões, há uma nota explícita.

---

## Apêndice A — Cheat sheet de kubectl

### A.1 Anatomia de um comando

```
kubectl [comando] [TIPO] [NOME] [flags]
        |         |      |      |
        |         |      |      +-- -n store-dev, -o yaml, --dry-run=client, -l app=store-api
        |         |      +--------- nome do objeto (opcional em list)
        |         +---------------- pod, deploy, svc, ing, cm, secret, pvc, sa, role...
        +-------------------------- get, describe, create, apply, delete, logs, exec, scale...
```

Abreviações aceitas para os tipos mais usados: `po` (pods), `deploy` (deployments), `rs`
(replicasets), `svc` (services), `ing` (ingresses), `cm` (configmaps), `ns` (namespaces), `no`
(nodes), `pvc` (persistentvolumeclaims), `pv` (persistentvolumes), `sa` (serviceaccounts), `sts`
(statefulsets), `ds` (daemonsets), `cj` (cronjobs), `hpa` (horizontalpodautoscalers), `netpol`
(networkpolicies), `pdb` (poddisruptionbudgets), `quota` (resourcequotas), `limits` (limitranges),
`ep` (endpoints), `crd` (customresourcedefinitions).

### A.2 Contexto, configuração e descoberta

| Comando | Descrição |
|---|---|
| `kubectl version` | Versão do cliente e do servidor |
| `kubectl version --client -o yaml` | Só a versão do cliente, sem contatar o cluster |
| `kubectl cluster-info` | Endereço do control plane e do CoreDNS |
| `kubectl cluster-info dump` | Despejo completo do estado do cluster (para suporte) |
| `kubectl config view` | Mostra o kubeconfig efetivo (com dados sensíveis ocultos) |
| `kubectl config view --raw` | Mostra o kubeconfig com certificados e tokens (cuidado) |
| `kubectl config get-contexts` | Lista contextos disponíveis; `*` marca o atual |
| `kubectl config current-context` | Nome do contexto ativo |
| `kubectl config use-context minikube` | Troca o contexto ativo |
| `kubectl config set-context --current --namespace=store-dev` | Fixa o namespace padrão do contexto |
| `kubectl config rename-context antigo novo` | Renomeia um contexto |
| `kubectl config delete-context eks-lab` | Remove um contexto do kubeconfig |
| `KUBECONFIG=~/.kube/a:~/.kube/b kubectl config view --flatten > ~/.kube/config` | Funde vários kubeconfig em um |
| `kubectl api-resources` | Lista todos os tipos de recurso, abreviações, grupo de API e se são namespaced |
| `kubectl api-resources --namespaced=false` | Só recursos de escopo de cluster (Node, PV, ClusterRole...) |
| `kubectl api-versions` | Lista todas as versões de API habilitadas no cluster |
| `kubectl explain pod.spec.containers` | Documentação do campo direto do servidor |
| `kubectl explain deploy.spec.strategy --recursive` | Árvore completa de campos do objeto |
| `kubectl auth can-i create deployments -n store-dev` | Testa a própria permissão |
| `kubectl auth whoami` | Mostra a identidade autenticada (usuário, grupos) |

> **Na prática:** `kubectl explain` é o único "manual offline" permitido em todo cluster, e funciona
> mesmo sem internet. Antes de procurar o campo no Google, tente `kubectl explain <tipo>.<caminho>`.

### A.3 Criação e aplicação de recursos

| Comando | Descrição |
|---|---|
| `kubectl apply -f pod.yaml` | Cria ou atualiza a partir do arquivo (declarativo) |
| `kubectl apply -f ./manifests/` | Aplica todos os YAML de um diretório |
| `kubectl apply -f ./manifests/ -R` | Aplica recursivamente, incluindo subdiretórios |
| `kubectl apply -k ./overlays/dev` | Aplica um overlay Kustomize |
| `kubectl apply -f https://exemplo.com/manifesto.yaml` | Aplica direto de uma URL |
| `kubectl create -f pod.yaml` | Cria; falha se o objeto já existir (imperativo) |
| `kubectl replace -f pod.yaml --force` | Deleta e recria o objeto (usar com cautela) |
| `kubectl diff -f deployment.yaml` | Mostra o que mudaria antes de aplicar |
| `kubectl create deployment store-api --image=store-api:1.0 --replicas=3` | Deployment imperativo |
| `kubectl create ns store-dev` | Cria um namespace |
| `kubectl create configmap store-config --from-literal=APP_ENV=dev` | ConfigMap a partir de literais |
| `kubectl create configmap store-config --from-file=./config/` | ConfigMap a partir de um diretório |
| `kubectl create secret generic store-db --from-literal=DB_PASSWORD=troque-me` | Secret genérico |
| `kubectl create secret docker-registry regcred --docker-server=... --docker-username=... --docker-password=...` | Secret de acesso a registry privado |
| `kubectl create secret tls store-tls --cert=tls.crt --key=tls.key` | Secret de certificado TLS |
| `kubectl create job manual-1 --from=cronjob/relatorio-diario` | Dispara um Job a partir de um CronJob |
| `kubectl expose deployment store-api --port=80 --target-port=8000` | Cria um Service para um Deployment |
| `kubectl run debug --image=busybox:1.36 -it --rm --restart=Never -- sh` | Pod temporário e interativo |
| `kubectl set image deployment/store-api store-api=store-api:1.1` | Troca a imagem de um container |
| `kubectl set env deployment/store-api APP_ENV=prod` | Define variável de ambiente |
| `kubectl set resources deployment/store-api --limits=cpu=500m,memory=512Mi` | Ajusta limites |
| `kubectl set serviceaccount deployment/store-api store-api-sa` | Troca a ServiceAccount |

### A.4 Geradores e `--dry-run=client` (o atalho mais importante da prova)

A forma mais rápida de escrever YAML correto é deixar o `kubectl` escrever o esqueleto:

```bash
# Pod
kubectl run store-api --image=docker.io/seuusuario/store-api:1.0 \
  --port=8000 --dry-run=client -o yaml > pod.yaml

# Deployment com 3 réplicas
kubectl create deployment store-api --image=docker.io/seuusuario/store-api:1.0 \
  --replicas=3 --dry-run=client -o yaml > deployment.yaml

# Service ClusterIP para um Deployment existente
kubectl expose deployment store-api --port=80 --target-port=8000 \
  --dry-run=client -o yaml > service.yaml

# Service NodePort direto
kubectl create service nodeport store-api --tcp=80:8000 \
  --dry-run=client -o yaml > svc-nodeport.yaml

# ConfigMap e Secret
kubectl create configmap store-config --from-literal=APP_ENV=dev \
  --dry-run=client -o yaml > configmap.yaml
kubectl create secret generic store-db --from-literal=DB_PASSWORD=s3nh4 \
  --dry-run=client -o yaml > secret.yaml

# CronJob
kubectl create cronjob relatorio --image=store-worker:1.0 --schedule="0 3 * * *" \
  --dry-run=client -o yaml > cronjob.yaml

# Ingress
kubectl create ingress store --rule="store.local/*=store-api:80" \
  --dry-run=client -o yaml > ingress.yaml

# RBAC
kubectl create sa store-api-sa --dry-run=client -o yaml > sa.yaml
kubectl create role leitor --verb=get,list,watch --resource=pods \
  --dry-run=client -o yaml > role.yaml
kubectl create rolebinding leitor-bind --role=leitor \
  --serviceaccount=store-dev:store-api-sa --dry-run=client -o yaml > rb.yaml
```

| Flag | Efeito |
|---|---|
| `--dry-run=client` | Renderiza localmente, não contata o servidor. Ideal para gerar YAML |
| `--dry-run=server` | Envia ao API Server, que valida e aplica admission controllers, mas não persiste |
| `-o yaml` / `-o json` | Formato de saída |
| `--save-config` | Grava a configuração aplicada na annotation `kubectl.kubernetes.io/last-applied-configuration` |

> **Atenção:** o YAML gerado vem com campos ruidosos (`creationTimestamp: null`, `status: {}`,
> `strategy: {}`). Remova-os antes de versionar. O plugin `kubectl neat` (seção A.13) faz isso
> automaticamente.

### A.5 Inspeção e consulta

| Comando | Descrição |
|---|---|
| `kubectl get pods` | Lista Pods do namespace atual |
| `kubectl get pods -A` | Lista Pods de todos os namespaces |
| `kubectl get pods -o wide` | Acrescenta nó, IP do Pod, nominated node e readiness gates |
| `kubectl get pod store-api-0 -o yaml` | Manifesto completo, incluindo status |
| `kubectl get all -n store-dev` | Recursos comuns do namespace (não inclui cm, secret, ing, pvc) |
| `kubectl get pods -l app=store-api,tier=backend` | Filtra por labels (AND) |
| `kubectl get pods -l 'env in (dev,staging)'` | Seletor de conjunto |
| `kubectl get pods -l '!canary'` | Pods sem a label `canary` |
| `kubectl get pods --field-selector status.phase=Running` | Filtra por campo do objeto |
| `kubectl get pods --field-selector spec.nodeName=node-2` | Pods de um nó específico |
| `kubectl get pods --sort-by=.status.startTime` | Ordena a saída |
| `kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'` | Ordena por reinícios |
| `kubectl get pods -w` | Modo watch: acompanha mudanças em tempo real |
| `kubectl get events --sort-by=.lastTimestamp` | Eventos do namespace em ordem cronológica |
| `kubectl get events -A --field-selector type=Warning` | Só avisos, em todo o cluster |
| `kubectl describe pod store-api-0` | Visão consolidada: spec, status, condições e eventos |
| `kubectl describe node node-1` | Capacidade, alocação, taints, condições e Pods do nó |
| `kubectl top nodes` | CPU e memória por nó (exige metrics-server) |
| `kubectl top pods --containers -n store-dev` | Consumo por container |
| `kubectl get pod store-api-0 -o jsonpath='{.status.podIP}'` | Extrai um campo específico |
| `kubectl get --raw /healthz` | Consulta um endpoint do API Server diretamente |
| `kubectl api-resources --verbs=list --namespaced -o name` | Lista tipos para varreduras completas |

### A.6 Saídas customizadas: `-o jsonpath` e `-o custom-columns`

O `jsonpath` navega na estrutura JSON do objeto. É a forma de responder perguntas que a saída padrão
não responde.

```bash
# Todas as imagens em uso no cluster, uma por linha
kubectl get pods -A -o jsonpath='{range .items[*]}{.spec.containers[*].image}{"\n"}{end}' \
  | sort -u

# Nome do Pod + nó + IP
kubectl get pods -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{.spec.nodeName}{"\t"}{.status.podIP}{"\n"}{end}'

# Somente o IP do primeiro Pod que casa com o label
kubectl get pods -l app=store-api -o jsonpath='{.items[0].status.podIP}'

# Endereços atrás de um Service
kubectl get endpointslice -l kubernetes.io/service-name=store-api \
  -o jsonpath='{range .items[*].endpoints[*]}{.addresses[0]}{"\n"}{end}'

# Todos os Secrets referenciados por Deployments
kubectl get deploy -A -o jsonpath='{range .items[*]}{.metadata.namespace}{"/"}{.metadata.name}{": "}{.spec.template.spec.containers[*].envFrom[*].secretRef.name}{"\n"}{end}'

# Node externo/interno
kubectl get nodes -o jsonpath='{range .items[*]}{.metadata.name}{"\t"}{range .status.addresses[*]}{.type}={.address}{" "}{end}{"\n"}{end}'

# Decodificar um Secret (base64)
kubectl get secret store-db -o jsonpath='{.data.DB_PASSWORD}' | base64 -d; echo
```

O `custom-columns` produz tabelas legíveis:

```bash
# Pod, imagem, reinícios e nó
kubectl get pods -o custom-columns=\
'NOME:.metadata.name,IMAGEM:.spec.containers[0].image,RESTARTS:.status.containerStatuses[0].restartCount,NO:.spec.nodeName'

# Requests e limits de cada Deployment
kubectl get deploy -A -o custom-columns=\
'NS:.metadata.namespace,NOME:.metadata.name,CPU_REQ:.spec.template.spec.containers[0].resources.requests.cpu,MEM_LIM:.spec.template.spec.containers[0].resources.limits.memory'

# PVCs com StorageClass e capacidade
kubectl get pvc -A -o custom-columns=\
'NS:.metadata.namespace,NOME:.metadata.name,STATUS:.status.phase,SC:.spec.storageClassName,TAM:.status.capacity.storage'

# Pods rodando como root (auditoria rápida)
kubectl get pods -A -o custom-columns=\
'NS:.metadata.namespace,POD:.metadata.name,RUNASNONROOT:.spec.securityContext.runAsNonRoot'
```

| Formato de saída | Uso |
|---|---|
| `-o wide` | Colunas extras prontas |
| `-o yaml` / `-o json` | Objeto completo |
| `-o name` | Só `tipo/nome`, útil em pipes: `kubectl delete $(kubectl get po -l x=y -o name)` |
| `-o jsonpath='{...}'` | Extração livre de campos |
| `-o jsonpath-file=q.txt` | Expressão jsonpath lida de arquivo |
| `-o custom-columns=A:.path,B:.path` | Tabela customizada |
| `-o custom-columns-file=cols.txt` | Definição de colunas em arquivo |
| `-o go-template='{{...}}'` | Template Go completo, com condicionais e laços |
| `--no-headers` | Suprime o cabeçalho (bom para scripts) |

### A.7 Logs, depuração e acesso ao container

| Comando | Descrição |
|---|---|
| `kubectl logs store-api-0` | Logs do container (único) do Pod |
| `kubectl logs store-api-0 -c sidecar` | Logs de um container específico |
| `kubectl logs -f store-api-0` | Segue os logs em tempo real |
| `kubectl logs --tail=100 store-api-0` | Últimas 100 linhas |
| `kubectl logs --since=15m store-api-0` | Últimos 15 minutos |
| `kubectl logs --since-time=2026-08-06T10:00:00Z store-api-0` | A partir de um instante |
| `kubectl logs -p store-api-0` | Logs da **encarnação anterior** do container (essencial em CrashLoopBackOff) |
| `kubectl logs -l app=store-api --max-log-requests=10 --prefix` | Logs de todos os Pods do label |
| `kubectl logs deploy/store-api --all-containers=true` | Logs via workload, não via Pod |
| `kubectl exec -it store-api-0 -- sh` | Shell interativo dentro do container |
| `kubectl exec store-api-0 -- env` | Executa um comando único |
| `kubectl exec store-api-0 -c sidecar -- cat /etc/nginx/nginx.conf` | Comando em container específico |
| `kubectl debug store-api-0 -it --image=nicolaka/netshoot --target=store-api` | Ephemeral container para depurar (imagem distroless sem shell) |
| `kubectl debug node/node-1 -it --image=busybox:1.36` | Pod privilegiado no host de um nó |
| `kubectl debug store-api-0 --copy-to=store-api-debug --set-image=*=busybox:1.36` | Cópia do Pod com imagem trocada |
| `kubectl cp store-api-0:/app/log.txt ./log.txt` | Copia do container para a máquina |
| `kubectl cp ./patch.py store-api-0:/app/patch.py` | Copia da máquina para o container |
| `kubectl port-forward pod/store-api-0 8080:8000` | Túnel local para o Pod |
| `kubectl port-forward svc/store-api 8080:80` | Túnel local para o Service |
| `kubectl attach -it store-api-0` | Conecta ao processo principal (stdin/stdout) |
| `kubectl events --for pod/store-api-0` | Eventos relacionados a um objeto específico |
| `kubectl get pod store-api-0 -o jsonpath='{.status.containerStatuses[0].lastState}'` | Motivo da última terminação (ex.: OOMKilled) |

> **Na prática:** a imagem `nicolaka/netshoot` traz `dig`, `curl`, `nc`, `tcpdump`, `ss`, `iperf` e
> `nslookup`. É a ferramenta padrão para depurar rede em Kubernetes:
> `kubectl run net --rm -it --image=nicolaka/netshoot --restart=Never -- bash`

### A.8 Edição, patch e anotação

| Comando | Descrição |
|---|---|
| `kubectl edit deploy store-api` | Abre o objeto no editor (`$KUBE_EDITOR` ou `$EDITOR`) |
| `kubectl patch deploy store-api -p '{"spec":{"replicas":5}}'` | Strategic merge patch (padrão) |
| `kubectl patch deploy store-api --type=merge -p '{"metadata":{"labels":{"env":"prod"}}}'` | JSON merge patch |
| `kubectl patch deploy store-api --type=json -p '[{"op":"replace","path":"/spec/replicas","value":4}]'` | JSON patch (RFC 6902) |
| `kubectl label pod store-api-0 env=prod` | Adiciona label |
| `kubectl label pod store-api-0 env=prod --overwrite` | Substitui label existente |
| `kubectl label pod store-api-0 env-` | Remove a label `env` |
| `kubectl annotate ing store "nginx.ingress.kubernetes.io/rewrite-target=/"` | Adiciona annotation |
| `kubectl taint nodes node-1 dedicado=batch:NoSchedule` | Aplica taint ao nó |
| `kubectl taint nodes node-1 dedicado-` | Remove o taint |
| `kubectl cordon node-1` | Impede o agendamento de novos Pods no nó |
| `kubectl uncordon node-1` | Reabilita o agendamento |
| `kubectl drain node-1 --ignore-daemonsets --delete-emptydir-data` | Esvazia o nó respeitando PDBs |

### A.9 Escala, rollout e disponibilidade

| Comando | Descrição |
|---|---|
| `kubectl scale deploy store-api --replicas=5` | Escala manualmente |
| `kubectl scale deploy store-api --replicas=5 --current-replicas=3` | Escala só se o estado atual bater |
| `kubectl scale sts store-db --replicas=3` | Escala um StatefulSet |
| `kubectl autoscale deploy store-api --min=2 --max=10 --cpu-percent=70` | Cria um HPA (autoscaling/v2) |
| `kubectl get hpa store-api -w` | Acompanha as decisões do autoscaler |
| `kubectl rollout status deploy/store-api` | Espera e reporta o progresso do rollout |
| `kubectl rollout status deploy/store-api --timeout=120s` | Com prazo máximo |
| `kubectl rollout history deploy/store-api` | Lista as revisões |
| `kubectl rollout history deploy/store-api --revision=3` | Detalha uma revisão |
| `kubectl rollout undo deploy/store-api` | Volta para a revisão anterior |
| `kubectl rollout undo deploy/store-api --to-revision=2` | Volta para uma revisão específica |
| `kubectl rollout pause deploy/store-api` | Congela o rollout (para aplicar várias mudanças) |
| `kubectl rollout resume deploy/store-api` | Retoma o rollout |
| `kubectl rollout restart deploy/store-api` | Recria todos os Pods (relê ConfigMaps e Secrets) |
| `kubectl get pdb -A` | Lista PodDisruptionBudgets e as disrupções permitidas |
| `kubectl wait --for=condition=Ready pod -l app=store-api --timeout=120s` | Bloqueia até os Pods ficarem prontos |
| `kubectl wait --for=condition=complete job/carga-inicial --timeout=300s` | Espera um Job terminar |
| `kubectl wait --for=delete pod/store-api-0 --timeout=60s` | Espera a remoção efetiva |

> **Nota:** `kubectl rollout restart` funciona alterando a annotation
> `kubectl.kubernetes.io/restartedAt` do template do Pod. É a forma correta de recarregar
> configuração; matar Pods com `kubectl delete pod` também funciona, mas não respeita a estratégia
> de rollout.

### A.10 Rede, Services e DNS

| Comando | Descrição |
|---|---|
| `kubectl get svc -A` | Todos os Services, com tipo, ClusterIP e portas |
| `kubectl get endpoints store-api` | IPs por trás do Service (API legada, ainda útil) |
| `kubectl get endpointslices -l kubernetes.io/service-name=store-api` | API atual de endpoints |
| `kubectl describe svc store-api` | Selector, portas, tipo e endpoints resolvidos |
| `kubectl get ing -A` | Todos os Ingresses, com hosts e endereços |
| `kubectl describe ing store` | Regras, backends, TLS e eventos do controller |
| `kubectl get ingressclass` | Controladores de Ingress instalados |
| `kubectl get netpol -n store-dev` | NetworkPolicies do namespace |
| `kubectl describe netpol default-deny -n store-dev` | Detalha as regras de ingress e egress |
| `kubectl port-forward svc/store-api 8080:80 -n store-dev` | Acesso local sem expor o Service |
| `kubectl proxy --port=8001` | Proxy autenticado para a API em `http://localhost:8001` |
| `kubectl run dns --rm -it --image=busybox:1.36 --restart=Never -- nslookup store-api.store-dev.svc.cluster.local` | Testa resolução DNS |
| `kubectl run curl --rm -it --image=curlimages/curl --restart=Never -- curl -s http://store-api.store-dev/health` | Testa conectividade HTTP entre Pods |
| `kubectl get pods -n kube-system -l k8s-app=kube-dns` | Estado do CoreDNS |
| `kubectl logs -n kube-system -l k8s-app=kube-dns --tail=50` | Logs do CoreDNS |
| `kubectl get cm coredns -n kube-system -o yaml` | Corefile em uso |

Formato canônico de nome DNS de um Service:

```
<service>.<namespace>.svc.cluster.local
```

Para Pods de um StatefulSet com headless Service:

```
<pod>.<service-headless>.<namespace>.svc.cluster.local
store-db-0.store-db.store-dev.svc.cluster.local
```

### A.11 RBAC, ServiceAccounts e autorização

| Comando | Descrição |
|---|---|
| `kubectl get sa -A` | Todas as ServiceAccounts |
| `kubectl get role,rolebinding -n store-dev` | RBAC namespaced |
| `kubectl get clusterrole,clusterrolebinding` | RBAC de escopo de cluster |
| `kubectl describe clusterrole view` | Detalha as regras de um ClusterRole padrão |
| `kubectl auth can-i create pods -n store-dev` | Testa a própria permissão |
| `kubectl auth can-i '*' '*' --all-namespaces` | Verifica se é administrador do cluster |
| `kubectl auth can-i delete deploy --as=system:serviceaccount:store-dev:store-api-sa` | Impersona uma ServiceAccount |
| `kubectl auth can-i list secrets --as=joao --as-group=devs -n store-prod` | Impersona usuário e grupo |
| `kubectl auth can-i --list -n store-dev` | Lista tudo o que o usuário atual pode fazer |
| `kubectl auth reconcile -f rbac.yaml` | Aplica RBAC preservando regras agregadas |
| `kubectl create token store-api-sa --duration=1h` | Emite um token de curta duração |
| `kubectl get clusterrolebinding -o wide \| grep cluster-admin` | Quem tem poder total no cluster |
| `kubectl get rolebinding -A -o custom-columns='NS:.metadata.namespace,NOME:.metadata.name,ROLE:.roleRef.name,SUJEITOS:.subjects[*].name'` | Auditoria de bindings |

> **Atenção:** `kubectl auth can-i` responde apenas sobre **autorização**. Um `no` pode vir do RBAC,
> mas um `yes` não garante sucesso: admission controllers (Pod Security Admission, Kyverno,
> ResourceQuota) ainda podem rejeitar a operação depois.

### A.12 Recursos, cotas e capacidade

| Comando | Descrição |
|---|---|
| `kubectl top nodes` | Uso de CPU/memória por nó |
| `kubectl top pods -A --sort-by=memory` | Pods que mais consomem memória |
| `kubectl describe node node-1 \| grep -A 10 "Allocated resources"` | Requests já reservados no nó |
| `kubectl get quota -A` | ResourceQuotas de todos os namespaces |
| `kubectl describe quota store-dev-quota -n store-dev` | Usado vs. limite de cada recurso |
| `kubectl get limitrange -n store-dev -o yaml` | Defaults e limites por container |
| `kubectl get pods -A -o json \| jq -r '.items[] \| select(.spec.containers[].resources.limits == null) \| .metadata.namespace + "/" + .metadata.name'` | Pods sem limits definidos |
| `kubectl get pc` (`priorityclasses`) | Classes de prioridade para preempção |
| `kubectl get --raw /metrics` | Métricas Prometheus do API Server |
| `kubectl get nodes -o custom-columns='NO:.metadata.name,CPU:.status.capacity.cpu,MEM:.status.capacity.memory,PODS:.status.capacity.pods'` | Capacidade nominal dos nós |

### A.13 Limpeza e remoção

| Comando | Descrição |
|---|---|
| `kubectl delete -f deployment.yaml` | Remove exatamente o que o arquivo cria |
| `kubectl delete -k ./overlays/dev` | Remove o que o overlay cria |
| `kubectl delete pod store-api-0` | Remove um Pod (o controlador pode recriá-lo) |
| `kubectl delete pods -l app=store-api` | Remove por label |
| `kubectl delete pods --field-selector status.phase=Failed -A` | Limpa Pods falhos |
| `kubectl delete pod store-api-0 --grace-period=0 --force` | Remoção forçada (último recurso) |
| `kubectl delete ns store-dev` | Remove o namespace e **tudo** dentro dele |
| `kubectl delete all --all -n store-dev` | Remove workloads e Services, mas não cm/secret/pvc/ing |
| `kubectl delete all,cm,secret,pvc,ing,netpol --all -n store-dev` | Limpeza abrangente do namespace |
| `kubectl delete job --field-selector status.successful=1` | Remove Jobs concluídos |
| `kubectl delete pvc store-db-data-store-db-0` | Remove o PVC (o PV segue a reclaimPolicy) |
| `kubectl patch pvc meu-pvc -p '{"metadata":{"finalizers":null}}'` | Destrava um PVC preso em Terminating |
| `minikube delete --all --purge` | Remove todos os clusters locais e seus dados |
| `eksctl delete cluster --name fiap-store --region us-east-1` | Destrói o cluster EKS e a stack CloudFormation |

> **Atenção:** `kubectl delete all --all` **não** apaga ConfigMaps, Secrets, PVCs, Ingresses,
> NetworkPolicies nem CRDs. O nome `all` é enganoso: refere-se a um grupo específico de tipos.
> Para apagar tudo com segurança, apague o namespace.

### A.14 Aliases, autocompletar e produtividade

Adicione ao `~/.bashrc` (ou `~/.zshrc`):

```bash
# Autocompletar
source <(kubectl completion bash)      # zsh: source <(kubectl completion zsh)

# Alias curto + autocompletar para o alias
alias k=kubectl
complete -o default -F __start_kubectl k

# Atalhos de uso diário
alias kg='kubectl get'
alias kd='kubectl describe'
alias kaf='kubectl apply -f'
alias kdel='kubectl delete'
alias kl='kubectl logs -f'
alias kex='kubectl exec -it'
alias kgp='kubectl get pods -o wide'
alias kga='kubectl get all'
alias kgpw='kubectl get pods -w'
alias kns='kubectl config set-context --current --namespace'
alias kctx='kubectl config use-context'
alias kdry='kubectl --dry-run=client -o yaml'

# Editor usado por kubectl edit
export KUBE_EDITOR="code --wait"       # ou vim, nano

# Namespace atual visível no prompt (com kube-ps1 instalado)
# source /usr/share/kube-ps1/kube-ps1.sh && PS1='$(kube_ps1)'$PS1
```

Variáveis de ambiente úteis:

| Variável | Efeito |
|---|---|
| `KUBECONFIG` | Caminho(s) do kubeconfig; aceita lista separada por `:` |
| `KUBE_EDITOR` | Editor usado por `kubectl edit` |
| `KUBECTL_EXTERNAL_DIFF` | Ferramenta usada por `kubectl diff` (ex.: `colordiff -N -u`) |
| `KUBECTL_INTERACTIVE_DELETE=true` | Pede confirmação antes de deletar |

Atalhos de digitação na prova prática:

```bash
# Cria um manifesto e já abre no editor
k create deploy store-api --image=nginx:1.27 $do > d.yaml && vim d.yaml
# onde: export do="--dry-run=client -o yaml"

# Deleção rápida sem esperar o grace period (só em lab)
export now="--force --grace-period=0"
k delete pod store-api-0 $now
```

### A.15 Plugins krew indispensáveis

O [krew](https://krew.sigs.k8s.io/) é o gerenciador de plugins oficial do `kubectl`.

```bash
# Instalação (Linux/macOS)
(
  set -x; cd "$(mktemp -d)" &&
  OS="$(uname | tr '[:upper:]' '[:lower:]')" &&
  ARCH="$(uname -m | sed -e 's/x86_64/amd64/' -e 's/\(arm\)\(64\)\?.*/\1\2/' -e 's/aarch64$/arm64/')" &&
  KREW="krew-${OS}_${ARCH}" &&
  curl -fsSLO "https://github.com/kubernetes-sigs/krew/releases/latest/download/${KREW}.tar.gz" &&
  tar zxvf "${KREW}.tar.gz" &&
  ./"${KREW}" install krew
)
export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"   # acrescente ao ~/.bashrc

kubectl krew install ctx ns tree neat stern who-can images access-matrix df-pv
```

| Plugin | Comando | Para que serve |
|---|---|---|
| `ctx` | `kubectl ctx` / `kubectl ctx eks-prod` | Lista e troca de contexto sem editar o kubeconfig |
| `ns` | `kubectl ns store-dev` / `kubectl ns -` | Troca o namespace padrão; `-` volta ao anterior |
| `tree` | `kubectl tree deploy store-api` | Mostra a árvore de ownership (Deployment → ReplicaSet → Pods) |
| `neat` | `kubectl get pod x -o yaml \| kubectl neat` | Remove campos gerados pelo servidor e devolve YAML limpo para versionar |
| `stern` | `stern store-api -n store-dev` | Segue logs de vários Pods e containers ao mesmo tempo, com cores por Pod |
| `who-can` | `kubectl who-can delete pods -n store-prod` | Inverso do `auth can-i`: quem pode fazer isso |
| `images` | `kubectl images -A` | Tabela de imagens em uso, com tamanho |
| `access-matrix` | `kubectl access-matrix -n store-dev` | Matriz de verbos permitidos por recurso |
| `df-pv` | `kubectl df-pv` | Uso real de disco de cada PersistentVolume |

Ferramentas fora do krew que valem instalar:

| Ferramenta | Para que serve | Endereço |
|---|---|---|
| `k9s` | TUI completa: navegar, ver logs, entrar em containers, editar objetos | https://k9scli.io/ |
| `kubectx` / `kubens` | Versões standalone dos plugins `ctx` e `ns` | https://github.com/ahmetb/kubectx |
| `stern` | Versão standalone do agregador de logs | https://github.com/stern/stern |
| `kubeconform` | Valida manifestos contra o schema da API, sem cluster | https://github.com/yannh/kubeconform |
| `kube-linter` | Detecta más práticas em manifestos e charts | https://docs.kubelinter.io/ |
| `helm` | Empacotamento e release de aplicações | https://helm.sh/ |
| `trivy` | Scanner de vulnerabilidades e misconfigurations | https://trivy.dev/ |

---

## Apêndice B — Modelos de manifesto prontos

Todos os manifestos desta seção são **completos e aplicáveis** contra o namespace `store-dev` e
usam a aplicação-exemplo do semestre, a FIAP Store. Substitua `seuusuario` pelo seu usuário do
Docker Hub. Para criar o namespace:

```bash
kubectl create namespace store-dev
kubectl config set-context --current --namespace=store-dev
```

### B.1 Namespace

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: store-dev
  labels:
    env: dev
    # Ativa o Pod Security Admission no nível "baseline" para admissão
    # e alerta quando o workload não atende ao nível "restricted".
    pod-security.kubernetes.io/enforce: baseline
    pod-security.kubernetes.io/enforce-version: latest
    pod-security.kubernetes.io/warn: restricted
    pod-security.kubernetes.io/warn-version: latest
```

### B.2 Pod

Pod isolado praticamente nunca vai para produção — quem faz isso é um controlador. Use este modelo
para experimentos e para entender o objeto mais básico.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
    env: dev
spec:
  # Encerramento gracioso: tempo que o kubelet espera após o SIGTERM
  terminationGracePeriodSeconds: 30
  securityContext:
    # Contexto de segurança no nível do Pod: vale para todos os containers
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
    - name: store-api
      image: docker.io/seuusuario/store-api:1.0
      imagePullPolicy: IfNotPresent
      ports:
        - name: http
          containerPort: 8000
          protocol: TCP
      env:
        - name: APP_ENV
          value: "dev"
        - name: DB_HOST
          value: "store-db.store-dev.svc.cluster.local"
        # Downward API: o Pod descobre o próprio nome e nó
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: NODE_NAME
          valueFrom:
            fieldRef:
              fieldPath: spec.nodeName
      resources:
        requests:
          cpu: "100m"
          memory: "128Mi"
        limits:
          cpu: "500m"
          memory: "256Mi"
      # Startup: dá tempo para a aplicação subir sem que liveness a mate
      startupProbe:
        httpGet:
          path: /health
          port: http
        failureThreshold: 30
        periodSeconds: 2
      # Liveness: se falhar, o container é reiniciado
      livenessProbe:
        httpGet:
          path: /health
          port: http
        initialDelaySeconds: 5
        periodSeconds: 10
        timeoutSeconds: 2
        failureThreshold: 3
      # Readiness: se falhar, o Pod sai dos Endpoints do Service
      readinessProbe:
        httpGet:
          path: /ready
          port: http
        initialDelaySeconds: 3
        periodSeconds: 5
        failureThreshold: 3
      securityContext:
        allowPrivilegeEscalation: false
        readOnlyRootFilesystem: true
        capabilities:
          drop: ["ALL"]
      volumeMounts:
        - name: tmp
          mountPath: /tmp
  volumes:
    # Necessário porque o rootfs é somente leitura
    - name: tmp
      emptyDir: {}
  restartPolicy: Always
```

### B.3 Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
    tier: backend
    env: dev
spec:
  replicas: 3
  revisionHistoryLimit: 5        # quantas revisões antigas manter para rollback
  progressDeadlineSeconds: 600   # depois disso o rollout é marcado como falho
  minReadySeconds: 10            # Pod precisa ficar Ready por 10s antes de contar
  selector:
    matchLabels:
      app: store-api
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # 1 Pod a mais durante a atualização
      maxUnavailable: 0    # nunca reduz a capacidade: zero downtime
  template:
    metadata:
      labels:
        app: store-api
        tier: backend
        env: dev
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: store-api-sa
      automountServiceAccountToken: false
      terminationGracePeriodSeconds: 30
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        seccompProfile:
          type: RuntimeDefault
      # Espalha as réplicas entre nós diferentes
      topologySpreadConstraints:
        - maxSkew: 1
          topologyKey: kubernetes.io/hostname
          whenUnsatisfiable: ScheduleAnyway
          labelSelector:
            matchLabels:
              app: store-api
      containers:
        - name: store-api
          image: docker.io/seuusuario/store-api:1.0
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 8000
          envFrom:
            - configMapRef:
                name: store-config
            - secretRef:
                name: store-db-secret
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
          livenessProbe:
            httpGet:
              path: /health
              port: http
            initialDelaySeconds: 10
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /ready
              port: http
            initialDelaySeconds: 3
            periodSeconds: 5
          lifecycle:
            preStop:
              exec:
                # Dá tempo ao kube-proxy de remover o Pod dos Endpoints
                command: ["/bin/sh", "-c", "sleep 5"]
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: tmp
              mountPath: /tmp
      volumes:
        - name: tmp
          emptyDir: {}
```

### B.4 Service — ClusterIP (padrão, só acessível dentro do cluster)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api
  namespace: store-dev
  labels:
    app: store-api
spec:
  type: ClusterIP
  selector:
    app: store-api      # precisa casar com as labels do template do Pod
  ports:
    - name: http
      port: 80          # porta do Service
      targetPort: http  # nome da porta no container (ou o número 8000)
      protocol: TCP
```

### B.5 Service — NodePort (exposto na porta de cada nó)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api-np
  namespace: store-dev
spec:
  type: NodePort
  selector:
    app: store-api
  ports:
    - name: http
      port: 80
      targetPort: 8000
      nodePort: 30080   # faixa padrão: 30000-32767; se omitido, é sorteado
      protocol: TCP
```

Acesso: `minikube service store-api-np -n store-dev --url` ou `http://<IP-do-nó>:30080`.

### B.6 Service — LoadBalancer (provisiona um balanceador do provedor de nuvem)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-api-lb
  namespace: store-dev
  annotations:
    # Na AWS, cria um Network Load Balancer voltado para a internet.
    # Exige o AWS Load Balancer Controller instalado no cluster.
    service.beta.kubernetes.io/aws-load-balancer-type: "external"
    service.beta.kubernetes.io/aws-load-balancer-nlb-target-type: "ip"
    service.beta.kubernetes.io/aws-load-balancer-scheme: "internet-facing"
spec:
  type: LoadBalancer
  # Preserva o IP de origem do cliente e evita saltos extras entre nós
  externalTrafficPolicy: Local
  selector:
    app: store-api
  ports:
    - name: http
      port: 80
      targetPort: 8000
      protocol: TCP
```

> **Atenção — custo:** cada Service do tipo LoadBalancer na AWS provisiona um NLB ou CLB real e é
> cobrado por hora mais tráfego. Em minikube, use `minikube tunnel` para simular. Não deixe
> LoadBalancers órfãos.

### B.7 Service headless (para StatefulSet)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: store-db
  namespace: store-dev
  labels:
    app: store-db
spec:
  clusterIP: None       # headless: sem IP virtual, DNS resolve para os IPs dos Pods
  selector:
    app: store-db
  ports:
    - name: postgres
      port: 5432
      targetPort: 5432
```

### B.8 Ingress com TLS

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: store-ingress
  namespace: store-dev
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/proxy-body-size: "8m"
    # Emissão automática de certificado quando o cert-manager está instalado
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  ingressClassName: nginx
  tls:
    - hosts:
        - store.exemplo.com
      # Secret do tipo kubernetes.io/tls contendo tls.crt e tls.key
      secretName: store-tls
  rules:
    - host: store.exemplo.com
      http:
        paths:
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: store-api
                port:
                  number: 80
          - path: /
            pathType: Prefix
            backend:
              service:
                name: store-front
                port:
                  number: 80
```

Certificado autoassinado para laboratório:

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout tls.key -out tls.crt -subj "/CN=store.exemplo.com/O=FIAP"

kubectl create secret tls store-tls \
  --cert=tls.crt --key=tls.key -n store-dev
```

### B.9 ConfigMap

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: store-config
  namespace: store-dev
data:
  # Pares chave-valor simples: viram variáveis de ambiente com envFrom
  APP_ENV: "dev"
  DB_HOST: "store-db.store-dev.svc.cluster.local"
  DB_PORT: "5432"
  LOG_LEVEL: "debug"
  # Arquivo inteiro: montado como arquivo dentro do container
  app.properties: |
    timeout=30
    retries=3
    feature.canary=false
  nginx.conf: |
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html;
      }
      location /healthz {
        return 200 'ok';
        add_header Content-Type text/plain;
      }
    }
```

Consumo como volume (arquivos) e como variáveis:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-front
  namespace: store-dev
  labels:
    app: store-front
spec:
  containers:
    - name: nginx
      image: nginx:1.27-alpine
      ports:
        - containerPort: 80
      envFrom:
        - configMapRef:
            name: store-config
      volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d/default.conf
          subPath: nginx.conf     # monta só uma chave, como arquivo
          readOnly: true
  volumes:
    - name: config
      configMap:
        name: store-config
        items:
          - key: nginx.conf
            path: nginx.conf
```

### B.10 Secret

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: store-db-secret
  namespace: store-dev
type: Opaque
stringData:
  # stringData aceita texto puro; o servidor converte para base64 em data
  DB_PASSWORD: "troque-esta-senha"
  DB_USER: "storeuser"
  API_TOKEN: "ey.exemplo.token"
```

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: store-tls
  namespace: store-dev
type: kubernetes.io/tls
data:
  # Ambos obrigatórios e em base64
  tls.crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCg==
  tls.key: LS0tLS1CRUdJTiBQUklWQVRFIEtFWS0tLS0tCg==
```

> **Atenção:** Secret **não é criptografia** — é apenas codificação base64. Sem
> `EncryptionConfiguration` no API Server, o conteúdo está em texto claro no etcd. Nunca versione um
> Secret em Git puro; use Sealed Secrets, External Secrets Operator, SOPS ou AWS Secrets Manager
> (capítulos 8, 10 e 11).

Montagem de um Secret como arquivos, com permissões restritas:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
  namespace: store-dev
spec:
  containers:
    - name: app
      image: busybox:1.36
      command: ["sh", "-c", "ls -l /etc/creds && sleep 3600"]
      volumeMounts:
        - name: creds
          mountPath: /etc/creds
          readOnly: true
  volumes:
    - name: creds
      secret:
        secretName: store-db-secret
        defaultMode: 0400   # somente leitura pelo dono
```

### B.11 PersistentVolumeClaim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: store-db-pvc
  namespace: store-dev
spec:
  accessModes:
    - ReadWriteOnce        # um único nó monta para leitura e escrita
  resources:
    requests:
      storage: 5Gi
  # Em minikube, "standard"; na AWS, "gp3" (EBS CSI). Omita para usar a
  # StorageClass padrão do cluster.
  storageClassName: standard
```

Uso do PVC em um Pod:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: store-db
  namespace: store-dev
  labels:
    app: store-db
spec:
  containers:
    - name: postgres
      image: postgres:16-alpine
      ports:
        - containerPort: 5432
      env:
        - name: POSTGRES_DB
          value: store
        - name: POSTGRES_USER
          valueFrom:
            secretKeyRef:
              name: store-db-secret
              key: DB_USER
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: store-db-secret
              key: DB_PASSWORD
        - name: PGDATA
          value: /var/lib/postgresql/data/pgdata
      volumeMounts:
        - name: dados
          mountPath: /var/lib/postgresql/data
  volumes:
    - name: dados
      persistentVolumeClaim:
        claimName: store-db-pvc
```

### B.12 StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: store-db
  namespace: store-dev
spec:
  serviceName: store-db          # obrigatório: aponta para o Service headless (B.7)
  replicas: 3
  podManagementPolicy: OrderedReady   # cria/remove Pods em ordem (0, 1, 2)
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      partition: 0               # atualiza a partir do ordinal indicado
  selector:
    matchLabels:
      app: store-db
  template:
    metadata:
      labels:
        app: store-db
    spec:
      terminationGracePeriodSeconds: 30
      securityContext:
        fsGroup: 999             # grupo do usuário postgres na imagem oficial
      containers:
        - name: postgres
          image: postgres:16-alpine
          ports:
            - name: postgres
              containerPort: 5432
          env:
            - name: POSTGRES_DB
              value: store
            - name: POSTGRES_USER
              valueFrom:
                secretKeyRef:
                  name: store-db-secret
                  key: DB_USER
            - name: POSTGRES_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: store-db-secret
                  key: DB_PASSWORD
            - name: PGDATA
              value: /var/lib/postgresql/data/pgdata
          resources:
            requests:
              cpu: "250m"
              memory: "256Mi"
            limits:
              cpu: "1"
              memory: "1Gi"
          readinessProbe:
            exec:
              command: ["sh", "-c", "pg_isready -U $POSTGRES_USER -d store"]
            initialDelaySeconds: 10
            periodSeconds: 5
          livenessProbe:
            tcpSocket:
              port: postgres
            initialDelaySeconds: 30
            periodSeconds: 10
          volumeMounts:
            - name: dados
              mountPath: /var/lib/postgresql/data
  # Cada réplica ganha seu próprio PVC: dados-store-db-0, dados-store-db-1, ...
  volumeClaimTemplates:
    - metadata:
        name: dados
      spec:
        accessModes: ["ReadWriteOnce"]
        storageClassName: standard
        resources:
          requests:
            storage: 5Gi
```

> **Nota:** os PVCs criados por `volumeClaimTemplates` **não são apagados** quando o StatefulSet é
> removido. Isso é proposital (protege os dados) e é fonte frequente de custo esquecido na AWS.
> Apague-os explicitamente: `kubectl delete pvc -l app=store-db`.

### B.13 DaemonSet

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-log-agent
  namespace: kube-system
  labels:
    app: node-log-agent
spec:
  selector:
    matchLabels:
      app: node-log-agent
  updateStrategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
  template:
    metadata:
      labels:
        app: node-log-agent
    spec:
      # Roda também nos nós do control plane, que normalmente têm taint
      tolerations:
        - key: node-role.kubernetes.io/control-plane
          operator: Exists
          effect: NoSchedule
        - key: node.kubernetes.io/not-ready
          operator: Exists
          effect: NoExecute
      hostNetwork: false
      priorityClassName: system-node-critical
      containers:
        - name: agent
          image: busybox:1.36
          command:
            - sh
            - -c
            - "while true; do tail -n 1 /var/log/containers/*.log 2>/dev/null | head -5; sleep 30; done"
          resources:
            requests:
              cpu: "50m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
          securityContext:
            readOnlyRootFilesystem: true
            allowPrivilegeEscalation: false
            capabilities:
              drop: ["ALL"]
          volumeMounts:
            - name: varlog
              mountPath: /var/log
              readOnly: true
      volumes:
        - name: varlog
          hostPath:
            path: /var/log
            type: Directory
```

### B.14 Job

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: carga-inicial
  namespace: store-dev
spec:
  completions: 5          # total de execuções bem-sucedidas necessárias
  parallelism: 2          # quantas rodam ao mesmo tempo
  backoffLimit: 4         # tentativas antes de marcar o Job como Failed
  activeDeadlineSeconds: 600   # tempo máximo total de execução
  ttlSecondsAfterFinished: 3600  # remove o Job 1h após terminar
  completionMode: Indexed # cada Pod recebe JOB_COMPLETION_INDEX
  template:
    metadata:
      labels:
        app: store-worker
        job: carga-inicial
    spec:
      restartPolicy: OnFailure   # obrigatório: Never ou OnFailure
      containers:
        - name: worker
          image: docker.io/seuusuario/store-worker:1.0
          command: ["python", "-c"]
          args:
            - |
              import os, time
              idx = os.getenv("JOB_COMPLETION_INDEX", "0")
              print(f"processando lote {idx}", flush=True)
              time.sleep(5)
              print("lote concluido", flush=True)
          envFrom:
            - configMapRef:
                name: store-config
            - secretRef:
                name: store-db-secret
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

### B.15 CronJob

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: relatorio-diario
  namespace: store-dev
spec:
  schedule: "0 3 * * *"           # todo dia às 03:00
  timeZone: "America/Sao_Paulo"   # disponível desde o Kubernetes 1.27
  concurrencyPolicy: Forbid       # não deixa duas execuções se sobreporem
  startingDeadlineSeconds: 300    # se atrasar mais que isso, pula a execução
  suspend: false
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 2
      activeDeadlineSeconds: 1800
      ttlSecondsAfterFinished: 86400
      template:
        metadata:
          labels:
            app: store-worker
            job: relatorio-diario
        spec:
          restartPolicy: OnFailure
          containers:
            - name: relatorio
              image: docker.io/seuusuario/store-worker:1.0
              command: ["python", "-c"]
              args:
                - |
                  from datetime import datetime
                  print(f"relatorio gerado em {datetime.now().isoformat()}", flush=True)
              envFrom:
                - configMapRef:
                    name: store-config
              resources:
                requests:
                  cpu: "100m"
                  memory: "128Mi"
                limits:
                  cpu: "500m"
                  memory: "256Mi"
```

Sintaxe do campo `schedule`:

```
 ┌─────── minuto (0 - 59)
 │ ┌───── hora (0 - 23)
 │ │ ┌─── dia do mês (1 - 31)
 │ │ │ ┌─ mês (1 - 12)
 │ │ │ │ ┌ dia da semana (0 - 6, domingo = 0)
 │ │ │ │ │
 * * * * *
```

| Expressão | Significado |
|---|---|
| `*/5 * * * *` | A cada 5 minutos |
| `0 * * * *` | A cada hora, no minuto 0 |
| `0 3 * * *` | Diariamente às 03:00 |
| `0 3 * * 1` | Toda segunda-feira às 03:00 |
| `0 0 1 * *` | Todo dia 1º do mês, à meia-noite |
| `@hourly`, `@daily`, `@weekly`, `@monthly` | Atalhos aceitos |

### B.16 HorizontalPodAutoscaler

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: store-api-hpa
  namespace: store-dev
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: store-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70    # percentual sobre o *request* de CPU
    - type: Resource
      resource:
        name: memory
        target:
          type: AverageValue
          averageValue: 200Mi
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
        - type: Percent
          value: 100          # pode dobrar o número de Pods
          periodSeconds: 30
        - type: Pods
          value: 4            # ou somar 4 Pods
          periodSeconds: 30
      selectPolicy: Max
    scaleDown:
      stabilizationWindowSeconds: 300   # espera 5 min de calmaria antes de reduzir
      policies:
        - type: Percent
          value: 50
          periodSeconds: 60
```

> **Nota:** o HPA depende do **metrics-server**. Em minikube: `minikube addons enable metrics-server`.
> Sem requests de CPU definidos no container, o HPA por `Utilization` não funciona — a porcentagem é
> calculada em relação ao request.

### B.17 NetworkPolicy default-deny

Primeiro, negue tudo no namespace:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
  namespace: store-dev
spec:
  # podSelector vazio = todos os Pods do namespace
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
```

Depois, libere apenas o necessário:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-egress
  namespace: store-dev
spec:
  podSelector: {}
  policyTypes:
    - Egress
  egress:
    # DNS é pré-requisito de quase tudo: libere sempre
    - to:
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: kube-system
          podSelector:
            matchLabels:
              k8s-app: kube-dns
      ports:
        - protocol: UDP
          port: 53
        - protocol: TCP
          port: 53
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-front-to-api
  namespace: store-dev
spec:
  podSelector:
    matchLabels:
      app: store-api
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: store-front
        # Também aceita tráfego do ingress-nginx, em outro namespace
        - namespaceSelector:
            matchLabels:
              kubernetes.io/metadata.name: ingress-nginx
      ports:
        - protocol: TCP
          port: 8000
```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-api-to-db
  namespace: store-dev
spec:
  podSelector:
    matchLabels:
      app: store-api
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: store-db
      ports:
        - protocol: TCP
          port: 5432
```

> **Atenção:** NetworkPolicy só tem efeito se o **CNI plugin suportar** (Calico, Cilium, Antrea,
> Weave). Em minikube com o driver padrão, ative com `minikube start --cni=calico`. Com o CNI errado,
> o objeto é aceito pela API e simplesmente ignorado — o pior tipo de falha silenciosa.

### B.18 ResourceQuota

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: store-dev-quota
  namespace: store-dev
spec:
  hard:
    # Computação
    requests.cpu: "4"
    requests.memory: "8Gi"
    limits.cpu: "8"
    limits.memory: "16Gi"
    # Armazenamento
    requests.storage: "50Gi"
    persistentvolumeclaims: "10"
    # Contagem de objetos
    pods: "30"
    services: "10"
    services.loadbalancers: "1"
    services.nodeports: "3"
    configmaps: "20"
    secrets: "20"
    count/deployments.apps: "10"
    count/jobs.batch: "20"
    count/cronjobs.batch: "5"
```

### B.19 LimitRange

```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: store-dev-limits
  namespace: store-dev
spec:
  limits:
    - type: Container
      default:              # limits aplicados quando o container não declara
        cpu: "500m"
        memory: "512Mi"
      defaultRequest:       # requests aplicados quando o container não declara
        cpu: "100m"
        memory: "128Mi"
      min:
        cpu: "50m"
        memory: "64Mi"
      max:
        cpu: "2"
        memory: "2Gi"
      maxLimitRequestRatio: # limite não pode ser mais que 4x o request
        cpu: "4"
        memory: "4"
    - type: Pod
      max:
        cpu: "4"
        memory: "4Gi"
    - type: PersistentVolumeClaim
      min:
        storage: "1Gi"
      max:
        storage: "20Gi"
```

> **Na prática:** ResourceQuota e LimitRange são complementares. A quota limita o **total** do
> namespace; o LimitRange garante que **cada** objeto tenha valores razoáveis e preenche os defaults.
> Sem LimitRange, um Pod sem requests é aceito e, com uma ResourceQuota de `requests.cpu` ativa, a
> criação é **rejeitada** — erro clássico do capítulo 5.

### B.20 ServiceAccount + Role + RoleBinding

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: store-api-sa
  namespace: store-dev
automountServiceAccountToken: false
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: store-api-reader
  namespace: store-dev
rules:
  # Leitura de configuração da própria aplicação
  - apiGroups: [""]
    resources: ["configmaps"]
    verbs: ["get", "list", "watch"]
  # Leitura de um Secret específico, e só dele
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["store-db-secret"]
    verbs: ["get"]
  # Descoberta de instâncias irmãs
  - apiGroups: [""]
    resources: ["pods", "endpoints", "services"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: store-api-reader-binding
  namespace: store-dev
subjects:
  - kind: ServiceAccount
    name: store-api-sa
    namespace: store-dev
roleRef:
  kind: Role
  name: store-api-reader
  apiGroup: rbac.authorization.k8s.io
```

Versão de escopo de cluster (ClusterRole + ClusterRoleBinding), para leitura de nós:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["nodes", "nodes/status"]
    verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-reader-binding
subjects:
  - kind: ServiceAccount
    name: store-api-sa
    namespace: store-dev
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

Verificação:

```bash
kubectl auth can-i get configmaps \
  --as=system:serviceaccount:store-dev:store-api-sa -n store-dev
# yes

kubectl auth can-i delete pods \
  --as=system:serviceaccount:store-dev:store-api-sa -n store-dev
# no
```

### B.21 PodDisruptionBudget

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-api-pdb
  namespace: store-dev
spec:
  # Use minAvailable OU maxUnavailable, nunca os dois
  minAvailable: 2          # também aceita percentual: "50%"
  selector:
    matchLabels:
      app: store-api
```

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: store-db-pdb
  namespace: store-dev
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: store-db
```

> **Nota:** o PDB só protege contra **disrupções voluntárias** (`kubectl drain`, upgrade de node
> group, Cluster Autoscaler). Falha de hardware, OOM e `kubectl delete pod` ignoram o PDB. Um PDB com
> `minAvailable` igual ao número de réplicas **trava** o `drain` indefinidamente — erro comum em
> manutenção de cluster.

### B.22 Argo CD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: store-dev
  namespace: argocd
  finalizers:
    # Garante que os recursos filhos sejam removidos junto com a Application
    - resources-finalizer.argocd.argoproj.io
spec:
  project: fiap-store
  source:
    repoURL: https://github.com/seuusuario/fiap-store-gitops.git
    targetRevision: main
    path: overlays/dev
    # Se o path for um chart Helm, use o bloco abaixo em vez de kustomize
    # helm:
    #   releaseName: store
    #   valueFiles:
    #     - values-dev.yaml
    kustomize:
      namePrefix: ""
      images:
        - docker.io/seuusuario/store-api:1.0
  destination:
    server: https://kubernetes.default.svc
    namespace: store-dev
  syncPolicy:
    automated:
      prune: true        # remove do cluster o que sumiu do Git
      selfHeal: true     # desfaz alterações manuais feitas no cluster
      allowEmpty: false
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
      - ServerSideApply=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  # Campos que o Argo CD deve ignorar ao calcular drift (ex.: escala pelo HPA)
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas
  revisionHistoryLimit: 10
```

AppProject correspondente, restringindo o que essa Application pode fazer:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: fiap-store
  namespace: argocd
spec:
  description: Aplicacoes da FIAP Store
  sourceRepos:
    - https://github.com/seuusuario/fiap-store-gitops.git
  destinations:
    - server: https://kubernetes.default.svc
      namespace: store-dev
    - server: https://kubernetes.default.svc
      namespace: store-prod
  clusterResourceWhitelist:
    - group: ""
      kind: Namespace
  # so entram aqui recursos NAMESPACED; ClusterRole/ClusterRoleBinding
  # sao de escopo de cluster e ja estao barrados pelo whitelist acima
  namespaceResourceBlacklist:
    - group: ""
      kind: ResourceQuota
    - group: rbac.authorization.k8s.io
      kind: RoleBinding
```

---

## Apêndice C — Guia de troubleshooting

### C.1 Método: sempre na mesma ordem

Depurar Kubernetes não é adivinhar: é percorrer a cadeia de responsabilidade de fora para dentro.
Faça sempre nesta ordem, e resista à tentação de pular etapas.

```
0. O kubectl fala com o cluster? kubectl cluster-info   |   kubectl get nodes
1. O objeto existe?              kubectl get <tipo> <nome> -n <ns>
2. O que o controlador diz?      kubectl describe <tipo> <nome> -n <ns>   (leia os Events!)
3. O que o cluster registrou?    kubectl get events -n <ns> --sort-by=.lastTimestamp
4. O que a aplicação diz?        kubectl logs <pod> [-c <container>] [-p]
5. O que se vê de dentro?        kubectl exec -it <pod> -- sh   |   kubectl debug ...
6. O que o nó diz?               kubectl describe node <no>     |   kubectl top nodes
```

> **Na prática:** 80% dos problemas de laboratório são resolvidos na etapa 2. A seção `Events` no
> fim do `kubectl describe pod` costuma conter a resposta literal, em inglês, com a causa.

#### C.1.1 Etapa 0 em detalhe: "The connection to the server ... was refused"

Este é, de longe, o erro mais frequente da primeira semana — e o único da lista que **não é um
problema de Kubernetes**, e sim de kubeconfig. Nenhum outro diagnóstico faz sentido enquanto ele
existir, porque o `kubectl` sequer conseguiu falar com o apiserver.

```
E0806 17:30:54.370165  32182 memcache.go:265] "Unhandled Error" err="couldn't get current server
API group list: Get \"https://127.0.0.1:62361/api?timeout=32s\": dial tcp 127.0.0.1:62361:
connect: connection refused"
The connection to the server 127.0.0.1:62361 was refused - did you specify the right host or port?
```

**Leia o endereço — ele identifica a causa:**

| Endereço no erro | Significado | Correção |
|---|---|---|
| `localhost:8080` | **Não há kubeconfig algum.** O kubectl caiu no default histórico | `minikube start`; ou `export KUBECONFIG=/caminho/config`; ou `aws eks update-kubeconfig --name <cluster>` |
| `127.0.0.1:<porta alta aleatória>` (ex.: `62361`) | Kubeconfig **órfão**: aponta para um cluster local que já existiu — minikube/kind apagado, ou Docker Desktop com Kubernetes desligado | `minikube start`; ligar o Kubernetes no Docker Desktop; ou trocar de contexto |
| `192.168.49.2:8443` | Endereço típico do minikube com driver docker: o cluster existe mas está **parado** | `minikube start` (ou `minikube status` para confirmar) |
| `https://<hash>.eks.amazonaws.com` com timeout | Cluster EKS apagado, credenciais AWS expiradas ou security group bloqueando | `aws sts get-caller-identity`; `aws eks describe-cluster --name <nome>`; renovar credenciais |
| `i/o timeout` em vez de `connection refused` | Há algo escutando, mas a rede não chega: VPN, firewall, endpoint privado do EKS | Conferir VPN/proxy; no EKS, checar `endpointPublicAccess` |

**Roteiro de diagnóstico:**

```bash
kubectl config current-context           # qual contexto está ativo?
kubectl config get-contexts              # quais existem? o CURRENT (*) é o que você espera?
kubectl config view --minify             # para onde ele aponta (server:) e com que credencial
kubectl version --client                 # funciona offline: prova que o binário está OK
minikube status                          # o cluster local está no ar?
docker ps | grep -E 'minikube|k3d|kind'  # o container do cluster existe?
```

**Correções por cenário:**

```bash
# cluster local parado ou inexistente
minikube start --driver=docker

# contexto errado ativo
kubectl config use-context minikube          # ou: kubectx minikube

# contexto órfão que você quer remover de vez
kubectl config delete-context <nome>
kubectl config delete-cluster <nome>
kubectl config unset users.<nome>

# EKS: regravar a entrada do cluster no kubeconfig
aws eks update-kubeconfig --region us-east-1 --name fiap-store

# kubeconfig em local não padrão
export KUBECONFIG=$HOME/.kube/config-fiap
```

> **Atenção:** `--dry-run=client` **não** torna o comando offline. `kubectl run`, `kubectl create`,
> `kubectl api-resources` e `kubectl explain` fazem *discovery* no servidor antes de qualquer coisa,
> e por isso falham exatamente com este erro quando não há cluster. O único comando realmente local
> é `kubectl version --client`. Sem cluster, escreva o manifesto à mão — é o que a Trilha A do
> laboratório do Capítulo 1 propõe.

> **Windows/WSL2:** kubeconfigs distintos no Windows e dentro do WSL são uma armadilha comum. O
> `~/.kube/config` do WSL não é o mesmo do `C:\Users\<voce>\.kube\config`. Padronize o trabalho
> dentro do WSL durante toda a disciplina.

### C.2 Árvore de decisão

```
O Pod aparece em "kubectl get pods"?
├── NÃO
│   ├── O controlador existe?  kubectl get deploy/rs/sts/job -n <ns>
│   │   ├── NÃO -> o manifesto não foi aplicado, ou foi no namespace errado.
│   │   │          Confira: kubectl config view --minify | grep namespace
│   │   └── SIM -> kubectl describe rs <nome>
│   │              Causas: ResourceQuota estourada, LimitRange rejeitando,
│   │              admission webhook negando, selector inválido, RBAC do controller.
│   └── Erro na criação? Leia a mensagem do kubectl apply: campo inválido,
│              apiVersion removida, selector imutável.
│
└── SIM -> qual é o STATUS?
    │
    ├── Pending
    │   └── kubectl describe pod <nome> -> seção Events
    │       ├── "0/N nodes are available: insufficient cpu/memory"
    │       │     -> requests altos demais, ou cluster cheio. Reduza requests,
    │       │        adicione nós, verifique kubectl describe node.
    │       ├── "node(s) had untolerated taint"
    │       │     -> falta toleration, ou o nó está cordon/NotReady.
    │       ├── "node(s) didn't match Pod's node affinity/selector"
    │       │     -> nodeSelector/affinity aponta para label inexistente.
    │       ├── "pod has unbound immediate PersistentVolumeClaims"
    │       │     -> vá para o ramo "PVC Pending".
    │       └── "0/N nodes are available: N Insufficient pods"
    │             -> limite de Pods por nó atingido (relevante no EKS, por ENI).
    │
    ├── ContainerCreating (parado nesse estado)
    │   └── Causas: volume não monta (CSI), Secret/ConfigMap ausente,
    │       imagem enorme sendo baixada, CNI não atribuiu IP.
    │       -> kubectl describe pod + kubectl get events + logs do kubelet
    │
    ├── ImagePullBackOff / ErrImagePull
    │   └── Nome, tag ou registry errados; imagem privada sem imagePullSecrets;
    │       arquitetura incompatível (arm64 vs amd64); rate limit do Docker Hub.
    │
    ├── CreateContainerConfigError
    │   └── ConfigMap ou Secret referenciado não existe, ou a chave não existe.
    │
    ├── CrashLoopBackOff
    │   └── kubectl logs <pod> --previous
    │       ├── Stack trace da aplicação  -> bug ou config faltando
    │       ├── "permission denied"       -> runAsNonRoot vs. arquivo do root,
    │       │                                readOnlyRootFilesystem
    │       ├── Nada nos logs, exit 137   -> OOMKilled, vá para esse ramo
    │       └── Container termina em 0    -> falta processo de longa duração
    │                                        (restartPolicy: Always com comando curto)
    │
    ├── Init:Error / Init:CrashLoopBackOff
    │   └── kubectl logs <pod> -c <init-container>
    │       Init containers rodam em sequência e precisam sair com código 0.
    │
    ├── OOMKilled
    │   └── limits.memory baixo demais, vazamento de memória, ou heap da JVM/Python
    │       maior que o limit. kubectl describe pod -> Last State: Terminated (137)
    │
    ├── Evicted
    │   └── Pressão de recurso no nó (disco, memória, PID). kubectl describe node
    │       -> Conditions: DiskPressure/MemoryPressure. Ajuste requests e QoS.
    │
    ├── Terminating (preso há minutos)
    │   └── Finalizer pendente, volume que não desmonta, nó morto.
    │
    └── Running mas a aplicação não responde
        ├── READY é 0/1?     -> readinessProbe falhando. kubectl describe pod.
        ├── Service sem Endpoints? -> selector não casa com as labels do Pod,
        │                             ou nenhum Pod está Ready.
        ├── DNS não resolve? -> CoreDNS, NetworkPolicy bloqueando porta 53,
        │                       nome FQDN incorreto.
        ├── Ingress 404?     -> host/path/ingressClassName errados, ou o
        │                       controller não reconheceu a regra.
        └── Ingress 502/503? -> backend sem Endpoints, porta errada, TLS quebrado.
```

### C.3 Tabela de referência: sintoma, causa, diagnóstico e correção

| Sintoma / Status | Causas prováveis | Comandos de diagnóstico | Correção |
|---|---|---|---|
| **`The connection to the server ... was refused`** (nenhum comando funciona) | Kubeconfig ausente (`localhost:8080`) ou órfão (`127.0.0.1:<porta alta>`); cluster local parado ou apagado; Docker Desktop com Kubernetes desligado. Afeta também `--dry-run=client`, `api-resources` e `explain`, que fazem *discovery* no servidor | `kubectl config current-context`; `kubectl config get-contexts`; `kubectl config view --minify`; `minikube status`; `kubectl version --client` (funciona offline, prova que o binário está OK) | `minikube start --driver=docker`; `kubectl config use-context <válido>`; `aws eks update-kubeconfig --name <cluster>`; ver **C.1.1** para a tabela completa por endereço |
| **ImagePullBackOff** | Nome da imagem ou tag inexistente; registry privado sem credencial; typo em `image:`; rate limit do Docker Hub anônimo | `kubectl describe pod <p>` (ver Events: `Failed to pull image`); `kubectl get pod <p> -o jsonpath='{.spec.containers[*].image}'` | Corrija o nome/tag; crie `kubectl create secret docker-registry regcred ...` e adicione `imagePullSecrets`; autentique com `docker login`; no minikube use `eval $(minikube docker-env)` ou `minikube image load` |
| **ErrImagePull** | Mesma família do anterior, porém no primeiro erro (antes do backoff); rede do nó sem saída; arquitetura incompatível (imagem amd64 em nó arm64) | `kubectl describe pod <p>`; `kubectl get nodes -o jsonpath='{.items[*].status.nodeInfo.architecture}'` | Publique a imagem multiarquitetura (`docker buildx build --platform linux/amd64,linux/arm64`); verifique NAT/proxy do nó |
| **ErrImageNeverPull** | `imagePullPolicy: Never` e a imagem não existe no nó | `kubectl describe pod <p>`; `minikube image ls \| grep store-api` | `minikube image load store-api:1.0`, ou mude a política para `IfNotPresent` |
| **CrashLoopBackOff** | Aplicação sai com erro; variável de ambiente faltando; dependência (banco) indisponível; comando errado no `command`; processo não é de longa duração | `kubectl logs <p> --previous`; `kubectl describe pod <p>` (Last State, Exit Code); `kubectl get pod <p> -o jsonpath='{.status.containerStatuses[0].lastState.terminated.reason}'` | Corrija a causa apontada no log; adicione initContainer que aguarda a dependência; ajuste `command`/`args`; suba um Pod de debug com a mesma imagem e `command: ["sleep","3600"]` para inspecionar |
| **Pending — insufficient cpu/memory** | Soma dos `requests` maior que a capacidade alocável; cluster cheio | `kubectl describe pod <p>` (Events do scheduler); `kubectl describe node \| grep -A 8 "Allocated resources"`; `kubectl top nodes` | Reduza `requests`; escale o cluster (`minikube node add`, node group na AWS); apague workloads ociosos |
| **Pending — untolerated taint** | Nó com taint (control plane, `NoSchedule` customizado) e Pod sem toleration; nó `cordon` | `kubectl describe node <n> \| grep -i taint`; `kubectl get nodes` (ver `SchedulingDisabled`) | Adicione `tolerations` ao Pod; `kubectl uncordon <n>`; remova o taint com `kubectl taint node <n> chave-` |
| **Pending — node affinity/selector** | `nodeSelector` ou `nodeAffinity` referenciando label que nenhum nó possui | `kubectl get nodes --show-labels`; `kubectl get pod <p> -o yaml \| grep -A 10 affinity` | Aplique a label no nó (`kubectl label node <n> disktype=ssd`) ou corrija o seletor |
| **Pending — unbound PVC** | Ver linha de PVC Pending | `kubectl get pvc -n <ns>`; `kubectl describe pvc <pvc>` | Ver correção da linha de PVC |
| **ContainerCreating (preso)** | Volume que não monta; Secret/ConfigMap ausente; CNI sem IP disponível; imagem muito grande | `kubectl describe pod <p>`; `kubectl get events -n <ns> --sort-by=.lastTimestamp`; logs do kubelet no nó | Crie o ConfigMap/Secret; verifique o CSI driver; verifique o CNI (`kubectl get pods -n kube-system`) |
| **CreateContainerConfigError** | `configMapRef`, `secretRef`, `configMapKeyRef` ou `secretKeyRef` apontando para objeto ou chave inexistente | `kubectl describe pod <p>` (Events: `configmap "x" not found`); `kubectl get cm,secret -n <ns>` | Crie o objeto no **mesmo namespace**; confira o nome da chave (é case-sensitive); use `optional: true` quando a variável for opcional |
| **CreateContainerError** | `command` aponta para binário inexistente na imagem; permissão negada; workdir inválido | `kubectl describe pod <p>`; `docker run --rm -it <imagem> sh` localmente | Corrija `command`/`args`; verifique se a imagem tem shell (distroless não tem) |
| **RunContainerError** | Montagem de volume conflitando com arquivo existente; `securityContext` incompatível | `kubectl describe pod <p>` | Ajuste `mountPath`/`subPath`; revise `runAsUser` e `fsGroup` |
| **Init:Error / Init:CrashLoopBackOff** | InitContainer saiu com código diferente de 0 | `kubectl logs <p> -c <init>`; `kubectl describe pod <p>` (seção Init Containers) | Corrija o script do init; garanta que a dependência aguardada existe; use timeout no loop de espera |
| **Init:0/2 (parado)** | InitContainer aguardando indefinidamente uma dependência que nunca fica pronta | `kubectl logs <p> -c <init> -f` | Verifique o Service/DNS que o init está aguardando |
| **OOMKilled (exit 137)** | `limits.memory` menor que o pico real; vazamento; heap da runtime maior que o limit | `kubectl describe pod <p>` → `Last State: Terminated, Reason: OOMKilled`; `kubectl top pod <p> --containers` | Aumente `limits.memory`; ajuste o heap da runtime (`--max-old-space-size`, `-Xmx`, `GOMEMLIMIT`); investigue o vazamento |
| **Evicted** | Nó sob `MemoryPressure`, `DiskPressure` ou `PIDPressure`; Pod BestEffort é o primeiro a cair | `kubectl get pods -A --field-selector status.phase=Failed`; `kubectl describe node <n> \| grep -A 6 Conditions`; `kubectl describe pod <p>` | Defina `requests` e `limits` (QoS Guaranteed/Burstable); limpe imagens do nó; aumente o disco; remova os Pods evicted com `kubectl delete pods --field-selector status.phase=Failed -A` |
| **Terminating (preso > 5 min)** | Finalizer não removido; volume que não desmonta; nó morto; `preStop` longo demais | `kubectl get pod <p> -o jsonpath='{.metadata.finalizers}'`; `kubectl describe pod <p>` | Corrija a causa do finalizer; em último caso `kubectl delete pod <p> --grace-period=0 --force` e, se necessário, `kubectl patch pod <p> -p '{"metadata":{"finalizers":null}}' --type=merge` |
| **Namespace preso em Terminating** | Recursos com finalizers pendentes; APIService indisponível | `kubectl get ns <ns> -o json \| jq .status`; `kubectl api-resources --verbs=list --namespaced -o name \| xargs -n1 kubectl get -n <ns> --show-kind --ignore-not-found` | Remova os finalizers dos objetos restantes; verifique `kubectl get apiservices \| grep False` |
| **Pod Running, READY 0/1** | `readinessProbe` falhando: path errado, porta errada, app ainda subindo, `initialDelaySeconds` curto | `kubectl describe pod <p>` (Events: `Readiness probe failed`); `kubectl exec <p> -- wget -qO- localhost:8000/ready` | Corrija path/porta da probe; aumente `initialDelaySeconds` ou use `startupProbe`; corrija o endpoint na aplicação |
| **Pod reinicia sozinho, sem crash** | `livenessProbe` agressiva demais matando app saudável (GC pause, startup lento) | `kubectl describe pod <p>` (Events: `Liveness probe failed`); `kubectl get pod <p> -o jsonpath='{.status.containerStatuses[0].restartCount}'` | Aumente `failureThreshold`/`periodSeconds`; adicione `startupProbe`; nunca aponte liveness para endpoint que consulta o banco |
| **Service sem Endpoints** | `selector` do Service não casa com as labels do Pod; nenhum Pod está Ready; `targetPort` errado | `kubectl get endpoints <svc>` (mostra `<none>`); `kubectl describe svc <svc>`; `kubectl get pods --show-labels` | Alinhe `spec.selector` do Service com `metadata.labels` do template do Pod; corrija a readinessProbe; confira `targetPort` |
| **Service com Endpoints mas sem resposta** | `targetPort` diferente da porta que a aplicação escuta; app ouvindo em `127.0.0.1` em vez de `0.0.0.0`; NetworkPolicy | `kubectl exec <p> -- ss -tlnp`; `kubectl run curl --rm -it --image=curlimages/curl --restart=Never -- curl -v http://<svc>.<ns>` | Corrija `targetPort`; faça a app bindar em `0.0.0.0`; revise NetworkPolicies |
| **DNS não resolve** | CoreDNS caído; NetworkPolicy bloqueando porta 53; FQDN incorreto; `dnsPolicy` errado | `kubectl get pods -n kube-system -l k8s-app=kube-dns`; `kubectl logs -n kube-system -l k8s-app=kube-dns`; `kubectl run dns --rm -it --image=busybox:1.36 --restart=Never -- nslookup kubernetes.default` | Reinicie o CoreDNS (`kubectl rollout restart deploy/coredns -n kube-system`); libere UDP/TCP 53 na NetworkPolicy; use `<svc>.<ns>.svc.cluster.local` |
| **DNS resolve, mas conexão recusada** | NetworkPolicy default-deny sem regra de egress; app não escutando; porta errada | `kubectl exec -it <p> -- nc -zv store-api 80`; `kubectl get netpol -n <ns>` | Adicione regra de egress; confira a porta real com `ss -tlnp` |
| **Ingress 404 (Not Found)** | `host` não bate com o `Host:` da requisição; `path`/`pathType` incorretos; `ingressClassName` ausente ou errado; Ingress em namespace diferente do Service | `kubectl describe ing <ing>`; `kubectl get ingressclass`; `curl -H "Host: store.exemplo.com" http://<ip-ingress>/api` | Corrija host/path; defina `ingressClassName: nginx`; lembre que Ingress e Service precisam estar no mesmo namespace |
| **Ingress 502 / 503** | Backend sem Endpoints; porta do Service errada; app respondendo HTTPS enquanto o controller fala HTTP | `kubectl get endpoints <svc>`; `kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller --tail=100` | Garanta Pods Ready; corrija a porta; ajuste a annotation `nginx.ingress.kubernetes.io/backend-protocol` |
| **Ingress com certificado inválido** | Secret TLS ausente, no namespace errado, ou com CN diferente do host | `kubectl get secret <tls> -n <ns>`; `kubectl describe ing <ing>`; `openssl s_client -connect host:443 -servername host` | Recrie o Secret TLS no namespace do Ingress; confira o CN/SAN; verifique o cert-manager (`kubectl get certificate,certificaterequest`) |
| **PVC Pending** | Nenhuma StorageClass padrão; StorageClass inexistente; sem PV compatível (modo de acesso, capacidade); provisioner com `WaitForFirstConsumer` e nenhum Pod agendado; zona incompatível na AWS | `kubectl describe pvc <pvc>`; `kubectl get storageclass`; `kubectl get pv` | Defina a StorageClass correta ou marque uma como padrão (`storageclass.kubernetes.io/is-default-class: "true"`); ajuste `accessModes`; crie o Pod que consome o PVC; garanta que o nó está na mesma AZ do EBS |
| **PVC Bound mas Pod não sobe** | `accessMode` ReadWriteOnce com dois Pods em nós diferentes; permissão de arquivo (`fsGroup`) | `kubectl describe pod <p>` (Events: `Multi-Attach error`); `kubectl get pvc <pvc> -o wide` | Use `ReadWriteMany` (EFS) ou garanta um Pod por vez; ajuste `securityContext.fsGroup` |
| **Forbidden (RBAC)** | Role/ClusterRole sem o verbo, recurso ou apiGroup necessário; binding no namespace errado; ServiceAccount errada no Pod | `kubectl auth can-i <verbo> <recurso> --as=system:serviceaccount:<ns>:<sa> -n <ns>`; `kubectl describe role <r> -n <ns>`; `kubectl get rolebinding -n <ns> -o yaml` | Acrescente o verbo/recurso na Role; confira o `apiGroup` (`""` para core, `apps`, `batch`, `networking.k8s.io`); confira `roleRef` e `subjects.namespace` |
| **"error: You must be logged in to the server (Unauthorized)"** | Certificado do kubeconfig expirado; token revogado; contexto apontando para cluster destruído | `kubectl config current-context`; `kubectl config view --minify`; para EKS: `aws sts get-caller-identity` | Regenere o kubeconfig (`aws eks update-kubeconfig --name <cluster> --region <r>`); renove credenciais |
| **Node NotReady** | kubelet parado; disco cheio; runtime (containerd) caído; rede do nó; certificado do kubelet expirado | `kubectl describe node <n>` (Conditions); `kubectl get node <n> -o yaml`; no nó: `systemctl status kubelet`, `journalctl -u kubelet -n 100` | Reinicie o kubelet; libere disco (`crictl rmi --prune`); verifique o CNI; no EKS, substitua a instância |
| **Node com DiskPressure** | Imagens acumuladas, logs sem rotação, emptyDir grande | `kubectl describe node <n>`; no nó: `df -h`, `crictl images` | `crictl rmi --prune`; configure rotação de logs; use `sizeLimit` em emptyDir |
| **HPA com TARGET `<unknown>`** | metrics-server ausente ou não pronto; container sem `requests.cpu` | `kubectl get hpa`; `kubectl top pods`; `kubectl get deploy metrics-server -n kube-system` | `minikube addons enable metrics-server`; defina `resources.requests.cpu` no container |
| **HPA não escala** | `maxReplicas` atingido; `stabilizationWindowSeconds` longo; métrica abaixo do alvo; PDB bloqueando | `kubectl describe hpa <hpa>` (seção Conditions e Events) | Ajuste limites e `behavior`; confirme que a carga realmente aumenta a métrica observada |
| **Deployment travado em rollout** | Novos Pods nunca ficam Ready; `progressDeadlineSeconds` expirado; quota impedindo `maxSurge` | `kubectl rollout status deploy/<d>`; `kubectl describe deploy <d>`; `kubectl get rs -l app=<app>` | Corrija a readinessProbe ou a imagem; `kubectl rollout undo deploy/<d>`; ajuste `maxSurge` sob quota apertada |
| **`kubectl drain` não termina** | PDB impedindo evicção; DaemonSets; Pods com emptyDir | `kubectl get pdb -A`; `kubectl describe pdb <pdb>` | Ajuste `minAvailable`; use `--ignore-daemonsets --delete-emptydir-data`; escale o workload antes do drain |
| **Argo CD OutOfSync permanente** | Campo alterado por outro controlador (HPA, admission mutation); anotações injetadas por webhook | `argocd app diff <app>`; `kubectl describe application <app> -n argocd` | Adicione `ignoreDifferences` para o campo; use `ServerSideApply=true` |
| **Argo CD `ComparisonError`** | Repositório inacessível; credencial expirada; path inexistente; chart Helm com erro | `kubectl logs -n argocd deploy/argocd-repo-server --tail=100` | Corrija a URL/credencial do repo; valide localmente com `kustomize build` ou `helm template` |

### C.4 Comandos de diagnóstico rápido (copiar e colar)

```bash
# Panorama geral: o que está quebrado agora
kubectl get pods -A --field-selector=status.phase!=Running,status.phase!=Succeeded

# Pods que mais reiniciam no cluster
kubectl get pods -A --sort-by='.status.containerStatuses[0].restartCount' | tail -15

# Todos os avisos recentes
kubectl get events -A --field-selector type=Warning --sort-by=.lastTimestamp | tail -30

# Nós com problema
kubectl get nodes -o wide
kubectl describe nodes | grep -E "Name:|Taints:|Conditions:|Pressure" | head -40

# Componentes de sistema
kubectl get pods -n kube-system

# Motivo da última terminação de um container
kubectl get pod <pod> -o jsonpath='{.status.containerStatuses[0].lastState.terminated}' | jq

# Ver o que o Service enxerga
kubectl get svc,endpoints,endpointslices -n <ns> -l app=store-api

# Teste completo de rede a partir de um Pod descartável
kubectl run netshoot --rm -it --image=nicolaka/netshoot --restart=Never -n store-dev -- bash
#   dig store-api.store-dev.svc.cluster.local
#   curl -v http://store-api/health
#   nc -zv store-db 5432
```

---

## Apêndice D — Glossário

**Admission Controller** — Plugin do API Server que intercepta requisições já autenticadas e
autorizadas, podendo validá-las (validating) ou modificá-las (mutating) antes da persistência no
etcd.

**Affinity / Anti-affinity** — Regras que atraem ou afastam Pods de nós (`nodeAffinity`) ou de outros
Pods (`podAffinity`, `podAntiAffinity`), usadas para espalhar réplicas e garantir disponibilidade.

**Annotation** — Par chave-valor em `metadata` destinado a metadados arbitrários (configuração de
controllers, URLs, checksums). Diferentemente das labels, não é usada em seletores.

**API Group** — Agrupamento lógico de recursos na API do Kubernetes, como `apps`, `batch`,
`networking.k8s.io` e `rbac.authorization.k8s.io`. O grupo core é representado pela string vazia.

**API Server (kube-apiserver)** — Componente do control plane que expõe a API REST do Kubernetes. É
o único componente que fala diretamente com o etcd e o ponto por onde passa toda operação.

**Argo CD** — Controlador GitOps declarativo para Kubernetes que sincroniza continuamente o estado
do cluster com manifestos versionados em um repositório Git.

**Argo Rollouts** — Controlador que substitui o Deployment para oferecer estratégias avançadas de
entrega, como blue-green e canary com análise automatizada de métricas.

**Autoscaler (Cluster Autoscaler / Karpenter)** — Componente que adiciona ou remove **nós** do
cluster conforme a demanda de Pods pendentes. Não confundir com HPA, que ajusta réplicas.

**Cert-manager** — Operador que emite e renova certificados TLS automaticamente em Kubernetes,
integrando-se ao Let's Encrypt e a CAs internas.

**cgroups (control groups)** — Recurso do kernel Linux que limita e contabiliza CPU, memória e I/O de
um conjunto de processos. É o que torna `resources.limits` efetivo.

**CNI (Container Network Interface)** — Especificação e conjunto de plugins que atribuem IPs aos Pods
e configuram a rede do cluster. Exemplos: Calico, Cilium, Flannel, AWS VPC CNI.

**ConfigMap** — Objeto que armazena configuração não sensível em pares chave-valor ou arquivos,
consumível como variáveis de ambiente ou volumes.

**Container Runtime** — Software que executa contêineres no nó (containerd, CRI-O). O Docker Engine
não é mais suportado diretamente desde a remoção do dockershim na versão 1.24.

**Control Plane** — Conjunto de componentes que decidem o estado do cluster: API Server, etcd,
scheduler, controller-manager e cloud-controller-manager.

**Controller** — Loop de reconciliação que observa o estado desejado (spec) e age para aproximar o
estado observado (status). É o padrão de projeto central do Kubernetes.

**CRD (CustomResourceDefinition)** — Objeto que estende a API do Kubernetes com um novo tipo de
recurso, base para operadores e para ferramentas como Argo CD, KEDA e Knative.

**CRI (Container Runtime Interface)** — Contrato gRPC entre o kubelet e o container runtime.

**CronJob** — Objeto `batch/v1` que cria Jobs conforme uma expressão cron.

**CSI (Container Storage Interface)** — Especificação padronizada para drivers de armazenamento em
Kubernetes; substitui os plugins in-tree.

**DaemonSet** — Controlador que garante uma cópia de um Pod em cada nó (ou em cada nó que atenda a um
seletor). Típico de agentes de log, métricas e rede.

**Deployment** — Controlador que gerencia ReplicaSets para oferecer atualizações declarativas,
rolling updates e rollback de aplicações stateless.

**Downward API** — Mecanismo que expõe metadados do próprio Pod (nome, namespace, labels, IP, limites
de recurso) como variáveis de ambiente ou arquivos.

**Drain** — Operação (`kubectl drain`) que remove Pods de um nó de forma ordenada, respeitando
PodDisruptionBudgets, para permitir manutenção.

**EKS (Amazon Elastic Kubernetes Service)** — Serviço gerenciado da AWS que opera o control plane do
Kubernetes; os nós podem ser EC2 gerenciados, autogerenciados ou Fargate.

**eksctl** — CLI oficial para criar, alterar e destruir clusters EKS, gerando pilhas do CloudFormation
por trás.

**Endpoints / EndpointSlice** — Objetos que listam os IPs e portas dos Pods prontos que atendem a um
Service. O EndpointSlice é a API atual, mais escalável.

**Ephemeral Container** — Container temporário injetado em um Pod em execução para depuração, sem
reiniciá-lo. Base do `kubectl debug`.

**etcd** — Banco de dados distribuído chave-valor, consistente e baseado em Raft, onde todo o estado
do cluster é persistido.

**Eviction** — Remoção de um Pod pelo kubelet devido à pressão de recursos no nó, ou pela API de
eviction em um drain.

**Fargate (AWS)** — Modo serverless de execução de contêineres na AWS; no EKS, cada Pod roda em uma
microVM isolada, sem que o usuário gerencie nós EC2.

**Finalizer** — Chave em `metadata.finalizers` que impede a remoção definitiva de um objeto até que um
controlador execute uma limpeza e remova a entrada.

**Fleet** — Motor de GitOps do Rancher, voltado à entrega de configuração para muitos clusters
simultaneamente.

**GitOps** — Prática operacional em que o estado desejado do sistema é declarado em Git e um agente
no cluster reconcilia continuamente o estado real com o versionado.

**Headless Service** — Service com `clusterIP: None`; o DNS resolve diretamente para os IPs dos Pods,
sem balanceamento. Requisito de StatefulSets.

**Helm** — Gerenciador de pacotes do Kubernetes; empacota manifestos em charts parametrizáveis e
gerencia releases com histórico e rollback.

**HPA (HorizontalPodAutoscaler)** — Controlador que ajusta o número de réplicas de um workload com
base em métricas de CPU, memória ou métricas customizadas.

**Image Pull Secret** — Secret do tipo `kubernetes.io/dockerconfigjson` usado pelo kubelet para
autenticar em registries privados.

**Ingress** — Objeto que define regras de roteamento HTTP/HTTPS (host, path, TLS) para Services. Só
funciona com um Ingress Controller instalado.

**Ingress Controller** — Implementação que lê objetos Ingress e configura um proxy real
(ingress-nginx, Traefik, AWS Load Balancer Controller).

**Init Container** — Container que roda até a conclusão, em sequência, antes dos containers
principais do Pod. Usado para migrações, espera por dependências e preparação de volumes.

**IRSA (IAM Roles for Service Accounts)** — Mecanismo do EKS que associa uma role do IAM a uma
ServiceAccount, permitindo acesso a serviços da AWS sem credenciais estáticas. O EKS Pod Identity é a
alternativa mais recente.

**Job** — Objeto `batch/v1` que executa Pods até um número definido de conclusões bem-sucedidas.

**KEDA (Kubernetes Event-driven Autoscaling)** — Projeto CNCF que escala workloads com base em
eventos externos (filas SQS, Kafka, Prometheus, cron), inclusive **até zero réplicas**.

**Knative** — Plataforma serverless sobre Kubernetes; o componente Serving oferece escala a zero e
versionamento de revisões, e o Eventing oferece roteamento de eventos.

**kubeadm** — Ferramenta oficial para inicializar e juntar nós em um cluster Kubernetes conforme as
boas práticas do projeto.

**kubeconfig** — Arquivo (por padrão `~/.kube/config`) com clusters, usuários e contextos usados pelo
`kubectl` e demais clientes.

**kubectl** — CLI oficial de interação com a API do Kubernetes.

**kubelet** — Agente que roda em cada nó, recebe PodSpecs do API Server, instrui o runtime a executar
os containers e reporta o status.

**kube-proxy** — Componente de nó que programa regras (iptables ou IPVS) para implementar o
balanceamento dos Services.

**Kustomize** — Ferramenta nativa do `kubectl` (`-k`) que compõe manifestos por sobreposição de
*bases* e *overlays*, sem templates.

**Kyverno** — Policy engine para Kubernetes que valida, muta, gera e verifica assinaturas de imagens
usando políticas escritas em YAML.

**Label** — Par chave-valor em `metadata.labels` usado por seletores para agrupar objetos. É o
mecanismo que liga Services a Pods e controladores a réplicas.

**LimitRange** — Objeto que define valores mínimos, máximos e padrões de recursos por container, Pod
ou PVC dentro de um namespace.

**Liveness Probe** — Verificação periódica que, ao falhar repetidamente, faz o kubelet **reiniciar** o
container.

**LoadBalancer (tipo de Service)** — Service que solicita ao provedor de nuvem a criação de um
balanceador externo apontando para os nós ou Pods.

**Manifest** — Arquivo YAML ou JSON que descreve o estado desejado de um ou mais objetos da API.

**metrics-server** — Agregador leve de métricas de CPU e memória que alimenta `kubectl top` e o HPA.

**minikube** — Distribuição local de Kubernetes que sobe um cluster de um ou mais nós em VM ou
contêiner, para desenvolvimento e ensino.

**Namespace** — Partição lógica do cluster que agrupa objetos, delimita nomes e serve de escopo para
quotas, RBAC e NetworkPolicies.

**NetworkPolicy** — Objeto que define regras de tráfego permitido (ingress e egress) entre Pods. Só é
efetivo com um CNI que a implemente.

**Node** — Máquina (física ou virtual) que executa Pods; roda kubelet, kube-proxy e um container
runtime.

**NodePort** — Tipo de Service que reserva a mesma porta (30000-32767) em todos os nós e a encaminha
para os Pods.

**OOMKilled** — Terminação de um container pelo kernel por exceder o limite de memória; aparece com
exit code 137.

**Operator** — Controlador customizado que codifica conhecimento operacional sobre uma aplicação,
tipicamente combinando CRDs com um loop de reconciliação.

**Overlay (Kustomize)** — Diretório que aplica modificações específicas de ambiente (dev, prod) sobre
uma base comum de manifestos.

**PDB (PodDisruptionBudget)** — Política que define o mínimo de Pods disponíveis (ou o máximo
indisponível) durante disrupções voluntárias.

**Pod** — Menor unidade implantável do Kubernetes: um ou mais containers que compartilham namespace de
rede, IPC e volumes, agendados juntos no mesmo nó.

**Pod Security Admission (PSA)** — Admission controller nativo que aplica os Pod Security Standards
(`privileged`, `baseline`, `restricted`) por namespace. Substituiu as PodSecurityPolicies, removidas
na versão 1.25.

**PriorityClass** — Objeto que define a prioridade de agendamento de um Pod e habilita preempção de
Pods de prioridade inferior.

**Prometheus** — Sistema de monitoramento e banco de séries temporais, padrão de fato para métricas
em ambientes cloud native.

**Probe** — Verificação executada pelo kubelet sobre um container. Há três tipos: `startupProbe`,
`livenessProbe` e `readinessProbe`.

**PV (PersistentVolume)** — Recurso de armazenamento no cluster, provisionado estática ou
dinamicamente, com ciclo de vida independente do Pod.

**PVC (PersistentVolumeClaim)** — Pedido de armazenamento feito por um usuário, que o Kubernetes
vincula (bind) a um PV compatível.

**QoS Class** — Classificação atribuída ao Pod conforme requests e limits: `Guaranteed` (iguais),
`Burstable` (parciais) ou `BestEffort` (ausentes). Determina a ordem de eviction.

**Rancher** — Plataforma de gestão multi-cluster que centraliza autenticação, políticas,
observabilidade e entrega (via Fleet) de vários clusters Kubernetes.

**RBAC (Role-Based Access Control)** — Modelo de autorização do Kubernetes baseado em Roles e
ClusterRoles (o que pode ser feito) ligados a sujeitos por RoleBindings e ClusterRoleBindings.

**Readiness Probe** — Verificação que determina se o Pod pode receber tráfego; ao falhar, o Pod é
removido dos Endpoints do Service sem ser reiniciado.

**Reconciliation Loop** — Ciclo contínuo de comparação entre estado desejado e observado, seguido de
ações corretivas. Fundamento operacional de todos os controladores.

**ReplicaSet** — Controlador que mantém um número fixo de réplicas de um Pod. Normalmente é gerenciado
indiretamente por um Deployment.

**ResourceQuota** — Objeto que limita o consumo agregado de recursos e a contagem de objetos em um
namespace.

**Rolling Update** — Estratégia de atualização que substitui Pods gradualmente, controlada por
`maxSurge` e `maxUnavailable`, permitindo implantação sem downtime.

**Secret** — Objeto para dados sensíveis, armazenados em base64 e, se configurado, criptografados em
repouso no etcd.

**SecurityContext** — Bloco que define atributos de segurança do Pod ou do container: usuário,
capabilities, `readOnlyRootFilesystem`, seccomp, `allowPrivilegeEscalation`.

**Selector** — Expressão que casa objetos por labels; existe na forma de igualdade
(`app=store-api`) e de conjunto (`env in (dev,prod)`).

**Serverless** — Modelo de execução em que a infraestrutura é totalmente gerenciada pelo provedor, com
escala automática (inclusive a zero) e cobrança por uso efetivo.

**Service** — Abstração que expõe um conjunto de Pods sob um nome DNS e um IP virtual estáveis, com
balanceamento de carga. Tipos: ClusterIP, NodePort, LoadBalancer e ExternalName.

**ServiceAccount** — Identidade usada por processos dentro de Pods para autenticar no API Server.

**Sidecar** — Container auxiliar que roda ao lado do container principal no mesmo Pod, adicionando
funcionalidade (proxy, log shipper, sincronizador de arquivos).

**StatefulSet** — Controlador para aplicações com estado: garante identidade de rede estável, ordem de
criação e remoção e armazenamento persistente dedicado por réplica.

**StorageClass** — Objeto que descreve uma "classe" de armazenamento e o provisionador responsável por
criar PVs dinamicamente (ex.: `gp3` no EBS).

**Taint / Toleration** — Taints marcam um nó como repulsivo a Pods; tolerations permitem que Pods
específicos ignorem essa marcação.

**Topology Spread Constraints** — Regras que distribuem réplicas de forma equilibrada entre domínios
de topologia (nós, zonas de disponibilidade).

**Trivy** — Scanner open source que detecta vulnerabilidades, segredos e configurações inseguras em
imagens, manifestos e repositórios, além de gerar SBOM.

**Volume** — Diretório acessível aos containers de um Pod. Pode ser efêmero (`emptyDir`), de
configuração (`configMap`, `secret`) ou persistente (`persistentVolumeClaim`).

**Webhook (Admission)** — Serviço HTTPS externo chamado pelo API Server para validar ou mutar objetos,
usado por Kyverno, OPA/Gatekeeper, cert-manager e service meshes.

**Workload** — Termo genérico para uma aplicação em execução no Kubernetes. Os recursos de workload
são Pod, Deployment, ReplicaSet, StatefulSet, DaemonSet, Job e CronJob.

---

## Apêndice E — Preparação do ambiente do aluno

### E.1 Requisitos mínimos de hardware

| Item | Mínimo | Recomendado |
|---|---|---|
| CPU | 4 núcleos com virtualização (VT-x/AMD-V) habilitada | 6 a 8 núcleos |
| Memória RAM | 8 GB | 16 GB |
| Disco livre | 30 GB | 60 GB (SSD) |
| Sistema operacional | Windows 11 com WSL2, macOS 13+, Ubuntu 22.04+ | — |
| Conexão | Banda larga estável (download de imagens de contêiner) | — |

> **Atenção:** com 8 GB de RAM, feche navegador e IDE antes de subir o minikube com 4 GB. Se a
> máquina não aguentar, use os laboratórios em nuvem indicados na seção E.7.

### E.2 Windows 11 com WSL2

**Passo 1 — Habilitar o WSL2.** Abra o PowerShell **como administrador**:

```powershell
wsl --install -d Ubuntu-22.04
wsl --set-default-version 2
wsl --update
```

Reinicie a máquina. Ao abrir o Ubuntu pela primeira vez, defina usuário e senha do Linux.

**Passo 2 — Verificar a versão.**

```powershell
wsl -l -v
#   NAME            STATE           VERSION
# * Ubuntu-22.04    Running         2
```

**Passo 3 — Instalar o Docker Desktop.** Baixe em https://www.docker.com/products/docker-desktop/ e,
durante a instalação, marque **"Use WSL 2 based engine"**. Depois, em
`Settings → Resources → WSL Integration`, habilite a distribuição Ubuntu-22.04.

**Passo 4 — Limitar o consumo do WSL2.** Crie `C:\Users\<seu-usuario>\.wslconfig`:

```ini
[wsl2]
memory=6GB
processors=4
swap=2GB
```

Aplique com `wsl --shutdown` no PowerShell e reabra o Ubuntu.

**Passo 5 — Instalar as ferramentas dentro do Ubuntu (WSL).** A partir daqui, **todos os comandos são
executados no terminal do Ubuntu**, não no PowerShell. Siga a seção E.4 (Linux).

> **Atenção:** trabalhe sempre dentro do sistema de arquivos do Linux (`/home/<usuario>/projetos`),
> nunca em `/mnt/c/...`. O acesso cruzado é dezenas de vezes mais lento e quebra permissões de
> arquivo, causando erros estranhos em volumes e em `kubectl apply`.

### E.3 macOS (Intel e Apple Silicon)

**Passo 1 — Instalar o Homebrew** (se ainda não tiver):

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
# Apple Silicon: adicione ao PATH conforme instruído ao final da instalação
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

**Passo 2 — Runtime de contêineres.** Escolha uma opção:

```bash
# Opção A: Docker Desktop (mais simples)
brew install --cask docker
open -a Docker      # conclua a configuração na interface

# Opção B: Colima (leve, sem GUI, sem licença comercial)
brew install colima docker docker-credential-helper
colima start --cpu 4 --memory 8 --disk 60
```

**Passo 3 — Ferramentas do Kubernetes.**

```bash
brew install kubectl minikube helm k9s awscli eksctl kubectx stern jq yq
brew install krew
echo 'export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
kubectl krew install tree neat who-can
```

**Passo 4 — Nota sobre Apple Silicon (M1/M2/M3/M4).** A arquitetura é `arm64`. Imagens publicadas só
para `amd64` falham com `exec format error`. Ao construir imagens que serão executadas no EKS
(`amd64`), use buildx:

```bash
docker buildx create --use --name multiarch 2>/dev/null || docker buildx use multiarch
docker buildx build --platform linux/amd64,linux/arm64 \
  -t docker.io/seuusuario/store-api:1.0 --push .
```

Para rodar localmente uma imagem só `amd64`, acrescente `--platform linux/amd64` ao `docker run`
(há perda de desempenho por emulação).

### E.4 Linux (Ubuntu / Debian)

**Passo 1 — Docker Engine (repositório oficial).**

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg lsb-release
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
  sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin

# Executar docker sem sudo
sudo usermod -aG docker $USER
newgrp docker
docker run --rm hello-world
```

**Passo 2 — kubectl.**

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check      # deve imprimir "kubectl: OK"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm kubectl kubectl.sha256
kubectl version --client
```

**Passo 3 — minikube.**

```bash
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
rm minikube-linux-amd64

minikube start --driver=docker --cpus=4 --memory=4096 --kubernetes-version=v1.33.0
minikube addons enable metrics-server
minikube addons enable ingress
minikube status
```

**Passo 4 — Helm.**

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

**Passo 5 — k9s.**

```bash
curl -sS https://webi.sh/k9s | sh
# alternativa: baixar o .deb da página de releases
# https://github.com/derailed/k9s/releases
k9s version
```

**Passo 6 — AWS CLI v2.**

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip -q awscliv2.zip && sudo ./aws/install
rm -rf awscliv2.zip aws
aws --version

aws configure          # informe Access Key, Secret Key, região (us-east-1) e formato (json)
aws sts get-caller-identity
```

**Passo 7 — eksctl.**

```bash
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_${PLATFORM}.tar.gz"
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

**Passo 8 — Ferramentas auxiliares.**

```bash
sudo apt-get install -y jq tree
# yq
sudo curl -L https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 \
  -o /usr/local/bin/yq && sudo chmod +x /usr/local/bin/yq
# stern
sudo curl -L https://github.com/stern/stern/releases/latest/download/stern_linux_amd64.tar.gz \
  | sudo tar xz -C /usr/local/bin stern
```

### E.5 VS Code e extensões

Instale o VS Code em https://code.visualstudio.com/ e adicione as extensões abaixo:

| Extensão | ID | Para que serve |
|---|---|---|
| Kubernetes | `ms-kubernetes-tools.vscode-kubernetes-tools` | Explorador de cluster, logs, exec, aplicar manifestos |
| YAML | `redhat.vscode-yaml` | Validação e autocompletar com schemas do Kubernetes |
| Docker | `ms-azuretools.vscode-docker` | Build, imagens, containers e Dockerfile |
| WSL | `ms-vscode-remote.remote-wsl` | Editar arquivos do WSL2 (obrigatório no Windows) |
| Remote - Containers | `ms-vscode-remote.remote-containers` | Dev containers |
| Python | `ms-python.python` | Desenvolvimento do `store-api` |
| GitLens | `eamodio.gitlens` | Histórico e autoria no Git (útil no capítulo de GitOps) |
| Helm Intellisense | `tim-koehler.helm-intellisense` | Autocompletar em templates de chart |
| AWS Toolkit | `amazonwebservices.aws-toolkit-vscode` | Navegar recursos AWS, incluindo Lambda e ECR |

Instalação por linha de comando:

```bash
code --install-extension ms-kubernetes-tools.vscode-kubernetes-tools
code --install-extension redhat.vscode-yaml
code --install-extension ms-azuretools.vscode-docker
code --install-extension ms-python.python
code --install-extension eamodio.gitlens
code --install-extension tim-koehler.helm-intellisense
code --install-extension amazonwebservices.aws-toolkit-vscode
```

Configure o autocompletar de manifestos do Kubernetes em `settings.json` do VS Code:

```json
{
  "yaml.schemas": {
    "https://raw.githubusercontent.com/yannh/kubernetes-json-schema/master/v1.33.0-standalone-strict/all.json": [
      "*.k8s.yaml",
      "manifests/**/*.yaml",
      "k8s/**/*.yaml"
    ]
  },
  "yaml.format.enable": true,
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "files.eol": "\n",
  "[yaml]": {
    "editor.autoIndent": "keep",
    "editor.defaultFormatter": "redhat.vscode-yaml"
  }
}
```

> **Atenção:** YAML **não aceita tabulação** para indentação. Se um manifesto der erro
> `found character '\t' that cannot start any token`, o problema é esse. A configuração acima já
> força espaços.

### E.6 Script de sanidade do ambiente

Salve como `verifica-ambiente.sh`, dê permissão (`chmod +x verifica-ambiente.sh`) e execute.

```bash
#!/usr/bin/env bash
# verifica-ambiente.sh — checagem do ambiente de Cloud Developer II
# Uso: ./verifica-ambiente.sh

set -u
OK=0; FALHAS=0; AVISOS=0
verde() { printf '\033[0;32m%s\033[0m\n' "$1"; }
verm()  { printf '\033[0;31m%s\033[0m\n' "$1"; }
amar()  { printf '\033[0;33m%s\033[0m\n' "$1"; }

checa_bin() {
  local bin="$1" obrigatorio="${2:-sim}"
  if command -v "$bin" >/dev/null 2>&1; then
    local v
    v=$("$bin" version --client --short 2>/dev/null \
        || "$bin" version --short 2>/dev/null \
        || "$bin" --version 2>/dev/null \
        || "$bin" version 2>/dev/null)
    verde "[OK]     $bin encontrado: $(echo "$v" | head -1)"
    OK=$((OK+1))
  elif [ "$obrigatorio" = "sim" ]; then
    verm "[FALHA]  $bin NAO encontrado (obrigatorio)"
    FALHAS=$((FALHAS+1))
  else
    amar "[AVISO]  $bin nao encontrado (opcional)"
    AVISOS=$((AVISOS+1))
  fi
}

echo "==================================================="
echo " Verificacao do ambiente — Cloud Developer II"
echo " Data: $(date '+%d/%m/%Y %H:%M')"
echo " Sistema: $(uname -s) $(uname -m)"
echo "==================================================="
echo
echo "--- 1. Binarios obrigatorios ---"
checa_bin docker
checa_bin kubectl
checa_bin minikube
checa_bin helm
echo
echo "--- 2. Binarios recomendados ---"
checa_bin k9s   nao
checa_bin aws   nao
checa_bin eksctl nao
checa_bin jq    nao
checa_bin yq    nao
checa_bin git   nao

echo
echo "--- 3. Docker daemon ---"
if docker info >/dev/null 2>&1; then
  verde "[OK]     daemon do Docker respondendo"
  OK=$((OK+1))
else
  verm "[FALHA]  daemon do Docker NAO responde (inicie o Docker Desktop/servico)"
  FALHAS=$((FALHAS+1))
fi

echo
echo "--- 4. Cluster Kubernetes ---"
if kubectl cluster-info >/dev/null 2>&1; then
  verde "[OK]     cluster acessivel: $(kubectl config current-context)"
  kubectl get nodes -o custom-columns='NO:.metadata.name,STATUS:.status.conditions[-1].type,VERSAO:.status.nodeInfo.kubeletVersion' --no-headers
  OK=$((OK+1))
else
  verm "[FALHA]  nenhum cluster acessivel (connection refused)."
  CTX=$(kubectl config current-context 2>/dev/null || echo "<nenhum>")
  SRV=$(kubectl config view --minify -o jsonpath='{.clusters[0].cluster.server}' 2>/dev/null)
  amar "         contexto ativo: $CTX"
  amar "         apontando para: ${SRV:-<nao definido>}"
  amar "         Se o endereco acima e 127.0.0.1 com porta alta, o kubeconfig esta ORFAO"
  amar "         (cluster local apagado ou Docker Desktop com Kubernetes desligado)."
  amar "         Corrija com:  minikube start --cpus=4 --memory=4096"
  amar "         ou troque de contexto:  kubectl config get-contexts && kubectl config use-context <nome>"
  FALHAS=$((FALHAS+1))
fi

echo
echo "--- 5. Addons e componentes ---"
if kubectl top nodes >/dev/null 2>&1; then
  verde "[OK]     metrics-server ativo (kubectl top funciona)"
  OK=$((OK+1))
else
  amar "[AVISO]  metrics-server indisponivel. Execute: minikube addons enable metrics-server"
  AVISOS=$((AVISOS+1))
fi

if kubectl get pods -n ingress-nginx >/dev/null 2>&1; then
  verde "[OK]     ingress-nginx presente"
  OK=$((OK+1))
else
  amar "[AVISO]  ingress controller ausente. Execute: minikube addons enable ingress"
  AVISOS=$((AVISOS+1))
fi

echo
echo "--- 6. Teste funcional ponta a ponta ---"
kubectl delete namespace sanidade --ignore-not-found >/dev/null 2>&1
kubectl create namespace sanidade >/dev/null 2>&1
kubectl create deployment teste --image=nginx:1.27-alpine -n sanidade >/dev/null 2>&1
kubectl expose deployment teste --port=80 -n sanidade >/dev/null 2>&1

if kubectl wait --for=condition=available deploy/teste -n sanidade --timeout=120s >/dev/null 2>&1; then
  verde "[OK]     Deployment criado, imagem baixada e Pod pronto"
  OK=$((OK+1))
  if kubectl run curl-teste --rm -i --restart=Never --image=curlimages/curl:8.10.1 \
       -n sanidade --timeout=90s -- -s -o /dev/null -w '%{http_code}' http://teste 2>/dev/null | grep -q 200; then
    verde "[OK]     Service resolvido por DNS e respondendo HTTP 200"
    OK=$((OK+1))
  else
    amar "[AVISO]  DNS ou Service nao responderam como esperado"
    AVISOS=$((AVISOS+1))
  fi
else
  verm "[FALHA]  o Deployment de teste nao ficou pronto em 120s"
  kubectl get pods -n sanidade
  kubectl describe pods -n sanidade | tail -20
  FALHAS=$((FALHAS+1))
fi
kubectl delete namespace sanidade --wait=false >/dev/null 2>&1

echo
echo "==================================================="
echo " Resultado: $OK OK | $AVISOS avisos | $FALHAS falhas"
if [ "$FALHAS" -eq 0 ]; then
  verde " Ambiente pronto para a disciplina."
  exit 0
else
  verm " Corrija as falhas acima antes da proxima aula."
  exit 1
fi
```

Saída esperada em um ambiente correto:

```
===================================================
 Verificacao do ambiente — Cloud Developer II
 Data: 06/08/2026 14:22
 Sistema: Linux x86_64
===================================================

--- 1. Binarios obrigatorios ---
[OK]     docker encontrado: Docker version 27.3.1, build ce12230
[OK]     kubectl encontrado: Client Version: v1.33.0
[OK]     minikube encontrado: minikube version: v1.35.0
[OK]     helm encontrado: v3.16.2+g13654a5

--- 4. Cluster Kubernetes ---
[OK]     cluster acessivel: minikube
minikube   Ready   v1.33.0

--- 6. Teste funcional ponta a ponta ---
[OK]     Deployment criado, imagem baixada e Pod pronto
[OK]     Service resolvido por DNS e respondendo HTTP 200

===================================================
 Resultado: 12 OK | 0 avisos | 0 falhas
 Ambiente pronto para a disciplina.
```

### E.7 Alternativas em nuvem (quando a máquina não aguenta)

| Opção | Descrição | Limite gratuito | Endereço |
|---|---|---|---|
| Killercoda | Cenários interativos de Kubernetes no navegador | Sessões de 1 hora, gratuito | https://killercoda.com/playgrounds/scenario/kubernetes |
| Play with Kubernetes | Cluster efêmero multi-nó no navegador | Sessões de 4 horas | https://labs.play-with-k8s.com/ |
| GitHub Codespaces | Ambiente Linux completo com Docker | 60 h/mês no plano gratuito; mais no GitHub Education | https://github.com/features/codespaces |
| AWS Academy / AWS Educate | Créditos e laboratórios para EKS | Conforme a instituição | https://aws.amazon.com/training/awsacademy/ |
| Google Cloud Shell | Terminal com kubectl e Docker pré-instalados | 50 h/semana | https://shell.cloud.google.com/ |

### E.8 Problemas frequentes na instalação

| Problema | Causa | Solução |
|---|---|---|
| `minikube start` falha com "This computer doesn't have VT-X/AMD-v enabled" | Virtualização desabilitada na BIOS/UEFI | Habilite Intel VT-x ou AMD-V na BIOS; no Windows, desative o Hyper-V se usar o driver VirtualBox |
| `permission denied while trying to connect to the Docker daemon socket` | Usuário fora do grupo `docker` | `sudo usermod -aG docker $USER && newgrp docker` |
| `The connection to the server localhost:8080 was refused` | Nenhum kubeconfig válido; cluster parado | `minikube start`; confira `echo $KUBECONFIG` e `kubectl config current-context` |
| `The connection to the server 127.0.0.1:<porta alta> was refused` (com `couldn't get current server API group list`) | Kubeconfig **órfão**: contexto de um minikube/kind apagado, ou Docker Desktop com Kubernetes desligado. Muito comum no macOS | `kubectl config get-contexts` para ver o contexto ativo; `minikube start` para recriar; `kubectl config use-context minikube`; ou `kubectl config delete-context <nome>` para limpar. Detalhes no Apêndice C.1.1 |
| `minikube` sobe mas Pods ficam em `ImagePullBackOff` | Sem acesso à internet, proxy corporativo ou rate limit do Docker Hub | Configure o proxy (`minikube start --docker-env HTTP_PROXY=...`); faça `docker login`; use `minikube image load` |
| `exec format error` ao rodar a imagem | Imagem `amd64` em Apple Silicon (ou vice-versa) | Reconstrua com `docker buildx --platform` |
| WSL2 consumindo toda a RAM (`vmmem`) | Sem limite configurado | Crie `.wslconfig` com `memory=6GB` e rode `wsl --shutdown` |
| `kubectl` lento e travando no WSL2 | Arquivos em `/mnt/c` | Mova o projeto para `~/projetos` dentro do Linux |
| minikube sem espaço em disco | Imagens acumuladas | `minikube ssh -- docker system prune -a`; ou `minikube delete && minikube start --disk-size=40g` |
| `x509: certificate has expired` após dias parado | Certificados do minikube expiraram | `minikube delete && minikube start` |

---

## Apêndice F — Mapa de competências

### F.1 Estrutura de avaliação da disciplina

| Instrumento | Cobertura | Formato | Peso sugerido |
|---|---|---|---|
| **Check Point 1** | Capítulos 1 a 4 | Prática em cluster local: criar e expor a FIAP Store, garantir alta disponibilidade e escala | 25% |
| **Check Point 2** | Capítulos 5 a 8 | Prática: isolamento por namespace, NetworkPolicy, cluster gerenciado, volumes, Jobs e Secrets | 25% |
| **Check Point 3** | Capítulos 9 a 11 | Prática: RBAC de mínimo privilégio, hardening e pipeline GitOps com Argo CD | 25% |
| **Projeto integrador** | Capítulos 1 a 11 | FIAP Store completa, versionada em Git, entregue por Argo CD, com HPA, probes, RBAC e NetworkPolicy | 25% |

### F.2 Mapeamento capítulo a capítulo

| Cap. | Título | Objetivos de aprendizagem (verbos de Bloom) | Check Point | Domínio CKA | Domínio CKAD | Domínio CKS |
|---|---|---|---|---|---|---|
| 1 | Orquestradores e arquitetura do Kubernetes | Identificar o problema que a orquestração resolve; descrever os componentes do control plane e do nó; explicar o loop de reconciliação; relacionar Borg, Omega e Kubernetes | CP1 | Cluster Architecture, Installation & Configuration (25%) | Application Design and Build (20%) | Cluster Setup (15%) |
| 2 | minikube e primeiro cluster | Instalar e configurar o ambiente local; operar drivers, perfis e addons; aplicar comandos básicos de kubectl; diagnosticar falhas de inicialização | CP1 | Cluster Architecture, Installation & Configuration (25%) | Application Deployment (20%) | — |
| 3 | Pods e Services em YAML | Construir manifestos válidos de Pod e Service; distinguir os tipos de Service; configurar probes; analisar a resolução de nomes via DNS | CP1 | Services & Networking (20%); Workloads & Scheduling (15%) | Application Design and Build (20%); Services and Networking (20%) | — |
| 4 | Replicação, alta disponibilidade e HPA | Aplicar Deployments e ReplicaSets; executar rolling update e rollback; configurar HPA e PDB; avaliar estratégias de disponibilidade | CP1 | Workloads & Scheduling (15%) | Application Deployment (20%); Application Observability and Maintenance (15%) | — |
| 5 | Networking e namespaces | Analisar o modelo de rede plana do Kubernetes; segmentar o cluster com namespaces; construir NetworkPolicies; aplicar ResourceQuota e LimitRange | CP2 | Services & Networking (20%) | Services and Networking (20%) | Cluster Hardening (15%); Minimize Microservice Vulnerabilities (20%) |
| 6 | Cluster na nuvem com Amazon EKS | Provisionar um cluster EKS com eksctl; comparar node groups gerenciados e Fargate; configurar IRSA; estimar e controlar custos | CP2 | Cluster Architecture, Installation & Configuration (25%) | Application Deployment (20%) | Cluster Setup (15%) |
| 7 | Volumes, persistência e daemons | Distinguir volumes efêmeros e persistentes; construir PVC e StatefulSet; aplicar DaemonSets; analisar StorageClasses e modos de acesso | CP2 | Storage (10%); Workloads & Scheduling (15%) | Application Environment, Configuration and Security (25%) | — |
| 8 | Jobs, Secrets e Endpoints | Construir Jobs e CronJobs; gerenciar Secrets e ConfigMaps com segurança; investigar Endpoints e EndpointSlices; integrar processamento por eventos | CP2 | Workloads & Scheduling (15%); Services & Networking (20%) | Application Design and Build (20%); Application Environment, Configuration and Security (25%) | Supply Chain Security (20%) |
| 9 | RBAC e controle de acesso | Explicar autenticação, autorização e admissão; construir Roles e Bindings de mínimo privilégio; auditar permissões; corrigir erros de Forbidden | CP3 | Cluster Architecture, Installation & Configuration (25%) | Application Environment, Configuration and Security (25%) | Cluster Hardening (15%) |
| 10 | Boas práticas e segurança | Aplicar Pod Security Admission e SecurityContext; avaliar imagens com Trivy; definir requests, limits e QoS; construir políticas com Kyverno; analisar riscos do OWASP K8s Top 10 | CP3 | Troubleshooting (30%) | Application Observability and Maintenance (15%) | System Hardening (10%); Minimize Microservice Vulnerabilities (20%); Supply Chain Security (20%); Monitoring, Logging and Runtime Security (20%) |
| 11 | GitOps com Argo CD e Rancher | Explicar os quatro princípios do OpenGitOps; construir um repositório GitOps com Kustomize; operar Argo CD (sync, prune, self-heal); comparar Argo CD e Fleet; projetar entrega progressiva | CP3 + Projeto | — | Application Deployment (20%) | Supply Chain Security (20%) |

### F.3 Distribuição dos domínios de prova pelos capítulos

**CKA — Certified Kubernetes Administrator**

| Domínio | Peso | Capítulos que preparam | Grau de cobertura |
|---|---|---|---|
| Cluster Architecture, Installation & Configuration | 25% | 1, 2, 6, 9 | Alto (falta apenas kubeadm e backup de etcd) |
| Workloads & Scheduling | 15% | 3, 4, 7, 8 | Alto |
| Services & Networking | 20% | 3, 5, 8 | Alto |
| Storage | 10% | 7 | Alto |
| Troubleshooting | 30% | Apêndice C + seções "Erros comuns" de todos os capítulos | Médio-alto (pratique com cenários de nó) |

**CKAD — Certified Kubernetes Application Developer**

| Domínio | Peso | Capítulos que preparam | Grau de cobertura |
|---|---|---|---|
| Application Design and Build | 20% | 1, 3, 8 | Alto |
| Application Deployment | 20% | 2, 4, 6, 11 | Alto |
| Application Observability and Maintenance | 15% | 4, 10 | Médio-alto |
| Application Environment, Configuration and Security | 25% | 7, 8, 9, 10 | Alto |
| Services and Networking | 20% | 3, 5 | Alto |

**CKS — Certified Kubernetes Security Specialist** (exige CKA válido)

| Domínio | Peso | Capítulos que preparam | Grau de cobertura |
|---|---|---|---|
| Cluster Setup | 15% | 1, 5, 6 | Médio |
| Cluster Hardening | 15% | 5, 9 | Alto |
| System Hardening | 10% | 10 | Médio |
| Minimize Microservice Vulnerabilities | 20% | 5, 10 | Alto |
| Supply Chain Security | 20% | 8, 10, 11 | Médio-alto |
| Monitoring, Logging and Runtime Security | 20% | 10 | Médio |

> **Nota:** os pesos de domínio são revisados periodicamente pela Linux Foundation. Confira o
> currículo vigente no repositório oficial https://github.com/cncf/curriculum antes de se inscrever.

### F.4 Competências transversais (serverless)

O tema serverless aparece diluído nas seções "Ponte com Serverless" e não é avaliado em um Check
Point isolado. O quadro abaixo mostra onde cada conceito é trabalhado.

| Conceito serverless | Capítulo(s) | Como aparece |
|---|---|---|
| Modelo de responsabilidade compartilhada | 1, 6 | Comparação entre autogerenciar o control plane, usar EKS e usar Lambda |
| Escala a zero | 4, 8 | KEDA `ScaledObject` com `minReplicaCount: 0`; Knative Serving |
| Cold start e latência de inicialização | 4, 6 | Impacto de imagem grande, `startupProbe` e provisioned concurrency |
| Autoscaling orientado a eventos | 4, 8 | KEDA com scaler de fila SQS alimentando o `store-worker` |
| Nós sem gerenciamento | 6, 7 | AWS Fargate para EKS: sem nó EC2, sem DaemonSet, sem volume hostPath |
| Processamento em lote sob demanda | 8 | Job e CronJob vs. Lambda acionada por EventBridge Scheduler |
| Roteamento de eventos e CloudEvents | 8, 11 | Knative Eventing (Broker/Trigger) e EventBridge |
| Modelo de custo por requisição | 6, 10 | Comparação de custo entre nó EC2 ocioso, Fargate e invocação de Lambda |
| Entrega progressiva de funções | 11 | Revisions do Knative e aliases ponderados da Lambda; Argo Rollouts em Kubernetes |

### F.5 Autoavaliação: checklist de saída da disciplina

Ao final do semestre, o aluno deve conseguir executar cada item abaixo **sem consultar a apostila**,
usando apenas `kubectl explain` e a documentação oficial.

- [ ] Explicar, em até três minutos, o caminho de uma requisição desde `kubectl apply` até o
      container em execução, nomeando cada componente envolvido.
- [ ] Subir um cluster local com minikube, habilitar os addons de ingress e metrics-server e validar
      o ambiente.
- [ ] Escrever, do zero, um Deployment com probes, requests, limits e securityContext restritivo.
- [ ] Expor esse Deployment com ClusterIP, NodePort e Ingress com TLS.
- [ ] Executar um rolling update, detectar a falha e reverter com `kubectl rollout undo`.
- [ ] Configurar um HPA e demonstrar a escala sob carga sintética.
- [ ] Isolar dois namespaces com NetworkPolicy default-deny e liberar somente o tráfego necessário.
- [ ] Criar um cluster EKS com eksctl, implantar a aplicação e **destruir tudo** ao final.
- [ ] Persistir dados com PVC e StatefulSet, e explicar por que os PVCs sobrevivem ao StatefulSet.
- [ ] Processar uma carga em lote com Job e agendá-la com CronJob.
- [ ] Criar uma ServiceAccount com Role de mínimo privilégio e provar a restrição com
      `kubectl auth can-i`.
- [ ] Escanear uma imagem com Trivy e corrigir ao menos uma vulnerabilidade de severidade alta.
- [ ] Publicar a FIAP Store por meio de um repositório GitOps sincronizado pelo Argo CD, com
      `prune` e `selfHeal` ativos.
- [ ] Diagnosticar, sozinho, um Pod em `CrashLoopBackOff`, `Pending`, `ImagePullBackOff` e
      `OOMKilled`, usando o Apêndice C apenas como conferência.
- [ ] Explicar quando **não** usar Kubernetes e qual alternativa serverless seria mais adequada.
