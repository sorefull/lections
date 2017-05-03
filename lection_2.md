# Lection 2 (activeRecord, activeModel)

![it is my favorite part of rails](https://media.tenor.co/images/e7dad789c2acd076ab3de961c0d43b45/tenor.gif)

---

## 1 Active Model

Active Model - module witch provides a known set of interfaces for usage in model classes, includes: attributes methods, model name introspections, translations and validations, serialization, callbacks.

### Contains:

* [Attributes methods](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/attribute_methods.rb)

  Public method `#attributes` returns hash with attributes and values of model.

* [Callbacks](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/callbacks.rb)

  You can define witch callbacks you want to use with `#define_model_callbacks` method. As default in rails you can use all without defining.

  ```ruby
    class User < ApplicationRecord
      after_create :do_something

      private
      def do_something
        # some stuff
      end
    end
  ```

  Callbacks:

  Creating an Object: `before_validation`,  `after_validation`,  `before_save`,  `around_save`,  `before_create`,  `around_create`,  `after_create`, `after_save`, `after_commit/after_rollback`.

  Updating an Object: `before_validation`, `after_validation`, `before_save`, `around_save`, `before_update`, `around_update`, `after_update`, `after_save`, `after_commit/after_rollback`.

  Destroying an Object: `before_destroy`, `around_destroy`, `after_destroy`, `after_commit/after_rollback`.

  You can use `if:` and `on:`, `unless:` conditions while setting callback:

  ```ruby
  before_validation :check_balance, if: :has_card?
  around_create :use_query, if: 'User.online.count > 1000'
  after_save :create_notification, on: :create
  ```

  You can skip callbacks if it is necessary, for example mailing while seeding your DB:

  ```ruby
  User.skip_callback(:save, :after, :send_congratulations_email)
  ```

* [Dirty](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/dirty.rb) - provides a way to track changes in your object in the same way as Active Record does.

  ```ruby
  class Person
    include ActiveModel::Dirty

    define_attribute_methods :name

    def initialize
      @name = nil
    end

    def name
      @name
    end

    def name=(val)
      name_will_change! unless val == @name
      @name = val
    end

    def save
      # do persistence work

      changes_applied
    end

    def reload!
      # get the values from the persistence layer

      clear_changes_information
    end

    def rollback!
      restore_attributes
    end
  end

  # A newly instantiated +Person+ object is unchanged:

    person = Person.new
    person.changed? # => false

  # Change the name:

    person.name = 'Bob'
    person.changed?       # => true
    person.name_changed?  # => true
    person.name_changed?(from: nil, to: "Bob") # => true
    person.name_was       # => nil
    person.name_change    # => [nil, "Bob"]
    person.name = 'Bill'
    person.name_change    # => [nil, "Bill"]

  # Save the changes:

    person.save
    person.changed?      # => false
    person.name_changed? # => false

  # Reset the changes:

    person.previous_changes         # => {"name" => [nil, "Bill"]}
    person.name_previously_changed? # => true
    person.name_previous_change     # => [nil, "Bill"]
    person.reload!
    person.previous_changes         # => {}

  # Rollback the changes:

    person.name = "Uncle Bob"
    person.rollback!
    person.name          # => "Bill"
    person.name_changed? # => false

  # Assigning the same value leaves the attribute unchanged:

    person.name = 'Bill'
    person.name_changed? # => false
    person.name_change   # => nil

  # Which attributes have changed?

    person.name = 'Bob'
    person.changed # => ["name"]
    person.changes # => {"name" => ["Bill", "Bob"]}

  # If an attribute is modified in-place then make use of
  # <tt>[attribute_name]_will_change!</tt> to mark that the attribute is changing.
  # Otherwise \Active \Model can't track changes to in-place attributes. Note
  # that Active Record can detect in-place modifications automatically. You do
  # not need to call <tt>[attribute_name]_will_change!</tt> on Active Record models.

    person.name_will_change!
    person.name_change # => ["Bill", "Bill"]
    person.name << 'y'
    person.name_change # => ["Bill", "Billy"]
  ```

* [Errors](http://api.rubyonrails.org/classes/ActiveModel/Errors.html)

* [Secure Password](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/secure_password.rb)

  ```ruby
    # Bycrypt mast be installed
    # gem 'bcrypt', '~> 3.1.7'

    # Schema: User(name:string, password_digest:string)
    class User < ActiveRecord::Base
      has_secure_password
    end

    user = User.new(name: 'david', password: '', password_confirmation: 'nomatch')
    user.save                                                       # => false, password required
    user.password = 'mUc3m00RsqyRe'
    user.save                                                       # => false, confirmation doesn't match
    user.password_confirmation = 'mUc3m00RsqyRe'
    user.save                                                       # => true
    user.authenticate('notright')                                   # => false
    user.authenticate('mUc3m00RsqyRe')                              # => user
    User.find_by(name: 'david').try(:authenticate, 'notright')      # => false
    User.find_by(name: 'david').try(:authenticate, 'mUc3m00RsqyRe') # => user
  ```

* I18n

* [Types](https://github.com/rails/rails/blob/master/activemodel/lib/active_model/type.rb)

* [Validations](http://guides.rubyonrails.org/active_record_validations.html)

  I prefer sexy validations, google it.

  ```ruby
    validates :name, :presence => true, :uniqueness => true
    validates :terms_of_service, acceptance: true
    validates :email, confirmation: true
    # <%= text_field :person, :email %>
    # <%= text_field :person, :email_confirmation %>
    validates :subdomain, exclusion: { in: %w(www us ca jp), message: "%{value} is reserved." }
    validates :legacy_code, format: { with: /\A[a-zA-Z]+\z/, message: "only allows letters" }
    validates :name, length: { minimum: 2 }
    validates :bio, length: { maximum: 500 }
    validates :password, length: { in: 6..20 }
    validates :registration_number, length: { is: 6 }
    # Numericaly is cool helper for number validation http://edgeguides.rubyonrails.org/active_record_validations.html#numericality
  ```

  Just like callbacks you can use `if:` and `on:`, `unless:` for your validations.

  While callbacks you can add error to make object not valid.

  ```ruby
  class User < ApplicationRecord
    include ActiveModel::Validations

    before_validation :check_stuff

    def check_stuff
      self.errors.add(:field, 'Error message') if self.some_condition
    end
  end
  ```

  You can get errors calling `#errors` on model after falling save. For example when form renders after rollback you can print errors array.

## 2 Active Record - connection of model class and table

ActiveRecord - connects classes to relational database tables to establish an almost zero-configuration persistence layer for applications. Includes associations, aggregations, validations, callbacks, transactions, schema and migrations. ActiveRecord supports adapters for many DataBases: mysql, postgresql, oracle, mongo etc. Interesting post SQLite vs MySQL vs PostgreSQL.

### Contains:

* [Associations](http://guides.rubyonrails.org/association_basics.html)

  Associations are relations beetween models, can be: belongs_to, belongs_to_polymorphic, has_one, has_many, has_one_though, has_many_through, has_and_belongs_to_many.

  Associations provides scopes we could use to get all posts for for user for example.

  For deleting all dependent models use `dependent: :destroy`.

  Sometimes we do not have object we belong to, so our object is invalid. Use 'optional: true' to make association optional.

  Association can be polymorphic, we use it when our model can be associated with different model classes. Classic example for polymorphic association is blog application with posts and comments. Post and comment can be liked by user.

  Sometimes we need custom associations names, so we can use this options: `class_name: `, `foreign_key: `, `primary_key: `. There are many different options for all associations.

* [Base](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/base.rb)

  ActiveRecord accepts hashes as parameters in constructor `user = User.new(name: "David", occupation: "Code Artist")`.

  Conditions builds SQL request string.

  ```ruby
    class User < ActiveRecord::Base
      def self.authenticate_unsafely(user_name, password)
        where("user_name = '#{user_name}' AND password = '#{password}'").first
      end

      def self.authenticate_safely(user_name, password)
        where("user_name = ? AND password = ?", user_name, password).first
      end

      def self.authenticate_safely_simply(user_name, password)
        where(user_name: user_name, password: password).first
      end
    end
  ```

  We can also use ranges and arrays for search

  ```ruby
    Student.where(grade: 9..12)
    Student.where(grade: [9,11,12])
  ```

  Using joins

  ```ruby
    Student.joins(:schools).where(schools: { category: 'public' })
    Student.joins(:schools).where('schools.category' => 'public' )
  ```

  Also we can use find, it will return FIRST element witch find

  ```ruby
    Person.find(user_id)
    Person.find_by(user_name: user_name, password: password)
    Person.find_by_user_name_and_password(user_name, password) # with dynamic finder
  ```

  [Serialize](http://api.rubyonrails.org/classes/ActiveRecord/AttributeMethods/Serialization/ClassMethods.html)- saving arrays, hashes, and other non-mappable objects in text columns

  ```ruby
  class User < ActiveRecord::Base
    serialize :preferences
  end

  user = User.create(preferences: { "background" => "black", "display" => large })
  User.find(user.id).preferences # => { "background" => "black", "display" => large }
  ```

* Enum

  ```ruby
    # Declare an enum attribute where the values map to integers in the database,
    # but can be queried by name. Example:

      class Conversation < ActiveRecord::Base
        enum status: [ :active, :archived ]
      end

      # conversation.update! status: 0
      conversation.active!
      conversation.active? # => true
      conversation.status  # => "active"

      # conversation.update! status: 1
      conversation.archived!
      conversation.archived? # => true
      conversation.status    # => "archived"

      # conversation.status = 1
      conversation.status = "archived"

      conversation.status = nil
      conversation.status.nil? # => true
      conversation.status      # => nil

    # Scopes based on the allowed values of the enum field will be provided
    # as well. With the above example:

      Conversation.active
      Conversation.archived

    # Of course, you can also query them directly if the scopes don't fit your
    # needs:

      Conversation.where(status: [:active, :archived])
      Conversation.where.not(status: :active)

    # You can set the default value from the database declaration, like:

      create_table :conversations do |t|
        t.column :status, :integer, default: 0
      end

    # Good practice is to let the first declared status be the default.

    # Finally, it's also possible to explicitly map the relation between attribute and
    # database integer with a hash:

      class Conversation < ActiveRecord::Base
        enum status: { active: 0, archived: 1 }
      end
  ```

* STI - single table inheritance

* Migrations

* Schema

* SecureToken

* [Transactions](http://api.rubyonrails.org/classes/ActiveRecord/Transactions/ClassMethods.html)

  Transactions are protective blocks where SQL statements are only permanent if they can all succeed as one atomic action. So if program calls rollback, transaction canceles.

  You can call transaction many ways:

  ```ruby
    # Call on ActiveRecord::Base
    ActiveRecord::Base.transaction do
      david.withdrawal(100)
      mary.deposit(100)
    end

    # Call on your Model classes
    Account.transaction do
      balance.save!
      account.save!
    end

    # Call on your objects
    balance.transaction do
      balance.save!
      account.save!
    end
  ```

  Note! Transactions protects only DB, all changes of files would not be rolled back.

  You can user `after_commit` callback to catch transaction falling.
