# Introduction to Heroku Deployment

In the last course, you learned how to deploy applications to AWS EKS. Because Heroku is so easy to use, we'll be able to get you up and running with Heroku a bit faster. Though Heroku is easy to use, it will require us to make some updates to how we structure our application so Heroku can build and run the application properly.

In this section, we'll discuss how to build and configure your application such that it runs properly on Heroku. Here's a sneak peek of what that entails:

* Creating and updating a requirements.txt file to install dependencies
* Setting up your environment variables
* Using a Procfile, using gunicorn to run the application
* We'll also discuss creating and managing your database in the next concept.



## Installing Dependencies

Deploying an application to Heroku is as simple as pushing that repository to Heroku, just like Github.
Heroku does a lot of things behind the scenes for us when we push a repository - including installing dependencies.
For a Python application, Heroku looks for a `requirements.txt` file that needs to include all of your dependencies.

If you want to follow along using the sample project, fork and clone it from this link before continuing.

```shell
git clone https://github.com/udacity/FSND/tree/master/projects/capstone/heroku_sample/starter
```

The sample project is live at https://sample-cem.herokuapp.com/. Try endpoints `/` and `/coolkids`.

In order to save our package requirements we'll use the following command:

```shell
pip freeze > requirements.txt
```

Pip freezing is a process in which pip reads versions of all packages in the current virtual environment and saves them to a text file. In our case, this the text file that Heroku will use to install dependencies.

Be advised: the `requirements.txt` file does not automatically update if you install a new library. It's like a snapshot - any changes after aren't reflected in a past snapshot. You'll want to ensure `requirements.txt` is up to date before pushing your app to Heroku.



## Environment Configuration

In previous projects, you used a bash file to set up local environment variables. You'll do the same here. We want them all contained in the same kind of file for easier transfer later to the Heroku interface.

If you're following along in the project, use `touch setup.sh` and set up all of your environment variables in that file.

Most of the work we do for Heroku will be in our application files or the command line. In order to give you some familiarity with the web interface, we'll set up the environment variables there, after we deploy our application. For now, check out the screenshot below to get used to the interface. Once you're in a project's settings, you'll see an option to `Reveal Config Vars`. Once you click on that, a table similar to that you see below will appear. Here, you define your variables just as you did in the `setup.sh` file, just without the equals signs!

After hitting Reveal Config Vars you’ll see a table like this where you can define environment variables for your hosted Heroku app.

## Gunicorn

Gunicorn is a pure-Python HTTP server for WSGI applications. We'll be deploying our applications using the Gunicorn webserver.

First, we need to install gunicorn using `pip install gunicorn`.

Next `touch Procfile` to create the file.

Procfile is exceedingly simple. It only needs to include one line to instruct Heroku correctly for us:

```shell
web: gunicorn app:app.
```

Just make sure your app is housed in `app.py` as it is in the sample project.

Go ahead and make those updates to the sample project if you're following along.

## Database Manage & Migrations

In the data modeling course, you learned how to use migrations to manage your database schema and changes that you make to it. Heroku can run all your migrations to the database you have hosted on the platform, but in order to do so, your application needs to include a manage.py file.

We'll need three new packages in the file. Run the following commands to install them:

```shell
pip install flask_script
pip install flask_migrate
pip install psycopg2-binary
```

The `manage.py` file will contain the following code:

```python
from flask_script import Manager
from flask_migrate import Migrate, MigrateCommand

from app import app
from models import db

migrate = Migrate(app, db)
manager = Manager(app)

manager.add_command('db', MigrateCommand)


if __name__ == '__main__':
    manager.run()
```

Now we can run our local migrations using our `manage.py` file, to mirror how Heroku will run behind the scenes for us when we deploy our application:

```shell
python manage.py db init
python manage.py db migrate
python manage.py db upgrade
```

Those last commands are the essential process that Heroku will run to ensure your database is architected properly.
We, however, won't need to run them again unless we're testing the app locally.

REMINDER: Because you installed new packages you need to use freeze again to update the `requirements.txt` file.


## Deploying to Heroku

Alright, it's the moment of truth! Now that you've learned how to configure an application for Heroku, you'll go ahead and deploy your application so you can access it from the cloud, as well as share it with others.

### Create Heroku app

In order to create the Heroku app run `heroku create name_of_your_app`. The output will include a git url for your Heroku application. Copy this as, we'll use it in a moment.

Now if you check your Heroku Dashboard in the browser, you'll see an application by that name. But it doesn't have our code or anything yet - it's completely empty. Let's get our code up there.

### Add git remote for Heroku to local repository

Using the git url obtained from the last step, in terminal run

```shell
git remote add heroku heroku_git_url
```

### Add postgresql add on for our database

Heroku has an addon for apps for a postgresql database instance. Run this code in order to create your database and connect it to your application

```shell
heroku addons:create heroku-postgresql:hobby-dev --app name_of_your_application
```

Breaking down the `heroku-postgresql:hobby-dev` section of this command:

* `heroku-postgresql` is the name of the addon.
* `hobby-dev` on the other hand specifies the tier of the addon, in this case the free version which has a limit on the amount of data it will store, albeit fairly high.

Run the next line in order to check your configuration variables in Heroku. 

```shell
heroku config --app name_of_your_application
```

You will see `DATABASE_URL` and the `URL` of the database you just created. That's excellent, but there were a lot more environment variables our apps use.

### Go fix our configurations in Heroku

In the browser, go to your Heroku Dashboard and access your application's settings.
Reveal your config variables and start adding all the required environment variables for your project.
For the purposes of the sample project, just add one additional one - ‘EXCITED’ and set it to true or false in all lowercase.

### Push it!

```shell
git add .
git ci -m "Chore: Create the configuration for Heroku"
git push heroku master
```

### Run migrations

Once your app is deployed, run migrations by running

```shell
heroku run python manage.py db upgrade --app name_of_your_application
```