# Some pre-request scripts for Postman

The following are some pre-request scripts I've created for authenticating with some services out there.

## Amazon SP-API

```js
const moment = require('moment');
const cryptojs = require('crypto-js');


// ============================================================================================
// REQUIRED CONFIGURATIONS


// IAM User credentials
const ACCESS_KEY = '';
const SECREY_KEY = '';


// LWA access token or RDT
// WARNING - This value remains valid for one hour only.
// If you are calling a restricted operation, pass in a Restricted Data Token (RDT) here instead of an LWA access token.
// - To get a LWA access token see https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#step-1-request-a-login-with-amazon-access-token
// - To get a RDT see https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/use-case-guides/tokens-api-use-case-guide/tokens-API-use-case-guide-2021-03-01.md
// and https://github.com/amzn/selling-partner-api-docs/blob/main/references/tokens-api/tokens_2021-03-01.md
AMZ_ACCESS_TOKEN = ''


// MarketplaceId and Region
// marketplaceId is required as a query string parameter but here we will include it automatically. Use MarketplaceId=ATVPDKIKX0DER for US.
// More info: https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#marketplaceid-values
const MARKETPLACE_ID = 'ATVPDKIKX0DER';
const REGION = 'us-east-1';


// Selling Partner API Endpoint
//   More info: https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#selling-partner-api-endpoints
//   for Sandbox: https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#selling-partner-api-sandbox-endpoints
// This value will be obtained from the URL automatically
const SP_API_ENDPOINT = pm.request.url.getHost();
// e.g. SP_API_ENDPOINT = 'sellingpartnerapi-na.amazon.com';
// e.g. SP_API_ENDPOINT = 'sandbox.sellingpartnerapi-na.amazon.com';


const REQUEST_VERB = pm.request.method;
// e.g. REQUEST_VERB = 'GET';


// ============================================================================================


const PATH = pm.request.url.getPath();
// e.g. PATH = '/orders/v0/orders';


const PAYLOAD = pm.request.body.toString();
// e.g. PAYLOAD = '';


// TODO: MarketplaceId or MarketplaceIds ?
pm.request.url.query.add({key: 'MarketplaceId', value: MARKETPLACE_ID});


const QUERY_STRING = pm.request.url.getQueryString();
// e.g. QUERY_STRING = `MarketplaceIds=${MARKETPLACE_ID}`;


// Dates for Signature Version 4
// More info: https://docs.aws.amazon.com/general/latest/gr/sigv4-date-handling.html
const now = moment().utc();
const amzDate = now.format('YYYYMMDD[T]HHmmss[Z]');
const eightChararactersDate = now.format('YYYYMMDD');


// Canonical Request
// More info: https://docs.aws.amazon.com/general/latest/gr/sigv4-create-canonical-request.html
const canonicalHeaders = `host:${SP_API_ENDPOINT}` + '\n'
  + `x-amz-date:${amzDate}` + '\n';
const signedHeaders = 'host;x-amz-date';
const canonicalRequest = REQUEST_VERB + '\n' // HTTPRequestMethod
  + PATH + '\n' // CanonicalURI, url-encoded, / if empty
  + QUERY_STRING + '\n' // CanonicalQueryString, only '\n' if empty
  + canonicalHeaders + '\n' // CanonicalHeaders
  + signedHeaders + '\n' // SignedHeaders
  + cryptojs.SHA256(PAYLOAD).toString().toLowerCase(); // HexEncode(Hash(RequestPayload))

console.log('Canonical request...\n' + canonicalRequest);


// String to Sign
// More info: https://docs.aws.amazon.com/general/latest/gr/sigv4-create-string-to-sign.html
const algorithm = 'AWS4-HMAC-SHA256';
const awsRegion = REGION;
const service = 'execute-api';
const terminationString = 'aws4_request'; // For AWS Signature Version 4, the value is aws4_request
// Credential Scope
// More info: https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#credential-scope
const credentialScope = `${eightChararactersDate}/${awsRegion}/${service}/${terminationString}`;
const stringToSign = algorithm + '\n' // Algorithm
  + amzDate + '\n' // RequestDateTime
  + credentialScope + '\n' // CredentialScope
  + cryptojs.SHA256(canonicalRequest).toString().toLowerCase(); // HashedCanonicalRequest

console.log('Credential Scope...\n' + credentialScope);
console.log('String to sign...\n' + stringToSign);


// Signature
// More info: https://docs.aws.amazon.com/general/latest/gr/sigv4-calculate-signature.html
const kSecret = SECREY_KEY;
const kDate = cryptojs.HmacSHA256(eightChararactersDate, 'AWS4' + kSecret);
const kRegion = cryptojs.HmacSHA256(awsRegion, kDate);
const kService = cryptojs.HmacSHA256(service, kRegion);
const kSigning = cryptojs.HmacSHA256(terminationString, kService);
const signature = cryptojs.HmacSHA256(stringToSign, kSigning);


const headers = {
  authorization: `${algorithm} `
    + `Credential=${ACCESS_KEY}/${credentialScope}, `
    + `SignedHeaders=${signedHeaders}, `
    + 'Signature=' + signature,
  host: SP_API_ENDPOINT,
  xAmzAccessToken: AMZ_ACCESS_TOKEN,
  xAmzDate: amzDate
};

console.log('Calculated header values...');
console.log(headers);


pm.request.headers.add({
  key: 'Authorization',
  value: headers.authorization
});

pm.request.headers.add({
  key: 'host',
  value: headers.host
});

pm.request.headers.add({
  key: 'x-amz-access-token',
  value: headers.xAmzAccessToken
});

pm.request.headers.add({
  key: 'x-amz-date',
  value: headers.xAmzDate
});

```

