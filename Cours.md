MangoDb stock les données au format json


Une collection est groupe de documents MangoDb. C'est l'équivalent d'une table dans un système de gestion de base combinées.
Les documents présents peuvent avoir des champs différents

Un document est un ensemble de données clé-valeur

table = collection

colonne = fields

Pas de schéma

Pas de jointure complexe

Tres gros jeu de données MangoDB => BEST

Compass permet de communiquer avec la BDD

Taille max de stockage 16Mo par fichier JSON

Ceci represente en JSON 
```JSON
{} = objet vide

{
	key: value
}
```

Le nom des clés d'un objet doivent être unique

Un objet JSON peut renvouer un objet null si il n'a pas de valeur

MongoDB vient ajouter des types:
 - Date : stocke sous la forme d'un entier signé de 8 octets
les secondes sont égale au nombre de secondes écoulées depuis l'époque d'Unix (01/01/1970 à minuit)
 - ObjectId : stock sur 12 octets, utilisé en interne par MongoDB afin de generer des identifiants
 - NumberLong  / NumberInt : par défaut, MongoDB considere toute valeur numérique comme un nombre à virgule code sur 8 octets. Representent des entiers signes sur 8 et 4 octets. 
 - BinData : pour stocker des chaines de caractères ne possedant pas de representation dans l'encodage UTF-8, ou n'importe 


### MongoDB Compass :

- Pour switch de BDD
``` bash
use maBDD
```

- Ajouter un user
```bash
db.createUser({"user": "superUser", "pwd": "password", "roles": [{"roles": "userAdminAnyDatabase", "db": "admin", "readWriteAnyDatabase"]})
```

- Création d'une  base de données
```bash
db.maCollection.insertOne({
	"une_cle": "une_valeur"
})
```

- Supprimer une BDD
```bash
use laBDDaSupprimer

db.dropDatabase();
```

#### Les helpers

```bash
db.runCommand({"collstats": "uneCollection"})

db.adminCommand("currentOp")
```

### Gestion des collections

#### Les collations

Une collation est un objet est une régle à établir qui sera spécifique à une langue

Exemple :

on peut mettre une collation pour toucher que les beret vert en France

```javascript
{
	**locale**: <string>,
	caseLevel: <boolean>,
	caseFirst: <string>,
	strength: <int>,
	numericOrdering: <boolean>,
	....
}
```

Au sein de ce type de document le champ locale est obligatoire

#### Creer une collection

```javascript
db.createCollection("maCollection", {"collation": {"locale": "fr"} })
```


### Les documents

#### Insertion d'un document

```javascript
db.maCollection.insert(< document ou un tableau de documents >)

db.personnes.insert([
	{"nom": "Mouranchon", "prenom": "Jean"},
	{"nom": "Lao", "prenom": "Kenji", "age": 20}
])	
```

Deprecated => fonctionne mais va etre enlever dans une prochaine mise à jour donc non recommander de l'utiliser

#### Modification d'un document

```javascript
db.collection.updateOne(<filtre>, <modifications>)

// Mauvaise pratique (ca va marcher mais forte chance de tout casser)
db.personnes.update(
	{"nom": "dupont"},
	{
		$set : { "nom": "Dupont" }
	}
)

// Bonne pratique
db.personnes.update(
	{"_id": ObjectId("dupont")},
	{
		$set : { "nom": "Dupont" }
	}
)
```

ATTENTION !!!! MangoDB est sensible à la casse

Pour bien modifier un document il faut selectionner son id

Cas de l'upsert : la combinaison d'update et d'insert

Upsert => combinaison d'update et d'insert, si le champ n'existe pas il le crée et l'ajoute au document

```javascript 
db.personnes.updateOne(
	{"prenom": "remi"},
	{
		$set : {"prenom": "Remi"}
	},
	{
		"upsert": true
	}
)
```

Effectuer des recherches sur la méthode 'findAndModify'

### Validation des documents

```javascript
var athletes = [{
	"nom": "Golden",
	"prenom": "Jean-Michel",
	"discipline": "Course"
},{
	"nom": "Calogero",
	"prenom": "Frédéric",
	"discipline": "Javelo"
},{
	"nom": "AlCapone",
	"prenom": "Titouan",
}]

db.athletes.insertMany(athletes)
```

