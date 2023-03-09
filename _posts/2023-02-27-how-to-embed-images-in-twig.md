---
layout: post
title: "How to embed images in Twig"
---

Recently I had to create PDF files in a Symfony project using Twig, however this
is not as easy as it seems, for example you need a special CSS file adapted to
print.

Another obstacle are images, in this post I explain why using images is a tricky
task and how I solved this problem.

This is the stack I used to generate PDF files:

- [Symfony 6](https://symfony.com/) - Web framework.
- [Twig 3](https://twig.symfony.com/doc/3.x/) - To generate the content of PDF
  files.
- [KnpSnappyBundle](https://github.com/KnpLabs/KnpSnappyBundle) - To convert
  Twig view to a PDF file.

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
does not exist and means nothing, therefore the image was never found.

## Using special asset configuration (it didn't work either)

As told before, the url `http://web.project.localhost` is not resolved when
requested inside the container, so what kind of URL is always valid inside a
container ? The answer is a loopback address, that's how I had the idea to
use `127.0.0.1` to load my images, so I had to configure a special _asset
package_ in `framework.yaml` file.

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

This would have worked except that, in order to view an image, I needed to be
<u>logged in</u> to the website, image was never displayed because instead of
an image I was receiving the login page.

## Embedding images in Twig

So finally I came up to the idea to embed images directly inside PDF file, I
wasn't so keen about this solution because it's not a _drop-in solution_, it
requires some configuration to work properly.

To embed an image within any HTML file you can use **DATA URIs** instead of an
image url, this is the syntax that DATA URI follows:

> `data:<mimetype>;<encoding>,<data>`

Here an example taken
from [Wikipedia](https://en.wikipedia.org/wiki/Data_URI_scheme#HTML):

```HTML
<img src="data:image/png;base64,iVBORw0KGgoAAA
ANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4
//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU
5ErkJggg==" alt="Red dot"/>
```

Luckily for me, somebody already created a
the [`data_uri`](https://twig.symfony.com/doc/3.x/filters/data_uri.html) filter
to create DATA URIs, this filter is not installed by default, so the following
packages are required `twig/html-extra` and `twig/extra-bundle`.

```console
composer require twig/html-extra twig/extra-bundle
```

To avoid any problems with paths I decided to use absolute paths, therefore I
configured this in `framework.yaml`, I used `base_path` instead of `base_urls`.

```diff
assets:
  packages:
    attachments-pdf:
-      base_urls: 'http://127.0.0.1/attachments'
+      base_path: '/app/assets/attachments'
``` 

Now I need a way to read the image content, this can be easily done in PHP
creating a custom Twig filter. You can use _make bundle_ to create a Twig
extension, our custom filter will be located inside.

```console
bin/console make:twig-extension FileExtension
```

Then we create `file_get_contents` Twig extension, as you can see our Twig
function will call PHP's `file_get_contents` function.

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
<img src="{{ asset('avatar.png', 'attachments-pdf') | file_get_contents | data_uri }}"/>
```

<!-- {% endraw %} -->

How it works ?

1. First `asset('avatar.png', 'attachments-pdf')` will generate an absolute
   path, in our example this will be `/app/assets/attachments/avatar.png`.
   Remember we configured this in `framework.yaml`.
2. Then `file_get_contents` filter will receive the file path and return the
   content of `avatar.png`.
3. Finally, we pass the image content to `data_uri` filter, this filter is very
   handy because it will do all the hard work for us: generate a valid DATA URI
   syntax, detect the mime type and convert the image to base 64.

## Conclusion

I explained my journey trying to display images in a PDF file, I tested many
solutions and I decided to use DATA URIs.

In order to use DATA URIs in my project I had to:

1. Install `data_uri` Twig filter.
2. Configure _assets packages_ in `framework.yaml`.
3. Create `file_get_contents` Twig filter.

I think using DATA URIs is the best solution when you are creating PDF files
because you will avoid all conflicts with URL.
