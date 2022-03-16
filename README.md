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



