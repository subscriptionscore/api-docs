---
title: Subscription Score Insights API reference

language_tabs: # must be one of https://git.io/vQNgJ
  - graphql
  - javascript

toc_footers:
  - <a href='mailto:hi@subscriptionscore.com'>Sign Up for an API Key</a>
  - <a style="opacity:0.25" href='https://github.com/slatedocs/slate'>Documentation Powered by Slate</a>

# includes:
#   - errors

search: true
---

# Introduction

Welcome to the <a href="https://subscriptionscore.com">Subscription Score</a> Insights API! You can use our API to access our score API endpoints, which can get you information on Domains and Email addresses.

We have language bindings in Shell and JavaScript. You can view code examples in the dark area to the right, and you can switch the programming language of the examples with the tabs in the top right.

The API uses <a href="https://graphql.org/">GraphQL</a> so that you can retrieve exactly the attributes that you need. A basic understanding of GraphQL is recommended, but not required to intgrate with the API.

# Considerations

## HTTPS

All requests must take place over HTTPS, the API will not respond to any requests made over HTTP.

## Emails

By default email addresses associated with mailing lists are hashed with a SHA-1 function before they are sent. This is for two reasons;

1. We wish to avoid exposing email addresses during transfer when at all possible.
2. Email addresses are stored as SHA-1 hashes in our database.

If you wish to send email addresses in the clear anyway then please provide the `cleartext: true` parameter with your query. The email address will be hashed before it is queried, so the outcome will be the same.

Note: if we return an email address with our response, it will be hashed with SHA-1.

## Discovery

By design, it should not be possible to discover email addresses associated with a domain by using the API. If you think we are exposing email addresses in any way then [please let us know](mailto:hi@subscriptionscore.com).

## Aliases

Some mailing list providers send emails from a different domain to the one they use as their primary web presence. For example, `facebook.com` sends all it's mail from `@facebookmail.com` addresses.

