# IBM API Connect Advanced Demo

## Sign and verify API with JWT

### Generate JWT

  High level steps:  
  1. Create an API and name it JWT
  2. Select "Secure using ClientID" and "CORS"
  3. Add path /gen
  4. Add operations "GET"
     - Add parameter "iss-claim", type "STRING", located in Header, and put " Enter https://myidp.ibm.com to match " in the description field  
     - Add parameter "aud-claim", type "STRING", located in Header, and put "ClientID1" in the description field  
     - In the Response section, change the description of the pre-supplied 200 status code to 200 OK.  
  5. Edit the assembly
     - Delete the invoke
     - Add "Set Variable", Click + Action field, set name "hs256-key", type "STRING", and enter JWK (JSON Web Key) for the value, eg.  { "alg": "HS256", "kty": "oct", "use": "sig", "k": "o5yErLaE-dbgVpSw65Rq57OA9dHyaF66Q_Et5azPa-XUjbyP0w9iRWhR4kru09aFfQLXeIODIN4uhjElYKXt8n76jt0Pjkd2pqk4t9abRF6tnL19GV4pflfL6uvVKkP4weOh39tqHt4TmkBgF2P-gFhgssZpjwq6l82fz3dUhQ2nkzoLA_CnyDGLZLd7SZ1yv73uzfE2Ot813zmig8KTMEMWVcWSDvy61F06vs_6LURcq_IEEevUiubBxG5S2akNnWigfpbhWYjMI5M22FOCpdcDBt4L7K1-yHt95Siz0QUb0MNlT_X8F76wH7_A37GpKKJGqeaiNWmHkgWdE8QWDQ", "kid": "hs256-key" }
  6. Add "JWT Generate" policy after Set Variable 
     - Enter request.headers.iss-claim in the Issuer Claim field.
     - Enter request.headers.aud-claim in the Audience Claim field.
     - Enter hs256-key in the Sign JWK variable name field.
     - Select HS256 in the Cryptogrpahic Algorithm field.
  7. Add "Gateway Script" to the assembly and add the following code
```
var apim = require('apim');
apim.setvariable('message.body',apim.getvariable('generated.jwt'));
```
  8. Test the API, Enter https://myidp.ibm.com in the iss-claim field.
  
### Validate JWT

  High level steps:  
  1. Create an API and name it "JWTVAL"
  2. Select "Secure using ClientID" and "CORS"
  3. Add path /val
  4. Add operations "GET"
     - Add parameter "Authorization", type "STRING", located in Header, and put "Enter Bearer <jwt>" in the description field  
     - In the Response section, change the description of the pre-supplied 200 status code to 200 OK.  
  5. Edit the assembly
     - Delete the invoke
     - Add "Set Variable", Click + Action field, set name "hs256-key", type "STRING", and enter JWK (JSON Web Key) for the value, eg.  { "alg": "HS256", "kty": "oct", "use": "sig", "k": "o5yErLaE-dbgVpSw65Rq57OA9dHyaF66Q_Et5azPa-XUjbyP0w9iRWhR4kru09aFfQLXeIODIN4uhjElYKXt8n76jt0Pjkd2pqk4t9abRF6tnL19GV4pflfL6uvVKkP4weOh39tqHt4TmkBgF2P-gFhgssZpjwq6l82fz3dUhQ2nkzoLA_CnyDGLZLd7SZ1yv73uzfE2Ot813zmig8KTMEMWVcWSDvy61F06vs_6LURcq_IEEevUiubBxG5S2akNnWigfpbhWYjMI5M22FOCpdcDBt4L7K1-yHt95Siz0QUb0MNlT_X8F76wH7_A37GpKKJGqeaiNWmHkgWdE8QWDQ", "kid": "hs256-key" }
  6. Add "JWT Validate" policy after Set Variable 
     - Enter hs256-key in the Verify Crypto JWK variable name field.
  7. Add "Gateway Script" to the assembly and add the following code
```
var apim = require('apim');
apim.setvariable('message.body',apim.getvariable('decoded.claims'));
```
  8. Add Catch, with the following Gateway Script code
