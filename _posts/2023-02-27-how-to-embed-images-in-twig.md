---
layout: post
title: "How to embed images in PDF files using Twig"
---

Recently I had to create PDF files in a Symfony project I'm working on.
This is the stack I use to generate PDF files:

- [Symfony 6](https://symfony.com/) - Web framework.
- [Twig 3](https://twig.symfony.com/doc/3.x/) - To generate the content of PDF
  files.
- [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) - To convert
  Twig view to PDF file.
  PDF files

Everything went well until I had to include images, this sounds like
a really simple task but I was

## Using `assets` to include an image (didn't work)

The naive solution is treat a PDF file as any other Twig view, simply
use `assets` function to include an image.

<!-- {% raw %} -->

```html
<img src="{{ asset('avatar.png') }}"/>
```

<!-- {% endraw %} -->

This didn't work because I was in a Dockerized environment, outside the
container the project's url was something
like `http://web.project.localhost`, but within the Docker container this URL
does not exist and means nothing so the image was never displayed.

## Using special asset configuration (didn't work)

As told before, the url `http://web.project.localhost` is not resolved when
requested inside the container, so what kind of URL is always valid inside a
container ? That's how I had the idea to use `127.0.0.1` to load my images, so I
had to configure a special _asset package_ in `framework.yaml` file.

```yaml
# ...
assets:
  packages:
    attachments-pdf:
      base_urls: 'http://127.0.0.1/attachments'
# ...
``` 

To generate an absolute url using `attachments-pdf` package:

<!-- {% raw %} -->

```html
<img src="{{ asset('avatar.png', 'attachments-pdf') }}"/>
```

<!-- {% endraw %} -->

This would have worked except that in order to view an image I needed to be
<u>logged in</u> to the website, image was never displayed because instead of
an image I was receiving the login page.

## Embedding images in Twig

So finally I came up to the idea to embed images directly inside PDF file, I
wasn't so keen about this solution because it's not a _drop-in solution_, it
requires some configuration to work properly.

To embed an image within any HTML file you can use DATA URIs instead of an image
url, this is the syntax that DATA URI follows:

> `data:<mimetype>;<encoding>,<data>`

Here an example taken
from [Wikipedia](https://en.wikipedia.org/wiki/Data_URI_scheme#HTML) to embed an
image in HTML:

```HTML
<img src="data:image/png;base64,iVBORw0KGgoAAA
ANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU
5ErkJggg==" alt="Red dot"/>
```

Luckily for me, somebody already created a
the [`data_uri`](https://twig.symfony.com/doc/3.x/filters/data_uri.html) filter
to create DATA URIs, this filter is not installed by default, so you have to
install `twig/html-extra` and `twig/extra-bundle`.

```console
composer require twig/html-extra twig/extra-bundle
```

Because I need to provide an absolute path instead of an absolute URL I modified
Symfony's configuration `framework.yaml` a bit, I used `base_path` instead
of `base_urls`.

```diff
# ...
assets:
  packages:
    attachments-pdf:
-      base_urls: 'http://127.0.0.1/attachments'
+      base_path: '/app/assets/attachments'
# ...
``` 

I also need a function to read the image content, this can be easily done in PHP
therefor I created my own Twig function. Use _make bundle_ to create a Twig
extension `src/Twig/FileExtension.php`.

```console
bin/console make:twig-extension FileExtension
```

Then we create `file_get_contents` Twig function, as you can see our Twig
function will call PHP's `file_get_contents` function.

```php
<?php // src/Twig/FileExtension.php

namespace App\Twig;

use Twig\Extension\AbstractExtension;
use Twig\TwigFilter;
use Twig\TwigFunction;

class FileExtension extends AbstractExtension
{
  public function getFilters(): array
  {
    return [];
  }

  public function getFunctions(): array
  {
    return [
      new TwigFunction('file_get_contents', $this->fileGetContents(...)),
    ];
  }

  public function fileGetContents(string $filename): string
  {
    return file_get_contents($filename);
  }
}
```

Finally, putting everything together:

<!-- {% raw %} -->

```html
<img src="{{ file_get_contents(asset('avatar.png', 'attachments-pdf')) | data_uri }}"/>
```

<!-- {% endraw %} -->

How it works ?

1. First `asset('avatar.png', 'attachments-pdf')` will generate an absolute
   path, in our case this will be `/app/assets/attachments/avatar.png`. Remember
   we configured this in `framework.yaml`.
2. Then `file_get_contents` function will receive the file path and return the
   content of `avatar.png`.
3. Finally, we pass the image content to `data_uri` filter, this filter is very
   handy because it will do all the hard work for us: generate a valid DATA URI
   syntax, detect the mime type and convert the image to base 64.

## Conclusion

I explained my journey trying to display images in a PDF file, finally I decided
to use DATA URIs.

In order to use DATA URIs in my project I had to:

1. Install `data_uri` Twig filter.
2. Configure _assets packages_ in `framework.yaml`.
3. Create `file_get_contents` Twig function.

I think using DATA URIs is the best solution, at least you are creating PDF files,
because this way you will avoid all conflicts with URL.
