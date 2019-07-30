---
title        : "Clean Code Tips and Tricks: Introduction & Variables"
date         : "2014-04-11"
tags         : [Clean Code, PHP, Software Principle]
---

I recently organized a workshop for my team members to talk about Clean Code. It
has been a major issue plaguing the team and has been slowing us down by
generating tons of unreadable and unmaintainable code. So I took the oppoturnity
of the Chinese New Year holidays and delivered these three-days talk online.

I had great feedback from my team members and learned a few things myself by
preaching the Clean Code concept along with part of the S.O.L.I.D Principle. The
following weeks brought exiciting and notable results, the code quality improved
significantly just by following these simple tips and tricks. Better still, even
some skilled developers with many years of work experience showed signs of
improvement in code quality.

So I figured it might be of help to other developers if I post these notes
online. I revised the notes I used to deliver the talks and incorporated some of
the new ideas I've learned, now I'm ready to share them with you.

This will be divided into three instalments as the original workshop lasted three
days as well, it'd be a pain to wade through in just one session.

Alright, enough talk, let's jump right in.

## What is this all about?

We want our code clean, and we want it now. It means readable, maintainable,
testable, and robust code.

I would say that understandability is the most important among all other
properties. People would enjoy working with your code if they are easy to
understand.

Maintainablity is the ability to freely extend a piece of existing code  without
having to modify too much. Having great maintainability is definitely
profitable for the long haul, you wouldn't be afraid of requirement changes so
much anymore.

Testability makes it easier to write tests for your code. Properly test-covered
codebase allows you to write, edit, modify, extend your way deep into the very
core of the software without worrying about your commit breaking too many
existing functionalities.

It also means robustness, write code that are strong, bug-free, rich in features
and exception handling that would still fight like a tough soldier after years
of production.

These characteristics are interconnected, you'd often times find that in order
to adopt one of them properly, you have to implement the other one, it's also
true that when you work for one aspect, others would improve as well.

And that is exactly what we want to achieve here. Since when it comes to
testability, maintainability and robustness, we have to turn our faces to
certain design patterns and principles to have a profound qualitative change to
your codebase. Though this workshop is mainly focused on readablity, you
will find a powerful impact on your codebase, not only do they become easier to
understand, but also more enjoyable to maintain and write tests for.

We do have a brief discussion about S.O.L.I.D principle on day 3, I think it's
great and should be adopted by every developer. I'm planning on writing about
other patterns and principles I've personally had experience with in the near
future, stay tuned if you are interested.

## But Why?

> "When it comes to writing code, an ounce of prevention is worth a pound of cure."

We all had this experience, whether you yourself or one of your co-workers who
wrote code carelessly and irresponsibly, which then led to bugs, performance
issues, and weird system behaviors nobody knew why and how. After days of
debugging, you finally pinpointed that piece of ~~shit~~, um I mean code, you
found it utterly gibberish and unbearably hard to understand. Then after pulling
all your hairs off, you finally understood the code and the cause of the issue,
just to be confronted by the harsh reality that it is nearly impossible to
modify or extend the orginal code without editing *too much* or breaking *too
many* existing functionalities. At some stage along this path, a hard decision
would be made to *rewrite* the whole thing and you wouldn't even be surprised to
find out the exact same issues are there all over again.

It's a vicious cycle with no way but only one way out -- WRITE, CLEAN, CODE.

## Meh, I'm Just Too Busy

No you are not.

It is more important to do the ***right stuff*** than
doing the ***wrong thing*** but being productive and busy as a bee. You are just
being a really naughty bee, now don't be a bad bee when asked to improve your
work ethics by saying, "but I'm already too busy polluting the honey with poison!"

Writing testable, robust and maintainable code is not a waste of time, it's what
you should do as a professional developer.

> “We spend most of our time staring into the abyss rather than power typing.”
> ~Douglas Crockford

Let's face the fact here, we are not like those dope hackers in the movies who
type like their lives depend on it in front of the computer. No, we spend most of
our time looking at ugly code, cursing the guy who committed it and sometimes
even later realized it was you who wrote it.

We spend so much time staring at code that in fact the time ratio of reading and
writing code is 10:1. For each hour of typing code, you spend 10 hours reading
it.

Writing beautiful, elegant, and understandable code is not altruism, it's to
relieve yourself from pulling all-nighters to catch up with schedule, and being
called in the middle of the night to be lashed about a bad bug.

Surprisingly only few people understands schedule delay and infesting bugs are
not because your team members are typing too slow, on the contrary, it's because
you are writing too fast without caring about code quality and have to spend
tons of time wading through horrific death swamp just to extend a new feature or
fix a minor bug. It ***is*** lucrative to spend more time writing better code,
and less time reading bad code.

## It's Your Responsibility

As a professional software developer, your job is not just to implement required
functionalities, you have to deliver them with good practices and code quality.

It is your responsibility to write clean and quality code, not your clients',
not your colleagues', not your team leader's or your boss's, yours. A famous
analogy is that you don't tell your doctor to skip washing their hands because
it saves time.

