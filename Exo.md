
# ExoBook

Créez une base de données sample nommée "sample_db" et une collection appelée "employees". Insérez les documents suivants dans la collection "employees":

{ name: "John Doe", age: 35, job: "Manager", salary: 80000 }

{ name: "Jane Doe", age: 32, job: "Developer", salary: 75000 }

{ name: "Jim Smith", age: 40, job: "Manager", salary: 85000 }

Écrivez une requête MongoDB pour trouver tous les documents dans la collection "employees".

Pour cela on écrit dans le bash de MongoDB ceci : 

```javascript
db.employees.insertMany([
	{ name: "John Doe", age: 35, job: "Manager", salary: 80000 },
	{ name: "Jane Doe", age: 32, job: "Developer", salary: 75000 },
	{ name: "Jim Smith", age: 40, job: "Manager", salary: 85000 }
])
```

![[Pasted image 20230206161003.png]]

Écrivez une requête pour trouver tous les documents où l'âge est supérieur à 33.

Pour avoir que les personnes avec un âge supérieur à 33 ans on fait :

```javascript
db.employees.find({"age": {$gt: 33}})
```

![[Pasted image 20230206161028.png]]

Écrivez une requête pour trier les documents dans la collection "employees" par salaire décroissant.

Pour trier les salaires de la collection "employees" par ordre décroissant on fait
```javascript
db.employees.find().sort({"salary":-1})
```

![[Pasted image 20230206161105.png]]

Écrivez une requête pour sélectionner uniquement le nom et le job de chaque document.

Pour afficher seulement le nom et le job de cache personne on fait :

```javascript
db.employees.find({},{"name":true,"job":true})
```

![[Pasted image 20230206161225.png]]

Écrivez une requête pour compter le nombre d'employés par poste.

Pour compter le nombre d'employés par poste on fait :

```javascript
db.employees.aggregate({ $group: { _id: "$job", count: { $sum: 1 } } })
```

![[Pasted image 20230206164731.png]]

Écrivez une requête pour mettre à jour le salaire de tous les développeurs à 80000.

Pour mettre à jour tout les salaires des développeurs à 80000€ on fait 

```javascript
db.employees.updateMany(

	{job: "Developer"},
	{$set: {salary: 80000}}

)
```

![[Pasted image 20230206165037.png]]

![[Pasted image 20230206165047.png]]


# Exo

Voici la base de donnees qui permet d'effectuer la serie d'exercices :

```
db.salles.insertMany([ 
   { 
       "_id": 1, 
       "nom": "AJMI Jazz Club", 
       "adresse": { 
           "numero": 4, 
           "voie": "Rue des Escaliers Sainte-Anne", 
           "codePostal": "84000", 
           "ville": "Avignon", 
           "localisation": { 
               "type": "Point", 
               "coordinates": [43.951616, 4.808657] 
           } 
       }, 
       "styles": ["jazz", "soul", "funk", "blues"], 
       "avis": [{ 
               "date": new Date('2019-11-01'), 
               "note": NumberInt(8) 
           }, 
           { 
               "date": new Date('2019-11-30'), 
               "note": NumberInt(9) 
           } 
       ], 
       "capacite": NumberInt(300), 
       "smac": true 
   }, { 
       "_id": 2, 
       "nom": "Paloma", 
       "adresse": { 
           "numero": 250, 
           "voie": "Chemin de l'Aérodrome", 
           "codePostal": "30000", 
           "ville": "Nîmes", 
           "localisation": { 
               "type": "Point", 
               "coordinates": [43.856430, 4.405415] 
           } 
       }, 
       "avis": [{ 
               "date": new Date('2019-07-06'), 
               "note": NumberInt(10) 
           } 
       ], 
       "capacite": NumberInt(4000), 
       "smac": true 
   }, 
    { 
       "_id": 3, 
       "nom": "Sonograf", 
       "adresse": { 
           "voie": "D901", 
           "codePostal": "84250", 
           "ville": "Le Thor", 
           "localisation": { 
               "type": "Point", 
               "coordinates": [43.923005, 5.020077] 
           } 
       }, 
       "capacite": NumberInt(200), 
       "styles": ["blues", "rock"] 
   } 
]) 
```

Exercice 1

Affichez l’identifiant et le nom des salles qui sont des SMAC.

Pour afficher toute les salles comportant l'identifiant "smac" on fait :

```javascript
db.salles.find({"smac":true}, {"_id": 1, "nom": 1})
```

