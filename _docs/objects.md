---
layout: docs
title: Objects
short_title: Objects
---

# How docassemble uses objects

[Python] allows [object-oriented programming] and so does
**docassemble**.

Here is a non-object-oriented way of saying hello to the user by name:

{% include side-by-side.html demo="hello-not-oop" %}

A better way is to define `user` as a **docassemble** object,
[`Individual`].

{% include side-by-side.html demo="hello-oop" %}

As explained in the [fields] section, variable names cannot contain
any punctuation other than the underscore.  So while `user_first_name`
is a valid variable name, `user.name.first` must be referring to
something different.  Periods in [Python] are used to refer to the
"attributes" of "objects."

An object is a special type of variable.  Rather than being a piece of
text, like `user_first_name` is, the variable `user` is an "object"
that is an "instance" of the "class" known as [`Individual`].

Using objects in **docassemble** requires a little bit of setup using
[initial blocks].  [`Individual`] is defined in the
[`docassemble.base.util`]<span></span> [Python module], so it was necessary to
bring that module into the interview with a [`modules`] block.  It was
also necessary to use an [`objects`] block to declare that `user` is
an instance of the class [`Individual`].

Objects have "attributes."  In the above example, `name` is an
attribute of the object `user`.  And `name` is itself an object (it is
an instance of the class [`IndividualName`], though you would need to
look at the [source code] to know that) with attributes `first` and
`last`.  The attributes `first` and `last` are not objects, but rather
pieces of text.  Anything before a `.` is an object, and anything
after the `.` is an attribute of the object.

Objects also have "methods," which are functions that return a value
based on the attributes of the object.  For example,
`user.age_in_years()` will return the current age of the `user` based
on the date defined in the attribute `user.birthdate`:

{% include side-by-side.html demo="age_in_years" %}

Methods are similar to attributes in that they are written with a `.`
before them.  The difference is that they run code to produce a value,
rather than simply accessing a stored value.  You can tell by the
presence of parentheses whether a method is being used.

Using objects in your interviews has a number of advantages over
using plain variables.

<a name="generic"></a>The first advantage is that you can write
[`generic object`] questions.  (See [modifiers] for documentation of
the [`generic object`] feature.)

For example, if you need to collect the phone numbers of three people,
the `grantor`, the `grantee`, and the `trustee`, you don't have to
write separate questions for `grantor.phone_number`,
`grantee.phone_number`, and `trustee.phone_number`.  You can write one
question to collect `x.phone_number`, where `x` is a "generic object"
that acts as a stand-in for any object of type `Individual`.

{% include side-by-side.html demo="generic-object-phone-number" %}

Any time **docassemble** needs to know the phone number of an
[`Individual`], this question will allow it to ask the appropriate question.

In the question text above, [`possessive()`] is a "method" that you can
use on any instance of the [`Individual`] class.  If `trustee`'s name is
Fred Smith, `trustee.possessive('phone number')` returns "Fred Smith's
phone number."  The method is pretty smart; `user.possessive('phone
number')` will return "your phone number."

Using objects also allows you to have different variables that refer
to the exact same thing.  For example, if `user` is already defined as
an object and you run this code:

{% highlight python %}
trustee = user
{% endhighlight %}

then you will define the variable `trustee` as being equivalent to the
`user` object.  `trustee.name.first` will always return the same thing
as `user.name.first`, and `trustee.phone_number` will always return
the same thing as `user.phone_number`.  In addition,
`trustee.possessive('phone number')` will return "your phone number."
You can write code that checks for the equivalence of objects, using
the `is` operator:

{% include side-by-side.html demo="user-is-trustee" %}

Object methods allow you to have a standard way of expressing
information even though the methods used to gather the information may
vary depending on the circumstances.  For example, the
[`age_in_years()`] function, discussed above, first looks to see if the
attribute `age` is defined, and if so will return that instead of
asking for the `birthdate` attribute:

{% include side-by-side.html demo="testage2" %}

Although objects are a fairly complicated concept, as you can see,
they allow you to write code that looks much like plain English.

In part, this is because objects allow you to do complicated things in
an implicit way.  For example, writing `${ grantee }` in a [Mako]
template will return the name of the grantee.  The interview
implicitly calls the method `__str()__` on `grantee`.
`grantee.__str()__` in turn calls `grantee.name.full()`, which strings
together the `grantee`'s full name from its constituent parts
(`name.first`, `name.middle`, `name.last`, and `name.suffix`), all but
the first of which are optional and will not be included if they are
not defined.

Note that object methods may depend upon particular attributes of
objects being defined.  If an attribute is needed but not defined,
**docassemble** will go looking for a `question` or `code` block that
defines the attribute.  For example, if you write this in a question:

{% highlight text %}
Remember that ${ trustee.possessive('phone number') } is
${ trustee.phone_number }.
{% endhighlight %}
    
then in order to ask the question, **docassemble** may ask you for the
trustee's name (so it can say "Remember that John Smith's phone number
is ..."), and then ask for the trustee's `phone_number` if it is not
already defined.

# Writing your own classes

If you are prepared to write your own [Python] code, it is pretty easy
to write your own classes.

For example, you could create your own [package] for interviews
related to cooking.

You would start by using the [package system] to create a
**docassemble** package called `cooking`, which would be given the
name `docassemble.cooking` (interview packages are subpackages of the
`docassemble` namespace package).

You would go into the package and edit the file
`docassemble/cooking/objects.py` and set the contents to the
following:

{% highlight python %}
from docassemble.base.core import DAObject

class Recipe(DAObject):
    def summary(self):
        return "#### Ingredients\n\n" + self.ingredients + "\n\n#### Instructions\n\n" + self.instructions
{% endhighlight %}

Your class `Recipe` needs to "inherit" from the basic **docassemble**
object called `DAObject`.  If you did not do this, **docassemble**
would not be able to ask questions to define attributes of `Recipe` objects.

The purpose of the `summary()` method is to summarize the contents of
the recipe.  It makes use of the attributes `ingredients` and
`instructions`.

If you are not familiar with [Python], `\n` inside quotation marks
indicates a line break and `+` in the context of text indicates that
the text should be strung together.  In [Markdown], `####` at the
start of a line indicates that the line is a section name.

Once you install the package on your server, you can use your class in
an interview:

{% include side-by-side.html demo="madlibs" %}

By the way, there is way to write the `summary()` method that is more
friendly to other interview authors:

{% highlight python %}
return "#### " + word('Ingredients') + "\n\n" + self.ingredients + "\n\n#### " + word('Instructions') + "\n\n" + self.instructions
{% endhighlight %}

If you use the [`word()`] function in this way, other people will be
able to use `docassemble.cooking.objects` in non-English interviews
without having to edit your code.  All they would have to do is
include the words `Ingredients` and `Instructions` in a translation
[YAML] file referenced in a `words` directive in the **docassemble**
[configuration].

## Using global variables in your classes

Normally in [Python] you can use global variables to keep track of
information that your methods need to know but that is not passed in
arguments to the methods.  For example, if you wanted to keep track of
whether to use Celsius or Fahrenheit when talking about temperatures,
you might be tempted to write:

{% highlight python %}
from docassemble.base.core import DAObject

temperature_type = 'Celsius'

class Recipe(DAObject):
    def summary(self):
        return "#### Ingredients\n\n" + self.ingredients + "\n\n#### Instructions\n\n" + self.instructions
    def get_oven_temperature(self):
        if temperature_type == 'Celsius':
            return str(self.oven_temperature) + ' °C'
        elif temperature_type == 'Fahrenheit':
            return str(self.oven_temperature) + ' °F'
        elif temperature_type == 'Kelvin': 
            return str(self.oven_temperature) + ' K'
{% endhighlight %}

(The `str()` function is a Python function that converts something to
text.  Here, it is necessary because `self.oven_temperature` may be a
number, and [Python] will complain if you ask it to "add" text to a
number.)

Then you to change the `temperature_type` from an interview, you might
write:

{% highlight yaml %}
---
modules:
  - docassemble.cooking.objects
---
initial: True
code: |
  if user_is_scientist:
    temperature_type = 'Kelvin'
  elif user_country in ['United States', 'Great Britain']:
    temperature_type = 'Fahrenheit'
...
{% endhighlight %}

This would be effective at changing the `temperature_type` variable
because the `modules` block loads all the names from
`docassemble.cooking.objects` into the variable store of the
interview, including `temperature_type`.

However, this is not [thread-safe] and it will not work correctly 100%
of the time.  If your server is under heavy load, users might randomly
be advised to turn their ovens to 350 degrees Celsius, which would
scorch the food.  This is because the variable `temperature_type`
exists at the level of the web server process, and the process might
be supporting several users simultaneously (in different "threads" of
the process).  Between the time one user sets `temperature_type` to
`Fahrenheit` and tries to use it, another user inside the same process
might set `temperature_type` to `Celsius`.

Therefore, it is important that you do not use global variables when
you write your own classes.  The simplest way to get around this
problem is to use the [`set_info()`] and [`get_info()`] functions from
[`docassemble.base.util`]:

{% highlight python %}
from docassemble.base.core import DAObject
from docassemble.base.util import get_info

class Recipe(DAObject):
    def summary(self):
        return "#### Ingredients\n\n" + self.ingredients + "\n\n#### Instructions\n\n" + self.instructions
    def get_oven_temperature(self):
        if get_info('temperature_type') == 'Celsius':
            return str(self.oven_temperature) + ' °C'
        elif get_info('temperature_type') == 'Fahrenheit':
            return str(self.oven_temperature) + ' °F'
        elif get_info('temperature_type') == 'Kelvin': 
            return str(self.oven_temperature) + ' K'
{% endhighlight %}

Then from your interview you can include [`docassemble.base.util`] as
one of the [`modules`] and then run [`set_info()`] in [`initial`]
code:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
  - docassemble.cooking.objects
---
initial: True
code: |
  if user_is_scientist:
    set_info(temperature_type='Kelvin')
  elif user_country in ['United States', 'Great Britain']:
    set_info(temperature_type='Fahrenheit')
  else:
    set_info(temperature_type='Celsius')
...
{% endhighlight %}

The values set by [`set_info()`] are forgotten after the user's screen
is prepared.  Therefore, it is necessary to run [`set_info()`] in an
[`initial`] code block so that values like `temperature_type` are put
in place before they are needed.

If you are an advanced programmer, you can do what
[`docassemble.base.util`] does and use Python's [threading module] to
store global variables.

{% highlight python %}
from docassemble.base.core import DAObject
import threading

__all__ = ['set_temperature_type', 'get_temperature_type', 'Recipe']

this_thread = threading.local

def set_temperature_type(type):
    this_thread.temperature_type = type

def get_temperature_type():
    return this_thread.temperature_type

