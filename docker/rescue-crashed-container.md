## First a short description of the issue
 - on the container running linuxserver/mariadb.
 - i could not start the container anymore, because of mysql start crash.
```
docker start -i mycontainer
[Note] Recovering after a crash using tc.log
[ERROR] Can't init tc log
[ERROR] Aborting
```
[same here](https://bbs.archlinux.org/viewtopic.php?id=206379)

The solution to this is to remove tc.log files (ex: with docker exec rm tc.log).<br />
Unfortunately, docker exec needs a running container, which is not the case here.<br />

## The solution
To execute commands in the container, we change the initial script entry point.<br />
This may not work with all container because you can use only scripts that are already in the container.<br />
And the script needs to be executable.
### Find the right script
To inspect the crashed container use:
```
docker commit mycontainer mycontainer-rescue
docker exec -ti mycontainer-rescue bash
find / -name "*.sh" -exec ls -l {} \;
```
In the case of linuxserver/mariadb we can use /usr/bin/entry.sh script.<br />
We can execute some bash with /usr/bin/entry.sh bash -c.
### Run your script instead of the entrypoint script
Now we will use this in the crashed container.<br />
Locate crashed container location:
```
id=$(docker inspect mycontainer |grep '"Id":'|sed 's/.*"Id":[ ]\+"\([^"\]\+\)".*/\1/g')
cd /var/lib/docker/containers/${id}
```
Modify the entry script:
```
nano config.*.json
"Path":"docker-entrypoint.sh" -> "Path":"/usr/bin/entry.sh"
"Args":["bash","-c","mysqld"]
"Entrypoint":["docker-entrypoint.sh"] -> "Entrypoint":["/usr/bin/entry.sh"]
```
Reload the daemon to take modification in account:
```
systemctl restart docker
```
Test:
```
docker start -i mycontainer
```
This will reproduce the first error.
Now you can debug mysql start (or crash start) like if you were on a standard machine.
### Example
in my case, i have to delete tc.log files
```
nano config.*.json
"Args":["bash","-c","find / -name 'tc.log' -exec rm {} \;"]
systemctl restart docker
systemctl start -i mycontainer
```
or less agressive:
```
nano config.*.json
"Args":["bash","-c","find / -name 'tc.log'"]
systemctl restart docker
systemctl start -i mycontainer
"Args":["bash","-c","rm /var/lib/mysql/tc.log"]
systemctl restart docker
systemctl start -i mycontainer
```

