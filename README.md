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
         - Response: 200
     - Target Services: Add Web Services --> Enter WSDL from https://integrationsuperhero.github.io/techcon2020/APICDevJam/resources/calculate.wsdl  
  2. Go to Assembly, remove Invoke
  3. Drag and drop financing Web Service Operations
  4. Edit /financing input --> + GET Calculate operation, map
  5. Edit /financing output --> + GET /calculate, map
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
