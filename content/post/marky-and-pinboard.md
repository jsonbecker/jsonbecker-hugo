---
title: "Pindown: Failed Dreams"
author: Jason P. Becker
date: 2014-05-13
tags: pinboard, markdown, python, code
---

I had never thought of a use for [Brett Terpstra's](http://www.brettterpstra.com) [Marky the Markdownifier](http://heckyesmarkdown.com) before listening today's [Systematic](http://5by5.tv/systematic/96). Why would I want to turn a webpage into Markdown?

When I heard that Marky has an API, I was inspired. [Pinboard](http://pinboard.in) has a "description" field that allows up to 65,000 characters. I never know what to put in this box. Wouldn't it be great to put the full content of the page in Markdown into this field?

I set out to write a quick Python script to:

1. Grab recent Pinboard links.
2. Check to see if the URLs still resolve.
3. Send the link to Marky and collect a Markdown version of the content.
4. Post an updated link to Pinboard with the Markdown in the description field.

If all went well, I would release this script on Github as **Pindown**, a great way to put Markdown page content into your Pinboard links.

The script below is far from well-constructed. I would have spent more time cleaning it up with things like better error handling and a more complete CLI to give more granular control over which links receive Markdown content. 

Unfortunately, I found that Pinboard consistently returns a [414 error code](http://www.checkupdown.com/status/E414.html) because the URLs are too long. Why is this a problem? Pinboard, in an attempt to [maintain compatibility](https://pinboard.in/api/) with the [del.ico.us](http://del.ico.us) API uses only GET requests, whereas this kind of request would typically use a POST end point. As a result, I cannot send along a data payload.

So I'm sharing this just for folks who are interested in playing with Python, RESTful APIs, and Pinboard. I'm also posting for my own posterity since a [non-Del.ico.us compatible version 2 of the Pinboard API is coming](https://groups.google.com/d/msg/pinboard-dev/Od6sCzREeBU/L-WKgX6vUDoJ).

```python
import requests
import json
import yaml


def getDataSet(call):
  r = requests.get('https://api.pinboard.in/v1/posts/recent' + call)
  data_set = json.loads(r._content)
  return data_set

def checkURL(url=""):
  newurl = requests.get(url)
  if newurl.status_code==200:
    return newurl.url
  else:
    raise ValueError('your message', newurl.status_code)

def markyCall(url=""):
  r = requests.get('http://heckyesmarkdown.com/go/?u=' + url)
  return r._content

def process_site(call):
  data_set = getDataSet(call)
  processed_site = []
  errors = []
  for site in data_set['posts']:
    try:
      url = checkURL(site['href'])
    except ValueError:
      errors.append(site['href'])
    description = markyCall(url)
    site['extended'] = description
    processed_site.append(site)
  print errors
  return processed_site

def write_pinboard(site, auth_token):
  stem = 'https://api.pinboard.in/v1/posts/add?format=json&auth_token='
  payload = {}
  payload['url'] = site.get('href')
  payload['description'] = site.get('description', '')
  payload['extended'] = site.get('extended', '')
  payload['tags'] = site.get('tags', '')
  payload['shared'] = site.get('extended', 'no')
  payload['toread'] = site.get('toread', 'no')           
  r = requests.get(stem + auth_token, params = payload)
  print(site['href'] + '\t\t' + r.status_code)

def main():
  settings = file('AUTH.yaml', 'rw')
  identity = yaml.load(AUTH.yaml)
  auth_token = identity['user_name'] + ':' + identity['token']
  valid_sites = process_site('?format=json&auth_token=' + auth_token)
  for site in valid_sites:
    write_pinboard(site, auth_token)

if __name__ == '__main__':
  main()
```