Exemple schéma de validation : 
```javascript
var schema = {
	"nom": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
		"description": "Chaîne de caractères+ expr régulière - obligatoire"
	},
	"prenom": {
		"bsonType": "string",
		"description": "Chaîne de caractères - obligatoire"
	},
	"discipline": {
		"enum": ["Course", "Nage", "Javelo"],
		"description": "Enumération - obligatoire"
	}
}
```
Maintenant que nos règles sont établies, nous allons modifier la collection a l'aide de la commande 'collMod'

```javascript
db.runCommand(
	{
		"collMod": "athletes",
		"validator": {
			$jsonSchema: {
				"bsonType": "object",
				"required": ["nom", "prenom", "discipline"],
				"properties": schema // Variable dans le code au dessus
			}
		}
	}
)
```

Essayer de lever une erreur qui permet de verifier que les schemas de validation appliquer fonctionnent

Les méthodes find et findOne ont la meme signature et permettent d'effectuer des requetes mongo

```javascript
db.collection.fin(<requete>, <projection>)
```

Chacun de ces paramètres sont tous deux des documents.

```javascript
DBQuery.shellBatchSize = 40
```

mongorc.js est un fichier de config qui se trouve a la racine du répertoire utilisateur

/home/userName

```javascript
db.maCollection.find().limit(12)
db.maCollection.find().limit(12).pretty()
```

On peut utiliser des opérateurs afin d'afiner les recherches

```javascript
db.maCollection.find({"age": {$eq: 76}})
```
$eq = equals

### Opérateur : 
- $eq = equals
- $ne = différent de
- $gt = supérieur à
- $gte = supérieur ou égal à
- $lt = inférieur à 
- $lte = inférieur ou égal à
- $in et $nin = présence et absence

On peut combiner ces opérateurs pour effectuer des recherches sur des intervalles : 

```javascript
db.maCollection.find({"age": { $lt: 50, $gt 20 }})
db.maCollection.find({"age": { $nin: [23, 45, 78] }})
db.maCollection.find({"age": { $exists: true }})
```


#### L'opérateur $expr

Il permet d'utiliser des expressions dans nos requetes. Ces expressions pourront contenir des operateurs, des objets ou encore des chemins de champ. (field path ("$maVariable" === this.)

On va chercher à afficher les documents dont la longueur du nom multipliée par 12 est supéreur  

```javascript
db.personnes.find({
	"nom": {$exists: 1},
	"age": {$exists: 1},
	$expr: {$gt: [{$multiply: [{ $strLenCP: "$nom"}, 12 ]}, "$age"]}	
},{
	"_id": 0,
	"nom": 1
	
})
```

#### Exemple: 
1) Afficher les comptes dont la sommes des opérations de débit est supérieure au montant du credit.

Création du document banque:

```javascript
db.banque.insertMany([     {"_id": 1, "credit": 2000, "debit":[1000, 22, 50]},     {"_id": 2, "credit": 200, "debit":[50, 180]},     {"_id": 3, "credit": 770, "debit":[1000]},     {"_id": 4, "credit": 0}  ])
```

Pour afficher touts les comptes dont la sommes des débit et supérieur au montant du credit on fait :
```javascript
db.banque.find({
	"debit":{$exists: 1},
	$expr: {$gt: [{$sum: "$debit"}, "$credit"]}
})
```

#### L'operateur $type

```javascript
{ champ: { $type: < type BSON > } } 
 
{ champ: { $type: < type BSON >, < type BSON > } } 
```

Pour faire de l'insertion de type : 

```javascript
db.personnes.insertOne({
	"nom": "Zidane", "prenom": "Z", "age": numberInt(50)
})
```

### L'operateur $mod (modulo)

```javascript
{ champ : { $mod: [diviseur, reste] } }
```

### L'operateur $where

Il permet d'executer directemement du code javascript sur la fonction

Il faut **priviligier** $expr à $where car il peut tout faire planté 

```javascript
db.personnes.find({ $where: "this.nom.length > 6"})

db.personnes.find({ $where: function(){
	return obj.nom.length > 6
}})
```

this. === obj.

### Les operateurs de tableau

