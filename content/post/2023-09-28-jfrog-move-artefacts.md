---
date: "2023-09-28T22:55:00Z"
title: How to move artefacts in a Jfrog repository
---

This one will be a quick one. Recently I had to move several hundred artefacts from a Jfrog repo folder up one level. Easy peasy, right? But the problem was that manual work from GUI would take forever and my work time is limited so I had to find a different way.

For this I've used [an API call](https://jfrog.com/help/r/jfrog-rest-apis/move-item) because I didn't have access to jfrog cli tool.

But the API call works for one file, not for a bulk move. And I had **many** to move.

Luckily I also had the RPM files in a folder so with some bash-fu I was able to put up together a list of RPMs. Then I iterated through that list and ran the API call for each one. And that was it. A simple a fast way to migrate artefacts from a Jfrog repository to a different place.

With this command I moved them from the externals folder up one level:
```
➤ for file in $(cat ~/externals-files.txt); do curl --keepalive-time 10 --max-time 18000 -vv -u$MYUSER:$MYPASS -X POST https://$JFROG_FQDN:443/artifactory/api/move/$REPO_NAME/externals/$file?to=$REPO_NAME/$file; done
```

If you'd prefer a more conservative approach, you could append `dry=1` to the URL, or replace `move` with `copy`. This would copy the files to the new location.


And if you want to upload new artefacts, you can use this API call:
```
for file in json5-0.9.11-py2.py3-none-any.whl; do curl --keepalive-time 10 --max-time 18000 -vv -u$MYUSER:$MYPASS -X PUT https://$JFROG_FQDN:443/artifactory/$REPO_NAME/externals/$file -T $file; done
```
