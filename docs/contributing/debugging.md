# Debugging

Debugging javaagent instrumentation can be a challenging task since instrumentation
code is directly inlined into target classes.

## Advice methods

Breakpoints do not work in advice methods, because their code is directly inlined
by ByteBuddy into the target class. It is good to keep these methods as small as possible.
The advice methods are annotated with:

```java
@net.bytebuddy.asm.Advice.OnMethodEnter
@net.bytebuddy.asm.Advice.OnMethodExit
```

These annotations have an option to disable inlining which can allow breakpoints to work within
advice methods. This should only be used for debugging and may break things. As such, it is best to
first try debugging the methods that advice is calling rather than the advice method itself.

```java
@Advice.OnMethodEnter(inline = false)
```

When inlined, the best approach to debug advice methods and agent initialization is to use the
following statements:

```java
System.out.println();
Thread.dumpStack();
```

## Agent initialization code

If you want to debug agent initialization code (e.g. `OpenTelemetryAgent`, `AgentInitializer`,
`AgentInstaller`, `OpenTelemetryInstaller`, etc.) then it's important to specify the `-agentlib:` JVM arg
before the `-javaagent:` JVM arg and use `suspend=y` (see full example below).

## Enabling debugging

The following example shows remote debugger configuration. The breakpoints
should work in any code except ByteBuddy advice methods.

```bash
java -agentlib:jdwp="transport=dt_socket,server=y,suspend=y,address=5000" -javaagent:opentelemetry-javaagent-<version>.jar -jar app.jar
```

## Shadow Renaming

One additional thing that complicates debugging is that certain packages are renamed to avoid
conflict with whatever might already be on the classpath. This results in classes that don't match
what the IDE is expecting for debug breakpoints. You can disable the shadow renaming for local
builds by adding the following to `~/.gradle/gradle.properties` before building.

```properties
disableShadowRelocate=true
```
