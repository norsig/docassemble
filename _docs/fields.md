---
layout: docs
title: Setting variables (and doing other things) with questions
short_title: Setting Variables
---

To instruct **docassemble** to store user input in a variable in
response to a [question], you need to include in your `question` a
variable name within a directive that indicates how you would like
**docassemble** to ask for the value of the variable.

# Directives

## <a name="yesno"></a><a name="noyes"></a>`yesno` or `noyes`

{% highlight yaml %}
---
question: Were you injured?
yesno: user_has_injury
---
{% endhighlight %}

The `yesno` statement causes a question to set a boolean (true/false)
variable when answered.

In the example above, the web app will present "Yes" and "No" buttons
and will set `user_has_injury` to `True` if "Yes" is pressed, and
`False` if "No" is pressed.

The `noyes` statement is just like `yesno`, except that "Yes" means
`False` and "No" means `True`.

{% highlight yaml %}
---
question: Were you not injured?
noyes: user_has_injury
---
{% endhighlight %}

## <a name="field with buttons"></a>`field` with `buttons`

{% highlight yaml %}
---
question: Do you understand what I have explained to you?
field: user_understands
buttons:
  - I understand: understands
  - I do not understand: does not understand
  - "I'm not sure": I'm not sure
---
{% endhighlight %}

(Note that in the example above, quotation marks are used because the
apostrophe in "I'm not sure" would otherwise confuse the [YAML]
parser.)

A `question` block with a `buttons` statement will set the variable
identified in `field` to a particular value depending on which of the
buttons the user presses.

The `buttons` statement must always refer to a [YAML] list, so that
**docassemble** knows the order of the buttons.

If an item under `buttons` is a [YAML] key-value pair (written in the
form of `- key: value`), then the key will be the button label that the
user sees, and the value will be what the variable identified in `field`
will be set to if the user presses that button.

An item under `buttons` can also be plain text; in that case
**docassemble** uses this text for both the label and the variable
value.  For example, this:

{% highlight yaml %}
---
field: user_gender
question: What is your gender?
buttons:
  - Male
  - Female
---
{% endhighlight %}

is equivalent to:

{% highlight yaml %}
---
field: user_gender
question: What is your gender?
buttons:
  - Male: Male
  - Female: Female
---
{% endhighlight %}

A powerful feature of `buttons` is the ability to use Python code to
generate button choices.  If an item under `buttons` is a key-value
pair in which the key is the word `code`, then **docassemble**
executes the value as Python code, which it expects to return a list.
This code is executed at the time the question is asked, and the code
can include variables from the interview.  **docassemble** will
process the resulting list and create additional buttons for each
item.  To illustrate this, the first example above could alternatively
have been written as:

{% highlight yaml %}
---
field: user_understands_no_attorney_client_relationship
question: |
  Your use of this system does not mean that you have a lawyer.  Do
  you understand this?
buttons:
  - "I understand": understands
  - code: |
      [{'does not understand':"I do not understand"}, {'unsure':"I'm not sure"}]
---
{% endhighlight %}

Note that the Python code needs to return key-value pairs (Python
dictionaries) where the key is what the variable should be set to and
the value is the button label.  This is different from the [YAML]
syntax.

## <a name="field with choices"></a>`field` with `choices`

{% highlight yaml %}
---
question: |
  What is your favorite color?
field: favorite_color
choices:
  - Red: red
  - Blue: blue
  - Green: green
---
{% endhighlight %}

This provides "radio buttons" with a "Continue" button.

## <a name="image button"></a>Adding images to `buttons` or `choices`

To add a decorative icon to a choice, use a key/value pair and add
`image` as an additional key.

{% highlight yaml %}
---
field: gender
question: |
  Are you a man or a woman?
buttons:
  - Man: masculine
    image: male
  - Woman: feminine
    image: female
---
{% endhighlight %}

This works with both `buttons` and `choices`.

## <a name="code button"></a>buttons/choices that embed `question` and `code` blocks

Multiple choice questions can embed `question` blocks and `code`
blocks.  These questions are just like ordinary questions, except they
can only be asked by way of the questions in which they are embedded.

You embed a question by providing a [YAML] key-value list (a
dictionary) (as opposed to text) as the value of a label in a
`buttons` or `choices` list.

