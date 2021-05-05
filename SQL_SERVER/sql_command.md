## Creation de base exemple

~~~~sql
CREATE DATABASE <name>;
GO
~~~~

## Supprimer une base

~~~~sql
DROP DATABASE <nom de la base>;
~~~~

## Creation de table exemple

~~~~sql
CREATE TABLE <name> (<colonne1> char(15),
                    <colonne2> varchar(20)
                    );
GO
~~~~

## Supprimer une table

~~~~sql
DROP TABLE <nom de la table>;
~~~~

## Ajouter une table dans une DATABASE

~~~~sql
USE <nom de la base>;
GO
~~~~

    <Creation de la table>

### Ajouer des donnée dans la table

~~~~sql
INSERT INTO <nom de la table> VALUES ('exemple1','exemple2');
~~~~

### Afficher les datas dans toute les colonnes d'une table

~~~~sql
USE <nom de la base>;
GO
~~~~

~~~~sql
SELECT * FROM <nom de la table>;
~~~~

## Créer un index

~~~~sql
CREATE INDEX <indexname> ON <tablename>
~~~~

### Créer un index cluster

~~~~sql
CREATE UNIQUE CLUSTERED INDEX <indexname> ON <tablename>
~~~~

### Créer un index non cluster

~~~~sql
CREATE NONCLUSTERED INDEX <indexname> ON <tablename>
~~~~

## Créer une table avec une clé primaire

    une clé primaire est la donnée qui permet d'identifier de manière unique un enregistrement dans une table.

~~~~sql
CREATE TABLE <tablename> (
    c1 <datatype> PRIMARY KEY,
    c2 <datatype>,
    c3 <datatype>,
);
~~~~

OU

~~~~sql
CREATE TABLE <tablename> (
    c1 <datatype>,
    c2 <datatype>,
    c3 <datatype>,
);
CONSTRAINT pk_primary_key PRIMARY KEY <colonnename>;
~~~~

## Supprimer une clé primaire

~~~~sql
ALTER TABLE <tablename>
DROP CONSTRAINT <primarykeyname>
~~~~

## Définir une clé secondaire

```sql
ALTER TABLE <tablename> CONSTRAINT fk_name FOREIGN KEY (<colonnename>) REFERENCES <tablename>(<colonnename>)
```

## Jointure 

### Définition
* INNER JOIN | jointure interne pour retourner les enregistrements quand la condition est vrai dans les 2 tables. C'est l'une des jointures les plus communes. 
* LEFT JOIN (ou LEFT OUTER JOIN) | jointure externe pour retourner tous les enregistrements de la table de gauche (LEFT = gauche) même si la condition n'est pas vérifié dans l'autre table.
* RIGHT JOIN (ou RIGHT OUTER JOIN) | jointure externe pour retourner tous les enregistrements de la table de droite (RIGHT = droite) même si la condition n'est pas vérifié dans l'autre table.
* FULL JOIN (ou FULL OUTER JOIN) | jointure externe pour retourner les résultats quand la condition est vrai dans au moins une des 2 tables.
* CROSS JOIN | jointure croisée permettant de faire le produit cartésien de 2 tables. En d'autres mots, permet de joindre chaque lignes d'une table avec chaque lignes d'une seconde table. Attention, le nombre de résultats est en général très élevé.

**cas particuliers** 
* SELF JOIN | permet d'effectuer une jointure d'une table avec elle-même comme si c'était une autre table.
* NATURAL JOIN | jointure naturelle entre 2 tables s'il y a au moins une colonne qui porte le même nom entre les 2 tables SQL
* UNION JOIN  | jointure d'union

**Sans intersection** | on va appliquer une clause WHERE pour exclure les intersections
* LEFT JOIN (sans intersection) | LEFT JOIN ... WHERE table.cle IS NULL
* RIGHT JOIN (sans intersection) | RIGHT JOIN ...WHERE table.cle IS NULL
* FULL JOIN (sans intersection) | FULL JOIN ... WHERE table1.cle IS NULL OR table2.cle IS NULL

### Exemple
```sql
USE MASTER
SELECT Table1.col1, Table1.col2, Table2.col1 FROM Table1
INNER JOIN Table2 ON Table1.col1 = Table2.col2
```

