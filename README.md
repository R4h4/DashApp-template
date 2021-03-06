# Dash Multi-Page App template
[Plotly's Dash](https://github.com/plotly/dash) is one of 
the most amazing projects of the past year, finally making it possible
to develop full-size web-applications using only Python.

However, the beginning of a new Dash-multi-page app can be tedious. 
This repository offers a basic template for such apps. To us it, 
clone this repository and remove the original git connection with the following commands:
``` git-clone-command
git clone --depth=1 https://github.com/r4h4/dash-multipage-template.git
rm -rf ./dash-multipage-template/.git
ren dash-multipage-template your-project-name
git init your-project-name
```
Next, install all requirements with the pip command:
```install-packages
pip install -r requirements.tx
```
In addition to the basic dash module 
([Core-](https://dash.plot.ly/dash-core-components) 
and [HTML-](https://dash.plot.ly/dash-html-components) Components), 
this template also contains [dash-bootstrap-components](https://dash-bootstrap-components.opensource.faculty.ai/).

## Deployment
There are multiple ways to put your app online, here are the three 
easiest:
### AWS Lambda (using Zappa)
Serverless deployment on AWS Lambda is maybe the easiest an, at least for the prototyping stage, cheapest option to deploy your app. The best way to do so is to use [Zappa](https://github.com/Miserlou/Zappa). 
Install the package with:
```install-zappa
pip install zappa
```
Afterward, you can initialize the process using `zappa init`. Afterward 
 I recommend following the instructions on the official page. 
 Point the `app_function` to `index.server`.
 
Note that AWS Lambda has a relatively moderate size limit of 500MB 
for temporary files (when the `slim_handler` is set to `True`), 
restricting the usage of large packages (i.e., SciPy alone brings 
100MB to the table). Furthermore, compiled packages like [gensim](https://github.com/RaRe-Technologies/gensim)
that do not have a dedicated Lambda deployment package (list 
can be found [here](https://github.com/Miserlou/lambda-packages)) 
require your computer to run Linux, since the packages are 
copied from your local machine onto your Lambda service. 

The workaround for this is to rebuild the app inside a Docker container that is running the Linux version used by AWS Lambda. A fitting image,
including Zappa, can be found on [DockerHub](https://hub.docker.com/r/mligus/zappa/dockerfile/).

To make this hack work, first, you have to delete all .pyc files, 
which are byte code generated by your Windows machine during the package installation.
```delete-pyc
del /S *.pyc
```
Afterward, you can start the docker container and activate the bash console.
```start-docker
docker run -ti -v %USERPROFILE%/.aws:/root/.aws -v <absolute_path_to_project_folder_on_host_sys>:/var/task --rm mligus/zappa bash
```
If everything worked correctly, you should see `zappa>` in your terminal. 
Now activate the virtual environment inside the container and install your requirements.
```install-recs
       zappa> source /var/venv/bin/activate
(venv) zappa> pip install -r requirements.txt
```
Final, you will be able to deploy your app. This instruction assumes that you already initialized Zappa/created the zappa_requirements.json before starting 
Docker.
```deploy-from-docker
zappa> zappa deploy dev
```
(Credit to [Wong Yan Yee](https://medium.com/@houdinisparks/how-i-build-an-authenticated-serverless-flask-api-with-zappa-and-docker-for-a-model-582fc48fa0e0) 
for this solution.)  
 
### Heroku
[Heroku](https://www.heroku.com/) is another simple way to run your app without having to manage any infrastructure. For deployment on Heroku, you need a WSGI HTTP server. Gunicorn is a simple way that can be installed using PyPi.
```
pip install gunicorn
```
Afterward, you have to create the `Procfile` in the base directory of your project. Here an example of how this can look like:
```Procfile
web: gunicorn --timeout 300 app:server
```
After signing up for Heroku, create a new app, and connect it with your GitHub repository. This step automatically creates a deployment-pipeline that builds and deploys every change made to the repository.

### Docker
Last but not least, you can also create a Docker container yourself and deploy how- and wherever you want. Similar to Heroku (which in the 
background also runs your app inside a Docker container on AWS 
infrastructure) I recommend using Gunicorn as WSGI HTTP server.
```
pip install gunicorn
```
This is how an example Dockerfile could look like:
```Dockerfile
FROM Python:3.7

USER root
WORKDIR /app
ADD . /app
RUN pip install --trusted-host pypi.python.org -r requirements.txt

EXPOSE 3000

ENV NAME Dash
CMD ["gunicorn", "-b", "0.0.0.0:8000", "app"]
```

If this template is helpful to you, support me by leaving a star 
on GitHub, clap for me on [Medium](https://medium.com/@karsteneckhardt) and consider following me.