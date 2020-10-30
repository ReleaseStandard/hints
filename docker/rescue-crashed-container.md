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
docker run -ti --rm mycontainer-rescue
find / -name "*.sh" -exec ls -l {} \;
```
**Example**<br />
In the case of linuxserver/mariadb we can use /usr/bin/entry.sh script.<br />
We can execute some bash with /usr/bin/entry.sh bash -c.
### Run your script instead of the entrypoint script
Now we will use this in the crashed container.<br />
#### Locate crashed container location:
```
id=$(docker inspect mycontainer |grep '"Id":'|sed 's/.*"Id":[ ]\+"\([^"\]\+\)".*/\1/g')
cd /var/lib/docker/containers/${id}
```
**Example**<br />
/var/lib/docker/0f46e1865d6ff7613cbee0f74ecea72d6fe3a48f644f9b692e9d4424e8bcc9bf/<br />
**WARN** make a backup of this folder somewhere, maybe twice is better.
#### Modify the entry script:
```
nano config.*.json
"Path":"docker-entrypoint.sh" -> "Path":"/usr/bin/entry.sh"
"Args":["bash","-c","mysqld"]
"Entrypoint":["docker-entrypoint.sh"] -> "Entrypoint":["/usr/bin/entry.sh"]
```
#### Reload the daemon to take modification in account:
```
systemctl restart docker
```
#### Test
```
docker start -i mycontainer
```
This will reproduce the first error.<br />
Now you can debug mysql start (or crash start) as if you were on a standard machine.

### Example with my issue
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
then restore initial configuration:
```
cp ${BACKUP}/config.*.json /var/lib/docker/containers/${id}/
systemctl restart docker
systemctl start -i mycontainer
```
If your issue if the same as me, now it works !<br />
Else, you will have to find more details on your issue.<br />
You will need several tries and so with below script you can get a slow but functionnal shell on the container:
```bash
function change_cmd() {
  logloc="/var/lib/docker/containers/PUT_YOUR_ID_HERE/";
  cmd=(bash -c);
  cmd[2]="$@";
  echo " cmd=$cmd";
  for((i=0;i<${#cmd[@]};i=i+1)) ; do
   j="${cmd[@]:$i:1}";
   j="${j//\"/\\\\\"}";
   cmd[$i]="\"${j}\"";
  done
  OFS=$IFS;
  IFS=,;
  ccmd="${cmd[*]//\//\\/}";
  echo " ccmd=$ccmd";
  IFS=$OFS;
  echo sed 's/\("Args":\["[^]]\+\]\)/"Args":['$ccmd']/g' config.v2.json
  sed "s/\(\"Args\":\[\"[^]]\+\]\)/\"Args\":[$ccmd]/g" ${logloc}/config.v2.json > /tmp/.t;
  mv /tmp/.t ${logloc}/config.v2.json;
  systemctl restart docker
}
function chcmd() {
  while true ; do
     read -p "> ";
     change_cmd $REPLY;
     docker start -i phonewikidb;
  done
}
```
To run the shell, replace PUT_YOUR_ID_HERE by your id, then use:
```
chcmd
```