![[Pasted image 20230206170756.png]]

Exercice 2

Affichez le nom des salles qui possèdent une capacité d’accueil strictement supérieure à 1000 places.

Pour afficher toutes les salles comportant une capacité d'accueil strictement supérieur à 1000 on fait :

```javascript
db.salles.find({"capacite":{$gt: 1000}}, {"nom": 1})

// $gt = strictement supérieur
```

![[Pasted image 20230206171059.png]]

Exercice 3

Affichez l’identifiant des salles pour lesquelles le champ adresse ne comporte pas de numéro.

Pour afficher que les salless avec le champ adresse on fait :

```javascript
db.salles.find({"adresse.numero": {$exists: true}}, {"_id": 1})
```

![[Pasted image 20230206172514.png]]

Exercice 4

Affichez l’identifiant puis le nom des salles qui ont exactement un avis.

Pour afficher l'identifiant puis le nom des salles qui ont un avis on fait :

```javascript
db.salles.find({"avis": {"$exists": true}}, {"_id": 1, "nom": 1})
```

![[Pasted image 20230207112052.png]]
Exercice 5

Affichez tous les styles musicaux des salles qui programment notamment du blues.

Pour afficher tous les styles musicaux des salles avec du blues on fait :

```javascript
db.salles.find({"styles": {$all : [ "blues"] } },{"nom": 1, "styles": 1})
```

![[Pasted image 20230207112121.png]]

Exercice 6

Affichez tous les styles musicaux des salles qui ont le style « blues » en première position dans leur tableau styles.

Pour afficher toutes les salles qui ont le style "blues" en première position on fait :

```javascript
db.salles.find({"styles.0": "blues"}, {"nom":1, "styles":1})
```

![[Pasted image 20230207112336.png]]

Exercice 7

Affichez la ville des salles dont le code postal commence par 84 et qui ont une capacité strictement inférieure à 500 places (pensez à utiliser une expression régulière).

Pour afficher la ville commencant par le code postal 84 et dont sa salle comporte strictement moins de 500 places on fait :

```javascript
db.salles.find({
	"capacite":{$lt: 500},
	"adresse.codePostal": { $regex: '84' }
},{
	"nom":1,
	"adresse.ville":1
})
```

![[Pasted image 20230207114057.png]]

Exercice 8

Affichez l’identifiant pour les salles dont l’identifiant est pair ou le champ avis est absent.

Pour afficher l'identifiant des salles dont l'identifiant est pair ou le champ avis est absent :

```javascript
db.salles.find({ $or: [{ "_id" : { $mod: [2, 0] }}, { "avis" : {$exists: 0}} ]}, {"_id": 1, "nom":1, "avis": 1}) 
```

![[Pasted image 20230207115426.png]]

Exercice 9

Affichez le nom des salles dont au moins un des avis comporte une note comprise entre 8 et 10 (tous deux inclus).

Pour afficher le nom des salles dont au moins un avis comporte une  note comprise entre 8 et 10 on fait :

```javascript
db.salles.find({"avis": {$elemMatch : {"note": { $gte:8, $lte:10 } } } }, {"nom":1, "avis":1})
```

![[Pasted image 20230207122523.png]]

Exercice 10

Affichez le nom des salles dont au moins un des avis comporte une date postérieure au 15/11/2019 (pensez à utiliser le type JavaScript Date).

Pour afficher le  nom des salles qui ont au moins un de leurs avis qui comporte une date postérieurer au 15/11/2019 on fait :

```javascript
db.salles.find({"avis.0.date": {$lt:ISODate("2019-11-15T00:00:00Z")}}, {"nom":1, "avis":1})
```

![[Pasted image 20230207121729.png]]

Exercice 11

Affichez le nom ainsi que la capacité des salles dont le produit de la valeur de l’identifiant par 100 est strictement supérieur à la capacité.

Pour afficher le nom et la capacité des salles par le produit de la valeur de leur identifiant par 100 est strictement supérieur à la capacité on fait

```javascript
db.salles.find({
	"_id":{$exists: 1},
	"capacite": {$exists:1},
	$expr: {$gt: [{$multiply: ["$_id", 100]}, "$capacite"]}
	
},{
	"nom": 1,
	"capacite": 1
})
```

![[Pasted image 20230207140733.png]]

Exercice 12

Affichez le nom des salles de type SMAC programmant plus de deux styles de musiques différents en utilisant l’opérateur $where qui permet de faire usage de JavaScript.

