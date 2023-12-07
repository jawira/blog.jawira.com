Should I use assert in production? 

The short answer is no. The thing is that assert function can be disabled using pho.ini configuration so you must not rely on this function to validated any variable.

## How does it work?

## Assert alternatives

Alternative 1: use if statement

Alternative 2: write your own function

Alternative 3: use ternary operator

## Conclusion

Never rely on Assert function since it should be disabled in production.

