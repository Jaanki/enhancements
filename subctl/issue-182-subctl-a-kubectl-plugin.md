# Support krew way of installing `subctl`

Related Issue:
[Add subctl as kubectl plugin](https://github.com/submariner-io/enhancements/issues/182)

## Summary
Krew is a tool that makes it easy to use kubectl plugins. Krew helps you discover plugins, install and manage them on your machine. It is
similar to tools like apt, dnf or brew. Today, over [200 kubectl plugins](https://krew.sigs.k8s.io/plugins/) are available on Krew.

A plugin is a standalone executable file, whose name begins with `kubectl-`.

More information can be found on [extending kubectl with plugins](https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/) page.

On the surface, installing a kubectl plugin seems simple enough – all you need to do is to place an executable in the user’s `PATH` prefixed
with `kubectl-` – that you may be considering some other alternatives to Krew, such as:

* Having the user manually download the plugin executable and move it to some directory in the PATH
* Distributing the plugin executable using an OS package manager, like Homebrew (macOS), apt/yum (Linux), or Chocolatey (Windows)
* Distributing the plugin executable using a language package manager (e.g. npm or go get)

While these approaches are not necessarily unworkable, potential drawbacks to consider include:

* How to get updates to users (in the case of manual installation)
* How to package a plugin for multiple platforms (macOS, Linux, and Windows)
* How to ensure your users have the appropriate language package manager (go, npm)
* How to handle a change to the implementation language (e.g. a move from npm to another package manager)

Krew solves these problems cleanly for all kubectl plugins, since it’s designed specifically to address these shortcomings. With Krew,
after you write a plugin manifest once your plugin can be installed on all platforms without having to deal with their package managers.

## Proposal

A `kubectl-subm` binary would be created and its krew manifest file written.

## Design

To generate `kubectl-subm` binary, the current [`dist/subctl-%.tar.xz`](https://github.com/submariner-io/subctl/blob/devel/Makefile#L92)
target in subctl repository will be edited to also create `kubectl-subm*.tar.gz` binaries and their checksum file. This is what [Krew 
expects](https://krew.sigs.k8s.io/docs/developer-guide/plugin-manifest/#sample-plugin-manifest).

    dist/subctl-%: cmd/bin/subctl-%
	mkdir -p dist
	tar -cJf $@.tar.xz --transform "s/^cmd.bin/subctl-$(VERSION)/" $<
    tar -cJf $@.tar.gz --transform "s/^cmd.bin.subctl-$(VERSION)/kubectl-subm-$(VERSION)/" $<
    md5sum $@.tar.gz > dist/kubectl-subm-$(VERSION)-checksums.txt

The above logic will create `kubectl-subm-$VERSION` binaries under the parent folder itself.

Another option is to drop `.xz` extension files and only keep `.gz` extension files. The effect of this change to other projects would 
need to be considered, if any.

The [root command name](https://github.com/submariner-io/subctl/blob/devel/cmd/subctl/root.go#L58) would need to be made variable based on
the argument provided. This would allow running `kubectl-subm` command for the plugin and `subctl` command for standalone `subctl` binary.

    // rootCmd represents the base command when called without any subcommands.
    var rootCmd = &cobra.Command{
    Use:   filepath.Base(os.Args[0]),
    Short: "An installer for Submariner",
    }

Once `.tar.gz` files are available, krew plugin manifest for subctl would be written as per
[these instructions](https://krew.sigs.k8s.io/docs/developer-guide/plugin-manifest/#sample-plugin-manifest).

## Usage

Once this is done, users would be able to install subctl via krew. Few of krew command examples are:

    kubectl krew install subm
    kubectl krew upgrade subm

and use `subctl` as `kubectl-subm` or `kubectl subm`.

    $ kubectl-subctl          
    An installer for Submariner
    
    Usage:
    kubectl-subctl [command]
    
    Available Commands:
    benchmark           Benchmark tests
    cloud               Cloud operations
    completion          Generate the autocompletion script for the specified shell
    deploy-broker       Deploys the broker
    diagnose            Run diagnostic checks on the Submariner deployment and report any issues
    export              Exports a resource to other clusters
    gather              Gather troubleshooting information from a cluster
    help                Help about any command
    join                Connect a cluster to an existing broker
    recover-broker-info Recovers the broker-info.subm file from the installed Broker
    show                Show information about Submariner
    unexport            Stop a resource from being exported to other clusters
    uninstall           Uninstall Submariner and its components
    upgrade             Upgrades Submariner
    verify              Run verifications between two clusters
    version             Get version information on subctl
    
    Flags:
    -h, --help   help for kubectl-subctl
    
    Use "kubectl-subctl [command] --help" for more information about a command.

## External Dependencies

None.

## User Impact

Users would be able to install `subctl` via krew.
