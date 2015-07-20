---
layout: publications
title: "Moving Symfony2 from ORM to MongoDB"
categories: publications
modified: 2015-06-16T11:57:41-04:00
toc: false
comments: true
image:
  feature: sf2.jpg
  teaser: sf2.jpg
---

##Installing mongo
Skip it if you already have mongo. In case if you don't, follow these easy steps for installation<br/>
I'm using mac so in my case it will be **brew**. Process is the same for Linux/Ubuntu, only one difference that you will use apt-get, yum, etc..

Update brew with:<br/>
```
brew update
```

Install mongodb<br/>
```
brew install mongodb
```

This command will install RoboMongo, http://robomongo.org/<br/>
```
brew casc install robomongo
```

Create directory for mongo collections, with<br/>
```
mkdir -p /data/db
```
and
```
sudo chmod -R 777 /data/db
```

Once mongodb installed on your machine, you can launch it with<br/>
```
mongod
```

PHP support for mongodb<br/>
```
brew install php55-mongo
```

To check that you have mongod installed, type<br/>
```
php -m | grep mongo
```

##Application vendors
Assume your composer look like this, you have symfony and doctrine ORM bundle installed
<br/>
{% highlight JavaScript %}
"php": ">=5.3.3",
"symfony/symfony": "2.6.*",
"doctrine/orm": "=v2.4.2",
"doctrine/doctrine-bundle": "~1.3",
"stof/doctrine-extensions-bundle": "1.2.*@dev",
"doctrine/doctrine-fixtures-bundle": "2.2.*@dev",
{% endhighlight %}
for mongo your will need these vendors
{% highlight JavaScript %}
"doctrine/mongodb-odm": "1.0.0-BETA12",
"doctrine/mongodb-odm-bundle": "3.0.0-BETA6",
{% endhighlight %}

Once you've added mongo vendors, update your system with<br/> 
```
composer update 
```

Add MongoDB bundle to your app/AppKernel.php (registerBundles public function)
{% highlight JavaScript %}
new Doctrine\Bundle\MongoDBBundle\DoctrineMongoDBBundle(),
{% endhighlight %}

And settings to your config.yml file

{% highlight JavaScript %}
doctrine_mongodb:
    connections:
        default:
            server: mongodb://localhost:27017
            options: {}
    default_database: aisel
    document_managers:
        default:
            auto_mapping: true
{% endhighlight %}

Once this step done you may check, that your symfony app has mongodb support with command<br/>
```
php app/console | grep mongodb
```

it will output you the list of available commands
 
{% highlight JavaScript %}
doctrine:mongodb:cache:clear-metadata    Clear all metadata cache for a document manager.
doctrine:mongodb:fixtures:load           Load data fixtures to your database.
doctrine:mongodb:generate:documents      Generate document classes and method stubs from your mapping information.
doctrine:mongodb:generate:hydrators      Generates hydrator classes for document classes.
doctrine:mongodb:generate:proxies        Generates proxy classes for document classes.
doctrine:mongodb:generate:repositories   Generate repository classes from your mapping information.
doctrine:mongodb:mapping:info            Show basic information about all mapped documents.
doctrine:mongodb:query                   Query mongodb and inspect the outputted results from your document classes.
doctrine:mongodb:schema:create           Create databases, collections and indexes for your documents
doctrine:mongodb:schema:drop             Drop databases, collections and indexes for your documents
doctrine:mongodb:schema:update           Update indexes for your documents
doctrine:mongodb:tail-cursor             Tails a mongodb cursor and processes the documents that come through
{% endhighlight %}


##Patching the code
Its not hard at all to move all your Symfony models that were done as entities to mongodb collection.
Doctrine ORM use **Entity** directory for models, with Doctrine ODM models must me located inside **Document** directory.

**Model definintien in ORM**
{% highlight JavaScript %}
<?php

namespace Aisel\AddressingBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;
use Gedmo\Mapping\Annotation as Gedmo;
use Doctrine\ORM\Mapping as ORM;

/**
 * Country
 *
 * @author Ivan Proskoryakov <volgodark@gmail.com>
 *
 * @ORM\HasLifecycleCallbacks()
 * @ORM\Entity(repositoryClass="Aisel\AddressingBundle\Entity\CountryRepository")
 * @ORM\Table(name="aisel_addressing_country")
 */
class Country
{
    /**
     * @var integer
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;
{% endhighlight %}

in MongoDB it become 
{% highlight JavaScript %}
<?php

namespace Aisel\AddressingBundle\Document;

use Symfony\Component\Validator\Constraints as Assert;
use Gedmo\Mapping\Annotation as Gedmo;
use Doctrine\ODM\MongoDB\Mapping\Annotations as ODM;

/**
 * Country
 *
 * @author Ivan Proskoryakov <volgodark@gmail.com>
 *
 * @ODM\HasLifecycleCallbacks()
 * @ODM\Document(collection="aisel_addressing_country")
 * @ODM\Document(repositoryClass="Aisel\AddressingBundle\Document\CountryRepository")
 */
class Country
{
    /**
     * @var string
     * @ODM\Id
     * @JMS\Type("string")
     */
    private $id;
{% endhighlight %}


**ORM ManyToOne**<br/>
{% highlight JavaScript %}
/**
 * @var Country
 * @ORM\ManyToOne(targetEntity="Aisel\AddressingBundle\Entity\Country")
 * @ORM\JoinColumn(name="country_id", referencedColumnName="id")
 */
private $country;
{% endhighlight %}

become ReferenceOne in ODM
{% highlight JavaScript %}
/**
 * @var Country
 * @ODM\ReferenceOne("Aisel\AddressingBundle\Document\Country", nullable=true)
 */
private $country;
{% endhighlight %}

**ORM OneToOne**<br/>
{% highlight JavaScript %}
/**
 * @var Country
 * @ORM\OneToOne(targetEntity="Aisel\AddressingBundle\Entity\Country")
 * @ORM\JoinColumn(name="country_id", referencedColumnName="id")
 */
private $country;
{% endhighlight %}

become ReferenceOne in ODM
{% highlight JavaScript %}
/**
 * @var Country
 * @ODM\ReferenceOne("Aisel\AddressingBundle\Document\Country", nullable=true)
 */
{% endhighlight %}