```javascript
{ $push: { champ: valeur, ...} }
```


#### Exemple: 
1) Ajouter à Yves une passion qui est Tekken7

Création du document
```javascript
db.hobbies.insertMany([   {"_id": 1, "nom": "Yves"},   {"_id": 2, "nom": "Sandra", "passions": []},   {"_id": 3, "nom": "Line", "passions": ["Théâtre"]}  ]) 
```

Ajout de la passion à Yves avec un UpdateOne et un $push
```javascript
db.hobbies.updateOne({"nom": "Yves"},{$push: { passions: "Tekken7"}})
```

2) Ajouter à Line la passions parachutisme puis l'enlever

Ajout de la passion parachutisme à Line

```javascript
db.hobbies.updateOne({"nom": "Line"},{$push: { "passions": "Parachutisme"}})
```

Enlever la passion parachutisme à Line

```javascript
db.hobbies.updateOne({"nom": "Line"},{$pull: { "passions": "Parachutisme"}})
```


L'operateur $addToSet permet d'éviter l'ajout de doublon.

#### Exemple:
- Sur la collection personne, récuperer les documents dont l'interet est jardinage
```javascript
db.personnes.find({"interets": "jardinage"})
```

- Sur la collection personne, récuperer les documents dont l'interet est jardinage
```javascript
db.personnes.find({"interets": {$all : [ "bridge", "jardinage"] } })
```

- Affiche toutes les personnes dont jardinage est en index 1
```javascript
db.personnes.find({"interets.1": "jardinage"})
```

- Affiche toutes les personnes dont la taille du tableau interet est égal à 2
```javascript
db.personnes.find({"interets": {$size: 2}})
```

- Affiche toutes les personnes dont la taille du tableau interet est supérieur à 2
```javascript
db.personnes.find({"interets.1": {$exists: 1}})
```

### L'opérateur $elemMatch

```javascript
db.eleves.find({"notes": { $elemMatch: { $gt: 0, $lt: 10} }})

db.eleves.find({"notes": { $all: [5, 7.50] } })
```

## Les tableaux de documents

Comment on insert des documents dans des tableaux ?

```javascript
db.eleves.find({"notes.note": 10})
```

Renvoyer les documents dont les eleves ont au moins une note entre 10 et 15 dans une matière quelconque :

```javascript
db.eleves.find({
	"notes": {
		$elemMatch: {
			"note": { $gt: 10, $lte: 15 }
		}
	}
})

// Remonte toutes les notes 
db.eleves.find({"notes.note" : { $gt: 10, $lte: 15 } })

// Tri toutes les notes qui ont -10 à l'index 0
db.eleves.find({"notes.0.note": {$lt: 10}})
```

## Le tri
```javascript
curseur.sort(1) // ordre croissant
curseur.sort(-1) // ordre décroissant
```

On peut appliquer une limit avec .limit("nb de données voulu")

## Les index

Liste de mot clé  pour un gain de temps dans les recherches

De manière générale ca facilite le temps de d'écriture et de lecture des BDD

On index en priorité les champs qu'on utilise le plus souvent dans notre application

filtrage === indexation

Algolia = Entreprise qui propose un produit de recherche sur le web via un modèle SaaS
ELKelastiksearch

La nature de votre application devra impacter votre logique d'indexation : est-elle orientée écriture (write-heavy) ? ou lecture (read-heavy) ?

Quand on créer des clés primaires et étrangères elles vont automatiquement être indexé

- Création d'index : 
```javascript
db.collection.createIndex(<champ + type>, <options?>)
```

#### Exemple: 

```javascript
db.personnes.insert( [ {"nom": "Durand", "prenom": "René", "interets": ["jardinage", "bricolage"], "age": 77},  {"nom": "Durand", "prenom": "Gisèle", "interets": ["bridge", "cuisine"], "age": 75},  {"nom": "Dupont", "prenom": "Gaston", "interets": ["jardinage",   "pétanque"], "age": 79},  {"nom": "Dupont", "prenom": "Catherine", "interets": ["cuisine"], "age": 66},  {"nom": "Duport", "prenom": "Eric", "interets": ["cuisine", "pétanque"], "age": 57},  {"nom": "Duport", "prenom": "Arlette","interets": ["jardinage"], "age": 80},  {"nom": "Lejeune", "prenom": "Jean","interets": ["jardinage"], "age": 75},  {"nom": "Lejeune", "prenom": "Mariette","interets": ["jardinage", "bridge"], "age": 66}  ]  )
```

