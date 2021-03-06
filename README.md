# mail-map
PHP Library for Imap Connections

# Requirements
* >= php 5.6
* php imap extension
* php mb_string extension

# Installation
```
composer require mail-map/mail-map
```

# Usage
This is a small client library that interfaces with php's imap library.

It is primarily intended for reading emails from a mail server, and does not currently support anything much more complex than that. The one caveat to that is that there is currently support for moving messages between mailboxes.

This was designed to be highly configurable and extendable. All the primary classes in the library are injected into one another via contract implementation, so its very easy to extend the classes or create your own implementations in order to hook into the library at any part of the process of making imap connection to returning the found emails.

A simple class that will query imap connections has been provided, however you are encouraged to extend this, or write your own in order to get more complex results.

## Example Usage
```php
// Instantiate a new MailMap with configuration for logging into the mail server
$mailMap = new \MailMap\MailMap([
    'host' => 'imap.gmail.com',     // Required argument
    'user' => 'example@gmail.com',  // Required argument
    'password' => 'password',       // Required argument
    'port' => 993,                  // Optional - Default 993
    'service' => 'imap',            // Optional - Default 'imap'
    'encryption' => 'ssl'           // Optional - Default 'ssl'
]);

// The 'query' method takes the name of the mailbox to query, and a 'callable'
// to add query conditions. The callable will receive a query object, and must
// return a query object
$emails = $mailMap->query('My-Inbox', function ($query) {
    return $query->since('07-Sep-2016')->limit(3);
});
```

## MailMap\\MailMap Api
A helper class for creating imap connections and searching emails

### query($inbox = 'INBOX', callable $queryCall = null)
Execute a query on the connection
* string $inbox - Name of the inbox to query on
* callable $queryCall - Callable that takes a MailMap\\Query Object as an argument. Conditions, sorting, and limits can be set on the query in the callable on the query object. The callable must return the query object.

Returns array of `MailMap\Mail` objects.

### mailboxes($mailboxSearch = '\*', $withConnection = false)
Get a list of available mailboxes on the connection
* string $mailboxSearch - Regex to search for mailboxes
* bool $withConnection - Mailboxes are returned from the imap library with the connection string as a prefix. By default, this will be stripped off, but passing in `true` will leave the connection string.

Returns array of strings.

## MailMap\\Query Object Api
Object that is passed into the `MailMap\MailMap`'s `query` method. This object contains methods that will add conditions, sorting, and limiting to the executed query.

### Conditions
Methods for adding conditions correspond to the [imap_search criteria in the php docs pages](http://php.net/manual/en/function.imap-search.php). There is an available method for each of these criteria.

The condition methods are as follows
* all()
* answered()
* bcc(string $match)
* before(string $date)
* body(string $match)
* cc(string $match)
* deleted()
* flagged()
* from(string $match)
* keyword(string $match)
* new()
* old()
* on(string $date)
* recent()
* seen()
* since(string $date)
* subject(string $match)
* text(string $match)
* to($match)
* unanswered()
* undeleted()
* unflagged()
* unkeyword()
* unseen()

See the [php docs on searching](http://php.net/manual/en/function.imap-search.php) for more details and on how to format arguments for these methods.

Example usage - Finding all deleted emails from `example@gmail.com` on August 1st 2015
```php
$query->deleted()->from('example@gmail.com')->on('01-Aug-2015');
```

### Sorting
The php imap library only supports one sort on each query, so additional sorting must be done elsewhere. Sort order can be added to the query by calling the 'order' method, and providing the order flag, and the direction to sort (default is 'desc')

Example usage - Sort the messages by subject in ascending order
```php
$query->order(SORTSUBJECT, 'asc');
```

Details on sorting, and the available sort flags can be found on the [imap_sort criteria in the php docs](http://php.net/manual/en/function.imap-sort.php).

### Limiting
The php imap library does not support limiting results. For this reason, there is a default maximum limit set on the returned results for queries of 100 as a safety net. This can be overwritten by simply passing in a larger value into the limit method, or by changing the static property on the Query class.

Change globally
```php
MailMap\Query::$max = 200;
```

Or set inline on an individual query
```php
$query->limit(200);
```

## MailMap\\Map Api
Objects that are returned by the `MailMap\MailMap`'s `query` method. The objects are generated by the `MailMap\MailFactory` class. As stated above, the factory and mail object classes can easily be extended, or a different implementation provided in order to meet specific requirements.

The `Mail` object has the following public properties
* int $id - Uid of the email
* int $msgNo - Message number of the email
* string $subject - Subject line
* int $date - Date as timestamp
* array $to - List of address objects that email was sent to
* array $cc - List of address objects that email cc'd
* array $bcc - List of address objects that email bcc'd
* array $sender - List of address objects of email sender
* array $from - List of address objects email sent from
* array $replyTo - List of address objects of reply to
* bool $recent - Recent flag
* bool $unseen - Unseen flag
* bool $flagged - Flagged flag
* bool $answered - Answered flag
* bool $deleted - Deleted flag
* bool $draft - Draft flag

An address object (see `to`, `cc`, `bcc`, `sender`, `from`, `replyTo` properties above) is a standard object that has an `address` property, that contains the email address as `'example@gmail.com'`, a `name` property that contains the name corresponding to the email address as `'John Smith'`, and the domain of the address as `'gmail.com'`.

### getStream()
Get the underlying imap stream resource

### header($key, $default = null)
Find an email header value, or return $default
* string $key - The name of the header
* mixed $default - Return this value if no header called $key is found

### body($mimeType = 'html')
Get the parts of the email that correspond to the mimeType, and piece them together as a string.
* string $mimeType - The mime type the mail should be returned as. Defaults to 'html'

### fullBody()
Gets the full body structure of the email as an array. The parts of the email are flattened by the mail factory, and are accessible by key. The keys are the part numbers as 1, 1.1, 2, 2.1, 2.1.1 etc.

### move($mailbox)
Move the email to another mailbox
* string $mailbox - Name of the mailbox to move mail to

Returns boolean for successful move

# Contributing
Contributions are welcome. This follows the PSR-2 coding standard and the PSR-4 autoloading standard.
