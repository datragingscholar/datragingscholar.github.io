---
title        : "Clean Code Tips and Tricks: Method Names and Single Responsibility Principle"
date         : "2014-04-12"
tags         : [Clean Code, PHP]
---

Hello fellow geeks, welcome back to the second instalment of Clean Code Tips and
Tricks. As said at the end of the [first one]({%post_url
2014-02-11-clean-code-tips-and-tricks-introduction-variables %}), we are going to
talk about method names today.

## Method Names

### Use verbs and questions
The same rules apply here as for varibale names, but instead of using nouns and
statements to describe variables, we use verbs and questions to name methods.

``` php
doSomething();

performSomeAction();

saveSomeData();

filterSomeInput();

validateSomeForm();
```

Methods answering questions should be, well, questions.

``` php
isUserActive();

isCustomerAdult();

shouldSendEmailAgain();

hasFailedAnyExam();
```

### Be Descriptive

Similar to variable names, be sure to describe your intention well and
wholly.

``` php
getData();

// More descriptive
getDailyVisitors();

// And fully describe the nature
getAverageDailyVisitors();
```

Method names do look longer this way, but it helps your readers to figure out
the usage of these methods without even looking at their definitions.

However, if a method name becomes too long, you may have failed to follow the
single responsibility rule, which means you need to break that method into
smaller ones.

### Examples

These are quiz designed for my team to practice naming methods, I'll list some
of them here as examples.

Note other examples are filtered 'cause they were only there to examine my
team's ability to properly name methods in English.

``` php
//Get a particular user's email address.
getUserEmailAddress($user_id);

//or to make it read more like natural language
getEmailAddressOfUser($user_id);
```

``` php
// Get number of users in a group
getGroupUserCount($group_id);

getUserCountInGroup($group_id);
```

``` php
// Update customer's defalut shipping address
updateDefaultAddress($new_address);

setDefaultAddress($new_address);

// If the customer entity has another property
// with a similar name, e.g. defaultPostalAddress
// then it's better to avoid ambiguity by
updateDefaultShippingAddress($new_shipping_address);

setDefaultPostalAddress($new_postal_address);

// If they are customer entity's methods,
// You can choose whether to include 'customer'
// in the function names.
// I would personally not include it
// since they will always be called with a Customer
// instance.
$customer->updateDefaultShippingAddress($new_shipping_address);

$customer->setCustomerDefaultPostalAddress($new_shipping_address);
```

``` php
// Update a given user's active status
activateUser($user_id); disableUser($user_id);

// Or with one function
$user->toggleActiveStatus();
toggleUserActiveStatus($user_id);
toggleActiveStatusForUser($user_id);
```

``` php
// Is there any order placed by a user in the last month
$user->hasPlacedAnyOrderLastMonth();

didUserPlaceAnyOrderLastMonth($user_id);

// Check if the email has been sent successfully
$email->hasSuccessfullySent();

didEmailSendSuccessfully($email_id);

// Is it necessary to send the notification again
$notification->shouldSendAgain();

shouldSendNotificationAgain($notification_id);

// Check if admin has the permission to perform an action
$admin->canPerform($action_id);

adminHasPermissionTo($admin_id, $action_id);

$admin->isAuthorizedTo($action_id);
```

## Single Responsibility Principle

Now let's talk about Single Responsibility Princeple, the first rule from
S.O.L.I.D.

### Concept

A class or a module in a program should focus on one task. Or as how it's
famously put on the Internet, "A class or module should do only one job, and do
it well."

One analogy is that of a keyboard. An input device we use everyday, it accepts
your keystrokes and transmits corresponding signals to the computer it's
connected to. It has only one functionality or interface if you will, but you
can use it to play games, write articles, or even play an instrument if you have
the right software installed on your computer. A game, a word processor, or an
instrument simulator. Now these are all interchangable parts on the other side
of the interface, but provides great potential of possible applications when
working together as a system, all the while having the keyboard only focus on
***one task***.

However, it absolutely makes no sense to integrate CPU in the keyboard. Despite
that it sounds a little bit cool to have a keyboard with computational power,
but you are introducing a completely different set of technology into the
keyboard, it'd be bloated, and it'd introduce a single point of failure problem
which makes your computer unusable when the keyboard decides to act up.

Think about toy blocks, they are simple in shapes and when stacked properly
together, you can build tons of different types of buildings with them. But if
their shapes become too complex and serve more purposes, they become
***specific***, only usable under certain situations, thus limiting your ability
to be versatile.

