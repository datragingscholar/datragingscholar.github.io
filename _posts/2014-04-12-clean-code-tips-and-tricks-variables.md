---
title        : "Clean Code Tips and Tricks: Variables"
date         : "2014-04-12"
tags         : [Clean Code, PHP, Programming Guideline]
---

Welcome back to the second installment of Clean Code Tips and Tricks. Today we
are going to talk about variables, specifically, naming your variables.

Naming things in programming is perhaps the most overlooked aspect that can
actually greatly help improving the readability of our code. You'll see after
reading this installment that giving variables proper names can make your
intentions clear and avoid ambiguity, moreover, hunting good names can often
reveal problems within your code.

In the process of finding good names for your variables, you may find out
problems with the content of the data they are going to hold and refactor your
code to have a better structure. For example, if you find it hard to name a
variable that's holding the return result of a method because it returns more
than one types of relatively unrelated data, that's a good sign that your method
is not properly structured and is trying to do more than one thing, it then
becomes clear that splitting the method into smaller ones each performs one
particular action would improve the cleanness of your codebase.

Many of the practices I suggest here for naming a variable apply to others such
as method names, class name, module names and even array key names. We'll cover
those that are different or specific in later installments, now let's get
started with variables first.


## Use English Words (Duh!?)

This is a huge problem many of my teams suffered, they used to use *PinYin* as
variable names, a method to write Chinese characters using letters to spell the
pronunciation. Problem is there are some guesswork involved before you can
determine what characters are being spelled out due to the huge amount of
homonyms in Chinese. Plus using roman letters only can't effectively express
intonations, which is crucial to the meaning of a phrase.

It's quite a common thing in China, many teams suffer from this problem. It
takes roughly 5-10 seconds for a native Chinese speaker to figure out the
meaning of a variable name by guessing and relying on context, quite ineffective
and ambiguous.

``` php
$baoxian // This can mean preservation or insurance
$youli // This can mean advantagous, powerful, or rational
$zhiwu // Plant, job position
$baodao // Student attendance, news report
```

It gets worse when your team has members from different countries or speak
different languages. I've seen lots of variable names written completely or
partially in Malayu when I was working in Malaysia, which was a huge pain for me
as a foreigner.

I personally estimated the rough amount of frequently used English words when
programming is only 100 - 500, give or take as people work in different types of
businesses. Comparing to high school graduates in China, they have learned at
least 2000 English vocabulary, which is more than enough to write English
variable names.

## Use Nouns Or Stataments

Variables are containers that hold data or state, you shouldn't name them with
verbs or questions.

``` php
// Nouns are the most common variable names.
$comment;
$avatar;
$firstname;
```

This is straightforward, we have been naming our variables with nouns all the
time, but when it comes to expressions and statements, sometimes we name them in
a way that'll confuse other people.

``` php
// It sounds more like an action.
$moveStudent = true;

// Should be a function name instead
public function moveStudent($student){}

// Now it looks like a question
$shouldMoveStudent = true;

// Interrogative sentences are good names
// for functions that answer a specific question
public function shouldMoveStudent($student)
{
    if (...) return false;

    return true;
}

// Finally a general sentence suit for a variable
// Nice and clear.
$studentShouldBeMoved = true;
```

## Don't Abbreviate, Don't Be Too Verbose Either

It sometimes surprises me how many people still name their variables with C or
even ealier programming language styles decades ago, when efficiency was the
utmost priority. They use abbreviations, single letter temporary variable names,
even some mathmatical terms and habits like `$tot`, `$sqrt`, or `$log`.

Please don't, as I noticed some new developers who didn't witness the C era
picked up the habits from other programmers, but have no idea why they name
like that, or what do they mean.

Not everyone understands your abbreviation, not everyone's feeling nostalgia,
and we don't have to shorten our code to save space anymore. Keep in mind that
this suggestion works and should be applied to function names, class names, and
module names.

``` php
// What do they mean?
$i, $q, $r, $c;

// What if someone don't understand these abbreviations?
$idx, $req, $res, $cnt;

// Better but not perfect.
$index, $request, $response, $count;

// Don't make them too long either
$customersHusbandOrWifeAddress; => $spouseAddress;
```

Sometimes it's hard to tell exactly how long is *too verbose*, but being too
long is still better than being too short. We have to try our best to get rid of
ambiguous abbreviations since it harms the understandability while on the other
hand unnecessarily verbose names just waste a little of your time.

A practical rule I read online is to determine the length of your variable name
based on the scope it lives. The wider this variable's scope is, the longer and
more specific the name of the variable should be since it would be even harder
to check back to the context where the variable was defined when it is so widely
referred to. And it is acceptable to name a counter as *$counter* when the scope
it lives in is only 3 lines of code.

## Be Descriptive

Be generous when you name your variables, try to describe everything about the
data or state your variables are holding.

