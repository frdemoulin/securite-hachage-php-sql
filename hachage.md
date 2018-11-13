# Sécurité des mots de passe utilisateurs en PHP

[source 1](https://openclassrooms.com/fr/courses/2644516-securisez-les-mots-de-passe-des-utilisateurs-avec-php) - [source 2](https://www.culture-informatique.net/cest-quoi-hachage/)

## I. Nécessité de sécuriser les mots de passe

* Un utilisateur possède généralement un nombre limité de mot de passe, si faille sur un site, tous ses comptes utilisateurs sont compromis

* Déontologiquement, un webmaster se doit de protéger les données de ses utilisateurs

* Légalement, la CNIL impose une garantie de confidentialité des données utilisateur (un webmaster doit donc mettre en place des systèmes de protection des données utilisateurs)

## II. Chiffrement vs hachage

Ces deux notions vont être explicitées par la suite mais dès àà présent, retenons que cryptage et hachage, c’est différent.

* Le but du cryptage (ou encore chiffrement) est de coder des données pour les rendre illisibles. Mais c’est aussi de permettre à quelqu’un possédant la clé et la méthode de déchiffrement de pouvoir décoder et donc pouvoir accéder aux informations en clair

* Dans le hachage, on n'a pas accès aux données en clair. L’objectif est de leur associer une empreinte caractéristique

## III. Le chiffrement de données

### 1. Généralités

Le chiffrement permet de dissimuler des informations pour garantir leur sécurité. Il prend en compte trois éléments :

* **Le message en clair** : texte ou information à chiffrer ;

* **La clé** : particule qui va permettre d'avoir un résultat unique et qui permettra de déchiffrer l'information ;

* **L'algorithme** : manière dont on va traiter les deux premiers éléments pour générer l'information chiffrée.

[Ressource sur le chiffrement](https://www.culture-informatique.net/comment-ca-marche-le-chiffrement/)

_Illustration_ : Le message "Bonjour" est envoyé à l'algorithme "Arcfour Stream" qui va le chiffrer avec la clé "MaCle" et renvoyer le message chiffré "TKWbl/p3DA==". Pour décrypter ce message, il suffirait de l'envoyer à l'algorithme avec la même clé et il renverrait "Bonjour".

<p align="center">
  <img width=60% src="https://sdz-upload.s3.amazonaws.com/prod/upload/tuto_sdz.jpg">
</p>

### 2. En PHP

Il existe des [extensions PHP](http://php.net/manual/fr/security.database.storage.php) gérant le chiffrement des données telles [Mcrypt](http://php.net/manual/fr/ref.mcrypt.php) ou [Mhash](http://php.net/manual/fr/ref.mhash.php).

Exemples de fonctions de cryptage / décryptage avec Mcrypt (chacune de ses fonctions prend deux arguments : un message à chiffrer/déchiffrer et une clé ) :

```
<?php
// Source www.stackoverflow.com

function encrypt($pure_string, $encryption_key) {
    $iv_size = mcrypt_get_iv_size(MCRYPT_BLOWFISH, MCRYPT_MODE_ECB);
    $iv = mcrypt_create_iv($iv_size, MCRYPT_RAND);
    $encrypted_string = mcrypt_encrypt(MCRYPT_BLOWFISH, $encryption_key, utf8_encode($pure_string), MCRYPT_MODE_ECB, $iv);
    return $encrypted_string;
}

function decrypt($encrypted_string, $encryption_key) {
    $iv_size = mcrypt_get_iv_size(MCRYPT_BLOWFISH, MCRYPT_MODE_ECB);
    $iv = mcrypt_create_iv($iv_size, MCRYPT_RAND);
    $decrypted_string = mcrypt_decrypt(MCRYPT_BLOWFISH, $encryption_key, $encrypted_string, MCRYPT_MODE_ECB, $iv);
    return $decrypted_string;
}
?>
```

En pratique, ces fonctions chiffrent leurs arguments avec l'algorithme Blowfish en mode ECB (ce mode découpe le message en blocs et chiffre chacun de ces blocs les uns après les autres).

Dans un script PHP, l'appel à ces fonctions se fait ainsi (activation requise de l'extension Mcrypt dans PHP) :
```
<?php
$messageAChiffrer = "Coucou je suis Maurice";
$cleSecrete = "MaCleEstIncassable";


// On chiffre le message
$messageChiffre = encrypt($messageAChiffrer, $cleSecrete);

// Pour le lire
$messageDechiffrer = decrypt($messageChiffre, $cleSecrete);
?>
```

## IV. Le hachage de données

### 1. Généralités

Le hachage permet de vérifier l'intégrité d'une information sans avoir à vérifier toute cette information.

En anglais : hash signifie pagaille, désordre, recoupe et mélange (par analogie avec la cuisine)

### 2. Sans hachage, la mauvaise pratique

_Exemple de mauvaise pratique dans le cas d'un système de connexion à un espace membres :_ les mots de passe sont enregistrés en clair dans la bdd et sont facilement lisibles par toute personne pouvant accéder à cette dernière

<p align="center">
  <img width=90% src="https://www.culture-informatique.net/WordPress3/wp-content/uploads/2016/05/pas-hachage.png">
</p>

### 3. Fonction de hachage

#### a. Définition

Une **fonction de hachage** est une fonction qui, à partir d'une donnée fournie en entrée, calcule une empreinte (ou signature) servant à identifier rapidement, bien qu'incomplètement, la donnée initiale.

_Illustration_ : Le message "Bonjour" est envoyé à l'algorithme de la fonction de hachage "md5" qui va le transformer avec perte de données en le formatant en une chaîne de 32 caractères

<p align="center">
  <img width=70% src="https://sdz-upload.s3.amazonaws.com/prod/upload/tuto_sdz_2.jpg">
</p>

_Exemple de hachage dans le cas d'un système de connexion à un espace membres :_ à l'enregistrement d'un nouveau compte utilisateur, le client ne va pas envoyer le mot de passe au serveur, mais une empreinte du mot de passe. Le serveur ne va pas enregistrer le mot de passe mais enregistrera son empreinte. Lorsque l’utilisateur cherchera à se connecter, le serveur n'ira pas vérifier si le mot de passe est identique à celui stocké en base, mais il vérifiera que l'empreinte du mot de passe saisi est bien la même que l'empreinte du mot de passe enregistré

<p align="center">
  <img width=90% src="https://www.culture-informatique.net/WordPress3/wp-content/uploads/2016/05/avec-hachage.png">
</p>

#### b. Propriétés

* La longueur de l'empreinte doit être toujours la même (quelque soit la longueur des données en entrée). La longueur d'une empreinte dépend de la fonction de hachage utilisée ;

* Il n’est _pas possible de trouver les données originales à partir des empreintes_ : les fonctions de hachage ne fonctionnent que dans un seul sens.

* Il ne doit pas être possible de prédire une empreinte (il n’est pas possible d’essayer d’imaginer ce que pourrait être l'empreinte en examinant les données) ;

* Pour des données différentes, les empreintes doivent être différentes.

#### c. Exemples de hachage (générées avec md5)

| Données en entrée              | Empreinte |
| :----------------------------- | :-------- |
| 123                            | 202cb962ac59075b964b07152d234b70 |
| 124                            | c8ffe9a587b126f152ed3d89a146b445 |
| 1234                           | 81dc9bdb52d04dc20036dbd8313ed055 |
| 123456789012345678901234567890 | a46857f0ecc21f0a06ea434b94d9cf1d |
| abcde                          | ab56b4d92b40713acc5af89985d4b786 |
| abcdef                         | e80b5017098950fc58aad83c8c14978e |

#### d. Utilité du hachage

Le hachage est utilisé pour comparer des données (en comparant leurs signatures).

Le hachage met en effet en avant les empreintes des données plutôt que les données elles-mêmes. Chaque donnée ayant sa propre signature :

* si les signatures sont identiques, alors les données le sont également ;

* inversement, si des signatures sont différentes, alors les données le sont également.

#### e. Exemples d'utilisations concrètes de fonctions de hachage

* Comparaison de mots de passe (cf. schéma précédent, schéma incomplet car manque le sel à l’enregistrement du mot de passe, sel = chaîne complémentaire)

* Vérification des données téléchargées. Par exemple, dans certains cas, lorsque vous téléchargez un fichier sur Internet la signature du fichier original est disponible (soit dans un fichier à part, soit directement affichée sur le site). Pour être sûr que le fichier a été correctement téléchargé, il vous suffit de vérifier que la signature du fichier téléchargé est bien identique à celle fournie

* La signature électronique (ou numérique) d'un document. Le document envoyé est passé dans une fonction de hachage et l’empreinte est envoyée chiffrée (avec le mécanisme de clés asymétriques) en même temps que le document. Il suffit au destinataire de déchiffrer l’empreinte reçue et de vérifier que celle-ci correspond bien au calcul de l’empreinte des données reçues

* Pour l’utilisation de tables de hachage, utilisées en développement des logiciels

* Pour le stockage des données : si la signature d’un fichier est présente, alors on peut dire que le fichier est présent. Il n’est pas nécessaire de l’enregistrer à nouveau. Cela peut être intéressant dans le cas de gros fichier (dans ce cas, il faudra beaucoup moins de temps pour calculer sa signature que de l’enregistrer à nouveau)

#### f. Et si des données ont la même empreinte ?

Quand des données produisent la même empreinte, on dit que l'on a affaire à une **collision**. Ce n’est généralement pas problématique car il ne s’agit que de contrôles : on contrôle que l’empreinte des données reçues est la même que l’empreinte des données envoyées.

Cela peut être plus gênant pour les mots de passe car deux mots de passe différents produisant la même empreinte permettront de s’identifier ! (c’est une des raisons pour laquelle on ajoute du « sel » au mot de passe, mais ce n’est pas la seule raison).

C’est carrément très gênant pour les tables de hachage ou pour les stockages de données. C’est pourquoi il est conseillé d’utiliser des fonctions de hachage avec de faibles taux de collisions.

#### g. Exemples de fonctions de hachage

**md5** - Cette fonction de hachage est toujours très utilisée bien qu’au niveau de la sécurité, il est recommandé de passer à des versions plus robustes car des suites de collisions ont été trouvées. Cette fonction renvoie une empreinte de 128 bits (ie 16 caractères)

/!\ On peut tenter de "décrypter" une empreinte md5 à l'aide de bases de données stockant l'association mot en clair - empreinte md5 comme ici ([lien décrypteur md5](https://www.md5online.fr/))

**SHA1** - Était la fonction remplaçante de md5 car elle produisait des empreintes 160 bits (20 caractères) et avec impossibilité de trouver des collisions... jusqu’en 2004-2005, date à laquelle des attaques ont prouvé des possibilités de générer des collisions. Depuis cette date, il n’est plus conseillé d’utiliser la fonction SHA1. Mais, elle l'est encore. Les certificats numériques utilisant SHA1 ne sont plus plus valides au 31 décembre 2016

**SHA2** - SHA256 et SHA512 sont deux des grands standards utilisés actuellement, car il n’y a pas à ce jour d’attaques ayant trouvé des failles de sécurité sur ces fonctions de hachage. Elles produisent des signatures de respectivement 256 (32 caractères) et 512 bits (64 caractères)

Ressource : [convertissseur bits <-> caractères](http://extraconversion.com/fr/stockage-de-donnees/caracteres/caracteres-en-bits.html)

## V. Application en PHP

### 1. La fonction password_hash()

Les algorithmes de hash sont implémentés dans PHP via la fonction [password_hash()](https://secure.php.net/manual/fr/function.password-hash.php). Celle-ci crée un nouveau hachage en utilisant un algorithme de hachage fort et irréversible.

_Exemple :_ hash de la string 'Bonjour' avec l'option PASSWORD_DEFAULT ( utilisation de l'algorithme bcrypt)

```
<?php
$empreinteDeBonjour = password_hash('Bonjour', PASSWORD_DEFAULT);
?>
```

### 2. Une utilisation pratique des hash en PHP

On peut s'en servir dans les systèmes d'authentification :

<p align="center">
  <img width=90% src="https://sdz-upload.s3.amazonaws.com/prod/upload/tuto_sdz_31.jpg">
</p>

**Avantage :** le mot de passe n'est pas stocké en clair dans la bdd.

**Inconvénient :** pas assez sécurisé du fait des collisions possibles selon la fonction de hachage

### 3. La sécurité d'un hash

Comme vu ci-dessus, si l'on possède l'empreinte d'un hash, on peut éventuellement remonter à la donnée en clair via des bases de données en ligne générées à partir d'attaques dictionnaire ou bruteforce ([lien décrypteur md5online](https://www.md5online.fr/)).

### 4. Les grains de sel (salt)

Pour éviter la problématique précédente, on utilise un **salt**. C'est un préfixe ou un suffixe que l'on ajoute au mot de passe en clair avant de le hacher, diminuant ainsi la probabilité que cette entrée ait été indexée dans une base de données telle md5online.

_Exemple :_ string 'Bonjour' à laquelle on a apposé le salt `1t5MyH45h` en préfixe, ce qui donne '1t5MyH45h_Bonjour'.

Fonctionnement du système d'enregistrement avec salt statique :

<p align="center">
  <img width=90% src="https://sdz-upload.s3.amazonaws.com/prod/upload/tuto_sdz_51.jpg">
</p>

Il est alors plus difficile de décrypter un tel mot de passe à partir de son empreinte, mais ce n'est pas impossible pour autant (bruteforce attaque). D'où l'idée de générer un salt de façon dynamique...

### 5. Un salt dynamique

Afin de limiter les failles par bruteforce, on utilise un salt différent (= dynamique) pour chaque utilisateur. Cela peut être son pseudo chiffré par un algorithme et une clé commune à tous les utilisateurs (/!\ ne pas utiliser l'id de l'utilisateur, trop commun !!!)

Fonctionnement du système d'enregistrement avec salt dynamique :

<p align="center">
  <img width=90% src="https://sdz-upload.s3.amazonaws.com/prod/upload/tuto_sdz_61.jpg">
</p>

Dans ce contexte, le fichier .htaccess est là pour vérifier que seule l'application puisse accéder au mot de passe contenu dans le fichier qu'il sécurise.

Avec ce système, si un hacker récupère notre bdd cherchant à récupérer les mots de passe utilisateurs, il devrait attaquer chaque salt par bruteforce pour chaque utilisateur. Dans le meilleur des cas (pour lui !), il ne pourrait récupérer que les mots de passe faibles. Ce vecteur d'attaque est trop gourmand en temps et en énergie, le hacker aura donc plutôt tendance à essayer une attaque par bruteforce directement sur le login du site... Mais dans ce cas, la responsabilité revient à l'utilisateur qui a un mot de passe trop faible et non pas à l'administrateur du site qui n'a pas subi de fuite de données.

Avec un salt dynamique, tout est sécurisé par le mot de passe se trouvant sous la protection du .htaccess. Il est donc possible de partager le code source de l'application car tant que le mot de passe, lui, n'est pas partagé, le tout reste sécurisé.