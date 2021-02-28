# 

<h1 align="center">
   CSI 2532 DEVOIR 1
</h1>

<p align="center">
  JACOB FODALE 300119336
</p>

# A : MODÈLE E-R



  ## A1 : RELATION, CARDINALITÉ & PARITCIPATION
  
  ### A
  
  ![A1 A](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/A1A.png?raw=true)
  
  ### B
  
  ![A1 B](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/A1B.png?raw=true)
  
  ### C
  
  ![A1 C](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/A1C.png?raw=true)
  

  ## A2 : CONCEPTION DE SYSTÈME
  
  ![A2](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/A2.png?raw=true)

  ## A3 : ALGÈBRE RELATIONNELLE
  
  ![image](https://user-images.githubusercontent.com/71846266/109406188-0c3d9980-7945-11eb-88e7-81938327a281.png)





# B : SQL

## B1 : LECTURE DE REQUÊTES SQL

### A


La première reqête s'exécute sans conflit. Voici la sortie:

![B1 A](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B1A.png?raw=true)

### B

La seconde reqête s'exécute pareillement. Voici la sortie:

![B1 B](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B1B.png?raw=true)


### C

La troisième requête ne s'exécute pas:

![B1 C Fonctinne pas](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B1C1.png?raw=true)

Le problème est qu'on produit le group  _users_19_ en utilisant le _id_ et le _name_ en tant que paramètre, mais la déclaration _GROUP BY_
situé au bas propose seulement un regroupement via _name_. On change donc la ligne pour inclure le _id_ aussi.

```sql
GROUP BY id, name
```

Conséquement, le code s'exécute et produit :

![B1 C Fonctinne ](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B1C2.png?raw=true)

Voici donc le code SQL complète:

```sql
WITH users_2019 (id, name) AS
 (SELECT *
 FROM users
 WHERE join_date BETWEEN '2019-01-01' AND '2019-12-31')
SELECT id,
 name,
 count(licenses.access_code) AS num
FROM users_2019
LEFT JOIN licenses ON licenses.user_id = id
GROUP BY id, name
ORDER BY num DESC;
```


## B2 : ÉCRITURE DE REQUÊTES SQL

### A

Pour satisfaire cette demande, il faut un SELECT avec une simple condition:

```sql

SELECT name FROM users WHERE join_date < '2020-01-01';

```

Voici le résultat :

![B2 A](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B2A.png?raw=true)

### B

Vu qu'il faut inclure toute les _users_ il faut un **SELECT** avec un **FROM** _users_. Il faut parcontre aussi accès au license. On utilise donc un **JOIN**.

Le join doit être **LEFT** puisqu'il faut inclure les _users_ qui n'ont pas de license. Finalement, on crée un group et un tire les résultat :

```sql
SELECT  name, COUNT(user_id) as num FROM users LEFT JOIN licenses ON users.id = licenses.user_id GROUP BY name, user_id ORDER by COUNT(user_id) DESC, name ASC;	
```
Voici le résultat :

![B2 B](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B2B.png?raw=true)

### C

Pour démontrer que les résultats fonctionne bien, j'insère un _user_ qui possède plus que 2 _license_ et un _user_ avec 0 _license_ :

```sql

INSERT INTO users (id, name, join_date)
		VALUES
		(52, 'Jacob', '2021-02-27'),
		(53, 'John', '2021-02-27');

```

J'accorde ensuite 3 license à John :

```sql
INSERT INTO licenses (user_id, software_name, access_code)
		VALUES
		(53, 'MS Word', 'klm101112'),
		(53, 'Chrome', 'password123'),
		(53, 'Sketch', 'x3y4z5');
```

Finalement, j'essaie le code de la question précédente :

```sql
SELECT  name, COUNT(user_id) as num FROM users LEFT JOIN licenses ON users.id = licenses.user_id GROUP BY name, user_id ORDER by COUNT(user_id) DESC, name ASC;	
```

Voici le résultat : 

![B2 C](https://github.com/jfoda041/csi2532_devoir1/blob/main/assets/B2C.png?raw=true)

### D

Un simple **UPDATE** devrait répondre à la demande:

``` sql
UPDATE softwares SET version= 51, released_date='2020-1-1' WHERE name = 'Sketch';
```



## B3 : MISE À JOUR DE SCHÉMA SQL

### A

Finalement le **UPDATE** n'était pas si simple. Il faut des migrations.

Au début, on ajoute la version du _software_ aux _licenses_ :

```sql

BEGIN;

   ALTER TABLE licenses ADD COLUMN software_version varchar(200);

COMMIT;

```

### B

Ensuite, on s'assure que la version est une clé primaire des _softwares_ pour qu'on peut avoir plusiers version d'un _software_ :

```sql

BEGIN;

   ALTER TABLE softwares DROP CONSTRAINT softwares_pkey;
   ALTER TABLE softwares ADD PRIMARY KEY (name, version);
   
COMMIT;

```

### C

Il faut aussi ajouter la version comme clé primaire des _licenses_ pour la même raison.

```sql

BEGIN;

   ALTER TABLE licenses DROP CONSTRAINT licenses_pkey;
   ALTER TABLE licenses ADD PRIMARY KEY (user_id,software_name,software_version);

COMMIT;

```

Voici un exemple d'insertion sql:

```sql
INSERT INTO licenses (user_id, software_name, access_code, software_version)
VALUES
 (48, 'Sketch', 'xxxyyy111', '52');
```

### D

Finalement on accorde le _software_ Sketch de _version_ 52 à tout les utilisateurs qui ne le possèdent pas en utilisant "1monthfree" :

``` sql

BEGIN;

   INSERT INTO licenses (user_id, software_name, access_code, software_version) 
   SELECT id, 'Sketch', '1monthfree', '52' FROM users ON CONFLICT DO NOTHING;

COMMIT;

```





