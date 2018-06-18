# IBM Watson Salesforce SDK

[![Build Status](https://travis-ci.org/watson-developer-cloud/salesforce-sdk.svg?branch=master)](https://travis-ci.org/watson-developer-cloud/salesforce-sdk/)
[![Slack](http://wdc-slack-inviter.mybluemix.net/badge.svg)](https://wdc-slack-inviter.mybluemix.net)



https://githubsfdeploy.herokuapp.com/app/githubdeploy/sfdcjames/salesforce-sdk

The IBM Watson Salesforce SDK uses the [Watson API](http://www.ibm.com/watson/developercloud/) services to help you solve complex problems using Apex in your Salesforce environment.

## Prerequisites

Using the Watson services requires creating service instances and getting credentials in IBM Cloud, which you can do [here](https://console.bluemix.net/registration/?target=/catalog/%3fcategory=watson&cm_mmc=OSocial_Wechat-_-Watson+Core_Watson+Core+-+Platform-_-WW_WW-_-salesforce&cm_mmca1=000000OF&cm_mmca2=10000409&). This means you will need to create an IBM Cloud account if you do not have one already.

Once you create an account, you can get credentials for the services you want by following these steps:

1. Log in to [IBM Cloud](https://console.bluemix.net/registration/?target=/catalog/%3fcategory=watson&cm_mmc=OSocial_Wechat-_-Watson+Core_Watson+Core+-+Platform-_-WW_WW-_-salesforce&cm_mmca1=000000OF&cm_mmca2=10000409&)

1. Create an instance of the desired service:
    1. In the left hamburger menu, select **Watson**.
    1. Click on **Browse Services**, located under **Watson Services** in the left menu.
    1. Select the services you'd like to use.

1. Copy the credentials at the bottom of the page for the selected services.

You'll also need a Salesforce account to run your Apex code. To get one, you can visit [this link](https://developer.salesforce.com/signup).

## Installation

There are three supported installation methods: automatically using Salesforce DX, manually using Salesforce DX, and manually using the Ant build tool. By default, all classes in the SDK will be deployed to your Salesforce org, but using Ant allows you to specify which services you want to deploy to save space. Note though that this is **only** supported with the Ant build tool method.

### Salesforce DX Automatic deployment

You can automatically deploy the SDK to a new scratch environment using the _Deploy to SFDX_ button.

_Note: To use the automatic deployment button, you must log in using a **Dev Hub** org. If you do not have one, there should be a link to get one after you click the button._

[![Deploy](https://deploy-to-sfdx.com/dist/assets/images/DeployToSFDX.svg)](https://deploy-to-sfdx.com/)

### Salesforce DX Manual deployment

You can also manually deploy the SDK code using the Salesforce DX CLI, which you can install [here](https://developer.salesforce.com/tools/sfdxcli).

The first step to manual deployment is cloning the repository from GitHub using the following command:

```bash
git clone https://github.com/watson-developer-cloud/salesforce-sdk
```

Be sure to navigate to the cloned directory before continuing.

To deploy to a scratch org, do the following:

1. Create a new scratch environment (optional if you don't want to re-use an existing one):

    ```bash
    sfdx force:org:create -a watson-sdk -s -f config/project-scratch-def.json
    ```

1. Push the source to the scratch environment:

    ```bash
    sfdx force:source:push
    ```

If you want to use the Watson SDK within a non-scratch environment you can follow these steps:

1. Authenticate the Salesforce DX CLI to the target environment:

    ```bash
    sfdx force:auth:web:login --setdefaultusername
    ```
    In the browser window that opens, sign in to your org with your credentials. More information [here](https://developer.salesforce.com/docs/atlas.en-us.sfdx_dev.meta/sfdx_dev/sfdx_dev_auth_web_flow.htm)

1. Convert the source code:

    ```bash
    sfdx force:source:convert -d mdapioutput/
    ```

1. Deploy the source code:

    ```bash
    sfdx force:mdapi:deploy -d mdapioutput/ -w 100
    ```

## Installation using the Ant Build Tool

Finally, you can also install or update the SDK using the Ant Build Tool. This method also allows you to specify which services you'd like to deploy.

1. Clone this repository from GitHub using the following command:

    ```bash
    git clone https://github.com/watson-developer-cloud/salesforce-sdk
    ```

1. Edit `install/build.properties` to insert your Salesforce username and password.  Since you will be using the API to access Salesforce, remember to [append your Security Token](http://www.salesforce.com/us/developer/docs/api/Content/sforce_api_concepts_security.htm#topic-title_login_token) to your password.

1. Open your command line to the `install` folder, then deploy using Ant:

    ```bash
    ant deployWatson
    ```

    By default, this will deploy all of the classes. To specify a service, use the `-Dservice` flag. Here is an example of just deploying the Watson Assistant service:

    ```bash
    ant deployWatson -Dservice=assistant
    ```

    Valid service names are all services listed [here](https://console.bluemix.net/catalog/?category=watson) written as one word (e.g. Visual Recognition becomes visualrecognition). The parameter is case-insensitive. To deploy multiple services, just run the command again with the next desired service flag.

## Authentication
To access your Watson services through Apex, you'll need to authenticate with your service credentials. There are two ways to do this: [using named credentials](#using-named-credentials) or [specifying credentials in the Apex code](#specifying-credentials-in-the-apex-code)

### Using `Named Credentials`

`Named Credentials` are the preferred way of authentication, since they allow you to keep sensitive information out of your code. However, they only work when using a **username and password**, so if you'd like to authenticate with an API key or IAM, you'll need to [set that up in your Apex code](#specifying-credentials-in-the-apex-code).

When creating a service instance like with `new Discovery()`, each service loads the credentials from `Named Credentials`. The SDK will use the service name and API version to build the `Named Credentials` name.

For example

```java
IBMDiscoveryV1 discovery = new IBMDiscoveryV1('2017-11-07');
```

Will look for the `watson_discovery_v1` named credentials while:

```java
IBMAssistantV1 assistant = new IBMAssistantV1('2018-02-16');
```

Will look for `watson_assistant_v1`.

In order to create **Named credentials**:

1. Go to _Setup_ by clicking on the gear icon on the top right of the Salesforce dashboard
1. Enter _Named Credentials_ in the quick find box and select the highlighted entry
1. Click on _New Named Credential_
1. Enter the following values:
    * Label: _A unique label that identifies your named credentials_
    * Name: `watson_<service_name_snake_case>_<api_version>`, e.g: `watson_assistant_v1`
    * URL: `<SERVICE_URL>`, e.g: `https://gateway.watsonplatform.net/assistant/api`
    * Identity Type: **Named Principial**
    * Authentication Protocol: **Password Authentication**
    * Username: `<USERNAME>`
    * Password: `<PASSWORD>`
1. Click on Save.

### Specifying credentials in the Apex code

Setting credentials in the code is always an option, and in fact, it's the only option if you're authenticating with an API key (in the case of the Visual Recognition service) or with IAM.

You can always set these values directly in the constructor or with a method call after instantiating your service.

_Note: You must set the service endpoint manually when setting your credentials this way. Otherwise, the SDK will default to searching for `Named Credentials` that you won't have set up._

#### Username and Password

```java
// in the constructor
IBMDiscoveryV1 discovery = new IBMDiscoveryV1('2017-11-07', 'USERNAME', 'PASSWORD');
discovery.setEndPoint('URL');
```

```java
// after instantiation
IBMDiscoveryV1 discovery = new IBMDiscoveryV1('2017-11-07');
discovery.setEndPoint('URL');
discovery.setUsernameAndPassword('USERNAME', 'PASSWORD');
```

#### API Key

```java
// in the constructor
IBMVisualRecognitionV3 visualRecognition = new IBMVisualRecognitionV3('2016-05-20', 'API_KEY');
visualRecognition.setEndPoint('URL');
```

```java
// after instantiation
IBMVisualRecognitionV3 visualRecognition = new IBMVisualRecognitionV3('2016-05-20');
visualRecognition.setEndPoint('URL');
visualRecognition.setApiKey('API_KEY');
```

#### Using IAM

When authenticating with IAM, you have the option of passing in:
  - the IAM API key and, optionally, the IAM service URL
  - an IAM access token

**Be aware that passing in an access token means that you're assuming responsibility for maintaining that token's lifecycle.** If you instead pass in an IAM API key, the SDK will manage it for you.

```java
// in the constructor, letting the SDK manage the IAM token
IBMWatsonIAMOptions options = new IBMWatsonIAMOptions.Builder()
  .apiKey('IAM_API_KEY')
  .url('IAM_URL') // optional - the default value is https://iam.bluemix.net/identity/token
  .build();
IBMDiscoveryV1 service = new IBMDiscoveryV1('2017-11-07', options);
service.setEndPoint('SERVICE_URL');
```

```java
// after instantiation, letting the SDK manage the IAM token
IBMDiscoveryV1 service = new IBMDiscoveryV1('2017-11-07');
IBMWatsonIAMOptions options = new IBMWatsonIAMOptions.Builder()
  .apiKey('IAM_API_KEY')
  .build();
service.setIamCredentials(options);
service.setEndPoint('SERVICE_URL');
```

```java
// in the constructor, assuming control of managing IAM token
IBMWatsonIAMOptions options = new IBMWatsonIAMOptions.Builder()
  .accessToken('ACCESS_TOKEN')
  .build();
IBMDiscoveryV1 service = new IBMDiscoveryV1('2017-11-07', options);
service.setEndPoint('SERVICE_URL');
```

```java
// after instantiation, assuming control of managing IAM token
IBMDiscoveryV1 service = new IBMDiscoveryV1('2017-11-07');
IBMWatsonIAMOptions options = new IBMWatsonIAMOptions.Builder()
  .accessToken('ACCESS_TOKEN')
  .build();
service.setIamCredentials(options);
service.setEndPoint('SERVICE_URL');
```

If at any time you would like to let the SDK take over managing your IAM token, simply override your stored IAM credentials with an IAM API key by calling the `setIamCredentials()` method again.

### Setting Remote Site Settings

The final piece of setup to access Watson services from your Salesforce environment is setting your remote site settings. To do so:

1. Go to _Setup_ by clicking on the gear icon on the top right of the Salesforce dashboard
1. Enter _Remote Site Settings_ in the quick find box and select the highlighted entry
1. Click _New Remote Site_
1. Add whatever name you desire, with the following URL: `https://gateway.watsonplatform.net/`
1. Click _Save_

If you're authenticating with IAM, you'll also need to add your IAM URL in your remote site settings. The default URL is `https://iam.ng.bluemix.net`.

## Examples

Getting started using a service is very simple! All services follow the same pattern of service instantiation, option building, and requesting. To get an idea, below is an example of using the Discovery service to get a list of your current environments:

```java
// Will load credentials from the `watson_discovery_v1` named credential
IBMDiscoveryV1 discovery = new IBMDiscoveryV1('2017-11-07');

// configuring options for listing environments
IBMDiscoveryV1Models.ListEnvironmentsOptions options = new
  IBMDiscoveryV1Models.ListEnvironmentsOptionsBuilder()
  .build();

// making request
IBMDiscoveryV1Models.ListEnvironmentsResponse environmentList = discovery.listEnvironments(options);
System.debug(environmentList);
```

Similarly, here is an example of creating an intent in the Watson Assistant service:

```java
// Will load credentials from the `watson_assistant_v1` named credential
IBMAssistantV1 assistant = new IBMAssistantV1('2018-02-16');

// configuring options for creating intent
IBMAssistantV1Models.CreateIntentOptions options = new
  IBMAssistantV1Models.CreateIntentOptionsBuilder()
  .workspaceId('<workspace_id>')
  .intent('MyIntent')
  .description('This is an example of creating an intent!')
  .build();

// making request
IBMAssistantV1Models.Intent intent = assistant.createIntent(options);
System.debug(intent);
```

The manner of instantiating and using services should be consistent no matter which you decide to use, which should make it easy to explore the many capabilities Watson services have to offer.

## Request and Response Headers

The SDK supports sending custom headers with any request as well as parsing headers that are returned by the service.

To send request headers, simply add them as a property when building up your `Options` model. Here's an example in the Discovery service:

```java
IBMDiscoveryV1Models.QueryOptions options = new
  IBMDiscoveryV1Models.QueryOptionsBuilder()
  .environmentId('<environment_id>')
  .collectionId('<collection_id>')
  .naturalLanguageQuery('Articles about the Boston Celtics')
  .addHeader('Custon-Header', 'custom_value') // custom header added here
  .build();
```

To get headers returned by the service, call the `getHeaders()` method on a response model. This is what it looks like to get the headers returned after making the above call:

```java
IBMDiscoveryV1Models.QueryResponse response = discovery.query(options);
Map<String, String> responseHeaders = response.getHeaders();
```

## Using the SDK with Lightning

The Watson Salesforce SDK models are Lightning-ready, meaning that you can access model properties through Javascript for your Lightning apps. Everything should work as expected, but it's important to note that there are two ways to go about dealing with dynamic models through Javascript. These models are ones which may have properties unknown until runtime and which extend `IBMWatsonDynamicModel`.

### Using the `additionalProperties` Object

Dynamic models have an extra "AuraEnabled" property called `additionalProperties`, which is a map that holds all of the dynamic properties returned with the model. If you're dealing with a dynamic model in your Javascript code and want to access any dynamic properties, you can do the following:

```javascript
action.setCallback(this, function(response) {
  var resp = response.getReturnValue(); // resp is a dynamic model
  console.log(resp.additionalProperties["enriched_text"]["concepts"][0]["text"]);
});
$A.enqueueAction(action);
```

Any properties within the `additionalProperties` object can be accessed exactly like a generic Javascript object.

### Using the Model String Representation

If going through the `additionalProperties` object is undesired for any reason, you can still access the dynamic properties as top-level properties with a small workaround. All models in the SDK override the `toString()` method to output the models as pretty-printed JSON with the additional properties brought up to the top level. Therefore, if you tweak your server-side controller to return the string representation instead of the model object, as follows:

```apex
public class ServerSideController {
  @AuraEnabled
  public static String query(String environmentId, String collectionId) {
    IBMDiscoveryV1 discovery = new IBMDiscoveryV1('2017-11-07');
    IBMDiscoveryV1Models.QueryOptions options = new IBMDiscoveryV1Models.QueryOptionsBuilder(environmentId, collectionId)
      .naturalLanguageQuery('example query')
      .build();
    IBMDiscoveryV1Models.QueryResponse response = discovery.query(options);

    // IMPORTANT: return the string instead of the model
    return response.toString();
  }
}
```

you can use `JSON.parse()` to access the whole object as a generic JSON object. Here is an example of accessing the same property in Javascript using this aternate method:

```javascript
action.setCallback(this, function(response) {
  var resp = response.getReturnValue(); // resp is now the string representation of our dynamic model
  console.log(JSON.parse(resp)["enriched_text"]["concepts"][0]["text"]);
});
$A.enqueueAction(action);
```

Both methods shown above will print out the same information.

## WebSocket Support
Both the Text to Speech and Speech to Text services have API endpoints which support the use of WebSockets. However, the use of WebSockets are not supported natively by Apex. If you would like to take advantage of this functionality, the recommendation would be to leverage the [Watson Speech Javascript SDK](https://github.com/watson-developer-cloud/speech-javascript-sdk).

The above library adds minimal overhead and will allow for the use of WebSockets. Also, being Javascript, this SDK can integrate smoothly with Lightning apps and be used alongside the Salesforce SDK.

## Functional Tests

The `force-app/main/test` folder contains the example calls for each service. These examples are used for functional testing of services. Developers can use them for reference and testing the installed SDK.

## Contributing

If you're interested in helping to make this project better, see [Contributing.md](.github/Contributing.md).

## License

This library is licensed under the MIT license. Full license text is
available in [LICENSE](LICENSE).
