# ocd-tagger

This folder has the s2i/bin/assemble scripts that that will tag an image with the git tag that built the image. It is installed using helm as a subcomponent of [ocd-builder](https://github.com/ocd-scm/ocd-builder)

With secure openshift clusters such as openshift.com you are not able to run builds of type Custom or Docker. This means you can only run s2i builds. Your main `ocd-builder` application build will pus to your apps latest image stream `$APP:latest`. The `oc-tagger` build will be triggered by pushes to `$APP/latest` so will run after your main build. The tagger build holds a `.s2i/bin/assemble` that calls `oc` to inspect `$APP:latest` to figure out the git tag it was built with. It then applies the same tag with `oc tag $APP:latest $APP:$VERSION`. 

If you are using ocd-chatbot it is helpful to have the tag build announce when it has finished. A big react build might take a couple of minutes and the tag build can take another 20s or so. Getting a success or failure announcement lets you know that you can tell the chatbot to promote your tagged app. To do this you need to base64 encode the announce script into the env var `CHAT_NOTIFY` which must be a script that sends `$MESSAGE` to your chatroom. 

For example with slack you would create an [inbound webhook](https://api.slack.com/incoming-webhooks#create_a_webhook) that will be shown to as a POST URL like this containing your unique IDs:

```
https://hooks.slack.com/services/Txxxxxx2/Buyyyyyy2/vzzzzzzzzzzzzzv
```

You then need to create a script such as `announce.sh` that posts $MESSAGE to that URL such as:

```
curl -X POST -H 'Content-type: application/json' --data "{\"text\":\"${MESSAGE}\"}" https://hooks.slack.com/services/Txxxxxx2/Buyyyyyy2/vzzzzzzzzzzzzzv
```

Once you have tested that that script works you then base64 encoded it with `base64 announce.sh` to give something like:

```
Y3VybCAtWCBQT1NUIC1IICdDb250ZW50LXR5cGU6IGFwcGxpY2F0aW9uL2pzb24nIC0tZGF0YSAie1widGV4dFwiOlwiJHtNRVNTQUdFfVwifSIgaHR0cHM6Ly9ob29rcy5zbGFjay5jb20vc2VydmljZXMvVHh4eHh4eDIvQnV5eXl5eXkyL3Z6enp6enp6enp6enp6dgo=
```

then define that payload as an env var on your tag build called `CHAT_NOTIFY`. The tag build will check if there is an env var `CHAT_NOTIFY` and if so run `base64 -d` on it to get your announce.sh, chmod it, and use it announce the tag build has finsihed. 
