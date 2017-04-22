# Rails

Rails is a web-application framework that includes everything needed to create database-backed web applications according to the [Model-View-Controller (MVC)](http://en.wikipedia.org/wiki/Model-view-controller) pattern.

## Main frameworks and modules:

* [ActionCable](https://github.com/rails/rails/tree/master/actioncable) - build-in framework for live connection, based on [WebSockets](https://en.wikipedia.org/wiki/WebSocket), provides real-time stuff: multiple connections, receiving web notifications.
* [ActionMailer](https://github.com/rails/rails/tree/master/actionmailer) - framework for designing email service layer. Can use many layouts, templates, queues (#deliver_now, #deliver_later) [etc](https://github.com/rails/rails/blob/master/actionmailer/lib/action_mailer.rb). We also can use [SMTP](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol) for sending emails.
* [ActionPack](https://github.com/rails/rails/tree/master/actionpack) - framework for handling and responding to web requests. Contains Action Dispatch and Action Controller modules. Action Dispatch is working with routing, urls, flash notifications, sessions, http headers [etc](https://github.com/rails/rails/blob/master/actionpack/lib/action_dispatch.rb). Action Controller no need to explain, you can see list of modules [here](https://github.com/rails/rails/blob/master/actionpack/lib/action_controller.rb).
* [ActionView](https://github.com/rails/rails/tree/master/actionview) - framework for handling view template lookup and rendering, and provides view helpers that assist when building HTML forms and many [more](https://github.com/rails/rails/blob/master/actionview/lib/action_view.rb).
* [ActiveJob](https://github.com/rails/rails/tree/master/activejob) - framework for declaring jobs and making them run on a variety of queueing backends.
* [ActiveModel](https://github.com/rails/rails/tree/master/activemodel) - module witch provides a known set of interfaces for usage in model classes, includes model name introspections, conversions, translations and validations, serialization.
* [ActiveRecord](https://github.com/rails/rails/tree/master/activerecord) - connects classes to relational database tables to establish an almost zero-configuration persistence layer for applications. Includes associations, [aggregations](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/aggregations.rb), validations, callbacks, [transactions](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/transactions.rb), schema and migrations. ActiveRecord supports adapters for many DataBases: [mysql](https://github.com/brianmario/mysql2), [postgresql](https://github.com/ged/ruby-pg), [oracle](https://github.com/rsim/oracle-enhanced), [mongo](https://github.com/jimm/activerecord-mongo-adapter) etc. Interesting post [SQLite vs MySQL vs PostgreSQL](https://www.digitalocean.com/community/tutorials/sqlite-vs-mysql-vs-postgresql-a-comparison-of-relational-database-management-systems).
* [ActiveSupport](https://github.com/rails/rails/tree/master/activesupport) - a collection of utility classes and standard library extensions that were found useful for the Rails framework, list [here](https://github.com/rails/rails/blob/master/activesupport/lib/active_support.rb). 