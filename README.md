# Continuous Deployment with GCP
Example of deploying a Flask web application with continuous deployment using PaaS and Google App Engine.


## Make full public available API
1. Setup GCP project, and enable Google Cloud Build
<br/><br/>
2. Using google cloud shell, generate ssh key to setup with github<br>
  ```ssh-keygen -t rsa```
  <br/><br/>
3. Create a new Github reporitory and Clone it<br>
  ```git clone git@github.com:VM-137/cd-on-gcp-flask-web-.git```
  <br/><br/>
4. Enter the repository folder and create (Makefile, requirements.txt, app.yaml, main.py, test_main.py)<br>
 The 'Makefile' will run the commands, the 'requierements.txt' will have a list of dependencies we will need, the 'app.yaml' will be the IaC and the 'main.py' will be the code that actually allow us to run flask, and 'test_main.py' will be for test.<br>
   ```
    cd cd-on-gcp-flask-web-
    touch Makefile
    touch requirements.txt
    touch app.yaml
    touch main.py
    touch main_test.py```
    <br/><br/>
5. Edit 'app.yaml' as we see in the [Official Documentation](https://github.com/GoogleCloudPlatform/python-docs-samples/tree/main/appengine/standard_python3/hello_world)<br>
```runtime: python39```
<br/><br/>
6. Edit 'main.py' <br>
    ```python
    from flask import Flask

    app = Flask(__name__)


    @app.route('/')
    def hello():
        """Return a friendly HTTP greeting."""
        return 'Hello World!'


    if __name__ == '__main__':
        app.run(host='127.0.0.1', port=8080, debug=True)
    ```
    <br/><br/>    
7. Edit 'requirements.txt' using official documentation and adding some libraries<br>
    ```
     Flask==2.1.0
     pytest
     pylint
    ```
    <br/><br/>
8. Edit 'test_main.py'
    ```python
    import main


    def test_index():
        main.app.testing = True
        client = main.app.test_client()

        r = client.get('/')
        assert r.status_code == 200
        assert 'Hello World' in r.data.decode('utf-8')
    ```
    <br/><br/>
9. Edit 'Makefile' <br>
    ```
    install:
        pip install --upgrade pip &&\
          pip install -r requirements.txt

    test:
        #

    lint:
        pylint --disable=R,C main.py

    all: install lint test
    ```
    <br></br>
9. Create virtual environment with the same name as the project and source <br>
    ```
    virtualenv --python $(which python) ~/.cd-on-gcp-flask-web-
    source ~/.cd-on-gcp-flask-web-/bin/activate
    ```
    <br></br>
    
10. Install packages <br>
    ```
    make install
    ```
    <br></br>
11. Create GCP app , deploy and check the url which is public available <br>
    ```
    gcloud app create
    gcloud app deploy
    ```
    <br></br>
    
## Set up Continuous Deployment
  First we need to enable Cloud Build on GCP and then Enable App engine service, [check this url](https://cloud.google.com/source-repositories/docs/automate-app-engine-deployments-cloud-build). Before that we can continue. <br>
1. Create 'cloudbuild.yaml' <br>
    ```
    touch cloudbuild.yaml
    ```
    <br></br>
2. Edit 'cloudbuild.yaml' <br>
    ```
    steps:
    - name: "gcr.io/cloud-builders/gcloud"
      args: ["app", "deploy"]
    timeout: "1600s"
    ```
    <br></br>
3. Check git status, commit and push <br>
    ```
    git status
    ```
    ```
    On branch main
    Your branch is up to date with 'origin/main'.
    Untracked files:
      (use "git add <file>..." to include in what will be committed)
            .gcloudignore
            Makefile
            app.yaml
            cloudbuild.yaml
            main.py
            requirements.txt
            test_main.py
    ```
    <br></br>
 we add all the files and check status again <br>
    ```
    git add *
    git status
    ```
    <br></br>
 it returns all ok, therefore next we will commit and then push. (if it is the first commit, you need to config username and email) <br>
    ```
    git commit -m "Adding project"
    git push
    ```
    <br></br>
 4. Create a build trigger<br>

    * In the GCP Console, open the Cloud Build Triggers page.

    * Open the Triggers page

    * If your Google Cloud project isn't selected, click Select a project, and then click the name of your Google Cloud project.

    * Click Create Trigger.

    * The Create trigger page opens.

    * Fill out the following options:
         In the Name field, type app-engine-test.
         Under Event, select Push to a branch.
         Under Source, select your Repository and ^master$ as your Branch.
         Under Configuration, select Cloud Build configuration file (yaml or json).
         In the Cloud Build configuration file location field, type cloudbuild.yaml after the /.

    * Click Create to save your build trigger.
    <br></br>
5. Check if a change starts the trigger, add some code to 'main.py', test it locally and if it works add, commit and push <br>
    ```python
    from flask import Flask
    from flask import jsonify

    app = Flask(__name__)


    @app.route('/')
    def hello():
        """Return a friendly HTTP greeting."""
        return 'Hello World!'


    @app.route('/name/<value>')
    def name(value):
        val = {"value": value}
        return jsonify(val)

    if __name__ == '__main__':

        app.run(host='127.0.0.1', port=8080, debug=True)
    ```
    
 6. Check Code Build history 
    
