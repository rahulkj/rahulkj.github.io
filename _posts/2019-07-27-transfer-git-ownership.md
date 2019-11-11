---
layout: post
title:  "Transfer all git repos to another org"
date:   2019-07-27 05:28:00 -0600
categories: git ownership
---

I wrote a handy script to transfer ownership of all repositories that belong to a user, to another user/org

The idea here is simple, lookup for all the repos that belongs to that user, loop through all the repos, and then migrate those to the other user/org

```
#!/bin/bash -x

export GIT_USER=
export GIT_TOKEN=
export ORG_NAME=
export NEW_ORG_NAME=

export OUTPUT_FILE=repos.json

rm -rf $OUTPUT_FILE
curl -u $GIT_USER:$GIT_TOKEN "https://api.github.com/users/$ORG_NAME/repos" -s > $OUTPUT_FILE

REPO_NAMES=$(cat $OUTPUT_FILE | jq -r '.[] | .name')

for REPO in $REPO_NAMES
do
  echo "repo is: $REPO"

  curl -u $GIT_USER:$GIT_TOKEN "https://api.github.com/repos/$ORG_NAME/$REPO/transfer" \
    -X POST -d '{"new_owner": "'"$NEW_ORG_NAME"'"}' \
    -H "Accept: application/vnd.github.nightshade-preview+json"
done

rm -rf $OUTPUT_FILE
```

This script can be found at https://gist.github.com/rahulkj/71db9bb451f578840b4d062cc0347be7

Happy transferring and maintaining your repositories!!