Indexation de l'age

```javascript
db.personnes.createIndex({"age": -1})
```

Récupéré tout les index

```javascript
db.personnes.getIndexes()
```

Un index peut porter sur plus d'un champ : c'est un index composé

L'ordre des champs indexé est très important

Pour supprimer un index
```javascript
db.personnes.dropIndex("age_-1")
```

Création d'un index avec un nom particulier :

```javascript
db.personnes.createIndex({"age": -1}, {"name": "index_age"})
```

Création d'un index avec une local :

```javascript
db.personnes.createIndex({"age": -1, "nom": 1}, {"name": "index_age_nom", collation: {locale: "fr"}})
```

## Les indexs géospaciaux

MongoDB permet l'utilisation de deux types d'index qui permetttent de gerer les requetes geospatiales : 

- Les indexs de type `2dshepre` sont utilises par des requetes geospatiales intervenant sur une surface spherique
- Les indexs `2d` concernent des requetes intervenant sur un plan Euclidien.

Pour un champ nomme `donneesSpatiales` d'une collection `cartographie` on peut par exemple creer un index de type `2d` avec la commande :

```javascript
db.cartographie.createIndex({"donneesSpatiales": "2d"})
```

Pour la creation d'un index `2dsphere` on utilisera plutot : 
```javascript
db.cartographie.createIndex({"donneesSpatiales": "2dsphere"})
```

Les index 2d font intervenir des coordonnées de type `legacy` 

```javascript
db.plan.insertOne({"nom":"firstPoint", "geodata": [1,1]})

db.plan.insertOne({"nom":"firstPoint_bis", "geodata": [4.7, 44.5]})

db.plan.insertOne({"nom":"firstPoint_bis_bis", "geodata": {"lon": 4.7, "lat": 44.5} })
```


## Les objets GeoJSON

Pour faire les tests : https://geojson.io/#map=2/0/20

```javascript
{type: <type d objet>, coordinates: <coordonnees>}
```

### Le type Point

```javascript
{
	"type": "Point",
	"coordinates": [14.0, 1.0]
}
```

### Le type MultiPoint

```javascript
{
	"type": "MultiPoint",
	"coordinates": [
		[14.0, 1.0], 
		[13.0, 3.0]
	]
}
```

### Le type LineString (line) : 
```javascript
{
	"type": "LineString",
	"coordinates": [
		[14.0, 1.0], 
		[13.0, 3.0]
	]
}
```


### Le type Polygon / MultiPolygon:

```javascript
// Polygon
{
	"type": "Polygon",
	"coordinates": [
		[
			[13.0, 1.0] , [13.0,3.0]
		],[
			[15.0, 1.0] , [15.0,3.0]
		]
	]
}

// MultiPolygon

{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "properties": {},
      "geometry": {
        "type": "MultiPolygon",
        "coordinates": [
          [
            [
              [
                -4.921875,
                64.16810689799152
              ],
              [
                -35.15625,
                41.50857729743935
              ],
              [
                7.3828125,
                19.973348786110602
              ],
              [
                37.6171875,
                41.77131167976407
              ],
              [
                44.6484375,
                71.41317683396566
              ],
              [
                16.5234375,
                71.74643171904148
              ],
              [
                -4.921875,
                64.16810689799152
              ]
            ],
            [
              [
                -1.0546875,
                53.330872983017066
              ],
              [
                -9.140625,
                48.69096039092549
              ],
              [
                0.3515625,
                40.713955826286046
              ],
              [
                8.7890625,
                41.244772343082076
              ],
              [
                10.546875,
                49.83798245308484
              ],
              [
                -1.0546875,
                53.330872983017066
              ]
            ]
          ],
          [
            [
              [
                -22.67578125,
                68.07330474079025
              ],
              [
                -29.355468750000004,
                64.16810689799152
              ],
              [
                -18.45703125,
                60.930432202923335
              ],
              [
                -10.72265625,
                65.36683689226321
              ],
              [
                -22.67578125,
                68.07330474079025
              ]
            ]
          ]
        ]
      }
    }
  ]
}
```

