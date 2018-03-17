---
title: "Github: Querying with GraphQL"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - github
  - graphql
---

[GraphQL](http://graphql.org/) is a data query language developed by [Facebook](https://www.facebook.com/) and provides an alternative to [REST](https://en.wikipedia.org/wiki/Representational_state_transfer).

Github exposes multiple [REST API](https://developer.github.com/v3/) endpoints and a single [GraphQL](https://developer.github.com/v4) endpoint.
<!--more-->

The Github REST API returns a large payload containing a lot of information, if only a subset of information is required it is left to the calling client to parse and extract the relevant elements. GraphQL allows the client to retrieve more than one item of data and to return only the data that is relevant; this was one of the [advantages of implimenting GraphQL identified by the Github developers](https://githubengineering.com/the-github-graphql-api/).

GraphQL allows two types of operations queries and mutations.

Comparing GraphQL to REST
- queries operate like GET requests
- mutations operate like POST/PATCH/DELETE

The mutation name determines which modification is executed.

The article looks at using GraphQL to run simple queries against information stored in Github.

# Setup
Basic usage of Github GraphQL is covered here [https://developer.github.com/v4/guides/forming-calls](https://developer.github.com/v4/guides/forming-calls). For simplicity I'll distill the information and follow the same examples in [cURL](https://curl.haxx.se/) and Advanced REST Client.

While a token is not always required to use the Github REST API, a token IS required to use the GraphQL endpoint. Therefore, the first thing required is to [create a personal access token](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/).

Because GraphQL operations consist of multiline JSON, GitHub recommends using the [Explorer](https://developer.github.com/v4/explorer/) to make GraphQL calls. Having used cURL and Advanced REST Client I would agree with this. Before using Explorer you are required to sign in and provide authorisation, when that is done you are free to create and execute GraphQL calls.

A GraphQL query is represented in JSON as a property with a key named query and a string value containing the actual query itself.
```
 {
   "query": "query { viewer { login }}"
 }
```

## Making your first GraphQL call

### with GitHub Explorer
The simplest way to execute the query is to use a dedicated tool such as the Github Explorer.

![01-github-graphql-explorer]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-22-github-querying-with-graphql/01-github-graphql-explorer.jpg){: .align-center}

Explorer makes the GraphQL queries easy to construct but you do not get to see the actual query that is sent to the GraphQL endpoint such as the content of JSON body or any escaped characaters as that is automatically handled by Explorer.

### with cURL 
[cURL](https://curl.haxx.se/) is a very useful and flexible tool that amongst other things can be used to make calls to HTTP services.

For the first example I've choosen the most basic query in the Github documentation, before running this query you will need your personal access token
```
curl -H "Authorization: bearer <your personal access token>" -d " \
 { \
   \"query\": \"query { viewer { login }}\" \
 } \
" https://api.github.com/graphql
```
The BODY of the message is wrapped in double quotes which means we have to escape new lines and other occurrences of double quotes; the query is not so easy to read. The same query can be written using single quotes, you won't have to escape out double quotes or new lines
```
curl -H "Authorization: bearer <your personal access token>" -d ' 
 { 
   "query": "query { viewer { login }}" 
 } 
' https://api.github.com/graphql
```

Or by using a [here document](https://en.wikipedia.org/wiki/Here_document)
```
curl -H "Authorization: bearer <your personal access token>" https://api.github.com/graphql  \
-d @- << EOF
 { 
   "query": "query { repository(owner:\"octocat\", name:\"Hello-World\") { issues(last:20, states:CLOSED) { edges { node { title  }  } } } } "
 }
EOF
```

cURL can also read files on the file system too so if your GraphQL request covers a lot of lines it will save time if you write it with a text editor and persist it on the file system. 

### with Advanced REST client
Add the following HTTP headers
- Content Type
- User Agent
- Authorization, the authorization details are bearer `<your personal access token>`

The method is POST

![02-arc-graphql-header]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-22-github-querying-with-graphql/02-arc-graphql-header.jpg){: .align-center}

The query is represented as JSON and entered in the BODY of the request
```
{
  "query": "query { repository(owner:\"octocat\", name:\"Hello-World\") { issues(last:20, states:CLOSED) { edges { node { title url  labels(first:5) {  edges {  node {  name  }  }  }  } } } } }"
}
```

![03-arc-graphql-body]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-22-github-querying-with-graphql/03-arc-graphql-body.jpg){: .align-center}

## GraphQL Variables
Anyone who has written queries will know that at some point there will be a need for a dynamic query.

Building up a query with strings is one way to be dynamic, another is to formulate the query that accepts parameters or variables; GraphQL has provision for the use variables. GraphQL variables are represented as part of the JSON request sent to the GraphQL endpoint with a key named variables and an object value of key/value pairs.
```
"variables": { 
    "repo_owner": "octocat", 
    "repo_name": "Hello-World"
}
```

The variable is then passed into the query itself
```
query($repo_owner:String, $repo_name:String)
```

There are a couple of things to note
- All of the variables needed for this query are declared up-front
- Variables are typed
- All variables declared must be used in the query otherwise GraphQL regards the query as invalid

GraphQL comes with a set of default scalar types out of the box
- Int: A signed 32‐bit integer
- Float: A signed double-precision floating-point value
- String: A UTF‐8 character sequence
- Boolean: true or false
- ID: The ID scalar type represents a unique identifier, sometimes used to refetch an object or as the key for a cache

The ID type is serialized in the same way as a String; however, defining it as an ID signifies that it is not intended to be human‐readable.

### Mandatory values
Putting an exlaimation mark after the variable name means that the parameter value is required and cannot be null.
```
query($repo_owner:String!, $repo_name:String!)
```

### Default values
Default values can also be assigned to the variables in the query by adding the default value after the type declaration.
```
query($repo_owner:String = "octocat", $repo_name:String = "Hello-World")
```

### Using variables with cURL
When using cURL in bash the dollar sign needs to be escaped with a backslash otherwise bash will try and evaluate it as an environment variable.

```
curl -H "Authorization: bearer <your personal access token>" https://api.github.com/graphql  \
-d @- << EOF
 { 
   "query": "query(\$number_of_repos:Int!) {
	  viewer {
		name
		 repositories(last: \$number_of_repos) {
		   nodes {
			 name
		   }
		 }
	   }
	}",
	"variables": {
		"number_of_repos": 3
	}	
} 
EOF
```

Here is our very first query re-written using variables, passing in the repository owner and name
```
curl -H "Authorization: bearer <your personal access token>" https://api.github.com/graphql  \
-d @- << EOF
{
   "query": "query(\$repo_owner:String!, \$repo_name:String!)
   { 
		repository(owner:\$repo_owner, name:\$repo_name) 
		{ 
			issues(last:20, states:CLOSED) 
			{ 
				edges 
					{ node 
						{  title  url labels(first:5) 
						{ edges 
							{ node 
								{ name } 
							} 
						} 
					}
				} 
			} 
		} 
	}",
	"variables": { "repo_owner": "octocat", "repo_name": "Hello-World" }
}
EOF
```

### Using variables with ARC
Assuming you have set the appropriate HTTP headers and set the method as POST you can just add the JSON as the request body
```
{ "query": "query($number_of_repos:Int!) { viewer { name repositories(last: $number_of_repos) { nodes { name } } } }",
	"variables": { "number_of_repos": 3 }
} 
```

The first query re-written using variables
```
{
   "query": "query($repo_owner:String!, $repo_name:String!) { repository(owner:$repo_owner, name:$repo_name) { issues(last:20, states:CLOSED) { edges { node {  title  url labels(first:5) { edges { node { name } } } } } } } }",
  "variables": { "repo_owner": "octocat", "repo_name": "Hello-World" }
}
```
 
# Summary
It's not just Facebook and Github that use GraphQL either, it seems to be rapidly adopted elsewhere, the beauty is that once you've learned the basics it is easy to create queries for most endpoints that support GraphQL.

