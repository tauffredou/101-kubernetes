
# 101 - Kubernetes (Solutions)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Préambule](#pr%C3%A9ambule)
  - [Installation de kubectl](#installation-de-kubectl)
- [Etape 1 : Namespace](#etape-1--namespace)
  - [Exercice 1](#exercice-1)
- [Etape 2 : pod](#etape-2--pod)
  - [Exercice 1](#exercice-1-1)
  - [Exercice 2](#exercice-2)
- [Etape 3 : deployment](#etape-3--deployment)
  - [Exercice 1](#exercice-1-2)
  - [Exercice 2](#exercice-2-1)
- [Etape 4 : service](#etape-4--service)
- [Etape 5 : ingress](#etape-5--ingress)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Préambule
Quelques liens utiles:
- https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- https://www.gitbook.com/book/ramitsurana/awesome-kubernetes/details

### Installation de kubectl

https://kubernetes.io/docs/tasks/tools/install-kubectl/

## Etape 1 : Namespace

### Exercice 1

> Crée ton namespace

```
kubectl create ns tau

# ou

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Namespace
metadata:
  name: tau
EOF
```

**Question**: Quelles resources sont présentes dans le namespace ?

*Réponse*: le serviceaccount "default". L'important ici est de bien noter l'option `--namespace` qui permet
de préciser le namespace avec lequel on travaille.
Il est possible de changer le namespace par défaut en éditant le fichier `~/.kube/config` ou en utilisant un outil 
comme [kubens](https://github.com/ahmetb/kubectx).

### Exercice 2

> Change de namespace par défaut pour utiliser le tien

```vi ~/.kube/config```

## Etape 2 : pod

### Exercice 1

> Crée un pod à partir d'un manifest yaml composé de:
> - un conteneur nginx
> - un conteneur tauffredou/quote-logger

```
kubectl apply -f solutions/ex1.1/pod.yaml
```
**Question**: Quel est l'ip du pod?

*Réponse 1*: `kubectl get po exo1.1 -o wide`

*Réponse 2*: `kubectl get po exo1.1 -o jsonpath='{.status.podIP}'` 

**Question**: Comment joindre le serveur nginx depuis quote-logger ? Depuis un autre pod ?

*Réponse*:`kubectl exec exo1.1 -c quote-logger curl localhost`

> Affiche les logs du quote-logger dans le pod

```
kubectl logs exo1.1 -c quote-logger
```

> Entre dans le conteneur nginx pour afficher les variables d'environement

```
kubectl exec exo1.1 -c nginx env
```

> Supprime le pod à l'aide de la commande `kubectl delete`

```
kubectl delete pod exo1.1
```

**Question**: Quel est l'état du pod ?

*Réponse*: Le pod est complétement supprimé. Il n'est pas relancé. Il n'est pas redémarrable, il n'y pas d'état "stoppé".

### Exercice 2

Il est possible de démarrer un pod en ligne de commande avec `kubectl run`

> Démarre un conteneur quote-logger en ligne de commande

```
kubectl run exo1dot2 --image tauffredou/quote-logger
```

**Question**: Quelles ressouces supplémentaires ont été créées ?

*Réponse*: deployment, replicatset

> Supprime le pod précédement créé

```
k delete po exo1dot2-67455455bf-d86sw
```

**Question** : Quel est l'état du pod cette fois-ci ?

*Réponse* : Le pod est supprimé. En revanche, un autre pod identique au premier
est créé pour satisfaire les contraintes du deployment (1 réplique)

## Etape 3 : deployment

### Exercice 1

> Crée un deployment à partir d'un manifest yaml dont les contraintes sont :
> - déployer 3 répliques
> - ...d'un pod identique à celui défini à l'étape 1

```
kubectl apply -f solutions/ex2.1/deployment.yaml
```

**Question** : combien de pods ont été alloués lors de mon deployment ?

*Réponse* : `kubectl get deploy` (colonne `CURRENT`) / `kubectl get po`

*=> 3 pods*

> Ajoute une réplique au deployment

```
sed -i 's/replicas: .\*$/replicas: 4/' solutions/ex2.1/deployment.yaml
kubectl apply -f solutions/ex2.1/deployment.yaml
```

### Exercice 2

> Retire 2 répliques au deployment via la commande scale

```
kubectl scale deployment exo2.1 --replicas 2
```

> Supprime le deployment

```
kubectl delete deployment exo2.1
```

**Question** : combien de pods reste-t-il après suppression du deployment ?

*Réponse* : `kubectl get po`

=> 1 pod, celui de l'étape 2, les 2 du deployment ont été supprimés car liés à ce dernier.

> Efface tous les déploiements restants.

```
kubectl delete deployment --all
```

## Etape 4 : service

> Crée un service à partir d'un manifest yaml :
> - qui cible le port HTTP (TCP/80) et utilise le même port en entrée
> - ...du déploiement créé précédemment dans l'exercice 2 (exo2.1)

```
kubectl apply -f solutions/ex3/service.yaml
```

> Affiche la description du service

```
kubectl describe service exo3
```

**Question** : que remarque-t-on ?

*Réponse* :
- le service crée une adresse Ip (virtuelle)
- la liste des endpoints est vide (le déploiement avait été supprimé, aucun routage possible !)

> Applique le déploiement précédent (exo2.1) :

```
kubectl apply -f solutions/ex2.1/deployment.yaml
```

Le service a un ensemble d'endpoints sur lequels le trafic peut être routé.

## Etape 5 : ingress

Hosts:
- a.xke.com
- b.xke.com

Paths:
/a -> service a
/b -> service b
## Etape 6 : application déploiement Canary
