# NAME

Mojolicious::Plugin::Wordpress - Use Wordpress as a headless CMS

# SYNOPSIS

    use Mojolicious::Lite;
    plugin wordpress => {base_url => "https://wordpress.example.com/wp-json"};

    get "/page/:slug" => sub {
      my $c = shift->render_later;
      $c->wp->get_page_p($c->stash("slug"))->then(sub {
        my $page = shift;
        $c->render(json => $page);
      });
    };

# DESCRIPTION

[Mojolicious::Plugin::Wordpress](https://metacpan.org/pod/Mojolicious::Plugin::Wordpress) is a plugin for getting data using the
Wordpress JSON API.

This plugin is currently EXPERIMENTAL. Let me know if you have any feedback at
[https://github.com/jhthorsen/mojolicious-plugin-wordpress/issues](https://github.com/jhthorsen/mojolicious-plugin-wordpress/issues).

# HELPERS

## get\_post\_p

    my $promise = $c->wp->get_post_p;
    my $promise = $c->wp->get_post_p($slug);
    my $promise = $c->wp->get_post_p(\%query);

This helper will be available, dependent on what you set ["post\_types"](#post_types) to. It
will return a [Mojo::Promise](https://metacpan.org/pod/Mojo::Promise) that will get a `$post` hash-ref or `undef` in
the fullfillment callback. The `$post` hash-ref will be exactly what was
returned through the API from Wordpress, or whatever the ["post\_processor"](#post_processor) has
changed it to.

## get\_posts\_p

    my $promise = $c->wp->get_posts_p;
    my $promise = $c->wp->get_posts_p(\%query);
    my $promise = $c->wp->get_posts_p({all => 1, post_processor => sub { ... }});

This helper will be available, dependent on what you set ["post\_types"](#post_types) to. It
will return a [Mojo::Promise](https://metacpan.org/pod/Mojo::Promise) that will get an array-ref of `$post` hash refs
in the fullfillment callback. A `$post` hash-ref will be exactly what was
returned through the API from Wordpress, or whatever the ["post\_processor"](#post_processor) has
changed it to.

## meta\_from

    my $meta = $c->wp->meta_from(\%post);

This helper will extract meta information from the Wordpress post and return a
`%hash` that looks something like this:

    {
      wp_canonical             => "",
      wp_title                 => "",
      wp_metadesc              => "",
      wp_description           => "",
      wp_opengraph_title       => "",
      wp_opengraph_description => "",
      wp_twitter_title         => "",
      wp_twitter_description   => "",
      ...
    }

Note that some keys might be missing or some keys might be added, depending on
how the Wordpress server has been set up.

Suggested Wordpress plugins: [https://wordpress.org/plugins/wordpress-seo/](https://wordpress.org/plugins/wordpress-seo/)
and [https://github.com/jhthorsen/wp-api-yoast-meta](https://github.com/jhthorsen/wp-api-yoast-meta).

## rewrite\_content

    $dom = $c->wp->rewrite_content($post->{content}{rendered});

This helper will rewrite a piece of HTML from Wordpress with the following
rules:

- Asset URL

    Will replace "src" and "srcset" on images with an URL to the local application,
    if ["base\_assets\_url"](#base_assets_url) is set in config or a "wp.assets" route could be found.

- Images

    All "height" and "width" attributes will be removed from "img" tags.

Note that more rules might be added in the future.

# ATTRIBUTES

## base\_url

    my $url = $wp->base_url;
    my $wp  = $wp->base_url("https://wordpress.example.com/wp-json");

Holds the base URL to the Wordpress server API, including "/wp-json".

## meta\_replacer

    my $cb = $wp->meta_replacer;
    my $wp = $wp->meta_replacer(sub { my ($c, $str) = @_ });

A callback used to search and replace meta data when calling ["meta\_from"](#meta_from).
The default callback will search and replace all occurances of "%%some\_key%%"
with `wp_$some_key` from ["stash" in Mojolicious::Controller](https://metacpan.org/pod/Mojolicious::Controller#stash).

## post\_processor

    my $cb = $wp->post_processor;
    my $wp = $wp->post_processor(sub { my ($c, $post) = @_ });

A code block that can be used to post process the JSON response from Wordpress.

## ua

    my $ua = $wp->ua;
    my $wp = $wp->ua(Mojo::UserAgent->new);

Holds a [Mojo::UserAgent](https://metacpan.org/pod/Mojo::UserAgent) object that is used to get data from Wordpress.

## yoast\_meta\_key

    my $str = $wp->yaost_meta_key;
    my $wp  = $wp->yaost_meta_key("yoast");

The key in the post JSON response that holds
[Yoast](https://wordpress.org/plugins/wordpress-seo/) meta information.

This information is not enabled by default. To enable it through the API, you
can add this plugin: [https://github.com/jhthorsen/wp-api-yoast-meta](https://github.com/jhthorsen/wp-api-yoast-meta).

# METHODS

## register

    $wp->register($app, \%config);
    $app->plugin(wordpress => \%config);

Used to register this plugin. `%config` can have:

- base\_assets\_url

    If `base_assets_url` is set, then a new route will be added to your `$app`,
    that will proxy GET requests to your Wordpress backend. This can be useful,
    if you want to mask/hide Wordpress URLs and rather let everything go through
    your [Mojolicious](https://metacpan.org/pod/Mojolicious) application.

    Example value:

        {base_assets_url => 'https://wordpress.com/wp-content/uploads'}

    The path added will either be defined by the `base_assets_route` config
    variable or default to "/uploads/\*proxy\_path", and the route will be named
    "wp.assets".

- base\_url

    See ["base\_url"](#base_url).

- post\_processor

    See ["post\_processor"](#post_processor).

- post\_types

    A list of post types available in the CMS. Defaults to:

        ["pages", "posts"]

    This list will generate helpers to fetch data from Wordpress. Example default
    helpers are:

        my $p = $c->wp->get_page_p(...);
        my $p = $c->wp->get_pages_p(...);
        my $p = $c->wp->get_post_p(...);
        my $p = $c->wp->get_posts_p(...);

    See ["get\_post\_p"](#get_post_p) and ["get\_posts\_p"](#get_posts_p) for more information.

    Suggested Wordpress plugin:
    [https://wordpress.org/plugins/custom-post-type-maker/](https://wordpress.org/plugins/custom-post-type-maker/)

- prefix

    The prefix for the helpers. Defaults to "wp".

- ua

    See ["ua"](#ua).

- yoast\_meta\_key

    See ["yoast\_meta\_key"](#yoast_meta_key).

# AUTHOR

Jan Henning Thorsen

# COPYRIGHT AND LICENSE

Copyright (C), Jan Henning Thorsen.

This program is free software, you can redistribute it and/or modify it under
the terms of the Artistic License version 2.0.
