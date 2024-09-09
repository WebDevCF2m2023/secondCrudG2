# SecondCrudG2

## Installation

Installation de Symfony `LTS` (Long Term Support) en mode webapp (avec la plupart des dépendances pour un site web)

    symfony new SecondCrudG2 --version=lts --webapp

On rentre dans le répertoire

    cd SecondCrudG2

On crée un repository sur `github` et on le lie au projet

    git remote add origin git@github.com:WebDevCF2m2023/secondCrudG2.git
    git branch -M main
    git push -u origin main

On crée le README.md sur github, on effectue un commit sur github, puis on le charge en local avec `git pull`

### Mise à jour de la dernière version de composer

    composer self-update

### Mise à jour des bibliothèques

    composer update

### Démarrage du serveur

    symfony serve -d

fermeture

    symfony server:stop

### Création d'un `controller`

    php bin/console make:controller MyController
        created: src/Controller/MyController.php
        created: templates/my/index.html.twig

### Voir les chemins de notre projet

    php bin/console debug:route

Pour le détail d'une route particulière

    php bin/console debug:route app_my

### Mise de `MyController` comme racine du projet

Dans le fichier `src/Controller/MyController.php`

```php
    #[Route('/my', name: 'app_my')]
    public function index(): Response
    return $this->render('my/index.html.twig', [
            'controller_name' => 'MyController',
        ]);
    
    ### en
    
    #[Route('/', name: 'homepage')]
    public function index(): Response
    return $this->render('my/index.html.twig', [
            'titre' => 'homepage',
        ]);
```

### Changement de `templates/my/index.html.twig`


```twig
{% extends 'base.html.twig' %}

{% block title %}{{ titre }}{% endblock %}

{% block body %}
    <div class="container">
        <h1>{{ titre }}</h1>
    </div>
{% endblock %}

```
    
### Création d'un environnement sécurisé

Copie de `.env` vers `.env.local`

    cp .env .env.local

Il faut changer la clé dans `.env.local`, et choisir une base de donnée (ici, on choisira `MySQL`)

```.env
    # choix prod -> production, test -> test, dev -> développement
APP_ENV=dev
APP_SECRET=uneclefsecrète

    # changement des lignes ci-dessous

# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=8.0.32&charset=utf8mb4"
# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=10.11.2-MariaDB&charset=utf8mb4"
DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"

    # Par

DATABASE_URL="mysql://root:@127.0.0.1:3306/secondcrudg2?serverVersion=8.0.31&charset=utf8mb4"
# DATABASE_URL="mysql://app:!ChangeMe!@127.0.0.1:3306/app?serverVersion=10.11.2-MariaDB&charset=utf8mb4"
# DATABASE_URL="postgresql://app:!ChangeMe!@127.0.0.1:5432/app?serverVersion=16&charset=utf8"

```

### On va créer la database via `Doctrine`

Symfony va chercher le chemin depuis `.env.local`

    php bin/console doctrine:database:create


### On va créer une entité

    php bin/console make:entity Article

        created: src/Entity/Article.php
        created: src/Repository/ArticleRepository.php

On va remplir les champs dans la console

```bash
New property name (press <return> to stop adding fields):
 > title

 Field type (enter ? to see all types) [string]:
 > string
string

 Field length [255]:
 > 160

 Can this field be null in the database (nullable) (yes/no) [no]:
 > no

 updated: src/Entity/Article.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > description

 Field type (enter ? to see all types) [string]:
 > text
text

 Can this field be null in the database (nullable) (yes/no) [no]:
 > no

 updated: src/Entity/Article.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > dateCreated

 Field type (enter ? to see all types) [string]:
 > datetime
datetime

 Can this field be null in the database (nullable) (yes/no) [no]:
 > no

 updated: src/Entity/Article.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 > published

 Field type (enter ? to see all types) [string]:
 > boolean
boolean

 Can this field be null in the database (nullable) (yes/no) [no]:
 > no

 updated: src/Entity/Article.php

 Add another property? Enter the property name (or press <return> to stop adding fields):
 >

  Success!

 Next: When you're ready, create a migration with php bin/console make:migration
```

On peut aller voir le fichier `src/Entity/Article.php`

Nous allons effectuer des modifications dans celui-ci pour l'adapter à MySQL 8.*

Documentation : https://www.doctrine-project.org/projects/doctrine-orm/en/3.2/reference/attributes-reference.html#attrref_column


