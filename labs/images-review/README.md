## Instructions 
|                      |                                                                         |
| -------------------- | ----------------------------------------------------------------------- |
| Application name     | custom-server                                                           |
| Image registry       | [registry.ocp4.example.com:8443](http://registry.ocp4.example.com:8443) |
| Image name           | custom-server:1.0.0                                                     |
| Registry username    | developer                                                               |
| Registry password    | developer                                                               |
| Registry email       | [developer@example.org](mailto:developer@example.org)                   |
| Registry secret name | registry-credentials                                                    |

Accédez à l'emplacement du conteneur fourni par l'exercice
```
$ cd ~/DO288/labs/images-review
```

## Créez et Publiez une image de conteneur 

### Avec Podman

Examiner le contenu du fichier conteneur
```
$ cat Containerfile
```

Connectez-vous au dépôt des conteneurs
```
$ podman login -u developer -p developer registry.ocp4.example.com:8443
```

Créez l’image du conteneur
```
$ podman build . -t registry.ocp4.example.com:8443/developer/custom-server:1.0.0
```



Répertorier les images de conteneurs locaux
```
$ podman images
```

Répertorier les conteneurs en cours d'exécution
```
$ podman ps
```

Exécution  de l’image de conteneur et ouvre un shell à l'intérieur du conteneur
```
$ podman run -it  registry.ocp4.example.com:8443/developer/custom-server:1.0.0  /bin/bash
```

Transférer l'image du conteneur vers le dépôt
```
$ podman push registry.ocp4.example.com:8443/developer/custom-server:1.0.0
```

Supprimer l’image a partir de son nom en local
```
$ podman rmi registry.ocp4.example.com:8443/developer/custom-server:1.0.0
```

### Avec Buildah

Créez l’image du conteneur
```
$ buildah bud -t  registry.ocp4.example.com:8443/developer/custom-server:1.0.1 .
```

Répertorier les conteneurs en cours d'exécution 
```
$ buildah containers 
```

Transférer l'image du conteneur vers le dépôt
```
$ buildah push registry.ocp4.example.com:8443/developer/custom-server:1.0.1
```

### Avec S2I

Construire à partir du répertoire local avec l'image de base __vrutkovs/golang-s2i__
```
$ s2i build . vrutkovs/golang-s2i registry.ocp4.example.com:8443/developer/custom-server:1.0.2
```

## Créez un image stream à partir d'une image de conteneur 

Connectez-vous à Red Hat OpenShif
```
$ oc login -u developer -p developer https://api.ocp4.example.com:6443
```

Assurez-vous que vous êtes dans le projet images-review

```
$ oc project images-review
```

Créez le secret __registry-credentials__ du dépôt  de  type __docker-registry__ qui contient les informations d'identification
```
$ oc create secret docker-registry registry-credentials 
--docker-server=registry.ocp4.example.com:8443 \
--docker-username=developer --docker-password=developer \
--docker-email=developer@example.org
```


Associer le secret au service account  par défaut
```
$ oc secrets link default registry-credentials --for=pull
```

Créez un image stream  custom-server  qui pointe vers l'image de conteneur que vous avez transférée vers le dépôt
```
$ oc import-image custom-server --confirm --from registry.ocp4.example.com:8443/developer/custom-server:1.0.0
```

## Créez une application à l'aide d'un nouvel image stream

Déployer une application à l'aide de l’image streams
```
$ oc new-app --name custom-server -i images-review/custom-server
```


Attendez que le module d'application soit prêt et exécuté
```
$ oc get pod
```

Créer une route pour exposer l'application
```
$ oc expose svc custom-server
```

Récupérer  le nom d'hôte de la route 
```
$ oc get route
```

Testez l'application 
```
$ curl http://custom-server-images-review.apps.ocp4.example.com
```