### L'operateur $nearSphere : 

```javascript
{
	$nearSphere: {
		$geometry: {
			type: "Point",
			coordinates: [<longitude>,<latitude>]
		},
		$minDistance: <distance en metres>,
		$maxDistance: <distance en metres>
	}
}

{
	$nearSphere: [ <x>, <y> ]
	$minDistance: <distance en radians>,
	$maxDistance: <distance en radians>
}
```

```javascript
var opera = { type: "Point", coordinates: [4.805325, 43.949749]}
```

Effectuer une requete sur la collection avignon

```javascript
db.avignon.find(
	{
		"localisation": {
			$nearSphere: {
				$geometry: opera
			}
		}
	}, {"_id": 0, "nom": 1}
)
```

![[Pasted image 20230208112755.png]]

### L'opérateur $geoWithin

Cela permet créer une forme géométrique et permet de capturer les objets JSON qui sont dedans

Cet operateur n'effectue aucun tri et ne necessite pas la creation d'un index geospatiale, on utilise de la maniere suivante : 

```javascript
{
	<champ des documents contenant les coordonnees> : {
		$geoWithin: {
			<operateur de forme>: <coordonnees>
		}
	}
}
```

Création d'un polygone pour notre exemple : 

```javascript
var polygone = [
	[43.9548, 4.80143],
	[43.95475, 4.80779],
	[43.95045, 4.81097],
	[43.4657, 4.80449]
]
```

La requete suivante utilise ce polygone : 

```javascript
db.avignon2d.find(
	{
		"localisation": {
			$geoWithin: {
				$polygon: polygone
			}
		}
	},{
		"_id": 0, "nom": 1
	}
)
```

### Signature pour le cas d'utilisation d'objets GeoJSON:

```javascript
{
	<champ des documents contenant les coordonnees> : {
		$geoWithin: {
			type: <"Polygon" ou "MultiPolygon">,
			coordonates: [<coordonnees>]
		}
	}
}
```


```javascript
var polygone = [
	[43.9548, 4.80143],
	[43.95475, 4.80779],
	[43.95045, 4.81097],
	[43.4657, 4.80449]
]


db.avignon.find(
	{
		"localisation": {
			$geoWithin: {
				type: "Polygon",
				coordinates: [polygone]
			}
		}
	},{
		"_id": 0, "nom": 1
	}
)
```


## Le framework d'agregation

MongoDB met a disposition un puissant outil d'analyse et de traitement de l'information : le pipeline d'agregation (ou framework)

Metaphore du tapis roulant d'usine

Méthode utilisée :

```javascript
db.collection.aggregate(pipeline, options)
```

- pipeline: designe un tableau d'étapes
- options: designe un document

Parmi les options, nous retiendrons :
- collation, permet d'affecter une collation à l'operation d'aggretion
- bypassDocumentValidation: fonctionne avec un operateur appele `$out ` et permet de passer au travers de la validation des documents.
- allowDiskUse: donne la possibilité de faire déborder les opérations d'écriture sur le disque

On peut appeler aggregate sans arguments :

```javascript
db.personnes.aggregate()
```

Au sein du shell, nous allons creer une variable pipeline : 

```javascript
var pipeline = []
db.personnes.aggregate(pipeline)
db.personne.aggregate(pipeline, {
	"collation": {
		"locale": "fr"
	}
})
```