```
var apim = require('apim');
apim.setvariable('message.body',apim.getvariable('jwt-validate.error-message'));  
```
  9. Test the API, Enter https://myidp.ibm.com in the iss-claim field.
  

## GraphQL Proxy
  
  High Level Steps:
  1. Create new API with Title: accounts and GraphQL server URL: https://graphql-test-server.us-east.cf.appdomain.cloud/accounts/graphql, activate ClientID, ClientSecret
     GraphQL Schema editor displays Type and Weight information. The weighting factor is used when calculating the type cost for a request to the GraphQL API. For example, a field that requires extensive CPU or memory use on the server to retrieve its value would be given a higher cost.
  2. Select Query to review the warning details for this type. In addition to the warning details, the Warning window gives an option to fix the warning by apply the limits. 
  3. Publish and test with the following message
```
  {
  accounts(limit: 2) {
    name {
      first
      last
    }
  }
}
```  
  This message will be rejected due to restricted query on creditCard
```
  {
  accounts(limit: 2) {
    name {
      first
      last
    }
    shippingAddress {
      building
      street
    }
  }
  creditCard {
    number
    expirationDate
  }
}
```
  

## API Connect Toolkit
  
  High Level Steps:  
  1. Install toolkit+cli+designer from API Manager and extract it, copy the apic to /usr/local/bin
  2. Download the toolkit credentials from API Manager
  3. apic client-creds:set $(pwd)/credentials.json
  4. Download the designer credentials from API Manager
  5. Create the environment variable APIC_DESIGNER_CREDENTIALS with value the path to designer_credentials.json 
     Ubuntu, edit /etc/environment, and run "source /etc/environment"
  6. Install NodeJS
```
  curl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -
  sudo apt-get install -y nodejscurl -fsSL https://deb.nodesource.com/setup_17.x | sudo -E bash -
  npm install npm --global
```  
  7. Install Loopback
```
  npm i -g @loopback/cli
```  
  8. Create API using Loopback from toolkit
```
 apic lb4 app  
  ? Project name: todo-list  
  ? Project description: A todo list API made with LoopBack 4.  
  ? Project root directory: (todo-list)  
  ? Application class name: (TodoListApplication)  
  ? Select features to enable in the project:  
  ❯◉ Enable eslint: add a linter with pre-configured lint rules  
   ◉ Enable prettier: install prettier to format code conforming to rules  
   ◉ Enable mocha: install mocha to run tests  
   ◉ Enable loopbackBuild: use @loopback/build helpers (e.g. lb-eslint)  
   ◉ Enable vscode: add VSCode config files  
   ◉ Enable docker: include Dockerfile and .dockerignore  
   ◉ Enable repositories: include repository imports and RepositoryMixin  
   ◉ Enable services: include service-proxy imports and ServiceMixin  
   # npm will install dependencies now  
   Application todo-list was created in todo-list.  
```
  9. Create lb4 model
```
  apic lb4 model
  ? Model class name: todo
  ? Please select the model base class: Entity (A persisted model with an ID)
  ? Allow additional (free-form) properties? No
  
  Let's add a property to Todo
  Enter an empty property name when done
  
  ? Enter the property name: id
  ? Property type: number
  ? Is id the ID property? Yes
  ? Is it required?: No
  ? Is id generated automatically? Yes
  ? Default value [leave blank for none]:
  
  Let's add another property to Todo
  Enter an empty property name when done
  
  ? Enter the property name: title
  ? Property type: string
  ? Is it required?: Yes
  ? Default value [leave blank for none]:
  
  Let's add another property to Todo
  Enter an empty property name when done
  
  ? Enter the property name: desc
  ? Property type: string
  ? Is it required?: No
  ? Default value [leave blank for none]:
  
  Let's add another property to Todo
  Enter an empty property name when done
  
  ? Enter the property name: isComplete
  ? Property type: boolean
  ? Is it required?: No
  ? Default value [leave blank for none]:
  
  Let's add another property to Todo
  Enter an empty property name when done
  
  ? Enter the property name:
  
   create src/models/todo.model.ts
   update src/models/index.ts

  Model todo was created in src/models/
```  
  10. Create lb4 datasource
