---
layout: post
title: "How to embed images in Twig"
---

Recently, I had to create PDF files in a Symfony project using Twig. However,
this is not as easy as it seems; for example, you need a special CSS file
adapted for printing.

Another obstacle is images. I didn't have any problem including images from an
external source, but the problem arose when I had to use images from the same
project. In this post, I explain why using images is a tricky task and how I
solved it.

I used the following stack to generate PDF files:

- [Symfony 6](https://symfony.com/) - Web framework.
- [Twig 3](https://twig.symfony.com/doc/3.x/) - To generate the content of PDF
  files.
- [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) - To convert
  Twig views to PDF files.

This article does not explain how to generate PDF files. I assume you already
know how to use Twig and KnpSnappyBundle.

## Using `assets` to include an image (it didn't work)

The naive solution is to treat a PDF file as any other Twig view and simply use
the `asset` function to include an image.

<!-- {% raw %} -->

```html
<img src="{{ asset('avatar.png') }}"/>
```

<!-- {% endraw %} -->

This didn't work because I was in a Dockerized environment. Outside the
container, the project's URL was something like `http://web.project.localhost`.
However, within the Docker container, this URL does not exist and therefore
means nothing. Thus, the image was never found.

## Using special asset configuration (it didn't work either)

As mentioned before, the URL `http://web.project.localhost` is not resolved when
requested inside the container. So what kind of URL is always valid inside a
container? The answer is a loopback address, which led me to use `127.0.0.1` to
load my images. Therefore, I configured a special _asset package_ in
`framework.yaml`:

```yaml
# config/packages/framework.yaml
assets:
  packages:
    attachments-pdf:
      base_urls: 'http://127.0.0.1/attachments'
```

To generate an absolute URL using the `attachments-pdf` package:

<!-- {% raw %} -->

```html
<img src="{{ asset('avatar.png', 'attachments-pdf') }}"/>
```

<!-- {% endraw %} -->

This would have worked except that I needed to be **logged in** to the website.
The image was never displayed because, instead of an image, I received the login
page.

## Embedding images in Twig

Finally, I considered embedding images directly inside a PDF file. I wasn't very
enthusiastic about this solution because it's not a _drop-in solution_; it
requires some configuration to work properly.

To embed an image within any HTML file, you can use **DATA URIs** instead of an
image URL. DATA URIs follow this syntax:

> `data:<mimetype>;<encoding>,<data>`

Here is an example taken
from [Wikipedia](https://en.wikipedia.org/wiki/Data_URI_scheme#HTML):

```HTML
<img src="data:image/png;base64,iVBORw0KGgoAAA
ANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU
5ErkJggg==" alt="Red dot"/>
```

Luckily for me, somebody already created the [
`data_uri`](https://twig.symfony.com/doc/3.x/filters/data_uri.html) filter to
create DATA URIs. This filter is not installed by default. The following
packages are required: `twig/html-extra` and `twig/extra-bundle`.

```console
composer require twig/html-extra twig/extra-bundle
```

To avoid any problems with paths, I decided to use absolute paths. Therefore, I
configured this in `framework.yaml`. I used `base_path` instead of `base_urls`:

```diff
# config/packages/framework.yaml
assets:
  packages:
    attachments-pdf:
-      base_urls: 'http://127.0.0.1/attachments'
+      base_path: '/app/assets/attachments'
```

Next, I needed a way to read the image content. This can be easily done in PHP
with `file_get_contents`. However, since Twig has no similar functionality, I
had to write my own `file_get_contents` Twig filter. You can use
`make:twig-extension` to create a Twig extension, which will contain our custom
filter.

```console
bin/console make:twig-extension FileExtension
```

Then, we create the `file_get_contents` Twig filter. As you can see, our Twig
function calls PHP's `file_get_contents` function.

```php
<?php // src/Twig/FileExtension.php

namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;
use Twig\TwigFunction;
use function file_get_contents;

class FileExtension extends AbstractExtension
{
  public function getFilters(): array
  {
    return [
      new TwigFunction('file_get_contents', file_get_contents(...)),
    ];
  }
}
```

Finally, putting everything together:

<!-- {% raw %} -->

```html
<img
  src="{{ asset('avatar.png', 'attachments-pdf') | file_get_contents | data_uri }}"/>
```

<!-- {% endraw %} -->

How does it work?

1. The `asset('avatar.png', 'attachments-pdf')` function generates an absolute
   path. In our example, this will be `/app/assets/attachments/avatar.png`.
   Remember, we configured this in `framework.yaml`.
2. The `file_get_contents` filter then receives the file path and returns the
   content of `avatar.png`.
3. Finally, we pass the image content to the `data_uri` filter. This filter is
   very handy because it does all the hard work for us: generates valid DATA URI
   syntax, detects the MIME type, and converts the image to base64.

## Conclusion

I explained my journey trying to display images in a PDF file. I tested many
solutions and decided to use DATA URIs.

To use DATA URIs, I had to:

1. Install the `data_uri` Twig filter.
2. Configure _assets packages_ in `framework.yaml`.
3. Create a `file_get_contents` Twig filter.

I think using DATA URIs is the best solution when you are creating PDF files.
This allows you to add images in any environment, dockerized or not.
