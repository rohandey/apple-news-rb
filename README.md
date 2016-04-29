# Apple News

A fully featured gem for building Apple News documents and interfacing with the Apple News API.

**NOTE:** this is very much a work in progress. There are a lot of things that don't work yet, and some things will probbaly change (even drastically), but the basics are here. You can create and fetch articles, and fetch information about channels and sections.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'apple-news'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install apple-news

## Configuration

In order to work with the Apple News API, we have to set a couple of configuration params that are available from [News Publisher](https://www.icloud.com/#newspublisher).

``` ruby
AppleNews.config.channel_id = "63aFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF"
AppleNews.config.api_key_id = "379FFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF"
AppleNews.config.api_key_secret = "miJAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="
```

## Fetching Data

### Channels

You can fetch the Channel information by calling:

``` ruby
channel = AppleNews::Channel.current
```

Because each channel has a separate API key, there is only ever one channel that you can fetch.

### Sections

You can either fetch sections by their ID, or by fetching all of them through the Channel object.

``` ruby
section = AppleNews::Section.new(section_id)

# or
channel = AppleNews::Channel.current
channel.default_section # for the default section
channel.sections # for all the sections
```

### Articles

You can fetch articles either by their ID, their channel, or their section.

``` ruby
article = AppleNews::Article.new(article_id)

# or
channel = AppleNews::Channel.current
channel.articles # all articles in the channel
channel.default_section.articles # all articles in the default section
```

## Submitting Articles

Apple News articles are submitted as "bundles", with the article content and attached files together in one request. Because of this, we have the concept of a Document. The Document is what contains the actual article content in the Apple News JSON format. The Article encapsulates this and also includes all of the files that will be submitted along with the document.

### Building Documents

Documents are built using the `AppleNews::Document` class. All documents must have an identifier (a string generated by you to determine article uniqueness, not the ID returned from the Apple News API), title, layout, components, and a default component text style.

In order to maintain idiomatic Ruby, all properties are accessed/set via the underscore version of the actual Apple News API property. For example, `componentTextStyles` becomes `component_text_styles`.

``` ruby
document = AppleNews::Document.new
document.identifier = "1234"
document.title = "Test Article"
document.layout = AppleNews::Layout.new(columns: 1, width: 1024)
document.component_text_styles[:default] = AppleNews::Style::ComponentText.new(
  font_name: 'Georgia',
  font_size: 14,
  text_color: '#000000'
)

document.components << AppleNews::Component::Heading.new(text: "Test Article")
document.components << AppleNews::Component::Body.new(text: "Just testing out this Ruby gem!")
```

Every component, style, property, etc. as defined by the [API documentation](https://developer.apple.com/library/ios/documentation/General/Conceptual/Apple_News_Format_Ref/index.html) has its own class. Each property can be set either by the constructor, or by calling the accessor method on the object. For example:

``` ruby
AppleNews::Component::Instagram.new(url: 'https://www.instagram.com/p/BB7mr0hsS4U/')

# or

component = AppleNews::Component::Instagram.new
component.url = "https://www.instagram.com/p/BB7mr0hsS4U/"
```

### Creating an Article

An article must have a document. Once it's created, you can set metadata flags and add files to the article bundle.

``` ruby
article = AppleNews::Article.new(nil, document: document)
article.is_preview = true

# There are 3 different ways you can add a file to the document
article.add_file(File.new("/path/to/image.jpg"))
article.add_file_at_path("/path/to/image.jpg")
article.add_string_as_file("image.jpg", image_contents, "image/jpeg")
```

### Saving a Document

Once you have your document built, you can submit it to the API.

```
article.save!
```

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake spec` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/hodinkee/apple-news-rb.

