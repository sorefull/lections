# Lection 1 (factories, tests, jobs)
---

## 1 Factories ([factory_girl gem](https://github.com/thoughtbot/factory_girl)) - build strategy for model testing

### Install:

* Add `gem 'factory_girl'` to Gemfile and install dependencies `bundle install`

### Using:

Defining factory:

  ```ruby
  # /spec/factories/user.rb
  FactoryGirl.define do
    factory :user do
      # we can use Faker gem for content
      email     { Faker::Internet.email }
      password  { Faker::Internet.password }
    end
  end
  ```

Defining factory with nested factory:

  ```ruby
  # /spec/factories/post.rb
  FactoryGirl.define do
    factory :post do
      title { Faker::Lorem.words(3).join(' ') }
      body  { Faker::Lorem.sentences(5).join(' ') }
      # use sequence for completely unique name
      sequence :slug do |number_of_post|
        number_of_post.to_s + title
      end
      # use associations for nested creation
      user
    end
  end
  ```

Using factories in code:

  ```ruby
  # It will create user with id = 1
  let(:user) { create(:user) }
  # It will create post and set user_id = 1
  let(:post_1) { create(:post, user: user) }
  # It will create post and nested user, posts user_id = 2
  let(:post_2) { create(:post) }
  ```

## 2 Rspec ([rspec-rails gem](https://github.com/rspec/rspec-rails)) - gem for testing. Supports tests for models, controllers, views. Actually first two are used.

### Install & Setup:

Full instruction you can find on official github page.

* Add `gem 'rspec-rails', '~> 3.5'` to Gemfile and install dependencies `bundle install`

* Install gem `rails generate rspec:install`. It will generate configuration files.

* Run `bundle exec rspec` for your first test run

### Model testing:

Model testing is useful for checking validation, callbacks correct working. Useful gem for testing validation is [shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers).

We can use before and after like in controllers and models.

  ```ruby
  # /spec/models/post_spec.rb
  require "rails_helper"

  RSpec.describe Post, :type => :model do
    # shoulda-matchers validation testing
    it { should validate_presence_of(:title) }
    it { should validate_presence_of(:body) }
    it { should validate_presence_of(:slug) }
    it { should belong_to(:user) }

    describe 'posts' do
      before :all do
        # do some stuff before all tests
      end
      before :each do
        # do some stuff before each test
      end
      after :each do
        # do some stuff after each test
      end
      after :all do
        # do some stuff after all tests
      end

      let(:post) { create(:post) }
      it "some description" do
        # expect(post.value).to be_truthy
        # expect(post.value).to be_falsey
        # expect(post.value).to be_empty
        # expect(post.value).to eq another_object
        # expect(post.value).to eq 1
        # expect(another_object.value).to eq 1
        # expect(another_object.value).to be_truthy
      end
    end
  end
  ```

### Controller testing:

We are testing controllers behavior, requests statuses, models state and quantity.

  ```ruby
  # /spec/controllers/posts_controller_spec.rb
  require 'rails_helper'

  RSpec.describe PostsController, type: :controller do
    render_views

    let(:user) { create(:user) }

    # we can use before&after file in models tests
    before :each do
      sign_in user
    end

    describe '#index' do
      let(:post) { create(:post, user: user) }
      # many requests has different codes, for example we can get 401 if unauthorized
      it 'returns status 200' do
        # method #get => GET request
        get :index
        expect(response.status).to eq(200)
      end
    end

    describe '#show' do
      let(:post) { create(:post, user: user) }
      it 'returns status 200' do
        # we can build params of request
        get :show, params: { id: post }
        expect(response.status).to eq(200)
      end
    end

    describe '#new' do
      it 'returns status 200' do
        get :new
        expect(response.status).to eq(200)
      end
    end

    describe '#create' do
      # haven't tried yet, but this mast work
      # it builds object instead of creation in db
      let(:post) { build_stubbed(:post, user: user) }
      it 'creates new post' do
        post_attributes = { title: post.title, body: post.body, slug: post.slug }
        # method #post => POST request
        post :create, params: { post: post_attributes }
        # we also can expect everytrithing like we did in model tests
        expect(Post.count).to eq(1)
        new_post = Post.first
        expect(new_post.user).to eq(user)
        expect(response.status).to eq(204)
      end
    end

    describe '#destroy' do
      let(:post) { create(:post, user: user) }
      it 'deletes post' do
        # method #delete => DELETE request
        delete :destroy, params: { id: post }
        expect(Post.count).to eq(0)
        expect(response.status).to eq(204)
      end
    end
  end
  ```