{% highlight yaml %}
---
question: What is your favorite color?
buttons:
  - Red:
      question: Dark red or light red?
      field: favorite_color
      buttons:
        - Dark Red
        - Light Red
  - Green:
      question: Dark green or light green?
      field: favorite_color
      buttons:
        - Dark Green
        - Light Green
---
{% endhighlight %}

While embedding `question` blocks can be useful sometimes, it is
generally not a good idea to structure interviews with a lot of
embedded questions.  You will have more flexibility if your questions
stand on their own.

It is also possible for multiple-choice questions to embed `code`
blocks that execute Python code.  (If you do not know what `code`
blocks are yet, read the section on [code blocks] first.)  This can be
useful when you want to set the values of multiple variables with one
button.

{% highlight yaml %}
---
question: What kind of car do you want?
buttons:
  - Ford Focus:
      code: |
        car_model = "Focus"
        car_make = "Ford"
  - Toyota Camry:
      code: |
        car_model = "Camry"
        car_make = "Toyota"
---
{% endhighlight %}

The question above tells **docassemble** that if the interview logic
calls for either `car_model` or `car_make`, the question should be
tried.  When the user clicks on one of the buttons, the code will be
executed and the variables will be set.

## <a name="field"></a>`field` without `buttons` or `choices`

{% highlight yaml %}
---
question: |
  Welcome to the interview!
subquestion: |
  Your participation means a lot to us.
field: user_saw_intro
---
{% endhighlight %}

A `question` with a `field` and no `buttons` will offer the user a
"Continue" button.  When the user presses "Continue," the variable
indicated by `field` will be set to `True`.

## <a name="signature"></a>`signature`

The `signature` directive presents a special screen in which the user
can sign his or her name with the trackpad or other pointing device.

For example:

{% highlight yaml %}
---
question: |
  Please sign your name below.
signature: user_signature
under: |
  ${ user_first_name } ${ user_last_name } 
---
{% endhighlight %}

On the screen, the `question` text appears first, then the
`subquestion` text, then the signature area appears, and then the
`under` text appears.

In this example, the `user_signature` variable will be set to an
object of type `DAFile`.  This variable can be included in the same
way that a document upload can be included.  For example:

{% highlight yaml %}
---
question: |
  Is this your signature?
subquestion: |
  ${ user_signature }
yesno: user_signature_verified
---
{% endhighlight %}

or, if you want to control the width of the image:

{% highlight yaml %}
---
question: |
  Is this your signature?
subquestion: |
  ${ user_signature.show(width='1in') }
yesno: user_signature_verified
---
{% endhighlight %}

## <a name="fields"></a>`fields`

The `fields` statement is used to present the user with a list of
fields.

{% highlight yaml %}
---
question: Tell me about yourself
fields:
  - Favorite color: user_favorite_color
  - Description of your ideal vacation: user_ideal_vacation
    datatype: area
    required: false
---
{% endhighlight %}

The `fields` must consist of a list in which each list item is one or
more key/value pairs.

There are a number of keys that have a special meaning (and therefore
are not available for use as a label).  If a key is not a special key,
it will be treated as a label, the value of which will refer to a
variable.

The following are the keys that have special meaning:

* `datatype`: affects how the data will be collected, validated and
  stored; see below.
* `required`: the value is either `true` or `false`.  By default, all
  fields are required.
* `hint`: in text inputs, the value is provided as a [placeholder].
* `help`: the value is explanatory help text that appears when user
  clicks on the label.
* `default`: the value will be set as the default value of the 
  field.
* `choices`: a list of possible options for a multiple choice field.
  Can be a list of key/value pairs (key is what the variable will be
  set to; value is the label seen by the user) or a list of plain text
  items (in which case the label and the variable value are the same).
  When the `datatype` is `object`, `object_radio`, or
  `object_checkboxes`, `choices` indicates list of objects from which
  the user will choose.
* `exclude`: this is used in combination with `choices` when the
  `datatype` is `object`, `object_radio`, or `object_checkboxes`.  Any
  object in `exclude` will be omitted from the list of choices if it
  is present in `choices`.
* `code`: code that generates a list of possible options for a
  multiple choice field.
* `shuffle`: the value is either `true` or `false`.  When used with
  `code` or `choices`, it randomizes the list of choices.
* `disable others`: if set to true, then when the user changes the
  value of the field to something, all the other fields in the
  question will be disabled.
* `note`: the value is [Markdown] text that will appear on the screen;
  useful for providing guidance to the user on how to enter
  information.