class Recipe(DAObject):
    def summary(self):
        return "#### Ingredients\n\n" + self.ingredients + "\n\n#### Instructions\n\n" + self.instructions
    def get_oven_temperature(self):
        if this_thread.temperature_type == 'Celsius':
            return str(self.oven_temperature) + ' °C'
        elif this_thread.temperature_type == 'Fahrenheit':
            return str(self.oven_temperature) + ' °F'
        elif this_thread.temperature_type == 'Kelvin': 
            return str(self.oven_temperature) + ' K'
{% endhighlight %}

We added an `__all__` statement so that interviews can a `module`
block including `docassemble.cooking.objects` does not clutter the
variable store with extraneous names like `threading`.  We also added
functions for setting and retrieving the value of the "temperature
type."

The temperature type is now an attribute of the object `this_thread`,
which is an instance of `threading.local`.  This attribute needs to be
set in `initial` code that will run every time a screen refreshes.

Now in your interview you can do:

{% highlight yaml %}
---
modules:
  - docassemble.cooking.objects
---
initial: True
code: |
  if user_is_scientist:
    set_temperature_type('Kelvin')
  elif user_country in ['United States', 'Great Britain']:
    set_temperature_type('Fahrenheit')
  else:
    set_temperature_type('Celsius')
...
{% endhighlight %}

Note that you do not need to worry about whether your global variables
are [thread-safe] if they do not change from interview to interview.

For example, if you only wanted to allow people to change the
temperature type from the **docassemble** [configuration], you could
do the following in your [Python module]:

{% highlight python %}
from docassemble.base.core import DAObject

from docassemble.webapp.config import daconfig
temperature_type = daconfig.get('temperature type', 'Celsius')
{% endhighlight %}

Then your interviews would not have to do anything with `temperature_type`.

Also, you could avoid the complication of global variables entirely if
you are willing to pass the temperature type as an argument to
`get_oven_temperature`:

{% highlight python %}
from docassemble.base.core import DAObject

class Recipe(DAObject):
    def get_oven_temperature(self, type):
        if type == 'Celsius':
            return str(self.oven_temperature) + ' °C'
        elif type == 'Fahrenheit':
            return str(self.oven_temperature) + ' °F'
        elif type == 'Kelvin': 
            return str(self.oven_temperature) + ' K'
{% endhighlight %}

Then you could have this in your interview:

{% highlight yaml %}
---
question: |
  What kind of temperature system do you use?
choices:
  - Celsius
  - Fahrenheit
  - Kelvin
field: temperature_type
---
{% endhighlight %}

and then in your question text you could write:

{% highlight text %}
Set your oven to ${ apple_pie.get_oven_temperature(temperature_type) }
and let it warm up.
{% endhighlight %}
    
# Standard docassemble classes

## <a name="DAObject"></a>DAObject

All **docassemble** objects are instances of the `DAObject` class.
`DAObject`s are different from normal [Python objects] because they
have special features that allow their attributes to be set by
**docassemble** questions.  If `fruit` is an ordinary [Python object]
and you refer to `fruit.seeds` when `seeds` is not an existing
attribute of `fruit`, [Python] will generate an [AttributeError].  But
if `fruit` is a `DAObject`, **docassemble** will intercept that error
and ask a question that offers to define `fruit.seeds`, or ask a
[`generic object`] question for object `DAObject` that offers to
define `x.seeds`.

From the interview author's perspective, `DAObject`s can be treated
like ordinary [Python objects] in most ways, but there are exceptions.

Suppose you try the following:

{% highlight yaml %}
---
modules:
  - docassemble.base.core
---
objects:
  - tree: DAObject
  - long_branch: DAObject
---
mandatory: True
question: |
  The length of the branch is ${ tree.branch.length }.
---
code: |
  tree.branch = long_branch
---
question: |
  What is the length of the branch on the tree?
fields:
  - Length: tree.branch.length
---
{% endhighlight %}

This will result in the following error:

> Found a reference to a variable 'long_branch.length' that could not
> be looked up in the question file or in any of the files
> incorporated by reference into the question file, despite reaching
> the very end of the file.

If you had a question that defined `long_branch.length` or a 
[`generic object`] question for the `x.length` where `x` is a `DAObject`, then
**docassemble** would use that question, but it is not able to ask for
the length of the branch with `tree.branch.length` since the intrinsic
name of the branch is `long_branch`, not `tree.branch`.

However, this will work as intended:

{% highlight yaml %}
---
modules:
  - docassemble.base.core
---
objects:
  - tree: DAObject
---
sets: tree.branch
code: |
  tree.initializeAttribute('branch', DAObject)
---
mandatory: True
question: |
  The length of the branch is ${ tree.branch.length }.
---
question: |
  What is the length of the branch on the tree?
fields:
  - Length: tree.branch.length
---
{% endhighlight %}

The `initializeAttribute` section here creates a new `DAObject` with
the intrinsic name of `tree.branch`, and adds the `branch` attribute
to the object `tree`.

Note that we had to add `sets: tree.branch` to the `code` section with
`tree.initializeAttribute('branch', DAObject)`, but this was not
necessary when the code was `tree.branch = long_branch`.  This is
because **docassemble** reads the code in `code` sections and looks
for assignments with the `=` operator, and keeps track of which code
sections define which variables.  But sometimes variables are assigned
by functions, and **docassemble** does not realize that.  The
`sets: tree.branch` line tells **docassemble** that the code promises
to define `tree.branch`.  See [`sets`] for more information.

One of the useful things about `DAObject`s is that you can write
[`generic object`] questions that work in a wide variety of
circumstances because the questions can use the variable name itself
when forming the text of the question to ask the user.

If you refer to a `DAObject` in a [Mako] template (or reduce it to
text with Python's [str function]), this will have the effect of
calling the `object_name()` method, which attempts to return a
human-friendly name for the object.

For example:

{% include side-by-side.html demo="daobject" %}

Although there is only one question for `x.color`, this question
generates both "What is the color of the turnip?" and "What is the
color of the front gate in the park?"  This is because `object_name()`
is implicitly called and it turns `park.front_gate` into "front gate
in the park."

The `object_name()` method is multi-lingual-friendly.  By using
`docassemble.base.util.update_word_collection()`, you can provide
non-English translations for words that come from variable names, such
as "turnip," "park," and "front gate."  By using
`docassemble.base.util.update_language_function()`, you can define a
non-English version of the `a_in_the_b()` function, which
`object_name()` uses to convert an attribute name like
`park.front_gate` into "front gate in the park."  (It calls
`a_in_the_b('front gate', 'park')`.)  So in a Spanish interview,
`park.front_gate.object_name()` would return "puerta de entrada en el
parque."  (The Spanish version of `a_in_the_b()` will be more
complicated than the English version because it will need to determine
the gender of the second argument.)

<a name="DAObject.object_possessive"></a>A related method of
`DAObject` is `object_possessive()`.  Calling
`turnip.object_possessive('leaves')` will return `the turnip's
leaves`.  Calling `park.front_gate.object_possessive('latch')` will
return `the latch of the front gate in the park`.

The `DAObject` is the most basic object, and all other **docassemble**
objects inherit from it.  These objects will have different methods
and behaviors.  For example, if `friend` is an [`Individual`] (from
[`docassemble.base.util`]), calling `${ friend }` in a [Mako] template
will not return `friend.object_name()`; rather, it will return
`friend.full_name()`, which may require asking the user for the
`friend`'s name.

## <a name="DAList"></a>DAList

A `DAList` acts like an ordinary [Python list], except that
**docassemble** can ask questions to define elements of the list.

Here is a simple "Mad Libs" interview that uses `DAList`s to keep
track of words:

{% include side-by-side.html demo="madlibs" %}

The variable `i` is special.  When the interview encounters
`person[0]` and sees that it is undefined, it will go searching for a
question that offers to define `person[0]`.  If it does not find that,
it will generalize and look for a question that offers to define
`person[i]`.  If that is not found, it will generalize further and
look for a question that offers to define `x[i]`.  Thus, the one
[`generic object`] question, which defines `x[i]` where `x` is a
`DAList`, will ask all of the questions in the interview.

For another example, suppose you want to work with a list of
prospective recipients of an e-mail.  You could define `recipient` as
a `DAList` containing five [`Individual`]s

{% include side-by-side.html demo="testdalist" %}

This will result in the following five questions being asked:

* What is the name of the trustee?
* What is the name of the beneficiary?
* What is the name of the grantor?
* What is the name of the fourth recipient?
* What is the name of the fifth recipient?

<a name="DAList.appendObject"></a>
The `appendObject()` method is similar to the `initializeAttribute()`
method we discussed earlier.  Running
`recipient.appendObject(Individual)` creates a new object of the class
[`Individual`] and adds it to the list.  In the example above, the first
such object is the fourth element in the list, which means that the
intrinsic name of the new object is `recipient[3]`.

A `DAList` can be given a default object type, so that
`appendObject()` can be called without an argument.  This default
object type is controlled by the `.objectFunction` attribute.  For
example, when a `PartyList` object is created, the `.objectFunction`
attribute is set to [`Person`].

If you want greater control over the way the questions are asked, you
could add a [`generic object`] question that is specific to the
recipients that were added with `appendObject()`.  For example:

{% highlight yaml %}
---
generic object: Individual
question: |
  The ${ ordinal(i) } ${ x.object_name() } must have a name.  What is it?
fields:
  - First Name: x[i].name.first
  - Last Name: x[i].name.last
---
{% endhighlight %}

The names of the fourth and fifth recipients are capable of being
asked by this question, since the pattern `x[i]` (where `x[i]` is an
[`Individual`]) matches the intrinsic names `recipient[3]` and
`recipient[4]`.  Since the other [`generic object`] question, which
matches `x` (where `x` is an [`Individual`]) also matches `recipient[3]`
and `recipient[4]`, the order in which you list the questions in the
[YAML] file will determine which one is chosen.  Later-appearing questions
take precedence, so you would need to place the second [`generic
object`] question somewhere after the first [`generic object`] question
in order for it to be chosen.

Other methods available on a `DAList` are:

* <a name="DAList.append"></a>`append(item_to_append)` - adds
  `item_to_append` to the end of the list.  Just like the
  [Python list] method of the same name.
* <a name="DAList.clear"></a><a name="DADict.clear"></a><a
  name="DASet.clear"></a>`clear()` - makes the list empty.
* <a name="DAList.extend"></a>`extend(extension_list)` - adds the
  items in the `extension_list` to the end of the list.  Just like the
  [Python list] method of the same name.
* <a name="DAList.first"></a>`first()` - returns the first element of
  the list; error triggered if list is empty
* <a name="DAList.last"></a>`last()` - returns the last element of the
  list; error triggered if list is empty
