# O quê preciso saber antes de começar?

## Qual distro Linux devo usar?

Devido ao fato de algumas importantes ferramentas como o systemd e o journald terem se tornado padrão, com a maioria das principais distribuições hoje disponíveis você conseguirá seguir treinamento. 

## Alguns sites que devemos visitar:

- [https://kubernetes.io](https://kubernetes.io)

- [https://github.com/kubernetes/kubernetes/](https://github.com/kubernetes/kubernetes/)

- [https://github.com/kubernetes/kubernetes/issues](https://github.com/kubernetes/kubernetes/issues)

- [https://www.cncf.io/certification/cka/](https://www.cncf.io/certification/cka/)

- [https://www.cncf.io/certification/ckad/](https://www.cncf.io/certification/ckad/)

- [https://12factor.net/pt_br/](https://12factor.net/pt_br/)

## E o k8s?

O projeto Kubernetes surgiu dentro da Google como seu orquestrador de containers, com seu design e desenvolvimento baseados no Borg e anunciado inicialmente em meados de 2014, já como um projeto *opensource*. O termo "kubernetes" em Grego significa "timoneiro", sendo que outros produtos também originaram-se do Bord, como o Apache Mesos e o Cloud Foundry.

Como Kubernetes é uma palavra difícil de se pronunciar - e de se escrever - a comunidade simplesmente o apelidou de **k8s** (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".

## Arquitetura do k8s

Assim como os demais orquestradores disponíveis, o k8s também segue um modelo *master/slave*, constituindo assim um *cluster*, onde para seu funcionamento devem existir no mínimo três nós: o nó master, responsável por padrão apenas pelo gerenciamento do *cluster*, e os demais como *workers*, executores das aplicações que nós queremos executar sobre esse cluster.

Embora exista a exigência de no mínimo três nós para a execução do k8s em um ambiente padrão, existem distribuições do k8s para um único nó. Exemplos são:

- [Minikube](https://github.com/kubernetes/minikube): Muito utilizado para implementar um cluster Kubernetes localmente para fins de desenvolvimento, testes e didáticos e que não deve ser utilizado para produção;

- [MicroK8s](https://microk8s.io): Desenvolvido pela Canonical, mesma empresa que desenvolve o Ubuntu, pode ser utilizada em diversas distribuições e tem como público algo desenvolvedores e profissionais de DevOps, podendo ser utilizada para ambientes de produção, em especial para *Edge Computing* e IoT;

- [k3s](https://k3s.io): Desenvolvido pela Rancher Labs, é um concorrente direto do MicroK8s, podendo ser executado inclusive em Raspberry Pi.

Abaixo um diagrama que mostra a arquitetura do k8s:

| ![Arquitetura Kubernetes](https://upload.wikimedia.org/wikipedia/commons/b/be/Kubernetes.png)|
|:--:| 
| *Arquitetura Kubernetes* |

##
- **API Server**: É um dos principais componentes do k8s. Ele quem fornece uma API que utiliza JSON sobre HTTP para comunicação principalmente utilizando o utilitário ```kubectl``` por parte dos administradores e para a comunicação entre os demais nós, conforme mostrado no gráfico, por meio de requisições REST;

- **etcd**: O etcd é um *datastore* chave-valor distribuído que o k8s utiliza para armazenar o status e as configurações do *cluster*. Todos os dados armazenados dentro do etcd são manipulados apenas através da API;

- **Scheduler**: É o *scheduler* quem selecionará em qual nó um determinado pod (a menor unidade de um *cluster* k8s - não se preocupe sobre isso por enquanto, nós falaremos mais sobre isso mais tarde) será executado, baseado na quantidade de recursos disponíveis, além de sabe o estado de cada um dos nós do *cluster*, garantindo que os recursos estejam bem distribuídos, baseando-se também em políticas definidas pelo usuário como por afinidade, localização de dados a serem lidos pelas aplicações, etc;

- **Controller Manager**: É o *controller manager* que garante que o cluster esteja no último estado definido na base de dados presente no etcd. Por exemplo: se no etcd um *deploy* está setado para possuir 10 réplicas em execução, é o *controller manager* quem irá verificar se o estado atual do cluster corresponde, se essas 10 réplicas estão de fato rodando corretamente e, em caso negativo, comunicará-se com a API do k8s para tal;

- **Kubelet**: O *kubelet* pode ser visto como o agente do k8s eecutado nos nós workers. É *ele* o responsável por de fato iniciar, parar, e manter os containers e os *pods* dentro do nós, direcionados pelo *controller* do *cluster*;

- **Kube-proxy**: Age como um *proxy* e um *load balancer*, efetuando o roteamento para o *pod* correto, cuidando de parte da rede do nó;

- **Container runtime**: O *container runtime* é o ambiente de execução de containers necessário para o funcionamento do k8s. Em 2016 suporte ao [rkt](https://coreos.com/rkt/) foi adicionado, porém desde o início o Docker já é funcional.

## Portas que devemos nos preocupar

**MASTER**

- API Server: 6443 TCP

- etcd: 2379-2380 TCP

- Kubelet: 10250 TCP, 10255 TCP

- Scheduler: 10251 TCP

- Controller Manager: 10252 TCP

- NodePort Services: 30000-32767 TCP

**WORKERS**

- Kubelet: 10250 TCP, 12255 TCP

- NodePort Services: 30000-32767 TCP

Você você opte pelo [Weave](https://weave.works) como *pod network*, devem ser liberadas também as portas 6783 e 6784 TCP.

## Tá, mas qual tipo de aplicação eu devo rodar sobre o k8s?

O melhor *app* para rodar em container, principalmente no k8s, são aplicações que seguem o [The Twelve-Factor App](https://12factor.net/pt_br/).

## Conceitos-chave do k8s

É importante saber que a forma como o k8s gerencia containers é ligeiramente diferente de outros orquestradores, como o Docker Swarm, sobretudo devido ao fato de que ele não trata os containers diretamente, mas sim através de *pods*. Vamos conhecer alguns dos principais conceitos que envolvem o k8s abaixo:

- **Pod**: O *pod* é o menor objeto do k8s. Como dito anteriormente, o k8s não trabalha com os containers diretamente, mas os organiza dentro de *pods*, que são abstrações que dividem os mesmos recursos, como endereços, ciclos de CPU e memória. Um pod, embora não seja comum, pode possuir vários containers;

- **Controller**: Um *controller* é o objeto responsável por interagir com o *API Server* e orquestrar algum outro objeto. Exemplos de objetos desta classe são *Deployments* e *Replication Controllers*;

- **ReplicaSets**: Um *ReplicaSet* é um objeto responsável por garantir a quantidade de pods em execução no nó;

- **Deployment**: É um dos principais *controllers* utilizados, o *Deployment* garante que um determinado número de réplicas de um *pod* através de um outro controller chamado *ReplicaSet* esteja em execução nos nós *workers* do *cluster*;

- **Jobs e CronJobs**: Responsáveis pelo gerenciamento de tarefas isoladas ou recorrentes.

# Minikube

## Requisitos básicos

É importante frisar que o Minikube deve ser instalado localmente, e não em um *cloud provider*. Por isso, as especificações de hardware abaixo são referentes à máquina local. 

## Instalação do Minikube no Linux

Antes de mais nada, verifique se a sua máquina suporta virtualização. No Linux, isto pode ser realizado com:

```
# grep -E --color 'vmx|svm
```

Caso a saída do comando não seja vazia, o resultado é positivo.

Após isso, vamos instalar o kubectl com os comandos:

```
# curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
# chmod +x ./kubectl
# sudo mv ./kubectl /usr/local/bin/kubectl
# kubectl version --client
```

Há a possibilidade de não utilizar um *hypervisor* para a instalação do Minikube, executando-o ao invés disso sobre o próprio host. Iremos utilizar o Oracle VirtualBox como *hypervisor*, que pode ser encontrado [aqui](https://www.virtualbox.org).

Efetue o download e a instalação do Minikube utilizando o comando abaixo:

```
# curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube
```

## Instalação do Minikube no macOS

No macOS, o comando para verificar se o processador suporte virtualização é:

```
# sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
```

Se você visualizar `VMX` na saída, o resultado é positivo.

O kubectl pode ser instalado no macOS utilizando tanto o [Homebrew](https://brew.sh), quanto o método tradicional. Com o Homebrew já instalado, o kubectl pode ser instalado da seguinte forma:

```
# brew install kubectl
```

Ou:

```
# brew install kubectl-cli
```

Já com o método tradicional, a instalação pode ser realizada com os seguintes comandos:

```
# curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl"
# chmod +x ./kubectl
# sudo mv ./kubectl /usr/local/bin/kubectl
# kubectl version --client
```

Por fim, efetua-se a instalação do Minikube com um dos dois métodos abaixo, também podendo optar-se pelo Homebrew ou pelo método tradicional:

```
# brew install minikube
```

Ou:

```
# curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 && chmod +x minikube
# sudo mv minikube /usr/local/bin
```

## Instalação do Minikube no Microsoft Windows

No Microsoft Windows, você deve executar o comando `systeminfo` no prompt de comando ou no terminal. Caso você visualize a similar com a abaixo é sinal que virtualização é suportada:

```textile
Hyper-V Requirements:     VM Monitor Mode Extensions: Yes
                          Virtualization Enabled In Firmware: Yes
                          Second Level Address Translation: Yes
                          Data Execution Prevention Available: Yes
```

Usuários de Microsoft Windows: caso a linha abaixo também esteja presente, não é necessária a instalação de um *hypervisor* como o Oracle VirtualBox:

```textile
Hyper-V Hyper-V Requirements:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.:     A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```

A instalação do kubectl pode ser realizada efetuando o download e [neste](https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/windows/amd64/kubectl.exe) link. Feito isso, também deve ser realizado download e a instalação de um *hypervisor* (preferencialmente o [Oracle VirtualBox](https://www.virtualbox.org)) caso no passo anterior não tenha sido acusada a presença de um. Efetue então o download do instalador do Minikube e execute-o [aqui]([Release v1.10.0 · kubernetes/minikube · GitHub](https://github.com/kubernetes/minikube/releases/latest)).

## Iniciando, parando e excluindo o Minikube

Quando operando em conjunto com um *hypervisor*, o Minikube cria uma máqunia virtual, onde dentro dela estarão todos os componentes do k8s para execução. Para realizar a inicialização desse ambiente, execute o comando:

```
# minikube start
```

Caso deseje parar o ambiente:

```
# minikube stop
```

Para excluir o ambiente:

```
# minikube delete
```

## Certo, e como eu sei que está tudo funcionando como deveria?

Uma vez iniciado, você deve ter uma saída na tela similar à seguinte:

```
# minikube start
🎉  minikube 1.10.0 is available! Download it: https://github.com/kubernetes/minikube/releases/tag/v1.10.0
💡  To disable this notice, run: 'minikube config set WantUpdateNotification false'

🙄  minikube v1.9.2 on Darwin 10.11
✨  Using the virtualbox driver based on existing profile
👍  Starting control plane node m01 in cluster minikube
🔄  Restarting existing virtualbox VM for "minikube" ...
🐳  Preparing Kubernetes v1.18.0 on Docker 19.03.8 ...
🌟  Enabling addons: default-storageclass, storage-provisioner
🏄  Done! kubectl is now configured to use "minikube"
```

Você pode então listar os nós que fazem parte do seu *cluster* k8s com o seguinte comando:

```
# kubectl get nodes
```

A saída será similar ao conteúdo abaixo:

```
# kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   8d    v1.18.0
```

Claramente, como a intenção do Minikube é executar o k8s em apenas um nó, é natural que seja apresentado apenas uma linha na saída do comando acima.

Caso os comandos acima sejam executados sem erro, a instalação do Minikube foi realizada com sucesso.

## Descobrindo o endereço do Minikube

Como dito anteriormente, o Minikube irá criar uma máquina virtual, assim como o ambiente para a execução do k8s localmente. Ele também irá configurar o kubectl para comunicar-se com o Minikube. Para saber qual é o endereço IP dessa máquina virtual, pode-se executar:

```
# minikube ip
```

O endereço apresentado é o qual deve ser utilizado para comunicação com o k8s.

## Acessando a máquina do Minikube via SSH

Para acessar a máquina virtual criada pelo Minikube, pode-se executar:

```
# minikube ssh
```

## Dashboard

O Minikube vem com um dashboard web interessante para que o usuário iniciante observe como funcionam os *workloads* sobre o k8s. Para habilitá-lo, o usuário pode digitar:

```
# minikube dashboard
```

## Logs

Os logs do Minikube podem ser acessados através do comando:

```
# minikube logs
```

# Instalação em cluster com três nós

## Requisitos básicos

Como já dito anteriormente, o Minikube é ótimo para desenvolvedores, estudos, testes, mas não tem como propósito a execução em ambiente de produção. Dito isso, a instalação de um cluster k8s para o treinamento irá requerer pelo menos três máquinas, físicas ou virtuais, cada qual com no mínimo a seguinte configuração:

- Distribuição: Debian, Ubuntu, CentOS, Red Hat, Fedora, SuSE;

- Processamento: 2 *cores*;

- Memória: 2GB

## Configuração de módulos de kernel

*Todos os comandos desta seção devem ser realizados como superusuário, salvo se dito o contrário.*

O k8s requer que certos módulos do kernel Linux estejam carregados para seu pleno funcionamento, e que esses módulos sejam carregados no momento da inicialização da máquina. Para tanto, crie o arquivo ```/etc/modules-load.d/k8s.conf```com o seguinte conteúdo em todos os seus nós:

```textile
br_netfilter
ip_vs
ip_vs_rr
ip_vs_sh
ip_vs_wrr
nf_conntrack_ipv4
```

## Atualização da distribuição

*Todos os comandos desta seção devem ser realizados como superusuário, salvo se dito o contrário.*

Em distribuições Debian e baseadas, como o Ubuntu execute o comando abaixo para atualizar a mesma em todos os seus nós:

```
# apt update && apt upgrade -y
```

Em distribuições Red Hat e baseadas:

```bash
# yum upgrade -y
```

## Instalação do Docker e do Kubernetes

A instalação do Docker pode ser realizada com apenas um comando, que deve ser realizado nos três nós:

```
# curl -fsSL https://get.docker.com | bash
```

O próximo passo é efetuar a adição dos repositórios do k8s e efetuar a instalação do kubeadm. Em distribuições Debian e baseadas, isso pode ser realizado com os comandos abaixo:

```
# apt-get update && apt-get install -y apt-transport-https
# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
# echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
# apt-get update
# apt-get install -y kubelet kubeadm kubectl
```

Já em distribuições Red Hat e baseadas, adiciona-se o repositório do k8s criando o arquivo ```/etc/yum.repos.d/kubernetes.repo``` com o conteúdo abaixo:

```textile
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
```

Os comandos abaixo desativam o firewall, instalam os pacotes do k8s e ativam o serviço do mesmo:

```
# setenforce 0
# systemctl stop firewalld
# systemctl disable firewalld
# yum install -y kubelet kubeadm kubectl
# systemctl enable kubelet && systemctl start kubelet
```

Ainda em distribuições Red Hat e baseadas, é necessário a configuração de alguns parâmetros extras no kernel por meio do sysctl. Estes podem ser setados criando o arquivo ```/etc/sysctl.d/k8s.conf``` com o seguinte conteúdo:

```textile
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

Agora, em ambas as distribuições e famílias, é muito importante verificar se o driver do cgroup utilizado pelo kubelet é o mesmo utilizado pelo Docker. Para tanto, execute:

```
# docker info | grep -i cgroup
Cgroup Driver: cgroupfs

# sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# systemctl daemon-reload
# systemctl restart kubelet
```

É necessário também desativar a swap em todos os nós com:

```
# swapoff -a
```

Além de comentar a linha referente à mesma no arquivo ```/etc/fstab```.

Após esses procedimentos, é interessante a reinicialização de todos os nós do *cluster*.

## Inicialização do cluster

Antes de inicializarmos o *cluster*, vamos efetuar o *download* das imagens que serão utilizadas, executando o comando abaixo no nó que será o *master*:

```
# kubeadm config images pull
```

Execute o comando abaixo também apenas no nó *master* para a inicialização do cluster. Caso tudo esteja bem, será apresentada ao término de sua execução o comando que deve ser executado nos demais nós para ingressar no cluster.

```
# kubeadm init --apiserver-advertise-address $(hostname -i)
```

A saída do comando será algo similar ao abaixo:

```
    [WARNING SystemVerification]: docker version is greater than the most recently validated version. Docker version: 18.05.0-ce. Max validated version: 17.03
...
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
... 
kubeadm join --token 39c341.a3bc3c4dd49758d5 IP_DO_MASTER:6443 --discovery-token-ca-cert-hash sha256:37092
...
```

## Configuração do arquivo de contextos do kubectl

Como dito anteriormente e de forma similar ao Docker Swarm, o próprio kubeadm já mostrará os comandos necessários para a configuração do kubectl de modo para que ele já se comunique com o cluster k8s. Para tanto, execute os comandos abaixo:

```
# mkdir -p $HOME/.kube
# sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## Inserindo os nós workers no cluster

Para inserir os nós *workers* no cluster, basta executar a linha que começa com ```kubeadm join``` nos mesmos.

## Instalação do pod network

Para os usuários do Docker Swarm, há uma diferença entre os dois orquestradores: o k8s por padrão não fornece uma forma para que pods e nós que estejam em nós distintos do *cluster* se comuniquem *out-of-the-box*. Para que isso ocorra, deve ser instalada uma solução de *pod networking* como *add-on*, existindo diversas opções disponíveis, cada qual com funcionalidades diferentes, como o [Flannel]([GitHub - coreos/flannel: flannel is a network fabric for containers, designed for Kubernetes](https://github.com/coreos/flannel#flannel)), o [Calico](http://docs.projectcalico.org/), [Romana](http://romana.io/), [Weave-net]([Weave Net: Network Containers Across Environments | Weaveworks](https://www.weave.works/products/weave-net/)), etc.

Mais sobre *pod networking* será tratado nos demais dias do treinamento.

Caso você ainda não tenha reiniciado os nós que compõem o seu *cluster*, você pode carregar os módulos do kernel necessários com o seguinte comando:

```
# modprobe br_netfilter ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack_ipv4 ip_vs
```

No curso, nós iremos utilizar o Weave-net, que pode ser instalado com o comando abaixo:

```
# kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
```

Verificando se o *pod network* foi criado com sucesso com o comando ```kubectl get pods -n kube-system```:

```
NAME                                READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-pfm2c            1/1     Running   0          8d
coredns-66bff467f8-s8pk4            1/1     Running   0          8d
etcd-docker-01                      1/1     Running   0          8d
kube-apiserver-docker-01            1/1     Running   0          8d
kube-controller-manager-docker-01   1/1     Running   0          8d
kube-proxy-mdcgf                    1/1     Running   0          8d
kube-proxy-q9cvf                    1/1     Running   0          8d
kube-proxy-vf8mq                    1/1     Running   0          8d
kube-scheduler-docker-01            1/1     Running   0          8d
weave-net-7dhpf                     2/2     Running   0          8d
weave-net-fvttp                     2/2     Running   0          8d
weave-net-xl7km                     2/2     Running   0          8d
```

Pode-se observar que há três containers do Weave-net em execução provendo a *pod network* para o nosso *cluster*.

## Verificando a instalação

Para verificar se a instalação está funcionando, e se os nós estão se comunicando, você pode executar o comando ```kubectl get nodes```no nó master, que deve lhe retornar algo como o conteúdo abaixo:

```
NAME        STATUS   ROLES    AGE   VERSION
docker-01   Ready    master   8d    v1.18.2
docker-02   Ready    <none>   8d    v1.18.2
docker-03   Ready    <none>   8d    v1.18.2
```

# Primeiros passos no k8s

## Exibindo informações detalhadas sobre os nós

```
# kubectl describe node [nó]
```

Exemplo:

```
# kubectl describe node docker-02
Name:               docker-02
Roles:              <none>
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=docker-02
                    kubernetes.io/os=linux
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    volumes.kubernetes.io/controller-managed-attach-detach: true
```

## Exibindo novamente token para entrar no cluster

Para visualizar novamente o token para inserção de novos nodes:

```
# kubeadm token create --print-join-command
```

## Ativando o autocomplete

Em distribuições Debian e baseadas, certifique-se que o pacote ```bash-completion``` esteja instalado. Instale-o com:

```
# apt install -y bash-completion
```

Em sistemas Red Hat e baseados, execute:

```
# yum install -y bash-completion
```

Feito isso, execute o seguinte comando:

```
# kubectl completion bash > /etc/bash_completion.d/kubectl
```

Efetue *logoff* e *login* para carregar o *autocomplete*. Caso não deseje, execute:

```
# source < (kubectl completion bash)
```

## Verificando os namespaces e pods

O k8s organiza tudo dentro de *namespaces*. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do *cluster*. Para visualizar os *namespaces* disponíveis no *cluster*, digite:

```
# kubectl get namespaces
NAME              STATUS   AGE
default           Active   8d
kube-node-lease   Active   8d
kube-public       Active   8d
kube-system       Active   8d
```

Vamos listar os *pods* do *namespace* **kube-system**:

```
# kubectl get pod -n kube-system
NAME                                READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-pfm2c            1/1     Running   0          8d
coredns-66bff467f8-s8pk4            1/1     Running   0          8d
etcd-docker-01                      1/1     Running   0          8d
kube-apiserver-docker-01            1/1     Running   0          8d
kube-controller-manager-docker-01   1/1     Running   0          8d
kube-proxy-mdcgf                    1/1     Running   0          8d
kube-proxy-q9cvf                    1/1     Running   0          8d
kube-proxy-vf8mq                    1/1     Running   0          8d
kube-scheduler-docker-01            1/1     Running   0          8d
weave-net-7dhpf                     2/2     Running   0          8d
weave-net-fvttp                     2/2     Running   0          8d
weave-net-xl7km                     2/2     Running   0          8d
```

Será que há algum *pod* escondido em algum *namespace*? É possível listar todos os *pods* de todos os *namespaces* com o comando abaixo:

```
# kubectl get pods --all-namespaces
```

Há a possibilidade, ainda, de utilizar o comando com a opção ```-o wide```, que disponibiliza maiores informações sobre o recurso, inclusive em qual nó o *pod* está sendo executado. Exemplo:

```
# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                                READY   STATUS    RESTARTS   AGE   IP             NODE        NOMINATED NODE   READINESS GATES
default       nginx                               1/1     Running   0          24m   10.44.0.1      docker-02   <none>           <none>
kube-system   coredns-66bff467f8-pfm2c            1/1     Running   0          8d    10.32.0.3      docker-01   <none>           <none>
kube-system   coredns-66bff467f8-s8pk4            1/1     Running   0          8d    10.32.0.2      docker-01   <none>           <none>
kube-system   etcd-docker-01                      1/1     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
kube-system   kube-apiserver-docker-01            1/1     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
kube-system   kube-controller-manager-docker-01   1/1     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
kube-system   kube-proxy-mdcgf                    1/1     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
kube-system   kube-proxy-q9cvf                    1/1     Running   0          8d    172.16.83.12   docker-03   <none>           <none>
kube-system   kube-proxy-vf8mq                    1/1     Running   0          8d    172.16.83.13   docker-02   <none>           <none>
kube-system   kube-scheduler-docker-01            1/1     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
kube-system   weave-net-7dhpf                     2/2     Running   0          8d    172.16.83.12   docker-03   <none>           <none>
kube-system   weave-net-fvttp                     2/2     Running   0          8d    172.16.83.13   docker-02   <none>           <none>
kube-system   weave-net-xl7km                     2/2     Running   0          8d    172.16.83.14   docker-01   <none>           <none>
```

## Executando nosso primeiro pod no k8s

Iremos iniciar o nosso primeiro *pod* no k8s. Para isso, executaremos o comando abaixo:

```
# kubectl run nginx --image nginx
pod/nginx created
```

Listando os *pods* com ```kubectl get pods```, obteremos a seguinte saída:

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          66s
```

Vamos olhar agora a descrição desse objeto dentro do *cluster*:

```
# kubectl describe pod nginx
Name:         nginx
Namespace:    default
Priority:     0
Node:         docker-02/172.16.83.13
Start Time:   Tue, 12 May 2020 02:29:38 -0300
Labels:       run=nginx
Annotations:  <none>
Status:       Running
IP:           10.44.0.1
IPs:
  IP:  10.44.0.1
Containers:
  nginx:
    Container ID:   docker://2719e2bc023944ee8f34db538094c96b24764a637574c703e232908b46b12a9f
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:86ae264c3f4acb99b2dee4d0098c40cb8c46dcf9e1148f05d3a51c4df6758c12
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Tue, 12 May 2020 02:29:42 -0300
```

## Verificar os últimos eventos do cluster

Você pode verificar quais são os últimos eventos do *cluster* com o comando ```kubectl get events```. Serão mostrados eventos como o *download* de imagens do Docker Hub (ou de outro *registry* configurado), a criação/remoção de *pods*, etc. Abaixo o resultado da criação do nosso container com Nginx:

```
LAST SEEN   TYPE     REASON      OBJECT      MESSAGE
5m34s       Normal   Scheduled   pod/nginx   Successfully assigned default/nginx to docker-02
5m33s       Normal   Pulling     pod/nginx   Pulling image "nginx"
5m31s       Normal   Pulled      pod/nginx   Successfully pulled image "nginx"
5m30s       Normal   Created     pod/nginx   Created container nginx
5m30s       Normal   Started     pod/nginx   Started container nginx
```

## Efetuar o dump de um objeto em formato YAML

Assim como quando se está trabalhando com *stacks* no Docker Swarm, normalmente recursos no k8s são declarados em arquivos **YAML** ou **JSON** e depois manipulados através do kubectl. Para nos poupar o trabalho de escrever o arquivo inteiro, pode-se utilizar como *template* o *dump* de um objeto já existente no k8s, como mostrado abaixo:

```
# kubectl get node nginx -o yaml > meu-primeiro.yaml
```

Será criado um novo arquivo chamado ```meu-primeiro.yaml```, resultante do redirecionamento da saída do comando ```kubectl get node nginx -o yaml```.

Abrindo o arquivo com ```vim meu-primeiro.yaml``` (você pode utilizar o editor que você preferir), teremos o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: "2020-05-12T05:29:38Z"
  labels:
    run: nginx
  managedFields:
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:labels:
          .: {}
          f:run: {}
      f:spec:
        f:containers:
          k:{"name":"nginx"}:
            .: {}
            f:image: {}
            f:imagePullPolicy: {}
            f:name: {}
            f:resources: {}
            f:terminationMessagePath: {}
            f:terminationMessagePolicy: {}
        f:dnsPolicy: {}
        f:enableServiceLinks: {}
        f:restartPolicy: {}
        f:schedulerName: {}
        f:securityContext: {}
        f:terminationGracePeriodSeconds: {}
    manager: kubectl
    operation: Update
    time: "2020-05-12T05:29:38Z"
  - apiVersion: v1
    fieldsType: FieldsV1
    fieldsV1:
      f:status:
        f:conditions:
          k:{"type":"ContainersReady"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Initialized"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
          k:{"type":"Ready"}:
            .: {}
            f:lastProbeTime: {}
            f:lastTransitionTime: {}
            f:status: {}
            f:type: {}
        f:containerStatuses: {}
        f:hostIP: {}
        f:phase: {}
        f:podIP: {}
        f:podIPs:
          .: {}
          k:{"ip":"10.44.0.1"}:
            .: {}
            f:ip: {}
        f:startTime: {}
    manager: kubelet
    operation: Update
    time: "2020-05-12T05:29:43Z"
  name: nginx
  namespace: default
  resourceVersion: "1673991"
  selfLink: /api/v1/namespaces/default/pods/nginx
  uid: 36506f7b-1f3b-4ee8-b063-de3e6d31bea9
spec:
  containers:
  - image: nginx
    imagePullPolicy: Always
    name: nginx
    resources: {}
    terminationMessagePath: /dev/termination-log
    terminationMessagePolicy: File
    volumeMounts:
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount
      name: default-token-nkz89
      readOnly: true
  dnsPolicy: ClusterFirst
  enableServiceLinks: true
  nodeName: docker-02
  priority: 0
  restartPolicy: Always
  schedulerName: default-scheduler
  securityContext: {}
  serviceAccount: default
  serviceAccountName: default
  terminationGracePeriodSeconds: 30
  tolerations:
  - effect: NoExecute
    key: node.kubernetes.io/not-ready
    operator: Exists
    tolerationSeconds: 300
  - effect: NoExecute
    key: node.kubernetes.io/unreachable
    operator: Exists
    tolerationSeconds: 300
  volumes:
  - name: default-token-nkz89
    secret:
      defaultMode: 420
      secretName: default-token-nkz89
status:
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2020-05-12T05:29:38Z"
    status: "True"
    type: Initialized
  - lastProbeTime: null
    lastTransitionTime: "2020-05-12T05:29:43Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: "2020-05-12T05:29:43Z"
    status: "True"
    type: ContainersReady
  - lastProbeTime: null
    lastTransitionTime: "2020-05-12T05:29:38Z"
    status: "True"
    type: PodScheduled
  containerStatuses:
  - containerID: docker://2719e2bc023944ee8f34db538094c96b24764a637574c703e232908b46b12a9f
    image: nginx:latest
    imageID: docker-pullable://nginx@sha256:86ae264c3f4acb99b2dee4d0098c40cb8c46dcf9e1148f05d3a51c4df6758c12
    lastState: {}
    name: nginx
    ready: true
    restartCount: 0
    started: true
    state:
      running:
        startedAt: "2020-05-12T05:29:42Z"
  hostIP: 172.16.83.13
  phase: Running
  podIP: 10.44.0.1
  podIPs:
  - ip: 10.44.0.1
  qosClass: BestEffort
  startTime: "2020-05-12T05:29:38Z"
```

Observando o arquivo acima, notamos que este reflete o **estado** do *pod* e que como desejamos utilizar tal arquivo apenas como um modelo, podemos apagar as entradas que armazenam dados de estado desse *pod*, como *status* e todas as demais configurações que são específicas dele. O arquivo final ficará semelhante a este:

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
```

Vamos agora remover o nosso *pod* com ```kubectl delete pod nginx```. A saída deve ser algo como:

```
pod "nginx" deleted
```

Vamos recriá-lo, agora a partir do nosso arquivo YAML:

```
# kubect create -f meu-primeiro.yaml
pod/nginx created
```

Observem que não foi necessário informar ao kubectl qual tipo de recurso seria criado, pois isso já está contido dentro do arquivo. Listando os *pods* disponíveis com ```kubectl get pods``` deve-se obter uma saída similar à esta:

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          109s
```

Uma outra forma de criar um arquivo de *template* é através da opção ```--dry-run``` do kubectl, com o funcionamento ligeiramente diferente dependendo do tipo de recurso que será criado. Exemplos:

Para a criação de um template de um *pod:*

```
# kubectl run meu-nginx --image nginx --dry-run=client -o yaml > pod-template.yaml
```

Para a criação de um template de um *deployment*:

```
# kubectl create deployment meu-nginx --image=nginx --dry-run=client -o yaml > deployment-template.yaml
```

A vantagem deste método é que não há a necessidade de limpar o arquivo, além de serem apresentadas apenas as opções necessárias do recurso.

## Socorro, são muitas opções!

Calma, nós sabemos. Mas o kubectl pode lhe auxiliar um pouco em relação a isso. Ele contém a opção ```explain```, que você pode utilizar caso precise de ajuda com alguma opção em específico dos arquivos de recurso. Abaixo alguns exemplos de sintaxe:

```
# kubectl explain [recurso]
# kubectl explain [recurso.caminho.para.spec]
# kubectl explain [recurso.caminho.para.spec] --recursive
```

Exemplos:

```
# kubectl explain deployment
# kubectl explain pod --recursive
# kubectl explain deployment.spect.template.spec
```

## Expondo o pod

Dispositivos fora do *cluster* por padrão não conseguem acessar os *pods* criados, como é comum em outros sistemas de containers. Para expor um *pod*, execute o comando abaixo:

```
# kubectl expose pod nginx
```

Será apresentada a seguinte mensagem de erro:

```
error: couldn't find port via --port flag or introspection
See 'kubectl expose -h' for help and examples
```

O motivo é devido ao fato de que o k8s não sabe qual é a porta destino do container que deve ser exposa (no caso, a 80/TCP). Para configurá-la, vamos primeiramente remover o nosso *pod* antigo:

```
# kubectl delete -f meu-primeiro.yaml
```

Abra agora o arquivo meu-primeiro.yaml e adicione o bloco abaixo:

```yaml
...
spec:
       containers:
       - image: nginx
         imagePullPolicy: Always
         ports: 
         - containerPort: 80
         name: nginx
         resources: {}
...
```

Atenção: arquivos YAML utilizam para sua tabulação dois espaços e não tab.

Feita a modificação no arquivo, salve-o e crie novamente o *pod* com o comando abaixo:

```
# kubectl create -f meu-primeiro.yaml
pod/nginx created

# kubectl get pod nginx
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          32s

# kubectl expose pod nginx
service/nginx exposed
```

O comando ```kubectl expose pod nginx``` cria um elemento do k8s chamado *Service*, utilizado justamente para expor *pods* para o mundo externo. Podemos listar todos os *services* com o comando abaixo:

```
# kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   8d
nginx        ClusterIP   10.105.41.192   <none>        80/TCP    2m30s
```

Pode é possível observar, há dois *services* no nosso *cluster*: o primeiro é para uso do próprio k8s, enquanto o segundo foi o quê acabamos de criar. Utilizando o ```curl```contra a o endereço IP mostrado na coluna *CLUSTER-IP*, deve nos ser apresentada a tela principal do Nginx:

```
# curl 10.105.41.192
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

Este *pod* está disponível para acesso a partir de qualquer nó do *cluster*.

## Limpando tudo e indo para casa

Para mostrar todos os recursos recém criados, pode-se utilizar uma das seguintes opções abaixo:

```
# kubectl get all
# kubectl get pod,service
# kubectl get pod,svc
```

Note que o k8s nos disponibiliza algumas abreviações de seus recursos. Com o tempo você irá se familiar com elas. Para apagar os recursos criados, você pode executar os seguintes comandos:

```
# kubectl delete -f meu-primeiro.yaml
# kubectl delete service nginx
```

Liste novamente os recursos para ver se os mesmos ainda estão presentes.