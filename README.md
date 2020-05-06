# demojenkins


I am going to create a real world environment , where multiple developers work together to create a real world product. For simplicity we 're taking here a website as a product.

I am using Two systems here one is *Redhat* system and other *Windows*
**Requirements:
  * Redhat system or any linux system
  * Jenkins Installed
  * Docker Installed
  * httpd docker image ( for running webserver)
  

#The Redhat system will be our server for jenkins. We are hosting the site on the Docker image. So make sure you have httpd image and docker installed in Redhat system 

#To check if httpd server is running use:

```systemctl status httpd```
or start using 
```systemctl start httpd ```


Next we need to setup two httpd webservers in docker containers ( One for testing environment and other for production. So run command :

``` docker run -dit --name testing_server -p 4546:80 -v /web:/usr/local/apache2/htdocs/ httpd

  docker run -dit --name production_server -p 4547:80 -v /original:/usr/local/apache2/htdocs/ httpd
```


Thus we have two web servers running. for testing_server the mount point * /web * is created , similarly, for production environment * /originalweb * is mounted with location */usr/local/apache2/htdocs/* of web server which is working directory of *httpd web server* . What that means is that , whatever file is under /web will be displayed on testing_server and the files under /originalweb are displayed on production server .


We will be creating three jobs on jenkins to automate the web site hosting. As soon as the developer commits and pushes the data on the github , the site will be available to clients once the Quality Assurance team allows it.


For that we need to create a repository on our system and two branches under it , let's say branches name are #master and #dev.


Under the master branch first of all we create an index.html file to be deployed.


After writing this html file , run the command
```git commit -m " first commit " -a ```

this will add and commit your file . and at last push the file to your github using :
``` git push ```

or we can also use hooks which are under 
```.git > hooks ```
create a file named **post-commit**  and write in bash script to push the code to github everytime you commit the file
``` !#/bin/bash
     git push 
```



For this file to be deployed , we now require jenkins, so that as soon as a new change is pushed to dev branch, it directly goes to testing server.


Login to jenkins and Create a job ,say Job1

After creating the job go to configure the job , and under source code management , enter the github repo url :
Configure > SCM > Git

Under Build Triggers Go for **"Github hook triggers for GITScm polling"** option

and in the Execute shell section under Build  paste this command so that the files of github repo will be copied to the /web location:

``` sudo cp -r * /web```


click on save

Similarly Create Job2 for the production server which will copy files from master branch in the /originalweb directory

Make sure Branch specifier is set to /master . Rest of the steps are same.

#Now for Quality assurance team we need a third job which will merge the code of developer with the master branch after testing.

So similarly create third job (Job3) :

### You have to give credentials in Credentials field for this job and for jobs 1 and 2 as well .

To trigger this job remotely, we need to setup a token:

Finally we need to push the code to master branch :
> Build > Post Build Actions 
> click ** Push only if build succeeds  and Merge results**


Let the developer changes some code in the file

and push the file to github .

#Now the need of web-hooks on github arises. So go to settings of your repo in github and add the url of jenkins server:
(We can use ngrok here)

So now when you'll push the code to github , jenkins will copy it from your repo to the /web directory of your system


Thus the file will be running on testing_web server .

Similarly the Job 2 will work for production system.

Job3 will work differently.

Remember we have given the access code in job three to trigger the job3 build.

so using that URL we can trigger this job

As soon as quality assurance team will go to respective URL Job3 will be triggered and any change made in dev branch will be merged in master branch and will be pushed to the master branch thus the new file will be deployed to the /originalweb

directory which is database for production server . Thus new updated site will be deployed.




