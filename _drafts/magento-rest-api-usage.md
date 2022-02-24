# Magento's REST API usage

In this guide I will be using `magento.local` as the server where I have running my instance of magento. Change it where necessary.

## Online REST API reference

The REST API Documentation can be accessed online here: https://magento.redoc.ly/. It describes the REST APIs that are available on the latest release of Magento 2.

This documentation uses [ReDoc](https://github.com/Rebilly/ReDoc) to organize and present schema files that follow an Open-API specification. The schema files this tool uses are generated from a running instance of the latest release of Magento 2 (Magento 2.4 currently), and it represents the state of the code at the time the file was generated.

## REST API reference in local instance

Magento uses [Swagger](http://swagger.io/) to display REST APIs for all installed products.

The Swagger UI is installed automatically on the server. As a result, you can generate live REST API documentation that can include:
- Adobe Commerce modules,
- third-party modules,
- and extension attributes
that have been installed on your system.

To view this documentation, use this URL: `http://magento.local/swagger`

By default, Magento returns documentation for resources available to anonymous users across all stores. If you specify a valid customer token (POST /V1/integration/customer/token) or admin token (POST /V1/integration/admin/token) in the `api_key` text box in the upper right corner, Swagger returns documentation for all the endpoints the user has access to.

The generated Swagger documentation provides the capability to test REST requests. A user can enter a sample request, then press the **Try it out!** button, and Swagger returns information such as a curl command, a request URL, a response body, a response code, and the response header. The **Try it out!** button will not work unless a bearer authorization token has been specified.

### Swagger documentation for a specific store view

To view the Swagger documentation for a specific store view, use this URL: `http://magento.cloud/swagger?store=<store_code>`

The value of store_code must be one of the following:

- `default`
- The assigned store code
- `all`. This value only applies to the CMS and Product modules. If this value is specified, the API call affects all the merchant's stores.

### Swagger documentation for Asynchronous API

You can also use Swagger to generate live asynchronous API REST documentation. To create this documentation, add the ?type=async parameter to the standard Swagger URL: `http://magento.cloud/swagger?type=async`

Swagger returns information about all resources available for asynchronous REST APIs.

### Web APIs allowed for anonymous users (aka. guest users)

The Magento web API framework allows guest users to access resources that are configured with the permission level of anonymous. Guest users are users who the framework cannot authenticate through existing authentication mechanisms.

Look at https://devdocs.magento.com/guides/v2.3/rest/anonymous-api-security.html for a list of both accessible and no longer available APIs for anonymous users.

## Token-based authentication

Intended for making web API calls from clients such as a mobile applications.

Here the REST client must supply an "access token" on the call.

For most web API calls, you supply this token in the `Authorization` request header with the "Bearer HTTP authorization scheme# to prove your identity.

### Access Token types

- Integration - The merchant determines which Magento resources the integration has access to. Default lifetime: Indefinite. It lasts until it is manually revoked.
- Admin - The merchant determines which Magento resources an admin user has access to. Default lifetime: 4 hours.
- Customer - Magento grants access to resources with the anonymous or self permission. Merchants cannot edit these settings. Default lifetime: 1 hour.

By default, an admin token is valid for 4 hours, while a customer token is valid for 1 hour.

You can change these values from Admin in **Stores > Settings > Configuration > Services > OAuth > Access Token Expiration**.

A cron job that runs hourly removes all expired tokens.

### Access tokens for Integrations

When a merchant creates and activates an integration, Magento generates:
- a Consumer Key,
- a Consumer Secret,
- an Access Token,
- and an Access Token Secret.

All of these entities are used for "OAuth-based authentication", but token-based authentication requires only the access token.

Use the following steps to generate an access token:

1. Log in to Admin and go to **System > Extensions > Integrations** to display the Integrations page.
2. Click "Add New Integration" to display the New Integration page.
3. Enter a unique name for the integration in the Name field. Then enter your admin password in the Your Password field. Leave all other fields blank.
4. Click the API tab. Select the Magento resources the integration can access. You can select all resources, or select a custom list.
5. Click Save to save your changes and return to the Integrations page.
6. Click the Activate link in the grid that corresponds to the newly-created integration.
7. Click Allow. A dialog will be open displaying the credentials mentioned above.

### Access tokens for Admin and Customer accounts

Magento provides a service for administrators and customers that returns a unique access token in exchange of the username and password of the given Magento account.

Elements in requests for access tokens:
- Endpoint: A combination of the server that fulfills the request, the web service, and the resource against which the request is being made. The resource can be "/V1/integration/admin/token" for admin accounts or "/V1/integration/customer/token" for customer accounts. For example `magento.local/rest/V1/integration/customer/token`
```
Server:         magento.local
Web Service:    rest
Resource:       /V1/integration/customer/token
```
- HTTP headers: the Content-Type header and the Accept. Can be "Content-Type:application/json" or "Content-Type:application/xml".
- Call payload: The call payload is a set of input parameters (URI parameters and query string) and attributes (the request body) that you supply with the request. In this case you have to send the credentials (The username and password for a Magento account) as input attributes.
```
For JSON content type:
{"username":"<USER-NAME>;", "password":"<PASSWORD>"}

For XML content type:
<login><username>customer1</username><password>customer1pw</password></login>
```

Examples:

```sh
# Request access token for customer "roni_cost@example.com"
curl -X POST "http://magento.local/index.php/rest/V1/integration/customer/token" \
    -H "Content-Type:application/json" \
    -d '{"username":"roni_cost@example.com", "password":"roni_cost3@example.com"}'
# Request access token for admin "admin"
curl -X POST "http://magento.local/index.php/rest/V1/integration/admin/token" \
     -H "Content-Type:application/json" \
     -d '{"username":"admin", "password":"admin123"}'
```

### Accessing restricted resources using the access token

Any web API call that accesses a resource that requires a permission level higher than `anonymous` must contain the authentication token in the header To do this, specify a HTTP header in the following format: `Authorization: Bearer <authentication token>`.

Examples:
```sh
# Accessing a resource authorized for customers.
curl -X GET "http://magento.local/index.php/rest/V1/customers/me" \
    -H "Authorization: Bearer xxxxxxxxxxxxxxxxx"
# Accessing a resource authorized for admins.
curl -X GET "http://magento.local/index.php/rest/V1/customers/2" \
    -H "Authorization: Bearer xxxxxxxxxxxxxxxxx"
```

## OAuth-based authentication

OAuth is a token-passing mechanism that allows a system to control which third-party applications have access to internal data without revealing or storing any user IDs or passwords.

In Magento, a third-party application that uses OAuth for authentication is called an "integration". An integration defines which resources the application can access. The application can be granted access to all resources or a customized subset of resources.

As the process of registering the integration proceeds, Magento creates the tokens that the application needs for authentication. It first creates a request token. This token is short-lived and must be exchanged for an access token. Access tokens are long-lived and will not expire unless the merchant revokes access from the application.

More info: https://devdocs.magento.com/guides/v2.3/get-started/authentication/gs-authentication-oauth.html

## Common REST call structure

A basic structure of a REST call in Magento is: `<HTTP verb> http://<host>/rest/<scope>/<endpoint>`


| ELEMENT   | DESCRIPTION                                                                        |
|-----------|------------------------------------------------------------------------------------|
| HTTP verb | One of GET, POST, PUT, or DELETE                                                   |
| host      | The hostname or IP address (and optionally, port) of the Magento installation.     |
| scope     | Specifies which store the call affects. In this tutorial, this value is `default`. |
| endpoint  | The full URI (Uniform Resource Identifier) to the endpoint. These values always start with `/V1`. For example, ``/V1/orders/4`. |

## webapi.xml

Any Magento or third-party service defines the web API resources and their permissions in the configuration file called `webapi.xml`. This file is located in `<module root dir>/vendor/<vendor-name>/<module-name>/etc/webapi.xml` where `<vendor-name>` is your vendor name (for example, magento) and `<module-name>` is your module name (which exactly matches its definition in composer.json). For example, the web API for the Customer service is defined in the `<magento_root>/vendor/magento/module-customer/etc/webapi.xml` configuration file.

Service data interfaces and builders define the required and optional parameters and the return values for the API calls.

If a service is not defined with this configuration file, it will not be exposed at all.

### Construct a request from webapi.xml

This example shows you how to construct a REST web API call to create an account.

**Step 1 - Explore the webapi.xml file**

Open [Magento/Customer/etc/webapi.xml](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Customer/etc/webapi.xml)

**Step 2 - Find the route element that defines the method**

Find the route element that defines the createAccount call:

```xml
<route url="/V1/customers" method="POST">
   <service class="Magento\Customer\Api\AccountManagementInterface" method="createAccount"/>
   <resources>
      <resource ref="anonymous"/>
   </resources>
</route>
```

**Step 3 - Use 'method' and 'url' attributes on the route element to construct the URI**

Use the values in the `method` and `url` attributes on the `route` element to construct the URI. In this example, the URI is: `POST /V1/customers`.

**Step 4 - Use 'class' attribute on the service element to identify the service interface**

Use the `class` attribute on the `service` element to identify the service interface. In this example, the service interface is the [AccountManagementInterface](https://github.com/magento/magento2/blob/2.3/app/code/Magento/Customer/Api/AccountManagementInterface.php) PHP file.

Open the AccountManagementInterface.php file and find the `createAccount` method, as follows:

```php
public function createAccount(
    \Magento\Customer\Api\Data\CustomerInterface $customer,
    $password = null,
    $redirectUrl = ''
)
```

The `createAccount` call requires a `customer` data object. The `password` and `redirectUrl` values are optional. The default password value is null and the default redirectUrl value is blank.

To pass the `customer` data object in the POST call payload, specify JSON or XML request body on the call.

**Step 5 - Construct the request**

In the `POST /V1/customers` call, you must specify a request body like this:

```json
{
  "customer": {
    "email": "user@example.com",
    "firstname": "John",
    "lastname": "Doe"
  },
  "addresses": [
    {
      "defaultShipping": true,
      "defaultBilling": true,
      "firstname": "John",
      "lastname": "Doe",
      "region": {
        "regionCode": "CA",
        "region": "California",
        "regionId": 12
      },
      "postcode": "90001",
      "street": ["Zoe Ave"],
      "city": "Los Angeles",
      "telephone": "555-000-00-00",
      "countryId": "US"
    }
  ]
}
```

## Request examples

### Create a customer account

**Endpoint**

`POST <host>/rest/<store_code>/V1/customers`

**Headers**

`Content-Type: application/json`

**Payload**

```json
{
  "customer": {
    "email": "jdoe@example.com",
    "firstname": "Jane",
    "lastname": "Doe",
    "addresses": [
      {
        "defaultShipping": true,
        "defaultBilling": true,
        "firstname": "Jane",
        "lastname": "Doe",
        "region": {
          "regionCode": "NY",
          "region": "New York",
          "regionId": 43
        },
        "postcode": "10755",
        "street": [
          "123 Oak Ave"
        ],
        "city": "Purchase",
        "telephone": "512-555-1111",
        "countryId": "US"
      }
    ]
  },
  "password": "Password1"
}
```

It is recommended that you substitute the value of the `email` parameter with a real email address so that you receive all notifications.

**Response**

```json
{
  "id": 2,
  "group_id": 1,
  "default_billing": "2",
  "default_shipping": "2",
  "created_at": "2017-01-31 01:18:13",
  "updated_at": "2017-01-31 01:18:13",
  "created_in": "Default Store View",
  "email": "jdoe@example.com",
  "firstname": "Jane",
  "lastname": "Doe",
  "store_id": 1,
  "website_id": 1,
  "addresses": [
    {
      "id": 2,
      "customer_id": 2,
      "region": {
        "region_code": "NY",
        "region": "New York",
        "region_id": 43
      },
      "region_id": 43,
      "country_id": "US",
      "street": [
        "123 Oak Ave"
      ],
      "telephone": "512-555-1111",
      "postcode": "10755",
      "city": "Purchase",
      "firstname": "Jane",
      "lastname": "Doe",
      "default_shipping": true,
      "default_billing": true
    }
  ],
  "disable_auto_group_change": 0
}
```

Now you can log in to the Luma store using the username "jdoe@example.com" and password "Password1".

- Log in to the Luma website using the email jdoe@example.com and password Password1.
- Click the account name (Jane) in the upper right corner and select My Account.
- Click Address Book to view the default billing and shipping addresses.

### Get the customer's access token

**Endpoint**

`POST <host>/rest/<store_code>/V1/integration/customer/token`

**Headers**

`Content-Type: application/json`

**Payload**

```json
{
  "username": "jdoe@example.com",
  "password": "Password1"
}
```

**Response**

`q0u66k8h42yaevtchv09uyy3y9gaj2ap`

Magento returns the customer's access token. This token must be specified in the authorization header of every call the customer makes on his or her own behalf.

### Create a quote (aka. cart)

When a customer adds an item to their shopping cart for the first time, Magento creates a quote. Magento uses a quote to perform tasks such as

- Track each item the customer wants to buy, including the quantity and base cost
- Gather information about the customer, including billing and shipping addresses
- Determine shipping costs
- Calculate the subtotal, add costs (shipping fees, taxes, etc.) and apply coupons to determine the grand total
- Determine the payment method
- Place the order so that the merchant can fulfill it.

**Types of carts**

Magento identifies three types of users that can create a shopping cart:

- An admin user can create a cart on behalf of a customer. For all admin requests, you must provide an admin authorization token in the call's authorization header.
- A logged-in customer. Calls to create a cart and add items must contain the customer's authorization token in the authorization header.
- A guest user. These users could be customers who haven't logged in yet, or they could be users who have no intention of creating an account. An anonymous user's cart is called a guest cart.

**For anonymous accounts**

Use the V1/guest-carts endpoint to create a cart on behalf of a guest. Do not include an authorization token. The quoteId for the guest customer quote will be masked.

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

None

**Response**

quoteId: 4

Some calls refer to this parameter (`quoteId`) as the `cartId`.

NOTE: `quoteId` values are not displayed on the website or in Admin.

### Get information about configurable products

To add a configurable product to a cart, you must specify the sku as well as the set of `option_id` and `option_value` pairs that make the product configurable.

In this example, we'll use "Chaz Kangeroo Hoodie" (sku: MH01) configurable product as an example. This product comes in three colors (black, gray, and orange) and five sizes (XS, S, M, L, XL). In the sample data, the `option_id` values for Size and Color are 141 and 93, respectively. You can use the `GET /V1/configurable-products/:sku/options/all` call to determine the `option_id` values for the given SKU.

The `GET /V1/configurable-products/:sku/children` call returns information about each combination of color and size, 15 in all for MH01.

The following sample shows the returned values for size and color for a small (S) gray "Chaz Kangeroo Hoodie". From there we now know the values for option_value for size and color are 168 and 52, so we're ready to add the product to the cart.

```json
{
  "custom_attributes": [
    {
      "attribute_code": "size",
      "value": "168"
    },
    {
      "attribute_code": "color",
      "value": "52"
    }
  ]
}
```

### Get information about bundle products

Here we can see how I extract information about the "Sprite Yoga Companion Kit" (sku: 24-WG080).

The kit contains the following items:

- Sprite Statis Ball in sizes 55 cm (sku: 24-WG081-blue), 65 cm (sku: 24-WG082-blue), or 75 cm (sku: 24-WG083-blue)
- Sprite Foam Yoga brick (sku: 24-WG084)
- Sprite Yoga Strap in lengths 6 ft (sku: 24-WG085), 8 ft (sku: 24-WG086), or 10 ft (sku: 24-WG087)
- Sprite Foam Roller (sku: 24-WG088)

The `GET <host>/rest/<store_code>/V1/bundle-products/24-WG080/options/all` call returns id values, as shown in the following simplified response:

```json
[
  {
    "option_id": 1,
    "title": "Sprite Stasis Ball",
    "required": true,
    "type": "radio",
    "position": 1,
    "sku": "24-WG080",
    "product_links": [
      {
        "id": "1",
        "sku": "24-WG081-blue",
        "option_id": 1,
        "qty": 1
      },
      {
        "id": "2",
        "sku": "24-WG082-blue",
        "option_id": 1,
        "qty": 1
      },
      {
        "id": "3",
        "sku": "24-WG083-blue",
        "option_id": 1,
        "qty": 1
      }
    ]
  },
  {
    "option_id": 2,
    "title": "Sprite Foam Yoga Brick",
    "required": true,
    "type": "radio",
    "position": 2,
    "sku": "24-WG080",
    "product_links": [
      {
        "id": "4",
        "sku": "24-WG084",
        "option_id": 2,
        "qty": 1
      }
    ]
  },
  {
    "option_id": 3,
    "title": "Sprite Yoga Strap",
    "required": true,
    "type": "radio",
    "position": 3,
    "sku": "24-WG080",
    "product_links": [
      {
        "id": "5",
        "sku": "24-WG085",
        "option_id": 3,
        "qty": 1
      },
      {
        "id": "6",
        "sku": "24-WG086",
        "option_id": 3,
        "qty": 1
      },
      {
        "id": "7",
        "sku": "24-WG087",
        "option_id": 3,
        "qty": 1
      }
    ]
  },
  {
    "option_id": 4,
    "title": "Sprite Foam Roller",
    "required": true,
    "type": "radio",
    "position": 4,
    "sku": "24-WG080",
    "product_links": [
      {
        "id": "8",
        "sku": "24-WG088",
        "option_id": 4,
        "qty": 1
      }
    ]
  }
]
```

### Add items to the cart

To check if an item was added, you can sign in as the customer and click on the shopping cart. All the items you added are displayed.

**For anonymous accounts**

Use the `V1/guest-carts/<cartId>/items` endpoint to add items to the cart on behalf of a guest. Do not include an authorization token. The payload and response is same as the logged-in customer for all product types, except for from quote ID in the payload.

#### Add a simple product to a cart

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/items`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "cartItem": {
    "sku": "WS12-M-Orange",
    "qty": 1,
    "quote_id": "4"
  }
}
```

**Response**

```json
{
  "item_id": 7,
  "sku": "WS12-M-Orange",
  "qty": 1,
  "name": "Radiant Tee-M-Orange",
  "price": 19.99,
  "product_type": "simple",
  "quote_id": "4"
}
```

#### Add a downloadable product to a cart

The requirements for adding a downloadable product to a cart are the same as a simple product.

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/items`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "cartItem": {
    "sku": "240-LV08",
    "qty": 1,
    "quote_id": "4"
  }
}
```

**Response**

```json
{
  "item_id": 8,
  "sku": "240-LV08",
  "qty": 1,
  "name": "Advanced Pilates & Yoga (Strength)",
  "price": 18,
  "product_type": "downloadable",
  "quote_id": "4",
  "product_option": {
    "extension_attributes": {
      "downloadable_option": {
        "downloadable_links": [
          5
        ]
      }
    }
  }
}
```

#### Add a configurable product to a cart

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/items`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "cartItem": {
    "sku": "MH01",
    "qty": 1,
    "quote_id": "4",
    "product_option": {
      "extension_attributes": {
        "configurable_item_options": [
          {
            "option_id": "93",
            "option_value": 52
          },
          {
            "option_id": "141",
            "option_value": 168
          }
        ]
      }
    },
    "extension_attributes": {}
  }
}
```

**Response**

```json
{
  "item_id": 13,
  "sku": "MH01-S-Gray",
  "qty": 1,
  "name": "Chaz Kangeroo Hoodie",
  "price": 52,
  "product_type": "configurable",
  "quote_id": "4",
  "product_option": {
    "extension_attributes": {
      "configurable_item_options": [
        {
          "option_id": "93",
          "option_value": 52
        },
        {
          "option_id": "141",
          "option_value": 168
        }
      ]
    }
  }
}
```

#### Add a bundle product to a cart

To add a bundle product to a cart, you must specify the `sku` of the bundle product, but not the individual items. You add individual items to the bundle product by specifying the `id` defined in the item’s `product_links` object. The `product_links` object primarily describes the ordering and placement of options on the customization page, but it also links an item’s `sku` and `id` to the `sku` of the bundle product.

For this example, we’ll configure the "Sprite Yoga Companion Kit" as follows:

- 65 cm Sprite Stasis Ball (id: 2)
- Sprite Foam Yoga Brick (id: 4)
- 8 ft Sprite Yoga strap (id: 6)
- Sprite Foam Roller (id: 8)

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/items`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "cartItem": {
    "sku": "24-WG080",
    "qty": 1,
    "quote_id": "4",
    "product_option": {
      "extension_attributes": {
        "bundle_options": [
          {
            "option_id": 1,
            "option_qty": 1,
            "option_selections": [2]
          },
          {
            "option_id": 2,
            "option_qty": 1,
            "option_selections": [4]
          },
          {
            "option_id": 3,
            "option_qty": 1,
            "option_selections": [6]
          },
          {
            "option_id": 4,
            "option_qty": 1,
            "option_selections": [8]
          }
        ]
      }
    }
  }
}
```

**Response**

```json
{
  "item_id": 9,
  "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
  "qty": 1,
  "name": "Sprite Yoga Companion Kit",
  "price": 68,
  "product_type": "bundle",
  "quote_id": "4",
  "product_option": {
    "extension_attributes": {
      "bundle_options": [
        {
          "option_id": 1,
          "option_qty": 1,
          "option_selections": [
            2
          ]
        },
        {
          "option_id": 2,
          "option_qty": 1,
          "option_selections": [
            4
          ]
        },
        {
          "option_id": 3,
          "option_qty": 1,
          "option_selections": [
            6
          ]
        },
        {
          "option_id": 4,
          "option_qty": 1,
          "option_selections": [
            8
          ]
        }
      ]
    }
  }
}
```

### Prepare for checkout: Estimate shipping costs & Set shipping and billing information

#### Estimate shippping costs

Note that the cost for the `flatrate` shipping method is $15. The Sprite Yoga Companion Kit bundled product counts as one item. The Advanced Pilates & Yoga item does not have a shipping charge because the customer downloads this item.

**For anonymous accounts**

Use the `V1/guest-carts/<cartId>/estimate-shipping-methods` endpoint to estimate shipping costs on behalf of a guest. Do not include an authorization token.

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/estimate-shipping-methods`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "address": {
    "region": "New York",
    "region_id": 43,
    "region_code": "NY",
    "country_id": "US",
    "street": [
      "123 Oak Ave"
    ],
    "postcode": "10577",
    "city": "Purchase",
    "firstname": "Jane",
    "lastname": "Doe",
    "customer_id": 4,
    "email": "jdoe@example.com",
    "telephone": "(512) 555-1111",
    "same_as_billing": 1
  }
}
```

**Response**

```json
[
  {
    "carrier_code": "flatrate",
    "method_code": "flatrate",
    "carrier_title": "Flat Rate",
    "method_title": "Fixed",
    "amount": 15,
    "base_amount": 15,
    "available": true,
    "error_message": "",
    "price_excl_tax": 15,
    "price_incl_tax": 15
  },
  {
    "carrier_code": "tablerate",
    "method_code": "bestway",
    "carrier_title": "Best Way",
    "method_title": "Table Rate",
    "amount": 5,
    "base_amount": 5,
    "available": true,
    "error_message": "",
    "price_excl_tax": 5,
    "price_incl_tax": 5
  }
]
```

#### Set shipping and billing information

In this call, you specify the shipping and billing addresses, as well as the selected `carrier_code` and `method_code`. Since the Table Rate shipping method costs only $5, the customer selected this option.

Magento returns a list of payment options and calculates the order totals.

The subtotal of the order is $160, and shipping charges are $5. The grand total is $165.

The available payment methods are `banktransfer` and `checkmo`. The customer will specify a payment method in the step "Send payment information".

NOTE: If you tried this call on your own, and the value of the shipping_amount parameter is 0, then you did not deactivate the “Spend $50 or more - shipping is free!” cart price rule. See [Deactivate a cart price rule](https://devdocs.magento.com/guides/v2.4/rest/tutorials/orders/order-config-store.html#price-rule) for details.

**For anonymous accounts**

Use the `V1/guest-carts/<cartId>/shipping-information` endpoint to set the billing and shipping information on behalf of a guest. Do not include an authorization token.

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/shipping-information`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "addressInformation": {
    "shipping_address": {
      "region": "New York",
      "region_id": 43,
      "region_code": "NY",
      "country_id": "US",
      "street": [
        "123 Oak Ave"
      ],
      "postcode": "10577",
      "city": "Purchase",
      "firstname": "Jane",
      "lastname": "Doe",
      "email": "jdoe@example.com",
      "telephone": "512-555-1111"
    },
    "billing_address": {
      "region": "New York",
      "region_id": 43,
      "region_code": "NY",
      "country_id": "US",
      "street": [
        "123 Oak Ave"
      ],
      "postcode": "10577",
      "city": "Purchase",
      "firstname": "Jane",
      "lastname": "Doe",
      "email": "jdoe@example.com",
      "telephone": "512-555-1111"
    },
    "shipping_carrier_code": "tablerate",
    "shipping_method_code": "bestway"
  }
}

