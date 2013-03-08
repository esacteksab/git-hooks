Let me break it down in pieces and explain the parts

.. code-block:: bash

    ## if a settings file or a models file change, restart uWSGI
    if [ $(git diff HEAD@{1} HEAD --name-only | grep -E
    'conf/*.py|web_site/*/models.py' -c) -ne 0 ]
    then
            source venv/bin/activate && fab restart_uwsgi
    fi

Like the comment says, I'm checking to see if a settings file (located in
`web_root/conf/`) or if a `models.py` file changes, I want to restart uWSGI
(necessary for these changes to take affect.

The next piece:

.. code-block:: bash

    ## if a migration file exists, syncdb and migrate
    if [ $(git diff HEAD@{1} HEAD --name-only | grep -E
    'web_site/*/migrations/*.py' -c) -ne 0 ]
    then
        source venv/bin/activate && fab syncdb && fab migrate_all
   fi

You don't need to run a syncdb or a migrate (if you're using South) every time
you deploy.  So I check to see if a new migration file exists, if so, then I
run a syncdb and a migrate.

And last, sadly a "dirty" hack (sort of). By default, nginx uses www-data
user/group to serve files and I'm actually managing my server as root (cause
real Sys Admins use root! :P)

.. code-block:: bash

    ## if a file changed in web_site/ change owner to www-data:www-data
    if [ $(git diff HEAD@{1} HEAD --name-only | grep -E 'web_site/' -c) -ne 0 ]
    then
       source venv/bin/activate && fab change_owner
    fi

What the above does, is checks to see if a file changed in the `web_root`
directory, if it
did, it changes the owner and group (this ensures that nginx can serve the
necessary files).

Git hooks have proven to be a time saver. Taking quite a bit of redundancy out
of the process/steps necessary to deploy/manage my django projects.

These of course are written to work with my workflow/habits and may not fit
everyone, but my hope is to help those who haven't established a workflow, or
to give a starting point for everyone else, and can tweak here/there as
necessary to support their workflow/habits.