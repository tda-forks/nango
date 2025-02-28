import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Connections API

The connections API is the recommended way to retrieve access tokens and Connection objects in programming languages where Nango does not yet offer a backend SDK.

## Base URL and authentication

Where you access the Connections API and how you authenticate depends on your Nango deployment.

<Tabs groupId="deployment" queryString>
<TabItem value="cloud" label="Nango Cloud">

You will need the `Secret Key` from your [Dashboard](https://app.nango.dev/).

API base URL: `https://api.nango.dev`

Authorization header: `Bearer <SECRET-KEY>`

  </TabItem>
  <TabItem value="localhost" label="Localhost">

API base URL: `http://localhost:3003`

No Authorization header is required.

  </TabItem>
  <TabItem value="self-hosted" label="Self-hosted">

API base URL: `https://<NANGO-HOST-AND-PORT>`

Authorization header: Only required if you [secured your instance](nango-deploy/oss-instructions.md#securing-your-instance). If so, Basic Auth as described in the docs there.

  </TabItem>
</Tabs>

## Retrieving access tokens and connection data {#connectionDetails}

Request type: `GET`  
Endpoint: `/connection/<CONNECTION-ID>?provider_config_key=<PROVIDER-CONFIG-KEY>`

If you are not familiar with the Provider Config Key and Connection Id parameters please take a look at the [Core Concepts](reference/core-concepts.md) page.

This is the recommended way to retrieve an access token to make an API call: Retrieve the current access token from `credentials.access_token` in the response.

This retrieves the full Connection object from Nango, which looks like this:

```js
{
id: 18393,                                  // Nango internal connection id
created_at: '2023-03-08T09:43:03.725Z',     // Creation timestamp
updated_at: '2023-03-08T09:43:03.725Z',     // Last updated timestamp (e.g. last token refresh)
provider_config_key: 'github',              // <PROVIDER-CONFIG-KEY>
connection_id: '1',                         // <CONNECTION-ID>
credentials: {
    type: 'OAUTH2',                         // OAUTH2 or OAUTH1
    access_token: 'gho_tsXLG73f....',       // The current access token (refreshed if needed)
    refresh_token: 'gho_fjofu84u9....',     // Refresh token (if available, otherwise missing)
    expires_at: '2024-03-08T09:43:03.725Z', // Expiration date of access token (only if refresh token is present, otherwise missing)
    raw: {                                  // Raw token response from the OAuth provider: Contents vary!
        access_token: 'gho_tsXLG73f....',
        token_type: 'bearer',
        scope: 'public_repo,user'
    }
},
connection_config: {},                      // Additional configuration, see Frontend SDK reference
account_id: 0,                              // ID of your Nango account (Nango Cloud only)
metadata: {}                                // Structured metadata retrieved by Nango
}
```

The metadata field contains [structured metadata](reference/core-concepts.md#metadata), which Nango obtained from the OAuth flow. This varies by provider and is documented on the provider's Nango documentation page.

:::tip Keep your access tokens fresh and don't cache them!
When you call this method the Nango server will check whether the access token needs to be refreshed, and, if needed, refresh it before returning it to you. This typically changes the access token.

Because of this it is important that you always use this method to get the latest access token from Nango just before making an API request. If you cache the access token on your end you risk working with an expired or revoked access token and your API call will fail!

We take great care to make sure that this call to get an access token is blazingly fast, so retrieving it fresh will not slow down your API requests.
:::

## Getting a list of all Connections

Request type: `GET`  
Endpoint: `/connection`

Returns a list of minimalistic Connection objects. This can be helpful if you need to check whether a user has setup a specific integration. Note that the list does not contain any access credentials and fetching it also does not refresh the access tokens of any Connections.

The return value looks like this:

```js
{
    connections: [
        {
            connection_id: '<CONNECTION-ID-1>',
            provider: '<PROVIDER-CONFIG-KEY-1>',
            created: '2023-03-08T09:43:03.725Z'
        },
        {
            //....
        }
        // Additional objects like the one above
    ];
}
```



## Getting  connections for a specific connection id

Request type: `GET`  
Endpoint: `/connection?connectionId=<connectonId>`

Returns a list of minimalistic Connection objects for only a single connectionId. This can be useful if you want to get all the connections that have been created for only a specific connection id. Note that the list does not contain any access credentials and fetching it also does not refresh the access tokens of any Connections.

The return value looks like this:

```js
{
    connections: [
        {
            connection_id: 'connectonId',
            provider: '<PROVIDER-CONFIG-KEY-1>',
            created: '2023-03-08T09:43:03.725Z'
        },
        {
            //....
        }
        // Additional objects like the one above
    ];
}
```
