
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

### Exercice 1 : création

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

### Exercice 2 : interaction

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

### Exercice 3 : création en ligne de commande

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

A partir de cette étape, nous allons construire l'application clickcount

### Exercice 1: redis

> Crée un manifest de deployment du composant "redis" :
> - image: redis
> - replicas: 1
> - le pod contient un "label" nommé "component" de valeur "redis"

```
kubectl apply -f solutions/ex3.1/deployment-redis.yaml
```

### Exercice 2: web

> Crée un manifest de deployment du composant "web" :
> - image: tauffredou/clickcount
> - replicas: 3
> - le pod contient un "label" nommé "component" de valeur "web"
> - l'adresse du redis est définie par une variable d'environnement nommée REDIS_HOST

**Note**: la valeur de REDIS_HOST peut être devinée dès cette étape mais la définition sera 
abordée dans l'étape suivante (service)

**Question** : combien de pods ont été alloués lors de mon deployment ?

*Réponse* : `kubectl get deploy` (colonne `CURRENT`) / `kubectl get po`

*=> 3 pods*

> Réduire le nombre de replicas du composant "web" à 2 instances

```
sed -i 's/replicas: .\*$/replicas: 2/' solutions/ex3.2/deployment-web.yaml
kubectl apply -f app/ex3.2/deployment-web.yaml
```

### Exercice 3 : cli

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

### Exercice 1 : redis

> Crée un service qui expose le deployment de redis

**Tips pour les plus valeureux**: Il n'y a qu'une seule instance de redis. 
Il n'est pas nécessaire d'utiliser un loadbalancer pour y accéder. 
La définition d'un service reste requis.

```
kubectl apply -f solutions/ex4.1/service-redis.yaml
```

> Affiche la description du service

```
kubectl describe service redis
```

**Question** : que remarque-t-on ?

*Réponse* :
- le service crée une adresse Ip (virtuelle)
- la liste des endpoints est vide (le déploiement avait été supprimé, aucun routage possible !)

**Question**: Quelle est l'adresse du service au sein du namespace ?

### Exercice 2 : web

> Crée un service qui expose le deployment de web en testant les types de services:
> - LoadBalancer
> - NodePort
> - ClusterIp

**Question**: Sachant que les noeuds du cluster sont dans un réseau privé, quel type de service vous semble
le plus adapté pour exposer le composant "web" sur internet ?

```
kubectl apply -f solutions/ex4.2/service-web.yaml
```

**Question**: Quelle est l'adresse du service au sein du namespace ?

## Etape 5 : ingress

### Exercice 1 : host

> Crée un ingress pour exposer le service web sur le host *<namespace>.xke.techx.fr*.

**Note**: le wildcard \*.xke.techx.fr est configuré pour pointer sur l'ingress controller du cluster kubernetes.

### Exercice 2 : path

> Crée un ingress rediriger le path */<ns>* vers le service web sur le host *all.xke.techx.fr.*

## Etape 6 : application au déploiement Canary

*...pour les plus téméraires.*


L'association Big-l'eux a demandé l'hébergement d'un compteur de clicks pour un événement qui aura lieu dans deux semaines.

> Livre une première version du compteur :
> - supprime tout ce qui concerne le déploiement web (deployment, service, ingress clickcount) des étapes précédentes, tout en conservant ce qui concerne le redis ;
> - redéploie la partie web identique à la précédente excepté :
>  - le deployment utilise un label *track* avec la valeur *stable* et donne vie à 4 répliques,
>  - le service cible uniquement le deployment avec le label *track* à *stable*.

```
kubectl delete deployment web
kubectl delete service web
kubectl delete ingress clickcount
kubectl apply -f solutions/ex6/deployment-web-stable.yaml
kubectl apply -f solutions/ex6/service-web-prod.yaml
kubectl apply -f solutions/ex6/ingress-prod.yaml
```

Après quelques essais, l'association est plutôt satisfaite du résultat mais la majorité des utilisateurs ont du mal à distinguer le bouton et les écritures. L'association demande une deuxième version plus contrastée et avec des écritures et bouton plus gros.

Afin de ne pas perturber l'utilisation, nous allons créer un environnement de staging pour valider faire valider nos développements au président de l'association. Un environnement de préproduction sera ensuite implémenté pour un déploiement mixte des 2 versions. L'URL de préproduction sera donnée à un panel des membres qui pourront comparer les 2 versions sans risquer une rupture de service si la 2e version présente des régressions.

> En se basant sur le déploiement de la première version (deployment, service, ingress clickcount), déploie une version de staging :
> - l'image à utiliser est *clook/clickcount:2*, 2 répliques sont suffisantes ;
> - le label *track* prend maintenant la valeur *canary* ;
> - l'ingress utilise l'URL cible *staging.<namespace>.xke.techx.fr*.

```
kubectl apply -f solution/ex6/deployment-web-canary.yaml
kubectl apply -f solution/ex6/service-web-staging.yaml
kubectl apply -f solution/ex6/ingress-staging.yaml
```

> Déploie maintenant l'environnement de préproduction qui «mélange» les 2 versions (ratio prod:staging = 2:1).

```
kubectl apply -f solution/ex6/service-web-preprod.yaml
kubectl apply -f solution/ex6/ingress-preprod.yaml
```

Les membres qui ont utilisé le rang de préproduction et le président de l'association ont donné leur aval pour cette deuxième version et souhaiteraient la voir en production pour tous les membres.

> Déploie la deuxième version en production et supprime les environnements de staging et préproduction.

```
sed 's@tauffredou/clickcount@clook/clickcount:2@' solution/ex6/deployment-web-stable.yaml
kubectl apply -f solution/ex6/deployment-web-stable.yaml
kubectl delete ingress clickcount-preprod clickcount-staging
kubectl delete service web-preprod web-staging
kubectl delete deployment web-canary
```
