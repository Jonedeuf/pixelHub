
**Auteur : Valentin Minnebo**

---

**Question 1 :**
Ici, c'est mongoimport qui a choisi les identifiants pour les nouveaux jeux crées. Dans le TP1, c'était PostgreSQL qui avait choisi l'id de Pixel par incrémentation automatique. De ce fait, sa valeur était facilement devinable avant même la création du nouveau joueur.

**Question 2 :**
Sans utiliser d'objets complexes tels que les ARRAY, il aurait fallu créer une table `Plateforme`pour les différentes plateformes prises en charge et une table `Plateforme_jeu`pour faire la liaison entre les jeux et les plateformes.

```sql
CREATE TABLE "Plateforme"(
    id SERIAL PRIMARY KEY,
    label VARCHAR(50)
);

CREATE TABLE "Plateforme_jeu"(
    id_jeu INTEGER REFERENCES "jeux"(id),
    id_plateforme INTEGER REFERENCES "Plateforme"(id),
    PRIMARY KEY (id_jeu, id_plateforme)
);
```

La requête SQL équivalente à celle de MongoDB ressemblerait à ça :

```sql
SELECT "Jeux".titre FROM "Jeux"
INNER JOIN "Plateforme_jeu" ON "Jeux".id = "Plateforme_jeu".id_jeu
INNER JOIN "Plateforme" ON "Plateforme_jeu".id_plateforme = "Plateforme".id
WHERE "Plateforme".label = 'Switch';
```

**Question 3 :**
Il est impératif d'ajouter cet argument aux commandes pour pouvoir indiquer que les indentifiants de connexion de l'utilisateur se trouvent dans la bdd système `admin` afin de lui accorder les droits d'accès nécessaires. L'utilisateur pixelhub est situé dans la collection système `admin.system.users` du conteneur MongoDB en cours.

**Question 4 :**
Il s'agit d'un problème et ce pour diverses raisons. L'une des raisons qui peut être citée est le non respect des règles de nommage par l'ajout arbitraire d'un champ mal nommé. Par exemple, dans le cas du projet à 4 développeurs, l'un d'entre eux ajoute 6 mois plus tard un nouveau document avec un champ "note_moyenne". Or, dans d'autres documents, il existe le champ "note" dont la fonction est identique, ce qui pose un conflit de nommage entre ces deux champs.


**Question 5 :**
Le résultat ne me surprends pas parce qu'il s'agit d'un cas classique de sensibilité à la casse (distinction majuscule/minuscule). Si aucun message d'erreur n'a été levé, c'est parce que MongoDB considère que `Jeux` est une collection vide lors du `find`, et non une collection inexistante (comme lorsqu'un champ n'est pas trouvé dans un enregistrement).

**Question 6 :** Voici l'erreur retournée par MongoDB :

```bash
MongoInvalidArgumentError: Update document requires atomic operators
```

Cette erreur est affichée parce qu'aucun opérateur ($set, $unset, $inc, etc...) n'a été spécifié dans le champ de modification de la commande. Si on voulait remplacer le document dans son intégralité, il aurait fallu changer de commande et passer de ```updateOne```à ```replaceOne```:

```javascript
db.jeux.replaceOne({ titre: "Valorant" }, { note: 4.4 })
```

**Question 7 :** En SQL, pour faire l'équivalent de ce qui a été fait dans l'exercice 12, il aurait fallu mettre à jour la table ```jeux``` pour y inclure un champ ```nbVotes``` puis il aurait fallu mettre à jour la colonne "Terraria" :

```sql
ALTER TABLE "Jeux"
ADD nbVotes INT;

UPDATE "Jeux"
SET nbVotes = 0
WHERE titre = 'Terraria';
```

Cette modification aurait impliquée que la valeur de ```nbVotes```pour les autres colonnes de la table auraient été nulles, ce qui aurait consommé plus de ressources pour des données vides.

**Question 8 :** Dans ```IGameCatalog```, MongoDB n'est pas mentionné une seule fois. Cela est important parce qu'il ne s'agit pas directement d'un utilitaire MongoDB mais d'une interface définissant les fonctions utilisables par les différentes couches d'accès en BDD, y compris celle de MongoDB (```MongoGameCatalog```). Il faut laisser cette interface telle quelle parce qu'elle représente une base solide pour d'autres couches d'accès à l'avenir.

