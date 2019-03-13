+++
date = "2019-03-13T03:00:00+00:00"
draft = true
showDate = true
tags = []
title = "Mini SK4: Creating the \"database\""

+++
As a part of the \[Mini SK4\] project, I needed a place to store the relationship between the NFC tag on the photo slide and the URI of the music.

1. hugo new site [name]
1. hugo new theme [name]
  1. Only needed so hugo can build the site, we're not actually going to use this. 
1. Edit the base config

You now have the most minimal hugo install!

1. Update the config with the JSON output
1. Add the layout file
1. Add content, hugo new [subdir]/[file-name].md
  1. The front matter needs to match what is in your layout file.

And now you have a local JSON API.

Deploying

1. Set-up a git repo and deploy it to your GitHub account.
1. Create a `gh-pages` branch and push to there as well.
1. Turn on GitHub Pages in settings

Forestry
1. Sign up and import your site
1. Adjust the deploy settings to build and use `gh-pages`
1. Setup your front matter and create a new page

Enjoy your free JSON API with an admin interface!

```
baseURL = "https://benortiz.github.io/playlist/"
languageCode = "en-us"
title = "Playlist JSON API"
theme = "themeless"

[outputs]
  section = ["json"]
```

```
# layout/_default/list.json.json
{{- .Scratch.Set "items" slice -}}
{{- range .Pages -}}
    {{- $.Scratch.Add "items" (dict "collectionId" .Params.collectionId "tagId" .Params.tagId) -}}
{{- end -}}
{{- .Scratch.SetInMap "data" "items" (.Scratch.Get "items") -}}
{{- .Scratch.SetInMap "output" "data" (.Scratch.Get "data") -}}
{{- .Scratch.Get "output" | jsonify -}}
```