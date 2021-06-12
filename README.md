# 3Commas Google App Script API Handler

This handler is designed to GET / POST data by interacting with the 3Commas api in conjunction with Google app Script.

You can view the public library [on Google App Script here](https://script.google.com/d/1xKmU5eYo0uYPOFJjZLFLe0-k4YtPSu0UKdUFrKKfxzVB5Be1NJ1xRky9/edit?usp=sharing).

You can choose to either add this as a library to your Google App Script project, or copy the code directly into your project. There are potential speed implications by choosing a library. However, in testing we have not noticed a significant impact by going with a library.

# Privacy

Privacy is important with your 3Commas' API information. When using this code as a library it's executed locally to your Google App Script environment and your data is never shared outside of your Google Script.

# Adding the Library

- ** Library ID:** - `1xKmU5eYo0uYPOFJjZLFLe0-k4YtPSu0UKdUFrKKfxzVB5Be1NJ1xRky9`
- [Google's documentation on adding libraries](https://developers.google.com/apps-script/guides/libraries)


## New GAS Editor

1. Open the script you with to include this library in.
2. Click the `+` next to Libraries.
3. Paste the Library ID that is above and click `Lookup`
4. Select the version you want.

## Old GAS Editor

1. On the Menu bar click `Resources` > `Libraries` 
2. Under `Add a Library` paste the Library ID and click `Add`
3. Select the version you want, or leave blank to always use the latest.
4. Click `Save`

# Using the API handler

## Adding your API keys to your script
Before performing any API calls you will need to have your keys property added to the script. The easiest way is to create a function that returns these API keys in the object format needed.

All keys passed to the API handler should be in the below format:

```json
{
    'apikey': '',
    'apisecret': ''
}
```

Keep in mind that storing API keys direction on your script is not a best practice. Google offers the ability to store these in user / script / document properties. You can refer to [this documentation](https://developers.google.com/apps-script/guides/properties) to set that up. See the below example for this in action.

** All the below examples assume you are using the function below to return your API keys**

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
}
```

### 2. Example of a single call.

This is an example of a single API GET call. Note that `limit` and `loop` are optional parameters and do not need to be passed in.

```javascript
async function test_get_call(){
  let endpoint = "/ver1/deals"
  let params = "&scope=active"

  let deals = await api_3c.GET(api_keys_3c(), endpoint, params)
}
```

## POST

The `POST` endpoint supports posts that only are for data or actually pushing data to 3Commas. The structure is relatively the same as the GET calls. 

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