Pour afficher le nom des salles de type SMAC avec plus de deux musiques différents en utilisant l'opérateur $where on fait :

```javascript
db.salles.find({$where: "this.smac == true && this.styles?.length >2"}, {"nom": 1})
```

Exercice 13

Affichez les différents codes postaux présents dans les documents de la collection salles.

Pour afficher les codes postaux de la collection salles on fait :

```javascript
db.salles.find({"adresse.codePostal": {$exists: 1}}, {"nom":1, "adresse.codePostal": 1 })
```

![[Pasted image 20230207145027.png]]

Exercice 14

Mettez à jour tous les documents de la collection salles en rajoutant 100 personnes à leur capacité actuelle.

Pour mettre à jour tous les documents de la collection salles en rajoutant 100 personnes à leur capacité actuelle on fait :

```javascript
db.salles.updateMany(
	{}, // on veut mettre à jour toute les collections
	{$inc: {"capacite": 100} // $inc permet d'ajouter 100 à toute les capacite des collections
})
```

Exercice 15

Ajoutez le style « jazz » à toutes les salles qui n’en programment pas.

Pour ajouter le style "jazz" à toutes les salles qui n'en programment pas on fait : 

```javascript
db.salles.updateMany(
	{"styles": { $ne: "jazz" } },
	{$push: { styles:"jazz"}}
)
```

![[Pasted image 20230207161404.png]]

Exercice 16

Retirez le style «funk» à toutes les salles dont l’identifiant n’est égal ni à 2, ni à 3.

Pour retirez le style "funk" à toutes les salles dont sont identifiant est ni 2, ni 3 on fait :

```javascript
db.salles.updateMany(
	{"styles": { $ne: [2, 3] } },
	{$pull: { styles:"funk"} }
)
```

![[Pasted image 20230207162023.png]]

Exercice 17

Ajoutez un tableau composé des styles «techno» et « reggae » à la salle dont l’identifiant est 3.

Pour ajouter les styles au techno et reggae à la salle avec l'identifiant 3 on fait

```javascript
db.salles.updateMany(
	{"_id": 3},
	{$push: { "styles": { "$each": ["techno","reggae"]}}
})
```

![[Pasted image 20230207164430.png]]

Exercice 18

Pour les salles dont le nom commence par la lettre P (majuscule ou minuscule), augmentez la capacité de 150 places et rajoutez un champ de type tableau nommé contact dans lequel se trouvera un document comportant un champ nommé telephone dont la valeur sera « 04 11 94 00 10 ».

Pour augmenter la capacité des places de 150 pour les salles commencant par P avec un champ en plus comportant le numéro de téléphone de la salle on fait :



```javascript
db.salles.updateOne(
	{
		"nom": {$regex : /^p/ , $options: 'i'},// On fait une regex qui prends que les noms
											// commencant par p (majuscule/minuscule) 
											// et on mets un options 'i' pour négliger la casse
	},{
		$inc: {"capacite": 150},		
		$set: {telephone: "04 11 94 00 10"}
	}
)
```

![[Pasted image 20230207171505.png]]

Exercice 19

Pour les salles dont le nom commence par une voyelle (peu importe la casse, là aussi), rajoutez dans le tableau avis un document composé du champ date valant la date courante et du champ note valant 10 (double ou entier). L’expression régulière pour chercher une chaîne de caractères débutant par une voyelle suivie de n’importe quoi d’autre est [^aeiou]+$.

Pour rajouter dans le tableau avis un document composer de la data courante et d'un champ note valant 10

```javascript
db.salles.updateMany({
	"nom": {$regex : /^[aeiou].*/ , $options: 'i'},
}, {
	$push: { "avis": {"date" : new Date(), "note": 10 }}
})
```


Exercice 20

En mode upsert, vous mettrez à jour tous les documents dont le nom commence par un z ou un Z en leur affectant comme nom « Pub Z », comme valeur du champ capacite 50 personnes (type entier et non décimal) et en positionnant le champ booléen smac à la valeur « false ».


Pour mettre à jour tout les documents commencent par z ou Z en modifiant la capacite en le champ capacite en "Pub Z" avec 50 personnes et en modifiant le champ smac à false on fait :

```javascript
db.salles.updateMany({
	"nom": {$regex: /^[zZ].*/, $options: 'i'}
}, {
	$set: {
		"nom" : "Pub Z",
		"capacite": 50,
		"smac": false
	}
},{
	upsert: true
})
```

