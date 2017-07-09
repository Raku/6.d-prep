## Introduction

There is probably a lot to discuss about Test and NativeCall module versionning. Since there are not really part of the language themselves they can be decoupled from the language version, but at the same time they are an important part of the functionnality a perl6 bundle provide you so tt's probably not an easy anwser.

Anyway 6.d is probably the occasion to fix this but also fix or address some issue that exist with the current NativeCall.

My experience with NativeCall is tied to the writing of a binding module (Gumbo), some work on DBIish to introduce more perl6ish method and the writing of GPTrixie, a NativeCall code generator.

I also had some work in the NativeCall.pm module itself (type checking for sub signature and ABI version for lib) and some lowlevel stuff (bool and size_t type)

These are just proposal and issue that bother me

## Library version and finding of said library

I added like 2 years ago? the possibility to pass the ABI version of the library to the `native` trait to find the right dll/so file.

There is still a big issue with how NC find the library file : The `native` trait and what you give to it is resolved at compile time to find the right file.

It's not a big deal when writing a simple p6 file, but it's really bad when writing a module and allowing (for example) the user to specify the location of the libfile with an ENV file.

There is already a bad workaround for this, you can give a sub to the native trait that will be executed at runtime.
This look like this
`use NativeCall :ALL; sub foo is native(sub {%*ENV<MY_LIBFOO> || guess_library_name(('foo', v42)) } )`

It's bad for lot of reasons :
-It will be executed for every declaration of a sub with this trait.
-You loose the 'magic' NC does to figure the filename depending on the plateform.
-It should be used for weird/rare issue, not a common use.

The second issue can be resolved (and it's done in my example) by calling the internal NC sub that does it. I am to blame for making it exposable to facilitate the testing of it. But I saw modules just rewriting the logic behind it.

I have a proposal at https://github.com/rakudo/rakudo/pull/716 it can be debated how it should work/name/whatever, but it's an idea how it can be solved.

## Type name

It can be debated (and already had on some PR related to this) but it could be a good idea to have different name for type used in NC context.
There is a double benefit from that. It avoid confusion with some perl6 native type like `int` and `num` that should not be used. It make a clear separation and it will probably be easier to have a mechanisme to add type in VMs than having to add specific code to specific perl6 type.

I mean `c_int` or `c_double` make it probably obvious at what they are.

## C Str type

Having a C str can be useful. A C str will be a string but that follow just libc rule of a string (string of char ending with \0). 

You can ask why? When we can already map `char *` to Str. Encoding is the issue as will assume unicode encoding by default, but you can add trait to specify other encoding. 
You can find it nice but it's not good for everything. For example Postgresql will returns you a char * for a String field and you have to request the encoding of it separated. So you only know the encoding of your data at runtime after executing the request. In this case every attempt at putting other encoding will probably lead to issues. I could probably map that to Buf and do weird stuff, but a C str has a meaning/definition. it's not just an array of char.


## Type check throwing error

When I added type checking on native sub signature, I make it die but it was reverted to warning because it will disrupt stuff writing toward 6.c. I think it should be made to die again.
