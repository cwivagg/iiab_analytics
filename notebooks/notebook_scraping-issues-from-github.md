## Data Retrieval and Cleaning
First, we retrieved all of the issues submitted to the IIAB repo. We can look [here](https://github.com/iiab/iiab/issues) to see that the most recent issue. In our pull (19 December 2021), the most recent issue was #3071. Since the GitHub API allows us to retrieve at most 100 issues per request, we submitted 31 requests to collect all of the issues. More details, including on how we created the authorization token, are available on GitHub's excellent [documentation](https://docs.github.com/en/rest/guides/getting-started-with-the-rest-api).

```
for i in {1..31}; do curl -H "Authorization: token $token" "https://api.github.com/repos/iiab/iiab/issues?page=$i&state=all&per_page=100" >> iiab_issues.json; done
```

IIAB uses Sprunge to collect logging information to help in troubleshooting. Among other things, the log contains the list of modules the user has installed. First, we collected all of the Sprunge links submitted in the IIAB issues.

```
grep -oE "http\:\/\/sprunge\.us\/[a-zA-Z0-9]+" iiab_issues.json|sort|uniq > sprunge_links.txt
```

Since the data size was small, we were able to run another command to look through the link contexts and make sure we weren't missing any malformed links.

```
cat iiab_issues.json|jq -Sc '.[].body'|grep sprunge| wc -l
```

Next, we retrieved the logs posted to Sprunge for each issue.

```
mkdir logs
while read s; do echo $s; curl $s > logs/${s:18:6}.txt; sleep 10; done<sprunge_links.txt
```

After retrieving the logs, we did some simple parsing to get the contents of the ```menus.json``` file. We confirmed that the parsing expression didn't miss any files by counting the number of occurrences of the distinctive token "menu_items_1", which is the key within the JSON that lists the modules the user has selected.
```
cd logs
cat *.txt|awk '/^\{$/,/^[[:space:]]*\}$/'|jq '.' -Sc > ../menus.json
cd ..
```

In addition to the Sprunge logs, we noted two occurrences of the "menu_items_1" token in the body of two IIAB issues. Both occurrences were associated with a marked-up copy of the ```menus.json``` file. We put parsing these on the to-dos list for later.
