
*You can download the complete source code of this tutorial [here](https://www.noodl.io/market/product/P201604181926406/noodlio-pay-smooth-payments-with-stripe-accept-payments-without-a-server-side-setup)*

# The easy way to integrate Stripe payments in your app, website or elsewhere

Whether you're building a marketplace, mobile app, online storefront, or subscription service, Stripe has the features you need to quickly get started. It is therefore not a surprise it is heavily used by Lyft, TaskRabbit, Instacart, and many other well-known companies. One the main advantages of this payment gateway is that it's built for developers with the eye on easy integration. This is an important differentiator; taking away development time on implementing a payment infrastructure helps developers focus on the more important aspects of their applications.

# Challenges

Nevertheless, integrating payments remains having its challenges and drawbacks. One of the major issues that is faced by the community (see [threads](http://stackoverflow.com/search?q=stripe) on StackOverflow) is the integration of the payment server. Despite the very elaborate documentation of Stripe, there are still a lot of unanswered questions, especially if you have never implemented a server before. Also, once you have successfully achieved to integrate a server, you'll need to make a link with your client side and afer that find a way to run it 24/7 on a VPS like Heroku, Cloud9, etc. This might be an costly undertaking as you will see yourself easily paying too much for unused server capacity.

# Solution

In this tutorial I am going to introduce you a new way to integrate Stripe payments, which takes away all the drawbacks that we just mentioned. We'll be leveraging the new but amazing [Stripe Payments API](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe) (also called Noodlio Pay) to replace our server-side. As you will see, this will save you a lot of precious time to learn a new server language, test, validate and so.

# Benefits of this approach

- **It's quick**: You can have a working payment server set up within a few minutes.
- **No server-side setup needed**: Simply send `HTTP POST` requests to the Noodlio Pay API from the client-side and the rest will be taken care of.
- **Cost efficient**: Hosting a complete server-side 24/7 can be a costly undertaking. This won't be a worry anymore: because the Noodlio Pay server is already hosted for you, you don't have to spend money on unused server capacity.
- **Instantaneous**: Thanks to the Stripe setup, you'll see the funds transferred immediately to your Stripe Account
- **Unlimited**: There are no restrictions on the number of requests that you can send through the Noodlio Pay server.
- **Broad support**: You can charge your customers with any client-side language through `HTTP POST` requests (for instance `Angular`, `React`, `Javascript`, etc.). In addition, there is support for dedicated server-languages such as `CURL`, `Java`, `NodeJS`, `PHP`, `Python`, `Objective-C`, `Ruby` and `.NET`.
- **Tested, pre-configured and maintained**: The team is constantly monitoring, testing and updating the server to conform to the latest developments.
- **Secure**: The servers are secure and they never store any transaction data.

# How it works

## 0. Stripe and Mashape Setup

We first need to define a couple of constants in our app. If you are working with Angular/Ionic `v1.x`, head over to `app.js` and copy and paste the following:

```
// Stripe Payments API
// Obtain from:
// - https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe
var NOODLIO_PAY_API_URL         = "https://noodlio-pay.p.mashape.com";
var NOODLIO_PAY_API_KEY         = "<YOUR-MASHAPE-API-KEY>";

// Stripe Account
// Connect on both:
// - https://www.noodl.io/pay/connect and
// - https://www.noodl.io/pay/connect/test
var STRIPE_ACCOUNT_ID           = "<YOUR-STRIPE-ACCOUNT-ID>"

// Define whether you are in development mode (TEST_MODE: true) or production mode (TEST_MODE: false)
var TEST_MODE = false;
```

The `NOODLIO_PAY_API_URL` is basically the location of the server and is fixed. The variable `TEST_MODE` simply takes the values `true` or `false` and defines whether we are in test mode (development) or production (actually charging the user). Now let's define two constants:

**Mashape**

To consume the Stripe Payments API, we'll need to obtain our unique `NOODLIO_PAY_API_KEY`. To do so, head over to [Mashape](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe) and click on the right "Get your API Keys and Start Hacking" or press on "Sign up free".

[<img src="http://noodlio-templates.firebaseapp.com/noodlio-pay/img/mashape-api-keys.png">](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe)

After you are signed in, you'll find your unique API Key in the request example on the [Stripe Payments API page](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe):

```
curl -X POST --include 'https://noodlio-pay.p.mashape.com/charge/token' \
  -H 'X-Mashape-Key: <YOUR-MASHAPE-API-KEY>' \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Accept: application/json' \
  ... other values
```

Replace the `NOODLIO_PAY_API_KEY` with this unique identifier.

**Stripe Account**

If you haven't already [sign up for a Stripe Account](https://www.stripe.com). After that, you'll need to retrieve your unique Stripe Account ID (field: `stripe_account`), which you can obtain on the following pages (Note: you'll need to visit both links once):

- For the production mode:
[https://www.noodl.io/pay/connect](https://www.noodl.io/pay/connect)
- For the development mode:
[https://www.noodl.io/pay/connect/test](https://www.noodl.io/pay/connect/test)

The Stripe Account ID looks something like `acct_12abcDEF34GhIJ5K`. Replace the constant `STRIPE_ACCOUNT_ID` wherever you have defined it.

That's it. Our server is configured and ready to receive payments.

## 1. Obtain the Stripe token (`source`)

Now we can head over to the fun part and start integrating the payments in our application. To do so, we'll first need to obtain the crucial parameter `source`. This parameter can be either a token (which we will obtain in this exercise) or a customer ID (mostly used for recurring payments which we will discuss in another tutorial). As we will be charging the client's credit card, we'll therefore need to obtain the token. Note that when obtaining the token, the server also validates the credit card input of the user.

There are two options to obtain a Stripe token:

### Option 1: Use the Stripe Payments API (Noodlio Pay)

This corresponds to sending a `HTTP POST` request with the credit card input (`number`, `cvc`, `exp_month` and `exp_year`) to the route [`/tokens/create`](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe#tokens-create) of the [Stripe Payments API (Noodlio Pay)](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe). For that we'll need to implement a custom form in which your customer can add their credit card details. The form data is subsequently send to the Stripe Payments API and returns the token upon success.

**Add the HTML form**

Let's start with integrating the form with the credit card details. In Angular `v1.x` we can do this as follows:

```
<div ng-controller="ExampleCtrl">
  <!-- html form -->
  <label class="item item-input">
    <span class="input-label">Card number</span>
    <input type="text" size="20" ng-model="FormData.number" placeholder="4242 4242 4242 4242"/>
  </label>

  <label class="item item-input">
    <span class="input-label">Exp. Mth (MM)</span>
    <input type="text" size="2" ng-model="FormData.exp_month"/ placeholder="01">
  </label>

  <label class="item item-input">
    <span class="input-label">Exp. Year (YYYY)</span>
    <input type="text" size="4" ng-model="FormData.exp_year"/ placeholder="2020">
  </label>

  <label class="item item-input">
    <span class="input-label">Sec. Code (CVC)</span>
    <input type="text" size="4" ng-model="FormData.cvc"/ placeholder="123">
  </label>
</div>
```

and a Submit button that is linked to our controller `ExampleCtrl` through the function `charge()`:

```
<div class="padding center">
  <button class="button button-block button-balanced" ng-click="charge()">Submit</button>
</div>
```

**Extend the `HTTP` headers**

To send the FormData to the Stripe Payments API, we'll need to make a HTTP request to the `NOODLIO_PAY_API_URL`, which in Angular `v1.x` can be achieved with the dependency `$http`. We define our controller therefore as follows:

```
.controller('ExampleCtrl', ['$scope', '$http', function($scope, $http) {

  // ...

}]);
```
*Note: for best practices you should do this in a factory or service*

Since it is hosted through Mashape, we'll also need to add authentication headers in our request. We can do this by using the `NOODLIO_PAY_API_KEY` and including it in the headers of the `$http` method. If you look at the [documentation of the route `charge/token`](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe#charge-token), the first part of the cURL request looks something like this:

```
curl -X POST --include 'https://noodlio-pay.p.mashape.com/charge/token'
  -H 'X-Mashape-Key: 3fEagjJCGAmshMqVnwTR70bVqG3yp1lerJNjsnTzx5ODeOa99V'
  -H 'Content-Type: application/x-www-form-urlencoded'
  -H 'Accept: application/json'
  ... other variables
```
From that we can see that in order to send requests to the API, we'll need to include the headers `X-Mashape-Key`, `Content-Type` and `Accept`. In Angular `v1.x` this translates to:

```
.controller('ExampleCtrl', ['$scope', '$http', function($scope, $http) {

  // add the following headers for authentication
   $http.defaults.headers.common['X-Mashape-Key']  = NOODLIO_PAY_API_KEY;
   $http.defaults.headers.common['Content-Type']   = 'application/x-www-form-urlencoded';
   $http.defaults.headers.common['Accept']         = 'application/json';


}]);
```

**Obtain the token**

Now that our headers are configured, we can continue by implementing the function to obtain the token for our `source` parameter. Here is the example code that we used to obtain the Stripe token:

```
// part of ExampleCtrl

$scope.FormData = {
    number: "4242424242424242",
    cvc: "256",
    exp_month: "08",
    exp_year: "2018",
    test: TEST_MODE,
  };

$scope.charge = function() {

  // init for the DOM
  $scope.ResponseData = {
    loading: true
  };

  // create a token and validate the credit card details
  $http.post(NOODLIO_PAY_API_URL + "/tokens/create", $scope.FormData)
  .success(
    function(response){

      if(response.hasOwnProperty('id')) {
        var token = response.id; $scope.ResponseData['token'] = token;

        // --> success, proceed with charging the user
        proceedCharge(token);
      } else {
        $scope.ResponseData['token'] = 'Error, see console';
        $scope.ResponseData['loading'] = false;
      };

    }
  )
  .error(
    function(response){
      $scope.ResponseData['token'] = 'Error, see console';
      $scope.ResponseData['loading'] = false;
    }
  );
};

```

As you can see from this example, we're sending a `HTTP POST` request to the url `NOODLIO_PAY_API_URL + "/tokens/create"` with the form encoded parameters corresponding to the credit card details of the user. When the response has a property `id`, i.e. a token (`source`), then we continue with charging the user through the function `proceedCharge(token)` (which we discuss below). We bind the variable `ResponseData` to the `$scope` to update the user through the DOM on the status of the payment (this is optional). You can see how a successful response looks like in the [official documentation](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe#tokens-create) (on the right under RESPONSE BODY).

### Option 2: Use Stripe's native Checkout form

The Checkout form is an embeddable payment form for desktop, tablet, and mobile devices. It works within your site: customers can pay instantly, without being redirected to complete the transaction.

The integration of the checkout form is to be discussed in another tutorial. If you would to integrate it yourself already, you can obtain an example source code or read guides/tutorials on how to embed the Checkout form in your application in the following languages: [Ionic/Angular](https://github.com/noodlio/noodlio-pay-ionic-example), [Sinatra](https://stripe.com/docs/checkout/sinatra), [Rails](https://stripe.com/docs/checkout/rails), [Flask](https://stripe.com/docs/checkout/flask), and [PHP](https://stripe.com/docs/checkout/php)

## 2. Charge the client

Now that you have obtained the token (`source`) from Step 1, let's proceed with charging the user. We can do so by sending another `HTTP POST` request to another route of the Stripe Payments API: [`charge/token`](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe#charge-token). As we can read from the docs, the form encode parameters are in this case `amount`, `description` (optional), `currency` and `stripe_account`. With the integration in Step 1 behind us, this is now a piece of cake and corresponds to:

```
// part of ExampleCtrl
// includes also the header extensions at the top

// charge the customer with the token from Step 1
function proceedCharge(token) {

  var param = {
    source: token,
    amount: 100, // amount in cents
    currency: "usd",
    description: "Your custom description here",
    stripe_account: STRIPE_ACCOUNT_ID,
    test: TEST_MODE,
  };

  $http.post(NOODLIO_PAY_API_URL + "/charge/token", param)
  .success(
    function(response){

      // --> success
      $scope.ResponseData['loading'] = false;

      if(response.hasOwnProperty('id')) {
        // success, check your Stripe account
        var paymentId = response.id; $scope.ResponseData['paymentId'] = paymentId;
      } else {
        $scope.ResponseData['paymentId'] = 'Error, see console';
      };

    }
  )
  .error(
    function(response){
      $scope.ResponseData['paymentId'] = 'Error, see console';
      $scope.ResponseData['loading'] = false;
    }
  );
};
```

The setup is similar and if the response has an `id` then this corresponds to the `paymentId` (store it in your database for your reference) which equals to a successful charge. This also means that you should have received the funds on your Stripe Account, visible in your [Dashboard](https://dashboard.stripe.com/dashboard). Don't forget to define the `amount` in cents. The parameter `stripe_account` thus indicates to which Stripe Account the funds should be transferred. Make sure you have included the right ID (see Step 0).
You can see how a successful response looks like in the [official documentation](https://market.mashape.com/noodlio/noodlio-pay-smooth-payments-with-stripe#charge-token) (on the right under RESPONSE BODY).

# Wrapping up

That's it, you have successfully integrated Stripe payments in your work. As you see, it was actually a piece of cake and the setup was quite short. Now you can focus on the more important aspects of your app and actually start making money from it.

If you have question, comments or some suggestions, please let us know. We love to hear from you.

# Source Code

You can download the complete source code from [this repository](https://www.noodl.io/market/product/P201604181926406/noodlio-pay-smooth-payments-with-stripe-accept-payments-without-a-server-side-setup). It also includes an example with Stripe Checkout. While the examples are made in AngularJS in an Ionic App, it is easy to reuse most of the code in other projects (whether native or hybrid)

If you have implemented your own template/starter/tutorial with the Stripe Payments API (Noodlio Pay) and would like to list it here, please [let the owner know](mailto:noodlio@seipel-ibisevic.com) and you can receive an award.

# Pricing

The use of the API hosted on Mashape is free and you can make unlimited requests. [**Click here for an overview of complementary licenses**](https://www.noodl.io/pay/plans)
