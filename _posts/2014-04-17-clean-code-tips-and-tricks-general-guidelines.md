---
title        : "Clean Code Tips and Tricks: General Guidelines"
date         : "2014-04-17"
tags         : [Clean Code, PHP, Programming Guideline]
---

Welcome back fellow computer witches and wizards, let's get back to where we
left on Clean Code Tips and Tricks. This will the be the last installment, we'll
talk about some general guidelines and tips to write better code.

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

Don't only make your dots or arrows neat and tidy, try to beautify everything
you come across, the main idea is to make your codebase easier on the eyes and
more pleasant to work with.

## Clean Up IF Statements

Nobody likes nested-IF statements, they increase the number of indentations
inside a function, which we discussed about in Clean Code Tips and Tricks:
Functions. After extracting excessive code and put them into other functions, we
sometimes still have nested-IF statements left. Let's talk about how to get rid
of them, and what are the benefits.

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

Much better, right? Keep in mind that the  key is to identify what is the *most
expected behavior*.

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
                 throw new FileExtensionNotAllowedException();
             }
         } else {
             throw new UserIsDisabledException();
         }
     } else {
        throw new UserAlreadyHasAvatar();
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
         throw new UserAlreadyHasAvatar();
     }

     if ($this->isDisabled()) {
         throw new UserIsDisabled();
     }

     if (!in_array($avatar->fileExtension, $allowedFileExtensions)) {
         throw new FileExtensionNotAllowed();
     }

     $this->avatar = $avatar;
     $this->save();

     return true;
}
```

Doesn't it look much more straightforward after we refactored it? Think about
what is the *actual thing* or the *most expected behavior* of the function you
are writing and put it at the end first, then think about what are the
conditions or exceptions when this behavior should not be executed, write them
down above and return early. This way you'll always end up with much cleaner
looking conditionals.

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

> "Oh damnation! Who deleted my commented code? I knew it would one day become
> useful, and I was right! Now it fits perfectly with our codebase but I have to
> write it again!

Says no one, ever.



## No God Classes

God classes have too many responsibilities, they are usually bloated and large
in size, they are monolitic and knows too much about business logics that don't
belong to them, and they are usually tightly coupled with other classes.
We'll discuss about S.O.L.I.D principle in a later series, god classes can be
identified and dealt with by following Single Responsibility Principle. I'll
leave that for later discussion, but let me first introduce you a way to
identify a god class.

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
they are grouped with name prefixes tells us a different story. If we were to
name the class properly to reflect all it's attributes, it should be called
`DataFileSplitterAndProcessor`.

Whenever you see a class with *AND* in it's name, or it's member functions are
clearly categorize-able by looking at their prefixes, you know that the class is
having multiple responsibilities and should be refactored. The above class
should be divided into `DataFileSplitter` and `DataFileProcessor`.


## Conclusion

Whew, what a journey. This is the end of our five-installment Clean Code Tips
and Tricks, I hope you enjoyed it as much as I enjoyed preparing it.

We talked about the importance of writing readable code that'll make you and
your co-workers understand better and enjoy working with. And we discussed many
tips and tricks to achieve clean code.

I have to admit that there are limitations such as some of the tips are purely
from my personal experience thus biased with my personal preferences, some
examples in PHP can't represent all other OOP languages, and I couldn't possibly
cover everything there's to know about clean code. Despite all these, the one
take away I hope you get is that ***We need to spend more time writing good
quality code, and less time reading bad ones.***

I plan to write more about design patterns in the comming weeks, please stay
tuned if you are interested, however be informed they won't be introductions or
tutorials as you can find better sources online and in books, it'll be just me
sharing my opinions and experiences with these that I've personal had
experiences with.

Have a nice day and see you soon.