When we create objects by let it doesn't creates in db immediately, it creates first time we call it. So after calling `let(:user) { create(:user) }` `User.first` will return 0. We can resolve this by calling `let!()`, this will create record immediately.

## 3 [Active Job](http://edgeguides.rubyonrails.org/active_job_basics.html) - Make work happen later

![later](https://68.media.tumblr.com/e643f3510f4a5e81510ddb36a1a3c359/tumblr_nkgo8l5tC61sk1rjvo1_500.jpg)

Active job is build-in framework for scheduling tasks and jobs in rails project.

### Example:

* First thing we need is to generate job

  ```
  rails g job generate_post
  ```

* Set job

  ```ruby
  class GeneratePostJob < ApplicationJob
    queue_as :default

    def perform(*args)
      Post.create(title: Faker::Lorem.words(2).join(' '), body: Faker::Lorem.sentences(4).join(' '))
    end
  end

  ```

* Initialize request

  ```ruby
  GeneratePostJob.set(wait: 1.minute).perform_later
  ```

* Run server and see in one minute this:

  ```sql
  Performing GeneratePostJob from Async(default)
     (0.1ms)  begin transaction
    SQL (0.3ms)  INSERT INTO "posts" ("title", "body", "created_at", "updated_at") VALUES (?, ?, ?, ?)  [["title", "error quos"], ["body", "Quibusdam sequi enim ut distinctio. Ducimus dolores in unde. Unde maxime harum vel saepe iste voluptates consequatur. Dolore eos libero in et."], ["created_at", 2017-04-21 14:44:04 UTC], ["updated_at", 2017-04-21 14:44:04 UTC]]
     (11.0ms)  commit transaction
  Performed GeneratePostJob from Async(default) in 27.72ms

  ```

### Some adapters:

[Backburner](https://github.com/nesquena/backburner), [Delayed Job](https://github.com/collectiveidea/delayed_job), [Qu](https://github.com/chanks/que), [Que](https://github.com/chanks/que), [queue_classic](https://github.com/QueueClassic/queue_classic), [Resque 1.x](https://github.com/resque/resque/tree/1-x-stable), [Sidekiq](http://sidekiq.org/), [Sneakers](https://github.com/jondot/sneakers), [Sucker Punch](https://github.com/brandonhilkert/sucker_punch), [Active Job Async Job](http://api.rubyonrails.org/classes/ActiveJob/QueueAdapters/AsyncAdapter.html), [Active Job Inline](http://api.rubyonrails.org/classes/ActiveJob/QueueAdapters/InlineAdapter.html).

```
|                   | Async | Queues | Delayed    | Priorities | Timeout | Retries |
|-------------------|-------|--------|------------|------------|---------|---------|
| Backburner        | Yes   | Yes    | Yes        | Yes        | Job     | Global  |
| Delayed Job       | Yes   | Yes    | Yes        | Job        | Global  | Global  |
| Qu                | Yes   | Yes    | No         | No         | No      | Global  |
| Que               | Yes   | Yes    | Yes        | Job        | No      | Job     |
| queue_classic     | Yes   | Yes    | Yes*       | No         | No      | No      |
| Resque            | Yes   | Yes    | Yes (Gem)  | Queue      | Global  | Yes     |
| Sidekiq           | Yes   | Yes    | Yes        | Queue      | No      | Job     |
| Sneakers          | Yes   | Yes    | No         | Queue      | Queue   | No      |
| Sucker Punch      | Yes   | Yes    | Yes        | No         | No      | No      |
| Active Job Async  | Yes   | Yes    | Yes        | No         | No      | No      |
| Active Job Inline | No    | Yes    | N/A        | N/A        | N/A     | N/A     |
```

### Callbacks:

* before_enqueue
* around_enqueue
* after_enqueue
* before_perform
* around_perform
* after_perform
