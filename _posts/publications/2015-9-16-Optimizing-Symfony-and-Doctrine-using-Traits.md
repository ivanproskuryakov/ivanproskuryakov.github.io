---
layout: publications
title: "Optimizing Symfony & Doctrine using Traits"
categories: publications
modified: 2015-09-16T11:57:41-04:00
toc: false
comments: true
image:
  feature: sf2.jpg
  teaser: sf2.jpg
---

Since v5.4.0, PHP implements a method of code reuse called Traits.<br/>
This means all duplicated code, which was not impossible to implement as functions or classes could be grouped now.
In big projects Doctrine entities or documents(in case of mongo) are always full of duplicated fields like: 
id, title, createdAt, updatedAt, etc. fields are common across all models and Traits are perfect solution for them.<br/>

Bellow example for Id, CreateAt, UpdatedAt and Name fields. 
I'm sure than 90% of doctrine fields could be unified and stored as traits.<br/>

With PHP Traits Doctrine Document become like this:
{% highlight JavaScript %} 
 <?php
 
 namespace Aisel\AddressingBundle\Document;
 
 use Symfony\Component\Validator\Constraints as Assert;
 use Doctrine\ODM\MongoDB\Mapping\Annotations as ODM;
 use JMS\Serializer\Annotation as JMS;
 use Aisel\ResourceBundle\Domain\IdTrait;
 use Aisel\ResourceBundle\Domain\UpdateCreateTrait;
 use Aisel\ResourceBundle\Domain\NameTrait;
 
 /**
  * City
  *
  * @author Ivan Proskuryakov <volgodark@gmail.com>
  *
  * @ODM\HasLifecycleCallbacks()
  * @ODM\Document(
  *      collection="aisel_addressing_city",
  *      repositoryClass="Aisel\ResourceBundle\Repository\CollectionRepository"
  * )
  */
 class City
 {
 
     use IdTrait;
     use NameTrait;
     use UpdateCreateTrait;
     
     // ... other fields impossible to group 
{% endhighlight %}     

Traits implementation
{% highlight JavaScript %}
<?php

namespace Aisel\ResourceBundle\Domain;

use Doctrine\ODM\MongoDB\Mapping\Annotations as ODM;
use JMS\Serializer\Annotation as JMS;

/**
 * IdTrait
 *
 * @author Ivan Proskuryakov <volgodark@gmail.com>
 *
 */
trait IdTrait
{

    /**
     * @var string
     * @ODM\Id
     * @JMS\Expose
     * @JMS\Type("string")
     */
    private $id;

    /**
     * Get id
     *
     * @return string
     */
    public function getId()
    {
        return $this->id;
    }
}
{% endhighlight %}

{% highlight JavaScript %}
<?php

namespace Aisel\ResourceBundle\Domain;

use Doctrine\ODM\MongoDB\Mapping\Annotations as ODM;
use JMS\Serializer\Annotation as JMS;

/**
 * NameTrait
 *
 * @author Ivan Proskuryakov <volgodark@gmail.com>
 *
 */
trait NameTrait
{

    /**
     * @var string
     * @ODM\Field(type="string")
     * @Assert\Type(type="string")
     * @Assert\NotNull()
     * @JMS\Expose
     * @JMS\Type("string")
     */
    private $name;

    /**
     * Set name
     *
     * @param  string $name
     * @return mixed
     */
    public function setName($name)
    {
        $this->name = $name;

        return $this;
    }

    /**
     * Get name
     *
     * @return string
     */
    public function getName()
    {
        return $this->name;
    }

}
{% endhighlight %}

{% highlight JavaScript %}
<?php

namespace Aisel\ResourceBundle\Domain;

use Doctrine\ODM\MongoDB\Mapping\Annotations as ODM;
use JMS\Serializer\Annotation as JMS;
use Gedmo\Mapping\Annotation as Gedmo;

/**
 * UpdateCreateTraitTrait
 *
 * @author Ivan Proskuryakov <volgodark@gmail.com>
 *
 */
trait UpdateCreateTrait
{

    /**
     * @var \DateTime
     * @ODM\Field(type="date")
     * @Gedmo\Timestampable(on="create")
     * @JMS\Expose
     * @JMS\Type("DateTime")
     */
    protected $createdAt;

    /**
     * @var \DateTime
     * @ODM\Field(type="date")
     * @Gedmo\Timestampable(on="update")
     * @JMS\Expose
     * @JMS\Type("DateTime")
     */
    protected $updatedAt;

    /**
     * Get createdAt
     *
     * @return \DateTime
     */
    public function getCreatedAt()
    {
        return $this->createdAt;
    }

    /**
     * Get updatedAt
     *
     * @return \DateTime
     */
    public function getUpdatedAt()
    {
        return $this->updatedAt;
    }

}

{% endhighlight %}