To make it easier to query mailing lists based on their primary domain we maintain a list of known aliases. This means that a [`domainScore`](#domain-score) query for `facebook.com` will return the same value as a query for `facebookmail.com`.

## Email Providers

A great many mailing lists are sent from generic email provider domains. For example, popular apps such as Gmass allow users to schedule and automate emails directly within Gmail, meaning that any Gmail user can send subscription emails from `@gmail.com` addresses.

Due to this, using the [`domainScore`](#domain-score) query against such domains as `gmail.com`, `outlook.com`, or any other email provider, would essentially yield a useless outcome. Therefore we don't provide this functionality for these domains and the result of such a query will always be empty.

However, using the the [`emailScore`](#email-score) query on a specific email address will still work fine.

# Authentication

> To authorize, provide a Auth Bearer HTTP header

<pre class="highlight graphql tab-graphql">
{ "Authorization": "Bearer TEST_KEY" }
</pre>

```javascript

const key = 'TEST_KEY'; // replace with your api key

const response = await fetch('https://api.subscriptionscore.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
    'Authorization': `Bearer ${key}`
  },
  body: JSON.stringify(query)
})
const data = await response.json();
```

> Make sure to replace `TEST_KEY` with your API key.

Subscription Score uses licence keys to allow you access to the API. You can register a new API key by sending us an email at [hi@subscriptionscore.com](mailto:hi@subscriptionscore.com).

The API expects for the licence key to be included in all API requests to the server in a header that looks like the following:

`Authorization: Bearer $TEST_KEY`

<aside class="notice">
You must replace <code>$TEST_KEY</code> with your personal API key.
</aside>

# Score Queries

## Domain Score

<pre class="highlight graphql tab-graphql">
query {
  domainScore(domain: "linkedin.com") {
    domain
    rank
    unsubscribeAvg
    frequencyPerWeek
    unsubscribeDifficulty
    contentQuality
    inboxPlacement {
      inbox
      spam
    }
  }
</pre>

```javascript
const query = `
query Search($domain: String!) {
  domainScore(domain: $domain) {
    domain
    rank
    unsubscribeAvg
    frequencyPerWeek
    unsubscribeDifficulty
    contentQuality
    inboxPlacement {
      inbox
      spam
    }
  }
}
`;

const variables = {
  domain: "linkedin.com"
};

const response = await fetch('https://api.subscriptionscore.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
    'Authorization': `Bearer ${key}`
  },
  body: JSON.stringify({
    query,
    variables
  })
});
const data = await response.json();
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "domainScore": {
      "domain": "linkedin.com",
      "rank": "B",
      "unsubscribeAvg": 1,
      "frequencyPerWeek": 0.691,
      "unsubscribeDifficulty": 1,
      "contentQuality": 2,
      "inboxPlacement": {
        "inbox": 0.874,
        "spam": 0.004
      }
    }
  }
}
```

This endpoint retrieves the score data for a specific domain. A domain score is an average of all the mailing lists that are sent from that domain.

### Query

`domainScore(domain: $domain): Score`

### Query Parameters

| Parameter | Type   | Example        | Description                                                                               |
| --------- | ------ | -------------- | ----------------------------------------------------------------------------------------- |
| domain    | String | "linkedin.com" | The domain that you're requesting the score data for. Do not include the protocol or path |

### Return type

If the domain is not known to send any mailing lists then the response will be empty. Otherwise it will return a single <a href="#score">Score</a> object.

<aside class="success">
Remember to add your Authentication key to the request!
</aside>

## Email Address Score

<pre class="highlight graphql tab-graphql">
query {
  emailScore(email: "noreply@youtube.com", cleartext: true) {
    email
    rank
    unsubscribeAvg
    frequencyPerWeek
    unsubscribeDifficulty
    contentQuality
    inboxPlacement {
      inbox
      spam
    }
  }
}
</pre>

```javascript
const query = `
query Search($email: String!, $cleartext: Boolean) {
  emailScore(email: $email, cleartext: $cleartext) {
    email
    rank
    unsubscribeAvg
    frequencyPerWeek
    unsubscribeDifficulty
    contentQuality
    inboxPlacement {
      inbox
      spam
    }
  }
}
`;

// hash the email with SHA-1
// (example from https://developer.mozilla.org/en-US/docs/Web/API/SubtleCrypto/digest)
const hashedEmail = window.crypto.subtle
  .digest('SHA-1', new TextEncoder().encode('noreply@youtube.com'))
  .then(hashBuffer => {
    return Array.from(new Uint8Array(hashBuffer))
      .map(b => b.toString(16).padStart(2, '0'))
      .join('');
  });

const response = await fetch('https://api.subscriptionscore.com/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
    'Authorization': `Bearer ${key}`
  },
  body: JSON.stringify({
    query,
    variables: {
      email: hashedEmail
    }
  })
});
const data = await response.json();
```

> The above command returns JSON structured like this:

```json
{
  "data": {
    "emailScore": {
      "email": "0f0f1abfc1a536c773a9b88c9b5303ba9171d521",
      "rank": "F",
      "unsubscribeAvg": 0,
      "frequencyPerWeek": 10.25,
      "unsubscribeDifficulty": 2,
      "contentQuality": 1,
      "inboxPlacement": {
        "inbox": 0.672,
        "spam": 0.148
      }
    }
  }
}
```

This endpoint retrieves the score of a specific email address.

### Query

`emailScore(email: $email, cleartext: $boolean): Score`

### Query Parameters

| Parameter | Type   | Default | Description                                                                                                                                                 |
| --------- | ------ | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| email     | String |         | The email address that the mailing list is sent from                                                                                                        |
| cleartext | Bolean | `false` | True if the email address is being sent in the clear (by default email addresses are hashed during transfer, see [email section for more details](#emails)) |

### Return type

If the email address is not known to send any mailing lists, the response will be empty.
Otherwise it returns a single <a href="#score">Score</a> object.

<aside class="notice">
Note; the returned email address is hashed with SHA-1, see <a href="#emails">email section</a> for more details.
</aside>

# Types

## Score

The Score type represents all the data we have on a domain or email address that contributes to an overall rank.

<pre class="highlight graphql tab-graphql">
type Score {
  rank: String
  unsubscribeAvg: Int
  unsubscribeRate: Float
  frequencyPerWeek: Float
  unsubscribeDifficulty: Int
  contentQuality: Int
  inboxPlacement: Placement

  email: String
  domain: String
}
</pre>

| Parameter             | Type                    | Description                                                                                                                                                                                                                         |
| --------------------- | ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| rank                  | String                  | The overall rank of a domain or mailing list, ranks are percentile based from F to A+                                                                                                                                               |
| unsubscribeAvg        | Int                     | The relative average unsubscribe rate. 1 = Above average, 0 = Average, -1 = Below Average                                                                                                                                           |
| unsubscribeRate       | Float                   | The absolute unsubscribe rate as a percentage. Bear in mind that absolute rates will be inflated, our average unsub rate is around 30%                                                                                              |
| frequencyPerWeek      | Float                   | The number of emails this domain or mailing list sends per week. The domain frequency is an average of all their mailing list frequencies                                                                                           |
| unsubscribeDifficulty | Int                     | The relative difficuly to unsubscribe from this mailing list. 1 = Easy, 2 = Tedious, 3 = Difficult. Easy is defined as a single click unsubscribe                                                                                   |
| contentQuality        | Int                     | The relative quality of the mailing list(s). Quality is determined by the user intent after receiving the mailing list emails, and is not indicative of actual email content. [Email us](mailto:hi@leavemealone.app) for more info. |
| inboxPlacement        | [Placement](#placement) | The percentage of times the mailing list(s) emails ended up in the spam or inbox mailboxes                                                                                                                                          |
| domain                | String                  | (if [domainScore query](#domain-score)) The normalized domain present in the request. This may be different to the request if a domain is aliased, or if a subdomain was provided                                                   |
| email                 | String                  | (if [emailScore query](#email-address-score)) The hashed email address that was provided in the query params                                                                                                                        |

## Placement

<pre class="highlight graphql tab-graphql">
type Placement {
  spam: Float
  inbox: Float
}
</pre>

The Placement type describes the percentage of times the mail is seen within the spam or inbox folders. Unlike ESP Inbox Placement data, this includes when a user moves the email to spam, not just where the email lands on reciept.

| Parameter | Type  | Description                                                      |
| --------- | ----- | ---------------------------------------------------------------- |
| inbox     | Float | Percentage of times the mailing list is seen in the inbox        |
| spam      | Float | Percentage of times the mailing list is seen in the spam mailbox |
