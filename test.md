# Installing the Jupyterlab-github extension for JupyterLab


Jupyterlab-github is an extension for accesing Github repositories from JupyterLab.
It will allow non-logged users to access the ```Examples``` folder. <br/>
When you install this extension, an additional filebrowser tab will be added to the left area of JupyterLab. This filebrowser allows you to select GitHub organizations and users, browse their repositories, and open the files in those repositories. If those files are notebooks, you can run them just as you would any other notebook. You can also attach a kernel to text files and run those. Basically, you should be able to open any file in a repository that JupyterLab can handle.

## Prerequisites


* JupyterLab
* A Github account for the server extension to work

As S2 is using JupyterLab version older than 3.0, we will use some old commands to install the extension.

## Installation


Install the serverextension using pip, and then enable it:
```
pip install jupyterlab-github
```
If you are running Notebook 5.2 or earlier, enable the server extension by running
```
jupyter serverextension enable --sys-prefix jupyterlab_github
```

* After the above changes the ```Dockerfile``` should look like this 

<br/>

![Upload Image](/extension.png)

<br/>

* Further build the docker image and give it your custom tag before pushing it to the DockerHub.

## Usage


After Installing the extension and building the docker image, we can easily make the deployment. 
<br/>
Once the deployment is done the main notebook page should look like this: 
<br/>

![](/ext.png)

<br/>

You can easily see the Github Icon appearing at the left side bar.

**Voila! you successfully installed the extension**

Next step, just fill in the ```Owner/repository``` in the form and save, you will be able to access the repository.

Note: the Examples folder is not accessible by default. You need to clone the repository ,thus making it public to access or simply use ```ThatOneTallKid/nm``` to ease up the process.

## Accessing Private Repositories from the server side


### Generate a github access token from github
 * Go to your account settings on GitHub and select "Developer Settings" from the left panel.
 * On the left, select "Personal access tokens"
 * Click the "Generate new token" button, and enter your password.
 * Give the token a description, and check the "repo" scope box.
 * Click "Generate token"
 * You should be given a string which will be your access token.

 <br/>

**You now need to add the credentials you got from GitHub to your notebook configuration file.**
Instructions for generating a configuration file can be found [here](https://jupyter-notebook.readthedocs.io/en/stable/config_overview.html#configure-nbserver). Once you have identified this file, add the following lines to it:
```
c.GitHubConfig.access_token = '< YOUR_ACCESS_TOKEN >'
```

where ```"< YOUR_ACCESS_TOKEN >"``` is the string value you obtained above.