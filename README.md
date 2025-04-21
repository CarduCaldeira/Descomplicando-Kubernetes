# Descomplicando o Kubernetes 

Anotações para uso pessoal baseado no curso Descomplicando o Kubernetes [Linuxtips](https://linuxtips.io/).

## Comandos básicos

Para criar um cluster chamado giropops com o kind:
```bash
kind create cluster --name giropops
```

Alguns comandos básicos para trabalhar com um cluster, nodes, namespaces:
```bash
kind get clusters
kind delete clusters $(kind get clusters)

kubectl get nodes

kubectl get namespaces
kubectl create namespace name-namespace
```

Alguns comandos básicos para trabalhar com os pods:
```bash
kubectl get pods # lista os pods do namespace default
kubectl get pod -n kube-system # lista os pods do namespace kube-system
kubectl run alpine --image alpine -it # executar no modo iterativo
kubectl get pods -A # lista todos os pods
kubectl get pods -A -o wide # informações detalhadas
```

```bash
kubectl run nginx --image nginx # cria um pod chamado nginx contendo o container criado da imagem  nginx
kubectl run alpine --image apline -it # cria e executa no modo iterativo 
kubectl run nginx-test --image=nginx --namespace=prod # cria o pod no namespace especificado
kubectl delete pod nginx --namespace=meu-namespace
```

Para criar um clustar com multiplos nodes utilize um arquivo yaml como abaixo:
```bash
cat << EOF > $HOME/kind-3nodes.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
EOF
```

Execute:

```bash
kind create cluster --name kind-multinodes --config $HOME/kind-3nodes.yaml
```

Também é possível criar pods com arquivo yaml:

```bash
kubectl run meu-nginx --image nginx --port 80 --dry-run=client -o yaml > pod1-template.yaml #simula a criação de um pod e salva as espicificações do pod em um arquivo yaml

kubectl apply -f pod-template.yaml #cria o pod com a partir do arquivo yaml

kubectl expose pod name-pod 

kubectl describe pods name-pod

kubectl logs name-pod

kubectl logs -f name-pod # Para ver os logs em tempo real
```

Note que para dar o expose é necessario ter definido a porta do container (nesse caso a porta de target será a mesma que a definida em containerPort).

```bash
kubectl get services

kubectl get all
kubectl get pod,service

kubectl delete -f pod-template.yaml
kubectl delete service nginx
```

Dentro de um pod os containers terao o mesmo IP. Portanto a comunicação dentro do pod é realizado pelo localhost e entre pods é pelo IP. O comando expose serve para sevir o pod fora do cluster (por padrão um pod tem acesso ao outro).

Para criar um pod com mais de um container:

```yaml
apiVersion: v1 # versão da API do Kubernetes
kind: Pod # tipo do objeto que estamos criando
metadata: # metadados do Pod
  name: giropops # nome do Pod que estamos criando
  labels: # labels do Pod
    run: giropops # label run com o valor giropops
spec: # especificação do Pod
  containers: # containers que estão dentro do Pod
  - name: girus # nome do container
    image: nginx # imagem do container
    ports: # portas que estão sendo expostas pelo container
    - containerPort: 80 # porta 80 exposta pelo container
  - name: strigus # nome do container
    image: alpine # imagem do container
    args:
    - sleep
    - "1800"
```

Name é o identificador do Pod, enquanto run (labels) é uma label usada para categorizar ou agrupar o Pod com outras entidades, facilitando a seleção e a automação de operações.

Para se conectar ao container:

```bash
kubectl attach giropops -c strigus
```

Usando o attach é como se estivéssemos conectando diretamente em uma console
de uma máquina virtual, não estamos criando nenhum processo dentro do container,
apenas nos conectando a ele. Para executar um comando

```bash
kubectl exec giropops -c strigus -- ls
```

Para se conectar ao pod e utiliza-lo de modo iterativo adicione a flag -it.

```bash
kubectl exec giropops -c strigus -it -- sh
```

## Resources e EmptyDir

É possível configurar volumes e limitar o uso de recursos por container ao definir um pod como abaixo:


```yaml
apiVersion: v1 
kind: Pod
metadata:
  name: giropops
  labels:
    run: giropops
spec:
  containers:
  - name: girus
    image: nginx 
    ports:
    - containerPort: 80 
    resources: 
      limits:
        memory: "128Mi"
        cpu: "0.5" 
      requests: 
        memory: "64Mi" 
        cpu: "0.3"
  - name: strigus
    image: ubuntu
    args:
    - sleep
    - infinity
    volumeMounts: 
    - name: primeiro-emptydir 
      mountPath: /giropops 
    resources: 
      limits:
        memory: "128Mi"
        cpu: "0.8" 
      requests: 
        memory: "64Mi" 
        cpu: "0.3"
  volumes:
    - name: primeiro-emptydir
      emptyDir:
        sizeLimit: 256Mi 
```

Requests é referente aos recursos garantidos ao container e o limits é a quantidade máxima de recurso que o container irá usar.