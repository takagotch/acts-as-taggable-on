### acts-as-taggable-on
---

https://github.com/mbleigh/acts-as-taggable-on

```sh
gem 'acts-as-taggable-on', '~> 6.0'
bundle
rake acts_as_taggable_on_engine:install:migrations
rake db:migrate

rale acts_as_taggable_on_engine:tag_names:collate_bin
```

```ruby
ActsAsTaggableOn.force_binary_collation = true

class User < ActiveRecord::Base
  acts_as_taggable
  acts_as_taggable_on :skills, :interests
end
class UsersContoller < ApplicationController
  def user_params
    params.require(:user).permit(:name, :tag_list)
  end
end
@user = User.new(:name => "Tky")

@user.tag_list.add("awesome")
@user.tag_list.remove("awesome")
@user.save

@user.tag_list.add("awesome", "slick")
@user.tag_list.remove("awesome", "slick")
@user.save

@user.tag_list.add("awesome slick", parse: true)
@user.tag_list.remove("awesome", parse: true)

@user.tag_list = "awesome, slick, hefty"
@user.save
@user.reload
@user.tags
# => [#<ActsAsTaggableOn::Tag_id: 1, name: "awesome", taggings_count: 1>,
      #<ActsAsTaggableOn::Tag id: 2, name: "slick", taggings_count: 1>,
      #<ActsAsTaggableOn::Tag id: 3, name: "hefty", taggings_count: 1>]

@user.skill_list = "joking, clowning, boxing"
@user.save
@user.reload
@user.skills
# => []

@user.skill_list.add("coding")

@user.skill_list
# => ["joking", "clowning", "boxing", "coding"]

@another_user = User.new(:name => "Alice")
@another_user.skill_list.add("clowning")
@another_user.save

User.skill_counts
# => []

class User < ActiveRecord::Base
  acts_as_ordered_taggable
  acts_as_ordered_taggable_on :skills, :interests
end

@user = User.new(:name => "Tky")
@user.tag_list = "east, south"
@user.save

@user.tag_list = "north, east, south, west"
@user.save

@user.reload
@user.tag_list # => ["north", "east", "south", "west"]

ActsAsTaggableOn::Tag.most_used
ActsAsTaggableOn::Tag.least_used

ActsAsTaggableOn::Tag.most_used(10)
ActsAsTaggableOn::Tag.least_used(10)

class User < ActiveRecord::Base
  acts_as_taggable_on :tags, :skills
  scope :by_join_date, order("created_at DESC")
end
User.tagged_with().by_join_date
User.tagged_with().by_join_date.paginate(:page => params[:page], :per_page => 20)
User.tagged_with([], :match_all => true)
User.tagged_with([], :any => true)
User.tagged_with([], :exclude => true)
User.tagged_with(['awesome', 'cool'], :on => tags, :any => true).tagged_with(['smart', 'shy'], :on => :skills, :any => true)

@bobby = User.find_by_name("Bobby")
@bobby.skill_list # => ["jogging", "diving"]
@frankie = User.find_by_name("Frankie")
@frankie.skill_list # => ["hacking"]
@tom = User.find_by_name("Tom")
@tom.skill_list # => ["hacking", "jogging", "diving"]
@tom.find_related_skills # => [<User name="Bobby">, <User name="Frankie">]
@bobby.find_related_skills # => [<User name="Tom">]
@frankie.find_related_skills # => [<User name="Tom">]


@user = User.new(:name => "Bobby")
@user.set_tag_list_on(:customs, "same, as, tag, list")
@user.tag_list_on(:customs) # => ["same", "as", "tag", "list"]
@user.save
@user.tags_on(:customs) # => [<Tag name='same'>, ...]
@user.tag_counts_on(:customs)
User.tagged_with("same", :on => :customs) # => [@user]

class MyParser < ActsAsTaggableOn::GenericParser
  def parse
    ActsAsTaggableOn::TagList.new.tap do |tag_list|
      tag_list.add @tag_list.split('|')
    end
  end
end

@user = User.new(:name => "Boddy")
@user.tag_list = "east, south"
@user.tag_list.add("north|west", parser: MyParser)
@user.tag_list # => ["north", "east", "south", "west"]

@user.tag_list_parser = MyParser
@user.tag_list.add("north|west")
@user.tag # => ["north", "east", "south", "west"]

ActsAsTaggableOn.default_parser = MyParser
@user = Usernew(:name => "Tky")
@user.tag_list = "east|south"
@user.tag_list # => ["east", "south"]

class User < ActiveRecord::Base
  acts_as_tagger
end
class Photo < ActiveRecord::Base
  acts_as_taggable_on :locations
end
@some_user.tag(@some_photo, :with => "paris, normandy", :on => :locations)
@some_user.owned_taggings
@some_user.owned_tags
Photo.tagged_with("paris", :on => :locations, :owned_by => @some_user)
@some_photo.locations_from(@some_user) # => ["paris", "normandy"]
@some_photo.owner_tags_on(@some_user, :location) # => [#<ActsAsTaggableOn::Tag id: 1, name: "paris">...]
@some_photo.owner_tags_on(nil, :locations) # => ...
@some_user.tag(@some_photo, :with => "paris, normandy", :on => :locations, :skip_save => true) # => ...@some_photo

@some_photo.tag_list # => []

def add_owned_tag
  @some_item = Item.find(params[:id])
  owned_tag_list = @some_item.all_tags_list - @some_item.tag_list
  owned_tag_list += [(params[:tag])]
  @tag_owner.tag(@some_item, :with => stringfy(owned_tag_list), :on => :tags)
  @some_item.save
end
def stringify(tag_list)
  tag_list.inject('') { |memo, tag| memo += () }[0..-1]
end

def remove_owned_tag
  @some_item = Item.find(params[:id])
  owned_tag_list = @some_item.all_tags_list - @some_item.tag_list
  owned_tag_list -= [(params[:tag])]
  @tag_owner.tag(@some_item, :with => stringfy(owned_tag_list), :on => :tags)
  @some_item.save
end

@bobby = User.find_by_name("Tky")
@bobby.skill_list # => ["jogging", "diving"]

@bobby.skill_list_changed? # => false
@bobby.changes # => {}

@bobby.skill_list = "swimming"
@bobby.changes.should == {"skill_list"=>["jogging, diving", ["swimming"]]}
@bobby.skill_list_changed? # => true

@bobby.skill_list_change.should == ["jogging, diving", ["swimming"]]


User.find(:first).posts.tag_counts_on(:tags)

module PostHelper
  include ActsAsTaggableOn::TagsHelper
end

class PostController < ApplicationController
  def tag_cloud
    @tags = Post.tag_counts_on(:tags)
  end
end

<% tag_cloud(@tags, %w(css1 css2 css3 css4)) do |tag, css_class| %>
  <%= link_to tag.name, { :action => :tag, :id => tag.name }, :class => css_class %>
<% end %>

.css1 { font-size: 1.0em; }
.css2 { font-size: 1.2em; }
.css3 { font-size: 1.4em; }
.css4 { font-size: 1.6em; }

ActsAsTaggableOn.remove_unused_tags = true
ActsAsTaggableOn.force_lowercase = true
ActsAsTaggableOn.force_parameterize = true
ActsAsTaggableOn.strict_case_match = true
ActsAsTaggableOn.strict_case_match = true
ActsAsTaggableOn.force_binary_collation = true
ActsAsTaggableOn.tags_table = 'aato_tags'
ActsAsTaggableOn.taggings_table = 'aato_taggings'
ActsAsTaggableOn.delimiter = ','


USE my_wonderful_app_db;
ALTER TABLE tags MODIFY name VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_bin;

bundle
rake spec

```