```

**Response**

```json
{
  "payment_methods": [
    {
      "code": "cashondelivery",
      "title": "Cash On Delivery"
    },
    {
      "code": "banktransfer",
      "title": "Bank Transfer Payment"
    },
    {
      "code": "purchaseorder",
      "title": "Purchase Order"
    },
    {
      "code": "checkmo",
      "title": "Check / Money order"
    }
  ],
  "totals": {
    "grand_total": 165,
    "base_grand_total": 165,
    "subtotal": 160,
    "base_subtotal": 160,
    "discount_amount": 0,
    "base_discount_amount": 0,
    "subtotal_with_discount": 160,
    "base_subtotal_with_discount": 160,
    "shipping_amount": 5,
    "base_shipping_amount": 5,
    "shipping_discount_amount": 0,
    "base_shipping_discount_amount": 0,
    "tax_amount": 0,
    "base_tax_amount": 0,
    "weee_tax_applied_amount": null,
    "shipping_tax_amount": 0,
    "base_shipping_tax_amount": 0,
    "subtotal_incl_tax": 160,
    "shipping_incl_tax": 5,
    "base_shipping_incl_tax": 5,
    "base_currency_code": "USD",
    "quote_currency_code": "USD",
    "items_qty": 4,
    "items": [
      {
        "item_id": 6,
        "price": 22,
        "base_price": 22,
        "qty": 1,
        "row_total": 22,
        "base_row_total": 22,
        "row_total_with_discount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "tax_percent": 0,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "discount_percent": 0,
        "price_incl_tax": 22,
        "base_price_incl_tax": 22,
        "row_total_incl_tax": 22,
        "base_row_total_incl_tax": 22,
        "options": "[]",
        "weee_tax_applied_amount": null,
        "weee_tax_applied": null,
        "name": "Radiant Tee-M-Orange"
      },
      {
        "item_id": 7,
        "price": 18,
        "base_price": 18,
        "qty": 1,
        "row_total": 18,
        "base_row_total": 18,
        "row_total_with_discount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "tax_percent": 0,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "discount_percent": 0,
        "price_incl_tax": 18,
        "base_price_incl_tax": 18,
        "row_total_incl_tax": 18,
        "base_row_total_incl_tax": 18,
        "options": "[{\"value\":\"Advanced Pilates & Yoga (Strength)\",\"label\":\"Downloads\"}]",
        "weee_tax_applied_amount": null,
        "weee_tax_applied": null,
        "name": "Advanced Pilates & Yoga (Strength)"
      },
      {
        "item_id": 8,
        "price": 68,
        "base_price": 68,
        "qty": 1,
        "row_total": 68,
        "base_row_total": 68,
        "row_total_with_discount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "discount_percent": 0,
        "price_incl_tax": 68,
        "base_price_incl_tax": 68,
        "row_total_incl_tax": 68,
        "base_row_total_incl_tax": 68,
        "options": "[{\"value\":\"1 x Sprite Stasis Ball 65 cm <span class=\\\"price\\\">$27.00<\\/span>\",\"label\":\"Sprite Stasis Ball\"},{\"value\":\"1 x Sprite Foam Yoga Brick <span class=\\\"price\\\">$5.00<\\/span>\",\"label\":\"Sprite Foam Yoga Brick\"},{\"value\":\"1 x Sprite Yoga Strap 8 foot <span class=\\\"price\\\">$17.00<\\/span>\",\"label\":\"Sprite Yoga Strap\"},{\"value\":\"1 x Sprite Foam Roller <span class=\\\"price\\\">$19.00<\\/span>\",\"label\":\"Sprite Foam Roller\"}]",
        "weee_tax_applied_amount": null,
        "weee_tax_applied": null,
        "name": "Sprite Yoga Companion Kit"
      },
      {
        "item_id": 13,
        "price": 52,
        "base_price": 52,
        "qty": 1,
        "row_total": 52,
        "base_row_total": 52,
        "row_total_with_discount": 0,
        "tax_amount": 0,
        "base_tax_amount": 0,
        "tax_percent": 0,
        "discount_amount": 0,
        "base_discount_amount": 0,
        "discount_percent": 0,
        "price_incl_tax": 52,
        "base_price_incl_tax": 52,
        "row_total_incl_tax": 52,
        "base_row_total_incl_tax": 52,
        "options": "[{\"value\":\"Gray\",\"label\":\"Color\"},{\"value\":\"S\",\"label\":\"Size\"}]",
        "weee_tax_applied_amount": null,
        "weee_tax_applied": null,
        "name": "Chaz Kangeroo Hoodie"
      }
    ],
    "total_segments": [
      {
        "code": "subtotal",
        "title": "Subtotal",
        "value": 160
      },
      {
        "code": "shipping",
        "title": "Shipping & Handling (Best Way - Table Rate)",
        "value": 5
      },
      {
        "code": "tax",
        "title": "Tax",
        "value": 0,
        "extension_attributes": {
          "tax_grandtotal_details": []
        }
      },
      {
        "code": "grand_total",
        "title": "Grand Total",
        "value": 165,
        "area": "footer"
      }
    ]
  }
}
```

### Create an Order: Send payment information

In this example the shopping cart contains three items totaling $108. The shipping charges are $10, making the grand total $118. We’re now ready to convert the quote to an order.

Your can check it with the following steps:
- Log in to the Luma store as the customer. The dashboard shows the order.
- Log in to Admin. Click Sales > Orders. The order is displayed in the grid. Its status is Pending.

#### Send payment information

When you submit payment information, Magento creates an order and sends an order confirmation to the customer.

Since we are using an offline payment method in this tutorial, we do not need to provide detailed payment information. The endpoint used in this example requires only the payment method and billing address information.

**For anonymous accounts**

Use the `V1/guest-carts/<cartId>/payment-information` endpoint to set the payment information on behalf of a guest. Do not include an authorization token. You must include the `email` attribute in the payload at the same level as `paymentMethod` and `billing_address`.

**Endpoint**

`POST <host>/rest/<store_code>/V1/carts/mine/payment-information`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <customer token>`

