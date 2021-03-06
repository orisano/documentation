---
title: Getting Started
kind: Documentation
aliases:
    - /tracing/profiling/getting_started
further_reading:
    - link: 'tracing/profiler/search_profiles'
      tag: 'Documentation'
      text: 'Learn more about available profile types.'
    - link: 'tracing/profiler/profiler_troubleshooting'
      tag: 'Documentation'
      text: 'Fix problems you encounter while using the profiler'
    - link: 'https://www.datadoghq.com/blog/introducing-datadog-profiling/'
      tags: 'Blog'
      text: 'Introducing always-on production profiling in Datadog.'
---

Profiler is shipped within the following tracing libraries. Select your language below to learn how to enable profiler for your application:

To get notified when a private beta is available for the **Node**, **Ruby**, **PHP**, or **.NET** Profiler, [sign up here][2].

{{< tabs >}}
{{% tab "Java" %}}

The Datadog Profiler requires [JDK Flight Recorder][1]. The Datadog Profiler library is supported in OpenJDK 11+, Oracle Java 11+, [OpenJDK 8 for most vendors (version 8u262)][2] and Zulu Java 8+ (minor version 1.8.0_212+). All JVM-based languages, such as Scala, Groovy, Kotlin, and Clojure are supported. To begin profiling applications:

1. If you are already using Datadog, upgrade your agent to version [7.20.2][3]+ or [6.20.2][3]+. If you don't have APM enabled to set up your application to send data to Datadog, in your Agent, set the `DD_APM_ENABLED` environment variable to `true` and listening to the port `8126/TCP`.

2. Download `dd-java-agent.jar`, which contains the Java Agent class files:

    ```shell
    wget -O dd-java-agent.jar 'https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=com.datadoghq&a=dd-java-agent&v=LATEST'
    ```

     **Note**: Profiler is available in the `dd-java-agent.jar` library in versions 0.55+.

3. Set `-Ddd.profiling.enabled` flag or `DD_PROFILING_ENABLED` environment variable to `true`. Update to your service invocation should look like:

    ```diff
    java -javaagent:dd-java-agent.jar -Ddd.profiling.enabled=true -jar <YOUR_SERVICE>.jar <YOUR_SERVICE_FLAGS>
    ```

4. After a minute or two, visualize your profiles on the [Datadog APM > Profiling page][4].


**Note**:

- The `-javaagent` argument needs to be before `-jar`, adding it as a JVM option rather than an application argument. For more information, see the [Oracle documentation][5]:

    ```shell
    # Good:
    java -javaagent:dd-java-agent.jar ... -jar my-service.jar -more-flags
    # Bad:
    java -jar my-service.jar -javaagent:dd-java-agent.jar ...
    ```

- We strongly recommend that you specify the `service` and `version` as this gives you the ability to slice and dice your profiles across these dimensions. Use environment variables to set the parameters:

| Environment variable                             | Type          | Description                                                                                      |
| ------------------------------------------------ | ------------- | ------------------------------------------------------------------------------------------------ |
| `DD_PROFILING_ENABLED`                           | Boolean       | Alternate for `-Ddd.profiling.enabled` argument. Set to `true` to enable profiler.               |
| `DD_SERVICE`                                     | String        | Your [service][3] name, for example, `web-backend`.     |
| `DD_ENV`                                         | String        | Your [environment][6] name, for example: `production`.|
| `DD_VERSION`                                     | String        | The version of your service.                             |
| `DD_TAGS`                                        | String        | Tags to apply to an uploaded profile. Must be a list of `<key>:<value>` separated by commas such as: `layer:api, team:intake`.  |

- By default, The Datadog Profiler keeps the bottom 256 frames in a Java profile. If your application creates stacks deeper than 256 frames, you can increase the threshold by passing a `-XX:FlightRecorderOptions=stackdepth=` option to your JVM.  For example:
    ```shell
    java -javaagent:dd-java-agent.jar -XX:FlightRecorderOptions=stackdepth=512 ...
    ```

[1]: https://docs.oracle.com/javacomponents/jmc-5-4/jfr-runtime-guide/about.htm
[2]: /tracing/profiler/profiler_troubleshooting/#java-8-support
[3]: https://app.datadoghq.com/account/settings#agent/overview
[4]: https://app.datadoghq.com/profiling
[5]: https://docs.oracle.com/javase/7/docs/technotes/tools/solaris/java.html
[6]: /tracing/guide/setting_primary_tags_to_scope/#environment
{{% /tab %}}

{{% tab "Python" %}}

The Datadog Profiler requires Python 2.7+. Memory profiling is available on Python 3.5+. To begin profiling applications:

1. If you are already using Datadog, upgrade your agent to version [7.20.2][1]+ or [6.20.2][1]+.

2. Install `ddtrace` which contains both tracing and profiler:

    ```shell
    pip install ddtrace
    ```

     **Note**: Profiler is available in the `ddtrace` library for versions 0.36+.

3. To automatically profile your code, set `DD_PROFILING_ENABLED` environment variable to `true` when you use `ddtrace-run`:

    ```
    DD_PROFILING_ENABLED=true ddtrace-run python app.py
    ```
    **Note:** `DD_PROFILING_ENABLED` is only supported in `dd-trace` version 0.40+. Use the alternate method if you are running an older version of `dd-trace`.

    **Alternate method**

    If you prefer to instrument the profiler through code, import `ddtrace.profile.auto`. After import, the Profiler starts:

    ```python
    import ddtrace.profiling.auto
    ```

