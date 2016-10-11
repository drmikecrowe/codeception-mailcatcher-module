# Codeception MailCatcher Module

[![Build Status](https://travis-ci.org/captbaritone/codeception-mailcatcher-module.svg)](https://travis-ci.org/captbaritone/codeception-mailcatcher-module)

This module will let you test emails that are sent during your Codeception
acceptance tests. It depends upon you having
[MailCatcher](http://mailcatcher.me/) installed on your development server.

It was inspired by the Codeception blog post: [Testing Email in
PHP](http://codeception.com/12-15-2013/testing-emails-in-php). It is currently
very simple. Send a pull request or file an issue if you have ideas for more
features.

## Installation

Add the package into your composer.json:

    {
        "require-dev": {
            "codeception/codeception": "*",
            "captbaritone/mailcatcher-codeception-module": "1.*"
        }
    }

Tell Composer to download the package:

    php composer.phar update

Then enable it in your `acceptance.suite.yml` configuration and set the url and
port of your site's MailCatcher installation:

    class_name: WebGuy
    modules:
        enabled:
            - MailCatcher
        config:
            MailCatcher:
                url: 'http://project.dev'
                port: '1080'

You will then need to rebuild your actor class:

    php codecept.phar build

## Optional Configuration

If you need to specify some special options (e.g. SSL verification or authentication
headers), you can set all of the allowed [Guzzle request options](https://guzzle.readthedocs.org/en/5.3/clients.html#request-options):

    class_name: WebGuy
    modules:
        enabled:
            - MailCatcher
        config:
            MailCatcher:
                url: 'http://project.dev'
                port: '1080'
                guzzleRequestOptions:
                    verify: false
                    debug: true
                    version: 1.0

You will then need to rebuild your actor class:

    php codecept.phar build

## Example Usage

    <?php

    $I = new WebGuy($scenario);
    $I->wantTo('Get a password reset email');

    // Cleared old emails from MailCatcher
    $I->resetEmails();

    // Reset
    $I->amOnPage('forgotPassword.php');
    $I->fillField("input[name='email']", 'user@example.com');
    $I->click("Submit");
    $I->see("Please check your email");

    $I->seeInLastEmail("Please click this link to reset your password");

## Actions

### resetEmails

Clears the emails in MailCatcher's list. This is prevents seeing emails sent
during a previous test. You probably want to do this before you trigger any
emails to be sent

Example:

    <?php
    // Clears all emails
    $I->resetEmails();
    ?>

### seeInLastEmail

Checks that an email contains a value. It searches the full raw text of the
email: headers, subject line, and body.

Example:

    <?php
    $I->seeInLastEmail('Thanks for signing up!');
    ?>

* Param $text

### seeInLastEmailTo

Checks that the last email sent to an address contains a value. It searches the
full raw text of the email: headers, subject line, and body.

This is useful if, for example a page triggers both an email to the new user,
and to the administrator.

Example:

    <?php
    $I->seeInLastEmailTo('user@example.com', 'Thanks for signing up!');
    $I->seeInLastEmailTo('admin@example.com', 'A new user has signed up!');
    ?>

* Param $email
* Param $text

### dontSeeInLastEmail

Checks that an email does NOT contain a value. It searches the full raw text of the
email: headers, subject line, and body.

Example:

    <?php
    $I->dontSeeInLastEmail('Hit me with those laser beams');
    ?>

* Param $text

### dontSeeInLastEmailTo

Checks that the last email sent to an address does NOT contain a value. It searches the
full raw text of the email: headers, subject line, and body.

Example:

    <?php
    $I->dontSeeInLastEmailTo('admin@example.com', 'But shoot it in the right direction');
    ?>

* Param $email
* Param $text

### grabMatchesFromLastEmail

Extracts an array of matches and sub-matches from the last email based on
a regular expression. It searches the full raw text of the email: headers,
subject line, and body. The return value is an array like that returned by
`preg_match()`.

Example:

    <?php
    $matches = $I->grabMatchesFromLastEmail('@<strong>(.*)</strong>@');
    ?>

* Param $regex

### grabFromLastEmail

Extracts a string from the last email based on a regular expression.
It searches the full raw text of the email: headers, subject line, and body.

Example:

    <?php
    $match = $I->grabFromLastEmail('@<strong>(.*)</strong>@');
    ?>

* Param $regex

### grabAllEmails

Retrieves the full raw text of the email: headers, subject line, and body.

Example:

    <?php
    $match = $I->grabAllEmails();
    ?>

Returns:

    [
        0 => [
            "id" => 1
            "sender" => <from@email.com>
            "recipients" => [
                0 => <to@email.com>
            ]
            "subject" => Here's my subject
            "source" => Date: Mon, 10 Oct 2016 22:55:55 -0400
    Return-Path: from@email.com
    To: to@email.com
    From: From <from@email.com>
    Subject: Here's my subject
    Message-ID: <5b83a7747e4065e19ac2f833311e1a9e@tld.com>
    X-Priority: 3
    X-Mailer: PHPMailer 5.2.1 (http://code.google.com/a/apache-extras.org/p/phpmailer/]
    MIME-Version: 1.0
    Content-Transfer-Encoding: 8bit
    Content-Type: text/html; charset="utf-8"

    Here's the body

            "size" => 3696
            "type" => text/html
            "created_at" => 2016-10-11T02:55:55.000+00:00
            "formats" => [
                "0" => html
            ]
            "attachments" => []
        ],
        1 => [
            "id" => 2
            "sender" => <from@email.com>
            "recipients" => [
                "0" => <to@email.com>
            ]
            "subject" => Here's my subject
            ....
        ]
    ]


### lastMessageFrom

Grab the full email object sent to an address.

Example:

    <?php
    $email = $I->lastMessageFrom('example@example.com');
    $I->assertNotEmpty($email['attachments']);
    ?>

### lastMessage

Grab the full email object from the last email.

Example:

    <?php
    $email = $I->grabLastEmail();
    $I->assertNotEmpty($email['attachments']);
    ?>

### grabMatchesFromLastEmailTo

Extracts an array of matches and sub-matches from the last email to a given
address based on a regular expression. It searches the full raw text of the
email: headers, subject line, and body. The return value is an array like that
returned by `preg_match()`.

Example:

    <?php
    $matchs = $I->grabMatchesFromLastEmailTo('user@example.com', '@<strong>(.*)</strong>@');
    ?>

* Param $email
* Param $regex

### grabFromLastEmailTo

Extracts a string from the last email to a given address based on a regular
expression.  It searches the full raw text of the email: headers, subject
line, and body.

Example:

    <?php
    $match = $I->grabFromLastEmailTo('user@example.com', '@<strong>(.*)</strong>@');
    ?>

* Param $email
* Param $regex

### grabEmailCount

Returns the total count of emails in the mailcatcher queue.

Example:

    <?php
    $match = $I->grabEmailCount();
    ?>

### seeEmailCount

Asserts that a certain number of emails have been sent since the last time
`resetEmails()` was called.

Example:

    <?php
    $match = $I->seeEmailCount(2);
    ?>

* Param $count

# License

Released under the same liceces as Codeception: MIT