* `label`: Instead of expressing your labels and variable names in the
  form of `- Label: variable_name`, you can specify a label using the
  `label` key and the variable name using the `field` key.
* `field`: See the explanation of `label` above.
* `html`: like `note`, except raw HTML.
* `script`: raw HTML to be appended to the bottom of the page; usually
  used for Javascript code that interacts with HTML specified in
  `html` entries.
* `css`: raw HTML to be appended to the HTML head; usually used to
  provide css classes for HTML specified in `html` entries.
* `no label`: if you use `no label` as the label for your variable,
  the label will be omitted.  On wide screens, the field will fill
  more of the width of the screen if the label is set to `no label`.
  To keep the width of the field normal, but have a blank label, use
  `""` as the label.

The possible `datatype` values are:

* `text`: a single-line text input box (the default).
* `area`: a multi-line text area
* `date`: a valid date.
* `email`: a valid e-mail address.
* `number`: a valid numeric value.
* `currency`: a valid numeric value; input box shows a currency symbol
  based on locale defined in the [configuration].
* `file`: a single file upload (a `DAFileList` [object] results).
* `files`: single or multiple file upload (a `DAFileList` [object]
  results).
* `camera`: like `file`, except with HTML5 that suggests using the
  device's camera to take a picture;
* `camcorder`: like `camera`, except for recording a video;
* `microphone`: like `camera`, except for recording an audio clip;
* `yesno`: checkbox with label, aligned with labeled fields.
* `yesnowide`: checkbox with label, full width of area.
* `checkboxes`: show `choices` list as checkboxes; variable will be a
  dictionary with items set to true or false depending on whether the
  option was checked.  No validation is done to see if the user
  selected at least one, regardless of the value of `required`.
* `object`: this is used when you would like to use a variable to
  refer to an existing object.  You can provide the list of objects to
  choose from in one of two ways.  The first way is to list the object
  names under `selections`.  The second way is to provide `code` that
  calls the `selections()` [function].  The user will be presented
  with a pull-down selector.
* `object_radio`: like `object`, except the user interface uses radio
  buttons rather than a pull-down list.
* `object_checkboxes`: this is used when you would like to use a
  question to set the elements of an object of type `DAList`.  The
  choices in `choices` (optionally modified by `exclude`) will be
  presented to the user as checkboxes.  The `.gathered` attribute of
  the variable will be set to `True` after the elements are set.
* `radio`: show `choices` list as radio buttons instead of a dropdown
  [select] tag (which is the default).  Variable will be set to the
  value of the choice.

For some field types, you can require input validation by adding the
following to the definition of a field:

