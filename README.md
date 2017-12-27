# iyzico-laravel
Iyzico Laravel Integration

Create an account from https://www.iyzico.com/

#Download iyzipay-php-master.zip file

Eject files to your laravel project vendor file

#Creatre PaymentController
```
<?php

namespace App\Http\Controllers;
require (base_path('vendor/iyzipay-php-master/IyzipayBootstrap.php'));

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;
use IyzipayBootstrap;
use Iyzipay\Options;


class PaymentController extends Controller
{
    public function payment()
    {
        IyzipayBootstrap::init();

        $options = new Options();
        $options->setApiKey("#YOUR API KEY#");
        $options->setSecretKey("#YOUR SECRET KEY#");
        $options->setBaseUrl("https://api.iyzipay.com");

        # create request class
        $request = new \Iyzipay\Request\CreateCheckoutFormInitializeRequest();
        $request->setLocale(\Iyzipay\Model\Locale::TR);
        $request->setConversationId("123456789");
        $request->setPrice("1");
        $request->setPaidPrice("1.2");
        $request->setCurrency(\Iyzipay\Model\Currency::TL);
        $request->setBasketId("B67832");
        $request->setPaymentGroup(\Iyzipay\Model\PaymentGroup::PRODUCT);
        $request->setCallbackUrl("https://#YOUR DOMAIN NAME#/callback");
        $request->setEnabledInstallments(array(2, 3, 6, 9));

        $buyer = new \Iyzipay\Model\Buyer();
        $buyer->setId("BY789");
        $buyer->setName("John");
        $buyer->setSurname("Doe");
        $buyer->setGsmNumber("+905350000000");
        $buyer->setEmail("email@email.com");
        $buyer->setIdentityNumber("74300864791");
        $buyer->setLastLoginDate("2015-10-05 12:43:35");
        $buyer->setRegistrationDate("2013-04-21 15:12:09");
        $buyer->setRegistrationAddress("Nidakule Göztepe, Merdivenköy Mah. Bora Sok. No:1");
        $buyer->setIp("85.34.78.112");
        $buyer->setCity("Istanbul");
        $buyer->setCountry("Turkey");
        $buyer->setZipCode("34732");

        $request->setBuyer($buyer);
        $shippingAddress = new \Iyzipay\Model\Address();
        $shippingAddress->setContactName("Jane Doe");
        $shippingAddress->setCity("Istanbul");
        $shippingAddress->setCountry("Turkey");
        $shippingAddress->setAddress("Nidakule Göztepe, Merdivenköy Mah. Bora Sok. No:1");
        $shippingAddress->setZipCode("34742");
        $request->setShippingAddress($shippingAddress);

        $billingAddress = new \Iyzipay\Model\Address();
        $billingAddress->setContactName("Jane Doe");
        $billingAddress->setCity("Istanbul");
        $billingAddress->setCountry("Turkey");
        $billingAddress->setAddress("Nidakule Göztepe, Merdivenköy Mah. Bora Sok. No:1");
        $billingAddress->setZipCode("34742");
        $request->setBillingAddress($billingAddress);

        $basketItems = array();
        $firstBasketItem = new \Iyzipay\Model\BasketItem();
        $firstBasketItem->setId("BI101");
        $firstBasketItem->setName("Binocular");
        $firstBasketItem->setCategory1("Collectibles");
        $firstBasketItem->setCategory2("Accessories");
        $firstBasketItem->setItemType(\Iyzipay\Model\BasketItemType::PHYSICAL);
        $firstBasketItem->setPrice("0.3");
        $basketItems[0] = $firstBasketItem;

        $secondBasketItem = new \Iyzipay\Model\BasketItem();
        $secondBasketItem->setId("BI102");
        $secondBasketItem->setName("Game code");
        $secondBasketItem->setCategory1("Game");
        $secondBasketItem->setCategory2("Online Game Items");
        $secondBasketItem->setItemType(\Iyzipay\Model\BasketItemType::VIRTUAL);
        $secondBasketItem->setPrice("0.5");

        $basketItems[1] = $secondBasketItem;
        $thirdBasketItem = new \Iyzipay\Model\BasketItem();
        $thirdBasketItem->setId("BI103");
        $thirdBasketItem->setName("Usb");
        $thirdBasketItem->setCategory1("Electronics");
        $thirdBasketItem->setCategory2("Usb / Cable");
        $thirdBasketItem->setItemType(\Iyzipay\Model\BasketItemType::PHYSICAL);
        $thirdBasketItem->setPrice("0.2");
        $basketItems[2] = $thirdBasketItem;
        $request->setBasketItems($basketItems);

        # make request
        $checkoutFormInitialize = \Iyzipay\Model\CheckoutFormInitialize::create($request, $options);

        return view('payment.payment_page');
    }

    public function paymentStatus()
    {
        IyzipayBootstrap::init();

        $options = new \Iyzipay\Options();
        $options->setApiKey("sandbox-vm7fzwANvAWm4X33mcBAcDbgomcfL3nN");
        $options->setSecretKey("sandbox-l3Dd9ra0xhUhidhxkttHvbuVyP4hX1en");
        $options->setBaseUrl("https://sandbox-api.iyzipay.com");

        $request1 = request();
        $token = $request1->get('token');

        # create request class
        $request = new \Iyzipay\Request\RetrieveCheckoutFormRequest();
        $request->setLocale(\Iyzipay\Model\Locale::TR);
        $request->setConversationId("123456789");
        $request->setToken($token);

        # make request
        $checkoutForm = \Iyzipay\Model\CheckoutForm::retrieve($request, $options);

        $control2 = $checkoutForm->getBasketId();

        # print result

        //Yapılan isteğin sonucunu bildirir. Başarılı ise success, hatalı ise failure döner.
        $status1 = $checkoutForm->getStatus();

        //Ödeme isteğinin durumunu gösterir. Success ise karttan ilgili tutar çekilmiştir.
        //SUCCESS, FAILURE, INIT_THREEDS, CALLBACK_THREEDS, BKM_POS_SELECTED, CALLBACK_PECCO
        $status2 = $checkoutForm->getPaymentStatus();

        //İşlem hatalıysa, bu hataya dair belirtilen mesajdır, locale parametresine göre dil desteği sunar.
        $status3 = $checkoutForm->getErrorMessage();

        //Ödemeye ait id, üye işyeri tarafından mutlaka saklanmalıdır. Ödemenin iptali ve iyzico ile iletişimde kullanılır.
        $paymentId = $checkoutForm->getPaymentId();
        $iyizicoFee = $checkoutForm->getIyziCommissionFee();
        $iyizico = $checkoutForm->getIyziCommissionRateAmount();

        //Ödemenin taksit bilgisi, tek çekim için 1 döner. Geçerli değerler: 1, 2, 3, 6, 9.
        $taksit = $checkoutForm->getInstallment();

        if($status1 == "success")
        {
            if($status2 == "SUCCESS")
            {
                //TO DO WHAT YOU NEED 
                return view('payment.success', compact('paymentId', 'iyizico', 'iyizicoFee', 'taksit'));
            }
            else
            {
                return view('payment.fail', compact('status3'));
            }
        }
        else
        {
            return view('payment.fail', compact('status3'));
        }
    }
}
```
#GIVE PERMISSOIN TO YOUR LARAVEL PROJECT TO ACCEPT FROM /callback URL

Go to VerifyCsrfToken.php file (app->Http->Middleware) and modify it;
```
 protected $except = [
        '/callback'
    ];
```  
Write your route to web.php

Ready!


