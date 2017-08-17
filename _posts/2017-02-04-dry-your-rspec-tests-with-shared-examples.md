---
layout: post
title: Dry your RSpec Tests with Shared Examples
tags: [ruby, rails, rspec, tdd, web development]
permalink: :title
---

Recently, after heavy refactoring in a project I had to spent good amount of time in writing specs. After writing almost similar test cases for some APIs, I thought of finding some solution to get rid of duplication in test cases. After reading articles on best practices and drying up tests, I came to know about shared examples and shared contexts. In my case, I ended up using shared examples and here is what I learned so far.

When you have multiple specs that describes similar behavior, it might be better to extract redundant examples in `shared examples` and use them in multiple specs.

<!--more-->

Suppose you have two models *User* and *Post*, and a user can have many posts. One should be able to view list of users and posts. Creating index action in users and posts controller will serve the purpose.

First write specs for index action of users controller which has responsibility of fetching users and rendering them with proper layout, and then write enough code to make tests pass.

{% highlight ruby %}
# users_controller_spec.rb
describe "GET #index" do
  before do
    5.times do
      FactoryGirl.create(:user)
    end
    get :index
  end
  it {  expect(subject).to respond_with(:ok) }
  it {  expect(subject).to render_template(:index) }
  it {  expect(assigns(:users)).to match(User.all) }
end

# users_controller.rb
class UsersController < ApplicationController
  ....
  def index
    @users = User.all
  end
  ....
end
{% endhighlight %}

Typically index action of any controller fetches and aggregates data from few resources if required, and adds pagination, searching, sorting, filtering or scoping. Finally, all these data is presented to views via html, or JSON or XML in apis. To simplify example, index actions of controllers will just fetch data and show them via views.

Same goes for index action of posts controller:
{% highlight ruby %}
# posts_controller_spec.rb
describe "GET #index" do
  before do
    5.times do
      FactoryGirl.create(:post)
    end
    get :index
  end
  it {  expect(subject).to respond_with(:ok) }
  it {  expect(subject).to render_template(:index) }
  it {  expect(assigns(:posts)).to match(Post.all) }
end

# posts_controller.rb
class PostsController < ApplicationController
  ....
  def index
    @posts = Post.all
  end
  ....
end

{% endhighlight %}
Rspec tests written for both users and posts controller are very similar. In both controllers,

 - Response code -- should be 'ok'
 - Both index action should render proper partial or view - in our case `index`
 - The data to be rendered, such as posts or users

Let's DRY specs of index action by using shared examples.

### Where to place

I like to place shared examples defined inside _specs/support/shared_examples_ directory so that all shared example related files are loaded automatically.

To read about other commonly used conventions to put shared examples from here: [shared examples documentation](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)

### How to define
Index action should respond with 200 success code (ok) and render index template.
{% highlight ruby %}
# specs/support/shared_examples/index_examples.rb
RSpec.shared_examples "index examples" do
  it {	expect(subject).to respond_with(:ok) }
  it {	expect(subject).to render_template(:index) }
end
{% endhighlight %}
Apart from it blocks, before and after hooks, let blocks, context and describe blocks can also be defined inside shared examples. I personally prefer to keep shared examples simple and concise, and don't add contexts and let blocks. Shared examples block also accepts parameters, which I will cover in subsequent sections.

### How to use

Adding `include_examples "index examples"` to users and posts controller specs includes "index examples" to our tests.

{% highlight ruby %}
# users_controller_spec.rb
describe "GET #index" do
  before do
    5.times do
      FactoryGirl.create(:user)
    end
    get :index
  end
  include_examples "index examples"
  it {  expect(assigns(:users)).to match(User.all) }
end

# similarly, in posts_controller_spec.rb
describe "GET #index" do
  before do
    5.times do
      FactoryGirl.create(:post)
    end
    get :index
  end
  include_examples "index examples"
  it {  expect(assigns(:posts)).to match(Post.all) }
end
{% endhighlight %}
You can also use `it_behaves_like` or `it_should_behaves_like` instead of include_examples in this case. `it_behaves_like` and `it_should_behaves_like` are actually aliases and works in same manner, so they can be used interchangeably. However, `include_examples` and `it_behaves_like` are different.

As stated in documentation,

 - include_examples` - includes the examples in the current context
 - `it_behaves_like` and `it_should_behave_like` - includes the examples in nested context

#### How does it make any difference?
RSpec documentation gives proper answer:

> When you include parameterized examples in the current context multiple times, you may override previous method definitions and last declaration wins.

So, when you face situation when parameterized examples contains methods that conflicts with another method in same context, you can replace `include_examples` with `it_behaves_like` method. This will create nested context and avoid this kind of situations.


### How to pass parameters to shared examples

Look at following line in users controller specs,
`it {  expect(assigns(:users)).to match(User.all) }` and  `it {  expect(assigns(:posts)).to match(Post.all) }`
from posts controller.

Now, controller specs can be refactored further by passing parameters to shared example as below:
{% highlight ruby %}
# specs/support/shared_examples/index_examples.rb

# here assigned_resource and resource class are parameters passed to index examples block
RSpec.shared_examples "index examples" do |assigned_resource, resource_class|
  it {	expect(subject).to respond_with(:ok) }
  it {	expect(subject).to render_template(:index) }
  it {  expect(assigns(assigned_resource)).to match(resource_class.all)   }
end
{% endhighlight %}

Now, make following changes in users and posts controller specs.
{% highlight ruby %}
# users_controller_spec.rb
describe "GET #index" do
  before do
    ...
  end
  include_examples "index examples", :users, User.all
end

# posts_controller_spec.rb
describe "GET #index" do
  before do
    ...
  end
  include_examples "index examples", :posts, Post.all
end
{% endhighlight %}
Now controller specs look clean, less redundant and more importantly, DRY. Furthermore, "index examples" can serve as basic structure for designing index action of other controllers.

### Conclusion

By moving common examples into separate file we can eliminate duplication and more importantly, we can improve consistency of our controller actions throughout the application. This is very useful in case of designing APIs, as we can use existing structure of RSpec tests to design tests and create APIs that adhere to common response structure. Mostly, I work with APIs and use shared examples to provide me common structure to design similar APIs.

Feel free to share how you DRY up your specs and	use shared examples.