* <a name="DAList.does_verb"></a><a name="DADict.does_verb"></a><a
  name="DASet.does_verb"></a>`does_verb(verb)` - like the
  `verb_present()` function from [`docassemble.base.util`], except
  that it uses the singular or plural form depending on whether the
  list has more than one element or not.
* <a name="DAList.did_verb"></a><a name="DADict.did_verb"></a><a
  name="DASet.did_verb"></a>`did_verb(verb)` - like the `verb_past()`
  function from [`docassemble.base.util`], except that it uses the
  singular or plural form depending on whether the list has more than
  one element or not.
* <a name="DAList.as_singular_noun"></a><a
  name="DADict.as_singular_noun"></a><a
  name="DASet.as_singular_noun"></a>`as_singular_noun()` - if the
  variable name is `case.plaintiff`, returns `plaintiff`; if the
  variable name is `applicant`, returns `applicant`.
* <a name="DAList.as_noun"></a><a name="DADict.as_noun"></a><a
  name="DASet.as_noun"></a>`as_noun()` - if the variable name is
  `case.plaintiff`, returns `plaintiffs` or `plaintiff` depending on
  the number of elements in the list; if the variable name is
  `applicant`, returns `applicants` or `applicant` depending on the
  number of elements in the list.
* <a name="DAList.number"></a><a name="DADict.number"></a><a
  name="DASet.number"></a>`number()` - returns the total number of
  elements in the list.  If necessary it will trigger questions that
  ask for all of the elements of the list to be populated.
* <a name="DAList.number_as_word"></a><a
  name="DADict.number_as_word"></a><a
  name="DASet.number_as_word"></a>`number_as_word()` - same as
  `number()`, except that the [`nice_number()`] function is applied to
  the result.
* <a name="DAList.remove"></a>`remove()` -
  removes the given elements from the list, if they are in the list.
* <a name="DAList.comma_and_list"></a><a
  name="DADict.comma_and_list"></a><a
  name="DASet.comma_and_list"></a>`comma_and_list()` - returns the
  elements of the list run through the [`comma_and_list()`] function.
* <a name="DAList.possessive"></a>`possessive()` - if the variable
  name is `plaintiff` and the target is `"fish"`, returns "plaintiff's
  fish" if there is one item in the list, and "plaintiffs' fish" if
  there is more than one item in the list.
* <a name="DAList.pronoun"></a><a name="DADict.pronoun"></a><a
  name="DASet.pronoun"></a>`pronoun()` - returns a pronoun like "you,"
  "her," or "him," "it", or "them," as appropriate, depending on the
  number of items in the list.
* <a name="DAList.pronoun_objective"></a><a
  name="DADict.pronoun_objective"></a><a
  name="DASet.pronoun_objective"></a>`pronoun_objective()` - for a
  `DAList`, this is the same as `pronoun()`.
* <a name="DAList.pronoun_possessive"></a><a
  name="DADict.pronoun_possessive"></a><a
  name="DASet.pronoun_possessive"></a>`pronoun_possessive()` - given a
  word like "fish," returns "her fish," "his fish," or "their fish,"
  as appropriate, depending on the number of items in the list.
* <a name="DAList.pronoun_subjective"></a><a
  name="DADict.pronoun_subjective"></a><a
  name="DASet.pronoun_subjective"></a>`pronoun_subjective()` - returns
  a pronoun like "you," "she," "he," or "they" as appropriate,
  depending on the number of items in the list.
* <a name="DAList.union"></a><a name="DADict.union"></a>`union()` -
  returns a [Python set] consisting of the elements of current list,
  considered as a set, combined with the elements of the other_set.
* <a name="DAList.intersection"></a><a
  name="DADict.intersection"></a>`intersection()` - returns a
  [Python set] consisting of the elements of the current list,
  considered as a set, that also exist in the other_set.
* <a name="DAList.difference"></a><a
  name="DADict.difference"></a>`difference()` - returns a [Python set]
  consisting of the elements of the current list, considered as a set,
  that do not exist in the other_set.
* <a name="DAList.isdisjoint"></a><a
  name="DADict.isdisjoint"></a>`isdisjoint()` - returns `True` if no
  elements overlap between the current list, considered as a set, and
  the other_set.  Otherwise, returns `False`.
* <a name="DAList.issubset"></a><a
  name="DADict.issubset"></a></a>`issubset()` - returns `True` if the
  current list, considered as a set, is a subset of the other_set.
  Otherwise, returns `False`.
* <a name="DAList.issuperset"></a><a
  name="DADict.issuperset"></a>`issuperset()` - returns `True` if the
  other_set is a subset of the current list, considered as a set.
  Otherwise, returns `False`.
* <a name="DAList.gather"></a><a name="DADict.gather"></a><a
  name="DASet.gather"></a>`gather()` - causes the elements of the list
  to be gathered and named.  Returns `True`.
* <a name="DAList.reset_gathered"></a><a name="DADict.reset_gathered"></a><a
  name="DASet.reset_gathered"></a>`reset_gathered()` - causes the
  list to be considered not-yet-gathered.  This can be used to query
  the user to add elements to a list that may already be considered
  complete.  If called with the optional keyword argument `recursive`
  set to `True` (the default is `False`), the lists inside the list
  are also marked as not-yet-gathered.

