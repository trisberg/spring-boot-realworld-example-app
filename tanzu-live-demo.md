# Tanzu Live Demo

1. Go to TAP GUI Create
2. Select "TAP Project Configuration" accelerator
    - Name of target project: tanzu-live
    - Git Repository: https://github.com/gothinkster/spring-boot-realworld-example-app
    - Git Branch: master
3. Unzip `~/Downloads/tap-initialize.zip`

## Deploy:

Create workload and prevent the servcie from scaling down

```
tanzu apps workload create tanzu-live -f ./tap-initialize/config/workload.yaml --live-update --tail
```

Check workload status:

```
tanzu apps workload get tanzu-live
```

## Test App:

Get the URL:

```
URL=$(kubectl get service.serving.knative.dev tanzu-live -ojsonpath='{.status.url}')
```

Get any tags (shouldn't be any):

```
curl ${URL}/tags
```

Create user:

```
curl -v -L ${URL}/users -H 'Content-Type: application/json' -H 'X-Requested-With: XMLHttpRequest' \
--data-raw '{"user": {"email": "john@jacob.com","password": "johnnyjacob", "username": "john"}}'
```

Log in and get token:

```
TOKEN=$(curl -s -L ${URL}/users/login -H 'Content-Type: application/json' -H 'X-Requested-With: XMLHttpRequest' --data-raw '{"user": {"email": "john@jacob.com", "password": "johnnyjacob"}}' | jq -r '.user.token')
```

Add an article:

```
curl -v -L ${URL}/articles -H 'Content-Type: application/json' -H "Authorization: Bearer $TOKEN" -H 'X-Requested-With: XMLHttpRequest' \
--data-raw '{
  "article": {
    "author": "Thomas",
    "body": "Testing TAP",
    "comments": [],
    "createdAt": "2022-01-02",
    "description": "Just a bit of testing",
    "favorited": true,
    "favoritesCount": 0,
    "slug": "test",
    "tagList": ["test", "tap", "tanzu", "live"],
    "title": "Testing",
    "updatedAt": "2022-01-04"
  }
}'
```

See if we can see article and tags:

``` 
curl ${URL}/articles 
curl ${URL}/tags
```

## Commit TAP artifacts to Git repo

Fork the [gothinkster](https://github.com/gothinkster/spring-boot-realworld-example-app) repo and check that out to your working directory.

Something like:

```
git clone git@github.com:trisberg/spring-boot-realworld-example-app.git
```

Copy `config` and `catalog` files to our fork:

```
cp -r ./tap-initialize/config ./spring-boot-realworld-example-app/.
cp -r ./tap-initialize/catalog ./spring-boot-realworld-example-app/.
```

Commit these added files:

```
cd ./spring-boot-realworld-example-app
git add config/ catalog/
git commit -m "Add demo artifacts"
git push origin
```

## Add catalog-info.yaml to TAP GUI's Backstage Catalog

Select [REGISTER ENTITY] from Home page.

Enter URL as 

```
https://github.com/trisberg/spring-boot-realworld-example-app/blob/demo/catalog/catalog-info.yaml
```

## Delete the workload

```
tanzu apps workload delete tanzu-live
```

## Remove the catalog entry

> Doesn't seem possible ATM but we can bump the tap-gui server pod

```
kubectl -n tap-gui delete pods -l=component=backstage-server
```