* `min`: for `currency` and `number` data types, require a minimum
  value.
  [See here for more information](http://jqueryvalidation.org/min-method).
* `max`: for `currency` and `number` data types, require a maximum
  value.
  [See here for more information](http://jqueryvalidation.org/max-method).
* `minlength`: require a minimum number of characters in a textbox,
  number of checkboxes checked, etc.
  [See here for more information](http://jqueryvalidation.org/minlength-method).
* `maxlength`: require a maximum number of characters in a textbox,
  number of checkboxes checked, etc.
  [See here for more information](http://jqueryvalidation.org/maxlength-method).

Here is a long example that illustrates many of these features.

![Screenshot of fields]({{ site.baseurl }}/img/fields-example.png)

The following `question` block generates the question above.  Compare
the screenshot to the `question` to see how it works.

{% highlight yaml %}
---
imports:
  - us
---
modules:
  - docassemble.base.util
---
mandatory: true
code: |
  need(final_screen)
---
sets: final_screen
question: All done
subquestion: |
  Your income is ${ user_annual_income }.
---
question: Tell me more about yourself
fields:
  - Description: user_description
    datatype: area
    hint: |
      E.g., you can describe your hair color, eye color, 
      favorite movies, etc.
  - Annual income: user_annual_income
    datatype: currency
    min: 100
  - E-mail address: user_email_address
    datatype: email
  - Been vaccinated: user_vaccinated
    datatype: yesno
  - Seen Mount Rushmore: mount_rushmore_visited
    datatype: yesnowide
  - Belly button type: belly_button_type
    datatype: radio
    choices:
      - Innie
      - Outie
  - html: |
      The date and time is <span class="mytime" id="today_time"></span>.
  - script: |
      <script>document.getElementById("today_time").innerHTML = Date();</script>
  - css: |
      <link rel="stylesheet" href="${ url_of('docassemble.demo:data/static/my.css') }">
  - Number of friends: number_of_friends
    datatype: radio
    choices:
      - One: 1
      - Two: 2
      - Three: 3
  - Degrees obtained: degrees
    datatype: checkboxes
    choices:
      - High school
      - College
      - Graduate school
  - State you grew up in: home_state
    code: |
      us.states.mapping('abbr', 'name')
  - note: |
      #### Politics

      Tell me about your political views.
  - no label: political_views
    default: I have no political views
    maxlength: 30
  - Political preference: political_party
    datatype: radio
    shuffle: true 
    choices:
      - Republican
      - Democrat
      - Independent
---
{% endhighlight %}

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testfields.yml){:target="_blank"}.)

The referenced [CSS file] contains the following:

{% highlight css %}
.mytime {
    color: green;
}
{% endhighlight %}

### Multiple-choice questions in `fields`

Note that adding `code` to a field makes it a multiple-choice
question.  If you have a multiple-choice question and you want to
reuse the same selections several times, you do not need to type in
the whole list every time.  You can define a variable to contain the
list and a `code` block that defines the variable.  For example:

{% highlight yaml %}
---
mandatory: true
code: |
  final_screen
---
sets: final_screen
question: |
  Your favorite fruit is the ${ favorite_fruit }, which is the king of
  all fruits.
subquestion: |
  Your brother, who is not so wise, is partial to the
  ${ favorite_fruit_of_brother }.
---
question: |
  What is your favorite fruit?
fields:
  - Fruit: favorite_fruit
    code: |
      myoptions
---
question: |
  What is your brother's favorite fruit?
fields:
  - Fruit: favorite_fruit_of_brother
    code: |
      myoptions
---
code: |
  myoptions = [
                {'apple': "Apples"},
                {'orange': "Oranges"},
                {'pear': "Pears"}
              ]
---
{% endhighlight %}

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testpulldown.yml){:target="_blank"}.)

### Assigning existing objects to variables

You might want to ask the user a multiple-choice question like "which
of the people you have told me about do you want to sue?

## <a name="sets"></a>`sets`

{% highlight yaml %}
---
question: We are all done.
sets: user_done
---
{% endhighlight %}

A `sets` line tells **docassemble** that if it is looking for the
value of a particular variable, it should try asking the question in
the question block containing the `sets` statement.

This does not necessarily mean that the variable identified in `sets`
will actually be set.  For example, the question above does not allow
the user to do anything.

When used as an "empty promise" of setting a variable, the `sets`
statement is useful for creating the final screen that the user sees
before exiting the interview.  For example, the interview logic can
cause the "question" above to appear by including a final line of code
such as:

{% highlight text %}
need(user_done)
{% endhighlight %}

The `user_done` variable will never be set to any value, but that is
ok, because you don't want the user to be able to "get past" the final
screen.

## <a name="sets special"></a>`sets` with special buttons/choices 

The `sets` statement is also used in conjunction with `buttons` in
multiple choice questions where there is no `field` to be set.

{% highlight yaml %}
---
question: We are all done.
sets: user_done
buttons:
  - Exit: exit
  - Restart: restart
---
{% endhighlight %}