Exercice 21

Affichez le décompte des documents pour lesquels le champ _id est de type « objectId ».

Pour afficher le décompte des documents pour lesquels le champ est de type objectId on fait : 

```javascript

db.salles.countDocuments({
	"_id": {$type: "objectId"}
})

```

![[Pasted image 20230208124042.png]]

Exercice 22

Pour les documents dont le champ _id n’est pas de type « objectId », affichez le nom de la salle ayant la plus grande capacité. Pour y parvenir, vous effectuerez un tri dans l’ordre qui convient tout en limitant le nombre de documents affichés pour ne retourner que celui qui comporte la capacité maximale.

```javascript
db.salles.find({
	"_id": {$not: {$type: "objectId"}},
},{
	"nom": 1,
	"capacite": 1
}).sort({"capacite": -1}).limit(1)
```

![[Pasted image 20230208123822.png]]

Exercice 23

Remplacez, sur la base de la valeur de son champ _id, le document créé à l’exercice 20 par un document contenant seulement le nom préexistant et la capacité, que vous monterez à 60 personnes.

```javascript
db.salles.updateMany(
	{
		"_id": {$type: ["objectId"]}
	},
	{
		$set: {"capacite": 60},
		$unset: { "smac": "" }
	}
)
```

Exercice 24

Effectuez la suppression d’un seul document avec les critères suivants : le champ _id est de type « objectId » et la capacité de la salle est inférieure ou égale à 60 personnes.

```javascript
db.salles.remove(
	{
		"_id": {$type: ["objectId"]},
		"capacite": {$lte: 60}
	}
)
```

![[Pasted image 20230208142934.png]]

Exercice 25

À l’aide de la méthode permettant de trouver un seul document et de le mettre à jour en même temps, réduisez de 15 personnes la capacité de la salle située à Nîmes.

```javascript
db.salles.updateOne(
	{
		"adresse.ville": "Nîmes"
	},{
		$inc: {"capacite": -15}
	}
)
```

![[Pasted image 20230208144933.png]]



# Exo Validation

Exercice 1 Modifiez la collection salle afin que soient dorénavant validés les documents destinés à y être insérés ; cette validation aura lieu en mode « strict » et portera sur les champs suivants :

nom sera obligatoire et devra être de type chaîne de caractères.

capacite sera obligatoire et devra être de type entier (int).

Dans le champ adresse, les champs codePostal et ville, tous deux de type chaîne de caractères, seront obligatoires.

Que constatez-vous lors de la tentative d’insertion suivante, et quelle en est la cause ?

```
db.salles.insertOne( 
{"nom": "Super salle", "capacite": 1500, "adresse": {"ville": "Musiqueville"}} 
) 
```

![[Pasted image 20230208163842.png]]

On voit qu'on crée une salle qui n'a pas le meme id que les autres puisque ici on crée un Id via un objectId. De plus on voit que le champ codePostal. La cause est qu'il n'y a pas de vérificateur de champs pour savoir si les champs sont tous bien incrémenter

Que proposez-vous pour régulariser la situation ?

Pour régulariser la situtation faut donc mettre un validateur de champs comme ci dessous:

```javascript
// Création du schéma de validation
var schema = {
	"nom": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
	"capacite": {
		"bsonType": "int",
	},
	"adresse.ville": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
	"adresse.codePostal": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
}

// Application du validateur avec le schéma de validation
db.runCommand(
	{
		"collMod": "salles",
		"validator": {			
			$jsonSchema: {
				"bsonType": "object",
				"required": ["nom", "capacite", "adresse.ville", "adresse.codePostal"],
				"properties": schema
			}
		}
	}
)

```

![[Pasted image 20230208165747.png]]

Exercice 2

Rajoutez à vos critères de validation existants un critère supplémentaire : le champ _id devra dorénavant être de type entier (int) ou ObjectId.

```javascript
// Création du schéma de validation
var schema = {
	"_id": {
		"bsonType": ["int", "objectId"],
	},
	"nom": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
	"capacite": {
		"bsonType": "int",
	},
	"adresse.ville": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
	"adresse.codePostal": {
		"bsonType": "string",
		"pattern": "^[A-Z].*",
	},
}

// Application du validateur avec le schéma de validation
db.runCommand(
	{
		"collMod": "salles",
		"validator": {			
			$jsonSchema: {
				"bsonType": "object",
				"required": ["_id","nom", "capacite", "adresse.ville", "adresse.codePostal"],
				"properties": schema
			}
		}
	}
)

```

