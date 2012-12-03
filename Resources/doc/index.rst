============
Installation
============
Dependencies
------------
This plugin depends on the JMSPaymentCoreBundle_, so you'll need to add this to your kernel
as well even if you don't want to use its persistence capabilities.

Configuration
-------------
::

    // YAML
    jms_payment_paypal:
        username: your api username (not your account username)
        password: your api password (not your account password)
        signature: your api signature
        debug: true/false # when true, connect to PayPal sandbox; uses kernel debug value when not specified


=====
Usage
=====
With the Payment Plugin Controller (Recommended)
------------------------------------------------
http://jmsyst.com/bundles/JMSPaymentCoreBundle/master/usage

Without the Payment Plugin Controller
-------------------------------------
The Payment Plugin Controller is made available by the CoreBundle and basically is the 
interface to a persistence backend like the Doctrine ORM. It also performs additional 
integrity checks to validate transactions. If you don't need these checks, and only want 
an easy way to communicate with the Paypal API, then you can use the plugin directly::

    $plugin = $container->get('payment.plugin.paypal_express_checkout');

.. _JMSPaymentCoreBundle: https://github.com/schmittjoh/JMSPaymentCoreBundle/blob/master/Resources/doc/index.rst


Setting external parameters for Paypal
---------------------------------------
Sometimes it may be important to put extra informations along with request made for paypal.
This can be achieved by explicitly setting *checkout_params* for ExtendedData entity.

::

     $ppc = $this->container->get('payment.plugin_controller'); // access JMSPaymentCorePluginController

     $currency = 'EUR';
     $invoiceNumber = 'A12345';
     $amount = '100'; // some arbitrary value, should be obtained dynamically
     $extendedData = new ExtendedData();
     $extendedData->set(array(
        'return_url' => $this->get('router')->generate('approvePayment', array('paymentId' => $paymentId), true),
        'cancel_url' => $this->get('router')->generate('cancelPayment', array(), true),
        'checkout_params' => array(                                 
                                    'L_PAYMENTREQUEST_0_NAME0' => 'Order name here',
                                    'L_PAYMENTREQUEST_0_NUMBER0' => $invoiceNumber,
                                    'L_PAYMENTREQUEST_0_AMT0' => $amount,
                                    'L_PAYMENTREQUEST_0_QTY0' => '1',
                                    'PAYMENTREQUEST_0_CURRENCYCODE' => $currency)
     
     ));
     // other payment params can be taken from https://www.x.com/developers/paypal/documentation-tools/paypal-payments-pro/integration-guide/WPWebsitePaymentsPro
     
     $instruction = new PaymentInstruction($amount, $currency, 'paypal_express_checkout', $extendedData);
     $ppc->createPaymentInstruction($instruction);


     $paymentId = $ppc->createPayment($instruction->getId(), $amount)->getId();
     
     
     $result = $this->ppc->approveAndDeposit($paymentId, $amount);
