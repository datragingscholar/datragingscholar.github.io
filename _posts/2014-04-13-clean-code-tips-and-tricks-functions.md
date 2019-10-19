---
title        : "Clean Code Tips and Tricks: Functions"
date         : "2014-04-13"
tags         : [Clean Code, PHP, Programming Guideline]
---

Hello fellow geeks, welcome back to the second instalment of Clean Code Tips and
Tricks.

In this post we'll talk about functions, or methods. We'll discuss some of the
specific guidelines for naming functions, and some tips on writing and working
with functions.

## Function Names

### Use Actions and Interrogative Sentences
The same rules apply here as for variable names, but instead of using nouns and
general statements to describe variables, we use verbs and questions to name
methods.

``` php
doSomething();

performSomeAction();

saveSomeData();

filterSomeInput();

validateSomeForm();
```

Interrogative sentences clearly states the intent of the function -- to answer a
specific question.

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
getCount();

// More descriptive
getDailyVisitors();

// And fully describe the nature
getAverageDailyVisitors();
```

Method names do look longer this way, but it helps your readers to figure out
the usage of these methods without even looking at their definitions.

However if a method name become too long, you may have given it more than one
thing to do, then it's advisable to break that function into smaller ones. We'll
talk about this in detail later.

Contrary to variable names, which the lengths of their names are proportionate
to the scope they live in, the lengths of method names should be
disproportionate to its scope.

A function that is widely used throughout the system indicates that it is
general and performs a certain task that is common in many aspects of the
software, hence it's name should be generalized and short. And a function with a
smaller scope (i.e. a private function) shows that it's usage are specific in
limited scenarios, you should name it longer with details to reflect it's
specifications.

### Examples

These are quiz designed for my team to practice naming methods, I'll list some
of them here as examples.

Some examples are filtered 'cause they were only there to examine my team's
ability to properly name methods in English.

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
// I would personally not include 'customer' in
// method names since they will always be called
// with a Customer instance.
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

### Name Functions Accurately

``` php
// If your method name suggests it does one thing,
// Don't make it do something else.

public function calculateStudentAverageScore(Student $student) {
    return array_sum($student->subjects->scores);
}
```

In the example above, we are returning the total score instead of the average
score. It happens more often than you think it would, some developers name their
method one thing and simply change their mind about it's purpose as they
implement it. Some are casued by someone modifying an existing method six months
later but misunderstood it's intention. It is extremely confusing especially
when the method is bloated and complicated. What if someone new joins the team
and uses the method *incorrectly* to generate the report card?

> Your average score is 692 points? I always knew my son would one day become a
doctor! ~ An excited Asian mom

And no, it's not racist when I say it. ;-D

## A Function Should Only Do One Thing

You've probably heard this popular phrase, ***A function should do one thing, it
should do it well, and it should do it only***.


Think about toy blocks, they are simple in shapes and when stacked properly
together, you can build tons of different types of buildings with them. But if
their shapes become too complex and start serving more than one purposes, they
become ***specific***, and hard to work with, limiting your ability to be
versatile, and increasing your chance of making mistakes.

### What Could Possibly Go Wrong?

Take a look at the following example, let's say that the initial intent of the
function was to calculate subtotal of a given list of products, and then a new
feature request came in to send that invoice to users, instead of putting that
abstraction into a separate function, someone lazy just put it here without
refactoring the name of the function.

``` php
public function calculateSubtotal($productList) {
    $subtotal = 0;
    foreach ($productList as $product) {
        $subtotal += $product->price * $product->quantity;
    }

    EmailService::sendInvoice($user_id, $subtotal);

    return $subtotal;
}
```

Now we end up emailing invoice in a method that is supposed to only calculate
the subtotal. Such code pollutes the codebase and is very hard to maintain. What
if you are asked to do your boss's dirty deeds by helping his buddy faking sales
record in your multi-tenant ecommerce system, and now random users are getting
invoices for products they never purchased? (FYI, a true story)

The last example illustrates a function that contains a misplaced business
logic, however there are cases with more subtle problems -- these functions
contain a set of abstractions that are tightly related but of different levels.
See the example code below.


``` php
// Note that for the purpose of illustration, the following
// code was written in a simplified way. It has more problems
// other than just violating the 'one thing rule'.
class Subject {
    public function enrollStudent(Student $student) {
        if ($student->hasOutstandingInvoice()) {
            throw new studentHasOutstandingInvoiceException();
        }

        if (in_array($student, $this->listStudents())) {
            throw new studentAlreadyEnrolledException();
        }

        if (count($this->students) >= 200) {
            throw new studentNumberExceededException();
        }

        $studentScheduledForExam = false;
        foreach($this->preliminaryExams as $examSchedule) {
            if (count($examSchedule['attendees']) <= 50) {
                $examSchedule['attendees'][] = $student;
                $studentScheduledForExam = true;
                break;
            }
        }

        if (!$studentScheduledForExam) {
            throw new NoValidPreliminaryExamScheduleException();
        }

        $this->students[] = $student;

        return $this;
    }
}
```

The above code performs a set of closely related actions, it checks if the
student has any outstanding bills to pay, prevents repetitive enrollment and
ensures that there would be no more than 200 students in the classroom. It then
tries to schedule the student for one of the preliminary exam schedules. And
finally it adds the student to the subject's student list and returns itself.

This looks reasonable. Many would claim that those validations and preliminary
exam scheduling are inseparable parts of the business process of enrolling the
student to a class, thus it should be considered as doing *one thing*.

Unfortunately although these steps are indeed part of the business logic, but it
is also true that the function does more than one thing. To understand why, we
have to discuss what exactly *one thing* means.

### Defining One Thing

There are some handy ways to determine if your function is doing only one thing.
Let's try to discuss and follow these tips and make our function a one thing
function first, and then we'll discuss what benefits it brings us which would
then answers the question of why should we do this in the first place.

> A one thing function should have one level of indentation.

Our enrollStudent() function clearly has more than that, look at the part where
we check and schedule preliminary exams. A function with multiple levels of
indentation indicates that it has multiple levels of implementations of
abstractions. In other words, the execution of the steps is not linear.

We all hate a big lengthy function with multiple levels of indentation that
the edge of the function looks like -- as Martin puts it -- a ragged saw. See
which of the following two functions seems easier to understand and work with.

``` php
public function doSomethingA() {
    doSomethingFirst();
    checkSomeOtherThing();

    doIt();
}

