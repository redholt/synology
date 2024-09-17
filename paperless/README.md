## Paperless Synology Stack

Paperlessngx and Docs can be found here https://hub.docker.com/r/linuxserver/paperless-ngx

## Usage


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
