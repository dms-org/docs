DMS Developer Documentation
===========================

DMS is PHP7 application framework that makes it quick and easy to build maintainable and robust apps of any scale.

 - Integrates with Laravel
 - Promotes separation of concerns and defines clear architectural boundaries
 - Build strongly typed, rich or anemic domain models to suit your application
 - Fully-featured data mapper ORM
 - Provides an integrated and powerful CMS/Backend framework
 - Reusuable functionality can be installed via composer packages 

Dashboard
---------
![Dashboard](/resources/images/cms/dashboard-1.jpg)

Overview
---------
![Overview](/resources/images/cms/overview-1.jpg)

Edit
---------
![Edit](/resources/images/cms/edit-1.jpg)

Code
----

```php
<?php declare(strict_types = 1);

namespace Dms\Package\Shop\Domain\Entities\Order;

use Dms\Common\Structure\DateTime\DateTime;
use Dms\Common\Structure\Money\Money;
use Dms\Core\Model\Object\ClassDefinition;
use Dms\Core\Model\Object\Entity;
use Dms\Core\Model\ValueObjectCollection;
use Dms\Core\Util\IClock;
use Dms\Package\PaymentGateway\Gateway\Response\TransactionReference;
use Dms\Package\Shop\Coupon\Core\Coupon;
use Dms\Package\Shop\Domain\Entities\Pricing\PricingType;
use Dms\Package\Shop\Domain\Exception\ShopException;

/**
 * @author Elliot Levin <elliotlevin@hotmail.com>
 */
class Order extends Entity
{
    const ORDER_NUMBER = 'orderNumber';
    const STATUS = 'status';
    const CUSTOMER_DETAILS = 'customerDetails';
    const PRICING_TYPE = 'pricingType';
    const ITEMS = 'items';
    const DISCOUNT = 'discount';
    const SHIPPING_DETAILS = 'shippingDetails';
    const TAX_FEE = 'taxFee';
    const COUPON = 'coupon';
    const NOTES = 'notes';
    const TRANSACTION_REFERENCE = 'transactionReference';
    const AMOUNT_REFUNDED = 'amountRefunded';
    const CREATED_AT = 'createdAt';
    const METADATA = 'metadata';

    /**
     * @var string
     */
    public $orderNumber;

    /**
     * @var OrderStatus
     */
    public $status;

    /**
     * @var CustomerDetails
     */
    public $customerDetails;

    /**
     * @var PricingType
     */
    public $pricingType;

    /**
     * @var ValueObjectCollection|OrderItem[]
     */
    public $items;

    /**
     * @var Money|null
     */
    public $discount;

    /**
     * @var ShippingDetails|null
     */
    public $shippingDetails;

    /**
     * @var Money|null
     */
    public $taxFee;

    /**
     * @var Coupon|null
     */
    public $coupon;

    /**
     * @var DateTime
     */
    public $createdAt;

    /**
     * @var TransactionReference|null
     */
    public $transactionReference;

    /**
     * @var Money
     */
    public $amountRefunded;

    /**
     * @var ValueObjectCollection|OrderNote[]
     */
    public $notes;

    /**
     * @var \ArrayObject
     */
    public $metadata;

    /**
     * Order constructor.
     *
     * @param string                    $orderNumber
     * @param OrderStatus               $status
     * @param CustomerDetails           $customerDetails
     * @param IClock                    $clock
     * @param PricingType               $pricingType
     * @param OrderItem[]               $items
     * @param Money|null                $discount
     * @param ShippingDetails           $shippingDetails
     * @param Money|null                $taxFee
     * @param Coupon|null               $coupon
     * @param TransactionReference|null $transactionReference
     * @param array                     $metadata
     */
    public function __construct(
        string $orderNumber,
        OrderStatus $status,
        CustomerDetails $customerDetails,
        IClock $clock,
        PricingType $pricingType,
        array $items,
        Money $discount = null,
        ShippingDetails $shippingDetails = null,
        Money $taxFee = null,
        Coupon $coupon = null,
        TransactionReference $transactionReference = null,
        array $metadata = []
    ) {
        parent::__construct();
        $this->orderNumber          = $orderNumber;
        $this->status               = $status;
        $this->customerDetails      = $customerDetails;
        $this->pricingType          = $pricingType;
        $this->items                = OrderItem::collection($items);
        $this->discount             = $discount;
        $this->shippingDetails      = $shippingDetails;
        $this->taxFee               = $taxFee;
        $this->coupon               = $coupon;
        $this->transactionReference = $transactionReference;
        $this->amountRefunded       = new Money(0, $pricingType->currency);
        $this->notes                = OrderNote::collection();
        $this->createdAt            = new DateTime($clock->utcNow());
        $this->metadata             = new \ArrayObject($metadata);
    }

    /**
     * Defines the structure of this entity.
     *
     * @param ClassDefinition $class
     */
    protected function defineEntity(ClassDefinition $class)
    {
        $class->property($this->orderNumber)->asString();

        $class->property($this->status)->asObject(OrderStatus::class);

        $class->property($this->customerDetails)->asObject(CustomerDetails::class);

        $class->property($this->pricingType)->asObject(PricingType::class);

        $class->property($this->items)->asType(OrderItem::collectionType());

        $class->property($this->shippingDetails)->nullable()->asObject(ShippingDetails::class);

        $class->property($this->discount)->nullable()->asObject(Money::class);

        $class->property($this->taxFee)->nullable()->asObject(Money::class);

        $class->property($this->coupon)->nullable()->asObject(Coupon::class);

        $class->property($this->transactionReference)->nullable()->asObject(TransactionReference::class);

        $class->property($this->amountRefunded)->asObject(Money::class);

        $class->property($this->notes)->asType(OrderNote::collectionType());

        $class->property($this->createdAt)->asObject(DateTime::class);

        $class->property($this->metadata)->asObject(\ArrayObject::class);
    }

    /**
     * @return Money
     */
    public function getSubtotal() : Money
    {
        $amountOfMoney = new Money(0, $this->pricingType->currency);

        foreach ($this->items as $item) {
            /** @var OrderItem $item */
            $amountOfMoney = $amountOfMoney->add($item->getTotalPrice());
        }

        return $amountOfMoney;
    }

    /**
     * @return Money
     */
    public function getTotal() : Money
    {
        $amountOfMoney = $this->getSubtotal();

        if ($this->discount) {
            $amountOfMoney = $amountOfMoney->subtract($this->discount);
        }

        if ($this->shippingDetails && $this->shippingDetails->shippingFee) {
            $amountOfMoney = $amountOfMoney->add($this->shippingDetails->shippingFee);
        }

        if ($this->taxFee) {
            $amountOfMoney = $amountOfMoney->add($this->taxFee);
        }

        return $amountOfMoney;
    }

    /**
     * @return bool
     */
    public function requiresShipping() : bool
    {
        return $this->items->any(function (OrderItem $item) {
            return $item->requiresShipping;
        });
    }

    /**
     * @return Money
     * @throws ShopException
     */
    public function getMaxRefundAmount() : Money
    {
        if (!$this->transactionReference) {
            throw new ShopException('The order has no transaction');
        }

        return $this->transactionReference->amount->subtract($this->amountRefunded);
    }
}
```

User Guide
==========

```eval_rst
.. toctree::
    :maxdepth: 3

    docs/prologue
    docs/getting-started
    docs/your-first-app
    docs/scaffolding
    docs/repositories
```