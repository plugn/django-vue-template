# django-vue-template for Vagrant and Django 2.0

Fork of https://github.com/ariera/django-vue-template adding:
* Django 2.0
* Configuration for Vagrant development
* Configuration for Heroku deployment

## Installation
Ensure that your `Vagrantfile` has a static IP using the following line:
```
config.vm.network :private_network, ip: "10.10.10.61"
```
SSH into vagrant and clone this repo.

If you want to use a different IP, change the `assetsPublicPath` in [`config/index.js`](config/index.js):
```JavaScript
module.exports = {
  dev: {
    ...
    assetsPublicPath: 'http://10.10.10.61:8080/',
    ...
  }
  ...
}
```
If using a **shared directory**, create a symbolic link for `node_modules` folder to a directory that is not shared (such as `home`) to prevent filesystem failures from node.js packages:
```bash
cd path/to/cloned/repo
mkdir /home/vagrant/node_modules
ln -s /home/vagrant/node_modules node_modules
```
Install npm dependencies:
```bash
npm install --no-bin-links
```
Install Python dependencies and activate Virtualenv
```
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```
## Development

* On one terminal run your webpack dev server

```bash
npm run dev
```

* On another terminal launch the Django application

```bash
python manage.py runserver
```

On the **host machine** point browser to http://10.10.10.61:8000, but test that you can also access the pure Django world, for example by going to the admin panel http://10.10.10.61:8000/admin

## Tests

``` bash
# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

## Heroku Deployment
Install [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli)

Create Heroku App:
```bash
heroku create
```
Set buildpacks:
```bash
heroku buildpacks:add --index 1 heroku/nodejs
heroku buildpacks:add --index 2 heroku/python
```
Set `ENVIRONMENT` environment variable to `'production'` this is used in a custom Django template [context processor](project/context_processors.py) to load production JavaScript bundles in the correct order when deployed
```bash
heroku config:set ENVIRONMENT=production
```
Set `DJANGO_DEBUG` environment variable, this will disable `DEBUG` in Django settings:
```bash
heroku config:set DJANGO_DEBUG=''
```
Add the Heroku app to Django `ALLOWED_HOSTS` [settings.py](project/settings.py):
```Python
...
ALLOWED_HOSTS = ['YOUR-APP.herokuapp.com', '10.10.10.61', 'localhost', '127.0.0.1']
...
```
Commit changes and push to Heroku:
```bash
git add -a
git commit -m "Update settings for Heroku"
git push heroku master
```