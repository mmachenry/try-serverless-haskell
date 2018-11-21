# try-haskell-serverless

I started off following the [serverless-haskell
documentation](https://hackage.haskell.org/package/serverless-haskell-0.6.7)
but I had to make sure to read the 0.6.7 which is what's available in Stack
LTS-12.19. When I ran `stack new` I was given 12.19 by default.r

I got into trouble following the new docs that I got from Google until I
realized they didn't match the version I was running. I'm also a big confused
that the serverless-haskell-0.8.3 documentation calls for a sererless.yaml file
with a runtime set to `haskell`. When I tried to deploy this I was given an
error indicating that it did not match any of the allowed runtimes and proceded
to list of the available ones, which were, as you'd expect, mostly Pyhton and
Node based.

Also important to note that I used 0.6.7 when I called the `npm install`
command from those docs. This was my process here.

Not mentioned in the docs was that I needed to add serverless-haskell, aeson,
and lens to the dependencies section of the executable of my package.yaml file
for this to build properly.

    stack new try-serverless-haskell
    cd try-serverless-haskell
    npm init .
    npm install --save serverless serverless-haskell@0.6.7
    vi serverless.yaml # cut and pasted what I found from tutorial
    vi app/Main.hs # cut and pasted what I found in tutorial
    vi package.yaml # add dependencies
    stack build
    sls deploy
    sls invoke local -f myfunc
    sls invoke myfunc.
 
Next I added the API Gateway so that this could be invoked over the web as a
REST API. I followed the [AWSLambda.Events.APIGateway
documentation](https://hackage.haskell.org/package/serverless-haskell-0.6.7/docs/AWSLambda-Events-APIGateway.html)
linked to in the serverless-haskell docs and copy and pasted their Main.hs out
of it and used that. I also copied their additions to serverless.yaml. Once I
finished this and built and ran `sls deploy` again I ran into this error when
hitting the URL.

    {"message": "Internal server error"}

So I check my CloudWatch logs where I found an error

    ./try-haskell-serverless-exe: /lib64/libm.so.6: version `GLIBC_2.27' not found (required by ./try-haskell-serverless-exe)

What ths means is that the system that I built the code on, my laptop, had the
required GLIBC package and was dynalically linking to that library. When
running on the AMI Linux system that Lambda's run on, that library seems to not
be available. So I looked up static linking, found [this blog
post](https://ro-che.info/articles/2015-10-26-static-linking-ghc), and add
these two build options to the ghc-options section of the executable in my
package.yaml file.

    -optl-static -optl-pthread

With this, I had a working REST API on AWS Lambda written in Haskell. It was
pretty straight forward. I'm leaving this repo here for myself in the future
and hopefully someone else.

Versions of software I was using
---
    $ stack --version
    Version 1.9.1, Git revision f9d0042c141660e1d38f797e1d426be4a99b2a3c (6168 commits) x86_64 hpack-0.31.0
    $ npm --version
    3.5.2
    $ sls --version
    1.33.2
    $ lsb_release -a
    No LSB modules are available.
    Distributor ID:	Ubuntu
    Description:	Ubuntu 18.04.1 LTS
    Release:	18.04
    Codename:	bionic

