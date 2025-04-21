---
layout:  post
title:   "Why I Hate Templating Languages"
slug:    why-i-hate-templating-languages
date:    2025-04-21T00:00:00
lastmod: 2025-04-21T00:00:00
tags:    [design,languages]
description:
---

In the past I have worked a lot in web-development. This goes back to the early
2000 when web-sites was primarily just CGI scripts and didn't used any JavaScript
at all. Back then JavaScript also was mostly very slow as most Browser interpreted
JavaScript.

Back in that old times every click on a button, link and so on was a full
request to the browser and a full new page has to be generated on the server
side.

Today this is different. A lot of websites just load once and then updates
itself by just sending JSON request and answers and "updating" the site. I
have worked both ways.

Do you know what I think is interesting? Those *updates* are usually hard todo,
it considers a lot of *state-changes* and sometimes it can be that the actual
content through a series of *updates* can be out of sync with the real data.

Stuff like, you add/delete things and suddenly a button stays active/inactive
because it doesn't *update* correctly. Because the programmer forget to set
the state of a button. State changes can be very very hard.

Then there are **new concepts**, or at least most people think it is new. For
example frameworks that just generates a whole page completely even for the
smallest change. But then a diff is made between the old state and the new state,
and the diff is used to update the current state.

This idea combines functional programming with non-functional programming. You
have two immutable state you can compare as data. The code for updating the current
state is just written once, so you don't have todo that, what means your
code is less error prone.

But you know, back when I was writing CGI scripts, that was already what i anyway
did. We just sent the full generated HTML page back, without *updating* the site.

A full Request/Response sometimes made the site a little bit slower to use,
but we didn't had the full problems and bugs *modern* websites have with that
state changes. It also had another benefit, a user always knew that he really
did an action.

Let's say I had to delete an entry from a table in a website. In an old site
i could click a **Delete** button and the full site had to reload, either
it showed the new updated table with it's deleted entry, or it didn't.

Did you every used a website where you *deleted* something, it removed the
entry from the website, but as soon you hit F5 it appeared again? Typical problems
of state handling and managing state. The UI probably send an XMLHTTPRequest
to the server, deleted it from the website but never checked if the request
was sucessful or not.

Maybe *modern* websites do that *correctly* by locking the UI, or give you a
response, maybe a loading icon or *whatever*, but then we also just could go
back to just loading the whole page, wouldn't it be easier?

So anyway, I wanted to talk about templating systems. Back at the early 2000
when web development emerged also the *patterns* about web development emerged. When I
started web development and around 2000, Perl was the dominant language
for creating websites. At the early beginning we also used the [CGI][CGI]
module for creating sites. So let's just look a little bit of the history
how websites are created.

# Printing

A Web Server is technically nothing more as a system to serve files. Can be
HTML files, but usually also any other file as images, CSS files or any other
content.

To allow *dynamic content* CGI was invented. When a file was marked as
*executable* on a Unix system than instead of serving that file, the file was
executed and whatever it print to it's **Standard Output** channel it serves
as the content.

This way we could run any program, start to use databases, allow user logins
and start to create changing websites. Today this is considered normal, back
in the old days, it wasn't.

Anyway let's say i wanted to create a table with data. Then in my programing
code i created the HTML. It maybe looked like this.

```perl
print "<table>\n";
for my $row ( @entries_from_database ) {
    print "<tr>";
    for my $column ( @$row ) {
        print "<td>", html_escape($column), "</td>";
    }
    print "</tr>";
}
print "</table>\n";
```

# HTML Functions

But you know, generating that HTML in strings again and again is a litte bit tedious.
So modules like [CGI][CGI] provided functions to help for that. Today this is
considered *deprecated* (By the CGI module, not by me).

But instead of the above it maybe then looked like this.

```perl
print "<table>\n";
for my $row ( @entries_from_database ) {
    tr(map { td($_) } @$row));
}
print "</table>";
```

the idea is simple. Instead of.

```perl
print "<h1>" . escape($title) . "</h1>";
```

you just write.

```perl
h1($title);
```

# Templating language

Back at the early 2000 web-development was a little bit different, because
we didn't had so much interactive stuff on web-pages going on. I guess maybe 90%
of whatever we served was just HTML, and only a little part of it was dynamic/interactive.

So your CGI scripts was mostly just generating HTML, with only a tiny little bit
of stuff that needed to be generated by the computer. For example we had those
stupid Homepage counters, current time and and so on. Not much.

On top of that web-developers started to use the MVC (Model View Controller)
*Design Pattern* borrowed from Smalltalk to develope there websites.

But somehow people misunderstand that pattern. For a long time people, even today,
teach that the so called *View* should be free of *logic*, what isn't true at all.

But anyway, the whole idea because most of the stuff was anyway HTML, people
*inverted* CGI. Instead of a program that prints HTML, why not have a HTML
document with only a small portions of what needs to be code?

