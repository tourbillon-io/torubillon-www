{:title "Workflow Use Cases"
 :layout :page
 :page-index 3}

At the heart of Tourbillon is a sophisticated workflow engine that you can rely
on to execute tasks in a timely fashion. Scheduling events within your own
application is notoriously difficult, especially for distributed systems. With
Tourbillon, it is as simple as defining a workflow and firing off events.

Want to try out the examples below? <a href="/#sign-up">sign up for a free API
key</a>, and you can start coding immediately.

### Transactional emails

If you are selling any sort of product or service online, communicating with
your customers at strategic times is one of the most important things that you
can do.

Consider the example of having a cart abandonment feature on an e-commerce
website: if a user places an item in her cart but does not proceed to check out,
you may wish to send her an email offering a coupon to try to gain a sale. In
this case, you can create a workflow using from the dashboard or -
programatically. For this example, we will create the workflow with the PHP
client library and show the interactions using the JavaScript library.

```php
<?php
$apiKey = '<YOUR API KEY>';
$apiSecret = '<YOUR API SECRET>';
$tourbillon = new \Tourbollon\Api($apiKey, $apiSecret);
$workflow = new \Tourbillon\Workflow();

$template = new \Tourbillon\Template(
    "Hello, {{ name }} - forgetting something?\n" .
    "Use coupon code \"OOPS-I-FORGOT\" when checking out to receive\n" .
    "20% off your entire order!"
);
$templateId = $template->save();

$timeout = new \Tourbillon\Events\Recurring(3 * 60 * 60);

$workflowId = $workflow->addTransition('start', 'in-cart', 'cart:add')
    ->addTransition('in-cart', 'in-cart', 'cart:add')
    ->addTransition('in-cart', 'abandoned', $timeout)
    ->addTransition('in-cart', 'complete', 'cart:checkout')
    ->setSecurity(\Tourbillon\Security::ACCESS_ANY)
    ->setInitialState('start')
    ->save();
```

After the workflow has been created, we can create a job from the workflow and
start sending events. Workflows provide the blueprint for any number of jobs.
In the case of this workflow, we want to send an email to a different user for
each job. The example below uses the JavaScript client library to create a
job from this workflow:

```javascript
var Jobs = require('tourbillon/Jobs');
var Subscribers = require('tourbillon/Subscribers');

var job = Jobs.fromWorkflow('<WORKFLOW ID>');
var emailSubscriber = new Subscribers.Email(
    'abby.abandoner@example.com',
    'A special offer for you',
    '<TEMPLATE ID>'
);
job.onTransition('in-cart', 'abandoned', emailSubscriber);

// ...
// User adds an item to their cart
job.trigger('item:add');

// ...
// User checks out - cancels abandoned cart email
job.trigger('cart:checkout');
```

### Automated business workflows

Tourbillon is perfectly suited to managing the type of business processes that
you would manage in a typical ERP suite. In this example, we will look at a
simpke workflow that describes the submission and grading of a student's paper.

```javascript
var Workflow = require('tourbillon/Workflow');
var Subscribers = require('tourbillon/Subscribers');

var gradingWorkflow = Workflow.create();
gradingWorkflow.addTransition('start', 'drafted', 'write')
    .addTransition('drafted', 'submitted', 'submit')
    .addTransition('submitted', 'passed', 'grade:a')
    .addTransition('submitted', 'passed', 'grade:b')
    .addTransition('submitted', 'drafted', 'grade:c')
    .addTransition('submitted', 'drafted', 'grade:d')
    .addTransition('submitted', 'failed', 'grade:f')
    .setInitialState('start')
    .save();

var emailStudentPass = Subscribers.Email.create(
    'student@example.com', 'You have passed the assignment', '<TEMPLATE ID>'
);
var emailStudentImprove = Subscribers.Email.create(
    'student@example.com', 'Your draft needs a little polishing', '<TEMPLATE ID>'
);
var emailStudentFail = Subscribers.Email.create(
    'student@example.com', 'You have failed the assignment', '<TEMPLATE ID>'
);

var gradingJob = Job.fromWorkflow(gradingWorkflow);
gradingJob.onTransition('submitted', 'passed', emailStudentPass)
    .onTransition('submitted', 'drafted', emailStudentImprove)
    .onTransition('submitted', 'failed', emailStudentFail)
    .save();
```