# bosh-nfsserver
A NFS server bosh release for use with PCF Volume Service

## HowTo create the bosh release

To initialize the project
```
bosh init-release
bosh generate-job nfsserver

# we don't need blobs
bosh  blobs
```

To build a release tgz
```
bosh create-release --force --timestamp-version --tarball nfsserver.tgz
```

## HowTo deploy
```
bosh upload-release
bosh releases | grep nfs
```

customize local.yml and deploy

```
bosh interpolate -l local.yml manifest.yml > manifest-real.yml
bosh deploy manifest-real.yml -d nfsserver
```

explore with

```
bosh ssh -d nfsserver
sudo su -
monit summary
df -h
cd /var/vcap/store/nfsserver/
```

You will note that the user vcap, group vcap exists in BOSH deployed VM with uid 1000 and gid 1000.
The persistent volume /export/vol1 (for example - see the jobs/ folder to change) is rw for vcap.
Note that this is different uid/gid from the Diego vcap container user which is uid 2000 gid 2000.
The nfsserver self-mount into /export/vol1 as test (for example - see the jobs/ folder to change)

## Sample app

You can explore pora
https://github.com/cloudfoundry/cf-volume-services-acceptance-tests/tree/master/assets/pora

```
# After it is enabled in PAS with also the NFS service broker deployed thru the errand
# with configuration in OpsManager

# as cf admin
cf enable-service-access nfs

# as a user I must know the address and info for my persistent volume NFS server and path
cf create-service nfs Existing nfs-demo -c '{"share": "{nfsserver-ip}/export/vol2"}'

cf push --no-start

# We use the NFS uid/gid here
cf bind-service pora nfs-demo -c '{"uid":"1000","gid":"1000"}'

cf restage pora

cf logs pora --recent
   (use the application http route and access its /create endpoint)


```


