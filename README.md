## XSpec Plugin for Maven

A very simple plugin for Maven that will execute your XSpec tests as part of the *test* phase of your Maven build, reports are generated and if any tests fail the build will be failed.
The XSpec Maven plugin is licensed under the [BSD license](http://opensource.org/licenses/BSD-3-Clause). The plugin bundles aspects of the XSpec processor implementaion (written in XSLT) from https://github.com/xspec/xspec which is released under the [MIT license](http://opensource.org/licenses/MIT). 

***Note*** at present only XSpec tests written in XSLT are supported. It should not be too difficult to add support for XQuery as well for a future release.

By default the plugin expects to find your tests in `src/test/xspec` and both XML and HTML reports will be generated into `target/xspec-reports`. In addition the XSLT compiled version of your XSpecs will be placed in `target/xspec-reports/xslt` for reference if you need to debug your tests.


### Goals

The plugin binds to the *verify* phase by default and there is only one goal: `run-xspec`.
The plugin has been published to [Maven Central](http://search.maven.org/#search%7Cga%7C1%7Ca%3A%22xspec-maven-plugin%22)

__Plugin declaration__
```xml
<build>
  <plugins>
    <plugin>
      <groupId>io.xspec.maven</groupId>
      <artifactId>xspec-maven-plugin</artifactId>
      <version>2.0.0</version>
      <dependencies>
        <!-- if you have a license, feel free to add Saxon-PE
           or Saxon-EE instead of Saxon-HE -->
        <dependency>
          <groupId>net.sf.saxon</groupId>
          <artifactId>Saxon-HE</artifactId>
          <!-- Saxon from 9.7.0-14 up until 10.1 have been tested and work correctly -->
          <version>10.1</version>
        </dependency>
        <!-- Saxon >= 10.0 requires XSpec 1.6. The bundled XSpec 1.5
             works correctly with Saxon 9.x, and you don't need this
             extra dependency -->
        <dependency>
          <groupId>io.xspec</groupId>
          <artifactId>xspec</artifactId>
          <version>1.6.0</version>
        </dependency>
      </dependencies>
      <configuration>
        <catalogFile>catalog.xml</catalogFile>
        <generateSurefireReport>true</generateSurefireReport>
        <saxonOptions>See https://github.com/xspec/xspec-maven-plugin-1/wiki</saxonOptions>
      </configuration>
      <executions>
        <execution>
          <phase>test</phase>
          <goals>
            <goal>run-xspec</goal>
          </goals>
        </execution>
      </executions>
    </plugin>
  </plugins>
</build>
```

### Configuration

There are several configuration options that you may specify:

* xspecCompiler
This is the path to the XSpec Compiler XSLT i.e. the `compiler/generate-xspec-tests.xsl` as provided by the de-facto XSpec processor implementation.
By default the XSpec compiler bundled in the XSpec Plugin in used, however this configuration option allows you to specify a custom version.

* xspecReporter
This is the path to the XSpec Reporter XSLT i.e. the `reporter/format-xspec-report.xsl` as provided by the de-facto XSpec processor implementation.
By default the XSpec reporter bundled in the XSpec Plugin in used, however this configuration option allows you to specify a custom version.

* testDir
This is the path to a folder containing your XSpec tests. The tests are expected to be named such that they end with the file extension `.xspec`.
By default the folder `src/test/xspec` is used.

* excludes
You may specify one or more filenames (or partial filenames), which when matached against XSpec paths in *testDir* are excluded from being executed.

* reportDir
This is the path to a folder where the XSpec tests reports will be stored.
By default the folder `target/xspec-reports` is used.

* catalogFile
This is the path to a catalog file, as defined in https://www.oasis-open.org/committees/entity/spec-2001-08-06.html. There is no default value, and is ignored if empty or if catalog file does not exists.

* surefireReportDir
This is the path where to write surefire reports, if '${generateSurefireReports} is 'true'. Default value is '${project.build.directory}/surefire-report'.

* generateSurefireReport
If set to true, generates a surefire report in '${surefireReportDir}'.

* saxonOptions
Allows to specify saxon configuration options. See [Wiki](https://github.com/xspec/xspec-maven-plugin-1/wiki) for more details.

### FAQ
* Where should I put my XSLT?

You can put it anywhere you like, although within `src/` would make the most sense! We would suggest keeping your XSLT files in `src/main/xsl/`. If you do that, then to reference the XSLT from your XSpec, you should set the `@template` attribute use relative path to that folder. For example, given `src/main/xsl/some.xslt` and `src/test/xspec/some.xspec`, your `some.xspec` would reference `some.xslt` like so:

```xml
<x:description xmlns:x="http://www.jenitennison.com/xslt/xspec"
  stylesheet="../../main/xsl/some.xslt">
  
  ...
```

* How can I skip the XSpec tests?

XSpec will adhere to the Maven option `-DskipTests`.
If you are doing this in a forked execution such as that used by the Maven Release plugin you may also have to use the Maven option `-Darguments="-DskipTests"`.

* Must I define the Saxon dependency ?

**Yes, you must**. This is to allow to choose between Saxon-HE, Saxon-PE or Saxon-EE, if you have licences. As Maven doesn't provide a mecanism for a default dependency, you must specify it. You can also choose another releases of Saxon ; 9.7.0-x works correctly.

* Must I define the XSpec dependency ?

The plugin bundles XSpec 1.5, but that isn't compatible with Saxon >= 10.0.

Saxon 10.x requires XSpec >= 1.6, which you need to add as a dependency.

If you're using Saxon 9.x, you don't need to specify the XSpec dependency.

* How are surefire reports generated ?

Surefire report is generated from the XSpec report, via a XSL. At this time, transformation should be improved, to have a good report in Jenkins. Any help will be appreciated.
