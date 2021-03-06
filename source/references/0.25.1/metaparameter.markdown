---
layout: legacy
title: Metaparameter Reference
---


Metaparameter Reference
=====


<p><strong>This page is autogenerated; any changes will get overwritten</strong> <em>(last generated on Thu Feb 24 17:22:35 -0800 2011)</em></p>


Metaparameters
=====

<p>Metaparameters are parameters that work with any resource type; they are part of the
Puppet framework itself rather than being part of the implementation of any
given instance.  Thus, any defined metaparameter can be used with any instance
in your manifest, including defined components.</p>

Available Metaparameters
-----


### alias

<p>Creates an alias for the object.  Puppet uses this internally when you
provide a symbolic name:</p>
<pre><code>
file { sshdconfig:
    path =&gt; $operatingsystem ? {
        solaris =&gt; &quot;/usr/local/etc/ssh/sshd_config&quot;,
        default =&gt; &quot;/etc/ssh/sshd_config&quot;
    },
    source =&gt; &quot;...&quot;
}

service { sshd:
    subscribe =&gt; file[sshdconfig]
}
</code></pre>
<p>When you use this feature, the parser sets <code>sshdconfig</code> as the name,
and the library sets that as an alias for the file so the dependency
lookup for <code>sshd</code> works.  You can use this parameter yourself,
but note that only the library can use these aliases; for instance,
the following code will not work:</p>
<pre><code>
file { &quot;/etc/ssh/sshd_config&quot;:
    owner =&gt; root,
    group =&gt; root,
    alias =&gt; sshdconfig
}

file { sshdconfig:
    mode =&gt; 644
}
</code></pre>
<p>There's no way here for the Puppet parser to know that these two stanzas
should be affecting the same file.</p>
<p>See the language tutorial for more information.</p>


### before

<p>This parameter is the opposite of <strong>require</strong> -- it guarantees
that the specified object is applied later than the specifying
object:</p>
<pre><code>
file { &quot;/var/nagios/configuration&quot;:
    source  =&gt; &quot;...&quot;,
    recurse =&gt; true,
    before =&gt; Exec[&quot;nagios-rebuid&quot;]
}

exec { &quot;nagios-rebuild&quot;:
    command =&gt; &quot;/usr/bin/make&quot;,
    cwd =&gt; &quot;/var/nagios/configuration&quot;
}
</code></pre>
<p>This will make sure all of the files are up to date before the
make command is run.</p>


### check

<p>Propertys which should have their values retrieved
but which should not actually be modified.  This is currently used
internally, but will eventually be used for querying, so that you
could specify that you wanted to check the install state of all
packages, and then query the Puppet client daemon to get reports
on all packages.</p>


### loglevel

<p>Sets the level that information will be logged.
The log levels have the biggest impact when logs are sent to
syslog (which is currently the default).  Valid values are <code>debug</code>, <code>info</code>, <code>notice</code>, <code>warning</code>, <code>err</code>, <code>alert</code>, <code>emerg</code>, <code>crit</code>, <code>verbose</code>.</p>


### noop

<p>Boolean flag indicating whether work should actually
be done.  Valid values are <code>true</code>, <code>false</code>.</p>


### notify

<p>This parameter is the opposite of <strong>subscribe</strong> -- it sends events
to the specified object:</p>
<pre><code>
file { &quot;/etc/sshd_config&quot;:
    source =&gt; &quot;....&quot;,
    notify =&gt; Service[sshd]
}

service { sshd:
    ensure =&gt; running
}
</code></pre>
<p>This will restart the sshd service if the sshd config file changes.</p>


### require

<p>One or more objects that this object depends on.
This is used purely for guaranteeing that changes to required objects
happen before the dependent object.  For instance:</p>
<pre><code>
# Create the destination directory before you copy things down
file { &quot;/usr/local/scripts&quot;:
    ensure =&gt; directory
}

file { &quot;/usr/local/scripts/myscript&quot;:
    source =&gt; &quot;puppet://server/module/myscript&quot;,
    mode =&gt; 755,
    require =&gt; File[&quot;/usr/local/scripts&quot;]
}
</code></pre>
<p>Multiple dependencies can be specified by providing a comma-seperated list
of resources, enclosed in square brackets:</p>
<pre><code>
require =&gt; [ File[&quot;/usr/local&quot;], File[&quot;/usr/local/scripts&quot;] ]
</code></pre>
<p>Note that Puppet will autorequire everything that it can, and
there are hooks in place so that it's easy for resources to add new
ways to autorequire objects, so if you think Puppet could be
smarter here, let us know.</p>
<p>In fact, the above code was redundant -- Puppet will autorequire
any parent directories that are being managed; it will
automatically realize that the parent directory should be created
before the script is pulled down.</p>
<p>Currently, exec resources will autorequire their CWD (if it is
specified) plus any fully qualified paths that appear in the
command.   For instance, if you had an <code>exec</code> command that ran
the <code>myscript</code> mentioned above, the above code that pulls the
file down would be automatically listed as a requirement to the
<code>exec</code> code, so that you would always be running againts the
most recent version.</p>


### schedule

<p>On what schedule the object should be managed.  You must create a
schedule object, and then reference the name of that object to use
that for your schedule:</p>
<pre><code>
schedule { daily:
    period =&gt; daily,
    range =&gt; &quot;2-4&quot;
}

exec { &quot;/usr/bin/apt-get update&quot;:
    schedule =&gt; daily
}
</code></pre>
<p>The creation of the schedule object does not need to appear in the
configuration before objects that use it.</p>


### subscribe

<p>One or more objects that this object depends on.  Changes in the
subscribed to objects result in the dependent objects being
refreshed (e.g., a service will get restarted).  For instance:</p>
<pre><code>
class nagios {
    file { &quot;/etc/nagios/nagios.conf&quot;:
        source =&gt; &quot;puppet://server/module/nagios.conf&quot;,
        alias =&gt; nagconf # just to make things easier for me
    }
    service { nagios:
        running =&gt; true,
        subscribe =&gt; File[nagconf]
    }
}
</code></pre>
<p>Currently the <code>exec</code>, <code>mount</code> and <code>service</code> type support
refreshing.</p>


### tag

<p>Add the specified tags to the associated resource.  While all resources
are automatically tagged with as much information as possible
(e.g., each class and definition containing the resource), it can
be useful to add your own tags to a given resource.</p>
<p>Tags are currently useful for things like applying a subset of a
host's configuration:</p>
<pre><code>
puppetd --test --tags mytag
</code></pre>
<p>This way, when you're testing a configuration you can run just the
portion you're testing.</p>
<hr />
<p><em>This page autogenerated on Thu Feb 24 17:22:36 -0800 2011</em></p>






