
l**Auteur : Valentin Minnebo**

---

**Question 1 :**
Les autres services implémentent chacun une famille de BDD différente. Chacune des familles sera utile car elles permettent d'implémenter des fonctionnalités pas forcément réalisables avec les autres.

**Question 2 :**
C'est le service redis qui n'a pas de section volumes. Cela implique que ses données ne seront pas conservées après fermeture du conteneur.

**Question 3 :**
Il s'agit des ports utilisés par l'application. 15432 est le port associé à la machine de l'utilisateur, et 5432 est le port d'écoute du conteneur.

**Question 4 :**
Cela n'est pas acceptable et ne sera surtout pas fait sur un serveur de production.

**Question 5 :**  Cela exécute dans les conteneurs les commandes spécifiées après `docker compose exec`. La commande `redi-cli`s'exécute dans le conteneur redis.

**Question 6 :** Cela fonctionne car le port 15432 de la machine hôte (localhost) est redirigé vers le port 5432 du conteneur Postgres.

**Question 7 :** Les joueurs ne sont pas dupliqués en base de données. Cela est du à ce morceau de code dans `Programs.cs`:

```csharp
if (!db.Players.Any())
{
    db.Players.AddRange(
        new Player { Pseudo = "Nova",  Coins = 1200 },
        new Player { Pseudo = "Krayz", Coins = 350  },
        new Player { Pseudo = "Ombre", Coins = 90   }
    );
    db.SaveChanges();
    }
```

La ligne `if (!db.Players.Any())` permet de vérifier si il existe déjà des données dans la table `Players`. Si c'est le cas, les joueurs ne sont pas ajoutés à nouveau en base de données. Et comme les données du conteneur sont enregistrées grâce à la présence d'un volume, l'arrêt puis le relancement de l'application ne causera pas une insertion supplémentaire de joueurs.

**Question 8 :** La valeur de `player.Id` vaut 4 avant l'appel de `SaveChangesAsync()`. C'est parce que cette valeur correspond en table à une clé primaire incrémentée automatiquement par PostgreSQL.

**Question 9 :** Les soldes reprennent leur valeur de départ après le ROLLBACK. Si le serveur s'éteint entre les deux updates, sans transaction, cela aura le même effet qu'un ROLLBACK, ce qui signifie que les modifications ne seront pas enregistrées en base de données, et donc Krayz n'obtiendra pas ses pièces supplémentaires.

**Question 10 :** B et l'API voyaient le solde original (non modifié) pendant la transaction de A. L'update de B a du attendre parce que la transaction de A était toujours en cours, il était nécessaire de la terminer avant. Les deux achats lancés en même temps sur deux téléphones causeraient des dysfonctionnements critiques, comme par exemple un solde de produits négatif.

**Question 11 :** C'est la BDD qui a refusé l'achat d'Ombre. Le SELECT a également échoué car l'invite de commande ne permet pas l'exécution d'autres instructions suite à une erreur dans la transaction. Après le ROLLBACK, la contrainte n'existe plus. On peut en conclure que le système de transaction de PostgreSQL permet d'assurer au maximuml'intégrité des données en base.

**Question 12 :** `TTL temporaire` renvoie le nombre de secondes qu'il reste avant que la valeur de la variable `temporaire`ne soit effacée. `GET temporaire`renvoie la valeur de la variable `temporaire`. Les données qui pourraient être supprimées après un certain laps de temps sont les données de matchmaking et les données en cache.

**Question 13 :** Dans le cas où 200 joueurs se connectent en même temps, cela causerait une surcharge de lecture d'une même variable avec `SET`, et la valeur finale serait incorrecte à cause de la réecriture massive en un temps limité. L'atomicité de la fonction INCR empêche ce genre de complications.

**Question 14 :** Il n'aurait pas été possible de ranger ces deux jeux dans une même table PostgreSQL. Plus précisément, cela aurait été possible, mais en ignorant les enregistrements 'multijoueur' et 'plateformes'. Il aurait fallu créer 2 nouvelles tables avec un id de jeu comme clé primaire/étrangère.

**Question 15 :**

```sql
SELECT p1.Pseudo FROM "Players" as p1
INNER JOIN "Amities" as a1 on p1.Id = a1.joueur_id
INNER JOIN "Amities" as a2 on a1.ami_i = a2.joueur_id
INNER JOIN "Players" as p2 on a2.ami_id = p2.Id
WHERE p2.Pseudo = 'Ana';
```

Il aurait donc fallu 3 INNER JOIN pour rendre cette requête réalisable sur PostgreSQL.

**Question 16 :** Le DELETE a été refusé car les noeuds à supprimer possédaient des relations avec d'autres noeuds : cela aurait causé des violations de contraintes. Le DETACH assure la suppression de ces noeuds en retirant également les relations qui y sont associées.

**Question 17 :** La clé 'survivant' a survécue au stop, mais pas au down. Les joueurs de PostgreSQL ont survécus dans tous les cas. Un volume permet d'enregistrer les données en base, même après suppression de son conteneur. Celui de redis ne possède pas de volume, ses données ne sont donc pas conservées après suppression.