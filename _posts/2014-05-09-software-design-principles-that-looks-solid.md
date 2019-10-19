---
title        : "Software Design Principles - That Looks S.O.L.I.D"
date         : "2014-05-09"
tags         : [Design Principles, PHP, SOLID]
---

This is the first post on Software Design Principles, I plan to cover all the
Design Principles that I've had personal experiences with, so that this series
will not be and should not be used as tutorials or introductions to those
principles for that I'll mostly be approaching them based on my personal (and
very likely biased) opinions and experiences. However, I'll try to give a brief
introduction to each concept as I go. Please consider reading about these
principles from myriad good sources online before you read my posts.

Design Principles, differ from Design Patterns, are generalized high level
software design guidelines and rules of thumbs for us to approach some of the
common design problems, they focus on conceptual designs, not the actual
implementation. On the other hand, Design Patterns focus on low level
implementation by providing reusable solutions to common programming problems.

For example, the Design Principle we are going to discuss today, S.O.L.I.D,
states some high level guidelines we should keep in mind when creating a class
or designing a member function, while Design Patterns such as the singleton
pattern, provides detailed steps to follow in order to create classes that can
only be instantiated once.

To be frank, I have the ambition to cover all Design Principles, Design
Patterns, Development Methodologies such as Extreme Programming, Architectural
Patterns like CQRS or MVC, and Development Philosophies like Agile Software
Development and Domain Driven Development. I don't know if I'll have the time
and energy to do all these, but since I like making plans and to plan far ahead,
I'm actually approaching all these topics with an order.

And it's a descending order by the relative effectiveness in improving a
development team's code quality. For example, we talked about Clean Code in our
five-installment series Clean Code Tips and Tricks first because I think it is
the most effective, almost all of the teams I worked with showed improvements in
code quality in a matter of weeks. And more importantly, it's general, it can be
and should be adopted by every development team. Architectural Patterns on the
other hand, are great, like MVC and it's variants, and are widely adopted by
development teams, but the problem is that teams adopt different Architectural
Patterns thus they are more of a subjective topic and are less general.

I'm approaching these topics in this way in hope that even if I couldn't find
the time or energy to cover everything, my posts would've already been
beneficial to my readers.

## S.O.L.I.D Principle

If you've read my blog series on Clean Code, which I recommend you do if you
haven't, we talked about why and how we should focus on writing code that are
easy on the eyes and enjoyable to work with by following some tips and
guidelines. Such a great way to improve your code quality in a short
period of time is often underrated and not getting enough attention. I think
Clean Code should be adopted by every development team and programmer to start
writing clean, beautiful, and elegant code.

Now let's move on to S.O.L.I.D principle. The reason that I chose to discuss
S.O.L.I.D first before all the other software design principles is that I think
it is the most effective in terms of improving maintainability, reusability, and
extensibility of your codebase. I think it is also the most fundamental
principle that is so influential that it is now standard, a default principle
which I think, again, should be adopted by every development team as long as you
work with an Object Oriented Programming Language.

### Single Responsibility Principle

