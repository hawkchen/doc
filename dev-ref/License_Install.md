Only Keikai EE requires a license file. You can choose one of the following
ways to put a license file:

# Default License Loading Path

ZSS loads a license file from the default path:

**`WEB-INF/classes/metainfo/zss/license/`**

Create the path above if it does not exist, and put the license key into
the path. This is the simplest way if you just have one zss-based web
application.

# Specify an Absolute Path with a Library Property

Some application servers like Weblogic could fail to locate the license
file in the default path. Then you can specify the absolute path of the
license with the following library property in `zk.xml` and copy your
ZSS license file there.

``` xml
<library-property>
    <name>io.keikait.Runtime.directory</name>
    <value>c:/systemAbsolutePath/my-licenses/</value>
</library-property>
```

It's also a way that multiple ZSS-bases applications can load the same
license file.

# Specify the Path in a System Property

Because
[`Library.getProperty()`](https://www.zkoss.org/javadoc/latest/zk/org/zkoss/lang/Library.html#getProperty(java.lang.String) will look for a system property if no
corresponding property defined in `zk.xml`, you can also pass the
license file path to ZSS via a system property.

For example in a Tomcat, you can add a `setenv.sh` (or `setenv.bat`)
that contains

``` text
export CATALINA_OPTS="$CATALINA_OPTS -Dio.keikait.Runtime.directory=/absolutePathToYourLicenseFilePath/"
```

Tomcat `catalina.sh` will invoke this script if exists.

Refer to your application server's documentation to set a system
property. In conclusion, you can pass the path in any way that can be
retrieved by Java's `System.getProperty()`, e.g. a system variable in
Windows.

# License Information

If the license is loaded successfully, you should see a license
information like below printed in your application server's console when
the server starts like:

``` text
*** Potix Corporation License Information ***

     Licensed Company: my company
     Certificate Number: 123456       
     Licensed Product: Keikai spreadsheet EE
     ...

     To renew, obtain more licenses, or if you require help, please contact info@zkoss.org.
```