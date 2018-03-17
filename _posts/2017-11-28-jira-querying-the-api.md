---
title: "Jira: Querying the API"
excerpt_separator: "<!--more-->"
last_modified_at: 2017-03-09T10:27:01-05:00
tags: 
  - jira
---

The JIRA URL for displaying issues resembles REST naming convention but the content returned is HTML, not very helpful unless you are prepared to parse the content. Fortunately [JIRA has a REST API](https://docs.atlassian.com/jira/REST/cloud/).

To access the JIRA REST API add a path of `/rest/asp/<api version>` to your JIRA server name
```
https://<jira server name>/rest/asp/<api version>
```

The JIRA REST API returns a JSON payload.
<!--more-->

The [JIRA REST API changes on a frequent basis](https://developer.atlassian.com/jiradev/jira-apis) and has multiple endpoints dictated by the version of the API so it's always worth staying up to date with the latest supported versions.

## Querying the API
The version of JIRA that I am using is an [on premise installation](https://confluence.atlassian.com/adminjiraserver071/installing-jira-applications-on-linux-802592173.html) rather than being hosted and is integrated with [Single Sign On SSO](https://en.wikipedia.org/wiki/Single_sign-on) via [Okta](https://www.okta.com/).

The JIRA API is accessible via username and password; the password is not the domain password but the password that is used to access JIRA, other installations may behave differently. 

The JIRA API can be called easily from the command line using cURL specify the username (curl will prompt for the password)
```
curl -u "<your JIRA username>" -H "Content-Type: application/json" https://<jira server name>/rest/api/latest/issue/<jira issue>
```

As an alternative to cURL tools such as Advanced REST client (or Postman) can also be used to query the JIRA API by providing the following HTTP headers
- content type
- user agent
- Authorization

To add the Authorisation header, add a new HTTP header, select Authorization, click on the pencil to fill in the details

![01-arc-basic-authorization]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-28-jira-querying-the-api/01-arc-basic-authorization.jpg){: .align-center}

Run the query

![02-arc-run-jira-query]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-28-jira-querying-the-api/02-arc-run-jira-query.jpg){: .align-center}

When using Basic Authorisation if the username or password provided to the JIRA REST API is incorrect after several attempts JIRA may reject future request and re-direct you to a Captcha screen. If your request is rejected the response header will contain an X-Seraph-LoginReason with a value of [AUTHENTICATION_DENIED or AUTHENTICATION_FAILED](https://developer.atlassian.com/server/jira/platform/basic-authentication/).

Any re-directs can be traced when using cURL by using  -L parameter and can see the rendered Captcha page.

## Returning back what you need
The JIRA REST API allows you to filter to selective fields using the fields parameter like below
```
https://<jira server name>/rest/api/latest/issue/<issue>?fields=<field list>
```

![03-jira-rest-specify-fields]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-28-jira-querying-the-api/03-jira-rest-specify-fields.jpg){: .align-center}

As JIRA can be customised the JIRA REST API provides an endpoint to return the fields are available so it is easy to include non standard fields in the response payload.
```
https://<jira server name>/rest/api/2/field
```

![04-jira-rest-custom-fields]({{ site.url }}{{ site.baseurl }}/assets/posts/2017-11-28-jira-querying-the-api/04-jira-rest-custom-fields.jpg){: .align-center}

The [JIRA REST API does not limit you to queries only](https://developer.atlassian.com/server/jira/platform/jira-rest-api-examples/); new items such as issues can be added too.

## Summary
The purpose of investigating the Github and JIRA API's is to provide a simple tool to be able to determine what code exists on different Git branches and relate the commits to work items in JIRA. Tool is now in place and working well.