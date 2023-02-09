
# Sujet : Météo


Jeu de données: 

Téléchargez ou générez un jeu de données de stations météorologiques, qui incluent la date, la température, la pression atmosphérique, etc.

1. Préparation des données:
	a. Importez les données de stations météorologiques dans MongoDB en utilisant la commande mongoimport.
	b. Assurez-vous que les données sont bien structurées et propres pour une utilisation ultérieure.

Réponse :
	a et b: 
		Pour cela j'ai importé un jeu de données de Kaggle qui est "Historical weather data of 194 country capitals" à l'adresse suivante "https://www.kaggle.com/datasets/balabaskar/historical-weather-data-of-all-country-capitals?resource=download". 
		Je me suis au préalable assuré que les données était compréhensible et en accord avec ce qui est demander au niveau de la présence de certaines données. 
		Cependant j'ai quand même modifier le type de certains champs notamment le champ date que j'ai passé en type string en type Date, j'ai aussi modifié le type des champs latitude, longitude, tavg, tmin, tmax, wdir, wspd et pres en number car ils étaient de base en type string.

2. Indexation avec MongoDB: 
	a. Créez un index sur le champ de la date pour améliorer les performances de la recherche. Utilisez la méthode createIndex (). 
	b. Vérifiez que l'index a été créé en utilisant la méthode listIndexes ().

Réponse : 
	a. Pour créer un index sur le champ date j'ai donc fais comme ceci : 
	
```javascript
db.weather.createIndex({
	"date" : 1, // Ici je créé mon index par ordre croissant pour avoir les dates les plus récentes
}, {
	"name": "index_date"
})
```

Voici donc une capture d'écran de la création de mon index : 
![[Pasted image 20230209102656.png]]
*Capture d'écran du retour de la requete*

b. Pour vérifier la présence de mon index en utilisant la méthode listIndexes() j'ai fais :

```javascript
db.runCommand ({	
	listIndexes: "weather"	
	}
)
```

Voici le résultat de notre requete avec une capture d'écran:

![[Pasted image 20230209104046.png]]
*Capture d'écran du retour de la requete*

Notre index ce situe dans le firstBatch dans le deuxième [Object] car quand nous avons créer notre index avec la commande précédente cela à créé un **objet** du nom "index_date" c'est donc pour cela que le firstBatch nous montre un objet.

3. Requêtes MongoDB: 
	a. Recherchez les stations météorologiques qui ont enregistré une température supérieure à 25°C pendant les mois d'été (juin à août). Utilisez la méthode find () et les opérateurs de comparaison pour trouver les documents qui correspondent à vos critères. 
	b. Triez les stations météorologiques par pression atmosphérique, du plus élevé au plus bas. Utilisez la méthode sort () pour trier les résultats.

Réponses : 
	a. Afin d'avoir toutes les stations météorologiques (donc ici dans notre cas les villes) qui ont enregistré une température supérieur à 25°C pendant les mois de juin à aout j'ai fais (ici j'ai pris comme année de référence l'été 2021) :

```javascript
db.weather.find(
	 {
		 "tmax": { $gte: 25 },
		 "date" : {$gte:ISODate("2020-06-01"), $lte:ISODate("2021-08-31")}
	},{
		"_id": 0,
		"country": 1, 
		"city": 1,
		"tmax": 1
	}
)
```

Voici le résultat de la requete précédente (il n'y a pas tout les résultats mais juste une partie dans la capture d'écran) :

![[Pasted image 20230209121831.png]]
*Capture d'écran du retour de la requete*

b.  Afin d'avoir la pression atmosphérique, du plus élevé au plus bas sur les stations météorologiques (donc dans notre cas les villes)  j'ai fais :

```javascript
db.weather.aggregate( [  
	{
		$group: {
			 _id: "$city", 
			 "press": {$max: "$pres"} 
		}
	}, { 
		$sort: { "press":-1 } 
	} 
] )
```

Voici donc le résultat avec la requete précédente (il n'y a pas tout les résultats mais juste une partie dans la capture d'écran):

![[Pasted image 20230209121645.png]]
*Capture d'écran du retour de la requete*

4. Framework d'agrégation: 
	a. Calculez la température moyenne par station météorologique pour chaque mois de l'année. Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données et grouper les données par mois. 
	b. Trouvez la station météorologique qui a enregistré la plus haute température en été. Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données et trouver la valeur maximale.

Réponse : 
	a. Afin de calculez la température moyenne par station pour chaque mois de l'année en utilisant un framework d'agrégation (ici j'ai pris comme année de référence l'année 2020): 

```javascript
var tempMoyenne = [
	{
		$match : {"date" : {$gte:ISODate("2020-01-01"), $lte:ISODate("2020-12-31")}}
	},{
		$addFields : {
		"mois": {$month: "$date"}
	}
	},{
		$group: {
			"_id": {mois: "$mois", "city": "$city"},
			"temp": {$avg: "$tavg"},
		}
	},{
		$project: {
			"_id_": 1,
			"temp": 1,			
		}
	},{
		$sort: {
			"_id": 1
		}
	}
]

db.weather.aggregate(tempMoyenne)
```

Voici le résultat de la requete précédente :

![[Pasted image 20230209135633.png]]
*Capture d'écran du retour de la requete*

Dans cette capture d'écran nous voyons que les stations météorologique pour le mois de Janvier puisque comme il y a 194 pays il faut déjà les passer une fois pour voir le mois de Février. Cependant si nous remplacons le $city par Paris par exemple cela nous donne:

![[Pasted image 20230209135916.png]]
*Capture d'écran de la requete jusqu'au mois de Mai.*



Trouvez la station météorologique qui a enregistré la plus haute température en été. Utilisez le framework d'agrégation de MongoDB pour effectuer des calculs sur les données et trouver la valeur maximale.

b. Afin de trouver la station météorologique qui a enregistré la plus haute température en été je me suis tout d'abord basé sur l'année 2020 comme année de référence puis j'ai fais l'agrégation ci dessous : 

```javascript
var tempMaxEte = [
	{
		$match : {"date" : {$gte:ISODate("2020-06-20"), $lte:ISODate("2020-09-22")}}
	},{
		$addFields : {
		"mois": {$month: "$date"}
	}
	},{
		$group: {
			"_id": {mois: "$mois", "city": "$city"},
			"temp": {$max: "$tmax"},
		}
	},{
		$project: {
			"_id_": 1,
			"temp": 1,			
		}
	},{
		$sort: {
			"temp": -1
		}
	},{
		$limit: 1		
	}
]

db.weather.aggregate(tempMaxEte)
```

Ce qui nous donne : 

![[Pasted image 20230209141913.png]]
*Capture d'écran du retour de la requete*

Donc on peut constaté que c'est au mois de Juillet à Kuwait City qu'il a fait le plus chaud avec 49.9°C

5. Export de la base de données: 
	a. Exportez les résultats des requêtes dans un fichier CSV pour un usage ultérieur. Utilisez la commande mongoexport pour exporter des données de MongoDB.

Réponse : 
a. L'export de la base de données porte le nom de [weather.csv] 
