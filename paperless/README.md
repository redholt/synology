## Paperless Synology Stack

Paperlessngx and Docs can be found here https://hub.docker.com/r/linuxserver/paperless-ngx

## Usage


What is it?

Document storage for all your files that you cannot keep track of. Has OCR for enhanced search. Installing in docker on our synology.

Plan:
Install Paperlessngx with a Postgres DB backed through container manager on the synology. Make sure you have container manager installed on your synology device, this is esentially docker for synology devices. If you are using Portainer then have a goosey here https://mariushosting.com/how-to-install-paperless-ngx-on-your-synology-nas/

This guide assumes you have some knowledge of compose and .yaml, particularly around formatting and structure. Luckily, the Container Manager package we are using is quite good at flagging formatting errors so it should help you along if you get stuck.

Links:

Paperless Install Links:
https://docs.paperless-ngx.com/setup/#docker_script

Dockerhub:
https://registry.hub.docker.com/r/paperlessngx/paperless-ngx/


Setting up:

https://drfrankenstein.co.uk/category/initial-setup-7-2/ - I am using Dr Frankensteins setup here which give a great base to work from. You will need to follow this to setup your synology environment and get your UID and GID which are needed later. 

Dr Frankenstein is a great resource so please tip him if you find his site useful and join the Discord! Moving on...

The paperless installation instructions point us here https://github.com/paperless-ngx/paperless-ngx/tree/main/docker/compose telling us to pick a compose file which we need to use to get use to build the containers with. 

We are going to setup our paperless instance in a project file within container manager so it is easy to rebuild if needed. Firstly we need to setup some files on our synology that we will later map to in the docker file.

Looking here https://github.com/paperless-ngx/paperless-ngx/blob/main/docker/compose/docker-compose.postgres-tika.yml we can see we need some volumes creating for the container which are:

data:
media:
db:
redis:
export:
consume:

I would create a paperlessngx parent folder with the above files living within that so paperlessngx/data etc. 

We are using the tika build as this allows importing of emails and office docs and I am sure I will want to use these (.doc / .docx etc) in the future so better to be prepared now than migrate later!

Once the folders are setup, head to container manager and start a new project. Call it whatever you want and select the path to the folder we have created earlier (/docker/paperlessngx in my case), and select Create file in the dropdown which will give us an area to enter some text. 

Head to the paperless site for the tika file with postgres here https://github.com/paperless-ngx/paperless-ngx/blob/main/docker/compose/docker-compose.postgres-tika.yml and copy the text from here to your project on the synology. 

At this point you can continue with next just be sure to untick the "build project on finsih" option as it will not work. This will give us access to a bigger editor in the paperlessngx/YAML Configurations tab once done making our revisions easier.

Now we need to make some changes here as we are using Container Manager which has some differences to Portainer or general CLI docker functions.


- You can remove the "docker.io/library/" part unders broker:image and db: image as container manager knows where to look.

- Under the volumes: blocks add the path to your own created folders within your paperlessngx folder so they will read as follows
	/path/to/file/:/data
	/path/to/file/:/var/lib/postgresql/data
      	/path/to/file/:/usr/src/paperless/data
      	/path/to/file/:/usr/src/paperless/media
      	/path/to/file/:/usr/src/paperless/export
      	/path/to/file/:/usr/src/paperless/consume


This is simply mapping your synology folders to the docker path so the container knows where to store files. This is useful to know when making future containers as most will require some folders mapping as per their documentation.PLEASE NOTE: your volume will likely be \volume1\.

- For each of the main services (broker, db, webserver, gotenberg, tika) add a few extra lines to help keep things tidy
	container_name: - name each container so they are easy to distinguish once built
	security_opt: - Add this line and the no-new-privileges:true condition to protect yourself from malicious actors/packages

-  the yaml points to a docker-compose.env file which we cannot use in container manager so go ahead and delete this line. This hold user-ids, passwords etc. but we can simply bring these into the main compose file. If you go here https://github.com/paperless-ngx/paperless-ngx/blob/main/docker/compose/docker-compose.env you will see the UID and GID fields we need to bring over. 
	UNder webserver:environment add the following
	USERMAP_UID: and USERMAP_GID: each followed by your respective IDs

- We also need to create a username and password for paperless as this is handled in the command line on traditional builds. Poking around the docs we can see that the following fields let us set these up (not ideal in this case but this a limitation of container manager). 
	PAPERLESS_ADMIN_USER:
	PAPERLESS_ADMIN_PASSWORD

Add these under the webserver:environment block with a user and pass of your choice. This will be used to login to your paperless instance. 

- Under webserver:ports you can change the mapping from 8000:8000 to what ever you like if you have other services running. I have Sabnzbd running which is commonly on 8000 so I use a different port. Just be sure to ONLY chnage the left port number as the right mapping relates to the containers service port i.e 8777:8000 is OK, 8000:8777 will not work. 

- Delete the "volumes" block at the bottom of the file.

The full file with my changes is here, feel free to copy it but be aware of the changes describe above. 

Now we are done! Be sure to save the file first then either have it build following the save or click Action in the top right then Build. 

Now visit your syno ip followed by your specificed port (http://localhost:8000 by default) from earler and login!