![[Pasted image 20230208170212.png]]

Que se passe-t-il si vous tentez de mettre à jour l’ensemble des documents existants dans la collection à l’aide de la requête suivante :

```
db.salles.updateMany({}, {$set: {"verifie": true}}) 
```

Une erreur de validation arrive
![[Pasted image 20230208170239.png]]

Supprimez les critères rajoutés à l’aide de la méthode delete en JavaScript

```javascript
db.runCommand(
	{
		"collMod": "salles",
		"validator": {}
	}
)
```

Exercice 3

Rajoutez aux critères de validation existants le critère suivant :

Le champ smac doit être présent OU les styles musicaux doivent figurer parmi les suivants : "jazz", "soul", "funk" et "blues".

```javascript

```

Que se passe-t-il lorsque nous exécutons la mise à jour suivante ?

db.salles.update({"_id": 3}, {$set: {"verifie": false}})

![[Pasted image 20230208171257.png]]

# Exo Aggregation

Exercice 1

Écrivez le pipeline qui affichera dans un champ nommé ville le nom de celles abritant une salle de plus de 50 personnes ainsi qu’un booléen nommé grande qui sera positionné à la valeur « vrai » lorsque la salle dépasse une capacité de 1 000 personnes. Voici le squelette du code à utiliser dans le shell :

```javascript
var pipeline = [{
	$match : {}
	}, {
		$project: {
			"adresse.ville": 1,
			"ville": {$gte: ["$capacite", 70]},
			"grande": {$gte: ["$capacite", 1000]}
		}
	},{
		$match: {
			"ville": true,
			"grande": true
		}
	}
]

db.salles.aggregate(pipeline)
```

![[Pasted image 20230208154957.png]]

Exercice 2

Écrivez le pipeline qui affichera dans un champ nommé apres_extension la capacité d’une salle augmentée de 100 places, dans un champ nommé avant_extension sa capacité originelle, ainsi que son nom.

```javascript
var pipeline = [{
	$match : {}
}, {
	$project: {
		"nom": 1,
		"apres_extension": {$sum: ["$capacite", 100]},
		"avant_extension": "$capacite"
	}
}]

db.salles.aggregate(pipeline)
```

![[Pasted image 20230208162150.png]]

Exercice 3

Écrivez le pipeline qui affichera, par numéro de département, la capacité totale des salles y résidant. Pour obtenir ce numéro, il vous faudra utiliser l’opérateur $substrBytes dont la syntaxe est la suivante :

```
{$substrBytes: [ < chaîne de caractères >, < indice de départ >, 
< longueur > ]} 
```


```javascript
var pipeline = [ { 
	$project: { 
		"_id": 0, 
		"departement": { 
			$substrBytes: ["$adresse.codePostal", 0, 2] 
		}, 
		"CapaciteTotal": "$capacite" 
		} 
	}, { 
		$group: { 
			"_id": "$departement", 
			"capacite_totale": { $sum: "$CapaciteTotal" } 
		} 
	} 
];

db.salles.aggregate(pipeline)
```

![[Pasted image 20230209220322.png]]

Exercice 4

Écrivez le pipeline qui affichera, pour chaque style musical, le nombre de salles le programmant. Ces styles seront classés par ordre alphabétique.

```javascript
var pipeline = [{
	$match : {}
	}, { 
		$unwind: "$styles"
	},{
		$group: { "_id": "$styles", "nbSalles": {$sum: 1} }
	},{
		$sort: {"_id": 1}
	}
]

db.salles.aggregate(pipeline)
```

![[Pasted image 20230208221859.png]]

Exercice 5

À l’aide des buckets, comptez les salles en fonction de leur capacité :

celles de 100 à 500 places

celles de 500 à 5000 places

```javascript
var pipeline = [
	{ 
		$bucket: {
			 groupBy: "$capacite",
			 boundaries: [100, 500, 5000], 
			 default: "other", 
			 output: { "count": { $sum: 1 } } } },
		{ $sort: { _id: 1 } }
]

db.salles.aggregate(pipeline)
```

![[Pasted image 20230208222445.png]]

Exercice 6

Écrivez le pipeline qui affichera le nom des salles ainsi qu’un tableau nommé avis_excellents qui contiendra uniquement les avis dont la note est de 10.