### What It Means For Classes

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
method works together to define the method, every method in a class servers
their purposes and create sense for the class, every class in a module fit
together to form a module, and every module work together as a system.

``` php
/*
 * A simple trick to tell if a class follows
 * SRP is to try to describe it as descriptive as you can.
 */

class EmailService {
    protected $invoice;

    function __construct(Invoice $invoice) {
        $this->invoice = $invoice;
    }

    private function calculateTax() {
        return $invoice->subtotal * 0.025;
    }

    public function emailInvoiceForUser(User $user) {
        /* Code to generate email message */
        if (!$emailHasSentSuccessfully) return false;

        return true;
    }
}
```

The above code shows a very poorly designed class, if you try your best to
verbosely describe the class, it's one that "Calculates the tax and generates
invoice email and sends email to a user". You may as well name the class as
***TaxAndInvoiceAndEmailService***.

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

## What It Means For Methods

A method should do ***one*** thing, it should do it ***well***, and it should do
it ***only***. Any method serves more than one purpose is not following SRP.

``` php
// If your method name suggests it does one thing,
// Don't make it do something else.

public function calculateStudentAverageScore(Student $student) {
    return array_sum($student->subjects->scores);
}
```

You are returning the total scores, not the average score in the example above.
It happens more often than you think it would, some developers simply name their
method one thing and change their mind about it's purpose as they implement it.
Some are casued by co-workers modifying an existing method but misunderstood
it's intention. It is extremely confusing expecially when the method is long and
complicated. What if someone new joins the team and uses the method to generate
the report card?

Your average score is 692 points? I always knew my son would one day become a
scientist!

``` php
// If your method name suggests it does one thing,
// Don't make it do more than that.
public function calculateSubtotal($productList) {
    $subtotal = 0;
    foreach ($productList as $product) {
        $subtotal += $product->price * $product->quantity;
    }

    EmailService::sendInvoice($user_id, $subtotal);

    return $subtotal;
}
```

No sir, no no no no no. You are emailing invoice in a method that is supposed to
only calculate the subtotal. Such code pollutes the codebase and is very hard to
maintain. Imaigine your co-worker's face when she sees calculateSubtotal
function throwing an *EmailDeliveryFailed* exception. Or what if you are
required to do your boss's dirty deeds by helping his buddy to fake a seller's
sales record in your multi-tenant ecommerce system, and now random users are getting
invoices for products they never purchased? (FYI, a true story)

``` php
public function enrolStudentToSemester(Student $student, Semester $semester) {
    if ($student->hasOutstandingInvoice) {
        $student->disableStudent();

        EmailService::sendEmail(
            $dean->emailAddress,
            "Warning: Student with outstanding fee"
        );
    } else {
        EmailService::sendEmail(
            $student->emailAddress,
            "Welcome to Semester" . $semester->name
        );
    }

    $student->currentSemester = $semester;

    return $student;
}
```

The above code does more than just enrolling students to a new semester, let's
revise it.

``` php
public function enrolStudentToSemester(Student $student, Semester $semester) {
    if ($student->hasOutstandingInvoice) {
        $student->disableStudent();

        sendOutstandingFeeWarningToDean($student);
    } else {
        sendWelcomeMessageToStudent($student, $semester);
    }

    $student->currentSemester = $semester;

    return $student;
}
```

Leaving specific email sending logics to other functions makes it much clearer.

``` php
/*
 * Sometimes you may have this kind of business logic
 * that sending notification is an inseparable part of
 * new user registration routine.
 */
public function registerUser($userData) {
    $user = addUserToDatabse($userData);

    sendNewUserNotificationToAdmin($user);
}

/*
 * But again, it works to some extend, but now these two
 * business logics are tightly coupled together.
 * You may want to refactor it into the following.
 */

public function registerUser($userData) {
    /* Do User Creation Stuff */
}

public function sendNewUserNotification($user) {
    /* Send notification to admin */
}

public function registerUserAndSendNotification($userData) {
    $user = registerUser($userData);

    sendNewUserNotification($user);
}
```

Now you can freely extend the above three functions as long as they do what
their name says and keeps their interface integrity, this also allows others to
freely choose which behavior fits their requirements best.

That's all for today's session folks, we talked about method naming tips, the
Single Responsibility Principle from S.O.L.I.D, and covered what SRP means to both
functions and classes. I hope you enjoyed it, we'll talk about some general tips
in the next instalment and a little bit on other S.O.L.I.D principles, see you
then.