**Payload**

```json
{
  "paymentMethod": {
    "method": "banktransfer"
  },
  "billing_address": {
    "email": "jdoe@example.com",
    "region": "New York",
    "region_id": 43,
    "region_code": "NY",
    "country_id": "US",
    "street": [
      "123 Oak Ave"
    ],
    "postcode": "10577",
    "city": "Purchase",
    "telephone": "512-555-1111",
    "firstname": "Jane",
    "lastname": "Doe"
  }
}
```

**Response**

An `orderID`, such as 3.

### Review the order with an admin account

**Endpoint**

`GET <host>/rest/<store_code>/V1/orders/3`

where 3 is the `orderid`.

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <administrator token>`

**Payload**

None

**Response**

```json
{
  "applied_rule_ids": "1",
  "base_currency_code": "USD",
  "base_discount_amount": 0,
  "base_grand_total": 165,
  "base_discount_tax_compensation_amount": 0,
  "base_shipping_amount": 5,
  "base_shipping_discount_amount": 0,
  "base_shipping_incl_tax": 5,
  "base_shipping_tax_amount": 0,
  "base_subtotal": 160,
  "base_subtotal_incl_tax": 160,
  "base_tax_amount": 0,
  "base_total_due": 165,
  "base_to_global_rate": 1,
  "base_to_order_rate": 1,
  "billing_address_id": 6,
  "created_at": "2017-08-21 22:22:19",
  "customer_email": "jdoe@example.com",
  "customer_firstname": "Jane",
  "customer_group_id": 1,
  "customer_id": 3,
  "customer_is_guest": 0,
  "customer_lastname": "Doe",
  "customer_note_notify": 1,
  "discount_amount": 0,
  "email_sent": 1,
  "entity_id": 3,
  "global_currency_code": "USD",
  "grand_total": 165,
  "discount_tax_compensation_amount": 0,
  "increment_id": "000000003",
  "is_virtual": 0,
  "order_currency_code": "USD",
  "protect_code": "61f76d",
  "quote_id": 4,
  "shipping_amount": 5,
  "shipping_description": "Best Way - Table Rate",
  "shipping_discount_amount": 0,
  "shipping_discount_tax_compensation_amount": 0,
  "shipping_incl_tax": 5,
  "shipping_tax_amount": 0,
  "state": "new",
  "status": "pending",
  "store_currency_code": "USD",
  "store_id": 1,
  "store_name": "Main Website\nMain Website Store\n",
  "store_to_base_rate": 0,
  "store_to_order_rate": 0,
  "subtotal": 160,
  "subtotal_incl_tax": 160,
  "tax_amount": 0,
  "total_due": 165,
  "total_item_count": 7,
  "total_qty_ordered": 4,
  "updated_at": "2017-08-21 22:22:20",
  "weight": 2,
  "items": [
    {
      "amount_refunded": 0,
      "applied_rule_ids": "1",
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 22,
      "base_price": 22,
      "base_price_incl_tax": 22,
      "base_row_invoiced": 0,
      "base_row_total": 22,
      "base_row_total_incl_tax": 22,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 3,
      "name": "Radiant Tee-M-Orange",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 22,
      "price": 22,
      "price_incl_tax": 22,
      "product_id": 1553,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 6,
      "row_invoiced": 0,
      "row_total": 22,
      "row_total_incl_tax": 22,
      "row_weight": 1,
      "sku": "WS12-M-Orange",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "weight": 1
    },
    {
      "amount_refunded": 0,
      "applied_rule_ids": "1",
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 18,
      "base_price": 18,
      "base_price_incl_tax": 18,
      "base_row_invoiced": 0,
      "base_row_total": 18,
      "base_row_total_incl_tax": 18,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 1,
      "item_id": 4,
      "name": "Advanced Pilates & Yoga (Strength)",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 18,
      "price": 18,
      "price_incl_tax": 18,
      "product_id": 49,
      "product_type": "downloadable",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 7,
      "row_invoiced": 0,
      "row_total": 18,
      "row_total_incl_tax": 18,
      "row_weight": 0,
      "sku": "240-LV08",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19"
    },
    {
      "amount_refunded": 0,
      "applied_rule_ids": "1",
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_original_price": 68,
      "base_price": 68,
      "base_price_incl_tax": 68,
      "base_row_invoiced": 0,
      "base_row_total": 68,
      "base_row_total_incl_tax": 68,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 5,
      "name": "Sprite Yoga Companion Kit",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 68,
      "price": 68,
      "price_incl_tax": 68,
      "product_id": 51,
      "product_type": "bundle",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 8,
      "row_invoiced": 0,
      "row_total": 68,
      "row_total_incl_tax": 68,
      "row_weight": 0,
      "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "updated_at": "2017-08-21 22:22:19",
      "weight": 0
    },
    {
      "amount_refunded": 0,
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 27,
      "base_price": 27,
      "base_price_incl_tax": 27,
      "base_row_invoiced": 0,
      "base_row_total": 27,
      "base_row_total_incl_tax": 27,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 6,
      "name": "Sprite Stasis Ball 65 cm",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 27,
      "parent_item_id": 5,
      "price": 27,
      "price_incl_tax": 27,
      "product_id": 29,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 9,
      "row_invoiced": 0,
      "row_total": 27,
      "row_total_incl_tax": 27,
      "row_weight": 0,
      "sku": "24-WG082-blue",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "parent_item": {
        "amount_refunded": 0,
        "applied_rule_ids": "1",
        "base_amount_refunded": 0,
        "base_discount_amount": 0,
        "base_discount_invoiced": 0,
        "base_original_price": 68,
        "base_price": 68,
        "base_price_incl_tax": 68,
        "base_row_invoiced": 0,
        "base_row_total": 68,
        "base_row_total_incl_tax": 68,
        "base_tax_amount": 0,
        "base_tax_invoiced": 0,
        "created_at": "2017-08-21 22:22:19",
        "discount_amount": 0,
        "discount_invoiced": 0,
        "discount_percent": 0,
        "free_shipping": 0,
        "is_qty_decimal": 0,
        "is_virtual": 0,
        "item_id": 5,
        "name": "Sprite Yoga Companion Kit",
        "no_discount": 0,
        "order_id": 3,
        "original_price": 68,
        "price": 68,
        "price_incl_tax": 68,
        "product_id": 51,
        "product_type": "bundle",
        "qty_canceled": 0,
        "qty_invoiced": 0,
        "qty_ordered": 1,
        "qty_refunded": 0,
        "qty_shipped": 0,
        "quote_item_id": 8,
        "row_invoiced": 0,
        "row_total": 68,
        "row_total_incl_tax": 68,
        "row_weight": 0,
        "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
        "store_id": 1,
        "tax_amount": 0,
        "tax_invoiced": 0,
        "updated_at": "2017-08-21 22:22:19",
        "weight": 0
      }
    },
    {
      "amount_refunded": 0,
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 5,
      "base_price": 5,
      "base_price_incl_tax": 5,
      "base_row_invoiced": 0,
      "base_row_total": 5,
      "base_row_total_incl_tax": 5,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 7,
      "name": "Sprite Foam Yoga Brick",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 5,
      "parent_item_id": 5,
      "price": 5,
      "price_incl_tax": 5,
      "product_id": 21,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 10,
      "row_invoiced": 0,
      "row_total": 5,
      "row_total_incl_tax": 5,
      "row_weight": 0,
      "sku": "24-WG084",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "parent_item": {
        "amount_refunded": 0,
        "applied_rule_ids": "1",
        "base_amount_refunded": 0,
        "base_discount_amount": 0,
        "base_discount_invoiced": 0,
        "base_original_price": 68,
        "base_price": 68,
        "base_price_incl_tax": 68,
        "base_row_invoiced": 0,
        "base_row_total": 68,
        "base_row_total_incl_tax": 68,
        "base_tax_amount": 0,
        "base_tax_invoiced": 0,
        "created_at": "2017-08-21 22:22:19",
        "discount_amount": 0,
        "discount_invoiced": 0,
        "discount_percent": 0,
        "free_shipping": 0,
        "is_qty_decimal": 0,
        "is_virtual": 0,
        "item_id": 5,
        "name": "Sprite Yoga Companion Kit",
        "no_discount": 0,
        "order_id": 3,
        "original_price": 68,
        "price": 68,
        "price_incl_tax": 68,
        "product_id": 51,
        "product_type": "bundle",
        "qty_canceled": 0,
        "qty_invoiced": 0,
        "qty_ordered": 1,
        "qty_refunded": 0,
        "qty_shipped": 0,
        "quote_item_id": 8,
        "row_invoiced": 0,
        "row_total": 68,
        "row_total_incl_tax": 68,
        "row_weight": 0,
        "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
        "store_id": 1,
        "tax_amount": 0,
        "tax_invoiced": 0,
        "updated_at": "2017-08-21 22:22:19",
        "weight": 0
      }
    },
    {
      "amount_refunded": 0,
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 17,
      "base_price": 17,
      "base_price_incl_tax": 17,
      "base_row_invoiced": 0,
      "base_row_total": 17,
      "base_row_total_incl_tax": 17,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 8,
      "name": "Sprite Yoga Strap 8 foot",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 17,
      "parent_item_id": 5,
      "price": 17,
      "price_incl_tax": 17,
      "product_id": 34,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 11,
      "row_invoiced": 0,
      "row_total": 17,
      "row_total_incl_tax": 17,
      "row_weight": 0,
      "sku": "24-WG086",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "parent_item": {
        "amount_refunded": 0,
        "applied_rule_ids": "1",
        "base_amount_refunded": 0,
        "base_discount_amount": 0,
        "base_discount_invoiced": 0,
        "base_original_price": 68,
        "base_price": 68,
        "base_price_incl_tax": 68,
        "base_row_invoiced": 0,
        "base_row_total": 68,
        "base_row_total_incl_tax": 68,
        "base_tax_amount": 0,
        "base_tax_invoiced": 0,
        "created_at": "2017-08-21 22:22:19",
        "discount_amount": 0,
        "discount_invoiced": 0,
        "discount_percent": 0,
        "free_shipping": 0,
        "is_qty_decimal": 0,
        "is_virtual": 0,
        "item_id": 5,
        "name": "Sprite Yoga Companion Kit",
        "no_discount": 0,
        "order_id": 3,
        "original_price": 68,
        "price": 68,
        "price_incl_tax": 68,
        "product_id": 51,
        "product_type": "bundle",
        "qty_canceled": 0,
        "qty_invoiced": 0,
        "qty_ordered": 1,
        "qty_refunded": 0,
        "qty_shipped": 0,
        "quote_item_id": 8,
        "row_invoiced": 0,
        "row_total": 68,
        "row_total_incl_tax": 68,
        "row_weight": 0,
        "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
        "store_id": 1,
        "tax_amount": 0,
        "tax_invoiced": 0,
        "updated_at": "2017-08-21 22:22:19",
        "weight": 0
      }
    },
    {
      "amount_refunded": 0,
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 19,
      "base_price": 19,
      "base_price_incl_tax": 19,
      "base_row_invoiced": 0,
      "base_row_total": 19,
      "base_row_total_incl_tax": 19,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 9,
      "name": "Sprite Foam Roller",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 19,
      "parent_item_id": 5,
      "price": 19,
      "price_incl_tax": 19,
      "product_id": 22,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 12,
      "row_invoiced": 0,
      "row_total": 19,
      "row_total_incl_tax": 19,
      "row_weight": 0,
      "sku": "24-WG088",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "parent_item": {
        "amount_refunded": 0,
        "applied_rule_ids": "1",
        "base_amount_refunded": 0,
        "base_discount_amount": 0,
        "base_discount_invoiced": 0,
        "base_original_price": 68,
        "base_price": 68,
        "base_price_incl_tax": 68,
        "base_row_invoiced": 0,
        "base_row_total": 68,
        "base_row_total_incl_tax": 68,
        "base_tax_amount": 0,
        "base_tax_invoiced": 0,
        "created_at": "2017-08-21 22:22:19",
        "discount_amount": 0,
        "discount_invoiced": 0,
        "discount_percent": 0,
        "free_shipping": 0,
        "is_qty_decimal": 0,
        "is_virtual": 0,
        "item_id": 5,
        "name": "Sprite Yoga Companion Kit",
        "no_discount": 0,
        "order_id": 3,
        "original_price": 68,
        "price": 68,
        "price_incl_tax": 68,
        "product_id": 51,
        "product_type": "bundle",
        "qty_canceled": 0,
        "qty_invoiced": 0,
        "qty_ordered": 1,
        "qty_refunded": 0,
        "qty_shipped": 0,
        "quote_item_id": 8,
        "row_invoiced": 0,
        "row_total": 68,
        "row_total_incl_tax": 68,
        "row_weight": 0,
        "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
        "store_id": 1,
        "tax_amount": 0,
        "tax_invoiced": 0,
        "updated_at": "2017-08-21 22:22:19",
        "weight": 0
      }
    },
    {
      "amount_refunded": 0,
      "applied_rule_ids": "1",
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_discount_tax_compensation_amount": 0,
      "base_original_price": 52,
      "base_price": 52,
      "base_price_incl_tax": 52,
      "base_row_invoiced": 0,
      "base_row_total": 52,
      "base_row_total_incl_tax": 52,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "discount_tax_compensation_amount": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 10,
      "name": "Chaz Kangeroo Hoodie",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 52,
      "price": 52,
      "price_incl_tax": 52,
      "product_id": 67,
      "product_type": "configurable",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 13,
      "row_invoiced": 0,
      "row_total": 52,
      "row_total_incl_tax": 52,
      "row_weight": 1,
      "sku": "MH01-S-Gray",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "weight": 1
    },
    {
      "amount_refunded": 0,
      "base_amount_refunded": 0,
      "base_discount_amount": 0,
      "base_discount_invoiced": 0,
      "base_price": 0,
      "base_row_invoiced": 0,
      "base_row_total": 0,
      "base_tax_amount": 0,
      "base_tax_invoiced": 0,
      "created_at": "2017-08-21 22:22:19",
      "discount_amount": 0,
      "discount_invoiced": 0,
      "discount_percent": 0,
      "free_shipping": 0,
      "is_qty_decimal": 0,
      "is_virtual": 0,
      "item_id": 11,
      "name": "Chaz Kangeroo Hoodie-S-Gray",
      "no_discount": 0,
      "order_id": 3,
      "original_price": 0,
      "parent_item_id": 10,
      "price": 0,
      "product_id": 56,
      "product_type": "simple",
      "qty_canceled": 0,
      "qty_invoiced": 0,
      "qty_ordered": 1,
      "qty_refunded": 0,
      "qty_shipped": 0,
      "quote_item_id": 14,
      "row_invoiced": 0,
      "row_total": 0,
      "row_weight": 0,
      "sku": "MH01-S-Gray",
      "store_id": 1,
      "tax_amount": 0,
      "tax_invoiced": 0,
      "tax_percent": 0,
      "updated_at": "2017-08-21 22:22:19",
      "weight": 1,
      "parent_item": {
        "amount_refunded": 0,
        "applied_rule_ids": "1",
        "base_amount_refunded": 0,
        "base_discount_amount": 0,
        "base_discount_invoiced": 0,
        "base_discount_tax_compensation_amount": 0,
        "base_original_price": 52,
        "base_price": 52,
        "base_price_incl_tax": 52,
        "base_row_invoiced": 0,
        "base_row_total": 52,
        "base_row_total_incl_tax": 52,
        "base_tax_amount": 0,
        "base_tax_invoiced": 0,
        "created_at": "2017-08-21 22:22:19",
        "discount_amount": 0,
        "discount_invoiced": 0,
        "discount_percent": 0,
        "free_shipping": 0,
        "discount_tax_compensation_amount": 0,
        "is_qty_decimal": 0,
        "is_virtual": 0,
        "item_id": 10,
        "name": "Chaz Kangeroo Hoodie",
        "no_discount": 0,
        "order_id": 3,
        "original_price": 52,
        "price": 52,
        "price_incl_tax": 52,
        "product_id": 67,
        "product_type": "configurable",
        "qty_canceled": 0,
        "qty_invoiced": 0,
        "qty_ordered": 1,
        "qty_refunded": 0,
        "qty_shipped": 0,
        "quote_item_id": 13,
        "row_invoiced": 0,
        "row_total": 52,
        "row_total_incl_tax": 52,
        "row_weight": 1,
        "sku": "MH01-S-Gray",
        "store_id": 1,
        "tax_amount": 0,
        "tax_invoiced": 0,
        "tax_percent": 0,
        "updated_at": "2017-08-21 22:22:19",
        "weight": 1
      }
    }
  ],
  "billing_address": {
    "address_type": "billing",
    "city": "Purchase",
    "country_id": "US",
    "email": "jdoe@example.com",
    "entity_id": 6,
    "firstname": "Jane",
    "lastname": "Doe",
    "parent_id": 3,
    "postcode": "10577",
    "region": "New York",
    "region_code": "NY",
    "region_id": 43,
    "street": [
      "123 Oak Ave"
    ],
    "telephone": "512-555-1111"
  },
  "payment": {
    "account_status": null,
    "additional_information": [
      "Bank Transfer Payment",
      ""
    ],
    "amount_ordered": 165,
    "base_amount_ordered": 165,
    "base_shipping_amount": 5,
    "cc_last4": null,
    "entity_id": 3,
    "method": "banktransfer",
    "parent_id": 3,
    "shipping_amount": 5
  },
  "status_histories": [],
  "extension_attributes": {
    "shipping_assignments": [
      {
        "shipping": {
          "address": {
            "address_type": "shipping",
            "city": "Purchase",
            "country_id": "US",
            "email": "jdoe@example.com",
            "entity_id": 5,
            "firstname": "Jane",
            "lastname": "Doe",
            "parent_id": 3,
            "postcode": "10577",
            "region": "New York",
            "region_code": "NY",
            "region_id": 43,
            "street": [
              "123 Oak Ave"
            ],
            "telephone": "512-555-1111"
          },
          "method": "tablerate_bestway",
          "total": {
            "base_shipping_amount": 5,
            "base_shipping_discount_amount": 0,
            "base_shipping_incl_tax": 5,
            "base_shipping_tax_amount": 0,
            "shipping_amount": 5,
            "shipping_discount_amount": 0,
            "shipping_discount_tax_compensation_amount": 0,
            "shipping_incl_tax": 5,
            "shipping_tax_amount": 0
          }
        },
        "items": [
          {
            "amount_refunded": 0,
            "applied_rule_ids": "1",
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 22,
            "base_price": 22,
            "base_price_incl_tax": 22,
            "base_row_invoiced": 0,
            "base_row_total": 22,
            "base_row_total_incl_tax": 22,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 3,
            "name": "Radiant Tee-M-Orange",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 22,
            "price": 22,
            "price_incl_tax": 22,
            "product_id": 1553,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 6,
            "row_invoiced": 0,
            "row_total": 22,
            "row_total_incl_tax": 22,
            "row_weight": 1,
            "sku": "WS12-M-Orange",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "weight": 1
          },
          {
            "amount_refunded": 0,
            "applied_rule_ids": "1",
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 18,
            "base_price": 18,
            "base_price_incl_tax": 18,
            "base_row_invoiced": 0,
            "base_row_total": 18,
            "base_row_total_incl_tax": 18,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 1,
            "item_id": 4,
            "name": "Advanced Pilates & Yoga (Strength)",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 18,
            "price": 18,
            "price_incl_tax": 18,
            "product_id": 49,
            "product_type": "downloadable",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 7,
            "row_invoiced": 0,
            "row_total": 18,
            "row_total_incl_tax": 18,
            "row_weight": 0,
            "sku": "240-LV08",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19"
          },
          {
            "amount_refunded": 0,
            "applied_rule_ids": "1",
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_original_price": 68,
            "base_price": 68,
            "base_price_incl_tax": 68,
            "base_row_invoiced": 0,
            "base_row_total": 68,
            "base_row_total_incl_tax": 68,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 5,
            "name": "Sprite Yoga Companion Kit",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 68,
            "price": 68,
            "price_incl_tax": 68,
            "product_id": 51,
            "product_type": "bundle",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 8,
            "row_invoiced": 0,
            "row_total": 68,
            "row_total_incl_tax": 68,
            "row_weight": 0,
            "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "updated_at": "2017-08-21 22:22:19",
            "weight": 0
          },
          {
            "amount_refunded": 0,
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 27,
            "base_price": 27,
            "base_price_incl_tax": 27,
            "base_row_invoiced": 0,
            "base_row_total": 27,
            "base_row_total_incl_tax": 27,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 6,
            "name": "Sprite Stasis Ball 65 cm",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 27,
            "parent_item_id": 5,
            "price": 27,
            "price_incl_tax": 27,
            "product_id": 29,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 9,
            "row_invoiced": 0,
            "row_total": 27,
            "row_total_incl_tax": 27,
            "row_weight": 0,
            "sku": "24-WG082-blue",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "parent_item": {
              "amount_refunded": 0,
              "applied_rule_ids": "1",
              "base_amount_refunded": 0,
              "base_discount_amount": 0,
              "base_discount_invoiced": 0,
              "base_original_price": 68,
              "base_price": 68,
              "base_price_incl_tax": 68,
              "base_row_invoiced": 0,
              "base_row_total": 68,
              "base_row_total_incl_tax": 68,
              "base_tax_amount": 0,
              "base_tax_invoiced": 0,
              "created_at": "2017-08-21 22:22:19",
              "discount_amount": 0,
              "discount_invoiced": 0,
              "discount_percent": 0,
              "free_shipping": 0,
              "is_qty_decimal": 0,
              "is_virtual": 0,
              "item_id": 5,
              "name": "Sprite Yoga Companion Kit",
              "no_discount": 0,
              "order_id": 3,
              "original_price": 68,
              "price": 68,
              "price_incl_tax": 68,
              "product_id": 51,
              "product_type": "bundle",
              "qty_canceled": 0,
              "qty_invoiced": 0,
              "qty_ordered": 1,
              "qty_refunded": 0,
              "qty_shipped": 0,
              "quote_item_id": 8,
              "row_invoiced": 0,
              "row_total": 68,
              "row_total_incl_tax": 68,
              "row_weight": 0,
              "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
              "store_id": 1,
              "tax_amount": 0,
              "tax_invoiced": 0,
              "updated_at": "2017-08-21 22:22:19",
              "weight": 0
            }
          },
          {
            "amount_refunded": 0,
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 5,
            "base_price": 5,
            "base_price_incl_tax": 5,
            "base_row_invoiced": 0,
            "base_row_total": 5,
            "base_row_total_incl_tax": 5,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 7,
            "name": "Sprite Foam Yoga Brick",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 5,
            "parent_item_id": 5,
            "price": 5,
            "price_incl_tax": 5,
            "product_id": 21,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 10,
            "row_invoiced": 0,
            "row_total": 5,
            "row_total_incl_tax": 5,
            "row_weight": 0,
            "sku": "24-WG084",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "parent_item": {
              "amount_refunded": 0,
              "applied_rule_ids": "1",
              "base_amount_refunded": 0,
              "base_discount_amount": 0,
              "base_discount_invoiced": 0,
              "base_original_price": 68,
              "base_price": 68,
              "base_price_incl_tax": 68,
              "base_row_invoiced": 0,
              "base_row_total": 68,
              "base_row_total_incl_tax": 68,
              "base_tax_amount": 0,
              "base_tax_invoiced": 0,
              "created_at": "2017-08-21 22:22:19",
              "discount_amount": 0,
              "discount_invoiced": 0,
              "discount_percent": 0,
              "free_shipping": 0,
              "is_qty_decimal": 0,
              "is_virtual": 0,
              "item_id": 5,
              "name": "Sprite Yoga Companion Kit",
              "no_discount": 0,
              "order_id": 3,
              "original_price": 68,
              "price": 68,
              "price_incl_tax": 68,
              "product_id": 51,
              "product_type": "bundle",
              "qty_canceled": 0,
              "qty_invoiced": 0,
              "qty_ordered": 1,
              "qty_refunded": 0,
              "qty_shipped": 0,
              "quote_item_id": 8,
              "row_invoiced": 0,
              "row_total": 68,
              "row_total_incl_tax": 68,
              "row_weight": 0,
              "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
              "store_id": 1,
              "tax_amount": 0,
              "tax_invoiced": 0,
              "updated_at": "2017-08-21 22:22:19",
              "weight": 0
            }
          },
          {
            "amount_refunded": 0,
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 17,
            "base_price": 17,
            "base_price_incl_tax": 17,
            "base_row_invoiced": 0,
            "base_row_total": 17,
            "base_row_total_incl_tax": 17,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 8,
            "name": "Sprite Yoga Strap 8 foot",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 17,
            "parent_item_id": 5,
            "price": 17,
            "price_incl_tax": 17,
            "product_id": 34,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 11,
            "row_invoiced": 0,
            "row_total": 17,
            "row_total_incl_tax": 17,
            "row_weight": 0,
            "sku": "24-WG086",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "parent_item": {
              "amount_refunded": 0,
              "applied_rule_ids": "1",
              "base_amount_refunded": 0,
              "base_discount_amount": 0,
              "base_discount_invoiced": 0,
              "base_original_price": 68,
              "base_price": 68,
              "base_price_incl_tax": 68,
              "base_row_invoiced": 0,
              "base_row_total": 68,
              "base_row_total_incl_tax": 68,
              "base_tax_amount": 0,
              "base_tax_invoiced": 0,
              "created_at": "2017-08-21 22:22:19",
              "discount_amount": 0,
              "discount_invoiced": 0,
              "discount_percent": 0,
              "free_shipping": 0,
              "is_qty_decimal": 0,
              "is_virtual": 0,
              "item_id": 5,
              "name": "Sprite Yoga Companion Kit",
              "no_discount": 0,
              "order_id": 3,
              "original_price": 68,
              "price": 68,
              "price_incl_tax": 68,
              "product_id": 51,
              "product_type": "bundle",
              "qty_canceled": 0,
              "qty_invoiced": 0,
              "qty_ordered": 1,
              "qty_refunded": 0,
              "qty_shipped": 0,
              "quote_item_id": 8,
              "row_invoiced": 0,
              "row_total": 68,
              "row_total_incl_tax": 68,
              "row_weight": 0,
              "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
              "store_id": 1,
              "tax_amount": 0,
              "tax_invoiced": 0,
              "updated_at": "2017-08-21 22:22:19",
              "weight": 0
            }
          },
          {
            "amount_refunded": 0,
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 19,
            "base_price": 19,
            "base_price_incl_tax": 19,
            "base_row_invoiced": 0,
            "base_row_total": 19,
            "base_row_total_incl_tax": 19,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 9,
            "name": "Sprite Foam Roller",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 19,
            "parent_item_id": 5,
            "price": 19,
            "price_incl_tax": 19,
            "product_id": 22,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 12,
            "row_invoiced": 0,
            "row_total": 19,
            "row_total_incl_tax": 19,
            "row_weight": 0,
            "sku": "24-WG088",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "parent_item": {
              "amount_refunded": 0,
              "applied_rule_ids": "1",
              "base_amount_refunded": 0,
              "base_discount_amount": 0,
              "base_discount_invoiced": 0,
              "base_original_price": 68,
              "base_price": 68,
              "base_price_incl_tax": 68,
              "base_row_invoiced": 0,
              "base_row_total": 68,
              "base_row_total_incl_tax": 68,
              "base_tax_amount": 0,
              "base_tax_invoiced": 0,
              "created_at": "2017-08-21 22:22:19",
              "discount_amount": 0,
              "discount_invoiced": 0,
              "discount_percent": 0,
              "free_shipping": 0,
              "is_qty_decimal": 0,
              "is_virtual": 0,
              "item_id": 5,
              "name": "Sprite Yoga Companion Kit",
              "no_discount": 0,
              "order_id": 3,
              "original_price": 68,
              "price": 68,
              "price_incl_tax": 68,
              "product_id": 51,
              "product_type": "bundle",
              "qty_canceled": 0,
              "qty_invoiced": 0,
              "qty_ordered": 1,
              "qty_refunded": 0,
              "qty_shipped": 0,
              "quote_item_id": 8,
              "row_invoiced": 0,
              "row_total": 68,
              "row_total_incl_tax": 68,
              "row_weight": 0,
              "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
              "store_id": 1,
              "tax_amount": 0,
              "tax_invoiced": 0,
              "updated_at": "2017-08-21 22:22:19",
              "weight": 0
            }
          },
          {
            "amount_refunded": 0,
            "applied_rule_ids": "1",
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_discount_tax_compensation_amount": 0,
            "base_original_price": 52,
            "base_price": 52,
            "base_price_incl_tax": 52,
            "base_row_invoiced": 0,
            "base_row_total": 52,
            "base_row_total_incl_tax": 52,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "discount_tax_compensation_amount": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 10,
            "name": "Chaz Kangeroo Hoodie",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 52,
            "price": 52,
            "price_incl_tax": 52,
            "product_id": 67,
            "product_type": "configurable",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 13,
            "row_invoiced": 0,
            "row_total": 52,
            "row_total_incl_tax": 52,
            "row_weight": 1,
            "sku": "MH01-S-Gray",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "weight": 1
          },
          {
            "amount_refunded": 0,
            "base_amount_refunded": 0,
            "base_discount_amount": 0,
            "base_discount_invoiced": 0,
            "base_price": 0,
            "base_row_invoiced": 0,
            "base_row_total": 0,
            "base_tax_amount": 0,
            "base_tax_invoiced": 0,
            "created_at": "2017-08-21 22:22:19",
            "discount_amount": 0,
            "discount_invoiced": 0,
            "discount_percent": 0,
            "free_shipping": 0,
            "is_qty_decimal": 0,
            "is_virtual": 0,
            "item_id": 11,
            "name": "Chaz Kangeroo Hoodie-S-Gray",
            "no_discount": 0,
            "order_id": 3,
            "original_price": 0,
            "parent_item_id": 10,
            "price": 0,
            "product_id": 56,
            "product_type": "simple",
            "qty_canceled": 0,
            "qty_invoiced": 0,
            "qty_ordered": 1,
            "qty_refunded": 0,
            "qty_shipped": 0,
            "quote_item_id": 14,
            "row_invoiced": 0,
            "row_total": 0,
            "row_weight": 0,
            "sku": "MH01-S-Gray",
            "store_id": 1,
            "tax_amount": 0,
            "tax_invoiced": 0,
            "tax_percent": 0,
            "updated_at": "2017-08-21 22:22:19",
            "weight": 1,
            "parent_item": {
              "amount_refunded": 0,
              "applied_rule_ids": "1",
              "base_amount_refunded": 0,
              "base_discount_amount": 0,
              "base_discount_invoiced": 0,
              "base_discount_tax_compensation_amount": 0,
              "base_original_price": 52,
              "base_price": 52,
              "base_price_incl_tax": 52,
              "base_row_invoiced": 0,
              "base_row_total": 52,
              "base_row_total_incl_tax": 52,
              "base_tax_amount": 0,
              "base_tax_invoiced": 0,
              "created_at": "2017-08-21 22:22:19",
              "discount_amount": 0,
              "discount_invoiced": 0,
              "discount_percent": 0,
              "free_shipping": 0,
              "discount_tax_compensation_amount": 0,
              "is_qty_decimal": 0,
              "is_virtual": 0,
              "item_id": 10,
              "name": "Chaz Kangeroo Hoodie",
              "no_discount": 0,
              "order_id": 3,
              "original_price": 52,
              "price": 52,
              "price_incl_tax": 52,
              "product_id": 67,
              "product_type": "configurable",
              "qty_canceled": 0,
              "qty_invoiced": 0,
              "qty_ordered": 1,
              "qty_refunded": 0,
              "qty_shipped": 0,
              "quote_item_id": 13,
              "row_invoiced": 0,
              "row_total": 52,
              "row_total_incl_tax": 52,
              "row_weight": 1,
              "sku": "MH01-S-Gray",
              "store_id": 1,
              "tax_amount": 0,
              "tax_invoiced": 0,
              "tax_percent": 0,
              "updated_at": "2017-08-21 22:22:19",
              "weight": 1
            }
          }
        ]
      }
    ]
  }
}
```

### Create an invoice: Capture payment

You create an invoice after you receive payment for an order. An invoice is structurally similar to an order, but an order contains more details.

This example creates a full invoice. You can create a partial invoice by adding to the payload an array of items to be invoiced.

In this example, the order was paid offline via a bank transfer. Therefore, you must tell Magento that payment for the order has been captured.

You will use the `order_item_id` values to create a shipment later.

You can check it with the following step:
- Log in to Admin. Click Sales > Invoices. The invoice is displayed in the grid. The status is Paid. Then click Sales > Orders. The status is Processing.

#### Capture payment

**Endpoint**

`POST <host>/rest/<store_code>/V1/order/3/invoice`

where 3 is the `orderid`.

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <administrator token>`

