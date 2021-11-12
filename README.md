# LBAW's framework

## Introduction

This README describes how to setup the development environment for LBAW 2020/21.
These instructions address the development with a local environment, i.e. on the machine (that can be a VM) **without using a Docker container for PHP or Laravel**.
Containers are used for PostgreSQL and pgAdmin, though.

The template was prepared to run on Linux 20.04LTS, but it should be fairly easy to follow and adapt for other operating systems.

* [Installing the Software Dependencies](#installing-the-software-dependencies)
* [Installing Docker and Docker Compose](#installing-docker-and-docker-compose)
* [Setting up the Development repository](#setting-up-the-development-repository)
* [Working with PostgreSQL](#working-with-postgresql)
* [Developing the project](#developing-the-project)
* [Publishing the image](#publishing-your-image)
* [Laravel code structure](#laravel-code-structure)


## Installing the Software Dependencies

To prepare you computer for development you need to install some software, namely PHP and the PHP package manager Composer.

We recommend using an __Ubuntu__ distribution that ships PHP 7.4 (e.g Ubuntu 20.04LTS).
You may install the required software with:

    sudo apt-get install git composer php7.4 php7.4-mbstring php7.4-xml php7.4-pgsql

The following links provide instructions for installing [Docker](https://docs.docker.com/get-docker/)and [Docker Compose](https://docs.docker.com/compose/install/)

## Setting up the Development repository

You should have your own repository and a copy of the demo repository in the same folder in your machine.
Then, copy the contents of the demo repository to your own.

    # Clone the group repository (lbaw21gg), if not yet available locally
    # Notice that you need to substitute gg by your group's number
    git clone https://git.fe.up.pt/lbaw/lbaw21/lbaw21gg.git

    # clone the LBAW's project skeleton
    git clone https://git.fe.up.pt/lbaw/template-laravel.git

    # remove the Git folder from the demo folder
    rm -rf template-laravel/.git
    # preserve existing README.md
    mv template-laravel/README.md template-laravel/README_lbaw.md

    # go to your repository
    cd lbaw21gg

    # make sure you are using the master branch
    git checkout master

    # copy all the demo files
    cp -r ../template-laravel/. .

    # add the new files to your repository
    git add .
    git commit -m "Base laravel structure"
    git push origin master

At this point you should have the project skeleton in your local machine and be ready to start working on it.
You may remove the __template-laravel__ demo directory, as it is not needed anymore.


## Installing local PHP dependencies

After the steps above you will have updated your repository with the required Laravel structure from this repository.
Afterwards, the command bellow will install all local dependencies, required for development.

    composer install


## Working with PostgreSQL

We've created a _docker-compose_ file that sets up __PostgreSQL__ and __pgAdmin4__ to run as local Docker containers.

From the project root issue the following command:

    docker-compose up

Navigate on your browser to http://localhost:4321 to access pgAdmin4 and manage your database. Depending on your installation setup, you might need to use the IP address from the virtual machine providing docker instead of localhost. Please refer to your installation documentation.
Use the following credentials to login:

    Email: postgres@lbaw.com
    Password: pg!password

On the first usage you will need to add the connection to the database using the following attributes:

    hostname: postgres
    username: postgres
    password: pg!password

Hostname is _postgres_ instead of _localhost_ since _Docker Compose_ creates an internal DNS entry to facilitate the connection between linked containers.


## Developing the project

You're all set up to start developing the project.
In the provided skeleton you will already find a basic todo list App, which you will modify to start implementing your own project.

To start the development server, from the project's root run:

    # Seed database from the seed.sql file.
    # Needed on first run and everytime the database script changes.
    php artisan db:seed

    # Start the development server
    php artisan serve

Access http://localhost:8000 to access the app. Username is `admin@example.com`, and password `1234`. These credentials are copied to the database on the first instruction above.

To stop the server just hit Ctrl-C.

## Laravel code structure

Before you start, you should make yourself familiar with [Laravel's documentation](https://laravel.com/docs/8.x).

In Laravel, a typical web request will touch the following concepts and files.

### 1) Routes

The web page is processed by *Laravel*'s [routing](https://laravel.com/docs/8.x/routing) mechanism.
By default, routes are defined inside *routes/web.php*. A typical *route* looks like this:

    Route::get('cards/{id}', 'CardController@show');

This route receives a parameter *id* that is passed on to the *show* method of a controller called *CardController*.

### 2) Controllers

[Controllers](https://laravel.com/docs/8.x/controllers) group related request handling logic into a single class.
Controllers are normally defined in the __app/Http/Controllers__ folder.

    class CardController extends Controller
    {
        public function show($id)
        {
          $card = Card::find($id);
          $this->authorize('show', $card);
          return view('pages.card', ['card' => $card]);
        }
    }

This particular controller contains a *show* method that receives an *id* from a route.
The method searches for a card in the database, checks if the user as permission to view the card, and then returns a view.

### 3) Database and Models

To access the database, we will use the query builder capabilities of [Eloquent](https://laravel.com/docs/8.x/eloquent) but the initial database seeding will still be done using raw SQL (the script that creates the tables can be found in __resources/sql/seed.sql__).

    $card = Card::find($id);

This line tells *Eloquent* to fetch a card from the database with a certain *id* (the primary key of the table).
The result will be an object of the class *Card* defined in __app/Card.php__.
This class extends the *Model* class and contains information about the relation between the *card* tables and other tables:

    /* A card belongs to one user */
    public function user() {
      return $this->belongsTo('App\User');
    }

    /* A card has many items */
    public function items() {
      return $this->hasMany('App\Item');
    }

### 4) Policies

[Policies](https://laravel.com/docs/8.x/authorization#writing-policies) define which actions a user can take.
You can find policies inside the __app/Policies__ folder.
For example, in the __CardPolicy.php__ file, we defined a *show* method that only allows a certain user to view a card if that user is the card owner:

    public function show(User $user, Card $card)
    {
      return $user->id == $card->user_id;
    }

In this example policy method, *$user* and *$card* are models that represent their respective tables, *$id* and *$user_id* are columns from those tables that are automatically mapped into those models.

To use this policy, we just have to use the following code inside the *CardController*:

    $this->authorize('show', $card);

As you can see, there is no need to pass the current *user*.

### 5) Views

A *controller* only needs to return HTML code for it to be sent to the *browser*. However we will be using [Blade](https://laravel.com/docs/8.x/blade) templates to make this task easier:

    return view('pages.card', ['card' => $card]);

In this example, *pages.card* references a blade template that can be found at __resources/views/pages/card.blade.php__.
The second parameter contains the data we are injecting into the template.

The first line of the template states that it extends another template:

    @extends('layouts.app')

This second template can be found at __resources/views/layouts/app.blade.php__ and is the basis of all pages in our application. Inside this template, the place where the page template is introduced is identified by the following command:

    @yield('content')

Besides the __pages__ and __layouts__ template folders, we also have a __partials__ folder where small snippets of HTML code can be saved to be reused in other pages.

### 6) CSS

The easiest way to use CSS is just to edit the CSS file found at __public/css/app.css__. You can have multiple CSS files to better organize your style definitions.

### 7) JavaScript

To add JavaScript into your project, just edit the file found at __public/js/app.js__.

### 8) Configuration

Laravel configurations ar acquired from environment variables. They can be available in the environment where the laravel process is started, or acquired by reading the `.env` file in the root folder of the laravel project. This file can set environment variables, which set or overwride the variables from the current context. You will likely have to update these variables, mainly the ones configuring the access to the database, starting with `DB_`.

If you change the configuration, you might need to run the following command to discard a compiled version of the configuration from laravel's cache:

    php artisan cache:clear
    php artisan config:clear

## Publishing your image

You should keep your git's master branch always functional and frequently build and deploy your code.
To do so, you will create a _Docker image_ for your project and publish it at [Docker Hub](https://hub.docker.com/), like you did for the PIU. LBAW's production machine will frequently pull all these images and make them available at http://lbaw21gg.lbaw-prod.fe.up.pt/.

BTW, this demo repository is available at http://demo.lbaw-prod.fe.up.pt/.
Make sure you are inside FEUP's network or are using the VPN.

First thing you need to do is create a [Docker Hub](https://hub.docker.com/) account and get your username from it.
Once you have a username, let your Docker know who you are by executing:

    docker login

Once your Docker is able to communicate with the Docker Hub using your credentials, configure the __upload_image.sh__ script with your username and the image name.
Example configuration:

    DOCKER_USERNAME=johndoe # Replace by your Docker Hub username
    IMAGE_NAME=lbaw21gg     # Replace by your LBAW group name

Afterwards, you can build and upload the docker image by executing that script from the project root:

    ./upload_image.sh

You can test locally the image, just published in the Docker Hub, by running:

    docker run -it -p 8000:80 -e DB_DATABASE="lbaw21gg" -e DB_USERNAME="lbaw21gg" -e DB_PASSWORD="PASSWORD" <DOCKER_USERNAME>/lbaw21gg

The above command exposes your application on http://localhost:8000.
The `-e` argument creates environment variables inside the container, used to provide Laravel with the required database configurations.

Your database configuration will be provided as an environment variable to your container on start. You do not need to specify it on you env file. Any specification there will be replaced when the docker image starts.

Finally, note that there should be only one image per group.
One team member should create the image initially and add his team to the **public** repository at Docker Hub.
You should provide your teacher the details for accessing your Docker image, namely, the Docker username and repository (*DOCKER_USERNAME/lbaw21gg*), in case it was changed.

While running your container, you can use another terminal to run a shell inside the container by executing:

    docker run -it lbaw21gg/lbaw21gg bash

Inside the container you may, for example, see the content of the Web server logs by executing:

    root@2804d54698c0:/# tail -f /var/log/nginx/error.log    # follow the errors
    root@2804d54698c0:/# tail -f /var/log/nginx/access.log   # follow the accesses


-- André Restivo, Tiago Boldt Sousa, 2021
