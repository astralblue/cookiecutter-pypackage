This block {%- raw %}::

    {%- import 'import_helper.rst' as import %}
    {%- with standard = {}, third_party = {}, local = {} %}
    {{- import.module_as(standard, 'ConfigParser', 'configparser') }}
    {{- import.module_as(standard, 'os.path', 'os_path') }}
    {{- import.module(local, 'my_project') }}
    {{- import.module(third_party, 'django.db.models',
                                   'django.db.backends.postgresql') }}
    {{- import.submodule_as(third_party, 'django.db', 'models', 'djmods') }}
    {{- import.submodule_as(third_party, 'django.db.backends',
                            'postgresql_psycopg2', 'djpsql') }}
    {{- import.submodule(standard, 'urllib', 'parse', 'request', 'error') }}
    {{- import.symbol_as(standard, 'os.path', 'join', 'join_path') }}
    {{- import.symbol(third_party, 'sqlalchemy.engine', 'Engine',
                                                        'Transaction') }}
    {{- import.symbol(third_party, 'sqlalchemy.engine', 'RowProxy') }}
    {{- import.emit(standard) }}
    {{- import.emit(third_party) }}
    {{- import.emit(local) }}
    {%- endwith %}

{% endraw -%} Becomes rendered as::

    {%- import 'import_helper.rst' as import %}
    {%- with standard = {}, third_party = {}, local = {} %}
    {{- import.module_as(standard, 'ConfigParser', 'configparser') }}
    {{- import.module_as(standard, 'os.path', 'os_path') }}
    {{- import.module(local, 'my_project') }}
    {{- import.module(third_party, 'django.db.models',
                                   'django.db.backends.postgresql') }}
    {{- import.submodule_as(third_party, 'django.db', 'models', 'djmods') }}
    {{- import.submodule_as(third_party, 'django.db.backends',
                            'postgresql_psycopg2', 'djpsql') }}
    {{- import.submodule(standard, 'urllib', 'parse', 'request', 'error') }}
    {{- import.symbol_as(standard, 'os.path', 'join', 'join_path') }}
    {{- import.symbol(third_party, 'sqlalchemy.engine', 'Engine',
                                                        'Transaction') }}
    {{- import.symbol(third_party, 'sqlalchemy.engine', 'RowProxy') }}
    {{- import.emit(standard)|indent(4) }}
    {{- import.emit(third_party)|indent(4) }}
    {{- import.emit(local)|indent(4) }}
    {%- endwith %}
