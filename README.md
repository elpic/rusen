[S]imple [RU]by [E]xception [N]otification
====

The Simple Ruby Exception Notification (a.k.a Rusen) gem provides a simple way for login and
sending errors in any ruby application.

The email includes information about the current request, session, and
environment, and also gives a backtrace of the exception.

Installation
---

Just add the following line in your Gemfile

```ruby
gem 'rusen'
```

Usage
---

### With Global Configuration.

The easiest way to use it is with global configuration.

First you configure Rusen
```ruby
require 'rusen'

Rusen.settings.outputs = [:io, :email]
Rusen.settings.sections = [:backtrace, :request, :session, :environment]
Rusen.settings.email_prefix = '[ERROR] '
Rusen.settings.sender_address = 'some_email@example.com'
Rusen.settings.exception_recipients = %w(dev_team@example.com test_team@example.com)
Rusen.settings.smtp_settings = {
  :address              => 'smtp.gmail.com',
  :port                 => 587,
  :domain               => 'example.org',
  :authentication       => :plain,
  :user_name            => 'dev_team@moove-it.com',
  :password             => 'xxxxxxx',
  :enable_starttls_auto => true
}
```
And the you can start sending notifications:
```ruby
begin
  method.call
rescue Exception => exception
  Rusen.notify(exception)
end
```
In this way if you modify the notification settings in runtime every notification
being sent after will take the modifications into account.

### With Local Configuration.

This method is intended fro more control, you may for example want to send emails when a particular exception occurs but juts print to stdout for others.
To archive this you can do the following:
```ruby
@email_settings = Settings.new
@email_settings.outputs = settings[:email]
Rusen.settings.sections = [:backtrace, :request, :session]
@email_settings.email_prefix = '[ERROR] '
@email_settings.sender_address = 'some_email@example.com'
@email_settings.exception_recipients = %w(dev_team@example.com test_team@example.com)
@email_settings.smtp_settings = {
                                  :address              => 'smtp.gmail.com',
                                  :port                 => 587,
                                  :domain               => 'example.org',
                                  :authentication       => :plain,
                                  :user_name            => 'dev_team@moove-it.com',
                                  :password             => 'xxxxxxx',
                                  :enable_starttls_auto => true
                                }

@email_notifier = Notifier.new(@email_settings)

@stdout_settings = Settings.new
@stdout_settings.outputs = settings[:io]
Rusen.settings.sections = [:backtrace]

@stdout_notifier = Notifier.new(@stdout_settings)
```
and then:
```ruby
begin
  method.call
rescue SmallException => exception
  @stdout_notifier.notify(exception)
rescue BigException => exception
  @email_notifier.notify(exception)
end

Middleware
---
Rusen comes with a rack and rails especial (soon to come) middleware for easy usage.

### Rack
To use Rusen in any rack application you just have to add the following code some where in your app (ex: config/initializers/rusen.rb):
```ruby
require 'rusen/middleware/rusen_rack'

use Rusen::Middleware::RusenRack,
    :outputs => [:io, :email],
    :sections => [:backtrace, :request, :session, :environment],
    :email_prefix => '[ERROR] ',
    :sender_address => 'some_email@example.com',
    :exception_recipients => %w(dev_team@example.com test_team@example.com),
    :smtp_settings => {
                        :address              => 'smtp.gmail.com',
                        :port                 => 587,
                        :domain               => 'example.org',
                        :authentication       => :plain,
                        :user_name            => 'dev_team@moove-it.com',
                        :password             => 'xxxxxxx',
                        :enable_starttls_auto => true
                      }
```
This will capture any unhandled exception send an email and write a trace in stdout.

Settings
---
### Outputs
Currently supported outputs are :io and :email, but the more outputs are easy to add so can customize Rusen to your needs.
Note: :io will only print to stdout for now but there are plans to extend it to anything that Ruby::IO supports

### Sections
You can choose the sections to output simply by setting the appropriate values in the configuration.

### Email Settings
All the email settings are self explanatory but you contact me if any of them need clarification.

Extending to more outputs
---
Soon to come!