The above example allows the user to "exit" the interview (be redirected
to a specific web site that is pre-set in the **docassemble**
[configuration] or "restart" the interview (go back to the beginning).

There are six special button functions:

* `restart`
* `exit`
* `leave`
* `continue`
* `refresh`
* `signin`

`restart` resets the user's variable store, except that any parameters
that were originally passed through as URL parameters will be used again.

`exit` means that the user's variable store will be reset and the user
will be redirected either to the URL given by the associated `url`
text, or if no `url` is defined, to the `exit` page defined in the
[configuration].  If the user tries to come back to the interview
again, he will start the interview again, as though it had never been
started.  Original URL parameters will be lost.

For example:

{% highlight yaml %}
---
question: |
  Congratulations, you found Nemo!
sets: user_done
choices:
  - Try again: restart
  - Learn More: exit
    url: https://en.wikipedia.org/wiki/Amphiprioninae
---
{% endhighlight %}

[Mako] can be used in the `url` text.

`leave` works like `exit` except that the user's variable store will be left
intact.  This means that if the user comes back to the interview
again, he will pick up where he left off.

`continue` means that **docassemble** will look for another question
in the interview that might define the necessary variable.

`refresh` re-runs the interview logic.  It has much the same effect as
refreshing the page in the browser.  It is useful in multi-user
interviews when the user is waiting for another user to finish
entering information.  It can also be useful in interviews that use
external data sources.

`signin` redirects the user to the **docassemble** sign-in page.

Instead of using `buttons`, you can use `choices` to get a radio list
instead of a selection of buttons.

{% highlight yaml %}
---
question: We are all done.
sets: user_done
choices:
  - Exit: exit
  - Restart: restart
---
{% endhighlight %}

The functionality is the same.

## <a name="event"></a>`event`

{% highlight yaml %}
---
event: role_event
question: All done for now.
subquestion: |
  Someone else needs to answer questions now.  You will be notified
  when you can resume the interview.
buttons:
  - Exit: leave
---
{% endhighlight %}

The `event` line acts just like `sets`: it advertises to
**docassemble** that the question will potentially define a variable.

In the example above, the `event` line tells **docassemble** that this
`question` should be displayed to the user if **docassemble**
encounters the `role_event`, which is a special "event" that can
happen in multi-user interviews (see
[roles]({{site.baseurl}}/docs/roles.html)).  The event is triggered
when the interview reaches a point when a person other than the
current user needs to answer a question.  For example, while a client
is filling out an interview, the interview logic might call for a
variable that can only be set by an advocate who reviews the client's
answers.  In this scenario, a `role_event` will be triggered.  When
this happens, **docassemble** will look for a `question` or `code`
block that defines the variable `role_event`, and it will find the
example question above.

## <a name="review"></a>review

A `review` block allows interview authors to provide a screen where
users can review and edit their answers.  Typically, the user will get
to this screen by selecting an option from the web app menu (e.g.,
"Review Answers"), or by clicking on a hyperlink within `subquestion`
text (e.g., "to review the answers you have provided so far, click
here").

Here is an example of a `review` block that is launched from the menu:

{% highlight yaml %}
---
event: review_answers
question: |
  Revisit questions
subquestion: |
  These are the questions you have answered so far.  Click to revisit.
review:
  - Favorite fruit: fruit
  - Favorite vegetable: vegetable
  - Favorite fungus: fungi
---
mandatory: true
code: |
  menu_items = [ action_menu_item('Review Answers', 'review_answers') ]
---
{% endhighlight %}

If `fruit` and `vegetable` have been defined, and the user selects
"Review Answers" from the menu, he or see will see this:

![Screenshot of review]({{ site.baseurl }}/img/review-block.png)

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testreview.yml){:target="_blank"}.)

Note that the `review` block does not show a link for "Favorite
fungus" because the variable `fungi` has not been defined yet.
However, once `fungi` is defined, the `review` block would show it.

This behavior is different from the typical behavior of
**docassemble** blocks.  Normally, referring to a variable that has
not yet been defined will trigger the asking of a question that will
define that variable.  In the `review` block, however, the presence of
an undefined variable simply causes the item to be omitted from the
display.

For more information about adding menu items, see the sections on
[special variables] and [functions].

### <a name="review customization"></a>Customizing the display of `review` options

You can provide the user with a review of answers and buttons that the
user can press to revisit an answer:

{% highlight yaml %}
---
event: review_answers
question: |
  Revisit your answers
review:
  - Revisit fruit: fruit
    button: |
      You said your favorite fruit was ${ fruit }.
  - Revisit vegetable: vegetable
    button: |
      You said your favorite vegetable was ${ vegetable }.
  - Revisit fungus: fungi
    button: |
      You said your favorite fungus was ${ fungi }.
---
{% endhighlight %}

This results in a screen that looks like this:

![Screenshot of review with buttons]({{ site.baseurl }}/img/review-block-buttons.png)

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testreview2.yml){:target="_blank"}.)

In addition, the `review` block, like the `fields` block, allows you
to use `note`, `html`, `script`, and `css` entries.

If these are modified with the optional `show if` modifier, they will
only be displayed if the variable referenced by the `show if` modifier
has been defined.  In addition, if any of these entries refer to a
variable that has not been defined yet, they will be omitted.

{% highlight yaml %}
---
event: review_answers
question: |
  Review your answers
review:
  - note: |
      Revisit your food preferences.
    show if: fruit
  - Favorite fruit: fruit
  - Favorite vegetable: vegetable
  - Favorite fungus: fungi
---
{% endhighlight %}