Seems more logical, right?

One of the first Templating Language was **P**ersonal **H**ome **P**age Tools.
Or just short. PHP. A replacement for a collection of Perl scripts.

The idea is as follow. You just can create a HTML document. But whenever you
need some piece of code. You can put it inside an opening `<?php` and a closing `?>`.

Like this.

```
<html>
<body>
    <h1>What Time is it?</h1>
    <p><?php echo php_function_to_get_time() ?></p>
</body>
</html>
```

You now what is funny? **PHP** isn't really used a Templating Language since
a very long time. Every PHP file just starts with.

```php
<?php
    echo "<table>\n";
    for my $row ( @entries_from_database ) {
        for my $columns ( @$row ) {
            echo "<td>" Tr(td($columns));
        }
    }
    echo "</table>";
?>
```

i don't know the exact PHP syntax for a for-loop and don't wanna to look it up,
so maybe the code above is wrong. But i assume you get the point. A templating
language that was designed to serve HTML with some Code snippets in it
basically turned into its own full blown programming language that now servers
HTML string through the call of `echo`.

So what is the difference here between the Perl way by using CGI with functions?
None at all! We just have gone full circle.

And you know what is even more funny. After we have gone full circle, people started
to develope???? Can you guess it?

THEY DEVELOPED TEMPLATING LANGUAGES IN PHP FOR PHP.

So instead of using PHP with `echo` command, now you can use that awesome
templating languages once again. So you basically have a templating language
inside a templating language. I am just waiting that someone develops once again
a templating language inside a templating language inside a template language
and name that language *Inception* or so.

# Template Language in Perl

## XML based

In Perl I have used three different Templating Languages. One was horrible, it
added it's templating stuff as HTML/XML Tags, so it is always *valid* HTML, but
the amount of mess you had to write was just horrible. Looked something like
this.

```
<div tl-if="variable>
</div>
```

The idea was that you could still generate a full valid/visible HTML that could
be opened by a designer just in the browser. Than a developer could enhance that
template with it's statement to generate a dynamic page.

## HTML::Template

