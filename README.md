你好！
很冒昧用这样的方式来和你沟通，如有打扰请忽略我的提交哈。我是光年实验室（gnlab.com）的HR，在招Golang开发工程师，我们是一个技术型团队，技术氛围非常好。全职和兼职都可以，不过最好是全职，工作地点杭州。
我们公司是做流量增长的，Golang负责开发SAAS平台的应用，我们做的很多应用是全新的，工作非常有挑战也很有意思，是国内很多大厂的顾问。
如果有兴趣的话加我微信：13515810775  ，也可以访问 https://gnlab.com/，联系客服转发给HR。
A FriendFeed Clone
==================

  This project created by a handful of FriendFeed enthusiasts, as FriendFeed
  was shutting down[1].

Google Cloud
============

You may need to create bucket in console.developers.google.com.

Init Google Cloud

    curl https://sdk.cloud.google.com | bash

Login, eg:

    gcloud auth login
    gcloud config set project "GCEAppId"

Set storage bucket to public

    gsutil defacl ch -g AllUsers:R gs://GCSBucketId

SSH

    gcloud compute --project "GCSAppId" ssh --zone "us-central1-f" "instance-1"

Attach Disk
==========
    gcloud compute zones list
    gcloud compute regions describe us-central1
    gcloud compute regions list

    // create then select regoin
    gcloud compute disks create ffdb --size 200GB
    gcloud compute instances attach-disk "instance-1" --disk ffdb --zone "us-central1-f"

  ssh to instance

    sudo mkdir /mnt/tmp
    sudo /usr/share/google/safe_format_and_mount -m "mkfs.ext4 -F" /dev/sdb /mnt/tmp

    sudo sh -c 'cat "/dev/sdb /srv ext4 noatime 0 0" >> /etc/fstab'
    sudo mount /srv


Firewall
========

If you need remote clients, you need to create gcloud firewall rules:

    gcloud compute firewall-rules list
    gcloud compute firewall-rules create ffapi --allow tcp:8901 --source-tags=instance-1 --source-ranges=REMOTE_IP/32 --description="ffapi"


Golang Env
==========

    sudo mkdir -p /srv/gopath/bin
    echo "export GOPATH=/srv/gopath" >> ~/.bashrc
    echo "export PATH=$GOPATH/bin:$PATH" >> ~/.bashrc

    cd ~/src && curl -L https://godeb.s3.amazonaws.com/godeb-amd64.tar.gz \
         | tar zx --strip 1 && ./godeb install 1.4.2
    mkdir /srv/gopath/bin && mv godeb /srv/gopath/bin/

    sudo apt-get install git-core -y
    sudo apt-get install imagemagick -y


Media Server
============

  Media files will be archived to Google Cloud Storage if it was from friendfeed.

    cp conf/example.media.json conf/media.json

  Change media.json according to your project.  

RocksDB
=======

    fab production deploy_env

Google OAUTH2
============

 * console.developers.google.com -> APIs & auth -> Consent Screen -> must have
   email 
 * Creadentials -> Create new client ID
 * Place json key file to conf/gauth.json

Web Dev
=======

    cd httpd
    go get .

Start develop

    go build; ./httpd -f=../conf/gauth.json -d

Gin(not working)
        
    go get github.com/codegangsta/gin
    export DEBUG=1
    export RPC_ADDRESS=localhost:8901
    export GoogleKeyFile=../conf/gauth.json
    gin -p 8080

Deploy FriendFeed
=================

You need fabric in your local machine if you run fab.

    sudo apt-get install -y fabric


Setup once

  * Create Google Cloud Engine
  * Create Google Cloud Storage Bucket, config file save to conf/gcs.json
  * Media config as described in previous section.

```
    openssl rand 40 -base64 > conf/salt.conf
    fab production bootstrap
    fab production deploy_env
    fab production deploy_config
```

SSL && Nginx

    fab production deploy_ssl
    fab production deploy_nginx

Routine update

    fab production deploy
    fab production deploy_client
    fab production deploy_web

deploy_client only start one client, if you need more, start ffclient manually.

[1] http://blog.friendfeed.com/2015/03/dear-friendfeed-community-were.html