```
  apic lb4 datasource
  ? Datasource name: db
  ? Select the connector for db: In-memory db (supported by StrongLoop)
  ? window.localStorage key to use for persistence (browser only):
  ? Full path to file for persistence (server only): ./data/db.json
  
    create src/datasources/db.datasource.json
    create src/datasources/db.datasource.ts
    update src/datasources/index.ts
  
  Datasource db was created in src/datasources/
```  
  11. Create db.json file for the in memory db content
```
  {
  "ids": {
    "Todo": 5
  },
  "models": {
    "Todo": {
      "1": "{\"title\":\"Take over the galaxy\",\"desc\":\"MWAHAHAHAHAHAHAHAHAHAHAHAHAMWAHAHAHAHAHAHAHAHAHAHAHAHA\",\"id\":1}",
      "2": "{\"title\":\"destroy alderaan\",\"desc\":\"Make sure there are no survivors left!\",\"id\":2}",
      "3": "{\"title\":\"play space invaders\",\"desc\":\"Become the very best!\",\"id\":3}",
      "4": "{\"title\":\"crush rebel scum\",\"desc\":\"Every.Last.One.\",\"id\":4}"
    }
  }
}
```  
  12. Create Repository
```
  apic lb4 repository
  ? Please select the datasource DbDatasource
  ? Select the model(s) you want to generate a repository Todo
  ? Please select the repository base class DefaultCrudRepository (Legacy juggler bridge)
     create src/repositories/todo.repository.ts
     update src/repositories/index.ts

  Repository TodoRepository was created in src/repositories/
```
  13. Create lb4 controller
```
  apic lb4 controller 
  ? Controller class name: todo
  Controller Todo will be created in src/controllers/todo.controller.ts
  
  ? What kind of controller would you like to generate? REST Controller with CRUD functions
  ? What is the name of the model to use with this CRUD repository? Todo
  ? What is the name of your CRUD repository? TodoRepository
  ? What is the name of ID property? id
  ? What is the type of your ID? number
  ? Is the id omitted when creating a new instance? Yes
  ? What is the base HTTP path name of the CRUD operations? /todos
     create src/controllers/todo.controller.ts
     update src/controllers/index.ts
  
  Controller Todo was created in src/controllers/
```
  14. Run the application
```
  npm start
```  
  In case of error
```
  sudo npm cache clear --force
  npm run update-package-locks
  npm ci (again)
  npm run build
```  
## App Connect Integration
  
