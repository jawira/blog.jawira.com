# Using ::getSize



```php
$size1 = $tmpFile->getSize(); // 0 
clearstatcache(false, $tmpFile->getRealPath());
$size2 = $tmpFile->getSize(); // 123
```
