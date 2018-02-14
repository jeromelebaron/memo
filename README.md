# Mémo

## Tables des matières

- [Java](#Java)
- [Git](#Git)
- [Apache](#Apache)
- [Maven](#Maven)
- [Architecture](#Architecture)
- [Déploiement](#Deploiement)
- [SQL](#SQL)
- [Docker](#Docker)
- [JavaScript](#JavaScript)
- [Bash](#Bash)

## Java

### Configuration

- changer de version rapidement sur les systèmes UNIX (en éditant le `.profile`)

```bash
alias j9="export JAVA_HOME=`/usr/libexec/java_home -v 9`; java -version"
alias j8="export JAVA_HOME=`/usr/libexec/java_home -v 1.8`; java -version"
alias j7="export JAVA_HOME=`/usr/libexec/java_home -v 1.7`; java -version"
```

### Java

#### Liens utiles

- [Design patterns](http://java-design-patterns.com/)

#### Lambda

- pour l'utilisation des lambdas de type `Consumer<T>` (équivalent du setter) il faut utiliser l'instance avec `::`. `objet::setter`
- ne pas hésiter à utiliser les fonctions en paramètre pour factoriser un code qui aurait un pattern commun. Exemple :

	```java
	// On note un pattern commun sur ces 3 conditions mais ce sont les setter qui changent
	if (dossier.getTuteurPedagogique() != null) {
	    final Contact contactTuteur = dossier.getTuteurPedagogique().getContact();
	    bandeauLSADTO.setNomPrenomTuteur(ArcaniaStringUtils.construireNomPrenom(contactTuteur));
	    bandeauLSADTO.setEmailTuteur(contactTuteur.getEmail());
	}
	if (dossier.getMA1() != null) {
	    final Contact contactMA = dossier.getMA1().getContact();
	    bandeauLSADTO.setNomPrenomMA(ArcaniaStringUtils.construireNomPrenom(contactMA));
	    bandeauLSADTO.setEmailMA(contactMA.getEmail());
	}
	if (dossier.getMA2() != null) {
	    final Contact contactMA = dossier.getMA2().getContact();
	    bandeauLSADTO.setNomPrenomMA2(ArcaniaStringUtils.construireNomPrenom(contactMA));
	    bandeauLSADTO.setEmailMA2(contactMA.getEmail());
	}
	// On va donc le factoriser en passant une fonction comme paramètre
	/**
	 * Pour traiter un {@link ContactApprenti} du {@link BandeauLSADTO}.
	 * @param contactApprenti le {@link ContactApprenti} à traiter.
	 * @param setNomPrenom la fonction pour le nom prénom à setter.
	 * @param setEmail la fonction pour le mail à setter.
	 */
	private void traiterContactApprenti(final ContactApprenti contactApprenti, final Consumer<String> setNomPrenom,
	                                    final Consumer<String> setEmail) {
	    if (contactApprenti != null) {
	        final Contact contact = contactApprenti.getContact();
	        setNomPrenom.accept(ArcaniaStringUtils.construireNomPrenom(contact));
	        setEmail.accept(contact.getEmail());
	    }
	}
	```
- les lambda permettent de facilement faire des concaténations de chaines. Par exemple lorsqu'on a une liste d'éléments et que l'on veut les concaténer avec une délimiteur :
	
	```java
	List<String> tutorialList = Arrays.asList("Spring MVC", "Java", "NodeJS", "C Sharp");
	// On utilise un stream puis le collector avec la méthode joining 
	String tutorials = tutorialList.stream().map(crunchy -> crunchy).collect(Collectors.joining(", "));
	// Le résultat est le suivant : Spring MVC, Java, NodeJS, C Sharp
	```
	Ce qui est intéressant c'est que le dernier élément n'aura pas le délimiteur. On peut aussi utiliser une autre méthode (qui aura le même effet):
	
	```java
	String tutorials2 = String.join(", ", tutorialList);
	```
	Ce qui est intéressant avec le stream c'est que si c'est une liste d'objets on peut accéder aux propriétés avec `crunchy -> crunchy.getName()`.

### Talend

- lorsque l'on fait une jointure avec plusieurs plusieurs composants de type `input`, il faut cliquer sur la clé du `trap` pour avoir accès à différentes options. On peut choisir le type de jointure (inner join, left outer ...) ainsi que les nombres d'éléments qui seront concernés par la jointure (le premier, tous ...)
- pour avoir la récupération automatique du schéma lors d'un `sqlOutput` il faut choisir le schéma en référentiel et en sélectionner un dans le contexte. Il faut ensuite choisir la table et cliquer sur `Guess Query`. Il va scanner par rapport au schéma et produire le select en incluant le contexte. Une fois cette opération faite, quand on aura relié le `trap`, celui-ci va deviner automatiquement les colonnes de la table en entrée et en sortie.
- pour les champs json il faut appliquer une transformation spéciale. Il faut aller dans le composant de sortie puis dans les réglages avancés. On ajoute ensuite une colonne, on la nomme avec les `""` puis dans expression SQL on ajoute `"?::json"` , position Remplacer et enfin la colonne de destination

### XML

- lors d'un export XML de cette façon :

	```java
	import java.io.ByteArrayInputStream;
	import java.io.ByteArrayOutputStream;
	import java.io.InputStream;
	
	import javax.xml.transform.OutputKeys;
	import javax.xml.transform.Transformer;
	import javax.xml.transform.TransformerException;
	import javax.xml.transform.TransformerFactory;
	import javax.xml.transform.dom.DOMSource;
	import javax.xml.transform.stream.StreamResult;
	import javax.xml.transform.stream.StreamSource;
	
	public class ExportXML {
		/**
	     * Pour écrire le contenu du {@link Document} dans un fichier XML.
	     * @param document le {@link Document} avec le contenu.
	     * @return le fichier sous forme de stream.
	     * @throws TransformerException en cas d'erreur d'écriture dans le fichier.
	     */
		public InputStream ecrireContenuFichierXML(final Document document) throws TransformerException {
		    final TransformerFactory transformerFactory = TransformerFactory.newInstance();
		    final Transformer transformer = transformerFactory.newTransformer();
		    final DOMSource source = new DOMSource(document);
		    final ByteArrayOutputStream dos = new ByteArrayOutputStream();
		    final StreamResult result = new StreamResult(dos);
		    transformer.setOutputProperty(OutputKeys.INDENT, "yes");
		    transformer.setOutputProperty("{http://xml.apache.org/xslt}indent-amount", "4");
		    transformer.transform(source, result); // Erreur ici
		    return new ByteArrayInputStream(bos.toByteArray());
		}
	}
	```

	si il y a une erreur de type :

	```java
	ERROR:  ''
	javax.xml.transform.TransformerException: java.lang.NullPointerException
	at com.sun.org.apache.xalan.internal.xsltc.trax.TransformerImpl.transform(TransformerImpl.java:716)
	at com.sun.org.apache.xalan.internal.xsltc.trax.TransformerImpl.transform(TransformerImpl.java:313)
	...
	```
	alors le problème vient d'un objet `null` lors d'une insertion dans une balise XML car la transformation exécute cette méthode (qui explique le nullpointer) :

	```java
	public void characters(String chars) throws SAXException {
		final int length = chars.length(); // NullPointer ici
	}
	```

	la solution est la suivante : ne pas ajouter `null` dans un noeud

	```java
	/**
	 * Pour ajouter du contenu dans un {@link Element}.
	 * @param document le {@link Document} pour créer.
	 * @param contenuBalise le contenu à ajouter.
	 * @param element l'{@link Element} pour lequel ajouter.
	 */
	default void ajouterContenu(final Document document, final Element element, final String contenuBalise) {
	    Text noeudRole = document.createTextNode("");
	    if (contenuBalise != null) {
	        noeudRole = document.createTextNode(contenuBalise);
	    }
	    element.appendChild(noeudRole);
	}
	```

### Hibernate/JPA

- si une erreur de ce type se présente `Caused by: org.hibernate.PersistentObjectException: detached entity passed to persist` c'est qu'un objet (lié le plus souvent) que l'on essaye de persister en cascade n'est pas attaché à la session hibernate. Pour palier au problème, il faut d'abord récupérer cet objet via hibernate et le problème devrait disparaitre. 
- attention lorsqu'on passe une liste pour un paramètre de type IN écrite en HQL, si la liste est vide alors cela va créer une erreur de syntaxe à cause de la parenthèse. Pour pallier le problème, il faut ajouter une valeur dans la liste.
- si une erreur de ce type se présente `illegal attempt to dereference collection` c'est qu'une collection présente dans la requête n'est pas peuplée. Il faut utiliser `JOIN FETCH` sur la collection pour contouner le problème.

### Spring

## Git

- pour supprimer un fichier sur le repository mais le garder en local (penser à l'ajouter dans le `.gitignore` ensuite) : 

	```git 
	git rm --cached -r somefile.ext
	```
- pour commiter an ajoutant tous les fichier puis en mettant le message

	```git
	git commit -am "Mon texte de commit"
	```

- en cas d'erreur de type `error: bad signature fatal: index file corrupt`, c'est seulement l'index qui est corrompu. Il faut faire les commande suivantes :

	```bash
	rm -f .git/index
	git reset
	```

- en cas d'erreur de type `error: inflate: data stream error (unknown compression method)` il faut faire les manipulations suivantes :
	- sauvegarder le fichier `.git`
	- faire la commande suivante `git fsck --full`
	- ensuite il va afficher le fichier corrompu
	- on le supprime avec la commande `rm -f .git/objects/56/d6c280ec4a5c91f442a23e6de7d8b425fb4623`
	- puis on répète l'opération jusqu'à ce que `git fsck --full` fonctionne
	- ensuite on récupère le nom des fichiers corrompus `git fsck --name-objects`
	- on les sauvegarde
	- on reset avec le remote `git reset --hard origin/votre-branche`
	- on copie colle à la main les nouveaux fichiers

## Apache

- configuration Apache 2.4 pour ajouter un virtual host :

		<VirtualHost *:80>
   	 		ServerAdmin webmaster@localhost
    		ServerName localhost
    		ServerAlias localhost
    		ProxyPreserveHost On
    		ProxyRequests Off
    		ProxyVia off
    		AcceptPathInfo On
    		RewriteEngine On
    
			Options FollowSymLinks
    		ProxyPass /api http://localhost:8080/api
    		ProxyPassReverse /api http://localhost:8080/api
    
    		ProxyPass / http://localhost:8888/
    		ProxyPassReverse / http://localhost:8888/
    
    		ErrorLog logs/arcania-error.log
			# Possible values include: debug, info, notice, warn, error, grit,
			# alert, emerge.
			LogLevel error
			CustomLog logs/arcania-access.log combined
		</VirtualHost>

## Maven

## Architecture

## Déploiement

- Outil pour déployer son environnement local sur une url publique [Blog netapsys](https://blog.netapsys.fr/exposer-son-environnement-local-rapidement-avec-ngrok/)

## SQL

- pour faire un `update` sur une sélection de lignes et non pas sur toute la table :

	```sql
	UPDATE contact_sites AS cs 
	SET id_site = s.id_etablissement
	FROM sites AS s
	WHERE cs.id_site = s.id and s.id_etablissement IS NOT null;
	```

- pour faire un `insert` dans une table sur une sélection d'éléments :

	```sql
	INSERT INTO table1 (column1, column2, someInt, someVarChar)
	SELECT table2.column1, table2.column2, 8, 'some string etc.'
	FROM table2
	WHERE table2.ID = 7;
	```	

### Postgre SQL

#### Dump 

- format backup

	```bash
	pg_dump --host localhost --port 5432 --username "arcane" --role "arcane" --no-password  --format tar --encoding UTF8 --verbose --file "D:\Projets\Arcania\Sprints\arcania-local-liquibase-20171127.backup" --table "public.databasechangelog" --table "public.databasechangeloglock" "arcane"
	```

- format plat

	```bash
	pg_dump --host localhost --port 5432 --username "arcane" --role "arcane" --no-password  --format plain --data-only --encoding UTF8 --verbose --file "D:\Projets\Arcania\Sprints\arcania-local-liquibase-201711271701.backup" --table "public.databasechangelog" --table "public.databasechangeloglock" "arcane"
	```

- pour pouvoir un drop des table avant le backup il suffit d'ajouter l'option ``-c`` à une commande

#### JSON

- postgreSQL permet d'avoir des champs json, voici la manière de faire des requêtes :

	```sql
	-- Ici on vérifie que la propriété suiviParticulier à la valeur false dans la colonne json formation
	SELECT *
	FROM dossiers
	WHERE formation ->> 'suiviParticulier' = 'true';
	```
- pour ajouter un champ json à un colonne existante il faut écrire un script comme ceci :

	```sql
	-- Ajout des nouvelles données naf du bloc employeur du cerfa pour un code naf en correspondance
UPDATE contrats c
SET employeur = employeur || jsonb_build_object('naf', json_build_object('id', r.id, 'code', r.code, 'libelle', r.libelle))
FROM referentiels r
WHERE c.employeur ->> 'codeNaf' = r.code
AND r.type = 'NAF';
	```
	Les `||` permettent de dire qu'il faut concaténer avec les éléments existants et ne pas remplacer. La fonction `jsonb_build_object(clé, valeur)` permet facilement de construire la paire clé/valeur avec en première valeur la clé et ensuite après une virgule la valeur et ainsi de suite pour autant de champs que désiré.

- pour renommer un champ json le plus simple est de créer un nouveau champ puis de supprimer l'ancien :

	```sql
	```

## Docker

- sur windows attention à la configuration des volumes docker qui sont généralement sur le disque d'installation. Pour pouvoir changer l'emplacement, il faut passer par le gestionnaire Hyper-V de windows.
- pour démarrer un container il faut lancer la ligne de commande suivante `docker-compose -f openldap.yml up -d`

## Javascript

### Langage

- pour vérifier qu'un objet n'est pas null, undefined, ou vide il suffit de faire un simple if, si il est une de ces trois conditions, on aura `false` sinon on aura `true`

	```javascript
	var vide = "";
	var nullable = null;
	var nonDefini = undefined;
	var nonNull = "non null"
	
	if (!vide) {
	    alert('Je suis vide');
	}
	if (!nullable) {
	    alert('Je suis null');
	}
	if (!nonDefini) {
	    alert('Je suis undefined');
	}
	if (nonNull) {
	    alert('Je suis ne suis pas vide');
	}
	```

### AngularJS

#### Liens utiles

- [guidelines Jon Papa](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md)
- [Tutoriel Grafikart](https://www.youtube.com/watch?v=cEJ7NVXCYsU&t=1943s)

#### Directives

- les différents éléments courants de configuration sont les suivants :

```javascript
(function() {
    'use strict';

    angular
        .module('core')
        .directive('inscriptionCampagne', inscriptionCampagne);

    /* @ngInject */
    function inscriptionCampagne() {
        return {
            scope: {
            },
            bindToController: {
                contactsSelectionnes: '='
            },
            restrict: 'E',
            replace: true,
            templateUrl: 'app/core/templates/inscriptionCampagne.tpl.html',
            controller: function (blockUI, notificationService, AutocompleteService, CampagneEditionService) {

                var vm = this;
                // Attributs publics
                vm.nomCampagne = '';
                vm.campagne = null;
                // Méthodes publiques
                vm.autocompleteCampagne = autocompleteCampagne;
                vm.inscrireAUneCampagne = inscrireAUneCampagne;

                /** Pour l'auto-complétion de la campagne. */
                function autocompleteCampagne(query) {
                    return AutocompleteService.autocompleteCampagne(query).then(function (response) {
                        return response;
                    });
                }

                /** Pour inscrire une liste de contact à une campagne. */
                function inscrireAUneCampagne() {
                    blockUI.start();
                    CampagneEditionService.copierVersCampagne(vm.campagne.id, vm.contactsSelectionnes).then(function () {
                        notificationService.success('message.succes.campagne.inscrire');
                    }).finally(function () {
                        blockUI.stop();
                    });
                }
            },
            controllerAs: 'vm'
        };

    }
})();
```

- le nom de la directive en camelCase sera transformé avec des tirets; ici `inscriptionCampagne` devient `inscription-campagne` (il en est de même pour les paramètres)
- soit on utilise `scope` soit `bindToController` pour le passage de paramètre sachant que `bindToController` est conseillé
- restrict est la façon dont on va utiliser la directive dans le code, la plus simple étant `'E'` je trouve pour passer des paramètres
	- `'E'` pour element `<my-loading title="Paramètres">...</my-loading>` 
	- `'A'` pour attribut `<div my-loading="Paramètres">...</div>` 
	- `'C'` pour classe `<div class="my-loading:Paramètres">...</div>` 
	- `'M'` pour commentaires  `<!-- directive: my-loading Paramètres -->`
- au niveau des paramètres, il y a trois possibilités et une chose à savoir :
	- `'@'` qui passe la valeur comme une chaine de caractères (utile pour les libellés, les id, les class ...)
	- `'='` qui passe un objet avec le binding habituel angular
	- `'&'` qui permet de passer une fonction
	- si on ne suffixe pas le type alors le nom à utiliser au moment de la déclaration sera le même, si on veut modifier cette valeur il suffit de faire comme suivant :
		
		```javascript
		bindToController: {
            contactsSelectionnes: '=contactsAAfficher'
        },
		```
		On utilisera donc le paramètre de la façon suivante : `contacts-a-afficher="..."`

### Angular 4

- pour faire une conversion de `string` en `number` il suffit simplement de préfixer la variable `string` par `+`
	
	```javascript
	const id = +this.route.snapshot.paramMap.get('id');
	``` 

## Bash

### Configuration UNIX

- pour avoir des raccourcis et autres liés à sa session, il faut éditer le fichier `.bash_profile` ou `.profile` qui va contenir tous ces éléments
- faire un alias :

	```bash
	alias ll='ls -l'
	```

- variable d'environnement :

	```bash
	# JAVA HOME PATH
	export JAVA_HOME=$TOOLS_DEV/Java/Contents/Home
	```

- path complet :

	```bash
	# FINAL PATH
	export PATH=$PATH:$JAVA_HOME/bin:$M2_HOME/bin:$MYSQL/bin:$SONAR_RUNNER_HOME/bin
	```

- faire un lien symbolique

	```bash
	ln -s origine destination
	```

- pour ajouter un nouvel utilisateur en tant que sudoer `sudo adduser <username> sudo` ou modifier le fichier `/etc/sudoer` en ajoutant une ligne `user ALL=(ALL) ALL`
- pour savoir ou on se trouve : `pwd`

### Configuration OSX

- ne plus prend en compte la casse pour l'auto-complétion dans le terminal : add `'set completion-ignore-case on'` to `~/.inputrc` (create file if necessary) to make bash tab-completion case-insensitive, like the Mac file systems (HFS+ and APFS)

## CSS/SCSS

- pour factoriser un pattern il faut utiliser les `@mixin`, exemple :

```scss
@mixin habillage-tableau($top-color, $color, $hover-color) {
    > thead {
        border-top-color: $top-color;
    }
    > tbody {
        > tr {
            > td.picto {
                i {
                    color: $color;
                    &:hover {
                        color: $hover-color;
                    }
                }
            }
        }
    }
}

table.table {
    &.bleu {
        @include habillage-tableau($bleu, $bleu, $bleuActive);
    }
}
```