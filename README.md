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

## GraphQL Proxy

## API Connect Toolkit

## App Connect Integration

