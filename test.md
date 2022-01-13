# Loading examples from the Github

There are two ways to do so. 
1. By cloning the s2 ```Examples``` from github repo and loading it to workspace
2. Use the extension (jupyterhub_github)

In this wiki, we will explore both ways.

# 1. Clone the github repository and load the examples

We can simply clone the github repository during the build time and move the Examples to our workplace, but there is a catch to it. As the S2-dev repository is private cloning the repository will require you github private access key. 


### Update the Dockerfile

```
 ARG BASE_IMAGE=jupyterhub/singleuser
FROM $BASE_IMAGE
USER root
RUN python3 -m pip install jupyterhub jupyterlab && \
    jupyter serverextension enable --py jupyterlab --sys-prefix  && \
    pip install kotlin-jupyter-kernel && \
    python -m pip install numpy && \
    python -m pip install matplotlib && \
    pip install tornado==6.1 && \
    apt-get update && \
    apt-get install -y --no-install-recommends git openjdk-11-jdk gnuplot gcc curl gnupg lsb-release && \
    export GCSFUSE_REPO=gcsfuse-`lsb_release -c -s`  && \
    echo "deb http://packages.cloud.google.com/apt gcsfuse-`lsb_release -c -s` main" | tee /etc/apt/sources.list.d/gcsfuse.list && \
    curl https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add - && \
    apt-get update && apt-get install -y gcsfuse

# download and install Google Cloud SDK
RUN curl https://dl.google.com/dl/cloudsdk/release/google-cloud-sdk.tar.gz > /tmp/google-cloud-sdk.tar.gz

RUN mkdir -p /usr/local/gcloud \
    && tar -C /usr/local/gcloud -xvf /tmp/google-cloud-sdk.tar.gz \
    && /usr/local/gcloud/google-cloud-sdk/install.sh \
    && apt-get -y clean all \
    && rm -rf /var/lib/apt/lists/*

ADD ./kotlin/kernel.json /opt/conda/share/jupyter/kernels/kotlin
ADD ./kotlin/.m2 /home/.m2
ADD ./kotlin/.jupyter_kotlin /home/.jupyter_kotlin
# add key file for accessing Google Storage
ADD ./key-file.json /home/key-file.json

USER jovyan
RUN git clone https://github.com/SpencerPark/IJava.git && \
    cd IJava/ && \
    ./gradlew installKernel

# cleanup
USER root
RUN rm -r /home/jovyan/work && \
    mkdir /home/jovyan/workspace/Examples && \
    chown -R jovyan: /home/jovyan/workspace/ && \ 
    rm -r /home/jovyan/IJava/     

USER jovyan
CMD   ["jupyterhub-singleuser"] 

# add JAVA_HOME and PATH environment variables
# do we still need these?
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV PATH $PATH:/usr/local/gcloud/google-cloud-sdk/bin

```
<br/>

### Make changes to `config-s2x.yaml` file

* Note that the access key is a highly sensitive thing so use it very carefully.
```
command: ['sh','-c', 'cp -r ~/../.jupyter_kotlin ~/ && cp -r ~/../.m2  ~/ && rm -rf ~/s2-dev && git clone https://<Your_access_key>@github.com/nmltd/s2-dev.git && cp -r ~/s2-dev/Examples ~/workspace/ ']
```
<br/>



### If you wish to not use your own access tiken you can add the following line instead

```
     command: ['sh','-c', 'cp -r ~/../.jupyter_kotlin ~/ && cp -r ~/../.m2  ~/  && cd ~/workspace/Examples/ && git clone https://github.com/ThatOneTallKid/nm ']
```


After making changes to the Dockerfile, the image must be pushed to the dockerhub.

Now, after updating both the Dockerfile and config-s2x.yaml file, the container is ready for deployment.

### If you further run into certain problems, you can simply check logs in your cloud shell

Get logs by typing the following command:

```
kubectl logs deployment/<name-of-deployment>
```

get name of your deployments:
```
kubectl get deployments
```

# 2. Installing the Jupyterlab-github extension for JupyterLab


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