## OAuth Authentication
   
  High Level Steps:
  1. Add CientID (Header X-IBM-Client-Id) /ClientSecret (X-IBM-Client-Secret) to the security definition of API
  2. Add "Authentication URL" to the list of User Registries on API Manager, Menu Resources --> User Registries --> Create, enter name "AppReg", URL "https://thinkibm-services.mybluemix.net/auth"
  3. Add OAuth Service, Menu Resoures --> OAuth Providers --> Add --> Native OAuth Provider  
     - Name: oauth
     - Title: oauth
     - Gateway Type: Datapower API Gateway
     - Grant types: Resource owner password
     - Client types: Confidential
     - Add or use default Scope 
  4. Enable API Registry on Sandbox Catalog registry setting, select API User Registries and Add App Registry. 
  5. Add the OAuth Service to the Sandbox Catalog, Sandbox --> Settings --> Oauth Providers
  6. Add OAuth to the API Security Definition and Security
     - Type: [[OAuth2]]
     - Flow: [[Resource owner]]
     - Token URL: keep default [[https://\$(catalog.url)/oauth/oauth2/token]]

## API Monetization
  
  High Level Steps:  
  
  0. If you don't have a Stripe account, you can create one here: https://dashboard.stripe.com/register. Refer to your Stripe dashboard to get your test API keys: see https://dashboard.stripe.com/apikeys
  1. Open Resources --> Billing --> Add.
  2. Enter a Title for your billing integration, for example My Stripe Billing
  3. Enter the test Publishable key and Secret key for your Stripe account. 
  4. Add your billing integration resource to a catalog --> Manage --> Billing, and then click Edit.
  5. Enable the Stripe payment method in the Developer Portal, Log in to the Developer Portal as an administrator --> Click Manage --> click Extend --> Enter Stripe into the search filter, select APIC Monetization Stripe Integration, and click Enable 
     Install screen for APIC Monetization Stripe Integration module
  6. The default settings for the APIC Monetization Stripe Integration module are now enabled, and you now need to edit those settings.
Click Configuration > System > IBM API Connect Billing.
   - Change the billing provider module mapping drop-down option to be the newly enabled APIC Monetization Stripe Integration (ibm_stripe_payment_method), and click Save configuration.
   - Module mapping screen for APIC Monetization Stripe Integration module
Click Configuration > System > IBM APIC Stripe Integration, and enter the same Stripe test API credentials that you entered for the My Stripe Billing integration resource 
  7. Download https://www.ibm.com/docs/en/SSMNED_v10/com.ibm.apic.apionprem.doc/findbranch_v6.txt --> rename to YAML --> Create new API --> Create new Product --> Add Billing Integration --> Publish the product
  
## Create API from Scratch
  
  High Level Steps:  
  1. Create new OpenAPI, with key fields  
     - Base Path: /financing
     - Definition: paymentAmount, type: Object
       - Properties: paymentAmount, type: float, Example: 199.99
     - Paths: /calculate (final path /financing/calculate)
       - Operations: GET
         - Parameters: amount (query, float), duration (query, int 32), rate (query, float)
         - Response: 200 --> add schema: paymentAmount
     - Target Services: Add Web Services --> Enter WSDL from https://integrationsuperhero.github.io/techcon2020/APICDevJam/resources/calculate.wsdl  
  2. Go to Assembly, remove Invoke
  3. Drag and drop financing Web Service Operations
  4. Edit /financing input --> + GET Calculate operation, map
  5. Edit /financing output --> + GET /calculate, assign to #definition/paymentAmount, map
  6. Add new API Logistics from https://integrationsuperhero.github.io/techcon2020/APICDevJam/resources/logistics.yaml
  7. Go to Activity Log --> Content: payload
  8. Go to Assembly of Logistics API --> Add Switch
     - case 0: shipping.calculate
     - case 1: get.stores
  9. shipping.calc --> Title: invoke_xyz --> add invoke: URL=$(shipping_svc_url)?company=xyz&from_zip=90210&to_zip={zip}, stop on error: unchecked, response object variable: xyz_response
  10. Clone invoke_xyz, edit, assign fields:
      - [[Title: invoke_cek]]
      - [[URL: \$(shipping_svc_url)?company=cek&from_zip=90210&to_zip= ]]
      - [[Response object variable: cek_response]]  
  11. Add map policy after invoke_cek
      - Add input:
        [[Context variable: xyz_response.body]]
        [[Name: xyz]]
        [[Content type: application/json]]
        [[Definition: #/definitions/xyz_shipping_rsp]]
      - Add another input:
        [[Context variable: cek_response.body]]
        [[Name:cek]]
        [[Content type: application/json]]
        [[Definition: #/definitions/cek_shipping_rsp]]
  12. Map the 2 inputs
  13. get.stores --> Add invoke
      [[Title: invoke_google_geolocate  ]]
      [[URL: https://maps.googleapis.com/maps/api/geocode/json?&address=]]
      [[Stop on error: unchecked ]]
      [[Response object variable (scroll to the bottom): google_geocode_response]]  
  14. Add gateway script
```
// Save the Google Geocode response body to variable
var mapsApiRsp = apim.getvariable('google_geocode_response.body');

// Get location attributes from geocode response body
var location = mapsApiRsp.results[0].geometry.location;

// Set up the response data object, concat the latitude and longitude
var rspObj = {"google_maps_link": "https://www.google.com/maps?q=" + location.lat + "," + location.lng};

// Save the output     
apim.setvariable('message.body', rspObj)
```
  15. Save and test
