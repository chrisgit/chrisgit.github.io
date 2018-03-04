---
title: "Github: Querying the API"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - github
---

The Git commands work well on a local repository and if you keep your source up to date you can query almost anything. If you use Github as a remote or centralised repository to store your source code then you can use the [GitHub API](https://developer.github.com/v3/) to query your repositories.
<!--more-->
## Calling the Github API
Github API can be called from the command line with curl (installed on Windows as part of Git Bash) or a browser based REST plugin, below is an example of querying the Github API asking for details about a particular user
```
curl https://api.github.com/users/chrisgit
```

![01-curl-query-user]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/01-curl-query-user.jpg){: .align-center}

## Private repositories
If you try and use the Github API to access the contents of a private repository you will get a 404 Not Found.

![02-curl-query-private-repo]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/02-curl-query-private-repo.jpg){: .align-center}

To access a private repository, you will need to be authenticated, with [curl you can use basic authentication](https://ec.haxx.se/http-auth.html) with username and password by adding the -u <username> parameter, you will be prompted for the password.

```
curl -i -u <username> https://api.github.com/repos/<owner>/<repo>
```

If using multi factor authentication then the password needs to be replaced by a [Github authorisation token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/), otherwise using basic authorisation will result in a 401 and an appropriate message (below)

![03-curl-query-private-repo-mfa]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/03-curl-query-private-repo-mfa.jpg){: .align-center}

Creating a token to call the GitHub API is relatively simple
> GitHub -> Settings -> Developer Settings -> Personal Access Tokens

After creating a token it needs to be stored in a vault (such as LastPass or Windows Credential Manager) or held locally, the tokens can be deleted and rotated if necessary. Viewing the repository using a token is achieved in curl by passing Authorization http header.
```
curl -H "Authorization: <token>" 
```

![04-curl-query-private-repo-with-token]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/04-curl-query-private-repo-with-token.jpg){: .align-center}

### Chrome advanced REST client
As an alternative to curl you can use browser developer tools or browser applications such as Advanced REST client or Postman for Chrome.

Advanced REST client or Postman can be installed from the [Chrome web store](https://chrome.google.com/webstore/category/extensions)

For the rest of this article I shall use Advanced REST Client (ARC).

The Advanced REST client will automatically launch after being installed, to launch the Advanced REST client in subsequent sessions open Chrome, in the address bar type 
```
chrome://apps/
```
press return and select ARC; 

![05-chrome-apps]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/05-chrome-apps.jpg){: .align-center}

ARC can then be used to make requests to the GitHub API

![06-arc-query-user]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/06-arc-query-user.jpg){: .align-center}

The Github API requires the [User-Agent header](https://developer.github.com/v3/#user-agent-required) to be set, this can be anything (unless you are testing specific cases), for the purposes of this example it has been set to Mozilla/5.0.

There are two other https headers that need be set
- Content-Type (for the response)
- Authorization (value containing the password or token when querying private repositories)

When using ARC is it very simple to change the request URL or add/remove headers, body etc.

The GitHub API restricts the payload by paginating data; [the default page size is 30 items](https://developer.github.com/v3/guides/traversing-with-pagination/) but this can be changed up to a maximum of 100 items.

If the results from the Github API query spans across multiple pages the response header will contain an entry named link; the link header will contain URL’s to the next and last pages.
```
link: 
  <https://api.github.com/repositories/115740959/commits?page=2>; rel="next", 
  <https://api.github.com/repositories/115740959/commits?page=4>; rel="last"
```

Using curl to view headers only we can see the Link header
 ```
curl -I https://api.github.com/repos/chrisgit/ruby-bitmap_editor/commits
 ```
![07-curl-headers-only-link-header]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/07-curl-headers-only-link-header.jpg){: .align-center}

Github API returns a lot of verbose information, there does not appear to be a way you can restrict the number of fields being return, instead this is left to accessing [Github via GraphQL](https://developer.github.com/v4/). Fortunately [GraphQL](http://graphql.org/) can be called from [curl or a browser](http://graphql.org/learn/) like a REST service.

### Git API cheat sheet
Examples will use Octocat repository

#### Information about a repository
https://api.github.com/repos/owner/repo
```
curl https://api.github.com/repos/octocat/Spoon-Knife
```

#### Compare two branches
https://api.github.com/repos/owner/repo/compare/branch1...branch2
```
curl https://api.github.com/repos/octocat/Spoon-Knife/compare/master...test-branch
```

#### Show the commits (for the master branch)
https://api.github.com/repos/owner/repo/commits
```
curl https://api.github.com/repos/octocat/Spoon-Knife/commits
```
Return 50 items per page and show the second page of commits
```
curl https://api.github.com/repos/octocat/Spoon-Knife/commits?page=2&per_page=50
```

#### List branches on the repository
https://api.github.com/repos/owner/repo/branches
```
curl https://api.github.com/repos/octocat/Spoon-Knife/branches
```

#### View details of a branch
https://api.github.com/repos/owner/repo/branches/branch
```
curl https://api.github.com/repos/octocat/Spoon-Knife/branches/test-branch
```

#### View commits of a branch
https://api.github.com/repos/owner/repo/commits?sha=branch 
```
curl https://api.github.com/repos/octocat/Spoon-Knife/commits?sha=test-branch
```

#### View commits of a branch between two dates
https://api.github.com/repos/owner/repo/commits?sha=branch&since=date&until=date
```
curl https://api.github.com/repos/octocat/Spoon-Knife/commits?sha=test-branch&since=2014-01-01&until=2014-12-31
```

#### Search for commits against src files
https://api.github.com/search/commits?q=repo:owner/repo+src

```
curl -H "Accept: application/vnd.github.cloak-preview" https://api.github.com/search/commits?q=repo:octocat/Spoon-Knife+src
```

```
curl -H "Accept: application/vnd.github.cloak-preview" https://api.github.com/search/commits?q=repo:octocat/Spoon-Knife+css
```
NB: you will need to add an accept HTTP header of [Accept application/vnd.github.cloak-preview](https://developer.github.com/v3/search/)

![08-arc-search-commits-accept-header]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-21-github-querying-the-api/08-arc-search-commits-accept-header.jpg){: .align-center}
