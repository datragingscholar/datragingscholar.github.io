---
title        : "Clean Code Tips and Tricks: Classes"
date         : "2014-04-15"
tags         : [Clean Code, PHP, Programming Guideline]
---

Welcome back to yet another post on Clean Code Tips and Tricks. We've covered
variables and functions so far, be sure to check these posts out first if you
haven't.

## Naming Classes

We've covered tips and guidelines on naming variables and functions, many of the
rules should apply to classes as well, keep in mind that naming things properly
is like documenting our codebase and provide valuable information to your
readers.

### Use Noun

A class is an abstraction of an entity with a collection of behaviors that can
be instantiated into objects, thus class names should be nouns, not verbs, not
any kind of statements.

``` php
// These are all silly class names that make no sense at all.
class moveStudent {}
class shouldSendEmail {}
class LogShouldArchive {}
class Swim {}
class Jump {}
```

### Be Reflective

Reflect the domain this class lives in or the architectural design your team
adopts. Stick with some conventions instead of inventing your own terms to avoid
confusions.

``` php
// Reflect the terms when you use factory or repository pattern
class UserFactory {}
class CouponRepository {}
```

### Be Verbose

Not quite like variables and functions, we verbosely name our classes for some
other reasons. One being each class represents an abstraction of entity, and
sometimes the same entity may evolve and become conceptually two entities and
should be abstracted into two classes. Naming them verbosely and reflect their
differences helps avoiding ambiguity.

The length a class name should have follows the same rule as functions do. The
more the class is used throughout the system, meaning the wider the scope it
lives in, the shorter and more concise it's name should be. And of course, the
narrower the scope, the longer and more specific you should name them.

## Tell Don't Ask

Tell don't ask is a well known principle that helps us to bundle data and
behaviors together rather than separating them. If we don't combine data and
actions together, we risk breaking encapsulation which is one of the core value
of object oriented programming.

``` php
class Wallet {
    private $customer;
    private $balance;

    public function __construct(Customer $customer, $balance = 0)
        $this->customer = $customer;
        $this->balance = $balance;
    }

    public function getBalance() {
        return $this->balance;
    }

    public function setBalance($value) {
        return $this->balance = $value;
    }
}
```

Take a look at this very simple Wallet class, it has some properties, a
straightforward constructor and a setter and a getter. Let's discuss what we
can do to make it better.

### Don't Use Setters

A class consists of properties and behaviors, if we overlook it's behaviors and
instead changing it's properties directly outside the class, we practically
took the business logic away from the class.

With a setter like *setBalance()*, the use case would contain too much business
logic that actually belongs to Wallet class. Look at the example code below,

``` php
class Order {
    public function process() {
        // Validate order...
        // Calculate subtotal...

        // Deduct balance
        $wallet = $this->customer->wallet;
        $balance = $wallet->getBalance();
        if ($balance < $subtotal) {
            throw new insufficientBalanceException();
        }

        $wallet->setBalance($balance - $subtotal);
    }
}
```

Checking if the wallet has sufficient balance and deduct from it should be a
business logic belonging to Wallet class, and the order process function is
clearly doing more than it should. Let's revise it to properly encapsulate
business logic.

``` php
class Wallet {
    ...

    public function deductBalance($value) {
        if ($this->balance < $value) {
            throw new insufficientBalanceException();
        }

        return $this->balance -= $value;
    }

    public function addBalance($value) {
        return $this->balance += value;
    }

    ...
}

```

We moved the sufficient balance checking and exception throwing back to Wallet
class and named the function *deductBalance* to reflect it's business logic. We
also added a new function called *addBalance* for use cases that adds fund to a
customer's wallet. Their names can be further improved according to the domain
language of this project, for example, *deductFee($amount)* and
*refund($amount)*. It is even advisable to have another setter function exactly
same as *refund($amount)* but named as *deposit($amount)* to separate two
different use cases if that's what the business process requires.

By doing this, we made sure that a state of an object should never be changed
without business logic, now there's no way of changing the wallet balance
without invoking strictly defined business logics.

Even a simple setter such as *setLastActivity($datetime)* that only receives the
value and changes the state, should not be named as a setter but also to reflect
the business logic it implies.

### Try to Avoid Getters

Not all getters are evil. We should try to write less getters because we often
abuse them by providing a way to leak data outside the class and perform
business logics with them. And again, those logics should be bond to the states
and stay inside the class to achieve encapsulation.

Let's say we are celebrating 10-year anniversary of our ecommerce system, and
anyone with a floor-rounded balance of 2003 -- our founding year, will get a 50%
off coupon. The code is shown below,

``` php
/*
 * The marketing team said this would encourage them
 * to topup, or spend more depending on how thick their
 * wallets are.
 */
public function grantAnniversaryCoupon() {
    foreach($customers as $customer) {
        $balance = $customer->wallet->getBanlance();
        $floor_balance = floor($balance);

        if ($floor_balance === 2003) {
            // Grant coupon.
        }
    }
}
```

