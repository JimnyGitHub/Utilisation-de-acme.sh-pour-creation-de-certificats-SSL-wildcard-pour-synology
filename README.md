# Utilisation de acme.sh pour création de certificats SSL wildcard pour synology

ACME veut dire -> Automatic Certificate Management Environment 

https://datatracker.ietf.org/doc/rfc8555/

## lien officiel

https://github.com/acmesh-official/acme.sh

## 1) installation du script acme.sh (a ne faire que une fois)

### prérequis

- se connecter en ssh sur son syno
- creer un repertoire pour recuperation de l'installeur

```
mkdir ~/temporaire
```

- creer un repertoire dans lequel va etre installer le script

```
mkdir -p ~/opt/acme.sh/
```

- creer un repertoire dans lequel va etre installé la configuration et
les données de acme.sh

(/acme.sh va écrire tous les fichiers, y compris cert/keys, configs)

```
mkdir -p ~/opt/acme.sh/data/
```

### telechargement du script

```
cd ~/temporaire
git clone https://github.com/acmesh-official/acme.sh.git
```

### premier lancement du script pour finaliser l'installation

```
cd ~/temporaire/acme.sh
./acme.sh --install -m nom.prenom@mail.org --nocron --home ~/opt/acme.sh/ --config-home ~/opt/acme.sh/data/
```

il faut mettre l'option --nocron sur le syno car Synology n'a pas de
planificateur en cours d'exécution qui est pris en charge par acme.sh

### prise en compte de la nouvelle installation

il faut quitter et ouvrir un nouveau SSH pour prise en compte de l'installation

## 2) configuration du script

### configuration pour que le script se mette a jour automatiquement

```
cd ~/opt/acme.sh/
./acme.sh --upgrade --auto-upgrade --nocron --home ~/opt/acme.sh/ --config-home ~/opt/acme.sh/data/
```

### verification de la version en cours

```
cd ~/opt/acme.sh/
./acme.sh --version
```

### mise a jour manuelle de acme.sh

```
cd ~/opt/acme.sh/
./acme.sh --version
./acme.sh --upgrade
./acme.sh --version
```

## Création d'un challenge DNS pour validation de domaine

Les challenges DNS sont des méthodes qui consistent à vérifier que vous êtes réellement le propriétaire d'un nom de domaine.
Il en existe de plusieurs types

### Cas de l'émission de certificats wildcard. (*.mondomaine.org)

Pour que Let's Encrypt émette un certificat wildcard, il faut utiliser un 'challenge' DNS connu sous le nom de Validation de domaine (DV).

Acme.sh s'intègre aux API de nombreux fournisseurs DNS majeurs et automatise complètement ce processus.

#### documentation

ici https://github.com/acmesh-official/acme.sh/wiki/dnsapi

### Utilisation du Dns OVH

#### documentation specifique Dns Ovh

ici https://github.com/acmesh-official/acme.sh/wiki/How-to-use-OVH-domain-api

à suivre pas à pas

#### Creation clef et secret

a été fait via https://eu.api.ovh.com/createApp/ comme indiqué dans la doc ci-dessus


##### exemple de config spécifique ovh en sortie de la création de la clef et du secret

Application name : application_acme_sh

Application description : pour usage de acme.sh sur le syno

Application Key : xxxxxxxxxxxxxxxx

Application Secret : zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz

##### export des variables avec clef et secret pour le Dns Ovh

variable pour la clef :

export OVH_AK#"xxxxxxxxxxxxxxxx"

variable pour le secret :

export OVH_AS#"zzzzzzzzzzzzzzzzzzzzzzzzzzzzzzzz"

## demander un nouveau certificat pour tout les sous domaines possibles

Pour générer réellement les certificats 

il faut lancer la ligne de commande suivante :

```
./acme.sh --issue -d *.mondomaine.org --dns dns_ovh --server letsencrypt
```

En sortie de cela, les certificats sont générés

## intallation des scripts sur le synology

### prerequis -> copier les certificats sur son pc local 

depuis le syno avec "File station"

ouvrir

```
/homes/monNomDeUser/opt/.acme.sh/
```

selection de "*.mondomaine.org" > télécharger le zip mondomaine.org.zip sur le disque du PC

### Decompresser l'archive des certificats

decompression de l'archive mondomaine.org.zip dans un répertoire du pc

### mise en place des nouveaux scripts sur le syno

Panneau de config

connectivité -> Securité

Certificat

(Sélection du certificat expiré puis)  -> Ajouter

(Remplacer un cerficat existant) OU Ajouter un nouveau certificat

Importer le certificat

#### depuis le répertoire du PC ou a été décompressée l'archive des cerficats

Donner les chemins

* de la Cle privée : fichier monDomaine.org.key
* du certificat : fichier monDomaine.org.cer
* du certificat intermédiaire  : fichier ca.cer

## Mise à jour des certificats

Pour remettre à jour les cerficats lorsqu'ils arrivent à expiration (au bout de 3 mois)

il faut suivre à nouveau les points depuis 

***demander un nouveau certificat pour tout les sous domaines possibles***
