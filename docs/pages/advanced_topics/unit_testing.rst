.. _unit_testing:

Unit testing of prompt_toolkit applications
===========================================

Testing user interfaces is not always obvious. Here are a few tricks for
testing prompt_toolkit applications.


Feeding input to an ``Application``
-----------------------------------

We can create a prompt_toolkit ``Application``; programmatically feed it some
input; have the key bindings process the input and then check what comes out of
it.

In the following example, we use a ``PromptSession``, but the same works for
any ``Application``.

.. code:: python

    from prompt_toolkit.shortcuts import PromptSession
    from prompt_toolkit.input import create_pipe_input
    from prompt_toolkit.output import DummyOutput

    def test_prompt_session():
        inp = create_pipe_input()

        try:
            inp.send_text("hello\n")
            session = PromptSession(
                input=inp,
                output=DummyOutput(),
            )

            result = session.prompt()
        finally:
            inp.close()

        assert result == "hello"


The output could be checked in a similar way, by creating a custom ``Output``
object, but usually it doesn't provide much additional value. (The output calls
can change any time when the rendering algorithm changes.)


Using a dummy output for everything
-----------------------------------

When using pytest, ``sys.stdout`` will be patched, and can't be used by
prompt_toolkit. An easy way to tell any prompt_toolkit code to write all its
output to a dummy output, is by creating a new ``AppSession`` with a
``DummyOutput``. This way, we don't need to inject it into every
``Application`` or ``print_formatted_text`` call.

.. code:: python

    from prompt_toolkit.application import create_app_session
    from prompt_toolkit.shortcuts import print_formatted_text
    from prompt_toolkit.output import DummyOutput

    def test_something():
        with create_app_session(output=DummyOutput()):
            ...
            print_formatted_text('Hello world')
            ...

Without the ``DummyOutput``, all code that contained a ``print_formatted_text``
call would fail in pytest, even if the purpose was to test the code around this
call.

Type checking
-------------

Prompt_toolkit 3.0 is fully type annotated. This means that if a
prompt_toolkit application is typed too, it can be verified with mypy. This is
complementary to unit tests, but also great for testing for correctness.
