# Upgrade Notes

If you are upgrading from **Fluent Bit <= 1.0.x** you should take in consideration the following relevant changes when switching to **Fluent Bit v1.1** series:

## Kubernetes Filter

We introduced a new configuration property called _Kube\_Tag\_Prefix_ to help Tag prefix resolution and address an unexpected behavior that landed in previous versions.

Duing 1.0.x release cycle, a commit in Tail input plugin changed the default behavior on how the Tag was composed when using the wildcard for expansion generating breaking compatibility with other services. Consider the following configuration example:

```
[INPUT]
    Name  tail
    Path  /var/log/containers/*.log
    Tag   kube.*
```

The expected behavior is that Tag will be expanded to:

```
kube.var.log.containers.apache.log
```

but the change introduced in 1.0 series switched from absolute path to the base file name only:

```
kube.apache.log
```

On Fluent Bit v1.1 release we restored to our default behavior and now the Tag is composed using the absolute path of the monitored file.

> Having absolute path in the Tag is relevant for routing and flexible configuration where it also helps to keep compatibility with Fluentd behavior.

This behavior switch in Tail input plugin affects how Filter Kubernetes operates. As you know when the filter is used it needs to perform local metadata lookup that comes from the file names when using Tail as a source. Now with the new _Kube\_Tag\_Prefix_ option you can specify what's the prefix used in Tail input plugin, for the configuration example above the new configuration will look as follows:

```
[INPUT]
    Name  tail
    Path  /var/log/containers/*.log
    Tag   kube.*

[FILTER]
    Name             kubernetes
    Match            *
    Kube_Tag_Prefix  kube.var.log.containers.
```

So the proper for _Kube\_Tag\_Prefix_ value must be composed by Tag prefix set in Tail input plugin plus the converted monitored directory replacing slashes with dots.
