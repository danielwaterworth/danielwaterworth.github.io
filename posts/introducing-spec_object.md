% Introducing spec_object
% Daniel Waterworth
% July 23, 2016

![](../images/orb.jpg)

## Rationale

Here's an interesting question: "what would a programming language look like
that allows you to describe what a program does, but not how that is
accomplished?". As programmers, we like to talk about declarative languages and
DSLs and APIs, but how far can declarative-ness be pushed?

Like all good fundamental CS questions, it was answered in the 70s and since
then there have been a number of so-called
[program specification languages][1].

Now, it's really important that you understand that I'm not talking about
[UML][2]; a language used to describe the architecture of software. I'm talking
about languages that express behaviour, but in a really high-level, abstract
kind of way.

These languages tend to be based on logic. The classic example is sorting.
Instead of saying, "in order to sort, we split the input into two equal halfs,
sort the two halfs recursively and merge the results", we say, "sorting is a
function that takes a list and produces a list. The output is a permutation of
the input and the output is [monotone][3]".

Expressing pure functions is all well and good, but what about real programs
with side-effects? Without further ado, let's dive into the `spec_object` DSL.

## A worked example

For this example, we're going to specify the behaviour of a hashtable. To limit
the scope, we're only going to consider `set`, `get` and `delete`.

`spec_object` models time explicitly. Time is linear, discrete and the future
doesn't exist. *If this causes you philosophical problems, I don't know what to
suggest.*

Specifying the behaviour of the `set` and `del` is the easy part:

~~~~ {.ruby}
behaviour :set do |args, output|
    output == nil
end

behaviour :del do |args, output|
    output == nil
end
~~~~

`get` is much more difficult. The behaviour of `get` depends on what has
already happened. There are a few differents cases to consider:

 * The key was never set,
 * The key was set and deleted,
 * The key was set and updated,

Any logical assertions we make should be true of all executions; we need to
somehow divide the space of possibilities to attack this problem. One natural
line of division is whether the output was `nil` or not.

~~~~ {.ruby}
behaviour :get do |args, output|
    # ite stands for if-then-else
    ite(
        output == nil,
        true, # This should be true when nil was the result
        true  # This should be true when it wasn't
    )
end
~~~~

If the output was `nil`, it means, either the key was never set or the key was
deleted and hasn't been set since:

~~~~ {.ruby}
behaviour :get do |args, output|
    the_key_was_never_set = true
    the_key_was_deleted_and_not_set_since = true

    ite(
        output == nil,
        either(the_key_was_never_set, the_key_was_deleted_and_not_set_since),
        true
    )
end
~~~~

The key never being set means there doesn't exist a time and a value where the
object received `:set` at that time with the `key` from `args` and the value:

~~~~ {.ruby}
key = args[0]

the_key_was_never_set =
    !exist do |set_time|
        exist do |value|
            received(:set).at(set_time).with(key, value)
        end
    end
~~~~

The key was deleted and not set again means there exists a time at which the
key was deleted for which there does not exist a later time at which the `key`
was set.

~~~~ {.ruby}
the_key_was_deleted_and_not_set_since =
    exist do |delete_time|
        key_was_deleted_at_delete_time =
            received(:del).at(delete_time).with(key)

        no_later_set_time =
            !exist do |later_set_time|
                later_set_time_is_later =
                    later_set_time > delete_time

                key_was_set_at_later_set_time =
                    exist do |value|
                        received(:set).at(later_set_time).with(key, value)
                    end

                both(later_set_time_is_later, key_was_set_at_later_set_time)
            end

        both(key_was_deleted_at_delete_time, no_later_set_time)
    end
~~~~

That covers the `output == nil` case. If the `output` is not `nil`, it means
that there exists a time at which the `key` was set with `output` and there
doesn't exist a later time at which the key was deleted or a set again.

~~~~ {.ruby}
exists do |set_time|
    key_was_set_at_set_time =
        received(:set).at(set_time).with(key, output)

    no_later_set_time =
        !exist do |later_set_time|
            later_set_time_is_later =
                later_set_time > set_time

            set_at_later_set_time =
                exist do |value|
                    received(:set).at(later_set_time).with(key, value)
                end

            both(later_set_time_is_later, set_at_later_set_time)
        end

    no_later_delete_time =
        !exist do |later_delete_time|
            later_delete_time_is_later =
                later_delete_time > set_time

            delete_at_later_delete_time =
                received(:set).at(later_delete_time).with(key)

            both(later_delete_time_is_later, delete_at_later_delete_time)
        end

    all(key_was_set_at_set_time, no_later_set_time, no_later_delete_time)
end
~~~~

And we're done. Tying it all together:

~~~~ {.ruby}
behaviour :get do |args, output|
    key = args[0]

    the_key_was_never_set =
        !exist do |set_time|
            exist do |value|
                received(:set).at(set_time).with(key, value)
            end
        end

    the_key_was_deleted_and_not_set_since =
        exist do |delete_time|
            key_was_deleted_at_delete_time =
                received(:del).at(delete_time).with(key)

            no_later_set_time =
                !exist do |later_set_time|
                    later_set_time_is_later =
                        later_set_time > delete_time

                    key_was_set_at_later_set_time =
                        exist do |value|
                            received(:set).at(later_set_time).with(key, value)
                        end

                    both(later_set_time_is_later, key_was_set_at_later_set_time)
                end

            both(key_was_deleted_at_delete_time, no_later_set_time)
        end

    the_key_was_last_set_to_output =
        exists do |set_time|
            key_was_set_at_set_time =
                received(:set).at(set_time).with(key, output)

            no_later_set_time =
                !exist do |later_set_time|
                    later_set_time_is_later =
                        later_set_time > set_time

                    set_at_later_set_time =
                        exist do |value|
                            received(:set).at(later_set_time).with(key, value)
                        end

                    both(later_set_time_is_later, set_at_later_set_time)
                end

            no_later_delete_time =
                !exist do |later_delete_time|
                    later_delete_time_is_later =
                        later_delete_time > set_time

                    delete_at_later_delete_time =
                        received(:set).at(later_delete_time).with(key)

                    both(later_delete_time_is_later, delete_at_later_delete_time)
                end

            all(key_was_set_at_set_time, no_later_set_time, no_later_delete_time)
        end

    ite(
        output == nil,
        either(the_key_was_never_set, the_key_was_deleted_and_not_set_since),
        the_key_was_last_set_to_output
    )
end
~~~~

It's kind of wordy; Partly because I intentionally used riduculously long
variable names and partly because I didn't break out the individual pieces into
their own methods, but hopefully you get the idea.

Having gone to the trouble of specifying the behaviour of an object using
logical language, `spec_object` is now able to detect cases where these
constraints are broken (such as when we insert `nil` as a value). This would
allow you to fuzz test your stateful APIs against their logical specification.

I hope you found this interesting. **Please note, `spec_object` is a
prototype and should not be used for anything you care about!**

[Here's a link to the repo.][4]

[1]: https://en.wikipedia.org/wiki/Specification_language
[2]: https://en.wikipedia.org/wiki/UML
[3]: https://en.wikipedia.org/wiki/Monotonic_function
[4]: https://github.com/DanielWaterworth/spec_object