> A class should have one, and only one, reason to change.
> ~ Uncle Bob in his [Priciples of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

A class should focus on a single set of ***cohesive*** tasks.

By cohesive, it means the member functions are highly interdependent and
interconnected, they work together to define the characteristic of the class.

Cohesion also means collaboration. Think about development teams in an IT
company, you don't often come across a front-end developer in a database team,
or a marketing person in a backend team who attends every sprint meeting. You
are a unit who does something well, and you team works together to do
something bigger well, and your whole development department collaborates
together to deliver a really complex system, then all the departments in your
company strive to make the business profitable.

It's exactly the same thing in writing source code, every line of code in a
method works together to define the method, every method in a class serves
their purpose and create sense for the class, every class in a module fit
together to form a module, and every module work together as a system.

I'd like to point out that this sounds similar to our ***one thing rule*** for
functions, except that it says functions should do ***one thing***, and classes
that follow SRP should have ***one responsibility***, or ***one reason to
change***. Now first of all, we should not get confused, I've seen people
claiming on the Internet that a class should do one thing and one thing only.
That sounds like a horrible disaster, having all the classes doing only one
particular *thing* is unimaginable.

Secondly, they do have many similarities. These two principles all try to
extract out excessive code that does not belong to the code unit to achieve
better maintainability, reduce coupling and improve cohesion. Now to save us
from unnecessary semantic debates, let's just agree on that ***one thing*** is
literally ***one particular task to achieve***, and ***one responsibility***
means a set of interdependent and interconnected jobs that can't and shouldn't
be divided into smaller groups. A class that has a single responsibility
indicates that there's only one reason to change the class, no matter it's
fixing a bug or adding a new feature, we do it all under the scope of this
particular responsibility.

However, these similarities give us a hint and we can practically use a similar
set of steps to refactor a class to let it comply with SRP -- We extract
excessive functions and properties and put them in other classes until we
couldn't extract anything more out of it and give the extracted code a meaningful
class name that is different from the original one.

Keep in mind this is kind of an extreme method, we do have to consider business
logic and domain requirements in the process. Nevertheless, this should be a
good way to get you started.

Another practical method comes from our last installment of Clean Code Tips and
Tricks, where we discussed a way to detect if a class is not following SRP. We
had a class called `DataFileSplitterAndProcessor`, it actually has two
responsibilities, splitting and processing data files, thus there are two
reasons to change the class -- Maintaining the splitting functionalities, and
maintaining the processing functionalities, which violates the SRP.

``` php
/*
 * A simple trick to tell if a class follows
 * SRP is to try to describe it as descriptive as you can.
 */

class EmailService {
    protected $invoice;

    function __construct(InvoiceRepository $invoice) {
        $this->invoice = $invoice;
    }

    private function calculateTax() {
        return $this->invoice->subtotal * 0.025;
    }

    public function emailInvoiceToUser(UserRepository $user) {
        /* Code to generate email message */
        if (!$emailHasSentSuccessfully)
            throw new EmailDeliveryFailedException();

        return true;
    }
}
```

The above code shows a very poorly designed class, if you try your best to
verbosely describe the class, it's one that "Calculates the tax and generates
invoice email and sends email to a user". You may as well name the class as
***TaxAndInvoiceAndEmailService***. In case you haven't noticed,
*calculateTax()* should belong to Invoice class, by calculating tax here in
*EmailService* breaks encapsulation.

This is how this simple trick works, if you ever see a class that should be
named with ***and*** in it's name, then it's most probably not following the
SRP.

You may argue that your business logic is such that an email would only be
generated and sent to the user in the case of sending invoices, it makes sense
to congregate these methods into one class.

That maybe true to some extend, but having separate ***EmailService***,
***InvoiceService*** and ***TaxService*** makes it much more clearer and easier
to extend, what if a wild *sending coupon expiring warning emails* or a
*applying discounts to invoices exceeds certain amount in subtotal* requirement
appears in the future? What if such requirements keep comming and eventually
make this class too complex and coupled to work with?

### Open-Closed Principle

> You should be able to extend a class’s behavior, without modifying it.
> ~ Robert C. Martin

A class should be open to extension, meaning a new behavior, functionality or
property, but should be closed to modification, no extension should result in
modifying of the existing code. Unless, of course, when you are fixing a bug.

``` php
class Product {
    protected  $name;
    protected  $price;

    public funcion __construct($name, price) {
        $this->name = $name;
        $this->price = $price;
    }

    public function displayName() {
        return $this->name;
    }

    public function displayPrice() {
        return $this->price;
    }

    public function updatePrice($new_price) {
        return $this->price = $new_price;
    }

    public function displayLabel() {
        return $this->name . ': $' . $this->price;
    }
}
```

Imagain we have a Product class shown above, now our client wants a new feature,
add a new type of product called promotional products each have their own
discount rate, but their discounted prices are only available on 11th of
November, the Chinese version of boxing day. Now, a conventional way to
implement the new feature would be as shown below,

``` php
class Product {

    protected $name;
    protected $price;
    protected $discount_rate; //<-

    public function __construct($name, $price, $discount_rate = NULL) {//<-
        $this->name = $name;
        $this->price = $price;
        $this->discount_rate = $discount_rate; //<-
    }

    ...
    ...

    public function displayDiscountRate() { //<-
        return $this->discount_rate;
    }

    public function updateDiscountRate($new_discount_rate) { //<-
        return $this->discount_rate = $new_discount_rate; //<-
    } //<-

    public function displayLabel() { //<-
        if (!isset($this->discount_rate) || NOTTODAY()) {
            return $this->name . ': $' . $this->price;
        }

        return $this->name .
               ': Full Price $' .
               $this->price .
               ' Discounted Price $'.
               $this->price * $this->discount_rate;
    }
}
```

To accomodate this new requirement, we had to add a property named
`$discount_rate`, and we had to modify the class constructor and `displayLabel`
function which is a bad idea that may lead to bugs and maintainability issues.

Note that we've talked about getting rid of class conditional switches in our
Clean Code Tips and Tricks: Classes post, you can observe a similar conditional
in the example code above as it checks if the instance has *discount_rate* set
and then performs actions accordingly. Our suggestion then was to favor
inheritance over conditionals in classes, Open-Closed Principle suggests the
same thing.

BTW, utilizing NULL is in most cases a bad idea, it causes null pointer
exceptions and may potentially lead to confusing bugs.

``` php
class Product {
    /* Original Product Class */
}

class PromotionalProduct extends Product {

    protected $discount_rate;

    public function __construct($name, $price, $discount_rate = 1) {
        $this->name = $name;
        $this->price = $price;
        $this->discount_rate = $discount_rate;
    }

    public function displayDiscountRate() {
        return $this->discount_rate;
    }

    public function updateDiscountRate($new_discount_rate) {
        return $this->discount_rate = $new_discount_rate;
    }

    public function displayLabel() {
        if (NOTTODAY()) {
            return $this->name . ': $' . $this->price;
        }

        return $this->name .
               ': Full Price $' .
               $this->price .
               ' Discounted Price $'.
               $this->price * $this->discount_rate;
    }
}
```

We did it without modifying the original Product class. By creating a new class
called `PromotionalProduct` that extends `Product`, we inherited all member
properties and methods, and by overwriting `displayLabel()` function's behavior, we
fullfiled the requirement, and we got rid of the conditional check since a
promotional product should always have a discount rate even if it's 1.

This looks better, but not yet bullet-proof, we'll refine this code more later.
Now this new additon may have bugs and issues, but we saved ourselves from
creating problems in existing codebase by modifying it, which may potentially
lead to much more confusing issues.

Another thing to note is that interfaces and inheritance are all viable ways to
achieve abstraction and polymorphism, choosing which one to use is out of the
scope of this post.

Plus the choice depends heavily on which programming language you use. For PHP,
inheritance is a way to share similar behavior, traits can be used to share code
among classes that need the same piece of code but share nothing else in common,
and interfaces provide a way to define common protocols.

### Liskov Substitution Principle

If Open-Closed Principle tells us how to avoid modifying existing code to
improve maintainability by abstraction, the Liskov Substitution Principle tells
us how to utilize abstraction ***correctly***.

> If for each object o1 of type S there is an object o2 of type T such that for
> all programs P defined in terms of T, the behavior of P is unchanged when o1
> is substituted for o2 then S is a subtype of T.
> ~ Barbara Liskov

What we want to accomplish is that if a class A is a subtype of B, our program
should remain unaffected if we replace object b of Class B with an object a of
class A.

``` php
class Invoice {
    ...
    ...

    public function calculateSubtotal($order) {
        $subtotal = 0;
        foreach ($order as $entry) {
            $subtotal +=
                $entry->product->displayPrice() * $entry->getQuantity();
        }

        return $subtotal;
    }
}
```

Say we have an `Invoice` class in our system, in conjuction with `Product` and
`PromotionalProduct` classes we wrote in our last example, we can calculate the
subtotal of a given order like this.

However, as you may have realized, this method should work fine before we added
`PromotionalProduct`, but the programmer who coded this didn't foresee the
comming requirement additon, thus it fails at calculating the correct subtotal
when we supply `$order` with at least one instance of `PromotionalProduct` in
it.

``` php
class Invoice {
    ...
    ...

    public function calculateSubtotal($order) {
        $subtotal = 0
        foreach ($order as entry) {
            if ($entry->product instanceof PromotionalProduct
                &&
                TODAY === BOXINGDAY) {

                $subtotal +=
                    $entry->product->displayPrice() * $entry->product->displayDiscountRate() * $entry->getQuantity();
            } else {
                $subtotal += $entry->product->displayPrice() * $entry->getQuantity();
            }
        }

        return $subtotal;
    }
}
```

Looks like we did it, by differentiating which class the `$product` is an
instance of, we perform calculations accordingly.

But still, this is a direct violation of Liskov Substitution Principle, we
substituted objects of `Product` class with objects of class
`PromotionalProduct` that extends `Product`, but our program was affected by
this substitution, we had to change its behavior to avoid bugs.

LSP implies that our program should remain ignorant of such substitution, it
shouldn't care if the object is of the base class, or the derived class.

If you ever come across a piece of code like this, performing actions according
to which class the object is an instance of, and the classes in the context
implements abstraction (interface or inheritance), it's a clear sign that you
may have violated LSP. This piece of code most probably have the knowledge of
every possible derivatives of the base class and need to be updated everytime we
add a new type of variant.

Some may argue if we modify the `displayPrice()` getter in the derived class
`PromotionalProduct` and let it return full prices and discounted prices
accordingly, then we don't have to know the object type in our `Invoice` class,
we can calculate the subtotal simply by calling each object's `displayPrice()`
function.

That is true and viable, as long as we have the confidence that every derived
class of the base class implements a displayPrice() properly, we don't have to
check the object type anymore. Doesn't this sound familiar? Yes, Interface.

``` php
interface Product {
    public function displayLabel();
    public function displayPrice();
}

class NormalProduct implements Product {
    private $name;
    private $price;

    ...

    public function displayLabel() {
        return $this->name . ': $' . $this->displayPrice();
    }

    public function displayPrice() {
        return $this->price;
    }
}

class PromotionalProduct implements Product {
    private $name;
    private $price;
    private $discount_rate;

    ...

    public function displayLabel() {
        return $this->name . ': $' . $this->displayPrice();
    }

    public function displayPrice() {
        if (TODAY != BOXINGDAY) {
            return $this->price;
        }

        return $this->price * $this->discount_rate;
    }
}

class Invoice {
    ...
    ...

    public function calculateSubTotal($order) {
        $subtotal = 0;
        foreach ($order as $entry) {
            $subtotal += $entry->product->displayPrice() * $entry->getQuantity();
        }

        return $subtotal;
    }
}
```

By implementing a common interface, we defined common protocols that both
*NormalProduct* and *PromotionalProduct* should comply, but they can implement
the function according to their specialized business logics. Now that our
*Invoice* class can confidently call *displayPrice()* on any object that
complies with *Product* interface.

Of course, if *Product* and *PromotionalProduct* shares a lot in common, we
could always use inheritance other than interface.

``` php
abstract class Product {
    private $name;
    private $price;

    ...

    abstract function displayPrice();

    public function displayLabel() {
        return $this->name . ': $' . $this->displayPrice();
    }
}

class NormalProduct extends Product {
    ...

    public function displayPrice() {
        return $this->price;
    }
}

class PromotionalProduct extends Product {
    ...

    public function displayPrice() {
        if (TODAY != BOXINGDAY) {
            return $this->price;
        }

        return $this->price * $this->discount_rate;
    }
}
```

Let's extend the example a bit to further visualize the topic. Let's ignore the
possibly flawed domain logic and just say that each product hold its own tax
rate and shipping cost which would be used when generating an invoice.
The catch here, is that products purchased with promotional price can't be
shipped outside of the country but normal products can, and to further encourage
sales, all promotional products on 11th of November are tax-free.

``` php
abstract class Product {
    private $name;
    private $price;
    private $weight;
    private $size;
    private $tax_rate;

    ...
    ...

    public function calculateShippingCost($province) {
        /*
         * Code to calculate shipping costs
         * according to provice, weight and size
         */
    }

    public function updateTaxRate($new_tax_rate) {
        $this->tax_rate = $new_tax_rate
    }

    public function getTaxRate() {
        return $this->tax_rate;
    }
}

class NormalProduct extends Product {
    ...
    ...

    /*
     * Nothing to overwrite
     * raleting to tax and shipping cost
     */
}

class PromotionalProduct extends Product {
    ...
    ...

    public function calculateShippingCost($province) {
        // Promitonal products can't be shipped outside the country on boxing day.
        if (!in_array($province, $domesticProvinces) && TODAY === BOXINGDAY) {
            throw new InvalidShippingProvinceException();
        }

        /* Calculate shipping cost normally */
    }

    public function getTaxRate() {
        if (TODAY === BOXINGDAY) {
            return 0;
        }

        return $this->tax_rate;
    }
}
```

We added a new property called `$tax`, a function to calculate shipping cost and
to display tax rate in the base class, then we inherited the base class in
*NormalProduct* class without any overwriting. In *PromotionalProduct* class
however, we overwrote `calculateShippingCost()` and `getTax()` functions to
reflect its specified requirements.

Now this looks perfect, what could ever go wrong? Let's take a look at our
`Invoice` class, which is a client (or user) of our base class `Product`.

``` php
class Invoice {

    ...
    ...

    public function calculateTotalShippingCosts($order) {
        $totalShippingCosts = 0;
        foreach ($order as $entry) {
            $totalShippingCosts += $entry->product->calculateShippingCost($order->address->province);
        }

        return $totalShippingCosts;
    }

    public function calculateTaxCredit($order) {
        $totalTaxCredit = 0;
        foreach ($order as $entry) {
            switch ($entry->product->getTax()) {
                case 0.5:
                    $totalTaxCredit += 1;
                    break;
                case 1.5:
                    $totalTaxCredit += 2;
                    break;
                case 2.5:
                    $totalTaxCredit += 3;
                    break;
                default:
                    throw new UnexpectedTaxValueExeception($entry->product);
                    break;
            }
        }

        return $totalTaxCredit;
    }
    ...
    ...
}
```

We have a `calculateTotalShippingCosts()` and a `calculateTaxCredit()` function
here which grants users credit according to the tax rate of the product they
purchased, this was intended as a virtual currency for tax-refund purposes.

Same as the last time, after we added the `PromotionalProduct` derived class,
these two functions in `Invoice` class broke. When there is at least one
instance of  `PromotionalProduct` in `$order`, `calculateShippingCosts()` would
sometimes throw an exception when it's boxing day and someone is purchasing
promotional product from overseas, this exception was never expected when
writing the `Invoice` class. While the `calculateTaxCredit()` function would
simply throw an error on boxing day since we didn't know any product could have
a tax rate of zero.

This is an illustration of both precondition and postcondition violation.

``` php
public function calculateShippingCost($province) {
    if (!in_array($province, $domesticProvinces) && TODAY === BOXINGDAY) {
        throw new InvalidShippingProvinceException();
    }

    / Calculate shipping cost normally */
}
```

Look at this function in our `PromotionalProduct` class, what we did wrong here
is that we made the precondition of the original `getShippingCost($province)`
function in abstract class `Product` ***stronger***, the original function
accepts any province or states in the world and yields shipping cost
accordingly, but in `PromotionalProduct` it only accepts domestic provinces on
boxing day, thus we strengthened the precondition, causing function callings in
clients or users of the base class to potentially fail since they passed the now
considered ***weaker*** parameter to our overwritten function.

``` php
public function getTaxRate() {
    if (TODAY === BOXINGDAY) {
        return 0;
    }

    return $this->tax_rate;
}
```

The `getTax()` function in our `PromotionalProduct` class on the other hand, by
adding one more possible return value, zero in this case, we made the
postcondition ***weaker***. This caused clients or users of the `Product` class
getting unexpected return values from instances of `PromotionalProduct`.

These all contributed to the unsuccessful substitution, the client of the base
class cannot simply work with the newly extended derivative properly.

> …when redefining a routine [in a derivative], you may only replace its
> precondition by a weaker one, and its postcondition by a stronger one.
> ~ Bertrand Meyer

To fix this problem, we have to strengthen the precondition in the base class
thus the extending class won't have a stronger one, and weaken the postcondition
so that the extending class won't have a weaker one.

``` php
abstract class Product {
    ...
    ...

    public function canShipOverseas(){
        return true;
    }

    public function calculateShippingCost($province) {
        if (!$this->canShipOverSeas()) {
            thow new InvalidShippingProvinceException();
        }

        /* As usual */
    }
}

class NormalProduct extends Product {
    ...
    ...

    /* Nothing as well */
}

class PromotionalProduct extends Product {
    ...
    ...

    public function canShipOverseas() {
        if (!in_array($province, $domesticProvinces) && TODAY === BOXINGDAY) {
            return false;
        }
    }

    /* Don't have to overwrite calculateShippingcost() this time */
}

class Invoice {
    ...
    ...

    public function calculateTaxCredit($order) {
        $totalTaxCredit = 0;
        foreach ($order as $entry) {
            switch ($entry->product->getTax()) {
                case 0:
                    $totalTaxCredit += 0;
                case 0.5:
                    $totalTaxCredit += 1;
                    break;
                case 1.5:
                    $totalTaxCredit += 2;
                    break;
                case 2.5:
                    $totalTaxCredit += 3;
                    break;
                default:
                    throw new UnexpectedTaxValueExeception($entry->product);
                    break;
            }
        }

        return $totalTaxCredit;
    }
}
```

As shown in the example code above, we added a *canShipOverseas()* function and
checks if it's true in the *calculateShippingCost()* function. Placing them all
in the base abstract class enforces the precondition so that it is possible to
raise exception on boxing day when some customer places an order from  overseas.

To fix the problem in *getTaxRate()* function is fairly easy, we just need to
acknowledge that the postcondition is that it may return zero, and just reflect
that in the *calculateTaxCredit()* function. Note that inheritance alone cannot
be a violation to LSP, there has to be at least one client program in the
context which would broke if an object of the base class was replaced by an
object of the subclass. If all client programs are able to perform the
substitution with unchanged behavior, then the code is compliant with LSP.

Another thing to keep in mind if you are confronted with a base class and it's
derived class that violates LSP and you feel there's no elegant way to make them
compliant, there's a good chance that the subclass should not inherit the base
class, they should be two different classes. This scenario is covered in the
famous Rectangle-Square problem, refer to it if you are interested to know more.

### Interface Segregation Principle

To put it simply, we split interfaces into smaller ones so that their
implementing classes wouldn't be forced to implement methods they don't want.

``` php
interface Faculty {
    public function teachClass($class);

    public function conductTutorial($tutorial);

    public function markAssignment($assignment);
}
```

Say we have a `Faculty` interface as shown above, now a lecture should teach
classes and mark assignments, but what if our business logic dictates that only
teaching assistant can conduct tutorials? By having our `Lecture` class
implementing this interface, we force it to conduct tutorials.

``` php
interface Teacher {
    public function teachClass($class);
}

interface Tutor {
    public function conductTutorial($tutorial);
}

interface Marker {
    public function markAssignment($assignment);
}

class Lecture implements Teacher, Marker {
    public function teachClass($class) {
    }

    public function markAssignment($assignment) {
    }
}

class TeachingAssistant implements Tutor, Marker {
    public function conductTutorial($tutorial) {
    }

    public function markAssignment($assignment) {
    }
}
```

Now this looks much more reasonable, having derived classes only implement
methods they are interested in decouples them with undesired responsibilities.

### Dependency Inversion Principle

Let's begin with an example.

``` php
class Email {
    public function send() {
        /* code to send email */
    }
}

class PromotionNotification {
    protected $email;

    function __construct(Email $email) {
        $this->email = $email;
    }

    public function send() {
        $this->email->send();
    }
}
```

We say that `PromotionNotification` class is tightly coupled with `Email` class
since it directly depends on it and knows it's business logic. The problem with
coupling is if we decide to add another way to send promotional messages,
through SMS for example, we'd have to go back to our `PromotionNotification`
class to change it's implementation, thus violating Open-Closed Principle.

Dependency Inversion Principle helps us to decouple our codebase. Note that DIP
and Dependency Injection are essentially two things, but you can view DI as a
means to achieve DIP. We'll talk about this later.

> Depend on abstractions, not on concretions.
> ...
> High level modules should not depend upon low level modules. Both should depend upon abstractions.
> ...
> ~ Robert C. Martin

So instead of letting `PromotionNotification` class depend on concrete `Email`
class, let's have it depend on an abstraction of it.

``` php
interface MessageService {
    public function send();
}

class Email implements MessageService {
    public function send() {
        /* Code to send email message */
    }
}

class SMS implements MessageService {
    public function send() {
        /* Code to send SMS message */
    }
}

class PromotionNotification {
    protected $messageService;

    function __construct(MessageService $messageService) {
        $this->messageService = $messageService;
    }

    public function send() {
        $this->messageService->send();
    }
}
```

Here we added an abstraction interface called `MessageService` and let both our
`Email` and `SMS` class to implement it. Now our `PromotionNotification` depends
on `MessageService` and by protocol defined, we know any derived class should
have a `send()` function implemented.

We are now complaint with DIP since we now don't depend on any concrete class,
and we are free to switch `MessageService` with any of it's implementing class
without having to modify existing code, or affect our system's behavior.

``` php
...

$promotionNotification = new PromotionNotification(new Email());

...

$promotionNotification->send();
```

This dependency injection technique is called the Contructor Injection, we
inject dependency when creating an object of `PromotionNotification` class. If
we'd like to specify the dependency as we call `send()` function, we can switch
to Method Injection.

``` php
class PromotionNotification {
    public function sendVia(MessageService $messageService) {
        $messageService->send();
    }
}
```

Note that `MessageService` has been moved out of the constructor and in the
`sendVia()` function's signature. Now we can switch implementing classes on the
fly.

``` php
$promotionNotification = new PromotionNotification();

...

$promotionNotification->sendVia(new Email());

$promotionNotification->sendVia(new SMS());
```

Choose either method according to your requirements.

You did it! This is the end of our discussion on S.O.L.I.D principle, a powerful
and well recognized design principle that I think should be adopted by every
project. We talked about each of the five principles, discussed some examples
, common misconceptions, and stressed some topics which I think are not getting
enough attention from sources I've read online. I hope you enjoyed it, and
looking forward to seeing you in the next installment on Software Design
Principles.