Most of the time I used [HTML::Template](https://metacpan.org/pod/HTML::Template)

Looks like this.

```
<html>
<head><title>Test Template</title></head>
<body>
    My Home Directory is <TMPL_VAR NAME=HOME>
    <p>
        My Path is set to <TMPL_VAR NAME=PATH>
    </p>
</body>
</html>
```

I used it a very long time, and hated it. The problem of HTML::Template is that
it is too limiting. It just supports. TMPL_IF, TMPL_LOOP, TMPL_VAR, TMPL_INLCUDE,
TMPL_ELSE.

Back in the days people always put this as an advantage. The idea was the the so called
*View* should have no logic at all. So you have very limited ability todo
any kind of logic in it. You basically just can insert variables. That's it.

But you know. There is a difference between practice and theory. While it seems
like a good idea, it isn't. Assume you have a Date field, and now you want to
print it. But, in which format?

Here are possible formats for the exact same date.

```
19.02.1983
1983-02-19
02/19/1983
19 02 1983
February 19, 1983
19 February 1983
```

So, here are some questions. Which format do you pick and where do you pick the format?

Let's say i just provide a `DATE` variable for my template. Then all i can do is
write `<TMPL_VAR DATE>` to insert that date somewhere.

That means the actual formating of the Date must be part of my Perl code. But wait.
Wasn't the whole idea of a *View* that the *View* decides how it should be formated?
Not my Perl code?

Let's say i only have one code-base (i had in the past) and now you have three
different websites, and each website once to use another format. How
do you do that?

Well, i can tell you how i did it. In the Perl code you basically have to create
all possible variations of a date before hand. So you had a database field
named `date`? Yeah nice, so out of that single field i created variables like.

`DATE_DAY`, `DATE_MONTH`, `DATE_YEAR`, `DATE_MONTH_TEXT`, `DATE_YEAR_SHORT`, ...

and so on. Maybe around 50 different variables so the Template system could pick
and build the correct *View* it needs.

But you know what. Now my Model/Controller code that actually just should
read/create and store data is full of view logic all over the place.

## Template

Then we had [Template::Toolkit](https://metacpan.org/pod/Template). When i have
to pick a Templating system, this is the one I would choose. It has the features
that today nearly every other Tempalting System in other languages copied.

For example to insert a variable I can write.

```
[%   GET variable %]    # 'GET' keyword is optional
[%       variable %]
[%       hash.key %]
[%         list.n %]
[%     code(args) %]
[% obj.meth(args) %]
[%  "value: $var" %]
```

or i also can set variables.

```
[% SET variable = value %]    # 'SET' also optional
[%     variable = other_variable
       variable = 'literal text @ $100'
       variable = "interpolated text: $var"
       list     = [ val, val, val, val, ... ]
       list     = [ val..val ]
       hash     = { var => val, var => val, ... }
%]
```

Besides getting and setting variables it also has for-loops, conditions but
also Pipes/Filters, Macros, Exception handling, ...

or in other words: I have another Programming Language, again ... AGAIN!!!!

# Template Languages are bad

And this is always what happens with every Templating Language i have seen.
When a Templating Language have less features like `HTML::Template` then it is
almost useless not providing much of a benefit at all.

When the Templating Language tries to be more useful, then you basically end
up with a new crappy, totally piece of crap shit language that no compiler
can read, usually IDE has problems, bad error handling, messages, no
IDE auto-completion and a weird horrible/verbose syntax compared to your normal
programming language.

Why do we do this kind of crap?

On top most programming language are now completely interpreted. `Template`
for example has a caching system. `Template` generates Perl code out of
every `Template`, and the cache updates when the `Template` changes. This
caching is also done for speed. Because directly executing Perl is faster than
writing an interpreter in Perl that can interpret/parse `Template`.

But you see how useless all of that is? How much time is spent always creating
again and again Templating Languages that are actually programming languages
just more worse?

For example and If condition in Template look like this.

```
[% IF condition %]
    content
[% ELSIF condition %]
    content
[% ELSE %]
    content
[% END %]
```

but you know what. This is what i would have written directly in Perl, even
without a module.

```perl
if ( $condition1 ) {
    print $contentA;
}
elsif ( $condition2 ) {
    print $contentB;
}
else {
    print $contentC;
}
```

But you know, i rather write Perl instead of `Template`. Because.

* It's easier to write
* A full programming language like Perl still has more features compared to `Template`
* Perl is supported by my IDE
* Syntax checking, error handling of Perl Code.
* I also can write/create functions.

The limitation of the last one is usually bypassed by a lot of Templating
Systems by creating small Template snippets you can just include/embed and
pass variables, or usually set some global context that is used.

HORRIBLE!!!

It's like

```perl
$global_variable = ...;
function(); # this function then uses $global_variable
```

instead of

```perl
function($variable)
```

But this isn't anything. Let's go back to our Date Formating. An often needed
task. So `Template` also offers *Plugins*. Now i can do.

```
[% USE date %]

# use current time and default format
[% date.format %]

# specify time as seconds since epoch
# or as a 'h:m:s d-m-y' or 'y-m-d h:m:s' string
[% date.format(960973980) %]
[% date.format('4:20:36 21/12/2000') %]
[% date.format('2000/12/21 4:20:36') %]
```

but then again. What's the point of that? Now my Templating System basically
offers me a module/library system, like a full Programing Language.

Seriously, you know what would be easier. If you just would stop using a Templating
System and just use Perl Code to generate the View. So your *View* is just
Perl Code. Not another porgramming language.

# MVC

This leads back to the idea of MVC. Look the idea of MVC comes from Smalltalk.
It was used to develope GUI applications and the idea was to seperate your
code into three different parts.

One code does the primary logic. It basically acts as a library you can call.
A High Level API. For example you could have functions like `get_user_by_id()`.
`create_new_user()`, and other stuff. This is the Part the **Model** should
handle.

Then you have the **View**. Also this one is just Program code, nothing different.
But it handles formating. For example `get_user_by_id()` could return a
data-structure returning all information to a user. Then a function like

`user_as_html()` would already be your View!

you also could generate things like. `user_as_json()`, `user_as_xml()`, `user_as_ascii()`
and so on. You can generate a dozens of different Views for the same data. But
the View still can, and maybe should be, your programming language!

Why learn and develope another crappy Templating Language?

The **Controller** is the thing that connects both. For example in an Web-Application
it is the part that reads the stuff sent from the Webbrowser and
the HTTP Request or other environment variables, reads cookie information from
database and so on. Does the right calls to your **Model** then generates the **View**.

It's always the same circle.

1. A request, Change, Action happens (however you wanna call that). **Controller**
2. Update your Global State (like database). **Model**
3. Generate a view from that State. **View**

you can basically develope any application like this. Stuff like **MVVM** and so
on are just minor changes to the same idea. You even can develope games this way.

The **Controller** is the part reading the user input. Your actions, be it from
your keyboard, game controller or network, then update the global Game State, this
is your **Model**. Then you generate a picture through your Graphics card from this
**View**. Preferably with at least 60fps.

You also could for example generate JSON-Data from the global state. Why? Maybe
you wanna send those information through the network and create a multiplayer
game? Then you need to share and send data and need yet another representation
of your data instead of an image.

You don't have to generate separted namespaces and call the one Controller, Model
and View, you don't need to make three libraries out of it. You even just can
use functions/procedures for it. But it is important that a single function only
does one thing. Means either it processes Input/Actions, updates the Model
or generates a View out from a Model. It all can be in a single file,
written in a single Programming language.

[CGI]:https://metacpan.org/dist/CGI/view/lib/CGI.pod
