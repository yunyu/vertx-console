Vert.x Console
==

This is a configurable administration console for Vert.x applications that provides a web interface for common administration and monitoring tasks.
The backend component uses Vert.x-Web, which is required for the console.

To use, merge the following into your POM (or the equivalent into your Gradle build script):

    <repositories>
        <repository>
            <id>jitpack.io</id>
            <url>https://jitpack.io</url>
        </repository>
    </repositories>

    <properties>
        <vertx.console.version>470c8346ef</vertx.console.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-base</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>
        <!-- Insert console pages here, e.g. -->
        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-services</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>
    </dependencies>

Then, create a `WebConsoleRegistry` in your application with a specified path, add the pages you wish to display, and mount it to a router:

	// Example with several pages loaded
    WebConsoleRegistry.create("/admin")
            .addPage(MetricsConsolePage.create(dropwizardRegistry))
            .addPage(ServicesConsolePage.create(discovery))
            .addPage(LoggingConsolePage.create())
            .addPage(CircuitBreakersConsolePage.create())
            .addPage(ShellConsolePage.create())
            .addPage(HealthConsolePage.create(healthChecks))
            .setCacheBusterEnabled(true) // Adds random query string to scripts
            .mount(vertx, router);

The available pages and their setup instructions are listed below.

For security reasons, you should set up an authentication mechanism and CSRF handler for the `/admin/*` route.
See Vert.x Web documentation for details.

The console will be accessible at the specified path (`/admin` for this example).

vertx-console-metrics
==

This page displays an overview of your application, and displays several important metrics (heap usage, HTTP requests, event bus, etc...).
It requires the following dependencies (note: the versions listed may not be the most recent, you can use newer versions):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-metrics</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-dropwizard-metrics</artifactId>
            <version>3.4.2</version>
        </dependency>
        <dependency>
            <groupId>io.prometheus</groupId>
            <artifactId>simpleclient_hotspot</artifactId>
            <version>0.0.23</version>
        </dependency>
        <dependency>
            <groupId>io.prometheus</groupId>
            <artifactId>simpleclient_dropwizard</artifactId>
            <version>0.0.23</version>
        </dependency>
        <dependency>
            <groupId>com.github.yunyu</groupId>
            <artifactId>prometheus-jvm-extras</artifactId>
            <version>1.1-SNAPSHOT</version>
        </dependency>

Once these have been added, enable metrics when starting your application and [set a name for the registry](http://vertx.io/docs/vertx-dropwizard-metrics/java/#_command_line_activation). For example, you can add the following flags:

    -Dvertx.metrics.options.enabled=true -Dvertx.metrics.options.registryName=vertx-dw

Then, acquire a reference to the metrics registry to create the page. For example:

        MetricRegistry dropwizardRegistry = SharedMetricRegistries.getOrCreate(
                System.getProperty("vertx.metrics.options.registryName") // or use hardcoded name
        );
        // Set up web console registry
        webConsoleRegistry.addPage(MetricsConsolePage.create(dropwizardRegistry));

The metrics page uses the default Prometheus registry. If you wish to use it with another registry, register the appropriate collectors as [listed here](https://github.com/yunyu/vertx-console-metrics/blob/49a70a5ea43f9bf7d6961278a756d378225f1748/src/main/java/in/yunyul/vertx/console/metrics/MetricsConsolePage.java#L24), and use `MetricsConsolePage.create(CollectorRegistry myPrometheusRegistry)`.

vertx-console-services
==

This page displays a filterable list of the service records available to Vert.x.
It requires the following dependencies (note: the versions listed may not be the most recent, you can use newer versions):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-services</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-service-discovery</artifactId>
            <version>3.4.2</version>
        </dependency>

Once these have been added, [create a service discovery instance](http://vertx.io/docs/vertx-service-discovery/java/#_creating_a_service_discovery_instance) and pass it to a console page instance. For example:

        // ServiceDiscovery discovery = ...
        // Set up web console registry
        webConsoleRegistry.addPage(ServicesConsolePage.create(discovery))
