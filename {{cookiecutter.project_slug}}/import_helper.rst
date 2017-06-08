.. {% raw %} ..

These are Jinja2 macros that can help conditional imports organized per:

- PEP 8: Group imports into Standard/third-party/package-local paragraphs
- OpenStack Style Guidelines H301: Do not import more than one module per line
- OpenStack Style Guidelines H306: Alphabetically order imports

Usage
=====

1.  Begin by importing this file and defining three empty dicts, e.g.
    ``standard``, ``third_party``, and ``local``.

2.  Invoke `module()`, `module_as()`, `symbol()`, and/or `symbol_as()` to
    schedule import of module(s) and/or symbol(s) for inclusion; use one of the
    three dicts to include in the right import paragraph.

3.  Finally, call the emit macro with each dict to emit the import statements.

The skeleton is::

    {%- import 'import_helper.rst' as import %}
    {%- with standard = {}, third_party = {}, local = {} %}
    <actual imports come here>
    {{- import.emit(standard) }}
    {{- import.emit(third_party) }}
    {{- import.emit(local) }}
    {%- endwith %}

See import_helper_examples.rst for a concrete example.

Macro Reference
===============

``module_as(paragraph, module, alias)``
---------------------------------------

Schedule import of a module under an alias, using the ``import <module> as
<alias>`` syntax.  Example::

    {{- import.module_as(standard, 'os.path', 'os_path') }}
    {{- import.module_as(standard, 'ConfigParser', 'configparser') }}

Becomes emitted later as::

    import ConfigParser as configparser
    import os.path as os_path

``module(paragraph, modules...)``
---------------------------------

Schedule import of one or more modules, using the ``import <module>`` syntax.
Example::

    {{- import.module(local, 'my_project') }}
    {{- import.module(third_party, 'django.db.models',
                                   'django.db.backends.postgresql') }}

Becomes emitted later as::

    import django.db.backends.postgresql
    import django.db.models

    import my_project

``submodule_as(paragraph, package, submodule, alias)``
------------------------------------------------------

Schedule import of a submodule under an alias, using the ``from <package>
import <submodule> as <alias>`` syntax.  Example::

    {{- import.submodule_as(third_party, 'django.db', 'models', 'djmods') }}
    {{- import.submodule_as(third_party, 'django.db.backends',
                            'postgresql_psycopg2', 'djpsql') }}

Becomes emitted later as::

    from django.db.backends import postgresql_psycopg2 as djpsql
    from django.db import models as djmods

``submodule(paragraph, package, submodules...)``
------------------------------------------------

Schedule import of one or more submodules, using the ``from <package> import
<submodule>`` syntax.  Example::

    {{- import.submodule(standard, 'urllib', 'parse', 'request', 'error') }}

Becomes emitted later as::

    from urllib import error
    from urllib import parse
    from urllib import request

``symbol_as(paragraph, module, symbol, alias)``
-----------------------------------------------

Schedule import of a non-module symbol, using the ``from <module> import
<symbol> as <alias>`` syntax.  Example::

    {{- import.symbol_as(standard, 'os.path', 'join', 'join_path') }}

Becomes emitted later as::

    from os.path import join as join_path

``symbol(paragraph, module, symbols...)``
-----------------------------------------

Schedule import of one or more non-module symbols, using the ``from <module>
import <symbol>, ...`` syntax.  Example::

    {{- import.symbol(third_party, 'sqlalchemy.engine', 'Engine',
                                                        'Transaction') }}
    {{- import.symbol(third_party, 'sqlalchemy.engine', 'RowProxy') }}

Becomes emitted later as::

    from sqlalchemy.engine import Engine, RowProxy, Transaction

``emit(paragraph)``
-------------------

Emit contents of an import paragraph.

Rules:

- If the given paragraph is empty (no imports scheduled in the paragraph), emit
  nothing.  Otherwise, emit an empty line followed by the import lines, one per
  module, per PEP 8.
- Sort import lines by their full module name, per H306.
- Put module imports one per line, per H301.
- Combine non-module symbol imports from the same module onto one line.
- Sort the symbols imported on the same line (from the same module) alphabetically.

These macros are designed to be used with leading whitespace stripping but not
trailing, e.g. ``{{- import.module(standard, 'os') }}``.  Emit each line,
including the initial empty line, with the newline at the beginning instead of
the end.  This means that the contents, if not empty, shall always begin with
two newlines at the beginning and no newlines at the end.

Macro Source
============

.. {% endraw %} ..

``
{%  macro _add_module(paragraph, package) -%}
{{-  paragraph.setdefault(package, [none, none, {}]) and '' -}}
{%- endmacro %}

{%  macro module_as(paragraph, module, alias) -%}
{{-  _add_module(paragraph, module) -}}
{{-  paragraph[module].__setitem__(0, false) or '' -}}
{{-  paragraph[module].__setitem__(1, alias) or '' -}}
{%- endmacro %}

{%  macro module(paragraph) -%}
{%-  with -%}
{%-   for module in varargs -%}
{{-    module_as(paragraph, module, none) -}}
{%-   endfor -%}
{%-  endwith -%}
{%- endmacro %}

{%  macro submodule_as(paragraph, package, submodule, alias) -%}
{%-  with module = package + '.' + submodule -%}
{{-   _add_module(paragraph, module) -}}
{{-   paragraph[module].__setitem__(0, true) or '' -}}
{{-   paragraph[module].__setitem__(1, alias) or '' -}}
{%-  endwith %}
{%- endmacro %}

{%  macro submodule(paragraph, package) -%}
{%-  with -%}
{%-   for submodule in varargs -%}
{{-    submodule_as(paragraph, package, submodule, none) -}}
{%-   endfor -%}
{%-  endwith -%}
{%- endmacro %}

{%  macro symbol_as(paragraph, module, symbol, alias) -%}
{{-  _add_module(paragraph, module) -}}
{{-  paragraph[module][2].__setitem__(symbol, alias) or '' -}}
{%- endmacro %}

{%  macro symbol(paragraph, module) -%}
{%-  with -%}
{%-   for symbol in varargs -%}
{{-    symbol_as(paragraph, module, symbol, none) -}}
{%-   endfor -%}
{%-  endwith -%}
{%- endmacro %}

{%  macro emit(paragraph) %}
{%-  with %}
{%-   if paragraph %}
{#     keep this comment for an empty line before import paragraph #}
{%-    for module, (style, alias, symbols) in paragraph|dictsort %}
{%-     if style is not none %}
{%-      if style %}
{%-       set from, submodule = module.rsplit('.', 1) %}
from {{ from }} import {{ submodule }}
{%-      else %}
import {{ module }}
{%-      endif %}
{%-      if alias is not none -%}
{# #} as {{ alias }}
{%-      endif %}
{%-     endif %}
{%-     if symbols %}
{%-      set comma = joiner(', ') %}
from {{ module }} import {# #}
{%-      for symbol, alias in symbols|dictsort -%}
{{-       comma() }}{{ symbol }}{% if alias is not none %} as {{ alias }}{% endif %}
{%-      endfor %}
{%-     endif %}
{%-    endfor %}
{%-   endif %}
{%-  endwith %}
{%- endmacro %}
``