```php
<?php

// src/Entity/Article.php

namespace App\Entity;

use App\Repository\ArticleRepository;
use Doctrine\DBAL\Types\Types;
use Doctrine\ORM\Mapping as ORM;

#[ORM\Entity(repositoryClass: ArticleRepository::class)]
class Article
{
    #[ORM\Id]
    #[ORM\GeneratedValue]
    #[ORM\Column(
        # on met l'id en unsigned
        options:
            [
                'unsigned' => true,
            ]
    )]
    private ?int $id = null;

    #[ORM\Column(length: 160)]
    private ?string $title = null;

    #[ORM\Column(type: Types::TEXT)]
    private ?string $description = null;

    #[ORM\Column(
        type: Types::DATETIME_MUTABLE,
        # on passe la valeur par défaut en CURRENT_TIMESTAMP
        options: [
            'default' => 'CURRENT_TIMESTAMP',
        ]
    )]
    private ?\DateTimeInterface $dateCreated = null;

    #[ORM\Column(
        options:
        # si aucun boolean n'est envoyé, la valeur par défaut est fausse
            [
                'default' => false,
            ]
    )]
    private ?bool $published = null;

    public function getId(): ?int
    {
        return $this->id;
    }

    public function getTitle(): ?string
    {
        return $this->title;
    }

    public function setTitle(string $title): static
    {
        $this->title = $title;

        return $this;
    }

    public function getDescription(): ?string
    {
        return $this->description;
    }

    public function setDescription(string $description): static
    {
        $this->description = $description;

        return $this;
    }

    public function getDateCreated(): ?\DateTimeInterface
    {
        return $this->dateCreated;
    }

    public function setDateCreated(?\DateTimeInterface $dateCreated): static
    {
        $this->dateCreated = $dateCreated;

        return $this;
    }

    public function isPublished(): ?bool
    {
        return $this->published;
    }

    public function setPublished(?bool $published): static
    {
        $this->published = $published;

        return $this;
    }
}

```

## Migration

    php bin/console make:migration
    
    created: migrations/Version20240906093756.php

Si ça a fonctionné, vérifiez le fichier dans le dossier `migrations`, la requête `SQL`devrait correspondre à vos attentes.

Pour effectuer la migration :

    php bin/console d:m:m
    php bin/console doctrine:migrations:migrate

Attention cette étape pourrait effacer des données (en dev uniquement)

## Création du CRUD de `Article`

    php bin/console make:crud Article

```bash
 Choose a name for your controller class (e.g. ArticleController) [ArticleController]:
 > AdminArticleCrud

 Do you want to generate PHPUnit tests? [Experimental] (yes/no) [no]:
 > yes

 created: src/Controller/AdminArticleCrudController.php
 created: src/Form/ArticleType.php
 created: templates/admin_article_crud/_delete_form.html.twig
 created: templates/admin_article_crud/_form.html.twig
 created: templates/admin_article_crud/edit.html.twig
 created: templates/admin_article_crud/index.html.twig
 created: templates/admin_article_crud/new.html.twig
 created: templates/admin_article_crud/show.html.twig
 created: tests/Controller/ArticleControllerTest.php

  Success!

 Next: Check your new CRUD by going to /admin/article/crud/

```

### Création des liens dans un menu

On va mettre ce menu dans `templates/base.html.twig` car tous les fichiers par défaut héritent de celui-ci :

```twig
{# templates/base.html.twig #}
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>{% block title %}Welcome!{% endblock %}</title>
        <link rel="icon" href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 128 128%22><text y=%221.2em%22 font-size=%2296%22>⚫️</text><text y=%221.3em%22 x=%220.2em%22 font-size=%2276%22 fill=%22%23fff%22>sf</text></svg>">
        {% block stylesheets %}
        {% endblock %}

        {% block javascripts %}
            {% block importmap %}{{ importmap('app') }}{% endblock %}
        {% endblock %}
    </head>
    <body>
        {# création du block pour la bare de navigation,
        qui n'est pas dans le block body, lui-même effacé
        par tous les fichiers twigs auto-générés #}
        {% block navbar %}
            {% include 'menu.html.twig' %}
        {% endblock %}
        {% block body %}{% endblock %}
    </body>
</html>

```

Puis le menu

```twig
{# templates/menu.html.twig #}
<nav>
    <a href="{{ path('homepage') }}">Accueil</a>
    <a href="{{ path('app_admin_article_crud_index') }}">Crud des articles</a>
</nav>
```

### Design des formulaires

Il existe des designs pré-établis dans Symfony :

https://symfony.com/doc/current/form/form_themes.html#symfony-builtin-forms

Si on veut en utiliser un principal, on va ouvrir le fichier

    config/packages/twig.yaml

Et rajouter

```twig
twig:
    file_name_pattern: '*.twig'
    # pour les formulaires par défaut
    form_themes: ['bootstrap_5_layout.html.twig']

when@test:
    twig:
        strict_variables: true

```

Notre formulaire est généré avec de l'html maintenant compatible avec Bootstrap !

Nous allons ajouter bootstrap avec `AssetMapper`

    php bin/console debug:asset-map --full

On va importer `Bootstrap 5` :

    php bin/console importmap:require bootstrap

On va mettre le lien vers le css dans le fichier `assets/app.js` car `AssetMapper` ne prend pas en charge le css (actuel)

```javascript
/*
assets/app.js
 */
import './bootstrap.js';
/*
 * Welcome to your app's main JavaScript file!
 *
 * This file will be included onto the page via the importmap() Twig function,
 * which should already be in your base.html.twig.
 */
// notre fichier bootsrtap
import './vendor/bootstrap/dist/css/bootstrap.min.css';
import './styles/app.css';

console.log('This log comes from assets/app.js - welcome to AssetMapper! 🎉');
```