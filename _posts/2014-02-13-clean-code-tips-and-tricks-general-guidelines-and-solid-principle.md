---
title        : "Clean Code Tips and Tricks: General Guidelines and S.O.L.I.D Principle"
date         : "2014-04-13"
tags         : [Clean Code, PHP, Software Principle]
---

Welcome back fellow computer witches and wizards, let's get back to where we
left on Clean Code Tips and Tricks. This will the be the last instalment, we'll
talk about some general guidelines and tips to write better code, and then
continue our discussion on S.O.L.I.D principle. Let's get started.

## Don't Abbreviate

We've already talked about this, no matter it's a module, a class, a method or a
variable, don't name it with acronyms unless it's well know by everyone or
at least by people in the specific field you are working in.

``` php
// Probably bad
$ecpt, $tot, $sx, $gw, $wptr;

// Ok
$gpa, $avg, $desc, $asc, $id, $url, $info;
```

## One Arrow(Dot) Per Line

Use linebreaks to make your code easier to read. You may want to follow a
different way to break your dots or arrows, it's okay as long as it make your
code easier to understand.

``` php
// Not so good
$this->setupConfigs()->registerProvider()->loadViews()->listenForLogout();

// Looks better
$this->setupConfigs()
     ->registerProvider()
     ->loadViews()
     ->listenForLogout();

return $this->project->deployments()->where('status', 'Available')->orderBy('updated_at', 'DESC')->get();

// Different styles are okay, too
return $this->project->deployments()
            ->where('status', 'Available')
            ->orderBy('updated_at', 'DESC')
            ->get();
```

## One Level of Indentation Per Method

It's a little bit too much to ask, but a generally good practice to keep your
method simple and clean. If you have too many indentation levels in your method,
check if it violates SRP, or is there anyway to optimize the algorithm.

``` php
public function runCommands(array $commands)
{
    if ($this->isEnabled) {
        foreach ($commands as $command) {
            $command->run();
        }
    }
}

// We can reduce levels of indentation by returning early
public function runCommands(array $commands)
{
    if (!$this->isEnabled) {
        return;
    }

    foreach ($commands as $command) {
        $command->run();
    }
}
```

## Clean Up IF Statements

The last example showed how to reduce levels of indentation by returning early,
it's a common method used to clean up if statements, we always want to do the
most expected thing ***lastly***.

By doing this, we can eradicate `else` statements, the structure of the if block
looks clearer and easier to read, and performing the most common or most
expected behavior at the end of the code block matches how our mind works.

``` php
public function getStatusAttribute()
{
    if (!$this->isDeployed) {
        $status = 'Undeployed';
    } elseif ($this->deployment->status === 'Active') {
        $status = 'Running';
    } else {
        $status = $this->deployment->status;
    }

    return $status;
}

/*
 * The above code is at best 'confusing'.
 * When there are lots of ifs and elseifs,
 * we are prone to make logic errors which lead to bugs,
 * and it's hard to understand what this piece
 * of code is actually trying to achieve.
 *
 * Let's try to eradicate elseif and else statements.
 */

public function getStatusAttribute()
{
    if (!$this->isDeployed) {
        return 'Undeployed';
    }

    if ($this->deployment->status === 'Active') {
        return 'Running';
    }

    return $this->deployment->status;
}
```

Much better, right? Now the key is to identify what is the *most expected
behavior*.

``` php
public function uploadAvatar($avatar)
{
     if (!isset($avatar)) {
         if ($this->isDisabled() === false) {
             if (in_array($avatar->fileExtension, $allowedFileExtensions)) {
                 $this->avatar = $avatar;
                 $this->save();

                 return true;
             } else {
                 LogService::logError(FileExtensionNotAllowed);

                 return false;
             }
         } else {
             LogService::logError(UserIsDisabled);

             return false;
         }
     } else {
        LogService::logError(UserAlreadyHasAvatar);

        return false;
     }
    /*
     * As the name of the method suggests,
     * We are setting user's avatar, so the
     * most expected behavior is, well, set
     * the avatar. Other branches may be
     * file extension unmatch, or user already
     * have an avatar, or the user is disabled.
     *
     * To refactor the above code, we just move
     * the most expected behavior to the last.
     */

     if (isset($avatar)) {
         LogService::logError(UserAlreadyHasAvatar);

         return false;
     }

     if ($this->isDisabled()) {
         LogService::logError(UserIsDisabled);

         return false;
     }

     if (!in_array($avatar->fileExtension, $allowedFileExtensions)) {
         LogService::logError(FileExtensionNotAllowed);

         return false;
     }

     $this->avatar = $avatar;
     $this->save();

     return true;
}
```