Describe your intention clearly, don't let your readers rely on context to
understand your variable's purpose.

Describe their natures fully. If you don't express all characteristics of your
variables, you risk confusing your readers.

``` php
// Count of what?
$count;

// Better, but have you described its purpose clearly?
$appliedCount;

// Perfect, especially when you have an $appliedTeachersCount
// in the context, this avoids ambiguity
$appliedStudentsCount;
```

We code within contexts, and can easily take the context for granted and assume
everyone who's going to read this piece of code processes the same level of
understanding of the context. It is often not the case, and if we can help our
readers grasp the meaning of our variables in it's proper context by naming them
correctly, why shouldn't we?

``` php
/*
 * It's hard to understand the full picture.
 * You are not describing every aspect of it.
 * Is it the average score of all subjects of a particular
 * student or is it the average score of the class?
 */
$averageScore = array_sum($scores)/count($scores);

// Much better
$studentAverageScore = array_sum($subjectsScores)/count($subjectsScores);
```

## More Examples

``` php
$times = AlarmService::getTotalTimes($alarm_id);

return $times;

// The following is much more descriptive

$alarmTriggeredTimes = AlarmService::getTotalTriggeredTimesFor($alarm_id);
```

``` php
$revenue = 0;

foreach (
    $company->getAllRevenue()
            ->latest()
            ->limit(12)
    as $temp )
{

    $revenue += $temp;
}

return $revenue;

/*
 * Don't confuse your readers with names like $temp
 * It's hard to understand this piece of code
 * without noticing what 12 means.
 */

$yearly_revenue = 0;

foreach (
    $company->getAllRevenue()
            ->latest()
            ->limit(12)
    as $monthly_revenue )
{

    $yearly_revenue += $monthly_revenue;
}
```

``` php
/*
 * Naming your variables and methods properly
 * makes your code read like natural language.
 * and become so much easier to read and work with.
 */

$userIsActive = $user->isActive();

$addressIsDefault = $address->isDefault();

$newVersionAvailable = VersionService::isNewVersionAvailable();

do {
    $allEmailsSent = EmailService::processEmailQueue();
} while (!$allEmailsSent)

if ($studentFailedAnyExam) {
    ReportService::generateTranscriptWithRedBackground($student_id);
}
```

## Declare Variables Close To Where You Use Them

Unless they are constants or class properties, declaring your variables too far
away from where you actually use them force your readers to jump around looking
for definitions.

``` php
/*
 * Declaring all variables at the beginning may be
 * considered a good practice under certain context
 * but it feels more like natural language to declare
 * only when you are about to use them.
 */

$expiringCoupons = $user->listExpiringCoupons();
$eligibleCouponValues = [30, 100, 250];
$hasIneligibleCoupon = false;

foreach ($expiringCoupons as $expiringCoupon) {

    if (!in_array($expiringCoupon->value, $eligibleCouponValues)) {
        $hasIneligibleCoupon = true;
        continue;
    }

    CouponService:extendCoupon($expiringCoupon->id, 30);
}

if (!$hasIneligibleCoupon) return;

EmailService::sendCouponExpiringWarning($user->id);
```

Reading and writing code requires a high level of concentration, tracking
half a dozen of variable names, their definitions and types while trying to
understand a piece of code is challenging to every human brain. Our efficiency
would deteriorate if we were constantly interrupted to look for variable
definitions across the lengthy source file.

## Use Explanatory Variables

Explanatory variables exist, while, to explain your code. By introducing a
variable that would otherwise be considered unnecessary and giving it a proper
name can do wonders to help your readers better understand the code.

``` php
foreach ($expiringCoupons as $expriringCoupon) {
    if (!in_array($expipringCoupon->value,
        config('promotion.nationalday.coupon_values')
       ) {
       ...
    }
}

// The above code works perfectly, there's nothing
// wrong with it, but the following is more friendly to
// your readers, they'll appreciate your efforts.

$eligibleCouponValues = config('promotion.nationalday.coupon_values');

foreach ($expiringCoupons as $expiringCoupon) {
    if (!in_array($expipringCoupon, $eligibleCouponValues)) {
        ...
    }
}
```

Note that defining *$eligibleCouponValues* was not necessary, however it helps
to understand the code as if we *documented* the nature of the data it holds.

That's it for today, we talked about how to properly name your variables to
improved readability and avoid ambiguity. I think it is the first and most
effective step towards a better and cleaner codebase. You may noticed that the
tips may not necessarily improve *efficiency* from a computer's perspective who
consumes code, but they reduces headache from a reader's perspective, we don't
want to favor computers by providing them efficient but hard to understand
instructions when your co-workers are the ones you should really care about.

It is true for the coming tips and practices in the following installments, and
also true for most design patterns and software development principles, but I
would say this is a rather pleasant trade-off of code efficiency for enjoyable
reading and efficient collaborating experiences, don't you agree?
