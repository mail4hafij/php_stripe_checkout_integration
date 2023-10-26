# php_stripe_checkout_integration
The following PHP code snippet shows a simple checkout session using stripe. 

#### Install stripe with composer.
```
composer require stripe/stripe-php
```

#### Code snippet (Make adjustment to your needs)
The following code is a basic example of stripe checkout.

```
public function checkout()
{
  // receives a post request 
  $ocr_num = $_POST['ocr_num'];
  
  // maintain a unique session and save it in the database
  // so that after successful checkout we can find out the session.
  $checkout_session = md5(uniqid(rand(), true));

  // auto loading the required stripe classes
  require_once 'vendor/autoload.php';
  
  // the stripe client
  $stripe = new \Stripe\StripeClient("YOUR_STRIPE_SECRET_GOES_HERE");
  
  // create a sripe customer
  $stripe_customer = $stripe->customers->create([
    'phone' => "CUSTOMER_MOBILE_NUMBER",
  ]);
  
  // save the stripe customer id in the database
  // if needed for reuse purposes
  $stripe_customer_id = $stripe_customer->id;
  
  // checkout
  $checkout = $stripe->checkout->sessions->create([
    'line_items' => [[
      'price_data' => [
        'currency' => 'sek',
        'unit_amount_decimal' => 123,
        'product_data' => [
          'name' => $ocr_num
        ]
      ],
      'quantity' => 1,
    ]],
    'customer' => $stripe_customer_id,
    'mode' => 'payment',
    'success_url' => "YOUR_WEBSITE.COM/YOUR_CONTROLLER/success/$checkout_session",
    'cancel_url' => "YOUR_WEBSITE.COM/YOUR_CONTROLLER/cancel/$checkout_session",
  ]);
  
  header('Content-Type: application/json');
  header("HTTP/1.1 303 See Other");
  header("Location: " . $checkout_session->url);
}

public function success($checkout_session)
{
  // find out the invoice by this checkout session
  // and set it paid.
}

public funciton cancel($checkout_session)
{
  // find out the invoice by this checkout session
  // and set it canceled.
}
```