References:
- https://docs.aws.amazon.com/general/latest/gr/signature-version-4.html
- https://github.com/amzn/selling-partner-api-docs/blob/main/guides/en-US/developer-guide/SellingPartnerApiDeveloperGuide.md#step-2-construct-a-selling-partner-api-uri

## Amazon MWS

```js

_ = require('lodash');
const moment = require('moment');
const cryptojs = require('crypto-js');



// ============================================================================================
// REQUIRED CONFIGURATIONS


// Amazon MWS account credentials
// The seller can find this in Seller Central (https://sellercentral.amazon.com/home) by going to
// Settings > User Permissions > Amazon MWS Developer Access Keys > Visit Developer Credentials > click "View" on "MWS credentials" title
// or Menu > Partner Network > Develop Apps > click "View" on "MWS credentials" title
// AWS Access key Id
const ACCCESS_KEY = '';
// Secret Access Key (A.K.A. Client secret)
const SECRET_KEY = '';


// SellerId or Merchant
// The seller identifier. In Seller Central this is called the Merchant Token.
// The seller can find this in Seller Central (https://sellercentral.amazon.com/home) by going to
// Settings > Account Info > Your Merchant Token.
const SELLER_ID = '';


// The version of the API section being called
// and the action you want to perform on the endpoint, such as the operation ListOrders
// More info: http://docs.developer.amazonservices.com/en_US/orders-2013-09-01/Orders_Overview.html
const API_ACTION = '';
// e.g. const API_ACTION = 'ListOrders';
const API_VERSION = '';
// e.g. const API_VERSION = '2013-09-01';


// The MarketplaceId. ATVPDKIKX0DER for US
// More info: http://docs.developer.amazonservices.com/en_US/dev_guide/DG_Endpoints.html
const MARKETPLACE_ID = 'ATVPDKIKX0DER'


// ============================================================================================


SIGNATURE_VERSION = '2';
SIGNATURE_METHOD = 'HmacSHA256';

const REQUEST_VERB = pm.request.method;
// e.g. REQUEST_VERB = 'POST';


// The Amazon MWS Endpoint. mws.amazonservices.com for US
// More info: http://docs.developer.amazonservices.com/en_US/dev_guide/DG_Endpoints.html
// This value will be obtained from the URL automatically
const MWS_ENDPOINT = pm.request.url.getHost();
// e.g. MWS_ENDPOINT = 'mws.amazonservices.com'


const PATH = pm.request.url.getPath();
// e.g. PATH = '/Orders/2013-09-01';


// Required request parameters
// More info: http://docs.developer.amazonservices.com/en_US/dev_guide/DG_RequiredRequestParameters.html
// and http://docs.developer.amazonservices.com/en_US/dev_guide/DG_QueryString.html
const now = moment().utc();
pm.request.url.query.add({key: 'AWSAccessKeyId', value: ACCCESS_KEY});
pm.request.url.query.add({key: 'Action', value: API_ACTION});
pm.request.url.query.add({key: 'LastUpdatedAfter', value: encodeURIComponent(now.format('YYYY-MM-DD[T]HH:mm:ss[Z]'))});
pm.request.url.query.add({key: 'MarketplaceId.Id.1', value: MARKETPLACE_ID});
pm.request.url.query.add({key: 'SellerId', value: SELLER_ID});
pm.request.url.query.add({key: 'SignatureMethod', value: SIGNATURE_METHOD});
pm.request.url.query.add({key: 'SignatureVersion', value: SIGNATURE_VERSION});
pm.request.url.query.add({key: 'Timestamp', value: encodeURIComponent(now.format('YYYY-MM-DD[T]HH:mm:ss[Z]'))});
pm.request.url.query.add({key: 'Version', value: API_VERSION});

const queryString = {};
pm.request.url.query.map(item => {
    queryString[item.key] = item.value;
});
console.log(queryString);
pm.request.url.query.clear();
_.forEach(_.keys(queryString).sort(), function(paramKey) {
  pm.request.url.query.add({key: paramKey, value: queryString[paramKey]});
});

const QUERY_STRING = pm.request.url.getQueryString();
// e.g. QUERY_STRING = 'AWSAccessKeyId=XXXXXXXXXXXXXXXXXXXX'
//   + '&Action=GetOrder'
//   + '&AmazonOrderId.Id.1=0000000'
//   + '&SellerId=XXXXXXXXXXXXXX'
//   + '&Signature=XxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXxXx'
//   + '&SignatureMethod=HmacSHA256'
//   + '&SignatureVersion=2'
//   + '&Timestamp=2014-10-21T19%3A49%3A02Z'
//   + '&Version=2013-09-01';

// Canonicalized Query String
// More info: http://docs.developer.amazonservices.com/en_US/dev_guide/DG_QueryString.html
// and https://docs.aws.amazon.com/general/latest/gr/signature-version-2.html
const canonizedQueryString = REQUEST_VERB + '\n'
  + MWS_ENDPOINT + '\n'
  + PATH + '\n'
  + QUERY_STRING;

console.log('Canonized Query String ...\n' + canonizedQueryString);

// Request signature
// More info: http://docs.developer.amazonservices.com/en_US/dev_guide/DG_SigningQueryRequest.html
// and https://docs.aws.amazon.com/general/latest/gr/signature-version-2.html
const signature = cryptojs.HmacSHA256(canonizedQueryString, SECRET_KEY).toString(cryptojs.enc.Base64);

pm.request.url.query.add({key: 'Signature', value: encodeURIComponent(signature)});

```

References:
- http://docs.developer.amazonservices.com/en_US/dev_guide/DG_SigningQueryRequest.html
- https://docs.aws.amazon.com/general/latest/gr/signature-version-2.html