**Payload**

```json
{
  "capture": true,
  "notify": true
}
```

**Response**

An invoice `id`, such as 3.

#### View the invoice

**Endpoint**

`GET <host>/rest/<store_code>/V1/invoices/3`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <administrator token>`

**Payload**

None

**Response**

```json
{
  "base_currency_code": "USD",
  "base_discount_amount": 0,
  "base_grand_total": 165,
  "base_discount_tax_compensation_amount": 0,
  "base_shipping_amount": 5,
  "base_shipping_incl_tax": 5,
  "base_shipping_tax_amount": 0,
  "base_subtotal": 160,
  "base_subtotal_incl_tax": 160,
  "base_tax_amount": 0,
  "base_to_global_rate": 1,
  "base_to_order_rate": 1,
  "billing_address_id": 6,
  "can_void_flag": 0,
  "created_at": "2017-08-21 22:36:02",
  "discount_amount": 0,
  "email_sent": 1,
  "entity_id": 3,
  "global_currency_code": "USD",
  "grand_total": 165,
  "discount_tax_compensation_amount": 0,
  "increment_id": "000000003",
  "order_currency_code": "USD",
  "order_id": 3,
  "shipping_address_id": 5,
  "shipping_amount": 5,
  "shipping_discount_tax_compensation_amount": 0,
  "shipping_incl_tax": 5,
  "shipping_tax_amount": 0,
  "state": 2,
  "store_currency_code": "USD",
  "store_id": 1,
  "store_to_base_rate": 0,
  "store_to_order_rate": 0,
  "subtotal": 160,
  "subtotal_incl_tax": 160,
  "tax_amount": 0,
  "total_qty": 9,
  "updated_at": "2017-08-21 22:36:03",
  "items": [
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 22,
      "base_price_incl_tax": 22,
      "base_row_total": 22,
      "base_row_total_incl_tax": 22,
      "base_tax_amount": 0,
      "entity_id": 3,
      "discount_tax_compensation_amount": 0,
      "name": "Radiant Tee-M-Orange",
      "parent_id": 3,
      "price": 22,
      "price_incl_tax": 22,
      "product_id": 1553,
      "row_total": 22,
      "row_total_incl_tax": 22,
      "sku": "WS12-M-Orange",
      "tax_amount": 0,
      "order_item_id": 3,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 18,
      "base_price_incl_tax": 18,
      "base_row_total": 18,
      "base_row_total_incl_tax": 18,
      "base_tax_amount": 0,
      "entity_id": 4,
      "discount_tax_compensation_amount": 0,
      "name": "Advanced Pilates & Yoga (Strength)",
      "parent_id": 3,
      "price": 18,
      "price_incl_tax": 18,
      "product_id": 49,
      "row_total": 18,
      "row_total_incl_tax": 18,
      "sku": "240-LV08",
      "tax_amount": 0,
      "order_item_id": 4,
      "qty": 1
    },
    {
      "base_price": 68,
      "base_price_incl_tax": 68,
      "entity_id": 5,
      "name": "Sprite Yoga Companion Kit",
      "parent_id": 3,
      "price": 68,
      "price_incl_tax": 68,
      "product_id": 51,
      "sku": "24-WG080-24-WG084-24-WG088-24-WG082-blue-24-WG086",
      "order_item_id": 5,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 27,
      "base_price_incl_tax": 27,
      "base_row_total": 27,
      "base_row_total_incl_tax": 27,
      "base_tax_amount": 0,
      "entity_id": 6,
      "discount_tax_compensation_amount": 0,
      "name": "Sprite Stasis Ball 65 cm",
      "parent_id": 3,
      "price": 27,
      "price_incl_tax": 27,
      "product_id": 29,
      "row_total": 27,
      "row_total_incl_tax": 27,
      "sku": "24-WG082-blue",
      "tax_amount": 0,
      "order_item_id": 6,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 5,
      "base_price_incl_tax": 5,
      "base_row_total": 5,
      "base_row_total_incl_tax": 5,
      "base_tax_amount": 0,
      "entity_id": 7,
      "discount_tax_compensation_amount": 0,
      "name": "Sprite Foam Yoga Brick",
      "parent_id": 3,
      "price": 5,
      "price_incl_tax": 5,
      "product_id": 21,
      "row_total": 5,
      "row_total_incl_tax": 5,
      "sku": "24-WG084",
      "tax_amount": 0,
      "order_item_id": 7,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 17,
      "base_price_incl_tax": 17,
      "base_row_total": 17,
      "base_row_total_incl_tax": 17,
      "base_tax_amount": 0,
      "entity_id": 8,
      "discount_tax_compensation_amount": 0,
      "name": "Sprite Yoga Strap 8 foot",
      "parent_id": 3,
      "price": 17,
      "price_incl_tax": 17,
      "product_id": 34,
      "row_total": 17,
      "row_total_incl_tax": 17,
      "sku": "24-WG086",
      "tax_amount": 0,
      "order_item_id": 8,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 19,
      "base_price_incl_tax": 19,
      "base_row_total": 19,
      "base_row_total_incl_tax": 19,
      "base_tax_amount": 0,
      "entity_id": 9,
      "discount_tax_compensation_amount": 0,
      "name": "Sprite Foam Roller",
      "parent_id": 3,
      "price": 19,
      "price_incl_tax": 19,
      "product_id": 22,
      "row_total": 19,
      "row_total_incl_tax": 19,
      "sku": "24-WG088",
      "tax_amount": 0,
      "order_item_id": 9,
      "qty": 1
    },
    {
      "base_discount_tax_compensation_amount": 0,
      "base_price": 52,
      "base_price_incl_tax": 52,
      "base_row_total": 52,
      "base_row_total_incl_tax": 52,
      "base_tax_amount": 0,
      "entity_id": 10,
      "discount_tax_compensation_amount": 0,
      "name": "Chaz Kangeroo Hoodie",
      "parent_id": 3,
      "price": 52,
      "price_incl_tax": 52,
      "product_id": 67,
      "row_total": 52,
      "row_total_incl_tax": 52,
      "sku": "MH01-S-Gray",
      "tax_amount": 0,
      "order_item_id": 10,
      "qty": 1
    },
    {
      "base_price": 0,
      "entity_id": 11,
      "name": "Chaz Kangeroo Hoodie-S-Gray",
      "parent_id": 3,
      "price": 0,
      "product_id": 56,
      "sku": "MH01-S-Gray",
      "order_item_id": 11,
      "qty": 1
    }
  ],
  "comments": []
}
```

### Create a shipment

To create a shipment, you need the `order_item_id` of each item to be shipped. To create a partial shipment, specify only those order_item_ids that are to be shipped now.

If the call is successful on a full shipment, Magento changes the status of an order to Complete.

Since the "Sprite Yoga Companion Kit" is a bundle item, you only need to include the top-level `order_item_id` (5). The `order_item_id` for the Radiant Tee-M-Orange is 3.

You can check it with the following step:
- Log in to Admin. Click Sales > Shipments. The shipment is displayed in the grid. Then click Sales > Orders. The order status is Complete.

**Endpoint**

`POST <host>/rest/<store_code>/V1/order/3/ship`

where 3 is the order id.

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <administrator token>`

