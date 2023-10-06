See Documentation here - https://aamarpay.readme.io/reference/initiate-payment-json

### # CSRF Token Error / 419
In Laravel, you can handle CSRF Token Errors (status code 419) by customizing the "VerifyCsrfToken middleware", which is located in the "app/Http/Middleware/VerifyCsrfToken.php" file. To address this error, you need to declare the success, fail, and cancel URLs/routes in the middleware.


```
<?php

namespace App\Http\Middleware;

use Illuminate\Foundation\Http\Middleware\VerifyCsrfToken as Middleware;

class VerifyCsrfToken extends Middleware
{
    /**
     * The URIs that should be excluded from CSRF verification.
     *
     * @var array<int, string>
     */
    protected $except = [
        'success',
        'fail',
        'cancel'
    ];
}
```

### Handle Session Log Out Error in Laravel
The error is related to the 'same_site' and 'secure' options in the config/session.php configuration file.

<pre>
Change 'secure' => env('SESSION_SECURE_COOKIE'), to 'secure' => true;<br>
Change 'same_site' => 'lax', to 'same_site' => 'none';
</pre>

### Update routes/web.php file

``` 
Route::get('/payment', [PaymentController::class,'payment'])->name('payment');

Route::post('/success', [PaymentController::class,'success'])->name('success');
Route::post('/fail', [PaymentController::class,'fail'])->name('fail');
Route::post('/cancel', [PaymentController::class,'cancel'])->name('cancel');

```

### Create PaymentController.php file and update it by the following code

``` 
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class PaymentController extends Controller
{
    public function payment(Request $request)
    {
        $store_id      = config('amarpay.store_id');
        $signature_kay = config('amarpay.signature_kay');

        $transaction_id = rand(000000000000, 999999999999);

        $url = 'https://sandbox.aamarpay.com/jsonpost.php';
        //For Live Transection Use "https://secure.aamarpay.com/jsonpost.php"


        $curl = curl_init();

        curl_setopt_array($curl, array(
            CURLOPT_URL => $url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_ENCODING => '',
            CURLOPT_MAXREDIRS => 10,
            CURLOPT_TIMEOUT => 0,
            CURLOPT_FOLLOWLOCATION => true,
            CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_CUSTOMREQUEST => 'POST',
            CURLOPT_POSTFIELDS =>'{
            "store_id": "'.$store_id.'",
            "tran_id": "'.$transaction_id.'",
            "success_url": "'.route('success').'",
            "fail_url": "'.route('fail').'",
            "cancel_url": "'.route('cancel').'",
            "amount": "10",
            "currency": "BDT",
            "signature_key": "'.$signature_kay.'",
            "desc": "Merchant Registration Payment",
            "cus_name": "Nazmul",
            "cus_email": "nazmul@gmail.com",
            "cus_add1": "House A-55 Road 10",
            "cus_add2": "Jhenaidah, Khulna, Bangladesh",
            "cus_city": "Jhenaidah",
            "cus_state": "Jhenaidah",
            "cus_postcode": "7200",
            "cus_country": "Bangladesh",
            "cus_phone": "+88001700000001",
            "type": "json"
        }',
            CURLOPT_HTTPHEADER => array(
                'Content-Type: application/json'
            ),
        ));

        $response = curl_exec($curl);

        curl_close($curl);

        $responseObject = json_decode($response, true);

        if (isset($responseObject['payment_url']) && $responseObject['payment_url'] != null) {
            return redirect()->away($responseObject['payment_url']);
        }else{
            return redirect()->route('home')->with('error', 'Payment Url Generation Failed!');
        }

    }

    //Get success response
    public function success(Request $request)
    {

        $request_id    = $request['mer_txnid'];
        $store_id      = config('amarpay.store_id');
        $signature_kay = config('amarpay.signature_kay');

        $url = "https://sandbox.aamarpay.com/api/v1/trxcheck/request.php?request_id=$request_id&store_id=$store_id&signature_key=$signature_kay&type=json";
        //For Live Transection Use "http://secure.aamarpay.com/api/v1/trxcheck/request.php"

        $curl = curl_init();

        curl_setopt_array($curl, array(
            CURLOPT_URL => $url,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_ENCODING => '',
            CURLOPT_MAXREDIRS => 10,
            CURLOPT_TIMEOUT => 0,
            CURLOPT_FOLLOWLOCATION => true,
            CURLOPT_HTTP_VERSION => CURL_HTTP_VERSION_1_1,
            CURLOPT_CUSTOMREQUEST => 'GET',
        ));

        $response = curl_exec($curl);

        curl_close($curl);
//        echo $response;
        return redirect()->route('home')->with('success', 'Order placed successfully');
    }

    //get failure response
    public function fail(Request $request)
    {
        return redirect()->route('home')->with('error', 'Order Failed!');
    }

    //
    public function cancel(Request $request)
    {
        return redirect()->route('home')->with('warning', 'Order cancelled!');
    }
}

```
