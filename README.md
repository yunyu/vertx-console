Vert.x Console
==

This is a configurable, extensible administration console for Vert.x applications that provides a web interface for common administration and monitoring tasks. A video demo is available [here](https://www.youtube.com/watch?v=I3ZWuOngesU).
The backend component requires Vert.x-Web.

The frontend uses the [PatternFly](http://www.patternfly.org/) CSS framework with [Vue.js](https://vuejs.org/) components and the [axios](https://github.com/mzabriskie/axios) HTTP client. Note that there is no requirement for console pages to be written with Vue, simply exporting a Vue-compatible [render function](https://vuejs.org/v2/guide/render-function.html) and appropriate `mounted` and `destroyed` methods (referencing `this.$el`) will allow you to use any framework you wish.

To use, merge the following into your POM (or the equivalent into your Gradle build script):

    <repositories>
        <repository>
            <id>jitpack.io</id>
            <url>https://jitpack.io</url>
        </repository>
    </repositories>

    <properties>
        <vertx.console.version>48a979a2eb</vertx.console.version>
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
    		// Add pages
            .addPage(MetricsConsolePage.create(dropwizardRegistry))
            .addPage(ServicesConsolePage.create(discovery))
            .addPage(LoggingConsolePage.create())
            .addPage(CircuitBreakersConsolePage.create())
            .addPage(ShellConsolePage.create())
            .addPage(HealthConsolePage.create(healthChecks))
            .setCacheBusterEnabled(true) // Adds random query string to scripts
            // Mount to router
            .mount(vertx, router);

The available pages and their setup instructions are listed below.

For security reasons, you should set up an authentication mechanism and CSRF handler for the `/admin/*` route.
See Vert.x Web documentation for details ([regarding auth](http://vertx.io/docs/vertx-web/java/#_creating_an_auth_handler), [regarding CSRF protection](http://vertx.io/docs/vertx-web/java/#_csrf_cross_site_request_forgery)). Also consider enabling [gzip compression](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionSupported-boolean-) in production, as it could potentially reduce bandwidth usage by a sizable amount.

The console will be accessible at the specified path (`/admin` in this example).

vertx-console-metrics
==

![](https://i.imgur.com/iMmH2SY.png)

This page displays an overview of your application, and includes several important metrics (heap usage, HTTP requests, event bus, etc...) as well as the ability to deploy verticles.
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

![](https://i.imgur.com/dfbvdHO.png)

This page displays a filterable list of the service records available to Vert.x.
It requires the following dependency (assuming that you have already [set up service discovery](http://vertx.io/docs/vertx-service-discovery/java/#_creating_a_service_discovery_instance)):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-services</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>

Once these have been added, pass your service discovery instance to the console page. For example:

        // ServiceDiscovery discovery = ...
        // Set up web console registry
        webConsoleRegistry.addPage(ServicesConsolePage.create(discovery));

vertx-console-logging
==

![](https://i.imgur.com/KU8PoSX.png)

This page allows you to view and configure loggers and their outputs. It is currently only compatible with Logback and SLF4J (due to difficulties with integrating appenders with Log4J2).
It requires the following dependency (assuming that you already have Logback and SLF4J configured in your application):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-logging</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>

Once this has been added, you can add the console page directly. For example:

        // Set up web console registry
        webConsoleRegistry.addPage(LoggingConsolePage.create());

vertx-console-circuit-breakers
==

![](https://i.imgur.com/G56nGtH.png)

This page allows you to view the status of the circuit breakers in your application.
It requires the following dependency (assuming that you have already [set up circuit breakers](http://vertx.io/docs/vertx-circuit-breaker/java/#_using_the_vert_x_circuit_breaker)):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-circuit-breakers</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>

Once this has been added, you can add the console page directly. For example:

        // Set up web console registry
        webConsoleRegistry.addPage(CircuitBreakersConsolePage.create());

vertx-console-shell
==

![](https://i.imgur.com/DlmyqeK.png)

This page allows you to administer your application via [Vert.x-Shell](http://vertx.io/docs/vertx-shell/java/#_base_commands).
It requires the following dependencies (note: the versions listed may not be the most recent, you can use newer versions):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-shell</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-shell</artifactId>
            <version>3.4.2</version>
        </dependency>

Once these have been added, you can add the console page directly. For example:

        // Set up web console registry
        webConsoleRegistry.addPage(ShellConsolePage.create());

vertx-console-health
==

![](https://i.imgur.com/W4tKZtw.png)

This page allows you to view the status of the health checks in your application.
It requires the following dependency (assuming that you have already [set up health checks](http://vertx.io/docs/vertx-health-check/java/#_using_vert_x_health_checks)):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-health</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>

Once these have been added, pass your health checks instance to the console page. For example:

        // Set up web console registry
        webConsoleRegistry.addPage(HealthConsolePage.create(healthChecks));

vertx-console-pools
==

![](https://i.imgur.com/9w4kk5U.png)

This page allows you to view the status of the worker and data source pools in your application.
You need to have a working metrics setup in order to use it (see the [vertx-console-metrics](#vertx-console-metrics) section for details), but adding the Overview page is optional.
It requires the following dependency (on top of the ones necessary for metrics):

        <dependency>
            <groupId>com.github.yunyu.vertx-console</groupId>
            <artifactId>vertx-console-pools</artifactId>
            <version>${vertx.console.version}</version>
        </dependency>

Once these have been added, pass your MetricsService instance to the console page. For example:

        // Set up web console registry
        MetricsService metricsService = MetricsService.create(vertx);
        webConsoleRegistry.addPage(PoolsConsolePage.create(metricsService));

API
==

Documentation for the API is currently in-progress. In the meantime, an [example console page](https://github.com/yunyu/vertx-console-example) template is available.