If you refer to a list in a [Mako] template (e.g., `The applicants
include: ${ applicant }`) or convert it to text with the
[str function] (e.g. (`str(applicant)`) in [Python] code, the result
will be the output of the `comma_and_list()` method.

The `DAList` uses the following attributes:

* `object_type`: a class of type [`DAObject`] or subclass thereof, or
  `None`.  Initially, this is set to `None`.  If set to an object
  type, such as `DAObject` or `Individual`, then new items will be
  created as objects of this type.
* `gathered`: a boolean value, initially undefined.  It is set to
  `True` when then all of the elements of the list are defined.
* `elements`: a [Python list] containing the elements of the list.
* `are_there_any`: a boolean value, initially undefined, indicating
  whether any values should be gathered.  The expectation is that the
  interview will define a [question] or [code block] that defines this
  attribute.
* `is_there_another`: a boolean value, initially undefined, indicating
  whether there are any additional values that should be gathered.
* `auto_gather`: a boolean value, initially `True`, indicating whether
  the interview should attempt to 

For more information about using [`DAList`] objects, see the section
on [groups].

## <a name="DADict"></a><a name="DADict.initializeObject"></a><a name="DADict.new"></a>DADict

A `DADict` acts like a [Python dictionary] except that dictionary keys
and values can be defined through **docassemble** questions.

To add a value that is a new **docassemble** object, you need to call
the `initializeObject()` method.

For example:

{% include side-by-side.html demo="dadict" %}

The `DADict` also uses a similar method called `.new()`.  This method
initializes a new object and makes it an entry in the dictionary.  For
example, if the dictionary is called `positions`, calling
`positions.new('file clerk', 'supervisor')` will result in the
creation of the object `positions['file clerk']` and the object
`positions['supervisor']`.  The type of object is given by the
`object_type` attribute, or `DAObject` if `object_type` is not set.
You can also pass a [list] and it will unpack the list, initializing
dictionary entries for each value.

`DADict`s use the same attributes that [`DAList`]s use.  It also uses:

* `new_item_name`: a text value, initially undefined, indicating the
key of a new item being gathered into the dictionary.
* `new_item_value`: a value, initially undefined, indicating the value
of a new item being gathered into the dictionary.  This is only used
when the `.object_type` of the [`DADict`] is not set.

`DADict`s use the same methods that [`DAList`]s use, except for
`.appendObject()`, `.append()`, `.remove()`, `.discard()`,
`.extend()`, `.first()`, and `.last()`.

It also uses the following methods, which correspond with the same
methods of the [Python dict].

* <a name="DADict.keys"></a>`keys()`
* <a name="DADict.values"></a>`values()`
* <a name="DADict.update"></a>`update()`
* <a name="DADict.pop"></a>`pop()`
* <a name="DADict.popitem"></a>`popitem()`
* <a name="DADict.setdefault"></a>`setdefault()`
* <a name="DADict.get"></a>`get()`
* <a name="DADict.copy"></a>`copy()`
* <a name="DADict.has_key"></a>`has_key()`
* <a name="DADict.items"></a>`items()`
* <a name="DADict.iteritems"></a>`iteritems()`
* <a name="DADict.iterkeys"></a>`iterkeys()`
* <a name="DADict.itervalues"></a>`itervalues()`

For most purposes, your code can treat a `DADict` object just like a
[Python dictionary].

{% highlight yaml %}
---
objects:
  things: DADict
---
code: |
  if 'abc' in things:
    gathered_abc = True
  else:
    gathered_abc = False
---
code: |
  if len(things) > 0:
    there_are_things = True
  else:
    there_are_things = False
---
mandatory: True
code: |
  things['abc'] = some_variable
---
code: |
  description = ''
  for key, value in things.iteritems():
    description += "* " + key + ": " + value + "\n"
---
{% endhighlight %}

For more information about using [`DADict`] objects, see the section
on [groups].

## <a name="DASet"></a>DASet

A `DASet` is like a [`DADict`] and a [`DAList`], except it acts like a
[Python] "[set]."

{% highlight yaml %}
---
objects:
  issues: DASet
---
code: |
  if user_needs_to_apply:
    issues.add('application')
---
{% endhighlight %}

`DASet`s use the same methods that [`DAList`]s use, except for
`.appendObject()`, `.append()`, `.extend()`, `.first()`, and
`.last()`.  It also uses the following methods, which correspond with
the methods of the [Python set].

* <a name="DASet.add"></a>`add()`
* <a name="DASet.copy"></a>`copy()`
* <a name="DASet.discard"></a>`discard()`
* <a name="DASet.difference"></a>`difference()`
* <a name="DASet.intersection"></a>`intersection()`
* <a name="DASet.isdisjoint"></a>`isdisjoint()`
* <a name="DASet.issubset">`issubset()`
* <a name="DASet.issuperset"></a>`issuperset()`
* <a name="DASet.pop"></a>`pop()`
* <a name="DASet.remove"></a>`remove()`
* <a name="DASet.union"></a>`union()`

For more information about using [`DASet`] objects, see the section
on [groups].

## <a name="DAFile"></a>DAFile

A `DAFile` object is used to refer to a file, which might be an
uploaded file, an assembled document, or a file generated by code.  It
has the following attributes:

* `filename`: the filename (e.g., `complaint.pdf`).
* `mimetype`: the MIME type of the file.
* `extension`: the file extension (e.g., `pdf` or `rtf`).
* `number`: the internal integer number used by **docassemble** to
  keep track of documents stored in the system.  (You will likely never need to
  use this.)
* `ok`: this is `True` if the `number` has been defined, and is
  otherwise `False`.  (You will likely never need to use this,
  either.)

You might work with `DAFile` objects in the following contexts:

* Your interview contains a [document upload field].  The variable
  representing the upload will be set to a [`DAFileList`] object after
  the upload has been done.  If the
  variable name is `pretty_picture`, then `pretty_picture[0]` will be
  a `DAFile` object.
* Your interview assembles a document and the document is assigned to
  a [`variable name`].  If the variable name is
  `motion_to_reconsider`, then `motion_to_reconsider` will be a
  [`DAFileCollection`] object, the attributes of which represent the
  various formats of the document.  For example,
  `motion_to_reconsider.pdf` (the `.pdf` here is an attribute, not a
  file extension) will be a `DAFile` object representing the PDF
  version of the document.
* Your interview contains code that needs to create a file.  You can
  use an [`objects`] block to create a blank [`DAFile`] object.  Then
  you would call [`.initialize()`](#DAFile.initialize) to give the
  file a name and a presence on the file system.
  
You can call [`.path()`](#DAFile.path) on a `DAFile` object to get the
actual file path.  Using the file path, can manipulate the underlying
file directly in whatever way you want.  However, the `DAFile` object
has a number of built-in methods for doing common things with files,
so it is a good idea to use the methods whenever possible.

While the `DAFile` object is saved in your interview dictionary like
any other variable, the content of the file may be stored on
[Amazon S3], [Azure blob storage], or the file system, depending on
the server's configuration.  The path you obtain from
[`.path()`](#DAFile.path) might be different from one screen of your
interview to another.  You should not save the path to a variable and
expect to be able to use that variable across screens of the
interview.  Rather, you should always access the file through the
`DAFile` object, using its built-in methods (such as
[`.path()`](#DAFile.path)).  These methods contain code that
automatically accounts for the fact that the file might be stored in
the cloud.  For example, if you use [Amazon S3], then when you call
[`.path()`](#DAFile.path)), this will cause the file to be retrieved
from [Amazon S3] and placed into a temporary directory.

The methods of `DAFile` are the following:

<a name="DAFile.initialize"></a>The `.initialize()` method transforms
a fresh, uninitialized `DAFile` object (e.g., a `DAFile` object
created by the [`objects`] block) into an object that can actually be
used as a file.  The method takes the optional keyword parameters
`filename` or `extension`.  The `.initialize()` method can be used as
follows (where `myfile` is a `DAFile` object):

* `myfile.initialize(filename='image.jpg')` - `filename` will be
  `image.jpg`, `extension` will be `jpg`, `mimetype` will be
  `image/jpeg`.
* `myfile.initialize(extension='jpg')` - `filename` will be
  `file.jpg`, `extension` will be `jpg`, `mimetype` will be
  `image/jpeg`.
* `myfile.initialize()` - `filename` will be `file.txt`, `extension`
  will be `txt`, `mimetype` will be `text/plain`.

If the object has already been initialized, the `.initialize()` method
can safely be called on it, but this will only have the effect of
calling `.retrieve()` on it, and the `filename` and `extension`
parameters will not overwrite existing values.

The following example uses the [Python Imaging Library] to create a
JPEG image.

{% include side-by-side.html demo="dafile" %}

<a name="DAFile.show"></a>The `.show()` method returns markup that
displays the file as an image.  This method takes an optional keyword
argument, `width`, which can be set to, e.g., `'1in'`, `'44mm'`, or
`'20pt'`.  See [inserting images] for more information about this
markup.

In the context of a [Mako] template, writing `${ myfile }` is
equivalent to writing `${ myfile. show() }` (where `myfile` is a
`DAFile` object).

<a name="DAFile.path"></a>The `.path()` method returns a complete file
path that you can use to read the file or write to the file.

<a name="DAFile.url_for"></a>The `.url_for()` method returns a URL at
which the file can be accessed.  The URL should only be used in the
context of the user's session and the user's web browser.  For
example, if you are using cloud storage as your form of
[data storage], the URL will link directly to the cloud and will
expire after an hour.  If you are not using cloud storage, the server
will only allow access to the file to the current user.

<a name="DAFile.retrieve"></a>The `.retrieve()` command ensures that a
stored file is ready for use on the system.  Calling `.retrieve` is
necessary because if **docassemble** is configured to use [Amazon S3]
or [Azure blob storage], documents are stored in the cloud, and the
server accesses them by copying them from the cloud to the server and
then copying them back to the cloud.  If the file does not exist yet,
calling `.retrieve()` will generate an error.

<a name="DAFile.slurp"></a>The `.slurp()` method reads the contents of
the file and returns them as a value.

{% highlight python %}
contents = the_file.slurp()
{% endhighlight %}

<a name="DAFile.readlines"></a>The `.readlines()` method reads the
contents of the file, line-by-line, and returns the lines as a list.

{% highlight python %}
for line in the_file.readlines():
  if line.startswith('header'):
    header_line = line
{% endhighlight %}

<a name="DAFile.set_mimetype"></a>The `.set_mimetype()` method sets
the `.mimetype` and `.extension` attributes based on the given
[MIME type].

{% highlight python %}
the_file.set_mimetype('image/jpeg')
{% endhighlight %}

<a name="DAFile.write"></a>The `.write()` method takes a variable as
an argument and writes the contents of the variable to the file.

{% highlight python %}
contents = the_file.slurp()
contents = re.sub(r'swords', r'ploughshares', contents)
the_file.write(contents)
{% endhighlight %}

<a name="DAFile.copy_into"></a>The `.copy_into()` method overwrites
any existing contents of the file with the contents of the file given
as an argument.

<a name="DAFile.from_url"></a>The `.from_url()` method overwrites
any existing contents of the file with the contents of the given URL.

{% highlight python %}
the_file.copy_into(other_file.path())
{% endhighlight %}

<a name="DAFile.commit"></a>The `.commit()` method ensures that
changes to the file are stored permanently.  Under normal
circumstances, **docassemble** will automatically commit changes when
the interview is finished processing (i.e. right before a new screen
appears), but `.commit()` can be called to ensure that changes are
written, just in case there is an error.

{% highlight python %}
the_file.commit()
{% endhighlight %}

## <a name="DAFileCollection"></a>DAFileCollection

`DAFileCollection` objects are created internally by **docassemble**
in order to refer to a document assembled by an
[`attachment`]/[`attachments`] block.  When such a block features a
`variable name`, then a variable by that name will be defined as a
`DAFileCollection` object.  The object is called a "collection"
because an [`attachment`] can have multiple formats: PDF, RTF, etc.

A `DAFileCollection` object has attributes for each file type
generated (e.g., `pdf` or `rtf`), where the attributes are objects of
type [`DAFile`].

For example, if the variable `my_file` is a `DAFileCollection`,
`my_file.pdf` will be a [`DAFile`] containing the PDF version, and
`my_file.rtf` will be a [`DAFile`] containing the RTF version.

In addition, each `DAFileCollection` object also has an attribute
`.info` containing information about the attachment, such as the name,
the filename (before an extension is added), and a description.  The
`.info` attribute is a dictionary containing the following keys:

* `name` is the printable name of the document
* `filename` is the base name used to generate the filename.  If
  `filename` is `custody_complaint`, the PDF file will be `custody_complaint.pdf`.
* `description` is the description of the attachment.

## <a name="DAFileList"></a>DAFileList

A `DAFileList` is a [`DAList`], the elements of which are expected to be
[`DAFile`] objects.

When a question has a field with a `datatype` for a file upload (see
[fields]), the variable will be defined as a `DAFileList` object
containing the file or files uploaded.

<a name="DAFileList.show"></a>It has one method, `.show()`, which
inserts markup that displays each file as an image.  This method takes
an optional keyword argument, `width`.

When included in a [Mako] template, a `DAFileList` object will effectively
call `show()` on itself.

## <a name="DAEmail"></a>DAEmail

The [e-mail receiving] feature converts actual e-mails into objects of
type `DAEmail`.  These objects have the following attributes:

* `short`: the code that was assigned by [`interview_email()`]
  (e.g. `ugjrye`) in order to create the e-mail address to which this
  e-mail was sent (e.g. `ugjrye@help.example.com`).
* `key`: the `key` that was passed to [`interview_email()`], or `None` if no
  `key` was passed.
* `index`: the `index` that was passed to [`interview_email()`], or
  `None` if no index was passed.
* `address_owner`: the e-mail address of the user whose identity and
  privileges were being used when [`interview_email()`] was called.
  If the user was not logged in when [`interview_email()`] was called,
  this will be `None`.
* `to_address`: a [`DAEmailRecipientList`] object representing the
  recipients of the e-mail.
* `cc_address`: a [`DAEmailRecipientList`] object representing the
  "carbon copy" recipients of the e-mail.
* `from_address`: a [`DAEmailRecipient`] object representing the
  sender of the e-mail.
* `reply_to`: a [`DAEmailRecipient`] object representing the
  the [Reply-to] header of the e-mail.
* `return_path`: a [`DAEmailRecipient`] object representing the
  the [Return-path] header of the e-mail.
* `subject`: the subject line of the e-mail.
* `datetime_message`: a [`datetime`] object representing the stated date
  and time of the e-mail message.
* `datetime_received`: a [`datetime`] object representing the actual
  date and time of the message.
* `body_text`: a [`DAFile`] object referring to a file containing the
  plain text version of the e-mail.  If the e-mail did not contain a
  plain text version, `body_text` will be `None`.
* `body_html`: a [`DAFile`] object referring to a file containing the
  [HTML] version of the e-mail.  If the e-mail did not contain a
  [HTML] version, `body_html` will be `None`.
* `attachment`: an object of type [`DAFileList`] containing any
  files that were attached to the e-mail.  Each attachment is a
  [`DAFile`] object.
* `headers`: a [`DAFile`] object referring to a file containing a
  [JSON] representation of the headers of the e-mail.  The format of
  the [JSON] file is a [list], where each item in the [list] is a
  [list] with two elements, the first of which is the name of the
  header (e.g., `To`, `From`), and the second element is the value.

## <a name="DAEmailRecipient"></a>DAEmailRecipient

A `DAEmailRecipient` object is used within [`DAEmail`] objects to
represent a single e-mail address and the name associated with the
e-mail address.

It has two attributes:

* `address`: the e-mail address (e.g., `fred1985@yahoo.com`).
* `name`: the name of the owner of the address (e.g., `Fred Smith`).

### <a name="DAEmailRecipient.email_address"></a>`.email_address()`

If `recipient` is a `DAEmailRecipient`, then calling
`recipient.email_address()` will return the person's name followed by
the person's e-mail address, in the standard e-mail format.  E.g.,
`'Fred Smith <fred1985@yahoo.com>'`.  If the name is not defined,
it will simply return the e-mail address (`fred1985@yahoo.com`).

You can suppress the inclusion of the person's name by setting the
optional keyword parameter `include_name` to `False`.

This method is intended to allow you to use `DAEmailRecipient` objects
in much the same way as [`Person`] objects are used when sending
e-mails with [`send_email()`].  (See the
[`.email_address()`] method for [`Person`] objects).

### <a name="DAEmailRecipient.exists"></a>`.exists()`

The `.exists()` method returns `True` if the `.address` attribute has
been defined, and `False` otherwise.

## <a name="DAEmailRecipientList"></a>DAEmailRecipientList

A `DAEmailRecipientList` is a [`DAList`] of [`DAEmailRecipient`] objects.

## <a name="DATemplate"></a>DATemplate

The [`template`] block allows you to store some text to a variable.  See
[template].  The variable will be defined as an object of the
`DATemplate` class.

Objects of this type have two attributes:

* `content`
* `subject`

When **docassemble** defines a [template], it assembles any [Mako] in
the `content` and option `subject` sets defines these attributes as
the resulting text.  Note that the text may have [Markdown]<span></span> [markup]
in it.

# <a name="person classes"></a>Classes for information about people and things

## <a name="Thing"></a>Thing

If `pet_rock` is a `Thing`, it will be an object with one attribute:

* `pet_rock.name` (object of class [`Name`])

If you include `${ pet_rock }` in text, the name of the `Thing` will
be inserted.  **docassemble** will look for a definition of
`pet_rock.name.text`.

## <a name="Person"></a>Person

The `Person` class encompasses [`Individual`]s as well as legal persons,
like companies, government agencies, etc.  If you create an object of
type `Person` by doing:

{% highlight yaml %}
---
objects:
  - opponent: Person
---
{% endhighlight %}

then you will create an object with the following built-in attributes:

* `opponent.name` (object of class [`Name`])
* `opponent.address` (object of class [`Address`])
* `opponent.location` (object of class [`LatitudeLongitude`])

Referring to a `Person` in the context of a [Mako] template will
return the output of `.name.full()`.

The following attributes are also used, but undefined by default:

* `email` (see [`.email_address()`] and [`send_email()`])
* `phone_number` (see [`.sms_number()`] and [`send_sms()`] )
* `mobile_number` (see [`.sms_number()`] and [`send_sms()`])

The following methods can be used:

### <a name="Person.possessive"></a>`.possessive()`

Calling `defendant.possessive('fish')` returns "ABC Corporation's fish" or
"your fish" depending on whether `defendant` is the user.
  
### <a name="Person.identified"></a>`.identified()`

Calling `defendant.identified()` returns `True` if `defendant.name.text`
has been defined.  The
[version for `Individual`s](#Individual.identified)
is different.

### <a name="Person.pronoun_objective"></a>`.pronoun_objective()`

Calling `defendant.pronoun_objective()` returns "it," while calling
`defendant.pronoun_objective(capitalize=True)` returns "It."

### <a name="Person.object_possessive"></a>`.object_possessive()`

Calling `defendant.object_possessive('fish')` returns "defendant's
fish."

### <a name="Person.is_are_you"></a>`.is_are_you()`

Calling `defendant.is_are_you()` returns "are you" if `defendant` is
the user, and otherwise returns "is defendant."  Calling
`defendant.is_are_you(capitalize=True)` returns "Are you" or "Is
defendant."

### <a name="Person.is_user"></a>`.is_user()`

Calling `defendant.is_user()` returns `True` if the `defendant` is the
user, and otherwise returns `False`.

### <a name="Person.address_block"></a>`.address_block()`

Calling `defendant.address_block()` will return the name followed by
the address, in a format suitable for inclusion in a document.  For example:

{% highlight text %}
[FLUSHLEFT] ABC Corporation [NEWLINE] 1500 Market Street [NEWLINE] Philadelphia, PA 19102
{% endhighlight %}

See [markup] for more information about how this will appear in documents.

### <a name="Person.do_question"></a>`.do_question()`

Calling `defendant.do_question('testify')` returns "do you testify" if
the defendant is the user, or otherwise it uses the defendant's name,
as in "does ABC Corporation testify."

### <a name="Person.did_question"></a>`.did_question()`

Calling `defendant.did_question('testify')` returns "did you testify" if
the defendant is the user, or otherwise it uses the defendant's name,
as in "did ABC Corporation testify."

### <a name="Person.were_question"></a>`.were_question()`

Calling `defendant.were_question('guilty')` returns "were you guilty" if
the defendant is the user, or otherwise it uses the defendant's name,
as in "was ABC Corporation guilty."

### <a name="Person.have_question"></a>`.have_question()`

Calling `defendant.have_question('lied')` returns "have you lied" if
the defendant is the user, or otherwise it uses the defendant's name,
as in "has ABC Corporation lied."

### <a name="Person.does_verb"></a>`.does_verb()`

Calling `defendant.does_verb('testify')` returns "testify" if the
defendant is the user, but otherwise returns "testifies."  The method
accepts the optional keyword arguments `present` and `past`, which are
expected to be set to `True` or `False`.  For example,
`defendant.does_verb('testify', past=True)` will return "testified."

### <a name="Person.did_verb"></a>`.did_verb()`

The `.did_verb()` method is like the
[`.does_verb()`](#Person.does_verb) method, except that it conjugates
the verb into the past tense.

### <a name="Person.subject"></a>`.subject()`

The `.subject()` method returns "you" if the person is the user, but
otherwise returns the person's name.

### <a name="Person.email_address"></a>`.email_address()`

Calling `defendant.email_address()` will return the person's name
followed by the person's e-mail address, in the standard e-mail
format.  E.g., `'ABC Corporation <info@abc.com>'`.  If the name is not
yet defined, the e-mail address by itself (`info@abc.com`) will be
returned.

If you want to force **docassemble** to ask for the recipient's name,
set the optional keyword parameter `include_name` to `True`.

You can suppress the inclusion of the person's name by setting
`include_name` to `False`.

### <a name="Person.sms_number"></a>`.sms_number()`

Calling `defendant.sms_number()` will return `defendant.mobile_number`
if the `.mobile_number` attribute exists; it will not cause
the question to be asked.  If the `.mobile_number` attribute does not
exist, it will use `defendant.phone_number`.

The method formats the phone number in [E.164] format.  It will make
use of `defendant.country` to format the phone number, since the
[E.164] format contains the international country code of the phone
number.  If the `.country` attribute is not defined, the method will
call [`get_country()`].  The `.country` attribute is expected to be a
two-letter, capitalized abbreviation of a country.  It is a good idea
to set the attribute using 

### <a name="Organization"></a>Organization

An `Organization` is a subclass of [`Person`].  It has the attribute
`.office`, which is an object of type [`OfficeList`].

It uses the following attributes, which by default are not defined:

* `handles`: refers to a list of problems the organization handles.
* `serves`: refers to a list of counties the organization serves.

<a name="Organization.will_handle"></a>The `.will_handle()` method
returns `True` or `False` depending on whether the organization serves
a given county or handles a given problem.  It takes two optional
keyword arguments: `problem` and `county`.  For example, you could
call `agency.will_handle(problem='Divorce', county='Hampshire County')`.

### <a name="Individual"></a>Individual

The `Individual` is a subclass of [`Person`].  This class should be used
for persons who you know are human beings.

If you create an object of type `Individual` by doing:

{% highlight yaml %}
---
objects:
  - president: Individual
---
{% endhighlight %}

then you will create an object with the following built-in attributes:

* `president.name` (object of class [`IndividualName`])
* `president.child` (object of class [`ChildList`])
* `president.income` (object of class [`Income`])
* `president.asset` (object of class [`Asset`])
* `president.expense` (object of class [`Expense`])

In addition, the following attributes will be defined by virtue of an
`Individual` being a kind of [`Person`]:

* `president.address` (object of class [`Address`])
* `president.location` (object of class [`LatitudeLongitude`])

The following attributes are also used, but undefined by default:

* `birthdate`
* `gender`

A number of useful methods can be applied to objects of class
`Individual`.  Many of them will respond differently depending on
whether the `Individual` is the user or not.  If you use these
methods, be sure to inform **docassemble** who the user is by
inserting the following [initial block]:

{% highlight yaml %}
---
initial: True
code: |
  set_info(user=user, role='user_role')
---
{% endhighlight %}

(If you include the [`basic-questions.yml`] file, this is done for you.)

### <a name="Individual.identified"></a>`.identified()`

Returns `True` if the individual's name has been defined yet,
otherwise it returns `False`.

### <a name="Individual.age_in_years"></a>`.age_in_years()`

`user.age_in_years()` the `user`'s age in years as a whole number.

There are two optional arguments that modify the method's behavior:

* `user.age_in_years(decimals=True)` returns the user's age in years
  with the fractional part included.
* `user.age_in_years(as_of="5/1/2015")` returns the user's age as of a
  particular date.

### <a name="Individual.first_name_hint"></a><a name="Individual.last_name_hint"></a>`.first_name_hint()` and `.last_name_hint()`

When you are writing questions in an interview, you may find yourself
in this position:

* You are asking for the name of a person;
* That person whose name you need may be the user;
* The user may be logged in;
* The user, if logged in, may have already provided his or her name on
the user profile page; and
* It would be repetitive for the user to retype his or her
name.

In this situation, it would be convenient for the user if the user's
name was auto-filled on the page.  The `.first_name_hint()` and
`.last_name_hint()` methods accomplish this for you.  You can ask for
an individual's name as follows:

{% highlight yaml %}
---
generic object: Individual
question: |
  What is ${ x.object_possessive('name') }?
fields:
  - First Name: x.name.first
    default: ${ x.first_name_hint() }
  - Middle Name: x.name.middle
    required: False
  - Last Name: x.name.last
    default: ${ x.last_name_hint() }
  - Suffix: x.name.suffix
    required: False
    code: |
      name_suffix()
---
{% endhighlight %}

For an explanation of how [`.object_possessive()`] works, see the
[`Person`] class.

### <a name="Individual.possessive"></a>`.possessive()`

If the individual's name is "Adam Smith," this returns "Adam Smith's."
 But if the individual is the current user, this returns "your."

### <a name="Individual.salutation"></a>`.salutation()`

Depending on the `gender` attribute, the `.salutation()` method
returns "Mr." or "Ms."

{% include side-by-side.html demo="salutation" %}

### <a name="Individual.pronoun_possessive"></a>`.pronoun_possessive()`

If the individual is `client`, then
`client.pronoun_possessive('fish')` returns "your fish," "his fish,"
or "her fish," depending on whether `client` is the user and depending
on the value of `client.gender`.  `client.pronoun_possessive('fish',
capitalize=True)` returns "Your fish," "His fish," or "Her fish."

If you want to refer to the individual in the third person even if the
individual is the user, write `client.pronoun_possessive('fish',
third_person=True)`.

For portability to different languages, this method requires you to
provide the noun you are modifying.  In some languages, the possessive
pronoun may be different depending on what the noun is.

### <a name="Individual.pronoun"></a>`.pronoun()`

Returns "you," "him," or "her," depending on whether the individual is
the user and depending on the value of the `gender` attribute.  If
called with `capitalize=True`, the word will be capitalized (for use
at the beginning of a sentence).

### <a name="Individual.pronoun_objective"></a>`.pronoun_objective()`

For the `Individual` class, `pronoun_objective()` does the same thing
as `pronoun`.  (Other classes returns "it.")  If called with
`capitalize=True`, the output will be capitalized.

### <a name="Individual.pronoun_subjective"></a>`.pronoun_subjective()`

Returns "you," "he," or "she," depending on whether the individual is
the user and depending on the value of the `gender` attribute.

You can call this method with the following optional keyword arguments:

* `third_person=True`: will use "he" or "she" even if the individual
is the user.
* `capitalize=True`: the output will be capitalized (for use at the
  beginning of a sentence)

### <a name="Individual.yourself_or_name"></a>`.yourself_or_name()`

Returns "yourself" if the individual is the user, but otherwise
returns the person's name.  If called with the optional keyword
argument `capitalize=True`, the output will be capitalized.

## <a name="Name"></a>Name

The `Name` is the base class for names of things, such as [`Person`].
For example, if `plaintiff` is a [`Person`], `plaintiff.name` is an
object of type `Name`.  If `plaintiff` is an [`Individual`],
`plaintiff.name` is an object of type [`IndividualName`], which is a
subtype of `Name`.  (The [`IndividualName`] is defined in the next
section.)

Objects of the basic [`Name`] class have just one attribute, `text`.
To set the name of a [`Person`] called `company`, for example, you can
do something like this:

{% include side-by-side.html demo="name-company-question" %}

There are multiple ways to refer to the name of an object, but the
best way is to write something like this:

{% include side-by-side.html demo="name-company" %}

Multiple ways of referring to the name of a [`Person`] are illustrated
in the following interview:

{% include side-by-side.html demo="name" %}

Note that `${ opponent.name.full() }`, `${ opponent.name }`, and `${
opponent }` all return the same thing.  This is because a [`Person`]
in the context of a [Mako] template returns `.name.full()`, and a
[`Name`] returns `.full()`.

The reason a name is not just a piece of text, but rather an object
with attributes like `text` and methods like `.full()`, is that some
objects have names with multiple parts that you will want to express
in multiple ways.  You might have a list of parties in a case, where
the parties can be companies or individuals.  It helps to have a
common way of referring to the names of these objects.

<a name="Name.full"></a>
<a name="Name.firstlast"></a>
<a name="Name.lastfirst"></a>
The [`Name`] and [`IndividualName`] objects support the following methods:

* `.full()`
* `.firstlast()`
* `.lastfirst()`

Applied to an [`IndividualName`] object, these methods return different
useful expressions of the name.  Applied to a [`Name`] object, these
methods all return the same thing -- the `.text` attribute.  This is
useful because you can write things like this, which lists the names
of the parties in a bullet-point list:

{% include side-by-side.html demo="lastfirst" %}

In this template, the author does not need to worry about which
parties are companies and which parties are individuals; the name will
be listed in the bullet-point list in an appropriate way.  For
individuals, the last name will come first, but for non-individuals,
the regular name will be printed.

<a name="Name.defined"></a>
The [`Name`] and [`IndividualName`] objects also support the method:

* `.defined()`

This returns `True` if the necessary component of the name (`.text`
for a [`Name`], `first` for an [`IndividualName`]) has been defined yet.
Otherwise it returns `False`.

### <a name="IndividualName"></a>IndividualName

The [`Individual`] class is a subclass of [`Person`].  It defines the
`name` attribute as an `IndividualName` rather than a [`Name`].  An
`IndividualName` uses the following attributes, which are expected to
be text:

* `first`
* `middle`
* `last`
* `suffix`

In the context of a [Mako] template, a reference to an `IndividualName` on
its own will return `.full()`.

The `full()` method attempts to form a full name from these
components.  Only `first` is required, however.  This means that if
you refer to an `IndividualName` in a [Mako] template, e.g., by
writing `${ applicant.name }`, **docassemble** will attempt to return
`applicant.name.full()`, and if `applicant.name.first` has not been
defined yet, **docassemble** will look for a question that defines
`applicant.name.first`.

Here is how `full()` and other methods of the `IndividualName` work:

* <a name="IndividualName.full"></a>`applicant.full()`: "John Q. Adams"
* `applicant.full(middle="full")`: "John Quincy Adams"
* <a name="IndividualName.firstlast"></a>`applicant.firstlast()`: "John Adams"
* <a name="IndividualName.lastfirst"></a>`applicant.lastfirst()`: "Adams, John"
* <a name="IndividualName.defined"></a>`applicant.defined()`: Returns
  `True` if the `.first` attribute has been defined yet. 

The `IndividualName` also uses the attribute `.uses_parts`, which is
`True` by default.  If `.uses_parts` is set to `False`, then the
methods of the object fall back on the methods of [`Name`], and the
individual's name is stored in the attribute `.text`.

## <a name="Address"></a>Address

An `Address` has the following text attributes:

* `address`: e.g., "123 Main Street"
* `unit`: e.g., "Suite 100"
* `city`: e.g., "Springfield"
* `state`: e.g., "MA"
* `zip`: e.g. "01199"
* `country`: e.g., 'US'

It also has an attribute `location`, which is a [`LatitudeLongitude`]
object representing the GPS coordinates of the address.

You can use the function 

If you refer to an address in a [Mako] template, it returns `.block()`.

<a name="Address.block"></a>
The `.block()` method returns a formatted address.  All attributes
except `unit` are required.

<a name="Address.geolocate"></a>
The `.geolocate()` method determines the latitude and longitude of the
address and stores it in the attribute `location`, which is a
[`LatitudeLongitude`] object.

<a name="Address.line_one"></a>
The `.line_one()` method returns the first line of the address,
including the unit, if the unit is defined.

<a name="Address.line_two"></a>
The `.line_two()` method returns the second line of the address,
consisting of the city, state, and zip code.

<a name="Address.on_one_line"></a>
The `.on_one_line()` method returns the address, consisting of
the city, state, and zip code, as a single line of text.

## <a name="LatitudeLongitude"></a>LatitudeLongitude

A `LatitudeLongitude` object represents the GPS coordinates of an
address or location.  `LatitudeLongitude` objects have the following
attributes:

* `latitude`: the latitude of the location.
* `longitude`: the longitude of the location.
* `description`: a textual description of the location.
* `known`: whether the GPS location is known yet.
* `gathered`: whether a determination of the GPS location has been
attempted yet.

One use for the `LatitudeLongitude` object is for mapping the
coordinates of an address.  The [`Address`] object has a method
`.geolocate()` for this purpose.

<a name="LatitudeLongitude.status"></a>
Another use for the `LatitudeLongitude` object is storing the GPS
location of the user's device.  Many web browsers, particularly those
on mobile devices, have a feature for determining the user's GPS
coordinates.  Usually the browser asks the user to consent to the
sharing of the location information.  To support this feature, the
`LatitudeLongitude` object offers the method `.status()`.

The following example shows how to gather the user's latitude and
longitude from the web browser.

{% highlight yaml %}
---
include:
  - basic-questions.yml
---
initial: True
code: |
  track_location = user.location.status()
---
{% endhighlight %}

Alternatively, if you do not want to include all of the questions and
code blocks of the [`basic-questions.yml`] file in your interview, you
can do:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
objects:
  - user: Individual
---
initial: True
code: |
  set_info(user=user, role='user_role')
  track_location = user.location.status()
---
{% endhighlight %}

If all goes well, the user's latitude and longitude will be gathered
and stored in `user.location.latitude` and `user.location.longitude`.
You can control when this happens in the interview by controlling when
`track_location` is set.  For example, you may wish to prepare the
user for this:

{% highlight yaml %}
---
initial: True
code: |
  set_info(user=user, role='user_role')
  if user_ok_with_sharing_location:
    track_location = user.location.status()
---
question: |
  We would like to gather information about your current location
  from your mobile device.  Is that ok with you?
yesno: user_ok_with_sharing_location
---
{% endhighlight %}

[`track_location`] is a [special variable] that tells **docassemble**
whether or not it should ask the browser for the user's GPS
coordinates the next time a question is posed to the user.  If
[`track_location`] is `True`, **docassemble** will ask the browser to
provide the information, and if it receives it, it will make it
available for retrieval through the [`user_lat_lon()`] function.

The `.status()` method looks to see if a latitude and longitude were
provided by the browser, or in the alternative that an error message
was provided, such as "the user refused to share the information," or
"this device cannot determine the user's location."  If the latitude
and longitude information is conveyed, `.status()` stores the
information in `user.location.latitude` and `user.location.longitude`.
The `.status()` method returns `False` in these situations, which
means "we already asked for the latitude and longitude and got a
response, so there is no longer any need for the browser to keep
asking for it."  Otherwise, it returns `True`, which means "the
browser has not yet been asked for the location information, so let's
ask it."

# Classes for currency

## <a name="Value"></a>Value

A `Value` is a subclass of [`DAObject`] that is intended to represent a
currency value that may or may not need to be asked for in an interview.

For example, suppose you want to have a variable that represents the
value of the user's real estate holdings.  But before you ask the
value of the user's real estate holdings, you will want to ask if the
user has real estate holdings at all.

A `Value` has two attributes, both of which are initially undefined:

* `.value`: intended to be a number
* `.exists`: a boolean value representing whether the value is applicable

The `.exists` attribute facilitates asking questions about values
using two screens: first, ask whether the value exists at all, then
ask for the value.  For example:

{% include side-by-side.html demo="exists" %}

The [`FinancialList`] object, explained below, represents a list of
`Value`s.  When computing a total of the values (with `.total()`), it
checks the `.exists` attributes of each `Value` to be defined.  This
causes questions to be asked about whether the `Value` is applicable
to the user's situation before the value itself is requested.

<a name="Value.amount"></a>To access the value of a `Value` object,
you can use the `.amount()` method.  If the `.exists` attribute is
`False`, it will return zero without asking for the `.value`.

Referring to a `Value` in a [Mako] template will show the `.amount()`.  The
value of `.amount()` is also returned when you pass a `Value` to the
[`currency()`] function.  For example:

{% highlight yaml %}
---
question: |
  The value of your real estate holdings is
  ${ currency(real_estate_holdings) }.
  
  An identical way of writing this number is 
  ${ currency(real_estate_holdings.amount()) }.
---
{% endhighlight %}

### <a name="PeriodicValue"></a>PeriodicValue

A `PeriodicValue` is a [`Value`] that has an additional attribute,
`period`, which is a number representing the number of times per year
the value applies.

{% include side-by-side.html demo="periodic-value" %}

<a name="PeriodicValue.amount"></a>To access the value of a
`PeriodicValue` object, you can use the `.amount()` method.  If the
`.exists` attribute is `False`, it will return zero without asking for
the `.value`.  By default, it returns the value for the period 1
(e.g., in the example above, period of 1 represents a year).  That is,
it will return the `.value` multiplied by the `.period`.

Referring to a `PeriodicValue` in a [Mako] template will show the
`.amount()`.  The value of `.amount()` is also returned when you pass
a `PeriodicValue` to the [`currency()`] function.

# Classes for lists of things

## <a name="PartyList"></a>PartyList

This is a subclass of [`DAList`].

It is indended to contain a list of [`Person`]s (or [`Individual`]s,
which are a type of [`Person`]) who are parties to a case.

## <a name="ChildList"></a>ChildList

This is a subclass of [`DAList`].

It is indended to contain a list of [`Individual`]s who are children.

## <a name="FinancialList"></a>FinancialList

This is a class intended to collect a set of financial items, such as
an individual's assets.

The `FinancialList` uses the following attributes:

* `gathering`: a boolean value that is initialized to `False`.  Set
this to `True` when your process of initializing the elements of the
list is ongoing and will span multiple questions.
* `gathered`: a boolean value that is initially undefined.  Set this
to `True` when you have finished determining what the elements of the
list are going to be.

The `FinancialList` has three methods:

* <a name="FinancialList.new"></a>`.new(item_name)`: gives the
  `FinancialList` a new attribute with the name `item_name` and the
  object type `Value`.
* <a name="FinancialList.total"></a>`.total()`: tallies up the total
  value of all `Value`s in the list for which the `exists` attribute
  is `True`.  It requires `.gathered` to be `True`, which means that a
  reference to `.total()` will cause **docassemble** to ask the
  questions necessary to gather the full list of items.
* <a name="FinancialList.existing_items"></a>`.existing_items()`:
  returns a list of types of amounts that exist within the financial
  list.

In the context of a [Mako] template, a `FinancialList` returns the result of
`.total()`.

Note that a `FinancialList` is a [`DAObject`] but not a [`DAList`].  It
tracks the items in the list using the attribute `elements`, which is
a [Python set].

### <a name="Asset"></a>Asset

This is a subclass of [`FinancialList`] that is intended to be used to
track assets.

Here is some example code that triggers questions that ask about asset
items.  Note that every [`Individual`] is initialized with an attribute
called `asset` that is an object of type `Asset`.

{% highlight yaml %}
---
mandatory: True
question: |
  Your total assets are ${ user.asset }.
---
generic object: Individual
code: |
  for asset_item in ['checking', 'savings', 'stocksbonds']:
    x.asset.new(asset_item)
  x.asset.gathered = True
---
generic object: Individual
question: |
  What kinds of assets ${ x.do_question("own") }?
fields:
  - Checking Account: x.asset.checking.exists
    datatype: yesnowide
  - Savings Account: x.asset.savings.exists
    datatype: yesnowide
  - Stocks and Bonds: x.asset.stocksbonds.exists
    datatype: yesnowide
---
generic object: Individual
question: |
  How much ${ x.do_question("have") } in 
  ${ x.pronoun_possessive("checking account") }?
fields:
  - Amount in Checking Account: x.asset.checking.value
    datatype: currency
---
{% endhighlight %}

(Additional questions asking about the value of asset items are
omitted.)

1. The inclusion of `user.asset` in a [Mako] template returns the value of
`user.asset.total()`.
2. The `.total()` method checks to see if `user.asset.gathered` is
`True`.  Since `user.asset.gathered` is initially undefined, this
triggers the code block that defines the elements of `user.asset`.
Note that we say the elements are "gathered" even though the
attributes of each element, `exists` and `value`, are still undefined.
3. The `.total()` method then goes through each element and checks to
see if the element `exists`.  This triggers the question that will
define `user.asset.checking.exists` and the other values.
4. If the `.total()` method finds that an element exists, it adds its
`value` to a subtotal.  This triggers the question that will
define `user.asset.checking.value`.
5. The `.total()` method does this for every element in `user.asset`
and finally returns a total.

Note that in this example, we did not have to worry about setting
`user.asset.gathering` because the process of populating the elements
of the asset list did not span multiple questions.

## <a name="PeriodicFinancialList"></a>PeriodicFinancialList

This is a class intended to collect a set of financial items that have
a periodic nature, such as an individual's income.

The `PeriodicFinancialList` uses the following attributes:

* `gathering`: a boolean value that is initialized to `False`.  Set
this to `True` when your process of initializing the elements of the
list is ongoing and will span multiple questions.
* `gathered`: a boolean value that is initially undefined.  Set this
to `True` when you have finished gathering all of the elements.

The `PeriodicFinancialList` has three methods:

* <a name="PeriodicFinancialList.new"></a>`.new(item_name)`: gives the
  `PeriodicFinancialList` a new attribute with the name `item_name`
  and the object type `PeriodicValue`.
* <a name="PeriodicFinancialList.total"></a>`.total()`: tallies up the total annual value of all `PeriodicValue`s in the list
  for which the `exists` attribute is `True`.

In the context of a [Mako] template, a `PeriodicFinancialList` returns `.total()`.

### <a name="Income"></a>Income

This is a subclass of [`PeriodicFinancialList`].

Here is some example code that triggers questions that ask about
income items.  Note that ever [`Individual`] has an attribute `income`
that is an object of type `Income`.

{% highlight yaml %}
---
mandatory: True
question: |
  Your total annual income is ${ user.income }.
---
generic object: Individual
code: |
  for income_item in ['employment', 'selfemployment']:
    x.income.new(income_item, period=12)
  x.income.gathered = True
---
generic object: Individual
question: |
  What kinds of income ${ x.do_question("have") }?
fields:
  - Employment: x.income.employment.exists
    datatype: yesnowide
  - Self-employment: x.income.selfemployment.exists
    datatype: yesnowide
---
generic object: Individual
question: |
  How much ${ x.do_question("make") } from employment?
fields:
  - Employment Income: x.income.employment.value
    datatype: currency
  - "": x.income.employment.period
    datatype: number
    code: |
      period_list()
---
{% endhighlight %}

(Not all necessary questions are shown.)

### <a name="Expense"></a>Expense

`Expense` is a [`PeriodicFinancialList`] representing a person's expenses.

## <a name="OfficeList"></a>OfficeList

An `OfficeList` object is a type of [`DAList`], the elements of which are expected to be
[`Address`] objects.  It is used in [`Organization`] objects.

# Classes for special purposes

## <a name="RoleChangeTracker"></a>RoleChangeTracker

The `RoleChangeTracker` class is provided to facilitate [multi-user
interviews] with **docassemble**'s [roles] system.  It facilitates
sending e-mails to the participants to let them know when the
interview needs their attention.  It keeps track of when these e-mails
have been sent to make sure that duplicative e-mails are not sent.

It has one method:

* <a name="RoleChangeTracker.send_email"></a>`role_change.send_email()`
  (not to be confused with the `send_email()` function)

Here is an example that demonstrates its use:

{% highlight yaml %}
---
modules:
  - docassemble.base.util
---
objects:
  - client: Individual
  - advocate: Individual
  - role_change: RoleChangeTracker
---
default role: client
code: |
  if user_logged_in() and \
      advocate.attribute_defined('email') and \
      advocate.email == user_info().email:
    user = advocate
    role = 'advocate'
  else:
    user = client
    role = 'client'
  set_info(user=user, role=role)
---
event: role_event
question: You are done for now.
subquestion: |
  % if 'advocate' in role_needed:
  An advocate needs to review your answers before you can proceed.

  Please remember the following link and come back to it when you
  receive notice to do so:

  [${ interview_url() }](${ interview_url() })
  % else:
  Thanks, the client needs to resume the interview now.
  % endif

  % if role_change.send_email(role_needed, advocate={'to': advocate, 'email': role_event_email_to_advocate}, client={'to': client, 'email': role_event_email_to_client}):
  An e-mail has been sent.
  % endif
decoration: exit
buttons:
  - Exit: leave
---
template: role_event_email_to_advocate
subject: |
  Client interview waiting for your attention: ${ client }
content: |
  A client, ${ client }, has partly finished an interview.

  Please go to [the interview](${ interview_url() }) as soon as
  possible to review the client's answers.
---
template: role_event_email_to_client
subject: |
  Your interview answers have been reviewed
content: |
  An advocate has finished reviewing your answers.

  Please go to [${ interview_url() }](${ interview_url() })
  to resume the interview.
---
{% endhighlight %}

The `send_email()` method's first argument is the special variable
`role_needed`, a [Python list] that **docassemble** defines internally
whenever there is a mismatch between the current user's role and the
role required by a question that needs to be asked.

The remaining arguments to `send_email()` are [keyword arguments],
where each keyword is the name of a possible role.  Each
[keyword argument] must be a [Python dictionary] containing the
following keys:

* `to`: this needs to be a [`Person`] (or a subclass, like
[`Individual`]).  The person's `email` attribute is expected to be
defined.
* `email`: this needs to a [`DATemplate`] containing the subject and
body of the e-mail that will be sent.

# <a name="instanceName"></a>How docassemble objects are different

For most purposes, **docassemble** objects behave just like [Python]
objects.  However, they have special properties that facilitate the
automatic asking of questions.  You may need to be mindful of these
special properties if you do anything fancy in your code.

In contrast to [Python objects] in general, **docassemble** objects
are aware of their first-given names.  All **docassemble** objects
have an `.instanceName` attribute.  So if you do:

{% highlight yaml %}
---
objects:
  - user: Individual
---
{% endhighlight %}

then `user.instanceName` will be `'user'`, and
`user.name.instanceName` will be `'user.name'`.

You can also initialize `user` in standard [Python] fashion:

{% highlight yaml %}
---
code: |
  user = Individual()
---
{% endhighlight %}

In this circumstance, **docassemble** uses some [magic] to set
`.instanceName` to `user`.  However, the magic has its limits.  For
example, the following does not work:

{% highlight yaml %}
---
code: |
  (user, advocate) = (Individual(), Individual())
---
{% endhighlight %}

If you ever get an error message in **docassemble** referring to
variables with a name like `qjAMyvGQYnyK`, and you are sure you did
not create such a variable, then you have an object that was unable to
determine its given name.

If you want to initialize objects using expressions more complicated
than `variable_name = ObjectName()`, you can -- you just need to
include the variable name as an argument to the object name.  For
example:

{% highlight yaml %}
---
code: |
  (user, advocate) = (Individual('user'), Individual('advocate'))
---
{% endhighlight %}

Attribute initialization does not have this limitation.

Always keep in mind that objects are given `.instanceName` attributes
as early as possible, and once an `.instanceName` is assigned, it will
not be overwritten unless you explicitly overwrite it.  For example,
if you do:

{% highlight yaml %}
---
code: |
  user.name = IndividualName()
---
{% endhighlight %}

then `user.name.instanceName` will return `'user.name'`, as you would
expect.  But if you do:

{% highlight yaml %}
---
code: |
  cool_name = IndividualName()
  cool_name.first = 'Groovy'
  cool_name.last = 'Jones'
---
code: |
  user.name = cool_name()
---
{% endhighlight %}

then `user.name.instanceName` will be `'cool_name'`, not `'user.name'`.

You can manually correct this:

{% highlight yaml %}
---
code: |
  user.name = cool_name()
  user.name.instanceName = 'user.name'
---
{% endhighlight %}

The `.instanceName` is not simply an internal attribute; it is used by
the [`.object_possessive()`] method to refer to the object in
human-readable format.

# Extending existing classes

If you want to add a method to an existing **docassemble** class, such
as [`Individual`], you do not need to reinvent the wheel or copy and
paste code from anywhere.  Just take advantage of [inheritance].

For example, if your package is `docassemble.missouri_family_law`, you
could create/edit the file
`docassemble/missouri_family_law/objects.py` and add the following:

{% highlight python %}
from docassemble.base.util import Individual

class Attorney(Individual):
    def can_practice_in(self, state):
        if state in self.bar_admissions and self.bar_admissions[state] is True:
            return True
        return False
{% endhighlight %}

Here you are defining the class `Attorney` as a subclass of
[`Individual`].  An object that is an instance of the `Attorney` class
will also be an instance of the [`Individual`] class.  The `Attorney`
class is said to "inherit" from the [`Individual`] class.  All of the
methods that can be used on an [`Individual`] can be used on an
`Attorney`.

This allows you to write an interview like the following:

{% include side-by-side.html demo="attorney" %}

Note that the `lawyer` object works just like an [`Individual`] object.
The `is_are_you()` method, which is defined in
[`docassemble.base.util`], works on the `Attorney` object, despite the
fact that the interview does not explicitly refer to
[`docassemble.base.util`] anywhere.  (The module
`docassemble.missouri_family_law.objects` imports
[`docassemble.base.util`].)

Note that the `can_practice_in()` method is only available for
`Attorney` objects.  If you added the following to the above
interview:

{% highlight yaml %}
---
objects:
  - user: Individual
---
mandatory: True
question: |
  % if user.can_practice_in('MA'):
  You can take this case yourself.
  % else:
  You will need to hire a lawyer to take the case.
  % endif
---
{% endhighlight %}

then you would get an error because `can_practice_in()` is not a valid
method for `user`, which is only an instance of the [`Individual`] class
and not an instance of the `Attorney` class.

[AttributeError]: https://docs.python.org/2/library/exceptions.html#exceptions.AttributeError
[Documents]: {{ site.baseurl }}/docs/documents.html
[Flask-Mail]: https://pythonhosted.org/Flask-Mail/
[HTML]: https://en.wikipedia.org/wiki/HTML
[Mako]: http://www.makotemplates.org/
[Markdown]: https://daringfireball.net/projects/markdown/
[Python dict]: https://docs.python.org/2/tutorial/datastructures.html#dictionaries
[Python dictionary]: https://docs.python.org/2/tutorial/datastructures.html#dictionaries
[Python list]: https://docs.python.org/2/tutorial/datastructures.html
[Python module]: https://docs.python.org/2/tutorial/modules.html
[Python object]: https://docs.python.org/2/tutorial/classes.html
[Python objects]: https://docs.python.org/2/tutorial/classes.html
[Python set]: https://docs.python.org/2/library/stdtypes.html#set
[Python]: https://www.python.org/
[YAML]: https://en.wikipedia.org/wiki/YAML
[`Address`]: #Address
[`Asset`]: #Asset
[`ChildList`]: #ChildList
[`DAFileCollection`]: DAFileCollection
[`DAFileList`]: #DAFileList
[`DAFile`]: #DAFile
[`DAEmail`]: #DAEmail
[`DAEmailRecipient`]: #DAEmailRecipient
[`DAEmailRecipientList`]: #DAEmailRecipientList
[`DADict`]: #DAFile
[`DAList`]: #DAList
[`DASet`]: #DAList
[`DAObject`]: #DAObject
[`DATemplate`]: #DATemplate
[`Expense`]: #Expense
[`FinancialList`]: #FinancialList
[`Income`]: #Income
[`IndividualName`]: #IndividualName
[`Individual`]: #Individual
[`LatitudeLongitude`]: #LatitudeLongitude
[`Name`]: #Name
[`Organization`]: #Organization
[`PeriodicFinancialList`]: #PeriodicFinancialList
[`Person`]: #Person
[`Value`]: #Value
[`age_in_years()`]: #Individual.age_in_years
[`attachment`]: {{ site.baseurl }}/docs/documents.html#attachments
[`attachments`]: {{ site.baseurl }}/docs/documents.html#attachments
[`basic-questions.yml`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/data/questions/basic-questions.yml
[`code`]: {{ site.baseurl }}/docs/code.html
[`currency()`]: {{ site.baseurl }}/docs/functions.html#currency
[`default role`]: {{ site.baseurl }}/docs/initial.html#default role
[`docassemble.base.legal`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/legal.py
[`docassemble.base.util`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/util.py
[`docassemble.base`]: {{ site.baseurl }}/docs/installation.html#docassemble.base
[`generic object`]: {{ site.baseurl }}/docs/modifiers.html#generic object
[`get_info()`]: {{ site.baseurl }}/docs/functions.html#get_info
[`initial`]: {{ site.baseurl }}/docs/logic.html#initial
[`interview_url()`]: {{ site.baseurl }}/docs/functions.html#interview_url
[`legal.py`]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/legal.py
[`mandatory`]: {{ site.baseurl }}/docs/logic.html#mandatory
[`modules`]: {{ site.baseurl }}/docs/initial.html#modules
[`multi_user`]: {{ site.baseurl }}/docs/special.html#multi_user
[`.object_possessive()`]: #DAObject.object_possessive
[`objects`]: {{ site.baseurl }}/docs/initial.html#objects
[`possessive()`]: #Individual.possessive
[`process_action()`]: {{ site.baseurl }}/docs/functions.html#process_action
[`question`]: {{ site.baseurl }}/docs/questions.html#question
[`set_info()`]: {{ site.baseurl }}/docs/functions.html#set_info
[`template`]: {{ site.baseurl }}/docs/template.html
[`track_location`]:  {{ site.baseurl }}/docs/special.html#track_location
[`word()`]: {{ site.baseurl }}/docs/functions.html#word
[classes]: https://docs.python.org/2/tutorial/classes.html
[configuration]: {{ site.baseurl }}/docs/config.html
[fields]: {{ site.baseurl }}/docs/fields.html
[Python function]: https://docs.python.org/2/tutorial/controlflow.html#defining-functions
[inheritance]: https://docs.python.org/2/tutorial/classes.html#inheritance
[initial block]: {{ site.baseurl }}/docs/initial.html
[initial blocks]: {{ site.baseurl }}/docs/initial.html
[keyword argument]: https://docs.python.org/2/glossary.html#term-argument
[keyword arguments]: https://docs.python.org/2/glossary.html#term-argument
[list]: https://docs.python.org/2/tutorial/datastructures.html
[markup]: {{ site.baseurl }}/docs/markup.html
[methods]: https://docs.python.org/2/tutorial/classes.html
[modifier]: {{ site.baseurl }}/docs/modifiers.html
[modifiers]: {{ site.baseurl }}/docs/modifiers.html
[multi-user interviews]: {{ site.baseurl }}/docs/roles.html
[object-oriented programming]: https://en.wikipedia.org/wiki/Object-oriented_programming
[package system]: {{ site.baseurl }}/docs/packages.html
[package]: {{ site.baseurl }}/docs/packages.html
[roles]: {{ site.baseurl }}/docs/roles.html
[source code]: {{ site.github.repository_url }}/blob/master/docassemble_base/docassemble/base/legal.py
[special variable]: {{ site.baseurl }}/docs/special.html
[str function]: https://docs.python.org/2/library/functions.html#str
[template]: {{ site.baseurl }}/docs/template.html
[thread-safe]: https://en.wikipedia.org/wiki/Thread_safety
[threading module]: https://docs.python.org/2/library/threading.html
[user login system]: {{ site.baseurl }}/docs/users.html
[`nice_number()`]: {{ site.baseurl }}/docs/functions.html#nice_number
[`comma_and_list()`]: {{ site.baseurl }}/docs/functions.html#comma_and_list
[`user_lat_lon()`]: {{ site.baseurl }}/docs/functions.html#user_lat_lon
[`sets`]: {{ site.baseurl }}/docs/fields.html#sets
[`send_email()`]: {{ site.baseurl }}/docs/functions.html#send_email
[`send_sms()`]: {{ site.baseurl }}/docs/functions.html#send_sms
[E.164]: https://support.twilio.com/hc/en-us/articles/223183008-Formatting-International-Phone-Numbers
[`get_country()`]: {{ site.baseurl }}/docs/functions.html#get_country
[`.sms_number()`]: #Person.sms_number
[`.email_address()`]: #Person.email_address
[`OfficeList`]: #OfficeList
[groups]: {{ site.baseurl }}/docs/groups.html
[question]: {{ site.baseurl }}/docs/questions.html#question
[code block]: {{ site.baseurl }}/docs/code.html#code
[dict]: https://docs.python.org/2/library/stdtypes.html#dict
[set]: https://docs.python.org/2/library/stdtypes.html#set
[magic]: https://docs.python.org/2/library/inspect.html
[Amazon S3]: https://aws.amazon.com/s3/
[Python Imaging Library]: https://en.wikipedia.org/wiki/Python_Imaging_Library
[e-mail receiving]: {{ site.baseurl }}/docs/background.html#email
[`interview_email()`]: {{ site.baseurl }}/docs/functions.html#interview_email
[`datetime`]: https://docs.python.org/2/library/datetime.html#datetime.datetime
[Return-path]: https://en.wikipedia.org/wiki/Bounce_address
[Reply-to]: https://www.ietf.org/rfc/rfc2822.txt
[JSON]: https://en.wikipedia.org/wiki/JSON
[MIME type]: https://en.wikipedia.org/wiki/Media_type
[Azure blob storage]: https://azure.microsoft.com/en-us/services/storage/blobs/
[document upload field]: {{ site.baseurl }}/docs/fields.html#interview_email
[`variable name`]: {{ site.baseurl }}/docs/documents.html#variable name
[inserting images]: {{ site.baseurl }}/docs/markup.html#inserting images
[data storage]: {{ site.baseurl }}/docs/docker.html#data storage