Every job has their own work ethics, guidelines and procedures they follow to do
their jobs right, so do we. If we want to be treated as professional developers,
then we should act like one.

## Code = Document

Your code is the best document if it's done right. No matter how ellaborate you
document your code, it won't be of any help if the code quality is horrible.

And everytime you update your codebase, corresponding changes should be
committed to the documents as well. If we don't even write good quality code, I
can't imagine how awful the document would look like.

My team maintains no document whatsoever, partly because it brings more overhead
than it brings good for a small team, but mostly because I believe
self-explanatory code invalidate the purpose of documentation.

However, I'm not saying documentless is the definite way to go, many teams still
maintain documentation or wiki about their codebase. But if you and your
co-workers can understand eachother's code without referring to comments or
documentation, then why not?

## Coder = Writer

At the dawn of Computer Science, programming was more of a method to talk to
computers, to let the computers know our purpose and do our deeds. But that has
changed since we invented high level progamming languages and introduced
teamwork to the making of software.

Now we must write for other humanzzz as well, not just computers. Writing code
that is hard to read and maintain by the future you or your co-workers brings
much more trouble than compiling errors and runtime exceptions.

Write as a writer, write stuff you and your co-workers would enjoy reading and
working with.

## Leave It Better Than You Found It

So what if you are working on a gigantic ball of mud codebase and it seems too
late to adopt any of these tips?

Actually it's never too late to start writing clean code, and if you can
convince yourself and you co-workers to improve the dirty code they come across,
you'll eventually have a clean codebase.

Now, note that leave the code you come across better than you found it does not
necessarily mean rewriting or refactoring. Fix the variable names, improve the
structure of the class, divide functions that don't follow Single Responsibility
Principle into smaller ones, and implement meaningful method names would be good
enough to eventually have notable effects.

## Variables

### Use English Words (Duh!?)

This is a huge problem my team suffered, they used to use *PinYin* as variable
names, a method to write Chinese characters using letters to spell the pronunciation.
Problem is there are some guesswork involved before you can understand the
meaning of a variable due to the huge amount of homonyms in Chinese. Plus
using roman letters only can't effectively express intonations, which is crucial
when determining the meaning of a phrase.

It's quite a common thing in China, many teams suffer from this problem. It
takes roughly 5-10 seconds for a native Chinese speaker to figure out the
meaning of a variable name, quite ineffective.

It gets worse when your team has members from different countries or speak diffirent
languages. I've seen lots of variable names written completely or partially in
Malay when I was working in Malaysia, which is a huge pain for me as a foreigner.

I personally estimated the rough amount of frequently used English words when
programming is only 100 - 500, give or take as people work in different types of
businesses. Comparing to high school graduates in China, they have learned at
least 2000 English vocabulary, which is more than enough to write English
variable names.

### Use Nouns Or Stataments

Variables are containers that hold data or state, you shouldn't name them with
verbs or questions.

``` php
// Nouns are the most common variable names.
$comment;
$avatar;
$firstname;
```

``` php
// It sounds more like an action, and should be a function name.
$moveStudent = true;

// Now it looks like a question, questions are more common in method names.
$shouldMoveStudent = true;

// Nice and clear.
$studentShouldBeMoved = true;
```

### Don't Abbreviate, Don't Be Too Verbose Either

It sometimes surprises me how many people still name their variables with C or
even ealier programming language styles decades ago, when efficiency was the
utmost priority. They use abbreviations, single letter temporary variable names,
even some mathmatical terms and habits like `$tot`, `$sqrt`, or `$log`.

Please don't, as I noticed some new developers who didn't witness the C era
picked up the habits from other programmers, but have no idea why they name
like that, or what do they mean.

Not everyone understands your abbreviation, not everyone's feeling nostalgia,
and we don't have to shorten our code to save space anymore.

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

### Be Descriptive

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

// Perfect, especially if you have a $appliedTeachersCount in the context, this avoids ambiguity
$appliedStudentsCount;
```

``` php
/*
 * It's hard to understand the full picture.
 * You are not describing every aspect of it.
 * Is it the average score of all subjects of a particular student?
 * Or is it the average score of the class?
 */
$averageScore = array_sum($scores)/count($scores);

// Much better
$studentAverageScore = array_sum($subjectsScores)/count($subjectsScores);
```

### More Examples

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

### Declare Variables Close To Where You Use Them

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

$hasIneligibleCoupon = false;
foreach ($expiringCoupons as $expiringCoupon) {
    eligibleCouponValues = [30, 100, 250];

    if (!in_array($expiringCoupon->value, $eligibleCouponValues)) {
        $hasIneligibleCoupon = true;
        continue;
    }

    CouponService:extendCoupon($expiringCoupon->id, 30);
}

if (!$hasIneligibleCoupon) return;

EmailService::sendCouponExpiringWarning($user->id);
```

Alight! Thanks for reading. We talked about why we have to write clean code, the
benefits of adopting it and some tips about variables. I hope you enjoyed the
post, we'll talk about functions and methods in the next instalment, later
geeks!