public function doSomethingB() {
    if (someConditions) {
        if (tryToDoThis) {
            doSomethingFirst;
        } else {
            foreach(things as thing) {
                doSomeOtherThingWith(thing);
            }
        }

        doTheActualThing;
    }
}
```

> A one thing function should work with one level of abstraction

It's easy to spot that doSomethingB() function in the last example checks some
conditions and loops a list to do ***some other actions*** with ***some other
things*** before it finally does the actual thing we intended it to do the very
last. This is an example of performing action at multiple levels of abstraction.

authenticateUser() is at a level of abstraction which is different than
persistUserCookie(), and renderHtmlReportCard(). It's like concating some
strings and save it to a log file and then inform a lecturer about an exam
schedule change -- they work at different levels of the system, one is lower and
deals with logging and data persistent and the other is performing a high level
business logic -- they should work together, but not be placed in the same
function.

> A one thing function should be extracted from until you can't anymore

We keep extracting things that are not related to performing the *actual thing*
from the function and put them into separate functions until we reach a point
where no more code can be extracted from it meaningfully.

If we try this with our enrollStudent() function, it'll become this,

``` php
class Subject {
    public function enrollStudent(Student $student) {

        $this->guardAgainstOutstandingInvoice($student);

        $this->guardAgainstRepetitiveEnrollment($student);

        $this->guardAgainstExcessiveStudent($student);

        $this->schedulePreliminaryExam($student);

        $this->students[] = $student;

        return $this;
    }

    private function guardAgainstOutstandingInvoice($student) {
        if ($student->hasOutstandingInvoice()) {
            throw new studentHasOutstandingInvoiceException();
        }
    }

    private function guardAgainstRepetitiveEnrollment($student) {
        if (in_array($student, $this->listStudents())) {
            throw new studentAlreadyEnrolledException();
        }
    }

    private function guardAgainstExcessiveStudent($student) {
        if (count($this->students) >= 200) {
            throw new studentNumberExceededException();
        }
    }

    private function schedulePreliminaryExam($student) {
        $examSchedule = $this->findFirstAvailableExamSchedule();

        if (!$examSchedule)
            throw new NoValidPreliminaryExamScheduleException();

        $examSchedule['attendees'][] = $student;
    }

