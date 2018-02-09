# Maintainance

* no support, for me it works ok :)
* has docker handling forthis image centos/mongodb-32-centos7
* fixed daily + hourly + weekly backup
* added debug, and verbosity

## Docker

You can mongodump without any mongo binary on the host, it will use docker exec to realize the dump inside the container.
You have to call the script from outside docker and have a volume mounted to get the backups

You will need to configure this


* in first docker specific section
you will need to activate the docker discovery lines, note the second, is the specific path fo mongodump on centos/mongodb-32-centos7 image on dockerhub, it could be easily patched.
DOCK=`docker inspect --format='{{.Name}}' ..........
MONGODUMP="docker exec -it $DOCK /opt/rh/rh-mongodb32/root/u......

* in second docker specific section, 184:997 are taken from the centos/mongodb-32-centos7 image on dockerhub
```
mkdir -p $BACKUPDIR
chmod g+w $BACKUPDIR
chmod g+s $BACKUPDIR
chown 184:997 -R $BACKUPDIR
```

* volume for backup like this
```
 mongodb:
    image: centos/mongodb-32-centos7
    environment:
      MONGODB_ADMIN_PASSWORD: yyyyyyyyy
      MONGODB_DATABASE: pythie
      MONGODB_PASSWORD: xxxxxxxxxxxx
      MONGODB_USER: crawler
    stdin_open: true
    volumes:
    - /var/lib/mongodb/data:/var/lib/mongodb/data
    - /var/backups/mongo:/var/backups/mongo
    tty: true
    ports:
    - 27017:27017/tcp
```

script called likes this

`bash ~/automongobackup.sh`
```
bash /root/automongobackup_docker_hmx.sh
creating dirs
/var/backups/mongo/127.0.0.1-1534.log
======================================================================
AutoMongoBackup VER 0.11

Backup of Database Server - XXXXXXXXXXX on 127.0.0.1
======================================================================
Backup Start Wed Jan 31 15:34:47 UTC 2018
======================================================================
Daily Backup of Databases
DEBUG   docker exec -it mongo /opt/rh/rh-mongodb32/root/usr/bin/mongodump --host=127.0.0.1:27017 --out='/var/backups/mongo/daily/2018-01-31_15h34m.Wednesday'  --username=pippo --password=xxxxxxxxx --authenticationDatabase= -d test_coll
2018-01-31T15:34:47.202+0000	writing test_coll.creative_fingerprints to 
2018-01-31T15:34:47.202+0000	writing test_coll.matched_creatives to 
2018-01-31T15:34:47.202+0000	writing test_coll.creatives to 
2018-01-31T15:34:47.205+0000	done dumping test_coll.matched_creatives (181 documents)
2018-01-31T15:34:47.234+0000	done dumping test_coll.creatives (23 documents)
2018-01-31T15:34:47.240+0000	done dumping test_coll.creative_fingerprints (5964 documents)
Tar and gzip to 2018-01-31_15h34m.Wednesday.tgz
Cleaning up folder at /var/backups/mongo/daily/2018-01-31_15h34m.Wednesday

Hourly Backup of Databases
DEBUG   docker exec -it mongo /opt/rh/rh-mongodb32/root/usr/bin/mongodump --host=127.0.0.1:27017 --out='/var/backups/mongo/hourly/2018-01-31_15h34m.Wednesday.1517412886'  --username=pippo --password=xxxxxxxx--authenticationDatabase= -d test_coll
2018-01-31T15:34:47.445+0000	writing test_coll.creative_fingerprints to 
2018-01-31T15:34:47.445+0000	writing test_coll.matched_creatives to 
2018-01-31T15:34:47.446+0000	writing test_coll.creatives to 
2018-01-31T15:34:47.448+0000	done dumping test_coll.matched_creatives (181 documents)
2018-01-31T15:34:47.474+0000	done dumping test_coll.creatives (23 documents)
2018-01-31T15:34:47.484+0000	done dumping test_coll.creative_fingerprints (5964 documents)
Tar and gzip to 2018-01-31_15h34m.Wednesday.1517412886.tgz
Cleaning up folder at /var/backups/mongo/hourly/2018-01-31_15h34m.Wednesday.1517412886
----------------------------------------------------------------------
Backup End Time Wed Jan 31 15:34:47 UTC 2018
======================================================================
Total disk space used for backup storage..
Size - Location
1.2M	/var/backups/mongo

======================================================================
```


## AutoMongoBackup

This is a very barebones port of the "AutoMySQLBackup":http://sourceforge.net/projects/automysqlbackup/ project. This bash script will allow you to do daily backups of a Mongo database. It includes periodic rotation so that you can keep historical weekly and monthly backups, as well as daily.

## License

This is released under GPL version 2.
