# Active Admin Editor
[![Build Status](https://travis-ci.org/ejholmes/active_admin_editor.png?branch=master)](https://travis-ci.org/ejholmes/active_admin_editor) [![Code Climate](https://codeclimate.com/github/ejholmes/active_admin_editor.png)](https://codeclimate.com/github/ejholmes/active_admin_editor)

This is a wysiwyg html editor for the [Active Admin](http://activeadmin.info/)
interface using [wysihtml5](https://github.com/xing/wysihtml5).

![screenshot](https://dl.dropbox.com/u/1906634/Captured/OhvTv.png)

[Demo](http://active-admin-editor-demo.herokuapp.com/admin/pages/new)

## Installation

Add the following to your `Gemfile`:

```ruby
gem 'active_admin_editor'
```

And include the wysiwyg styles in your `application.css`:

```scss
//= require active_admin/editor/wysiwyg
```

Then run the following to install the default intializer:

```bash
$ rails g active_admin:editor
```

## Usage

This gem provides you with a custom formtastic input called `:html_editor` to build out a wysihtml5 enabled input.
All you have to do is specify the `:as` option for your inputs.

**Example**

```ruby
ActiveAdmin.register Page do
  form do |f|
    f.inputs do
      f.input :title
      f.input :content, as: :html_editor
    end

    f.buttons
  end
end
```

## Uploads

The editor supports uploading of assets directly to an S3 bucket. Edit the initializer that
was installed when you ran `rails g active_admin:editor`.

```ruby
ActiveAdmin::Editor.configure do |config|
  config.s3_bucket = '<your bucket>'
  config.aws_access_key_id = '<your aws access key>'
  config.aws_access_secret = '<your aws secret>'
  # config.storage_dir = 'uploads'
end
```

Then add the following CORS configuration to the S3 bucket:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<CORSConfiguration xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <CORSRule>
        <AllowedOrigin>*</AllowedOrigin>
        <AllowedMethod>PUT</AllowedMethod>
        <AllowedMethod>POST</AllowedMethod>
        <AllowedMethod>GET</AllowedMethod>
        <AllowedMethod>HEAD</AllowedMethod>
        <MaxAgeSeconds>3000</MaxAgeSeconds>
        <ExposeHeader>Location</ExposeHeader>
        <AllowedHeader>*</AllowedHeader>
    </CORSRule>
</CORSConfiguration>
```

## Uploads with custom uploader action

You also can upload images via a own provided action. For example when you want to preprocess the images. For this you have to set the uploader_action_path in the initializer. When you have set the S3 config and the uploader_action_path, the uploader_action_path will win.

```ruby
ActiveAdmin::Editor.configure do |config|
  config.uploader_action_path = '/path/to/the/uploader'
end
```

__pseudocode of an uploader action within active admin__

```ruby
collection_action :upload_image, :method => :post do
  img = ImageUploader.new(image: params[:file])
  img.upload

  # IMPORTANT the image url must be set as the headers location porperty
  render json: {location: img.remote_url} , location: img.remote_url
end
```


## Configuration

You can configure the editor in the initializer installed with `rails g
active_admin:editor` or you can configure the editor during
`ActiveAdmin.setup`:

```ruby
ActiveAdmin.setup do |config|
  # ...
  config.editor.aws_access_key_id = '<your aws access key>'
  config.editor.s3_bucket = 'bucket'
end
```

## Parser Rules

[Parser rules](https://github.com/xing/wysihtml5/tree/master/parser_rules) can
be configured through the initializer:

```ruby
ActiveAdmin::Editor.configure do |config|
  config.parser_rules['tags']['strike'] = {
    'remove' => 0
  }
end
```

Be sure to clear your rails cache after changing the config:

```bash
rm -rf tmp/cache
```

## Custom template paths
The toolbar and the uploader are client side templates (ejs)
To customize these you can set custom paths your very own templates.
__Its important that the template has a different name then (toolbar or uploader)!__

```ruby
ActiveAdmin::Editor.configure do |config|
  config.template_paths = {
      toolbar: 'path to my custom toolbar template',
      uploader: 'path to my custom uploader template'
  }
end
```
__Please check the original templates for an example__
app/assets/javascripts/active_admin/editor/templates

### Make your templates available
Unfortunately sproket can not find templates from your
asset folder when requireing them in a gem. So you have to
make your templates manually avaliable.

```javascript
//= require_tree ./path_to_your_templates
```

## Heroku

Since some of the javascript files need to be compiled with access to the env
vars, it's recommended that you add the [user-env-compile](https://devcenter.heroku.com/articles/labs-user-env-compile)
labs feature to your app.

1. Tell your rails app to run initializers on asset compilation

   ```ruby
   # config/environments/production.rb
   config.initialize_on_precompile = true
   ```
2. Add the labs feature

   ```bash
   heroku labs:enable user-env-compile -a myapp
   ```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Added some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

### Tests

Run RSpec tests with `bundle exec rake`. Run JavaScript specs with `bundle
exec rake konacha:serve`.