4. After a minute or two, visualize your profiles on the [Datadog APM > Profiler page][2].

**Note**:

- Alternatively, profile your service by running it with the wrapper `pyddprofile`:

    ```shell
    $ pyddprofile server.py
    ```

- It is strongly recommended to add tags like `service` or `version` as it provides the ability to slice and dice your profiles across these dimensions, enhancing your overall product experience. Use environment variables to set the parameters:

| Environment variable                             | Type          | Description                                                                                      |
| ------------------------------------------------ | ------------- | ------------------------------------------------------------------------------------------------ |
| `DD_PROFILING_ENABLED`                           | Boolean       | Set to `true` to enable profiler. Supported from tracker version 0.40+.              |
| `DD_SERVICE`                                     | String        | The Datadog [service][3] name.     |
| `DD_ENV`                                         | String        | The Datadog [environment][4] name, for example, `production`. |
| `DD_VERSION`                                     | String        | The version of your application.                             |
| `DD_TAGS`                                        | String        | Tags to apply to an uploaded profile. Must be a list of `<key>:<value>` separated by commas such as: `layer:api,team:intake`.   |

<div class="alert alert-info">
Recommended for advanced usage only.
</div>

- When your process forks using `os.fork`, the profiler is stopped in the child process.

  For Python 3.7+ on POSIX platforms, a new profiler is started if you enabled the profiler via `pyddprofile` or `ddtrace.profiling.auto`.

  If you manually create a `Profiler()`, use Python < 3.6, or run on a non-POSIX platform, manually restart the profiler in your child with:

   ```python
   ddtrace.profiling.auto.start_profiler()
   ```

- If you want to manually control the lifecycle of the profiler, use the `ddtrace.profiling.profiler.Profiler` object:

    ```python
    from ddtrace.profiling import Profiler

    prof = Profiler()
    prof.start()

    # At shutdown
    prof.stop()
    ```

[1]: https://app.datadoghq.com/account/settings#agent/overview
[2]: https://app.datadoghq.com/profiling
[3]: /tracing/visualization/#services
[4]: /tracing/guide/setting_primary_tags_to_scope/#environment
{{% /tab %}}

{{% tab "Go" %}}

The Datadog Profiler requires Go 1.12+. To begin profiling applications:

1. If you are already using Datadog, upgrade your agent to version [7.20.2][1]+ or [6.20.2][1]+.

2. Get `dd-trace-go` using the command:

    ```shell
    go get gopkg.in/DataDog/dd-trace-go.v1
    ```

     **Note**: Profiler is available in the `dd-trace-go` library for versions 1.23.0+.

3. Import the [profiler][2] at the start of your application:

    ```Go
    import "gopkg.in/DataDog/dd-trace-go.v1/profiler"
    ```

4. Add the following snippet to start the profiler:

    ```Go
    err := profiler.Start(
        profiler.WithService("<SERVICE_NAME>"),
        profiler.WithEnv("<ENVIRONMENT>"),
        profiler.WithVersion("<APPLICATION_VERSION>"),
        profiler.WithTags("<KEY1>:<VALUE1>,<KEY2>:<VALUE2>"),
    )
    if err != nil {
        log.Fatal(err)
    }
    defer profiler.Stop()
    ```

4. After a minute or two, visualize your profiles in the [Datadog APM > Profiler page][3].

**Note**:

- It is strongly recommended to add tags like `service` or `version` as it provides the ability to slice and dice your profiles across these dimensions, enhancing your overall product experience. Use profiler configuration to set the parameters:

| Method | Type          | Description                                                                                                  |
| ---------------- | ------------- | ------------------------------------------------------------------------------------------------------------ |
|  WithService     | String        | The Datadog [service][4] name, for example `my-web-app`.             |
|  WithEnv         | String        | The Datadog [environment][5] name, for example, `production`.         |
|  WithVersion     | String        | The version of your application.                                                                             |
|  WithTags        | String        | The tags to apply to an uploaded profile. Must be a list of in the format `<KEY1>:<VALUE1>,<KEY2>:<VALUE2>`. |

- Alternatively you can also set the profiler configuration using environment variables:

| Environment variable                             | Type          | Description                                                                                      |
| ------------------------------------------------ | ------------- | ------------------------------------------------------------------------------------------------ |
| `DD_SERVICE`                                     | String        | The Datadog [service][4] name.     |
| `DD_ENV`                                         | String        | The Datadog [environment][5] name, for example, `production`. |
| `DD_VERSION`                                     | String        | The version of your application.                             |
| `DD_TAGS`                                        | String        | Tags to apply to an uploaded profile. Must be a list of `<key>:<value>` separated by commas such as: `layer:api,team:intake`.   |

[1]: https://app.datadoghq.com/account/settings#agent/overview
[2]: https://godoc.org/gopkg.in/DataDog/dd-trace-go.v1/profiler#pkg-constants
[3]: https://app.datadoghq.com/profiling
[4]: /tracing/visualization/#services
[5]: /tracing/guide/setting_primary_tags_to_scope/#environment
{{% /tab %}}

{{< /tabs >}}


## Further Reading

{{< partial name="whats-next/whats-next.html" >}}