**Question 9 :** L'exception obtenue est un ```System.FormatException```sur le champ ```joueursParEquipe```. L'erreur s'est produite parce que losque l'API fait appel à /games, la fonction correspondante dans ```MongoGameCatalog``` convertit les jeux récupérés dans MongoDB en instances de la classe ```Game```, or sa définition dans ```Models```ne spécifie pas la présence du champ ```joueursParEquipe```, donc la déserialisation échoue.

**Question 10 :** Aucune exception n'est levée, mais le champ ```joueursParEquipe```n'apparait nulle part dans la requête API parce qu'il a été ignoré lors de la sérialisation.

**Question 11 :** L'avantage de ```[BsonIgnoreExtraElements]``` est de faciliter l'écriture du code grâce à une instruction qui tient sur 1 ligne, mais son inconvénient est la perte de données qui peuvent être utiles. Les avantages/inconvénients s'inversent pour ```[BsonExtraElements]``` : il empêche la perte de données utiles mais le code devient moins simple, avec la manipulation de structures dynamiques (```BsonDocument```). Enfin, les hiérarchies de classes ont comme avantage d'être très typées et propres, limitant le risque de cassage de l'application par mauvais typage. Cependant, elles sont assez fastidieuses à mettre en place.

**Question 12 :** 

```sql
SELECT genre, AVG(note) AS noteMoyenne, COUNT(*) AS nombre
FROM jeux
GROUP BY genre
ORDER BY noteMoyenne DESC;
```

Dans ce cas précis, MongoDB n'apporte rien par rapport à PostgreSQL qui est plus simple à prendre en main pour les requêtes d'agrégation. En revanche, MongoDB possède comme avantage d'être beaucoup moins rigide de PostgreSQL en ce qui concerne la structure des données. Cela lui permet notamment de gérer des enregistrements ayant des champs très différents les uns des autres sans pour autant à devoir les déclarer à l'avance dans une structure de table.

**Question 13 :** C'est la deuxième requête qui a raison parce que Krayz a mis une note de 5 étoiles sur aucun jeu, donc rien ne doit être retourné. La requête d cherche les jeux qui possèdent au moins un avis donné par "Krayz" et un avis de 5 étoiles, mais sans pour autant faire la liaison entre les deux. Stardew Valley possède un avis de 4 étoiles par Krayz et de 5 étoiles par Nova  : il remplit toutes les conditions pour être retourné, malgré le fait que l'avis à 5 étoiles n'ait pas été donné par Krayz. 

**Question 14 :** Au départ, l'enregistrement Stardew Valley faisait 273 octets. Après l'ajout des avis, il en fait 748. 748 - 273 = 475. Le champ avis de l'enregistrement fait 475 octets. 475/3 = 158. Un avis fait en moyenne 158 octets. 16 777 216 - 273 = 16 776 943 octets allouables aux avis. 16 776 943/158 = 106 183 avis possibles sur Stardew Valley. Il est à noter que le nombre d'avis rédigeables peut encore réduire si certains d'entre eux sont particulièrement longs et que le nombre d'octets allouables aux avis ne comptabilise pas d'autres champs qui pourraient être ajoutés à Stardew Valley (exemple : un champ "nb_achats"). Cela représente un vrai risque parce que même si 106 183 avis parrait être un grand nombre, les avis sur un jeu populaire s'accumulent très rapidement et la limite est vite dépassée.

**Question 15 :** ```$lookup```renvoie toujours un tableau parce qu'il peut exister des cas où plusieurs enregistrements sont référencés par un champ plutôt qu'un seul. Comme MongoDB ne connait pas à l'avance le nombre de documents référencés, il les liste tous dans un tableau. L'ajout de `{ $unwind: "$studio" }` déstructure le tableau comportant le studio de développement pour ne retourner qu'un dictionnaire avec les informations.

**Question 16 :** Pour l'expérience 1, il aurait fallu modifier les 40 jeux développés par le studio. Pour les expériences 2 et 3, MongoDB n'a rien signalé car il n'y a aucune erreur bloquante. Dans le cas de l'expérience 2, rien n'a été retourné car il y a sensibilité à la casse. Avec une contrainte de clé étrangère sur PostgreSQL, la suppression du studio aurait été impossible car une violation de clé étrangère aurait été levée. Donc dans pixelHub c'est l'application elle même qui est responsable de l'intégrité des références étant donné que la souplesse de MongoDB ne permet pas d'assurer cela.

