---
layout: publications
title: "Customer birthday mails in Magento"
categories: publications
modified: 2013-07-10T11:57:41-04:00
toc: false
comments: true
image:
  feature: Magento.jpg
  teaser: Magento.jpg
---

Script that grabs the customers are celebrating their birthday and sends them email.

**Files:**<br/>
1. MAGENTO_ROOT/birthday/birthday.php<br/>
2. MAGENTO_ROOT/app/etc/modules/Magazento_Bigzipper.xml – our small extension config<br/>
3. MAGENTO_ROOT/app/locale/ru_RU/template/email/birthday_default.html – email template with id birthday_default<br/>
4. MAGENTO_ROOT/app/code/local/Magazento/Bigzipper/etc.xml – settings, we register email template here<br/>


**1. birthday.php**

{% highlight php %}
<?php

    ini_set('display_errors',1);
    error_reporting(E_ALL|E_STRICT);
    require_once('../app/Mage.php');
    Mage::app('');

    $daymonthNow = date('d').'-'.date('m');
    $collection = Mage::getModel("customer/customer")
        ->getCollection()->addAttributeToSelect("*")
        ->addFieldToFilter("dob",array('notnull' => false));

    foreach ($collection as $customer)
    {

        $dob = explode(' ',$customer->getDob());
        $date = explode('-',$dob[0]);
        $daymonthBirthday = $date[1].'-'.$date[2];

        if ( $daymonthBirthday  == $daymonthNow ) {

            $customerName = mb_convert_case($customer->getFirstname(), MB_CASE_TITLE, "UTF-8");
            $customerEmail = $customer->getEmail();
            sendBirthdayEmail($customerEmail,$customerName);
        }
    }

function sendBirthdayEmail($customerName,$customerEmail) {
    $storeId = Mage::app()->getStore()->getStoreId();
    $mailTemplate = Mage::getModel('core/email_template');
    $mailTemplate->setDesignConfig(array('area'=>'frontend', 'store'=>$storeId));
    $emailId = 'birthday_default';

    $mailTemplate->sendTransactional(
        $emailId,
        array('email'=>$customerEmail, 'name'=>$customerName),
        $customerEmail,
        null,
        array(
            'customer_name' => $customerName
        )
    );
}

?>
{% endhighlight %}


**2. Magazento_Bigzipper.xml**

{% highlight xml %}
<?xml version="1.0"?>
<config>
    <modules>
        <Magazento_Bigzipper>
            <active>true</active>
            <codePool>local</codePool>
        </Magazento_Bigzipper>
    </modules>
</config>
{% endhighlight %}

**3. birthday_default.html**

{% highlight html %}
<?xml version="1.0"?>
<config>
    <modules>
        <Magazento_Bigzipper>
            <active>true</active>
            <codePool>local</codePool>
        </Magazento_Bigzipper>
    </modules>
</config>
{% endhighlight %}

**4. config.xml**

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<config>
    <modules>
        <Magazento_Bigzipper>
            <version>1.0.2</version>
        </Magazento_Bigzipper>
    </modules>

    <global>
        <template>
            <email>
                <birthday_default>
                    <label>Birthday Default</label>
                    <file>birthday_default.html</file>
                    <type>html</type>
                </birthday_default>
            </email>
        </template>
    </global>
</config>
{% endhighlight %}
