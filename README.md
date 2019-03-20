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

# ATTRIBUTES

## base\_url

    my $url = $wp->base_url;
    my $wp  = $wp->base_url("https://wordpress.example.com/wp-json");

Holds the base URL to the Wordpress server, including "/wp-json".

## processor

    my $cb = $wp->processor;
    my $wp = $wp->processor(sub { my ($c, $post) = @_ });

A code block that can be used to post process the JSON response from Wordpress.

## ua

    my $ua = $wp->ua;
    my $wp = $wp->ua(Mojo::UserAgent->new);

Holds a [Mojo::UserAgent](https://metacpan.org/pod/Mojo::UserAgent) object that is used to get data from Wordpress.

## yoast\_meta\_key

    my $str = $wp->yaost_meta_key;
    my $wp  = $wp->yaost_meta_key("yoast_meta");

The key in the post JSON response that holds YOAST meta information.

# METHODS

## register

    $wp->register($app, \%config);
    $app->plugin(wordpress => \%config);

Used to register this plugin. Each key in `%config` that matches
["ATTRIBUTES"](#attributes) will be used as an attribute.

# AUTHOR

Jan Henning Thorsen

# COPYRIGHT AND LICENSE

Copyright (C), Jan Henning Thorsen.

This program is free software, you can redistribute it and/or modify it under
the terms of the Artistic License version 2.0.
