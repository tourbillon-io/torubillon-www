{:title "Task Use Cases"
 :layout :page
 :page-index 2}

Tourbillion can be used us a wide variety of applications to schedule either one-time or
repeating events. While we hope that you use this service in innovative ways, here are
a few use cases to help get your creative juices flowing.

Before trying any of the examples below, you will need to <a href="/#sign-up">sign up for
a free API key</a>.

### Calendar Event Reminders

Say you run an online calendar service, and you want your users to have the ability to
sign up for an email or SMS reminder for their calendar events. However, managing an application
that guarantees timely message delivery and exactly-once semantics is not something that
is terribly exciting to you.

With tourbillon, adding timed message delivery can be done in just 2 steps:

#### Create a message template

After logging into your account, you can visit the "Templates" page to create a
<a href="http://mustache.github.io/">Mustache</a> template to be used for your message. If you
plan to send both SMS and email messages, you may want to create a short template for SMS and
a longer one for email. All of the placeholders in the template will be populated with the
payload that you send to tourboillon when creating an event.

    Hello, {{ name }}. Don't forget that you have {{ title }}
    coming up in {{ timeAhead }} minutes!

Take note of the ID that is returned when you create the template. You will use this ID in your
application to send messages. Don't worry if you lose the ID - you can always return to your
dashboard to find it later.

<p class="alert alert-info">
Did you know that in addition to manually creating templates through the dashboard,
you can also create them programmatically <a href="/api-docs">via our API</a>?
</p>

#### Use a client library to schedule messages

Once you have created a template, you can start scheduling messages programmatically using a
client library or directly with our REST API. Below is an example using our PHP client
library:

```php
<?php
$apiKey = '<YOUR API KEY>';
$apiSecret = '<YOUR API SECRET>';
$templateId = '<YOUR MESSAGE TEMPLATE ID>';
$sendAt = '2015-08-20 13:50:00Z';

$emailSubscriber = new \Tourbillon\EmailSubscriber(
    'f.flintstone@example.com',
    'New event reminder',
    $templateId,
    [
        'name' => 'Fred Flintstone',
        'title' => 'Auto Repair',
        'timeAhead' => 10
    ]
);
$event = new \Tourbillon\Event($sendAt, $emailSubscriber);

$tourbillon = new \Tourbollon\Api($apiKey, $apiSecret);
$tourbillon->createEvent($event);
```

With just a few lines of code, you have added a scheduled email send to your application!

### Run periodic tasks on your WordPress site

WordPress is an excellent platform for running blogs and other content-driven website, but it does not
support any reliable<a href="#footnote-wp-cron">*</a> periodic task execution out of the box.

We will use Tourbillon to trigger the Wordpress's cron script every minute, which will execute any tasks
scheduled with `wp-cron()`. First, we need to disable WordPress's default behaviour to check for scheduled
tasks on every page load:

```php
// inside wp-config.php
define('DISABLE_WP_CRON', true);
```

Second, we will log into the dashboard to create a webhook that will fire on the wp-cron.php URL.

<IMAGE HERE>

There are several advantages to using tourbillon to trigger Wordpress's cp-cron system. First, you can
guarantee that tasks will be executed on time even if during low tracffic times. Second, if you have any
long-running background tasks, they will not cause a visitor's browser to time out, losing you  exposure.
Third, you can see any output generated on your dashboard, allowing you to easily troubleshoot possible
issues.

<p><a name="footnote-wp-cron"></a>* <small>WordPress's wp-cron system can only run tasks when a visitor
loads the site, which does not work well for low-traffic sites, sites that have long-running background tasks,
or sites that need to guarantee timely execution of scheduled tasks.</small></p>