```javascript
var pipeline = [
	{ 
		$match: { 
			"avis.note": 10 } 
		},{ 
		$project: { 
			"nom": 1, "avis_excellents": { 
				$filter: { 
					input: "$avis", as: "avis", 
					cond: { 
						$eq: ["$$avis.note", 10] 
					} 
				} 
			}
		}
	}
]

db.salles.aggregate(pipeline)
```

![[Pasted image 20230208222904.png]]

# Exo Index

Exercice 1

Un bref examen de vos fichiers journaux a révélé que la plupart des requêtes effectuées sur la collection salles cible des capacités ainsi que des départements, comme ceci :

```
db.salles.find({"capacite": {$gt: 500}, "adresse.codePostal": /^30/}) 
db.salles.find({"adresse.codePostal": /^30/, "capacite": {$lte: 400}}) 
```

Que proposez-vous comme index qui puisse couvrir ces requêtes ?

- On peut mettre en index la capacité en la filtrant de manière décroissante pour trouver plus vite toute les salles qui propose le plus grand nombre de capacite d'accueil. Pour cela on doit faire :

```javascript
db.salles.createIndex({"capacite": -1}, ,))

db.salles.createIndex({"capacite" : -1, "adresse.codePostal": 1}, {"name": "index_capacite_adresseCP"})
```

![[Pasted image 20230208153531.png]]

Détruisez ensuite l’index créé.

- Pour supprimer l'index crée on fait :

```javascript
db.salles.dropIndex("index_capacite_adresseCP")
```

![[Pasted image 20230208153606.png]]

# Exo Géo
Exercice 1

Vous disposez du code JavaScript suivant qui comporte une fonction de conversion d’une distance exprimée en kilomètres vers des radians ainsi que d’un document dont les coordonnées serviront de centre à notre sphère de recherche. Écrivez la requête $geoWithin qui affichera le nom des salles situées dans un rayon de 60 kilomètres et qui programment du Blues et de la Soul.

```
var KilometresEnRadians = function(kilometres){ 
   var rayonTerrestreEnKm = 6371; 
   return kilometres / rayonTerrestreEnKm; 
}; 
 
var salle = db.salles.findOne({"adresse.ville": "Nîmes"}); 
 
var requete = { ... }; 
 
db.salles.find(requete ... }; 
```


Réponse :

```javascript
var KilometresEnRadians = function(kilometres){ 
   var rayonTerrestreEnKm = 6371; 
   return kilometres / rayonTerrestreEnKm; 
}; 
 
var salle = db.salles.findOne({"adresse.ville": "Nîmes"}); 
 
var requete = {
   "localisation": {
      $geoWithin: {
         $centerSphere: [
            [
	            salle.adresse.localisation.coordinates[0], 
	            salle.adresse.localisation.coordinates[1] 
	        ],
            KilometresEnRadians(60)
         ]
      }
   },
   "styles": {
      $in: ["Blues", "Soul"]
   }
}; 
 
db.salles.find(requete, {"nom": 1}); 

```

Exercice 2:

Écrivez la requête qui permet d’obtenir la ville des salles situées dans un rayon de 100 kilomètres autour de Marseille, triées de la plus proche à la plus lointaine :

```
var marseille = {"type": "Point", "coordinates": [43.300000, 5.400000]} 
 
db.salles.find(...) 
```

```javascript

var KilometresEnRadians = function(kilometres)
{   
	var rayonTerrestreEnKm = 6371;   
	return kilometres / rayonTerrestreEnKm;   
}; 

var marseille = 
{
	"type": "Point", 
	"coordinates": [43.300000, 5.400000]
}

var requete = {  
	"adresse.localisation": {  
	    $geoWithin: {  
			$centerSphere: 
			[  
				[
					marseille.coordinates[0],
					marseille.coordinates[1]
				],  
				KilometresEnRadians(100)  
			]  
		}  
	},  
}

db.salles.find(requete, {"nom": 1}).sort( { "adresse.localisation": 1 })

```

Exercice 3:

Soit polygone un objet GeoJSON de la forme suivante :

```
var polygone = { 
     "type": "Polygon", 
     "coordinates": [ 
            [ 
               [43.94899, 4.80908], 
               [43.95292, 4.80929], 
               [43.95174, 4.8056], 
               [43.94899, 4.80908] 
            ] 
     ] 
}

db.salles.find(requete, {"nom": 1})

```

Donnez le nom des salles qui résident à l’intérieur.

![[Pasted image 20230209222411.png]]
