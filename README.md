# 3Commas Google App Script API Handler

This handler is designed to GET / POST data by interacting with the 3Commas api in conjunction with Google app Script.

You can view the public library [on Google App Script here](https://script.google.com/d/1xKmU5eYo0uYPOFJjZLFLe0-k4YtPSu0UKdUFrKKfxzVB5Be1NJ1xRky9/edit?usp=sharing).

You can choose to either add this as a library to your Google App Script project, or copy the code directly into your project. There are potential speed implications by choosing a library. However, in testing we have not noticed a significant impact by going with a library.

For more details on interacting with the 3Commas API you can use their [GitHub Documentation here](https://github.com/3commas-io/3commas-official-api-docs).

# Privacy

Privacy is important with your 3Commas' API information. When using this code as a library it's executed locally to your Google App Script environment and your data is never shared outside of your Google Script.

# Tips on using this library

- Google sets default quotas on each Google Acount. This means you have a potential 5 minute execution time. Some data calls to 3Commas can become quite large. You may need to break these calls into multiple functions that get triggered. You can use Google's [documentation on Quotas](https://developers.google.com/apps-script/guides/services/quotas) for more info.
- Basic error handling is implemented within the Library, however, the goal is to pass the response from the API back. Be sure to implement basic error handling on the responses.
- Limit larger calls to by using date or limit params.
- For `POST`, `PUT`, and `DELETE` endpoints, the parameters may be sent as a `param` string or within the payload. You may mix parameters between both the `param` string and or `payload` if you wish to do so. If a parameter sent in both the `param` string and the `payload`, the `param` string parameter will be used.


# Adding the Library

- **Library ID:** - `1xKmU5eYo0uYPOFJjZLFLe0-k4YtPSu0UKdUFrKKfxzVB5Be1NJ1xRky9`
- [Google's documentation on adding libraries](https://developers.google.com/apps-script/guides/libraries)


## New GAS editor

1. Open the script you with to include this library in.
2. Click the `+` next to Libraries.
3. Paste the Library ID that is above and click `Lookup`
4. Select the version you want.

## Old GAS editor

1. On the Menu bar click `Resources` > `Libraries` 
2. Under `Add a Library` paste the Library ID and click `Add`
3. Select the version you want, or leave blank to always use the latest.
4. Click `Save`

# Using the API handler

## Adding your API keys to your script
Before performing any API calls you will need to have your keys property added to the script. The easiest way is to create a function that returns these API keys in the object format needed.

All keys passed to the API handler should be in the below format:

```javascript
{
    'apikey': '',
    'apisecret': ''
}
```

Keep in mind that storing API keys direction on your script is not a best practice. Google offers the ability to store these in user / script / document properties. You can refer to [Google's documentation](https://developers.google.com/apps-script/guides/properties) to set that up. See the below example for this in action.

**All the below examples assume you are using the function below to return your API keys**

**Example function to return your keys**
```javascript
function api_keys_3c() {
  return {
    'apikey': myApiKeyFunction(),
    'apisecret': myApiSecretFunction()
  }
}
```


## GET

The API library supports two types of GET calls. 
1. A looped GET call that will return every item until the loop's limit is reached.
2. A loop call that returns a single API call with with a limit of 25 (default 3Commas limit.)

### 1. Example of a looped call

This looped call passes in the limit of 15000 and sets the loop to true. The result output will run until either it fetches all the data, or the limit is reached. Be reasonable with your limits based on the level of data you are fetching. Google has a [hard limit of 5 minutes on execution time](https://developers.google.com/apps-script/guides/services/quotas) for all `@gmail.com` accounts. 

```javascript
async function test_get_call(){
  let endpoint = "/ver1/deals"
  let params = "&scope=active"
  let limit = 15000
  let loop = true

  let deals = await api_3c.GET(api_keys_3c(), endpoint, params, loop, limit)

  if(deals.length > 0){
    return deals.data
  }
}
```

### 2. Example of a single call.

This is an example of a single API GET call. Note that `limit` and `loop` are optional parameters and do not need to be passed in.

```javascript
async function test_get_call(){
  let endpoint = "/ver1/deals"
  let params = "&scope=active"

  let deals = await api_3c.GET(api_keys_3c(), endpoint, params)

  if(deals.status === 200){
      return deals.data
  }
}
```


## POST

The `POST` method supports updating and pushing new data into 3Commas based on the endpoints you are using. The structure is similar to the GET calls. Note that you can send parameters within the params or the payload.

The `POST` and `PATCH` endpoints do not supprt JSON in the payload. The payload is simply what you'd pass to the params.

A successful `POST` response will be a 201 status code.

### POST API example

This example implements some basic filtering using data that is returned from the library.



```javascript
async function example_post_call() {

  let endpoint = `/ver1/accounts/111111111/load_balances`
  let params = ""
  let apiKeys = api_keys_3c()
  let payload = null
  let postCall = await api_3c.POST(apiKeys, endpoint, params, payload)

  if (postCall.status !== 201) {
    console.log(postCall.error)
  } else {
    console.log(postCall.data)
  }
}

```

## PATCH

The `PATCH` method should be used when updating existing information already within 3Commas. This method updates the specific fields passed and returns the data object that you updated. Note that you can send parameters within the params or the payload.

**The `POST` and `PATCH` endpoints do not supprt JSON in the payload. The payload is simply what you'd pass to the params.**

### Patch API Example

In the below example we are using both a parameter within the params url string and in the payload of the `PATCH` request. This will use both parameters to update the deal. For more details on this reference the Tips to Using this Library above.

A successful `PATCH` response will be a 200 status code.

**Function Example**
```javascript
async function patch_test_call(){
  let endpoint = "/ver1/deals/569900008/update_deal"
  let params = "&max_safety_orders=30"
  let payload = "take_profit=1"
  let apiKeys = api_keys_3c()

  let deals = await api_3c.PATCH(apiKeys, endpoint, params, payload)
  if(deals.status === 200){
    return deals.data
  }
}
```

**Example Response**

```javascript
{ data:  
  { id: 569900008,
     type: 'Deal',
     ...
     max_safety_orders: 30,
     deal_has_error: false,
     from_currency_id: 0,
     ...
     take_profit: '1',
     ...
     strategy: 'long',
     bot_events: 
      [ ... ] },
  status: 200,
  headers: 
   {}
```

## DELETE

The delete method is not widely supported with 3Commas and the 3Commas documentation mixes a traditional `DELETE` request with a `POST` request to a `/delete` endpoint. Be sure you're intending to use the method `DELETE`.

Note that when you first delete an item you'll be returned a `true` value in the data object. Any future `DELETE` attempts at that item will continue to return `true`.

**Function Example**

In the `DELETE` example below we make a simple delete to the `/grid_bots` endpoint and delete a grid bot. The 3C API returns a simple true / false if the delete was successful and a `404` error if the delete was unsuccessful in finding the data.

```javascript
async function test_delete_call(){
  let endpoint = "/ver1/grid_bots/{id}"
  let apiKeys = api_keys_3c()

  let deals = await api_3c.DELETE(apiKeys, endpoint)
  if(deals.status === 200 || deals.data == true){
    return true
  }
}
```

**Example Delete Response**

```javascript
{ 
  data: 'true',
  status: 200,
  headers: 
   {}
}
```


# API Call Responses

The response object will contain a data array, status, headers, and a potential error object. If the data response is successful this data array will be a parsed JSON object.

###  `GET` non-looped / `POST` / `PATCH`

Note here that a looped GET response will look different.

#### Sucessful response example:

```javascript
{ 
  data: [
    {},
    {}
  ],
  status: 200,
  headers: {} 
}
```

#### Failed response example:

A failed response will contain an additional error key to parse the message on. The data will most likely not be parsed for this object as 3Commas returns invalid JSON for a failed call.

**Note that you can get a 200 response even though the call is invalid.**

```javascript
{ data: '',
  status: 200,
  headers: {},
  error: 'Invalid JSON object was returned. Check to make sure your endpoint is correct.' }
```

###  `GET` Looped Response

A `GET` looped response will include the data object containing all the data from the looped response. No other values are added for a successful call.

**Sucessful response example:**
```javascript
{
  'data' : [
    {},
    {}
  ]
}
```

**Failed response example:**

A failed `GET` looped call will contain an error message detailing the issue. Usually for a loop you should also check the logs as additional call logging happens for each loop.

```javascript
{
  'data' : [
    {},
    {}
  ],
  'error' : 'error message here'
}
```