    private function findFirstAvailableExamSchedule() {
        foreach($this->preliminaryExams as $examSchedule) {
            if (count($examSchedule['attendees']) <= 50) {
                return $examSchedule;
            }
        }

        return false;
    }
}
```

We can see that by extracting excessive code from the function and put them into
separate functions respectively, we end up with only 6 lines of code that except
for all the function calls, does only one thing -- enroll the student.

Each of the guard functions performs their own validation and we even split
preliminary exam scheduling code into two functions, one in charge of finding an
available schedule slot, the other dedicates in the actual scheduling.

By applying the three rules we discussed about to each of every function in this
example, you'll find that they all have one level of indentation, they work
under the same level of abstraction within themselves, and we practically cannot
extract anything meaningful from any of them anymore.

Now we can say our functions comply the one thing rule, but what good does this
do?

### Was That Really Necessary?

The most obvious change was the reduction of the length of our enrollStudent()
function. You may think it's cheating because all we did was to put those lines
of code into other functions. That is true, however we did successfully reduced
the length of the original function and now every function in that class is
short and clean.

Would you rather work with one gigantic function, or a dozen of well organized
smaller ones each does exact one thing at a time? Our brains are not good at
following up complex systems and tracking down complicated logics, by
decomposing a large concept into smaller pieces each containing highly cohesive
set of steps, we ease the difficulty of understanding and maintaining the code,
which is once again, the main purpose of Clean Code.

By reorganizing excessive code into other functions and having our original
function call them, we encapsulated details of other levels of implementation
into other functions. This allows our readers to get a brief idea by a quick
glance at the short function, they may then choose the follow these function
calls that interest them to read the details, or they could ignore all the
boring details and just focus on what this function does.

Another added advantage is that by creating more functions, which is a byproduct
of the extraction process, we end up with more ***names***. And if we follow the
advise on naming your functions properly, these names would serve as
documentation to your code, they help us to understand the intention of the
code, it's almost like putting labels on your drawers that keeps your socks and
underwear separately.

Even if an excessive piece of code would only be executed along with a business
logic, it should be split into a different function, you never know if a new
feature request in the future would required that same functionality from a
different business process. This improves code reusability.

## Function Parameters

Now let's talk about an important part of a function's signature, its required
parameters.

### No More Than 3

A function should really have no more than 3 parameters. A function with 2
parameters each one is of boolean type would already yield a total of 4 possible
combinations, imagine a function with 7 mix-typed parameters.

``` php
public function registerStudent($firstname, $lastname, $dob, $major,
    $secondary_major, $fatherdob, $motherdob, $address) {
    ...
}

registerStudent('James', 'Bond', 'Computer Science', 'Network Security',
'1990-01-01', '1966-02-02', '1970-03-03', '1-1-1, Broad Avenue');
```

Look at this function with a messy signature like this, it would be really
confusing when we invoke the function call. What are these dates for? What is
the correct order?

When you encounter a function with more than 3 parameters, check if it follows
the one thing rule first, there's a good chance it's receiving too many
parameters because it's trying to do multiple things. If that's not the case,
use a value object to pass the data around, or at least utilize array to
aggregate parameters.

``` php
public function registerStudent($student) {
    ...
}

registerStudent([
    'firstname'=> 'James',
    'lastname' => 'Bond',
    'dob' => '1990-01-01',
    'major' => 'Computer Science',
    'secondary_major' => 'Network Security',
    'fatherdob' => '1966-02-02',
    'motherdob' => '1970-03-03',
    'address' => '1-1-1, Broad Avenue',
]);
```

### Avoid Boolean Switches

That's right, hash tag noboolean, we should avoid using parameter switches. A
parameter switch is a parameter, often boolean in a function that is used for
conditional switches.

``` php
public function printStudentReport($student, $gpa = false) {
    if ($gpa) {
        // Convert results into gpa scoring system
    } else {
        // Use normal scoring system
    }
}
```

Print a student's academic report is the *one thing* this function was intended
to do, but whether to use GPA scoring schema or a regular 100 scoring schema
should not be part of it's logic, it should be up to the higher level of the
business flow to decide what it wants.

``` php
public function printStudentReportWithNormalSchema($student) {}
public function printStudentReportWithGPASchema($student) {}
```

Now the calling party can freely decide what behavior they expect and invoke
corresponding calls accordingly.

``` php
public function printStudentReport($student, $gpa = false) {
    if ($gpa) {
        // Convert results into gpa scoring system
    } else {
        // Use normal scoring system
    }

    if ($student->isFinalYear()) {
        // Append final class result
    }
}
```

Note that whether the student is a final year student and choose whether or not
to append the final class result in the report is indeed part of the printing
logic. However it works at a different abstraction level and should be extracted
out, now combine these together we get the following functions.

``` php
public function printStudentReportWithNormalSchema($student) {
    $this->appendFinalClassResultIfApplicable($student);

    // Do the actual printing.
}

public function printStudentReportWithGPASchema($student) {
    $this->appendFinalClassResultIfApplicable($student);

    // Do the actual printing.
}

private function appendFinalClassResultIfApplicable($student) {
    if (!$student->isFinalYear())
        return;

    // Append final class result
}
```

Alrighty, this is it folks. We've talked about naming functions, the one thing
rule and some other handy tips and guidelines when working with functions. I
hope you like it and see you in the next installment.
