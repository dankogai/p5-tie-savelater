[![build status](https://secure.travis-ci.org/dankogai/p5-tie-savelater.png)](http://travis-ci.org/dankogai/p5-tie-savelater)

p5-tie-savelater
================
# NAME

Tie::SaveLater - A base class for tie modules that "save later".

# SYNOPSIS

```perl
package Tie::Storable;
use base 'Tie::SaveLater';
use Storable qw(retrieve nstore);
__PACKAGE__->make_subclasses;
sub load{ retrieve($_[1]) };
sub save{ nstore($_[0], $_[0]->filename) };
1;

# later
use Tie::Storable;
{
    tie my $scalar => 'Tie::Storable', 'scalar.po';
    $scalar = 42;
} # scalar is automatically saved as 'scalar.po'.
{
    tie my @array => 'Tie::Storable', 'array.po';
    @array = qw(Sun Mon Tue Wed Fri Sat);
} # array is automatically saved as 'array.po'.
{
    tie my %hash => 'Tie::Storable', 'hash.po';
    %hash = (Sun=>0, Mon=>1, Tue=>2, Wed=>3, Thu=>4, Fri=>5, Sat=>6);
} # hash is automatically saved as 'hash.po'.
{
    tie my $object => 'Tie::Storable', 'object.po';
    $object = bless { First => 'Dan', Last => 'Kogai' }, 'DANKOGAI';
} # You can save an object; just pass a scalar
{
    tie my $object => 'Tie::Storable', 'object.po';
    $object->{WIFE} =  { First => 'Naomi', Last => 'Kogai' };
    # you can save before you untie like this
    tied($object)->save;
}
```

# DESCRIPTION

Tie::SaveLater make you easy to write a modules that "save later",
that is, save on untie. 

## WHY?

Today we have a number of serializers that store complex data
structures, from [Data::Dumper](https://metacpan.org/pod/Data%3A%3ADumper) to [Storable](https://metacpan.org/pod/Storable).  If those core
modules are not enough, you have [YAML](https://metacpan.org/pod/YAML) and [DBI](https://metacpan.org/pod/DBI) and more via CPAN.

Problem?  You have to save AFTER you are done with your data
structure.  Don't forget to save when you are out of scope just like
locking the door before you leave.

But can't you make it so it autosaves as Hotel doors autolocks?
That's exactly what this module is for.  This module comes with
[Tie::DataDumper](https://metacpan.org/pod/Tie%3A%3ADataDumper), [Tie::Storable](https://metacpan.org/pod/Tie%3A%3AStorable), and [Tie::YAML](https://metacpan.org/pod/Tie%3A%3AYAML) so you
can make your data structures autosave today!

## DETAILS

["SYNOPSIS"](#synopsis) illustrates how to implement [Tie::Storable](https://metacpan.org/pod/Tie%3A%3AStorable) in seven
lines.  Suppose your module is _Tie::Them_, Your module needs to do
the following;

- assign Tie::SaveLater as your base class
- call \_\_PACKAGE\_->make\_subclasses

    That automatically builds _Tie::Them::_SCALAR, _Tie::Them::_ARRAY,
    and _Tie::Them::_HASH for you.

- define `load()` as a class method

    Here is a more descriptive way to define Tie::Storable::load().

    ```perl
    sub load{
      my $class    = shift;
      my $filename = shift;
      return retrieve($filename) 
    };
    ```

    First argument is a class name (you don't need that in this case) and
    the second argument is the filename.  It must return a loaded object.

- define `save()` as an object method

    Here is a more descriptive way to define Tie::Storable::save().

    ```perl
    sub save{ 
        my $self = shift;
        my $filename = $self->filename;
        return nstore($self, $filename);
    };
    ```

    It takes only one argument -- `$self`.  And you can obtain the
    filename as `$self->filename`.  

    You can also obtain optional arguments that are fed in `<tie`> as
    `$self->options` .

    ```perl
    tie my $obj, 'Tie::Them', 'them.obj', 0666, qw/more options/;
    ```

    In the statement above, `$self->options` returns (0666, 'more',
    'options').  This is handy you want to overload `FETCH()`,
    `STORE()` and other tie methods for more minute control.

## EXPORT

None by default.

# SEE ALSO

[perltie](https://metacpan.org/pod/perltie), [Tie::Scalar](https://metacpan.org/pod/Tie%3A%3AScalar), [Tie::Array](https://metacpan.org/pod/Tie%3A%3AArray), [Tie::Hash](https://metacpan.org/pod/Tie%3A%3AHash)

[Tie::Storable](https://metacpan.org/pod/Tie%3A%3AStorable), [Tie::YAML](https://metacpan.org/pod/Tie%3A%3AYAML), [Tie::DataDumper](https://metacpan.org/pod/Tie%3A%3ADataDumper)

# AUTHOR

Dan Kogai, `<dankogai at cpan.org>`

# COPYRIGHT AND LICENSE

Copyright (C) 2006-2020 by Dan Kogai

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself, either Perl version 5.8.8 or,
at your option, any later version of Perl 5 you may have available.