Doesn't it look much more straightforward after we refactored it? Try this trick
everytime when you encounter nested if statements.

## Don't Comment

This is an exaggeration. Comments are sometimes necessary, but we are
***overdoing*** it.

``` php
/*
 * Dear fellow co-workers:
 * I consulted with our marketing team, they said the client
 * must have more than 100 orders in total, but Bob said there's
 * a bug in the CouponService so we have to check if the user has
 * too many coupons to handle. Then Joe talked with the boss and
 * confirmed that this should not include inactive clients.
 *
 * Hope that clears it up. Feel free to ask me questions.
 */
if ($client->getTotalOrders() > 100 && $client->getCouponCount() <= 10 && $client->inActive() {
    return $client->id;
}
```

Oh, how nice of you to leave such a thoughtful comment to your co-workers, but
unfortunately it's not the best practice.

> "Comments are often lies waiting to happen."

Think about this, what if the marketing team later changed their minds? What if
Bob fixed the bug in CouponService? What if Joe got it all wrong and the boss
meant something different?

They are not lies when you write them down, yet. But they will eventually become
lies if given time. That's not the real problem yet, wait when someone comes back
to reflect Bob's bug fix here and forgot to change your comments and a new guy
joins the team believing that the bug never got fixed by reading your comment.

By writting comments like this, you are adding *extra* stuff to maintain, and
you risk misleading yourself and your co-workers. Write comments only when it's
definitely necessary.

## Get Rid of Zombie Code

Zombie code are code blocks that are commented for some reason, and that reason
may have been forgotten by the person who did it. They are normally useless, and
often there for a very long time, thus not up to date with the newest codebase.
They disturbs code reading, uglifies our codebase, and worst of all, they may
accidentally come back to life.

> "Wow, look what I found? I think this commented method is definitely going to
> work perfectly. I wonder who did this to such wonderful and elegant code."

Get rid of them by following these steps:
* Comment out temporarily unwanted code block
* Put a comment there stating the current date
* If you find a commented code block without a date, add current date
* If you find a commented code block that is more than 30 days old, delete it

Don't worry, I've never heard anyone complaining,

> "Oh damnation! Who deleted my commented code? I was planning to use it three
> monthes later, now I have to write it again!"

Says no one, ever.

## Aggregate Parameters

If you are passing too many arguments to a function, it can get really messy and
ambiguous.

``` php
//FileA.php
public function createUser(username, password, motherdob, fatherdob, dob) {
    DB::run(
        "INSERT INTO users(username, password, motherdob, fatherdob, dob)",
        username, password, motherdob, fatherdob, dob);

    return true;
}

//FileB.php
createUser('test', '123321', '1940-01-01', '1945-01-01', '1973-01-01');
```

When calling `createUser()`, you'd have to refer to the documentation or go the
definition to understand what these dates are for.

``` php
//FileA.php
public function createUser($user) {
    DB::run(
        "INSERT INTO users(username, password, motherdob, fatherdob, dob)",
        $user['username'],
        $user['password'],
        $user['motherdob'],
        $user['fatherdob'],
        $user['dob'];
    );

    return true;
}

//FileB.php
createUser([
    'username' => 'test',
    'password' => '123321',
    'motherdob' => '1940-01-01',
    'fatherdob' => '1945-01-01',
    'dob' => '1973-01-01',
]);
```

We'd have to refer to the definition to figure out what are the elements needed
in the array to call `createUser`, but it now lets your readers have a clear idea
about how the function call works.

## Don't Repeat Yourself

DRY is a famous software engineering principle and has a profound impact on how
we code nowadays. I won't go into details about DRY, but you can find many great
resources online if you are interested.

Simply put, never write code that is similar or equal to what you or your
co-workers have written before.

* Same code pieces in different methods
  * Refactor to a new method
* Same code pieces in sibling class
  * Refactor to a method in the paraent class
* Similar code in a class
  * Refactor to a method with parameters
* Duplicated code in different classes
  * Consider using traits or equivelent techniques in other languages

## No God Classes

God classes have too many responsibilities, they are usually bloated and large
in size, they are monolitic and knows too much about business logics that don't
belong to them, and they are usually tightly coupled with other classes. As we
have discussed before, such problem can be resolved by following Single
Responsibility Principle. I won't repeat on this topic, but would like to
introduce another technique to identify a god class.

``` php
// Look for name prefixes in function signatures
class DataFileProcessor
{
    public function splitFileByLineCount();
    public function splitFileByByte();
    public function splitFileByKeyword();

    public function processSalesData();
    public function processProductsData();
    public function processImages();
}
```