Now the problem is that checking if the floor-rounded value of a customer's
wallet equals to 2003 and perform business logics accordingly is
grantAnniversaryCoupon() function's logic, but getting the actual *floor-rounded
value of your wallet balance* is strictly a wallet's own business, no one else's.

``` php
class Wallet {
    ...

    public function getBalance() {
        return $this->balance;
    }

    public function getFloorRoundedBalance() {
        return floor($this->balance);
    }

    ...
}

public function grantAnniversaryCoupon() {
    foreach($customers as $customer) {
        if ($customer->wallet->getFloorRoundedBalance() === 2003) {
            // Grant coupon.
        }
    }
}
```

Eradicating getters is not the point here, but rather to guard against leaking a
class's own logic to the outside by blindly giving out getters.

## Handle Static Methods With Care

Static methods are available in many object oriented languages, they come in
handy when we want to invoke a function call without instantiating the class
that contains the behavior. However, misusing static methods can lead to bugs
that are tricky to track down.

``` php
// This should work perfectly
Calculator::sum(2, 3);

// This is bad.
Counter::increase(1);
```

The main difference between the two static methods is that the first one is
stateless and the second is stateful. Stateless means that the Calculator does
not remember anything, it simply performs the calculation and returns the
result. You can run it for as many times as you like, and in as many context as
you like, the results remain the same.

On the other hand, a stateful static function is bad because it stores
information from each execution, and the results differs according to how many
times you run it, or where you run it. It is equivalent to sharing the same
global variable, now two pieces of code using the counter simultaneously would
get unreasonable results from their point of view.

## A Little Help For Value Objects

Value objects, unlike entities, their equity depends on if all their states are
of the same value. It is quite a common requirement to determine equity of two
value objects.

``` php
class StreetAddress {
    private $street;
    private $city;

    public function __construct($street, $city) {
        $this->street = $street;
        $this->city = $city;
    }

    public function equals(StreetAddress $streetAddress) {
        return $this->street === $streetAddress->street && $this->city === $streetAddress->city;
    }
}

$streetAddress1->equals($streetAddress2);
```

By adding this help function to our value object class, we now have a convenient way
of examining the equity of two instances of it. We also ensured encapsulation here
by locking the logic of comparison inside the class itself other than calling
getters and do it manually outside the class. If StreetAddress ever gets a new
property, we only have one line of code to update without breaking any other
existing code.

## Replace Conditional Switches With Inheritance

We talked about why and how we should get rid of parameter switches from
function signatures, they serve as a conditional switch to alter the behavior of
the function. Having parameter switches suggests that there are multiple
variants of the function, and they should be extracted out into specialized ones.

The same thing applies to classes. When you observe a class performing actions
according to a conditional switch, it is a good sign that the class should be
broken into sub classes extending the original one.

``` php
class Faculty {
    private $fullname;
    private $title;

    public function __construct($fullname, $title) {
        $this->fullname = $fullname;
        $this->title = $title;
    }

    public function markAssignment() {
        // All faculty members can mark assignments.
    }

    public function printJobDescription() {
        switch ($this->title) {
            case 'Professor':
                echo 'I teach classes';
                break;
            case 'Associate Professor':
                echo 'I teach classes and invigilate exams';
                break;
            case 'Teaching Assistant':
                echo 'I conduct tutorials and invigilate exams';
                break
            default:
                throw new InvalidTitleException();
        }
    }
}
```

Take a look at the example above. It is clear that the business logic requires
professors and associate professors but teaching assistant to teach classes,
associate professors and teaching assistant but professors to invigilate exams,
and finally only teaching assistant can conduct tutorials.

The reason someone abstracted these faculty roles into a single class might be
that they share lots of properties and behaviors, for example, *getSalary()*,
*getYearlyBonus()*, *calculatePerformanceIndex()* and so on. However it is also
true that each of them have their own specializations. Having to use switch
conditionals to differentiate objects is a clear sign that the class is not
following the Single Responsibility Principle, more on that in a later series.

``` php
abstract class Faculty {
    private $fullname;
    private $title;

    public function markAssignment() {
        // Mark the assignment.
    }

    abstract public function printJobDescription();
}

class Professor extends Faculty {
    public function printJobDescription() {
        echo 'I teach classes';
    }
}

class AssociateProfessor extends Faculty {
    public function printJobDescription() {
        echo 'I teach classes and invigilate exams';
    }
}

class TeachingAssistant extends Faculty {
    public function printJobDescritption() {
        echo 'I conduct tutorials and invigilate exams';
    }
}
```

Now we have three clearly defined subclasses each has their own specializations,
we won't have trouble extending these subclasses, or even add a new faculty role
in the future. Polymorphism and inheritance are for specialization, and we
should always favor them over conditionals.

This is the end of our discussion on Clean Code Tips and Tricks: Classes. I hope
you like it and see you in the final installment.
