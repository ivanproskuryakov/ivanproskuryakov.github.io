---
layout: publications
title: "Export Magento orders to ChannelAdvisor"
categories: publications
modified: 2013-07-27T11:57:41-04:00
toc: false
comments: true
image:
  feature: Magento.jpg
  teaser: Magento.jpg
---
ChannelAdvisors uses SOAP for API. Of cource plain POST requests, CURL, file_get_contents are the nice things, but for me SOAP is a much easier to use.<br/>
For more information visit developer.channeladvisor.com website, it contains all the info you will need.<br/>
Page http://developer.channeladvisor.com/display/cadn/SubmitOrder for Order Submit

First of all you have to synchronise all products inside the store with your ChannelAdvisor account.
A. Check does SKU exists with call “DoesSkuExist”<br/>
B. Synchronize your product with “SynchInventoryItem”<br/>

When all product SKU’s would be been synchronized, you may add sales with method “SubmitOrder”<br/>
Once order would be processed, order will be marked inside Magento with the flag, to skip the order in the next run.
Script also skips all the orders with statuses “canceled” and “closed”, orders other with other statuses will be processed.<br/>

> [Sources: https://github.com/ivanproskuryakov/MagentoChanneladvisor](https://github.com/ivanproskuryakov/MagentoChanneladvisor)

magento.php - main file, you will need to launch it each time with the browser or run it automatically with crontab<br/>
states.php – list of the states, we need this for name conversions<br/>
settings.php – file with your settings (account credentials for ChannelAdvisor, e-mail etc..)<br/>

**magento.php**
{% highlight php %}
<?php
set_time_limit(0);
error_reporting(E_ALL);
ini_set('error_reporting', E_ALL);
ini_set('memory_limit', '1024M');
include_once 'settings.php';
include_once 'states.php';
include_once "../app/Mage.php";

umask (0);
$app = Mage::app()->setCurrentStore(Mage_Core_Model_App::ADMIN_STORE_ID);
$orders = Mage::getModel('sales/order')->getCollection();
$log ='';

/*
 * SYNC SKUS
 */
$client = new SoapClient("http://api.channeladvisor.com/ChannelAdvisorAPI/v7/InventoryService.asmx?WSDL");
$headersData = array('DeveloperKey' => DEVELOPERKEY, 'Password' => PASSWORD);
$head = new SoapHeader("http://api.channeladvisor.com/webservices/","APICredentials",$headersData);
$client->__setSoapHeaders($head);
foreach ($orders as $_order) {


 if (($_order->getStatus() == 'canceled') || ($_order->getStatus() == 'closed') || ($_order->getStatus() == 'complete')) continue;
 if (($_order->getData('ext_order_id') =='ORDER_EXPORTED_CA')) continue;

 $items = $_order->getItemsCollection();
 foreach ($items AS $itemid => $_item) {

 $item = array();
 $item['Sku'] = $_item->getSku();
 $item['Title'] = $_item->getName();

 // check sku exists
 $result = $client->DoesSkuExist(array('accountID'=>ACCOUNTID,'sku'=>$item['Sku']));
 $skuExists = $result->DoesSkuExistResult->ResultData;

// synch sku
 if ($skuExists == false) {
 $result = $client->SynchInventoryItem(array('accountID'=>ACCOUNTID,'item'=>$item));
 $log.='SKU=>'.$item['Sku'] .' NAME=>'.$item['Title'] .' STATUS=> '.$result->SynchInventoryItemResult->Status.'<br/>';
 }

 }
}


/*
 * ADD SALES
 */
$total = 0;
$client = new SoapClient("http://api.channeladvisor.com/ChannelAdvisorAPI/v7/OrderService.asmx?WSDL");
$headersData = array('DeveloperKey' => DEVELOPERKEY, 'Password' => PASSWORD);
$head = new SoapHeader("http://api.channeladvisor.com/webservices/","APICredentials",$headersData);
$client->__setSoapHeaders($head);
foreach ($orders as $_order) {
 if (($_order->getStatus() == 'canceled') || ($_order->getStatus() == 'closed') || ($_order->getStatus() == 'complete')) continue;
 if (($_order->getData('ext_order_id') == 'ORDER_EXPORTED_CA')) continue;

$total++;
 $shippingAddress = $_order->getShippingAddress();
 $billingAddress = $_order->getBillingAddress();
 $items = $_order->getItemsCollection();
 $_order->setData('ext_order_id','ORDER_EXPORTED_CA');

 /*
 * ###################################################################################################
 *
 * Time
 */
 $_date = date('Y-m-d',strtotime($_order->getCreatedAt()));
 $_time = date('H:i:s',strtotime($_order->getCreatedAt()));
 $_datetime = $_date.'T'.$_time;
 $order['OrderTimeGMT'] = $_datetime;
 $order['ClientOrderIdentifier'] = $_order->getIncrementId();


 /*
 * Status
 */
 $OrderStatus = array();

 if ($_order->getStatus() == 'processing') {
 $_paymentStatus = 'Cleared';
 }
 if (($_order->getStatus() == 'pending') || ($_order->getStatus() == 'holded') ||
 ($_order->getStatus() == 'fraud') || ($_order->getStatus() == 'pending_payment') ||
 ($_order->getStatus() == 'pending_paypal') || ($_order->getStatus() == 'holded')) {
 $_paymentStatus = 'NotSubmitted';
 }

 if ($_order->getStatus() == 'processing') {
 $_checkoutStatus = 'Completed';
 }
 if (($_order->getStatus() == 'pending') || ($_order->getStatus() == 'holded') ||
 ($_order->getStatus() == 'fraud') || ($_order->getStatus() == 'pending_payment') ||
 ($_order->getStatus() == 'pending_paypal') || ($_order->getStatus() == 'holded')) {
 $_checkoutStatus = 'OnHold';
 }
 $OrderStatus['CheckoutStatus'] = 'Completed';
 $OrderStatus['CheckoutDateGMT'] = $_datetime;
 $OrderStatus['PaymentStatus'] = $_paymentStatus;
 $OrderStatus['PaymentDateGMT'] = $_datetime;
 $OrderStatus['ShippingStatus'] = 'Unshipped';
 $OrderStatus['ShippingDateGMT'] = $_datetime;
 $order['OrderStatus']=$OrderStatus;


 /*
 * Customer Email
 */
 $order['BuyerEmailAddress']=$_order->getCustomerEmail();
 $order['EmailOptIn']='false';


 /*
 * Billing
 */
 $BillingInfo = array();
 $BillingInfo['AddressLine1'] = $billingAddress->getStreet(1);
 $BillingInfo['AddressLine2'] = $billingAddress->getStreet(2);
 $BillingInfo['City'] = $billingAddress->getCity();
 $BillingInfo['Region'] = convert_state(ucfirst( $billingAddress->getRegion() ),'abbrev');
 $BillingInfo['PostalCode'] = $billingAddress->getPostcode();
 $BillingInfo['CountryCode'] = $billingAddress->getCountryId();
 $BillingInfo['FirstName'] = $billingAddress->getFirstname();
 $BillingInfo['LastName'] = $billingAddress->getLastname();
 $BillingInfo['PhoneNumberDay'] = $billingAddress->getTelephone();
 $BillingInfo['PhoneNumberEvening'] = $billingAddress->getTelephone();
 $order['BillingInfo']=$BillingInfo;


 /*
 * Payment
 */
 $PaymentInfo = array();
 $PaymentInfo['PaymentType'] = $_order->getIncrementId();
 $PaymentInfo['PaymentTransactionID'] = $_order->getIncrementId();
 if ($_order->getPayment()->getMethod() == 'paypal_standard' ) {
 $PaymentInfo['MerchantReferenceNumber'] = $_order->getIncrementId(); //MerchantReferenceNumber
 $PaymentInfo['PaymentType'] = 'PP'; //PaymentType
 $PaymentInfo['PaymentTransactionID'] = $_order->getPayment()->getLastTransId(); //PaymentTransactionID
 $PaymentInfo['PaypalID'] = $_order->getPayment()->getAdditionalInformation('paypal_payer_id'); //PaypalID
 }
 if ($_order->getPayment()->getMethod() == 'authorizenet' ) {
// var_dump($_order->getPayment()->getData());
 $PaymentInfo['MerchantReferenceNumber'] = $_order->getIncrementId(); //MerchantReferenceNumber
 $_authorize_cards = reset($_order->getPayment()->getAdditionalInformation('authorize_cards'));
 $_cc_type = $_authorize_cards['cc_type'];
 if ($_cc_type =='AE') $_cc_type = 'AX';
 $PaymentInfo['PaymentType'] = $_cc_type; //PaymentType
 $PaymentInfo['PaymentTransactionID'] = $_authorize_cards['last_trans_id']; //PaymentTransactionID
 $PaymentInfo['CreditCardLast4'] = $_authorize_cards['cc_last4']; //CreditCardLast4
 }
 if ($_order->getPayment()->getMethod() == 'googlecheckout' ) {
 $PaymentInfo['MerchantReferenceNumber'] = $_order->getIncrementId(); //MerchantReferenceNumber
 $PaymentInfo['PaymentType'] = 'GG'; //PaymentType
 $PaymentInfo['CreditCardLast4'] = $_order->getPayment()->getData('cc_last4'); //CreditCardLast4
 $_googlecheckoutTransId = str_replace("-capture", "", $_order->getPayment()->getData('last_trans_id'));
 $PaymentInfo['PaymentTransactionID'] = $_googlecheckoutTransId; //PaymentTransactionID
 }
 $order['PaymentInfo']=$PaymentInfo;

 /*
 * ShoppingCart
 */
 $ShoppingCart = array();
 $ShoppingCart['CartID'] = 0;
 $ShoppingCart['CheckoutSource'] = 'Unspecified';
 $ShoppingCart['VATTaxCalculationOption'] = 'Unspecified';
 $ShoppingCart['VATShippingOption'] = 'Unspecified';
 $ShoppingCart['VATGiftWrapOption'] = 'Unspecified';

$LineItemSKUList = array();
 foreach ($items AS $itemid => $item) {
 $OrderLineItemItem = array();
 $OrderLineItemItem['ItemSaleSource'] = 'MAGENTO_ENTERPRISE';
 $OrderLineItemItem['UnitPrice'] = $item->getPrice();
 $OrderLineItemItem['LineItemID'] = $item->getProductId();
 $OrderLineItemItem['AllowNegativeQuantity'] = 'false';
 $OrderLineItemItem['Quantity'] = $item->getQtyOrdered();
 $OrderLineItemItem['SKU'] = $item->getSku();
 $OrderLineItemItem['BuyerFeedbackRating'] = '';
 $OrderLineItemItem['VATRate'] = 0;
 $OrderLineItemItem['TaxCost'] = $item->getData('tax_amount');
 $LineItemSKUList['OrderLineItemItem'][] = $OrderLineItemItem;
 }
 $ShoppingCart['LineItemSKUList'] = $LineItemSKUList;

$OrderLineItemInvoice1 = array();
 $OrderLineItemInvoice1['LineItemType'] = 'SalesTax';
 $OrderLineItemInvoice1['UnitPrice'] = $_order->getTaxAmount();
 $OrderLineItemInvoice2 = array();
 $OrderLineItemInvoice2['LineItemType'] = 'Shipping';
 $OrderLineItemInvoice2['UnitPrice'] = $_order->getShippingAmount();
 $OrderLineItemInvoice3 = array();
 $OrderLineItemInvoice3['LineItemType'] = 'VATShipping';
 $OrderLineItemInvoice3['UnitPrice'] = $_order->getShippingTaxAmount();

$ShoppingCart['LineItemInvoiceList'] = array();
 $ShoppingCart['LineItemInvoiceList'][] = $OrderLineItemInvoice1;
 $ShoppingCart['LineItemInvoiceList'][] = $OrderLineItemInvoice2;
 $ShoppingCart['LineItemInvoiceList'][] = $OrderLineItemInvoice3;

$order['ShoppingCart']=$ShoppingCart;

 /*
 * ShippingInfo
 */
 $ShippingInfo = array();
 $ShippingInfo['AddressLine1'] = $shippingAddress->getStreet(1);
 $ShippingInfo['AddressLine2'] = $shippingAddress->getStreet(2);
 $ShippingInfo['City'] = $shippingAddress->getCity();
 $ShippingInfo['Region'] = convert_state(ucfirst( $shippingAddress->getRegion() ),'abbrev'); // IL
 $ShippingInfo['PostalCode'] = $shippingAddress->getPostcode();
 $ShippingInfo['CountryCode'] = $shippingAddress->getCountryId();
 $ShippingInfo['FirstName'] = $shippingAddress->getFirstname();
 $ShippingInfo['LastName'] = $shippingAddress->getLastname();
 $ShippingInfo['PhoneNumberDay'] = $shippingAddress->getTelephone();
 $ShippingInfo['PhoneNumberEvening'] = $shippingAddress->getTelephone();
 $order['ShippingInfo']=$ShippingInfo;


 $_order->setStatus('complete');
 $_order->save();
// var_dump($order);
// exit();


 $result = $client->SubmitOrder(array('accountID'=>ACCOUNTID,'order'=>$order));
 $log.='ORDER[SUBMIT]=>'.$order['ClientOrderIdentifier'].' STATUS=> '.$result->SubmitOrderResult->Status.'<br/>';
}

/*
 * EMAIL
 *
 */
$log.='Total sales: '.$total.'<br/>';
echo $log;
mail(EMAIL_TO, EMAIL_SUBJECT, $log, EMAIL_HEADERS);
exit('EXIT');
?>
{% endhighlight %}

**states.php**
{% highlight php %}
<?php
    function convert_state($name, $to='name') {
    $states = array(
        array('abbrev'=>'AL', 'name'=>'Alabama'),
        array('abbrev'=>'AK', 'name'=>'Alaska'),
        array('abbrev'=>'AS', 'name'=>'American Samoa'),
        array('abbrev'=>'AZ', 'name'=>'Arizona'),
        array('abbrev'=>'AR', 'name'=>'Arkansas'),
        array('abbrev'=>'AF', 'name'=>'Armed Forces Africa'),
        array('abbrev'=>'AA', 'name'=>'Armed Forces Americas'),
        array('abbrev'=>'AC', 'name'=>'Armed Forces Canada'),
        array('abbrev'=>'AE', 'name'=>'Armed Forces Europe'),
        array('abbrev'=>'AM', 'name'=>'Armed Forces Middle East'),
        array('abbrev'=>'AP', 'name'=>'Armed Forces Pacific'),
        array('abbrev'=>'CA', 'name'=>'California'),
        array('abbrev'=>'CO', 'name'=>'Colorado'),
        array('abbrev'=>'CT', 'name'=>'Connecticut'),
        array('abbrev'=>'DE', 'name'=>'Delaware'),
        array('abbrev'=>'DC', 'name'=>'District of Columbia'),
        array('abbrev'=>'FM', 'name'=>'Federated States Of Micronesia'),
        array('abbrev'=>'FL', 'name'=>'Florida'),
        array('abbrev'=>'GA', 'name'=>'Georgia'),
        array('abbrev'=>'GU', 'name'=>'Guam'),
        array('abbrev'=>'HI', 'name'=>'Hawaii'),
        array('abbrev'=>'ID', 'name'=>'Idaho'),
        array('abbrev'=>'IL', 'name'=>'Illinois'),
        array('abbrev'=>'IN', 'name'=>'Indiana'),
        array('abbrev'=>'IA', 'name'=>'Iowa'),
        array('abbrev'=>'KS', 'name'=>'Kansas'),
        array('abbrev'=>'KY', 'name'=>'Kentucky'),
        array('abbrev'=>'LA', 'name'=>'Louisiana'),
        array('abbrev'=>'ME', 'name'=>'Maine'),
        array('abbrev'=>'MH', 'name'=>'Marshall Islands'),
        array('abbrev'=>'MD', 'name'=>'Maryland'),
        array('abbrev'=>'MA', 'name'=>'Massachusetts'),
        array('abbrev'=>'MI', 'name'=>'Michigan'),
        array('abbrev'=>'MN', 'name'=>'Minnesota'),
        array('abbrev'=>'MS', 'name'=>'Mississippi'),
        array('abbrev'=>'MO', 'name'=>'Missouri'),
        array('abbrev'=>'MT', 'name'=>'Montana'),
        array('abbrev'=>'NE', 'name'=>'Nebraska'),
        array('abbrev'=>'NV', 'name'=>'Nevada'),
        array('abbrev'=>'NH', 'name'=>'New Hampshire'),
        array('abbrev'=>'NJ', 'name'=>'New Jersey'),
        array('abbrev'=>'NM', 'name'=>'New Mexico'),
        array('abbrev'=>'NY', 'name'=>'New York'),
        array('abbrev'=>'NC', 'name'=>'North Carolina'),
        array('abbrev'=>'ND', 'name'=>'North Dakota'),
        array('abbrev'=>'MP', 'name'=>'Northern Mariana Islands'),
        array('abbrev'=>'OH', 'name'=>'Ohio'),
        array('abbrev'=>'OK', 'name'=>'Oklahoma'),
        array('abbrev'=>'OR', 'name'=>'Oregon'),
        array('abbrev'=>'PW', 'name'=>'Palau'),
        array('abbrev'=>'PA', 'name'=>'Pennsylvania'),
        array('abbrev'=>'PR', 'name'=>'Puerto Rico'),
        array('abbrev'=>'RI', 'name'=>'Rhode Island'),
        array('abbrev'=>'SC', 'name'=>'South Carolina'),
        array('abbrev'=>'SD', 'name'=>'South Dakota'),
        array('abbrev'=>'TN', 'name'=>'Tennessee'),
        array('abbrev'=>'TX', 'name'=>'Texas'),
        array('abbrev'=>'UT', 'name'=>'Utah'),
        array('abbrev'=>'VT', 'name'=>'Vermont'),
        array('abbrev'=>'VI', 'name'=>'Virgin Islands'),
        array('abbrev'=>'VA', 'name'=>'Virginia'),
        array('abbrev'=>'WA', 'name'=>'Washington'),
        array('abbrev'=>'WV', 'name'=>'West Virginia'),
        array('abbrev'=>'WI', 'name'=>'Wisconsin'),
        array('abbrev'=>'WY', 'name'=>'Wyoming'),
    );

    $return = false;
    foreach ($states as $state) {
        if ($to == 'name') {
            if (strtolower($state['abbrev']) == strtolower($name)){
                $return = $state['name'];
                break;
            }
        } else if ($to == 'abbrev') {
            if (strtolower($state['name']) == strtolower($name)){
                $return = strtoupper($state['abbrev']);
                break;
            }
        }
    }
    return $return;
}
?>
{% endhighlight %}

**settings.php**
{% highlight php %}
<?php
    /*
     * Channel
     */
    define('DEVELOPERKEY','########-####-####-####-############');
    define('PASSWORD','########');
    define('localID','');
    define('ACCOUNTID','########-####-####-####-############');
 
    /*
     * FTP + DIRECTORY
     */
    define('IMPORT_DIR', __DIR__.'/GLOBAL/');
    define('IMPORT_DIR_HISTORY', __DIR__.'/GLOBAL_HISTORY/');
    define('FTP_SERVER',"#");
    define('FTP_USER',"#");
    define('FTP_PASSWORD',"#");
     
    /*
     * EMAIL
     */
    define('EMAIL_TO', 'youremail@gmail.com');
    define('EMAIL_SUBJECT', 'SCV PRODUCT IMPORT '. date("Y-m-d H:i:s", time()));
    define('EMAIL_HEADERS', "From: noreply@email.com \r\n" .
        "Reply-To: noreply@email.com \r\n" .
        "Content-type: text/html; charset=UTF-8 \r\n");
     
 
?>
{% endhighlight %}