Le filtrage avec $match
Cela permet d'obtenir des pipelines performants avec des temps d'éxecution courts. Normalement $match doit intervenir le plus en amont possible dans le pipeline car $match agit comme un filtre en réduisant le nombre de documents à traiter plus en aval dans le pipeline. (Dans l'idéal on devrait le trouver comme premier operateur).

La syntaxe est la suivante : 

```javascript
{ $match : {<requete>}}
```


Commencons par la premiere etape : 

```javascript
var pipeline =[{
	$match : {
		"interets": "jardinage"
	}
}]

db.personnes.aggregate(pipeline)
```

Cela correspond à la requete : 

```javascript
db.personnes.find({"interets": "jardinage"})
```

```javascript
var pipeline = [{
	$match : {
		"interets": "jardinage"
	},
	$match: {
		"nom": /^L/,
		"age": {$gt: 70}
	}
}]

db.personnes.aggregate(pipeline)
```

Selection/modification de champs: $project

Sert à faire des projections 

Syntaxe : 
```javascript
{ $project: {<spec>} }
```

```javascript
var pipeline = [{
	$match : {
		"interets": "jardinage"
	}
}, {
	$project: {
		"_id": 0,
		"nom": 1,
		"prenom": 1,
		"super_senior": {$gte: ["$age", 70]}
	}
},{
	$match: {
		"super_senior": true
	}
}]

db.personnes.aggregate(pipeline)
```

![[Pasted image 20230208120541.png]]

```javascript
var pipeline = [{
	$match : {
		"interets": "jardinage"
	}
}, {
	$project: {
		"_id": 0,
		"nom": 1,
		"prenom": 1,
		"ville": "$adresse.ville"
	}
},{
	$match: {
		"ville": {$exists: true}
	}
}]

db.personnes.aggregate(pipeline)
```

![[Pasted image 20230208121136.png]]

### L'opérateur $addFields

```javascript
{ $addFields : {<nouveau champ>: <expression>, ... } }
```

```javascript
db.personnes.aggregate({
	$addFields : {
		"numero_secu_s": ""
	}
})
```

Permet d'ajouter un champ plus simplement qu'avec $project


```javascript
db.achats.insertMany([
	{"nom": "Pascal","prenom": "Léo","achats": [112.29, 88.36, 72.01],"reductions": [12.30, 2.01]},
	{"nom": "Perez","prenom": "Alex","achats": [20.01, 296.35],"reductions": [9.91, 0.87]}
])

db.achats.aggregate([
	{$addFields: {"total_achats": { $sum: "$achats" }, "total_reduc": { $sum: "$reductions" }}},
	{$addFields: {"total_final": { $subtract: [ "$total_achats",  "$total_reduc" ] }}  },
	{$project: {"_id": 0,"nom": 1,"prenom":1,"Total payé": "$total_final"}  }
])
```

Bonus expliquez ce qui se passe plus bas :  

```javascript
db.achats.aggregate([
	{$addFields: {"total_achats": { $sum: "$achats" },"total_reduc": { $sum: "$reductions" }}},
	{$addFields: {"total_final": { $subtract: [ "$total_achats",  "$total_reduc" ] }}},
	{
		$project: {
			"_id": 0,
			"nom": 1,
			"prenom": 1,
			"Total payé": { 
				$divide: [{
					$subtract: [{
						$multiply: ['$total_final', 100]},{
							$mod: [{
								$multiply: ['$total_final', 100] }, 1] }] }, 100]
							}
						}
					}  
])
```

On ajoute les champs à total_achas, total_reduc et total_final en appliquant des formules mathématiques à ces derniers. 
Donc total achats est créer en additionnant tout les achats.
Donc total reduc est créer en additionnant tout les reductions.
Donc total total_final est créer en soustraient total_achats par total_reduc.

Puis enfin on créer le champs total payer qui est égal au pourcentage du total final.

### L'operateur $group

```javascript
{
	$group: {
		"_id": <expression>,
		<champ>: { <operateur d accumulation > }
	}
}
```

operateur d'accumulation:  `$push`, `$sum`, `$avg`, `$min`, `$max`

```javascript
var pipeline = [{
	$group: {
		"_id": "$age",
		"nombre_personnes": { $sum: 1 }
	},{
		$sort: {
			"nombre_personnes": 1
		}
	}
}]

db.personnes.aggregate(pipeline)


// un opérateur fais comme l'autre mais tout seul sans le pipeline

db.personnes.aggregate([
	{
		$sortByCount: "$age"
	}
])

var pipeline = ({
	$group: {
		"_id": null,
		"nombre_personnes": {$sum: 1}
	}
})

var pipeline = [
	{
		$match: {
			"age": { $exists: true }
		}
	},{
		$group: {
			"_id": null,
			"avg": {
				$avg: "$age"
			}
		}
	},{
		$project: {
			"_id": 0,
			"Age_moyen":{
				$ceil: '$avg'
			}
		}
	}
]

```