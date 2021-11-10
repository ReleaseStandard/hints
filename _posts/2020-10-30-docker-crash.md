---
title: "Rescue crashed docker container"
author: "Release Standard"
---
This is a guide on how to rescue a crashed docker container, focused on one example : linuxserver/mariadb, but could be applied to others with few changes.



## The issue

 - on the container running linuxserver/mariadb.

 - i could not start the container anymore, because of mysql start crash (no docker exec !).

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