**Payload**

The `notify` field is used to trigger Magento to send the shipment email. The `tracks` array optionally allows you to include one or more tracking numbers for the shipment.

```json
{
  "items": [
    {
      "order_item_id": 3,
      "qty": 1
    },
    {
      "order_item_id": 5,
      "qty": 1
    },
    {
      "order_item_id": 11,
      "qty": 1
    }
  ],
  "notify": true,
  "tracks": [
    {
      "track_number": "1Y-9876543210",
      "title": "United Parcel Service",
      "carrier_code": "ups"
    }
  ]
}
```

**Response**

A shipment ID, such as 3.

### Issue a partial refund

Magento 2.1.3 introduced two endpoints that streamline the process of issuing a refund by creating a creditmemo and updating the order or invoice in one call.

| ENDPOINT                               | DESCRIPTION                                  |
|----------------------------------------|----------------------------------------------|
| `POST /V1/order/<order_ID>/refund`     | Issues an offline refund                     |
| `POST /V1/invoice/<invoice_ID>/refund` | Issue a refund with an online payment system |

In this example, the customer did not like the fit of the Radiant T-M-Orange shirt and wants a refund.

Since the customer paid for the order with a bank transfer, we’ll call `POST /V1/order/<order ID>/refund`. The `order_item_id` for the Radiant Tee-M-Orange is 3.

The `arguments` object allows you to adjust the amount of the credit to be refunded. Since the customer used the `tablerate` shipping method, which applied to the whole order, we’ll assume that a refund can’t be applied to the shipping costs. Therefore, the shipping_amount is set to 0.

If the customer had selected the `flatrate` shipping method ($5 per item), we would set the value of `shipping_amount` to 5.

The `return_to_stock_items` array specifies which `order_item_ids` can be returned to stock and be resold.

You can check it with the following step:
- Log in to Admin. Click Sales > Credit Memos. The credit memo is displayed in the grid.

**Endpoint**

`POST <host>/rest/<store_code>/V1/order/5/refund`

**Headers**

`Content-Type: application/json`

`Authorization: Bearer <administrator token>`

**Payload**

```json
{
  "items": [
    {
      "order_item_id": 3,
      "qty": 1
    }
  ],
  "notify": true,
  "arguments": {
    "shipping_amount": 0,
    "adjustment_positive": 0,
    "adjustment_negative": 0,
    "extension_attributes": {
      "return_to_stock_items": [
        3
      ]
    }
  }
}
```

**Response**

A credit memo id, such as 3.
