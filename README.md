
# 101 - Kubernetes (Solutions)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Préambule](#pr%C3%A9ambule)
  - [Installation de kubectl](#installation-de-kubectl)
- [Etape 1 : Namespace](#etape-1--namespace)
  - [Exercice 1](#exercice-1)
  - [Exercice 2](#exercice-2)
- [Etape 2 : pod](#etape-2--pod)
  - [Exercice 1 : création](#exercice-1--cr%C3%A9ation)
  - [Exercice 2 : interaction](#exercice-2--interaction)
  - [Exercice 3 : création en ligne de commande](#exercice-3--cr%C3%A9ation-en-ligne-de-commande)
- [Etape 3 : deployment](#etape-3--deployment)
  - [Exercice 1: redis](#exercice-1-redis)
  - [Exercice 2: web](#exercice-2-web)
  - [Exercice 3 : cli](#exercice-3--cli)
- [Etape 4 : service](#etape-4--service)
  - [Exercice 1 : redis](#exercice-1--redis)
  - [Exercice 2 : web](#exercice-2--web)
- [Etape 5 : ingress](#etape-5--ingress)
  - [Exercice 1 : host](#exercice-1--host)
  - [Exercice 2 : path](#exercice-2--path)
- [Etape 6 : application déploiement Canary](#etape-6--application-d%C3%A9ploiement-canary)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Préambule
Quelques liens utiles:
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://www.gitbook.com/book/ramitsurana/awesome-kubernetes/details

### Installation de kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl/

Déchiffre la configuration de kubectl
```
gpg --output kubeconfig --decrypt kubeconfig.enc
```

Copie la configuration de kubeconfig dans ~/.kube/config (à éditer si elle existe déjà)


## Etape 1 : Namespace

### Exercice 1

> Crée ton namespace

<details><summary>Solution</summary>
<p>

```
kubectl create ns tau
```

ou

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: tau
EOF
```

</p>
</details>
<br />

**Question**: Quelles resources sont présentes dans le namespace ?

<details><summary>Réponse</summary>
<p>

Le serviceaccount "default". L'important ici est de bien noter l'option `--namespace` qui permet
de préciser le namespace avec lequel on travaille.

</p>
</details>
<br />

Il est possible de changer le namespace par défaut en éditant le fichier `~/.kube/config` ou en utilisant un outil 
comme [kubens](https://github.com/ahmetb/kubectx).

### Exercice 2

> Change de namespace par défaut pour utiliser le tiens

<details><summary>Solution</summary>
<p>

```
vi ~/.kube/config
```

```
contexts:
- context:
    cluster: xke.techx.fr
    user: xke.techx.fr
    namespace: changeme
  name: xke.techx.fr
```

</p>
</details>
<br />

L'outil https://github.com/ahmetb/kubectx est très pratique pour changer de namespace et de contexte.

## Etape 2 : pod

### Exercice 1 : création

> Crée un pod à partir d'un manifest yaml composé de:
> - un conteneur nginx
> - un conteneur tauffredou/quote-logger

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex1.1/pod.yaml
```

</p>
</details>
<br />

**Question**: Quel est l'ip du pod?

<details><summary>Réponse 1</summary>
<p>

```
kubectl get po exo1.1 -o wide
```

</p>
</details>
<br />

<details><summary>Réponse 2</summary>
<p>

```
kubectl get po exo1.1 -o jsonpath='{.status.podIP}'
```

</p>
</details>
<br />

**Question**: Comment joindre le serveur nginx depuis quote-logger ? Depuis un autre pod ?

<details><summary>Réponse</summary>
<p>

```
kubectl exec exo1.1 -c quote-logger curl localhost
```

</p>
</details>
<br />

### Exercice 2 : interaction

> Affiche les logs du quote-logger dans le pod

<details><summary>Solution</summary>
<p>

```
kubectl logs exo1.1 -c quote-logger
```

</p>
</details>
<br />

> Entre dans le conteneur nginx pour afficher les variables d'environement

<details><summary>Solution</summary>
<p>

```
kubectl exec exo1.1 -c nginx env
```

</p>
</details>
<br />

> Supprime le pod à l'aide de la commande `kubectl delete`

<details><summary>Solution</summary>
<p>

```
kubectl delete pod exo1.1
```

</p>
</details>
<br />

**Question**: Quel est l'état du pod ?

<details><summary>Réponse</summary>
<p>

Le pod est complétement supprimé. Il n'est pas relancé. Il n'est pas redémarrable, il n'y pas d'état "stoppé".

</p>
</details>
<br />

### Exercice 3 : création en ligne de commande

Il est possible de démarrer un pod en ligne de commande avec `kubectl run`

> Démarre un conteneur quote-logger en ligne de commande

<details><summary>Solution</summary>
<p>

```
kubectl run exo1dot2 --image tauffredou/quote-logger
```

</p>
</details>
<br />

**Question**: Quelles ressouces supplémentaires ont été créées ?

<details><summary>Réponse</summary>
<p>

deployment, replicatset

</p>
</details>
<br />

> Supprime le pod précédement créé

<details><summary>Solution</summary>
<p>

```
kubectl delete po exo1dot2-67455455bf-d86sw
```

</p>
</details>
<br />

**Question** : Quel est l'état du pod cette fois-ci ?

<details><summary>Réponse</summary>
<p>

Le pod est supprimé. En revanche, un autre pod identique au premier
est créé pour satisfaire les contraintes du deployment (1 réplique)

</p>
</details>
<br />

## Etape 3 : deployment

A partir de cette étape, nous allons construire l'application clickcount

### Exercice 1: redis

> Crée un manifest de deployment du composant "redis" :
> - image: redis
> - replicas: 1
> - le pod contient un "label" nommé "component" de valeur "redis"

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex2.1/deployment.yaml
```

</p>
</details>
<br />

### Exercice 2: web

> Crée un manifest de deployment du composant "web" :
> - image: tauffredou/clickcount
> - replicas: 3
> - le pod contient un "label" nommé "component" de valeur "web"
> - l'adresse du redis est définie par une variable d'environnement nommée REDIS\_HOST

**Note**: la valeur de REDIS\_HOST peut être devinée dès cette étape mais la définition sera 
abordée dans l'étape suivante (service)

**Question** : combien de pods ont été alloués lors de mon deployment ?

<details><summary>Réponse</summary>
<p>

`kubectl get deploy` (colonne `CURRENT`)

ou

`kubectl get pod`


*=> 3 pods*
</p>
</details>
<br />

> Réduire le nombre de replicas du composant "web" à 2 instances

<details><summary>Solution</summary>
<p>

```
sed -i 's/replicas: .\*$/replicas: 2/' solutions/ex3.2/deployment-web.yaml
kubectl apply -f app/ex3.2/deployment-web.yaml
```

</p>
</details>
<br />

### Exercice 3 : cli

> Retire 2 répliques au deployment via la commande scale

<details><summary>Solution</summary>
<p>

```
kubectl scale deployment exo2.1 --replicas 2
```

</p>
</details>
<br />

> Supprime le deployment

<details><summary>Solution</summary>
<p>

```
kubectl delete deployment exo2.1
```

</p>
</details>
<br />

**Question** : combien de pods reste-t-il après suppression du deployment ?

<details><summary>Réponse</summary>
<p>

```
kubectl get pod
```

=> 1 pod, celui de l'étape 2, les 2 du deployment ont été supprimés car liés à ce dernier.

</p>
</details>
<br />

> Efface tous les déploiements restants.

<details><summary>Solution</summary>
<p>

```
kubectl delete deployment --all
```

</p>
</details>
<br />

## Etape 4 : service

### Exercice 1 : redis

> Crée un service qui expose le deployment de redis

**Tips pour les plus valeureux**: Il n'y a qu'une seule instance de redis. 
Il n'est pas nécessaire d'utiliser un loadbalancer pour y accéder. 
La définition d'un service reste requis.

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex4.1/service-redis.yaml
```

</p>
</details>
<br />

> Affiche la description du service

<details><summary>Solution</summary>
<p>

```
kubectl describe service redis
```

</p>
</details>
<br />

**Question** : que remarque-t-on ?

<details><summary>Réponse</summary>
<p>

- le service crée une adresse Ip (virtuelle)
- la liste des endpoints est vide (le déploiement avait été supprimé, aucun routage possible !)

</p>
</details>
<br />

**Question**: Quelle est l'adresse du service au sein du namespace ?

### Exercice 2 : web

> Crée un service qui expose le deployment de web en testant les types de services:
> - LoadBalancer
> - NodePort
> - ClusterIp

**Question**: Sachant que les noeuds du cluster sont dans un réseau privé, quel type de service vous semble
le plus adapté pour exposer le composant "web" sur internet ?

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex4.2/service-web.yaml
```

=> utilisation de ClusterIp (option par défaut) et d'un ingress à l'étape suivante

</p>
</details>
<br />

**Question**: Quelle est l'adresse du service au sein du cluster ?

<details><summary>Réponse</summary>
<p>

```
kubectl get service web
```

ou

```
kubectl describe service web
```

</p>
</details>
<br />

## Etape 5 : ingress

### Exercice 1 : host

> Crée un ingress pour exposer le service web sur le host *<namespace>.xke.techx.fr*.

**Note**: le wildcard \*.xke.techx.fr est configuré pour pointer sur l'ingress controller du cluster kubernetes.

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex5.1/ingress.yaml
```

</p>
</details>
<br />


### Exercice 2 : path

> Crée un ingress rediriger le path */<ns>* vers le service web sur le host *all.xke.techx.fr.*

<details><summary>Solution</summary>
<p>

```
kubectl apply -f solutions/ex5.2/ingress-path.yaml
```

</p>
</details>
<br />