![Screenshot of review with note]({{ site.baseurl }}/img/review-block-note.png)

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testreview3.yml){:target="_blank"}.)

The `review` block allows you to add `help` text to an entry, in
which case the text is shown underneath the hyperlink.  If this text
expects a variable to be defined that has not actually been defined,
the item will not be shown.  Note: this is not available with the
`button` display format.

{% highlight yaml %}
---
event: review_answers
question: |
  Review your answers
review:
  - Favorite fruit: fruit
    help: |
      You indicated you liked ${ fruit }.
  - Favorite vegetable: vegetable
    help: |
      You indicated you liked ${ vegetable }.
  - Favorite fungus: fungi
    help: |
      You indicated you liked ${ fungi }.
---
{% endhighlight %}

![Screenshot of review with help]({{ site.baseurl }}/img/review-block-help.png)

([Try it out here](https://demo.docassemble.org?i=docassemble.demo:data/questions/testreview4.yml){:target="_blank"}.)

### <a name="review auto"></a>Why can't `review` blocks be automatically generated?

The list of variables to display to the user in a `review` block needs
to be specified by the interview author.  There are several reasons
why this needs to be done manually as opposed to automatically:

1. Variables in your interview may be interdependent.  You do not
   necessarily want to allow the interviewee to edit any past answer
   at will because this may result in internal inconsistencies or
   violations of the logic of your interview.  For example, if your
   interview has a variable called `eligible_for_medicare`, which is
   set after the user answers a series of questions, you would not
   want the user to be able to go back and set his or her age to 30,
   at least not without a reconsideration of the definition of
   `eligible_for_medicare`.  Therefore, it is important that the
   interview author control what the user can edit.
2. A list of answers already provided might not be user-friendly
   unless the interview author presents it in a logically organized
   fashion.  The order in which the questions were asked is not
   necessarily the most logical way to present the information for
   editing.

# <a name="variable names"></a>A note about variable names

Variable names are [Python identifiers], which means they can be any
sequence of uppercase or lowercase letters, digits, and underscores,
except the first character cannot be a digit.  No spaces are allowed
and no punctuation is allowed except for the underscore, `_`.

The following are valid variable names:

* `fried_fish1`
* `NyanCat`
* `nyancat` (variables are case-sensitive, so this is not the same as
  the above)
* `__f645456DG_greij_43` (but why would you use something so ugly?)
* `USER_PHONE_NUMBER` (ok, but why are you yelling?)

The following are **not** valid variable names, and if you try to use
such variable names you will may get an error or unexpected results:

* `8th_plaintiff` (you can't begin a variable name with a number;
  [Python] will say "invalid syntax")
* `fried.fish1` (this is valid code, but [Python] will think you are
referring to the attribute `fish1` of the object `fried`)
* `user's_phone_number` (apostrophes are not allowed; [Python]
  recognizes them as single quotes)
* `favorite animal` (spaces are not allowed)
* `beneficiary#1` (punctuation marks other than `_` are not allowed)
* `applicant_résumé` (only plain alphabet characters can be used)

See [reserved variable names] for a list of variable names that you
cannot use because they conflict with built-in names that [Python] and
**docassemble** use.

[configuration]: {{ site.baseurl }}/docs/configuration.html
[select]: http://www.w3schools.com/tags/tag_select.asp
[placeholder]: http://www.w3schools.com/tags/att_input_placeholder.asp
[code blocks]: {{ site.baseurl }}/docs/code.html
[Mako]: http://www.makotemplates.org/
[Markdown]: https://daringfireball.net/projects/markdown/
[YAML]: https://en.wikipedia.org/wiki/YAML
[object]: {{ site.baseurl }}/docs/objects.html
[objects]: {{ site.baseurl }}/docs/objects.html
[Python identifiers]: https://docs.python.org/2/reference/lexical_analysis.html#identifiers
[reserved variable names]: {{ site.baseurl }}/docs/reserved.html
[Python]: https://en.wikipedia.org/wiki/Python_%28programming_language%29
[question]: {{ site.baseurl }}/docs/questions.html
[CSS file]: https://github.com/jhpyle/docassemble/blob/master/docassemble_demo/docassemble/demo/data/static/my.css
[function]: {{ site.baseurl }}/docs/functions.html
[functions]: {{ site.baseurl }}/docs/functions.html
[special variables]: {{ site.baseurl }}/docs/special.html