You may have guessed, even though these functions look like they all
process the data files, thus should be in `DataFileProcessor`, but the fact
they are grouped with name prefixes tells us a different story. It would be
better to split it into `DataFileSplitter` and `DataFileProcessor` classes.

## S.O.L.I.D Principle

Thus far, except for SRP, we focused on how to write code that are easy on the
eyes and enjoyable to work with. Which I think is a must before discussing any
patterns or principles. I believe making source code readable is the first
priority but is often underrated.

I hope I have convinced you how important it is to write beautiful, clean, and
elegant code. Now let's move on to S.O.L.I.D, which I think every developer
should follow as long as they are working with an OOP language.

Again, this is by no means a solid introduction to S.O.L.I.D, you can find much
better resources online, it's rather some tips from my personal experiences and
additions I'd like to point out which were not covered or stressed enough by
resources I read.

### Single Responsibility Principle

True we've already covered SRP, but I'd like to point out that our previous
discussion were not entirely accurate.

> A class should have one, and only one, reason to change.
> ~ Uncle Bob in his [Priciples of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

Some people tend to believe that *One Reason to Change* does not equal to *One
Thing* which you probably have heard in ***A class should do one thing, and one
thing well.***, it would be a disaster to have all classes doing only one
particular *thing*.

To save us from semantic debate, I believe responsibility means a set of
interdependent and interconnected jobs that can't and shouldn't be divided into
smaller groups. Having a single responsibility means there's only one reason to
change the class, no matter it's fixing a bug or adding a new feature, we do it
all under the scope of this particular responsibility.

Imagain if we have a class called `DataFileSplitterAndProcessor`, it actually has
two responsibilities, splitting and processing data files, thus there are two
reasons to change the class, which violates the SRP.

Secondly, SRP only covers classes, does that mean modules and methods can have
mulitple responsibilites or mulitple reasons to change?

In a sense, yes. For example we've seen a function called
`registerUserAndSendNotification()` in one of our previous examples, and we can
easily examplify a module name `DataFileModule` which handles everything about
data files, creating, deleting, splitting, and processing.

However I personally like to think all entities in a software, including
modules, classes, methods, or even interfaces should follow SRP. If we define
Responsibility properly, every one of them is performing a set of interdependent
and interconnected jobs under the scope of their particular responsibility.

After all, our mind sucks when dealing with complex systems, but we are very
good at finding creative solutions under a specific scope. If we take a closer
look at DRY principle, the first step to don't repeat yourself is to divide your
software into smaller modules, and modules into smaller submodules which become
classes, classes into routines or algorithms implementation, until we reach a
point where they contain the smallest piece of our business logic, thus, having
only one responsibility. Then, by making sure

> The smallest piece of our business logic only occur once in our entire system.

we achieve DRY principle.

Sounds familiar, right? As we have discussed previously, think when would this
scenario take place.

> Similar code in different methods in a class, refactor into a new method.

``` php
class DataFileSplitter {
    public function SplitDataFileByLine($file, $linecount) {
        /* Code to open file */

        /* Code to split file with $linecount each */
    }

    public function SplitDataFileByByte($file, $byte) {
        /* Code to open file */

        /* Code to split file with $byte each */
    }
}

/*
 * We can see that two functions are not following SRP, they open files and
 * split files, resulting in a piece of repeated code. Let's refactor,
 */

class DataFileSplitter {
    private function OpenDataFile($file) {
        /* code to open file */
    }

    public function SplitDataFileByLine($file, $linecount) {
        $this->OpenDataFile($file);

        /* Code to split file with $linecount each */
    }

    public function SplitDataFileByByte($file, $byte) {
        $this->OpenDataFile($file);

        /* Code to split file with $byte each */
    }
}

/*
 * Since openning file would probably be share by all methods in
 * DataFileSplitter class, and every instance of the class should
 * bind to a $file, so we can do that with property and construct
 * method too if you like.
 */
```

You can see that by following DRY, we actually achieved SRP for each method. To
conclude, I think we should know that SRP meant *A class should only have one
reason to change.* but it's safe and quite desirable to make every entity in our
software to follow it, in their own way.

### Open-Closed Principle

> You should be able to extend a class’s behavior, without modifying it.
> ~ Robert C. Martin

A class should be open to extension, meaning a new behavior, functionality or
property, but should be closed to modification, no one should modify the
existing code of the class. Unless, of course, when you are fixing a bug.

This is a powerful idea, if we managed to achieve this, any new features and
additions to functionalities resulted by requirement changes or new requests
from stakeholders can be implemented by adding new code, not changing old ones.

``` php
class Product {
    protected  $name;
    protected  $price;

    public function getName() {
        return $this->name;
    }

    public function setName($name) {
        return $this->name = $name;
    }

    public function getPrice() {
        return $price;
    }

    public function setPrice() {
        return $this->price = $price;
    }

    public function displayLabel() {
        return $this->name . ': $' . $this->price;
    }
}
```

Imagain we have a Product class shown above, now our client wants a new feature,
where there are certain promotional products each have their own discount rate,
but their discounted prices are only available on 11th of November, the Chinese
version of boxing day. Well, a conventional way to implement the new feature is

``` php
class Product {
    protected $name;
    protected $price;
    protected $discount_rate; //<-

    ...
    ...

    public function getDiscountRate() { //<-
        return $this->discount_rate;
    }

    public function setDiscountRate($discount_rate) { //<-
        return $this->discount_rate = $discount_rate;
    }

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
`$discount_rate`, its getter and setter, and worst of all, we had to modify
`displayLabel` function which may lead to bugs and maintainability issues.

The way to comply with Open-Closed Principle is by using abstraction and
polymorphism, namely inheritance.

``` php
class Product {
    /* Original Product Class */
}

class PromotionalProduct extends Product {
    protected $discount_rate;

    public function getDiscountRate() {
        return $this->discount_rate;
    }

    public function setDiscountRate($discount_rate) {
        return $this->discount_rate = $discount_rate;
    }

    public function displayLabel() {
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

We did it without modifying the original Product class. By creating a new class
called `PromotionalProduct` that extends `Product`, we inherited all member
properties and methods, and by overwriting `displayLabel()` function's behavior, we
fullfiled the requirement.

Now this new additon may have bugs and issues, but we saved ourselves from
creating problems in existing codebase by modifying it, which may potentially
lead to much more confusing issues.

Two things worth noticing here, the first being interfaces and inheritance are
all viable ways to achieve abstraction and polymorphism, choosing which one to
use is out of the scope of this post.

Plus the choice depnds heavily on which programming language you use. For PHP,
inheritance is a way to share similar behavior, traits can be used to share code
among classes that need the same piece of code but share nothing else in common,
and interfaces provide a way to define common protocols.

Secondly, we achieved Open-Closed Principle by inheritance, but it's not the
best solution, as we'll continue to discuss this example in --

### Liskov Substitution Principle

If Open-Closed Principle tells us how to avoid modifying existing code to
achieve maintainablity and robustness by abstraction, the Liskov Substitution
Principle tells us how to utilize abstraction ***correctly***.

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
            $subtotal += $entry->product->price * $entry->quantity;
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
when we supply `$order` with at least one `PromotionalProduct` in it.

``` php
class Invoice {
    ...
    ...

    public function calculateSubtotal($order) {
        $subtotal = 0
        foreach ($order as entry) {
            if ($entry->product instanceof PromotionalProduct
                &&
                $entry->date === BOXINGDAY) {

                $subtotal +=
                    $entry->product->price * $entry->product->discount_rate * $entry->quantity;
            } else {
                $subtotal += $entry->product->price * $entry->quantity;
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

LSP also implies that our program should remain ignorant of such substitution,
it shouldn't care if the object is of the base class, or the derived class.

If you ever come across a piece of code like this, performing actions according
to which class the object is an instance of, and the classes in the context
implements abstraction (interface or inheritance), it's a clear sign that you
may have violated LSP. This piece of code most probably have the knowledge of
every possible derivatives of the base class and need to be updated everytime we
add a new one.

The simple solution is to not use abstraction, treat `Product` and
`PromotionalProduct` as two different entities dispite that they share a lot in
common.

Though some may argue if we modify the `getPrice()` getter in the derived class
`PromotionalProduct` and let it return full prices and discounted prices
accordingly, then we don't have to know the object type in our `Invoice` class,
we can calculate the subtotal simply by calling each object's `getPrice()` function.

That is true, our example here has been a subtle one, as the famous Rectangle
and Square example Robert C. Martin described
[here](https://drive.google.com/file/d/0BwhCYaYDn8EgNzAzZjA5ZmItNjU3NS00MzQ5LTkwYjMtMDJhNDU5ZTM0MTlh/view).

Let's extend the example a bit to further visualize the problem. Let's say we
need to calculate shipping costs by a given province name, and we want a new
property holding tax rate for each product. The catch here, is that promotional
products can't be shipped outside of the country but normal products can, and to
further encourage sales, all promotional products on 11th of November are tax-free.

``` php
class Product {
    ...
    ...

    protected $tax;

    public function getShippingCost($province) {
        /* Code to calculate shipping costs */
    }

    public function setTax($tax) {
        return $this->tax = $tax;
    }

    public function getTax() {
        return $this->tax;
    }

    ...
    ...
}

class PromotionalProduct {
    ...
    ...

    public function getShippingCost($province) {
        if (!in_array($province, $domesticProvinces) && TODAYISTHEDAY) {
            return 'Product unavailable in your country';
        }

        / Calculate shipping cost normally */
    }

    public function getTax() {
        if (TODAYISTHEDAY) {
            return 0;
        }

        return $this->tax;
    }

    ...
    ...
}
```

We added a new property call `$tax`, it's getter and setter, and a function to
calculate shipping cost in the base class, and we inherited and overwrote
`getShippingCost()` and `getTax()` functions to reflect the requirements.

Now this looks perfect, what could ever go wrong? Let's take a look at our
`Invoice` class, which by the way, is a client (or user) of our base class
`Product`.

``` php
class Invoice {

    ...
    ...

    public function calculateTotalShippingCosts($order) {
        $totalShippingCosts = 0;
        foreach ($order as $entry) {
            $totalShippingCosts += $entry->product->getShippingCost($order->address->province);
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

Same as the last time, after we extended `Product` class with a new
`PromotionalProduct` class, these two functions in `Invoice` class broke. When
there is at least one `PromotionalProduct` in `$order`,
`calculateShippingCosts()` wouldn't return anything meaningful since
`$product->getShippingCost()` now sometimes returns a string which we never
expected when writting the `Invoice` class, and `calculateTaxCredit()` function
would simple throw an error since we didn't know any product could have a tax
rate of zero.

This is an illustration of both precondition and postcondition violation.

``` php
public function getShippingCost($province) {
    if (!in_array($province, $domesticProvinces) && TODAYISTHEDAY) {
        return 'Product unavailable in your country';
    }

    / Calculate shipping cost normally */
}
```

Look at this function in our `PromotionalProduct` class, what we did wrong here
is that we made the precondition of the original `getShippingCost($province)`
function in `Product` class ***stronger***, the original function accepts any
province or states in the world and yields shipping cost accordingly, but in
`PromotionalProduct` it only accepts domestic provinces, thus we strengthened
the precondition, causing function callings in clients or users of the base
class to potentially fail.

``` php
public function getTax() {
    if (TODAYISTHEDAY) {
        return 0;
    }

    return $this->tax;
}
```

And the `getTax()` function in our `PromotionalProduct` class, by adding one
more possible return value, zero, we made the postcondition ***weaker***. This
caused clients or users of the `Product` class getting unexpected return values
from instances of `PromotionalProduct`, which means the substitution didn't
successfully happen without causing issue or altering existing behaviors.

> …when redefining a routine [in a derivative], you may only replace its
> precondition by a weaker one, and its postcondition by a stronger one.
> ~ Bertrand Meyer

Abstraction is a core concept of OOP, but sometimes we establish inheritance or
implementation relationships same as we would categorize objects in real life. A
Teacher and a Student, a Product and an AuctionProduct, a Viewer and a Streamer,
we simply think they are similar objects but overlooked the fact that they may
behave completely differently in our system.

The correct way to establish abstraction is through behavior, afterall if
clients or users of the base class aren't allowed to expect ***exactly the
same behavior*** from all the derived classes, then we may as well make them
independent entities.

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

    function __construct() {
        $this->email = new Email();
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

So instead letting `PromotionNotification` class depend on concrete `Email`
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
`Email` and `SMS` class to implement it. Now our `PromotionNotification` depend
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

## Conclusion

Whew, what a journey. This is the end of our three-instalment Clean Code Tips
and Tricks, I hope you enjoyed it as much as I enjoyed preparing it.

We talked about the importance of writing readable code that'll make you and
your co-workers understand better and enjoy working with. Then we talked
about the S.O.L.I.D principle which I think is very important to make our code
maintainable and robust.

I have to admit that there are limitations such as some of the tips are purely
from my personal experience thus biased with my personal preferences, some
examples in PHP can't represent all other OOP languages, and my introduction to
S.O.L.I.D is not nearly as solid as those fantastic sources online. Dispite all
these, the one take away I hope you get is that ***We need to spend more time
writing good quality code, and less time reading bad ones.***

I plan to write more about design patterns in the comming weeks, please stay
tuned if you are interested, however be warned that I won't be writing
introductions or tutorials, I'll be sharing my opinions and experiences with
them.

Have a nice day and see you soon.
