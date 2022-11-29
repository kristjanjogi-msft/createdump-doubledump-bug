# createdump-doubledump-bug

A bug in the [createdump](https://github.com/dotnet/coreclr/blob/master/Documentation/botr/xplat-minidump-generation.md) tool which causes double crash dumps to be created for a single crash when running in a Linux container in Kubernetes.

## Reproduction prerequites

1. .NET 7 SDK
1. Docker Desktop with Linux containers and Kubernetes support

## Reproduction steps:

1. Clone the repository:
`git clone https://github.com/kristjanjogi-msft/createdump-doubledump-bug.git`

1. `cd createdump-doubledump-bug`

1. Build and publish the crashing application:
`dotnet publish -c Release -o published CrashingApplication/CrashingApplication.csproj`

1. Build the Docker image:
`docker build -t crashingapplication:1.0 .`

1. Run a Kubernetes pod in which the createdump tool creates two crash dumps for a single crash:
`kubectl apply -f doubledump.yaml`

1. View the evidence, two crash dumps instead of one are created:
`kubectl logs doubledump`

```
Hello, World!
Unhandled exception. System.Exception: Crash
   at Program.<Main>$(String[] args) in [redacted]\createdump-doubledump-bug\CrashingApplication\Program.cs:line 4
[createdump] Gathering state for process 1 dotnet
[createdump] Crashing thread 00000001 signal 00000006
[createdump] Writing full dump to file /tmp/crashdump.1.1669709005
[createdump] Written 220954624 bytes (53944 pages) to core file
[createdump] Target process is alive
[createdump] Dump successfully written in 186ms
[createdump] Gathering state for process 1 dotnet
[createdump] Crashing thread 00000001 signal 0000000b
[createdump] Writing full dump to file /tmp/crashdump.1.1669709006
[createdump] Written 220954624 bytes (53944 pages) to core file
[createdump] Target process is alive
[createdump] Dump successfully written in 222ms
```

## An observation of something that causes only a single dump to be created

It seems that the Kubernetes setting `shareProcessNamespace: true` causes a single crash dump to be created. The mechanism of this is out of scope of this README. Furthermore, `shareProcessNamespace: true` is not an acceptable workaround to prevent double crash dumps from being created.

1. Run a Kubernetes pod in which the createdump tool creates one crash dump for a single crash:
`kubectl apply -f singledump.yaml`

1. View the evidence, one crash dumps is created:
`kubectl logs singledump`

```
Hello, World!
Unhandled exception. System.Exception: Crash
   at Program.<Main>$(String[] args) in [redacted]\createdump-doubledump-bug\CrashingApplication\Program.cs:line 4
[createdump] Gathering state for process 63 dotnet
[createdump] Crashing thread 0000003f signal 00000006
[createdump] Writing full dump to file /tmp/crashdump.63.1669709372
[createdump] Written 220954624 bytes (53944 pages) to core file
[createdump] Target process is alive
[createdump] Dump successfully written in 202ms
```
